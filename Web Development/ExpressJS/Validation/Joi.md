# Joi — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Joi is the most powerful schema description language and data validator for JavaScript, letting you describe data using a simple, intuitive, and readable language.

**Technical Definition**
Joi is an object schema validation library for JavaScript that constructs immutable schema objects from type factories (`Joi.string()`, `Joi.object()`, etc.) and chainable rules. Validation is a two-step process: a schema is constructed using types and constraints, and then a value is validated against the schema with `schema.validate(value)` or `schema.validateAsync(value)`. The result is an object containing `{ error, value, warning, debug, artifacts }`; if the input is valid, `error` is `undefined`, and if invalid, `error` is a `ValidationError` with structured `details`. Joi schemas are immutable: every rule addition returns a new schema rather than modifying the original. It provides over 150 built-in validators across strings, numbers, dates, arrays, objects, and binaries, with chainable rules that read like English.

**Beginner-Friendly Explanation**
Joi lets you describe what “correct data” looks like using a language that reads almost like a sentence: `Joi.string().email().required()` means “a string, must be a valid email, and is required.” You build a schema once, and Joi checks incoming data against it. If the data is good, you get the cleaned, validated value; if not, Joi tells you exactly what went wrong and why. It is like a quality-control inspector for your API data.

---

### Key Characteristics

- **Expressive, chainable rules** — Over 150 built-in validators with readable method chains such as `.min(3).max(30).alphanum().
- **Immutable schemas** — Every rule addition returns a new schema; the original is never mutated, enabling safe reuse and composition.
- **Two-step usage** — Construct a schema, then validate a value against it.
- **Rich error reporting** — `ValidationError` objects include `details` with `path`, `message`, `type`, and `context` for field-level feedback.
- **Sync and async validation** — `validate()` is synchronous; `validateAsync()` returns a Promise and is required when `.external()` validators are used.
- **Powerful composition** — `concat()`, `fork()`, `extend()`, `link()`, and `when()` enable reusable, conditional, and recursive schemas.

---

### Prerequisites

- Node.js installed and a project initialized with `npm init`.
- The `joi` package installed (`npm install joi`).
- Basic JavaScript knowledge (objects, arrays, functions, Promises).
- For Express integration: an Express app with `express.json()` or `express.urlencoded()` middleware enabled.

---

### Related Programming Areas

- **Express validation** — Joi schemas are commonly wrapped in Express middleware that validates `req.body`, `req.query`, or `req.params`.
- **API design** — Schemas define the input contract of an API.
- **Security** — Validation prevents malformed or malicious data from reaching databases and business logic.
- **Schema composition** — Reusable schema components reduce duplication across endpoints.
- **Error handling** — Joi’s `ValidationError` integrates with centralized error middleware.

---

## Core Concept 1: Joi

### Definitions

**Core Definition**
Joi is a schema-based validation library that lets you define blueprints for JavaScript objects and validate data against those blueprints.

**Technical Definition**
Joi exposes type factories (`Joi.string()`, `Joi.number()`, `Joi.object()`, `Joi.array()`, `Joi.alternatives()`, etc.) that return schema objects inheriting from a common `Base` class. These schemas are immutable and support method chaining for adding rules, metadata, defaults, and conditional logic. Validation is performed with `schema.validate(value, [options])`, which returns `{ error, value, warning, debug, artifacts }`. Joi does not throw on validation failure by default; it returns the error in the result object, allowing the caller to decide how to respond.

**Beginner-Friendly Explanation**
Joi is the library itself. You import it, create schemas with it, and use it to validate data. It does not do anything on its own until you tell it what rules to apply.

---

### Purposes

- To provide a readable, declarative language for describing data shapes and constraints.
- To validate incoming data with minimal imperative code.
- To produce structured error information for client feedback.
- To support both synchronous and asynchronous validation workflows.

---

### Syntax Rules and Structure

#### Complete General Syntax: Basic Schema and Validation

```javascript
const Joi = require('joi');

