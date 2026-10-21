# express-validator — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`express-validator` is a set of Express.js middlewares that wraps validator.js, providing declarative validation and sanitization chains for request bodies, parameters, queries, cookies, and headers.

**Technical Definition**
`express-validator` provides chain creator functions — `body()`, `query()`, `param()`, `cookie()`, `header()`, and the generic `check()` — that return `ValidationChain` objects. Each chain contains all built-in validators, sanitizers, and utility modifiers from validator.js, plus the ability to define custom validators and sanitizers via `.custom()` and `.customSanitizer()`. Chains are middleware and are passed to route handlers. Validation results are collected with `validationResult(req)`, and validated/sanitized data can be retrieved with `matchedData()`. Sanitizers mutate the request, updating the field value in place (e.g., `body('message').trim()` updates `req.body.message`). Because validator.js operates on strings, standard validators and sanitizers convert non-string values to strings first.

**Beginner-Friendly Explanation**
`express-validator` is a toolkit that makes checking incoming request data easy and reliable. Instead of writing dozens of `if` statements, you attach a chain of rules directly to a field — like `body('email').isEmail().trim()` — and the library handles the rest. It can validate the data, clean it up (trim whitespace, normalize emails), and collect any errors into a structured list you can send back to the client. It is designed to plug naturally into Express routes as middleware.

---

### Key Characteristics

- **Express-native middleware** — Validation chains are middleware; they run before your route handler and integrate seamlessly with Express's `app.use()` and route methods.
- **Chain-based API** — Each method on a chain returns the chain itself, enabling fluent, readable rule composition.
- **Location-aware** — `body()`, `query()`, `param()`, `cookie()`, and `header()` target specific request locations; `check()` searches across all locations.
- **Validation and sanitization combined** — The same chain can reject invalid data and transform valid data (trimming, normalizing, casting).
- **Customizable** — Custom validators and sanitizers handle application-specific rules such as database uniqueness checks and cross-field comparisons.
- **Structured results** — `validationResult()` provides `isEmpty()`, `array()`, `mapped()`, `formatWith()`, and `throw()` for flexible error handling.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- The `express-validator` package installed via npm.
- Understanding of Express middleware and the `(req, res, next)` signature.
- Familiarity with `req.body`, `req.query`, `req.params`, `req.cookies`, and `req.headers`.

---

### Related Programming Areas

- **Validation** — `express-validator` is one of several validation libraries (Joi, Zod, Yup, Ajv, class-validator).
- **Sanitization** — Cleaning and transforming accepted data.
- **Error handling** — Validation failures produce structured `400` responses.
- **Security** — Preventing injection, XSS, and malformed input.
- **API design** — Validation chains define the input contract for routes.

---

## Core Concept 1: `express-validator`

### Definitions

**Core Definition**
`express-validator` is the library itself — a collection of Express middlewares for validating and sanitizing HTTP requests.

**Technical Definition**
`express-validator` exposes chain creators (`body`, `query`, `param`, `check`, etc.), result utilities (`validationResult`, `matchedData`), and schema validation via `checkSchema()`. It depends on validator.js for its standard validators and sanitizers. All standard validators and sanitizers are available on every chain. The library is framework-agnostic in its core but is designed specifically for Express middleware integration.

**Beginner-Friendly Explanation**
`express-validator` is the package you install and import. It gives you the tools (`body`, `query`, `param`, `check`) to build validation rules and the utilities (`validationResult`, `matchedData`) to read the results.

---

### Purposes

- To provide a complete validation and sanitization solution for Express applications.
- To integrate validation into the middleware pipeline without leaving Express's architecture.
- To leverage the extensive validator.js library of built-in validators and sanitizers.
- To produce structured validation errors and validated data for downstream handlers.

---

### Syntax Rules and Structure

#### Complete General Syntax: Basic Import and Usage

```javascript
const { body, query, param, validationResult, matchedData } = require('express-validator');

app.post('/user',
  body('email').isEmail(),
  body('password').isLength({ min: 8 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    const data = matchedData(req);
    res.json({ user: data });
  }
);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `body`, `query`, `param` | Chain creators targeting specific request locations |
| `validationResult` | Collects validation errors from the request |
| `matchedData` | Extracts validated/sanitized data |

#### Syntax Rules

- Chains are middleware; they must be passed to the route handler.
- `validationResult(req)` must be called in a subsequent middleware or handler to check for errors.
- `matchedData(req)` should be called only after confirming that validation passed.

#### Constraints and Limitations

- The library does not automatically reject requests; you must check `validationResult` and respond with `400` yourself.
- Standard validators operate on strings; non-string values are converted first.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Minimal Validation Setup

```javascript
const express = require('express');
const { body, validationResult } = require('express-validator');
const app = express();
app.use(express.json());

app.post('/user',
  body('email').isEmail().withMessage('Valid email required'),
  body('password').isLength({ min: 8 }).withMessage('Password must be at least 8 characters'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    res.status(201).json({ message: 'User created' });
  }
);

app.listen(3000);
```

**Step-by-Step Explanation**

1. **Chains registered** — `body('email')` and `body('password')` are middleware chains.
2. **Validation runs** — When a request arrives, the chains execute before the handler.
3. **Result checked** — `validationResult(req)` collects any errors; if errors exist, a `400` is returned.

**Expected Output**

```
POST /user  { "email": "alice@example.com", "password": "secret123" }
→ 201 { "message": "User created" }

POST /user  { "email": "invalid", "password": "123" }
→ 400 { "errors": [
  { "msg": "Valid email required", "path": "email" },
  { "msg": "Password must be at least 8 characters", "path": "password" }
] }
```

**Why This Output Occurs**

- The chains validate the body; `validationResult` collects the errors, and the handler returns them as a `400` response.

---

### Real-World Cases

- **REST APIs** — Validating request bodies, query parameters, and route parameters across all endpoints.
- **Form handling** — Validating and sanitizing login and registration forms.
- **Webhook endpoints** — Ensuring incoming payloads conform to expected schemas.

---

## Core Concept 2: `body()`

### Definitions

**Core Definition**
`body(fields?, message?)` is a chain creator that validates and sanitizes fields exclusively in `req.body`.

**Technical Definition**
`body()` is equivalent to `check()` but restricted to `req.body`. It accepts one or more field names (strings or arrays) and an optional default error message. If no fields are specified, it selects the entire body. The chain supports all standard validators, sanitizers, and modifiers. When a sanitizer is applied, the sanitized value is written back to `req.body`.

**Beginner-Friendly Explanation**
`body()` targets the request body — the data sent in a POST, PUT, or PATCH request. Use it for form fields and JSON payloads.

---

### Purposes

- To validate and sanitize fields in `req.body` specifically.
- To avoid validating fields in other locations (query, params, headers) unintentionally.
- To apply body-specific sanitization that updates the request body in place.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
body(fields?: string | string[], message?: any): ValidationChain
```

**Component Breakdown**

| Parameter | Role |
|-----------|------|
| `fields` | Field name(s) to validate; omitted means the whole body |
| `message` | Default error message for the chain |

#### Syntax Rules

- `body()` only reads from `req.body`.
- Sanitizers applied via `body()` mutate `req.body`.
- Multiple fields can be passed as an array: `body(['email', 'username'])`.

#### Constraints and Limitations

- `body()` does not validate query parameters, route parameters, cookies, or headers.
- Non-string values are converted to strings by standard validators.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Body Validation and Sanitization

```javascript
app.post('/comment',
  body('email').isEmail().normalizeEmail(),
  body('text').not().isEmpty().trim().escape(),
  body('notifyOnReply').toBoolean(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.json({ body: req.body }); // sanitized values
  }
);
```

**Expected Output**

```
POST /comment  { "email": " Alice@Example.com ", "text": "  Hello world :>)  ", "notifyOnReply": "true" }
→ 200 {
  "body": {
    "email": "alice@example.com",
    "text": "Hello world :&gt;)",
    "notifyOnReply": true
  }
}
```

**Why This Output Occurs**

- `normalizeEmail()` lowercases and trims the email; `trim()` removes whitespace from `text`; `escape()` converts `<`, `>`, `&` to HTML entities; `toBoolean()` converts the string `"true"` to boolean `true`.

---

### Real-World Cases

- **User registration** — Validating email, password, and username in the request body.
- **Comment systems** — Sanitizing text content before storage.
- **Settings updates** — Converting string booleans to real booleans.

---

## Core Concept 3: `query()`

### Definitions

**Core Definition**
`query(fields?, message?)` is a chain creator that validates and sanitizes fields exclusively in `req.query` — the query string parameters of the request URL.