const schema = Joi.object({
  username: Joi.string().alphanum().min(3).max(30).required(),
  password: Joi.string().min(8).required()
});

const { error, value } = schema.validate({
  username: 'johndoe',
  password: 'secret123'
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `Joi.object({ ... })` | Creates an object schema |
| `Joi.string()` | Creates a string schema |
| `.required()` | Makes the field mandatory |
| `schema.validate(value)` | Validates the value; returns `{ error, value }` |

#### Syntax Rules

- Schemas can be built from type factories or plain objects where every key is assigned a Joi type.
- Values are optional by default; use `.required()` to mandate presence.
- Schemas are immutable; each rule returns a new schema.
- `validate()` is synchronous; `validateAsync()` returns a Promise.

#### Constraints and Limitations

- Joi is not a sanitizer by default; it validates and can coerce types, but does not automatically escape HTML.
- Complex schemas can become large; composition and reusable components are essential.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Basic User Schema

```javascript
const Joi = require('joi');

const userSchema = Joi.object({
  username: Joi.string().alphanum().min(3).max(30).required(),
  email: Joi.string().email({ minDomainSegments: 2 }).required(),
  password: Joi.string().min(8).required()
});

const validData = { username: 'alice', email: 'alice@example.com', password: 'secret123' };
const invalidData = { username: 'a!', email: 'not-an-email', password: '123' };

console.log(userSchema.validate(validData));
// { value: { username: 'alice', email: 'alice@example.com', password: 'secret123' } }

console.log(userSchema.validate(invalidData).error.details);
// [
//   { message: '"username" must only contain alpha-numeric characters', path: ['username'], ... },
//   { message: '"username" length must be at least 3 characters long', path: ['username'], ... },
//   { message: '"email" must be a valid email', path: ['email'], ... },
//   { message: '"password" length must be at least 8 characters long', path: ['password'], ... }
// ]
```

**Expected Output**

```
Valid data: { value: { username: 'alice', email: 'alice@example.com', password: 'secret123' } }
Invalid data: structured error details with field-level messages
```

**Why This Output Occurs**

- Joi applies all rules in the chain; valid data passes and is returned in `value`, while invalid data produces a `ValidationError` whose `details` array describes each failure.

---

### Real-World Cases

- **Express API request validation** — Validating `req.body` before database insertion.
- **Form validation** — Reusing the same schemas on the client and server.
- **Configuration validation** — Ensuring environment variables and config files conform to expected shapes.

---

## Core Concept 2: Schemas

### Definitions

**Core Definition**
A Joi schema is an immutable blueprint that describes the expected type, constraints, and behavior of a value.

**Technical Definition**
Schemas are created by Joi’s type factories and inherit from the `Base` class. Each schema contains flags (e.g., `presence`, `default`, `description`, `label`), terms (e.g., `whens`, `examples`), and a list of rules. Object schemas support `.keys()`, `.pattern()`, `.and()`, `.or()`, `.xor()`, `.with()`, and more. Array schemas support `.items()`, `.min()`, `.max()`, `.unique()`, and `.ordered()`. Alternative schemas support `.try()` for multiple valid types.

**Beginner-Friendly Explanation**
A schema is the blueprint for your data. It says “this field must be a string between 3 and 30 characters” or “this field is a number and must be at least 18.” You create one schema per shape of data and reuse it wherever that shape appears.

---

### Purposes

- To define the exact shape and constraints of data in a reusable, declarative form.
- To support nested objects, arrays, and alternative types.
- To attach metadata (`description`, `label`, `example`) for documentation and error messages.
- To enable default values and failover behavior.

---

### Syntax Rules and Structure

#### Complete General Syntax: Schema Types and Common Rules

```javascript
const schema = Joi.object({
  username: Joi.string().alphanum().min(3).max(30).required(),
  age: Joi.number().integer().min(18).max(120),
  email: Joi.string().email(),
  tags: Joi.array().items(Joi.string()).min(1).max(5),
  address: Joi.object({
    street: Joi.string().required(),
    city: Joi.string().required()
  }),
  id: Joi.alternatives().try(Joi.string().uuid(), Joi.number().integer())
});
```

**Component Breakdown**

| Schema Type | Common Rules |
|-------------|-------------|
| `Joi.string()` | `.alphanum()`, `.min()`, `.max()`, `.pattern()`, `.email()`, `.uri()` |
| `Joi.number()` | `.integer()`, `.min()`, `.max()`, `.positive()`, `.precision()` |
| `Joi.array()` | `.items()`, `.min()`, `.max()`, `.unique()` |
| `Joi.object()` | `.keys()`, `.pattern()`, `.and()`, `.or()`, `.xor()`, `.with()` |
| `Joi.alternatives()` | `.try()` |

#### Syntax Rules

- Object keys are optional by default; use `.required()` to mandate them.
- `.default(value)` applies a value when the input is `undefined`.
- `.label(name)` customizes the field name used in error messages.
- `.when(ref, options)` adds conditional rules based on other fields.

#### Constraints and Limitations

- Empty strings are not allowed by default; use `.allow('')` to permit them.
- Schemas are immutable; each rule returns a new schema, so reassign the result when building conditionally.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Nested Object and Array Schema

```javascript
const Joi = require('joi');

const orderSchema = Joi.object({
  orderId: Joi.string().uuid().required(),
  items: Joi.array().items(
    Joi.object({
      productId: Joi.number().integer().positive().required(),
      quantity: Joi.number().integer().min(1).required()
    })
  ).min(1).required(),
  shipping: Joi.object({
    street: Joi.string().required(),
    city: Joi.string().required(),
    zip: Joi.string().pattern(/^\d{5}$/).required()
  }).required()
});

const validOrder = {
  orderId: '123e4567-e89b-12d3-a456-426614174000',
  items: [{ productId: 1, quantity: 2 }],
  shipping: { street: 'Main St', city: 'Springfield', zip: '12345' }
};

console.log(orderSchema.validate(validOrder).error); // undefined (valid)
```

**Expected Output**

- Valid data produces `error: undefined`; invalid data produces structured details for each failing field.

---

### Real-World Cases

- **E-commerce orders** — Validating nested order items and shipping addresses.
- **User profiles** — Validating nested address objects and arrays of tags.
- **Configuration schemas** — Describing complex application settings.

---

## Core Concept 3: Schema Validation

### Definitions

**Core Definition**
Schema validation is the act of checking a value against a Joi schema and receiving a result object that indicates success or failure.

**Technical Definition**
`schema.validate(value, [options])` validates synchronously and returns `{ error, value, warning, debug, artifacts }`. If valid, `error` is `undefined`; if invalid, `error` is a `ValidationError` with a `details` array. `schema.validateAsync(value, [options])` returns a Promise and is required when `.external()` validators are used. `Joi.assert(value, schema, [message], [options])` throws if validation fails.

**Beginner-Friendly Explanation**
Validation is running your data through the schema. You get back a result that tells you whether the data passed and, if it failed, what exactly was wrong.

---

### Purposes

- To determine whether a value conforms to a schema.
- To retrieve a cleaned, coerced value on success.
- To obtain structured error details on failure.
- To support both synchronous and asynchronous validation paths.

---

### Syntax Rules and Structure

#### Complete General Syntax: Validation Options

```javascript
const { error, value } = schema.validate(data, {
  abortEarly: false,     // collect all errors
  stripUnknown: true,    // remove unknown keys
  convert: true          // coerce types
});
```

**Component Breakdown**

| Option | Effect |
|--------|--------|
| `abortEarly` | When `true` (default), stops at the first error; when `false`, returns all errors |
| `stripUnknown` | Removes keys not defined in the schema |
| `convert` | Enables type coercion (e.g., string to number) |

#### Syntax Rules

- `abortEarly: false` is essential for comprehensive form validation.
- `stripUnknown: true` prevents mass assignment by removing extra fields.
- Use `validateAsync()` when any chain includes `.external()`.

#### Constraints and Limitations

- With `abortEarly: false`, the `value` may be incomplete if validation failed.
- `stripUnknown` does not strip unknown keys from nested objects unless applied consistently.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Collecting All Errors

```javascript
const schema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required()
});

const { error } = schema.validate(
  { email: 'invalid', password: '123' },
  { abortEarly: false }
);

console.log(error.details);
// [
//   { message: '"email" must be a valid email', path: ['email'], ... },
//   { message: '"password" length must be at least 8 characters long', path: ['password'], ... }
// ]
```

**Expected Output**

- All validation failures are returned together because `abortEarly` is `false`.

---

### Real-World Cases

- **Form validation** — Returning all field errors at once for a better user experience.
- **API input validation** — Collecting all issues before responding with a `400`.
- **Bulk operations** — Identifying all invalid records in a batch.

---

## Core Concept 4: Schema Composition

### Definitions

**Core Definition**
Schema composition is the practice of building complex schemas by combining, extending, or deriving from simpler schemas.

**Technical Definition**
Joi provides several composition mechanisms: `concat(schema)` merges two schemas into a new one; `fork(paths, adjuster)` creates a new schema with modifications to specific paths; `extend(extension)` creates a customized Joi instance with new types or rules; `link(ref)` references another schema for recursive definitions; and `when(ref, options)` applies conditional rules based on other fields.

**Beginner-Friendly Explanation**
Composition lets you avoid copy-pasting schemas. You can start with a base schema and add rules, create variations for different endpoints, or build recursive structures like nested comments.

---

### Purposes

- To reuse common schema fragments across multiple endpoints.
- To create variations of a base schema (e.g., create vs. update) without duplication.
- To define recursive schemas safely with `link()`.
- To apply conditional validation based on other fields with `when()`.

---

### Syntax Rules and Structure

#### Complete General Syntax: Concat, Fork, and Conditional

```javascript
const baseSchema = Joi.object({
  name: Joi.string().required(),
  email: Joi.string().email()
});

// Merge schemas
const extendedSchema = baseSchema.concat(
  Joi.object({ age: Joi.number().integer().min(18) })
);

// Make specific fields required
const requiredSchema = baseSchema.fork(['email'], (schema) => schema.required());

// Conditional validation
const conditionalSchema = Joi.object({
  type: Joi.string().valid('personal', 'business').required(),
  companyName: Joi.when('type', {
    is: 'business',
    then: Joi.string().required(),
    otherwise: Joi.forbidden()
  })
});
```

**Component Breakdown**

| Method | Effect |
|--------|--------|
| `concat(schema)` | Merges two schemas into a new schema |
| `fork(paths, adjuster)` | Returns a new schema with modifications to specified paths |
| `when(ref, options)` | Applies conditional rules based on another field |
| `link(ref)` | References another schema for recursive structures |

#### Syntax Rules

- `concat()` clones the target and merges properties from the source.
- `fork()` does not mutate the original schema; it returns a new one.
- `when()` can use `is`, `then`, `otherwise`, and `switch` for complex conditions.

#### Constraints and Limitations

- `concat()` may produce unexpected results if the two schemas have conflicting rules for the same key.
- Recursive schemas via `link()` require careful placement of the link reference.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Base Schema with Forks for Create and Update

```javascript
const Joi = require('joi');

const baseUser = Joi.object({
  username: Joi.string().alphanum().min(3).max(30),
  email: Joi.string().email(),
  password: Joi.string().min(8)
});

// Create: all fields required
const createUser = baseUser.fork(['username', 'email', 'password'], (s) => s.required());

// Update: all fields optional
const updateUser = baseUser;

console.log(createUser.validate({}).error); // error: username required
console.log(updateUser.validate({}).error); // undefined (valid)
```

**Expected Output**

- `createUser` rejects empty objects because forked fields are required; `updateUser` accepts empty objects because fields are optional.

---

### Real-World Cases

- **CRUD APIs** — A base schema for a resource, with `fork` for create (required) and update (optional).
- **Multi-step forms** — Conditional schemas that reveal fields based on previous answers.
- **Recursive comment threads** — `link()` for nested replies.

---

## Core Concept 5: Custom Validators

### Definitions

**Core Definition**
Custom validators are user-defined functions attached to a Joi schema via `.custom()` or `.external()` that perform validation logic beyond the built-in rules.

**Technical Definition**
`.custom(fn)` adds a synchronous validator. The function receives `(value, helpers)` and must return the value (optionally transformed) or call `helpers.error()` to signal failure. `.external(fn)` adds an asynchronous validator that runs after all other validations and may return a Promise; it is the correct choice for database lookups and other async operations because Joi’s core `validate()` is synchronous. `validateAsync()` must be used when `.external()` is present.

**Beginner-Friendly Explanation**
Custom validators let you add your own rules. Use `.custom()` for synchronous checks (like matching two passwords). Use `.external()` for asynchronous checks (like checking if an email is already in the database).

---

### Purposes

- To enforce application-specific rules not covered by built-in validators.
- To perform asynchronous checks such as database uniqueness.
- To transform values during validation (e.g., trimming, hashing).
- To produce custom error messages for domain-specific failures.

---

### Syntax Rules and Structure

#### Complete General Syntax: Custom and External Validators

```javascript
// Synchronous custom validator
const schema = Joi.object({
  password: Joi.string().min(8).required(),
  confirm: Joi.string().custom((value, helpers) => {
    if (value !== helpers.state.ancestors[0].password) {
      return helpers.error('any.invalid');
    }
    return value;
  })
});

// Asynchronous external validator
const asyncSchema = Joi.object({
  email: Joi.string().email().required().external(async (value, helpers) => {
    const exists = await db.emailExists(value);
    if (exists) {
      throw new Error('Email already exists');
    }
    return value;
  })
});
```

**Component Breakdown**

| Method | Async? | Use Case |
|--------|--------|----------|
| `.custom(fn)` | No | Synchronous checks, cross-field comparisons |
| `.external(fn)` | Yes | Database lookups, external service checks |

#### Syntax Rules

- `.custom()` must return the value or call `helpers.error()`.
- `.external()` may return a Promise; use `validateAsync()` to await it.
- External validators run after all other validations.

#### Constraints and Limitations

- `.custom()` cannot be asynchronous; use `.external()` for async operations.
- External validators add latency; keep database lookups efficient.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Async Email Uniqueness with `.external()`

```javascript
const Joi = require('joi');

const schema = Joi.object({
  email: Joi.string().email().required().external(async (value) => {
    const existing = await db.findByEmail(value);
    if (existing) {
      throw new Error('This email is already registered');
    }
    return value;
  })
});

try {
  await schema.validateAsync({ email: 'taken@example.com' });
} catch (err) {
  console.log(err.message); // "This email is already registered"
}
```

**Expected Output**

- If the email exists, `validateAsync` rejects with the custom error message; if not, validation succeeds.

---

### Real-World Cases

- **User registration** — Checking email or username uniqueness asynchronously.
- **Password confirmation** — Synchronous cross-field matching.
- **Inventory checks** — Asynchronously verifying product availability.

---

## Core Concept 6: Error Messages

### Definitions

**Core Definition**
Error messages in Joi are the human-readable strings produced when validation fails, customizable per rule and per field.

**Technical Definition**
Joi produces a `ValidationError` whose `details` array contains objects with `message`, `path`, `type`, `context`, and `local` properties. Messages can be customized with `.messages({ 'error.type': 'Custom message' })`, where error types include `string.empty`, `string.email`, `string.min`, `any.required`, etc.. `.label(name)` customizes the field name used in messages. `abortEarly: false` ensures all error messages are collected.

**Beginner-Friendly Explanation**
Error messages tell the user what went wrong. Joi gives you default messages, but you can replace them with your own friendly text, like changing “string.min” to “Password must be at least 8 characters.”

---

### Purposes

- To provide clear, user-friendly feedback for invalid input.
- To localize or customize messages for different audiences.
- To map validation failures to field-specific error responses.
- To avoid exposing internal validation rule names to end users.

---

### Syntax Rules and Structure

#### Complete General Syntax: Custom Messages

```javascript
const schema = Joi.object({
  email: Joi.string().email().required().messages({
    'string.empty': 'Email is required',
    'string.email': 'Email must be a valid email address',
    'any.required': 'Email field is mandatory'
  }),
  password: Joi.string().min(8).messages({
    'string.min': 'Password must be at least 8 characters'
  })
});
```

**Component Breakdown**

| Error Type | When It Fires |
|------------|---------------|
| `string.empty` | The field is an empty string |
| `string.email` | The value is not a valid email |
| `string.min` | The string is shorter than the minimum |
| `any.required` | A required field is missing |

#### Syntax Rules

- `.messages()` accepts an object mapping error type codes to custom strings.
- Use `'*'` as a wildcard to apply a message regardless of error type.
- `.label()` changes the field name shown in messages.

#### Constraints and Limitations

- Error type codes are specific to each schema type; refer to the API documentation for the full list.
- Custom messages must not leak sensitive validation logic.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Field-Level Custom Messages

```javascript
const schema = Joi.object({
  email: Joi.string().email().required().messages({
    'string.empty': 'Please enter your email',
    'string.email': 'That doesn’t look like a valid email',
    'any.required': 'Email is required'
  })
});

const { error } = schema.validate({ email: '' });
console.log(error.details[0].message); // "Please enter your email"
```

**Expected Output**

- The custom message replaces the default for the matching error type.

---

### Real-World Cases

- **Public APIs** — Friendly, localized error messages for end users.
- **Form validation** — Field-specific messages displayed next to inputs.
- **Compliance** — Messages that avoid technical jargon.

---

## Core Concept 7: Joi Best Practices

### Definitions

**Core Definition**
Joi best practices are the design and implementation patterns that keep Joi-based validation complete, secure, maintainable, and performant.

**Technical Definition**
Best practices include: validating at the boundary (in middleware before business logic); using `abortEarly: false` to collect all errors; using `stripUnknown: true` to prevent mass assignment; reusing and composing schemas rather than duplicating them; using `.external()` and `validateAsync()` for asynchronous checks; leveraging `.messages()` for user-friendly errors; and avoiding validation context leakage when reusing schema instances.

**Beginner-Friendly Explanation**
Best practices are the habits that make Joi reliable. Validate everything at the door, collect all errors, strip extra fields, reuse schemas, and use async validation for database checks.

---

### Purposes

- To ensure comprehensive validation coverage across all input sources.
- To prevent security issues such as mass assignment and injection.
- To maximize schema reuse and maintainability.
- To provide consistent, user-friendly error responses.

---

### Syntax Rules and Structure

#### Complete General Syntax: Express Middleware Best Practice

```javascript
function validate(schema, source = 'body') {
  return (req, res, next) => {
    const { error, value } = schema.validate(req[source], {
      abortEarly: false,
      stripUnknown: true
    });

    if (error) {
      return res.status(400).json({
        errors: error.details.map((d) => ({
          field: d.path.join('.'),
          message: d.message
        }))
      });
    }

    req[source] = value; // replace with validated, coerced data
    next();
  };
}
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Boundary validation | Middleware before route handler |
| Collect all errors | `abortEarly: false` |
| Prevent mass assignment | `stripUnknown: true` |
| Use validated value | `req[source] = value` |

#### Syntax Rules

- Validate **all** relevant sources (`body`, `query`, `params`) with the appropriate schema.
- Always set `abortEarly: false` for comprehensive error reporting.
- Use `stripUnknown: true` to remove unrecognized fields.
- Replace the raw request property with the validated, coerced `value`.
- Reuse schemas via `concat()` or `fork()` rather than duplicating rules.
- Use `validateAsync()` when schemas contain `.external()` validators.

#### Constraints and Limitations

- `stripUnknown` may remove legitimate fields if the schema is incomplete; audit schemas carefully.
- Reusing schema instances can carry validation context between requests; create fresh clones when necessary.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Express Validation Pipeline

```javascript
const express = require('express');
const Joi = require('joi');
const app = express();
app.use(express.json());

const createUserSchema = Joi.object({
  email: Joi.string().email().required().messages({
    'string.email': 'Email must be valid',
    'any.required': 'Email is required'
  }),
  password: Joi.string().min(8).required().messages({
    'string.min': 'Password must be at least 8 characters'
  }),
  username: Joi.string().alphanum().min(3).max(30).required()
});

function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true
    });

    if (error) {
      const errors = error.details.map((d) => ({
        field: d.path.join('.'),
        message: d.message
      }));
      return res.status(400).json({ errors });
    }

    req.body = value;
    next();
  };
}