**Technical Definition**
`query()` is equivalent to `check()` but restricted to `req.query`. In Express 5, `req.query` is a getter and is immutable; therefore, sanitizers applied via `query()` do not mutate `req.query`. Instead, the recommended approach is to use `matchedData()` to retrieve the sanitized values. Query parameters are always strings (or arrays of strings), so type coercion (`toInt()`, `toBoolean()`) is commonly needed.

**Beginner-Friendly Explanation**
`query()` targets the query string — the part of the URL after `?`. Use it for pagination parameters, search terms, and filters. Since Express 5 makes `req.query` immutable, use `matchedData()` to get the sanitized values.

---

### Purposes

- To validate and sanitize query string parameters such as `page`, `limit`, and `sort`.
- To coerce string query parameters into numbers or booleans.
- To enforce allowed values and formats for filters.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
query(fields?: string | string[], message?: any): ValidationChain
```

#### Syntax Rules

- `query()` only reads from `req.query`.
- In Express 5, sanitizers do **not** mutate `req.query`; retrieve sanitized values with `matchedData()`.
- Use `.optional()` for query parameters that may be omitted.

#### Constraints and Limitations

- Query parameters are strings; numeric validation requires `.toInt()` or `.toFloat()`.
- Express 5's immutable `req.query` means direct mutation is not possible.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Pagination Query Validation

```javascript
app.get('/search',
  query('page').optional().isInt({ min: 1 }).toInt(),
  query('limit').optional().isInt({ min: 1, max: 100 }).toInt(),
  query('q').optional().isString().trim(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    const data = matchedData(req, { locations: ['query'] });
    res.json(data);
  }
);
```

**Expected Output**

```
GET /search?page=2&limit=20&q=node
→ 200 { "page": 2, "limit": 20, "q": "node" }

GET /search?page=abc
→ 400 { "errors": [{ "path": "page", "msg": "Invalid value" }] }
```

**Why This Output Occurs**

- `toInt()` converts the string query values to numbers; `matchedData` retrieves the sanitized values because `req.query` is immutable in Express 5.

---

### Real-World Cases

- **Search endpoints** — Validating search terms and filter parameters.
- **Pagination** — Ensuring `page` and `limit` are positive integers within bounds.
- **Sorting** — Restricting sort fields to allowed values.

---

## Core Concept 4: `param()`

### Definitions

**Core Definition**
`param(fields?, message?)` is a chain creator that validates and sanitizes route parameters exclusively in `req.params`.

**Technical Definition**
`param()` is equivalent to `check()` but restricted to `req.params`. Route parameters are always strings extracted from the URL path, so numeric validation and conversion are common. Like `query()`, sanitizers applied via `param()` may not mutate `req.params` in all Express versions; `matchedData()` should be used to retrieve sanitized values.

**Beginner-Friendly Explanation**
`param()` targets the dynamic segments of the URL path, like `:id` in `/users/:id`. Use it to validate and convert route parameters.

---

### Purposes

- To validate route parameters such as numeric IDs or slugs.
- To coerce string route parameters into integers for database queries.
- To reject malformed route parameters before they reach handlers.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
param(fields?: string | string[], message?: any): ValidationChain
```

#### Syntax Rules

- `param()` only reads from `req.params`.
- Use `.isInt()` and `.toInt()` for numeric IDs.
- Use `matchedData()` to retrieve sanitized parameter values.

#### Constraints and Limitations

- Route parameters are always strings.
- Sanitizers may not persist in `req.params`; `matchedData()` is the reliable retrieval method.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Numeric ID Validation

```javascript
app.get('/users/:id',
  param('id').isInt({ min: 1 }).withMessage('ID must be a positive integer').toInt(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    const data = matchedData(req);
    res.json({ userId: data.id });
  }
);
```

**Expected Output**

```
GET /users/42  → 200 { "userId": 42 }
GET /users/abc → 400 { "errors": [{ "path": "id", "msg": "ID must be a positive integer" }] }
```

**Why This Output Occurs**

- `isInt()` rejects non-numeric IDs; `toInt()` converts valid string IDs to numbers; `matchedData` retrieves the converted value.

---

### Real-World Cases

- **REST resource identifiers** — Validating `:id` parameters across CRUD endpoints.
- **Slug validation** — Ensuring URL slugs meet format requirements.
- **UUID routes** — Validating UUID-formatted route parameters.

---

## Core Concept 5: `check()`

### Definitions

**Core Definition**
`check(fields?, message?)` is the generic chain creator that searches for fields across **all** request locations — `req.body`, `req.cookies`, `req.headers`, `req.query`, and `req.params`.

**Technical Definition**
`check()` is the main API for validating and sanitizing HTTP requests with express-validator. It gives access to all built-in validators and sanitizers. When a field is present in more than one location, all instances are processed by the chain. If no fields are specified, the whole request location is selected. `body()`, `cookie()`, `header()`, `param()`, and `query()` are convenience wrappers around `check()` restricted to a single location.

**Beginner-Friendly Explanation**
`check()` is the all-purpose validator. If you do not know or care where the data lives — body, query, params, headers, or cookies — `check()` will find it. The location-specific functions (`body()`, `query()`, etc.) are just `check()` with a built-in location filter.

---

### Purposes

- To validate a field regardless of which request location it appears in.
- To apply the same validation rules across multiple locations.
- To provide a single, generic entry point for validation chains.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
check(fields?: string | string[], message?: any): ValidationChain
```

#### Syntax Rules

- `check()` searches all five request locations.
- If a field exists in multiple locations, all instances are validated.
- Location-specific wrappers (`body()`, etc.) are preferred when the location is known, for clarity.

#### Constraints and Limitations

- `check()` may produce unexpected matches if the same field name exists in multiple locations with different meanings.
- For precise control, use location-specific chain creators.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Cross-Location Validation

```javascript
app.post('/secure',
  check('apiKey').exists().withMessage('API key required'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.json({ ok: true });
  }
);
```

**Expected Output**

```
POST /secure  (apiKey in query string)
→ 200 { "ok": true }

POST /secure  (no apiKey anywhere)
→ 400 { "errors": [{ "msg": "API key required", "path": "apiKey" }] }
```

**Why This Output Occurs**

- `check('apiKey')` finds the field in any location; `exists()` ensures it is present somewhere.

---

### Real-World Cases

- **API key authentication** — Accepting the key from headers, query, or body.
- **Multi-source tokens** — Validating tokens that may appear in cookies or headers.
- **Flexible form handling** — Validating fields regardless of encoding.

---

## Core Concept 6: Validation Chains

### Definitions

**Core Definition**
A validation chain is a sequence of validators, sanitizers, and modifiers applied to one or more request fields, created by `body()`, `query()`, `param()`, `check()`, or other chain creators.

**Technical Definition**
Validation chains are middleware functions created by chain creator functions. Each method on the chain returns the chain itself, enabling method chaining. A chain has three kinds of methods: **validators** (determine if the value is valid), **sanitizers** (transform the value), and **modifiers** (control how the chain behaves, such as `.optional()`, `.withMessage()`, `.bail()`, and `.if()`). Chains are executed in the order they are registered. All validators and sanitizers from validator.js are available as standard methods.

**Beginner-Friendly Explanation**
A validation chain is a list of rules attached to a field. You start with `body('email')` and then chain rules: `.isEmail().trim().normalizeEmail()`. Each method adds another rule or transformation. The chain itself is middleware — you hand it to Express, and it runs before your handler.

---

### Purposes

- To compose multiple validation and sanitization rules into a single, readable middleware.
- To control the execution order of validators and sanitizers.
- To apply modifiers such as optionality, custom messages, and early bail-out.
- To keep validation logic declarative and close to the route definition.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
body('email')
  .optional()
  .trim()
  .isEmail().withMessage('Valid email required')
  .normalizeEmail()
  .bail();
```

**Component Breakdown**

| Method Type | Examples |
|-------------|----------|
| Validator | `.isEmail()`, `.isLength()`, `.isInt()`, `.exists()` |
| Sanitizer | `.trim()`, `.normalizeEmail()`, `.escape()`, `.toInt()` |
| Modifier | `.optional()`, `.withMessage()`, `.bail()`, `.if()` |

#### Syntax Rules

- Each method returns the chain, enabling fluent chaining.
- Validators record errors; sanitizers mutate the value; modifiers change chain behavior.
- `.bail()` stops validation of the current chain after the first error.
- `.optional()` skips validation when the field is `undefined`, `null`, or an empty string.