app.post('/users', validate(createUserSchema), (req, res) => {
  res.status(201).json({ user: req.body });
});

app.listen(3000);
```

**Expected Output**

```
POST /users  { "email": "alice@example.com", "password": "secret123", "username": "alice" }
→ 201 { "user": { "email": "alice@example.com", "password": "secret123", "username": "alice" } }

POST /users  { "email": "invalid", "password": "123", "username": "a!" }
→ 400 { "errors": [
  { "field": "email", "message": "Email must be valid" },
  { "field": "password", "message": "Password must be at least 8 characters" },
  { "field": "username", "message": "\"username\" must only contain alpha-numeric characters" }
] }
```

**Why This Output Occurs**

- The middleware validates with `abortEarly: false` and `stripUnknown: true`, collects all field errors, and replaces `req.body` with the validated value on success.

---

### Real-World Cases

- **Production APIs** — Boundary validation with reusable schemas and consistent error envelopes.
- **Multi-endpoint applications** — Base schemas extended with `fork` for different operations.
- **Security-sensitive systems** — `stripUnknown` prevents mass assignment; async validation prevents duplicate accounts.

---

## References

- Joi Official API Documentation (v17.13.8) — https://joi.dev/api/17.x.x
- Joi Introduction — https://joi.dev/
- Schema Definition System (DeepWiki) — https://deepwiki.com/hapijs/joi/2.1-schema-definition-system
- Advanced Features: Extension Mechanism (DeepWiki) — https://deepwiki.com/hapijs/joi/2.3-advanced-features
- Difference between custom() and external() (Stack Overflow) — https://stackoverflow.com/questions/76032198/what-is-the-difference-between-jois-custom-and-external-methods
- How to use Joi for validation in Node.js (CoreUI) — https://coreui.io/answers/how-to-use-joi-for-validation-in-nodejs/
- express-joi-validation (npm) — https://www.npmjs.com/package/express-joi-validation
- Validation in Express Apps Using Joi (TowardsDev) — https://towardsdev.com/validation-in-express-apps-using-joi-755b05f35a8b
- Joi external() async validation (GitHub) — https://github.com/hapijs/joi/issues/2206
- Joi fork() for required fields (Stack Overflow) — https://stackoverflow.com/questions/30302636/how-to-ignore-required-in-joi-validation
- Joi link() for recursive schemas (Stack Overflow) — https://stackoverflow.com/questions/67552819/how-to-validate-nested-object-with-the-same-pattern-but-no-limit-dept-with-joi
- Joi messages() customization (Stack Overflow) — https://stackoverflow.com/questions/48719727/node-js-joi-how-to-display-a-custom-error-messages
- Joi best practices for Express validation (Egghead) — https://cdn.egghead.io/lessons/egghead-validating-input-to-an-express-api
- Joi schema reuse and context (Stack Overflow) — https://stackoverflow.com/questions/79218305/joi-validation-context-carry-over