#### Constraints and Limitations

- Standard validators convert non-string values to strings first.
- Sanitizers mutate the request; this can affect subsequent chains and the handler.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Chained Validation and Sanitization

```javascript
app.post('/register',
  body('email')
    .trim()
    .isEmail().withMessage('Invalid email')
    .normalizeEmail(),
  body('password')
    .isLength({ min: 8 }).withMessage('Password too short')
    .bail(),
  body('username')
    .optional()
    .isAlphanumeric().withMessage('Username must be alphanumeric'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.status(201).json({ message: 'Registered' });
  }
);
```

**Expected Output**

```
POST /register  { "email": " Alice@Example.com ", "password": "secret123", "username": "alice" }
→ 201 { "message": "Registered" }  (email trimmed and normalized)

POST /register  { "email": "invalid", "password": "123" }
→ 400 { "errors": [
  { "msg": "Invalid email", "path": "email" },
  { "msg": "Password too short", "path": "password" }
] }
```

**Why This Output Occurs**

- `.trim()` runs first, then `.isEmail()` validates the trimmed value, and `.normalizeEmail()` normalizes it. The password chain uses `.bail()` to stop after the length failure.

---

### Real-World Cases

- **Registration forms** — Multi-rule chains for email, password, and username.
- **Search filters** — Optional query parameters with type coercion.
- **API input validation** — Reusable chains for common field types.

---

## Core Concept 7: Sanitization

### Definitions

**Core Definition**
Sanitization is the process of transforming request field values into a clean, safe, or correctly typed form — such as trimming whitespace, normalizing email addresses, escaping HTML entities, or casting strings to numbers.

**Technical Definition**
Sanitizers are chain methods that mutate the field value and persist the updated value back into the request (except where the request location is immutable, as with Express 5's `req.query`). All sanitizers from validator.js are available as standard sanitizers. Sanitization is distinct from validation: validation **rejects** invalid data, while sanitization **transforms** data. The recommended pattern is to sanitize first, then validate, so that validation operates on the cleaned value.

**Beginner-Friendly Explanation**
Sanitization is cleaning up the data before you use it. Trimming extra spaces, making email addresses lowercase, removing dangerous HTML characters — these are all sanitization steps. It does not reject data; it transforms it.

---

### Purposes

- To remove noise (whitespace, inconsistent formatting) from input.
- To cast values to the correct JavaScript type (`toInt()`, `toBoolean()`, `toFloat()`).
- To normalize formats such as email addresses and URLs.
- To provide a basic line of defense against XSS by escaping HTML entities.

---

### Syntax Rules and Structure

#### Complete General Syntax: Common Sanitizers

```javascript
body('email').normalizeEmail();
body('text').trim().escape();
body('age').toInt();
body('notify').toBoolean();
body('name').blacklist('<>');
```

**Component Breakdown**

| Sanitizer | Effect |
|-----------|--------|
| `.trim()` | Removes leading/trailing whitespace |
| `.normalizeEmail()` | Lowercases and normalizes email format |
| `.escape()` | Converts `<`, `>`, `&`, `"`, `'` to HTML entities |
| `.toInt()` | Converts to integer |
| `.toBoolean()` | Converts to boolean |
| `.blacklist(chars)` | Removes specified characters |

#### Syntax Rules

- Sanitizers can be chained before or after validators.
- Sanitization mutates the request; subsequent chains and the handler see the sanitized value.
- In Express 5, `req.query` is immutable; use `matchedData()` to retrieve sanitized query values.

#### Constraints and Limitations

- Sanitization does not replace validation; it complements it.
- `.escape()` is not sufficient for all XSS contexts; context-specific encoding is required for HTML attributes, JavaScript, and CSS.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Sanitization Chain

```javascript
app.post('/profile',
  body('email').normalizeEmail(),
  body('bio').trim().escape(),
  body('age').toInt(),
  body('subscribe').toBoolean(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.json(req.body); // sanitized values
  }
);
```

**Expected Output**

```
POST /profile  {
  "email": " Alice@Example.com ",
  "bio": "  Hello <script>  ",
  "age": "30",
  "subscribe": "true"
}
→ 200 {
  "email": "alice@example.com",
  "bio": "Hello &lt;script&gt;",
  "age": 30,
  "subscribe": true
}
```

**Why This Output Occurs**

- `normalizeEmail()` cleans the email; `trim()` and `escape()` clean the bio; `toInt()` and `toBoolean()` cast the types.

---

### Real-World Cases

- **Comment systems** — Escaping HTML to prevent XSS.
- **User profiles** — Normalizing emails and trimming names.
- **API parameters** — Casting query strings to numbers and booleans.

---

## Core Concept 8: Custom Validators

### Definitions

**Core Definition**
Custom validators are user-defined functions attached to a validation chain via `.custom()` that perform application-specific validation logic beyond the built-in validator.js rules.

**Technical Definition**
`.custom(validator)` adds a custom validator function to the chain. The validator receives the field value and a context object containing `req`, `location`, `path`, and `pathValues`. The field is valid if the validator returns a truthy value or a Promise that resolves. The field is invalid if the validator returns a falsy value, rejects a Promise, or throws. Thrown errors or rejected Promises can carry custom error messages. Custom sanitizers via `.customSanitizer()` transform the value instead of validating it.

**Beginner-Friendly Explanation**
Custom validators let you write your own rules. For example, you can check if an email is already in use in your database, or if a password confirmation matches the password. You write a function, and express-validator runs it as part of the chain.

---

### Purposes

- To validate data against application-specific rules (uniqueness, cross-field matching, business constraints).
- To perform asynchronous validation such as database lookups.
- To produce custom error messages tailored to the application.

---

### Syntax Rules and Structure

#### Complete General Syntax: Custom Validator

```javascript
body('email').custom(async (value, { req, location, path }) => {
  const user = await User.findByEmail(value);
  if (user) {
    throw new Error('Email already in use');
  }
  return true;
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `.custom(fn)` | Attaches the custom validator |
| `value` | The field value |
| `{ req, location, path }` | Context object |
| `throw new Error(...)` | Signals invalidity with a custom message |

#### Syntax Rules

- Return a truthy value or a resolving Promise for valid data.
- Return a falsy value, throw, or reject a Promise for invalid data.
- Thrown values or rejection reasons become the error message.

#### Constraints and Limitations

- Custom validators run per field, not across fields; use the `req` context for cross-field checks.
- Asynchronous custom validators add latency; database calls should be efficient.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Async Custom Validator for Email Uniqueness

```javascript
app.post('/register',
  body('email').isEmail().custom(async (value) => {
    const existing = await User.findByEmail(value);
    if (existing) {
      throw new Error('This email is already registered');
    }
    return true;
  }),
  body('passwordConfirmation').custom((value, { req }) => {
    return value === req.body.password;
  }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.status(201).json({ message: 'Registered' });
  }
);
```

**Expected Output**

```
POST /register  { "email": "new@example.com", "password": "secret123", "passwordConfirmation": "secret123" }
→ 201 { "message": "Registered" }

POST /register  { "email": "taken@example.com", "password": "secret123", "passwordConfirmation": "secret123" }
→ 400 { "errors": [{ "msg": "This email is already registered", "path": "email" }] }

POST /register  { "email": "new@example.com", "password": "secret123", "passwordConfirmation": "different" }
→ 400 { "errors": [{ "msg": "Invalid value", "path": "passwordConfirmation" }] }
```

**Why This Output Occurs**

- The async custom validator checks the database and throws on existing emails. The cross-field validator compares two fields via `req`.

---

### Real-World Cases

- **Email uniqueness** — Checking whether an email is already registered.
- **Password confirmation** — Ensuring two fields match.
- **Business rules** — Validating age restrictions, date ranges, or inventory availability.

---

## Core Concept 9: Error Handling

### Definitions

**Core Definition**
Error handling in express-validator is the process of collecting validation failures via `validationResult()`, formatting them, and returning an appropriate `400 Bad Request` response to the client.

**Technical Definition**
`validationResult(req)` returns a `Result` object with methods: `isEmpty()`, `array()`, `mapped()`, `formatWith()`, and `throw()`. Errors can be formatted as an array of error objects (each with `msg`, `path`, `location`, `type`, and `value`) or as an object keyed by field name. `formatWith()` allows custom error shapes. `throw()` throws a decorated error, useful for integrating with Express error-handling middleware. Custom validators can provide custom error messages via thrown errors or `.withMessage()`.

**Beginner-Friendly Explanation**
Error handling is what you do when validation fails. express-validator gives you a result object. You check if it has errors; if it does, you format them nicely and send a `400` response to the client.

---

### Purposes

- To collect all validation failures in one place.
- To format errors consistently for API clients.
- To integrate validation failures with Express's error-handling middleware.
- To provide field-level feedback for forms.

---

### Syntax Rules and Structure

#### Complete General Syntax: Basic Error Handling

```javascript
const errors = validationResult(req);
if (!errors.isEmpty()) {
  return res.status(400).json({ errors: errors.array() });
}
```

**Component Breakdown**

| Method | Role |
|--------|------|
| `validationResult(req)` | Creates the Result object |
| `.isEmpty()` | Checks for errors |
| `.array()` | Returns errors as an array |
| `.mapped()` | Returns errors keyed by field |
| `.throw()` | Throws a decorated error |

#### Syntax Rules

- Call `validationResult(req)` in a middleware after the validation chains.
- Use `.isEmpty()` to check for errors; `.array()` or `.mapped()` to retrieve them.
- Use `.throw()` if integrating with a centralized Express error handler.

#### Constraints and Limitations

- `validationResult` must be called after all chains have run; otherwise, errors may be incomplete.
- The default error message is `"Invalid value"`, which is not descriptive; use `.withMessage()` to customize.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Formatted Error Responses

```javascript
app.post('/user',
  body('email').isEmail().withMessage('Invalid email'),
  body('password').isLength({ min: 8 }).withMessage('Password too short'),
  (req, res) => {
    const result = validationResult(req);
    if (!result.isEmpty()) {
      return res.status(400).json({
        errors: result.array({ onlyFirstError: true })
      });
    }
    res.status(201).json({ message: 'Created' });
  }
);
```

**Expected Output**

```
POST /user  { "email": "invalid", "password": "123" }
→ 400 { "errors": [
  { "msg": "Invalid email", "path": "email" },
  { "msg": "Password too short", "path": "password" }
] }
```

**Why This Output Occurs**

- The custom messages set by `.withMessage()` are used in the error array.

---

#### Example 2: Using `.throw()` with Centralized Error Handling

```javascript
app.post('/user',
  body('email').isEmail(),
  (req, res, next) => {
    try {
      validationResult(req).throw();
      res.status(201).json({ message: 'Created' });
    } catch (err) {
      next(err); // forward to Express error handler
    }
  }
);

app.use((err, req, res, next) => {
  res.status(400).json({ errors: err.array() });
});
```

**Expected Output**

```
POST /user  { "email": "invalid" }
→ 400 { "errors": [{ "msg": "Invalid value", "path": "email" }] }
```

**Why This Output Occurs**

- `.throw()` throws an error decorated with the validation result methods, which the Express error handler catches and formats.

---

### Real-World Cases

- **API error contracts** — Returning consistent validation error shapes.
- **Form feedback** — Displaying field-level errors to users.
- **Centralized error handling** — Using `.throw()` to route validation failures through the main error handler.

---

## Core Concept 10: express-validator Best Practices

### Definitions

**Core Definition**
`express-validator` best practices are the design, security, and maintainability patterns that keep validation and sanitization reliable, complete, and safe across an Express application.

**Technical Definition**
Best practices include: validating at the boundary as middleware before business logic; validating all relevant request locations (`body`, `query`, `params`, headers, cookies); sanitizing before validating where appropriate; using `matchedData()` to retrieve validated/sanitized data instead of reading raw request properties (especially in Express 5 where `req.query` is immutable); rejecting or stripping unknown fields; using `.bail()` to avoid redundant error messages; and treating validation errors as `400 Bad Request` responses with structured details. Validation should be paired with database constraints and parameterized queries for defense in depth.

**Beginner-Friendly Explanation**
Best practices are the habits that keep validation effective. Validate everything at the door, sanitize before you validate, use `matchedData()` to get clean data, and always return proper `400` responses when something is wrong.

---

### Purposes

- To ensure comprehensive coverage of all input sources.
- To prevent security vulnerabilities through consistent validation and sanitization.
- To retrieve only validated and sanitized data for use in handlers.
- To maintain consistent error contracts across the API.

---

### Syntax Rules and Structure

#### Complete General Syntax: Best-Practice Validation Pipeline

```javascript
app.post('/user',
  body('email').trim().isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }).bail(),
  body('age').optional().isInt({ min: 18 }).toInt(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    const data = matchedData(req); // only validated/sanitized fields
    res.status(201).json({ user: data });
  }
);
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Validate at boundary | Chains as middleware before handler |
| Sanitize before validate | `.trim()` before `.isEmail()` |
| Retrieve clean data | `matchedData(req)` |
| Structured errors | `validationResult(req).array()` |
| Early bail-out | `.bail()` for dependent rules |

#### Syntax Rules

- **Validate all sources** — Use `body()`, `query()`, `param()`, and `header()` as needed.
- **Sanitize before validating** — Trim and normalize before applying format validators.
- **Use `matchedData()`** — Do not read raw request properties after validation.
- **Return `400` for validation failures** — Never proceed with invalid data.
- **Use `.bail()`** — Avoid accumulating redundant errors for dependent rules.
- **Pair with database constraints** — Validation is not a substitute for unique indexes and parameterized queries.

#### Constraints and Limitations

- `matchedData()` excludes invalid data by default; use `onlyValidData: false` only if you have a specific reason.
- In Express 5, `req.query` is immutable; `matchedData()` is the recommended retrieval method.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Production-Ready Validation Pipeline

```javascript
const express = require('express');
const { body, query, param, validationResult, matchedData } = require('express-validator');
const app = express();
app.use(express.json());

app.post('/users/:id',
  [
    param('id').isInt({ min: 1 }).toInt(),
    body('email').trim().isEmail().normalizeEmail(),
    body('password').isLength({ min: 8 }).bail(),
    body('age').optional().isInt({ min: 18 }).toInt(),
    query('notify').optional().isBoolean().toBoolean()
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    const data = matchedData(req);
    res.status(201).json({ data });
  }
);

app.listen(3000);
```

**Expected Output**

```
POST /users/42?notify=true  {
  "email": " Alice@Example.com ",
  "password": "secret123",
  "age": "30"
}
→ 201 {
  "data": {
    "id": 42,
    "email": "alice@example.com",
    "password": "secret123",
    "age": 30,
    "notify": true
  }
}

POST /users/abc  { "email": "invalid", "password": "123" }
→ 400 { "errors": [
  { "path": "id", "msg": "Invalid value" },
  { "path": "email", "msg": "Invalid value" },
  { "path": "password", "msg": "Invalid value" }
] }
```

**Why This Output Occurs**

- All relevant locations are validated and sanitized; `matchedData` retrieves only validated/sanitized fields with correct types; invalid data produces a structured `400`.

---

### Real-World Cases

- **Production APIs** — Comprehensive validation pipelines with `matchedData` for safe data access.
- **Express 5 migrations** — Adopting `matchedData` to handle immutable `req.query`.
- **Security-sensitive endpoints** — Sanitization and validation paired with database constraints.

---

## References

- express-validator Official Documentation — https://express-validator.github.io/docs/
- check() API — https://express-validator.github.io/docs/api/check/
- The Validation Chain — https://express-validator.github.io/docs/guides/validation-chain/
- Customizing express-validator — https://express-validator.github.io/docs/guides/customizing/
- Custom validators/sanitizers — https://express-validator.github.io/docs/guides/custom-validators-sanitizers/
- Sanitization — https://express-validator.github.io/docs/guides/sanitization/
- Schema Validation — https://express-validator.github.io/docs/guides/schema-validation/
- validationResult API — https://express-validator.github.io/docs/api/validation-result/
- matchedData API — https://express-validator.github.io/docs/api/matched-data/
- express-validator 实战：Express 数据校验完全指南 — https://www.guance.com/learn/articles/express-validator-express
- Result Processing (DeepWiki) — https://deepwiki.com/express-validator/express-validator/3.5-result-processing
- Validation Chains (DeepWiki) — https://deepwiki.com/express-validator/express-validator/2.1-validation-chains
- Express 5 breaking changes and matchedData — https://codeberg.org/badbabykosh/curriculum/src/commit/fc7fc10c2d362e18e8c316b001189e79859ab6b5/nodeJS/express/forms_and_data_handling.md
- express-validator npm: Secure Usage Guide — https://safeguard.sh/resources/blog/express-validator-npm-secure-usage-guide
- express-validator v7+ escape sanitizer removal — https://github.com/secondsky/claude-skills/blob/main/plugins/api-security-hardening/skills/api-security-hardening/SKILL.md