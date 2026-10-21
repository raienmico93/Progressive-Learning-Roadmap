# Express.js Custom Error Classes — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Custom error classes in Express.js are user-defined JavaScript classes that extend the native `Error` object to represent specific, categorized failure conditions with additional context—such as HTTP status codes, machine-readable error codes, and structured validation details—enabling precise and consistent error handling.

**Technical Definition**
A custom error class extends the built-in `Error` class and augments it with application-specific properties. The standard pattern sets `this.name` to the class name, attaches a `statusCode` for HTTP responses, optionally sets an `isOperational` flag to distinguish expected failures from programming bugs, and invokes `Error.captureStackTrace(this, this.constructor)` to preserve a clean stack trace that excludes the constructor frame. Custom error classes are typically organized in a hierarchy: a base `AppError` class carries common properties, and specific subclasses (`ValidationError`, `NotFoundError`, `UnauthorizedError`, `ForbiddenError`) set appropriate defaults for their respective HTTP status codes. In the centralized error handler, these typed errors are distinguished via `instanceof` checks or property inspection (`statusCode`, `code`), allowing the handler to return correct status codes and safe messages while masking programming errors as generic `500` responses.

**Beginner-Friendly Explanation**
Custom error classes are your own special types of errors. Instead of throwing a generic `Error` with just a message, you create a `NotFoundError` that automatically carries a `404` status, or a `ValidationError` that carries a `400` status and a list of field-level problems. This makes your error handling code cleaner: instead of checking error messages with string comparisons, you check the error type, and your central error handler knows exactly how to respond. It is like having labeled boxes for different kinds of problems instead of one big box labeled “stuff went wrong.”

---

### Key Characteristics

- **Extends the native `Error`** — Custom errors inherit `message`, `stack`, and standard error behavior, ensuring compatibility with JavaScript's `throw`/`catch` and Express's error propagation.
- **Carries structured metadata** — Properties such as `statusCode`, `code`, `details`, and `isOperational` travel with the error, enabling the central handler to make informed response decisions.
- **Enables type-based handling** — `instanceof` checks or `name` inspection allow the error handler to route different error types to different response formats or status codes.
- **Supports operational/programming distinction** — The `isOperational` flag separates expected runtime failures (invalid input, missing resources) from bugs that should be masked as `500` errors.
- **Preserves clean stack traces** — `Error.captureStackTrace(this, this.constructor)` ensures the stack trace starts at the point where the error was thrown, not inside the constructor itself.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- Understanding of JavaScript classes, inheritance, and the `Error` constructor.
- Familiarity with Express error-handling middleware and the four-argument signature `(err, req, res, next)`.
- Basic knowledge of HTTP status codes (`400`, `401`, `403`, `404`, `500`).

---

### Related Programming Areas

- **Error handling middleware** — Custom error classes are consumed by centralized error handlers.
- **Operational vs. programming errors** — The `isOperational` distinction drives response strategy.
- **Validation** — `ValidationError` subclasses carry field-level error details.
- **Logging and monitoring** — Structured error properties enable richer logs and metrics.
- **API contract design** — Machine-readable error codes support client-side error handling.

---

## Core Concept 1: Custom Errors

### Definitions

**Core Definition**
A custom error is a user-defined error class that extends `Error` to represent a specific category of failure with additional properties and behavior tailored to the application.

**Technical Definition**
Custom errors extend the native `Error` class and add properties such as `statusCode`, `code`, and `isOperational`. They are thrown in route handlers and services using `throw new CustomError(...)` and are caught by Express's error-handling middleware. The base custom error class typically sets `this.name = this.constructor.name` so that the error's name reflects its class, and calls `Error.captureStackTrace(this, this.constructor)` to produce a clean stack trace. Specific subclasses inherit from the base class and provide default messages and status codes, reducing boilerplate at throw sites.

**Beginner-Friendly Explanation**
A custom error is your own error type. Instead of throwing `new Error('not found')`, you throw `new NotFoundError()`, and that error already knows it should produce a `404` response. You define these error types once, and then your route handlers just throw the right one.

---

### Purposes

- To categorize errors by type so that the centralized handler can respond with the correct HTTP status and message.
- To carry additional context (validation details, resource identifiers) alongside the error.
- To reduce boilerplate at throw sites by providing sensible defaults in the error class.
- To enable type-safe error handling through `instanceof` checks and class inheritance.

---

### Syntax Rules and Structure

#### Complete General Syntax: Base `AppError` Class

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode || 500;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `extends Error` | Inherits standard error behavior and stack trace support |
| `super(message)` | Passes the message to the native `Error` constructor |
| `this.name` | Sets the error name to the subclass name for logging and identification |
| `this.statusCode` | HTTP status code for the response |
| `this.isOperational` | Marks the error as an expected runtime failure rather than a bug |
| `Error.captureStackTrace` | Excludes the constructor from the stack trace for cleaner debugging |

#### Complete General Syntax: Specific Subclasses

```javascript
class ValidationError extends AppError {
  constructor(message, details = []) {
    super(message || 'Validation failed', 400);
    this.details = details;
  }
}

class NotFoundError extends AppError {
  constructor(message) {
    super(message || 'Resource not found', 404);
  }
}

class UnauthorizedError extends AppError {
  constructor(message) {
    super(message || 'Unauthorized access', 401);
  }
}
```

#### Syntax Rules

- Always call `super(message)` before accessing `this`.
- Set `this.name` explicitly so that `err.name` is meaningful in logs and in `instanceof`-free checks.
- Invoke `Error.captureStackTrace(this, this.constructor)` to keep stack traces clean.
- Subclasses should provide default messages and status codes while allowing overrides.

#### Constraints and Limitations

- Removing `Error.captureStackTrace` results in stack traces that include the error constructor frame, which can obscure the true origin of the error.
- Setting `isOperational = true` on programming bugs can cause them to be exposed to clients incorrectly.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Base and Specific Custom Errors in a Route

```javascript
const express = require('express');
const app = express();

class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode || 500;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message, details = []) {
    super(message || 'Validation failed', 400);
    this.details = details;
  }
}

class NotFoundError extends AppError {
  constructor(message) {
    super(message || 'Resource not found', 404);
  }
}

app.get('/users/:id', (req, res, next) => {
  const id = Number(req.params.id);
  if (!Number.isInteger(id)) {
    throw new ValidationError('Invalid user ID', [{ field: 'id', issue: 'must be an integer' }]);
  }
  const user = findUser(id);
  if (!user) {
    throw new NotFoundError(`User with ID ${id} not found`);
  }
  res.json(user);
});

app.use((err, req, res, next) => {
  if (err instanceof ValidationError) {
    return res.status(err.statusCode).json({ error: err.message, details: err.details });
  }
  if (err instanceof NotFoundError) {
    return res.status(err.statusCode).json({ error: err.message });
  }
  res.status(err.statusCode || 500).json({ error: 'Internal Server Error' });
});

app.listen(3000);
```

**Step-by-Step Explanation**

1. **Error hierarchy** — `ValidationError` and `NotFoundError` extend `AppError`, inheriting `statusCode`, `isOperational`, and the clean stack trace behavior.
2. **Throw site** — The route handler throws the appropriate error type based on the failure condition.
3. **Centralized handler** — `instanceof` checks route each error type to the correct response format.

**Expected Output**

```
GET /users/abc → 400 { "error": "Invalid user ID", "details": [{ "field": "id", "issue": "must be an integer" }] }
GET /users/99  → 404 { "error": "User with ID 99 not found" }
```

**Why This Output Occurs**

- The thrown `ValidationError` carries a `400` status and `details` array; the `NotFoundError` carries a `404` status. The handler uses `instanceof` to distinguish them and send appropriate responses.

---

### Real-World Cases

- **Validation errors** — `ValidationError` with field-level details for form submissions.
- **Missing resources** — `NotFoundError` when database lookups return `null`.
- **Authentication failures** — `UnauthorizedError` with a `401` status for invalid credentials.

---

## Core Concept 2: Error Inheritance

### Definitions

**Core Definition**
Error inheritance is the mechanism by which custom error classes extend a base error class (or the native `Error`), inheriting its properties and behavior while adding or overriding specific characteristics.

**Technical Definition**
In JavaScript, subclassing `Error` requires careful handling because the native `Error` constructor creates an error object whose prototype chain must be correctly maintained. When extending `Error`, calling `super(message)` initializes the error, but in some JavaScript environments (particularly when transpiling to ES5), the prototype chain may not be set correctly, causing `instanceof` checks to fail. The recommended fix is to add `Object.setPrototypeOf(this, new.target.prototype)` in the constructor of the base error class. Additionally, `Error.captureStackTrace(this, this.constructor)` ensures that the stack trace starts at the throw site rather than inside the constructor. A single base `AppError` class is often recommended for all application-level errors, with subclasses providing differentiated properties and defaults.

**Beginner-Friendly Explanation**
Error inheritance is how one error class can be based on another. You create a general `AppError` with common features like a status code, and then specific errors like `NotFoundError` and `ValidationError` inherit from it. This means you write the common setup once and only add what is different for each specific error.

---

### Purposes

- To share common error properties and behavior across multiple error types without duplication.
- To enable `instanceof` checks that work correctly across the inheritance hierarchy.
- To support a single base error class that all application errors extend, simplifying handling.
- To preserve clean and accurate stack traces through proper constructor techniques.

---

### Syntax Rules and Structure

#### Complete General Syntax: Proper Error Subclassing

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode || 500;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
    // Required for instanceof to work when transpiled
    Object.setPrototypeOf(this, new.target.prototype);
  }
}

class NotFoundError extends AppError {
  constructor(message) {
    super(message || 'Not found', 404);
  }
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `super(message)` | Initializes the base `Error` with the message |
| `Object.setPrototypeOf(this, new.target.prototype)` | Ensures the prototype chain is correct for `instanceof` |
| `Error.captureStackTrace(this, this.constructor)` | Excludes the constructor from the stack trace |
| Subclass `super(message, 404)` | Inherits base behavior with a specific status code |

#### Syntax Rules

- Always call `super()` before using `this`.
- Use `Object.setPrototypeOf(this, new.target.prototype)` when targeting environments that may transpile classes to ES5.
- Use `Error.captureStackTrace(this, this.constructor)` for clean stack traces.
- Keep a single base `AppError` class for all application-level errors.

#### Constraints and Limitations

- Without `Object.setPrototypeOf`, `instanceof` checks may fail in transpiled code.
- Multiple levels of inheritance can make the hierarchy harder to reason about; keep the hierarchy shallow.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Three-Level Inheritance with `instanceof`

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
    Object.setPrototypeOf(this, new.target.prototype);
  }
}

class NotFoundError extends AppError {
  constructor(message) {
    super(message || 'Resource not found', 404);
  }
}

class UserNotFoundError extends NotFoundError {
  constructor(userId) {
    super(`User with ID ${userId} not found`);
    this.userId = userId;
  }
}

const err = new UserNotFoundError(42);

console.log(err instanceof UserNotFoundError); // true
console.log(err instanceof NotFoundError);      // true
console.log(err instanceof AppError);           // true
console.log(err instanceof Error);              // true
console.log(err.name);                          // "UserNotFoundError"
console.log(err.statusCode);                    // 404
console.log(err.userId);                        // 42
```

**Expected Output**

```
true
true
true
true
UserNotFoundError
404
42
```

**Why This Output Occurs**

- The prototype chain is correctly established from `UserNotFoundError` through `NotFoundError` and `AppError` to `Error`. The `name` reflects the most specific class, and the `statusCode` is inherited from `NotFoundError`.

---

### Real-World Cases

- **Domain-specific hierarchies** — `PaymentError` → `InsufficientFundsError`, `CardDeclinedError`.
- **Multi-level categorization** — `AuthError` → `TokenExpiredError`, `InvalidCredentialsError`.
- **Framework integration** — All application errors extend a single `AppError` base for uniform handling.

---

## Core Concept 3: Error Properties

### Definitions

**Core Definition**
Error properties are the named fields attached to a custom error object—such as `statusCode`, `code`, `details`, and `isOperational`—that carry structured information about the failure beyond the standard `message` and `stack`.

**Technical Definition**
Custom error classes augment the native `Error` object with properties that the centralized error handler and logging infrastructure can inspect. Common properties include `statusCode` (the HTTP status code), `code` (a machine-readable string such as `'NOT_FOUND'` or `'VALIDATION_ERROR'`), `details` (structured validation or context data), and `isOperational` (a boolean distinguishing expected failures from bugs). The `http-errors` package also sets both `status` and `statusCode` for compatibility, and allows arbitrary custom properties to be attached. These properties travel with the error through the middleware stack and are available in the error handler's `err` argument.

**Beginner-Friendly Explanation**
Error properties are extra pieces of information you attach to an error. Beyond the message (“User not found”), you can attach a status code (`404`), a short code (`'USER_NOT_FOUND'`), and even the ID that was searched for. Your error handler can then use these properties to build a precise response and log entry.

---

### Purposes

- To carry the HTTP status code from the throw site to the response.
- To provide a stable, machine-readable `code` that clients can use for programmatic handling.
- To attach structured details (e.g., field-level validation errors) for diagnostic and client use.
- To enable the error handler to distinguish operational failures from programming bugs via `isOperational`.

---

### Syntax Rules and Structure

#### Complete General Syntax: Error with Properties

```javascript
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode || 500;
    this.code = code;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message, details) {
    super(message || 'Validation failed', 400, 'VALIDATION_ERROR');
    this.details = details || [];
  }
}
```

**Component Breakdown**

| Property | Role |
|----------|------|
| `statusCode` | HTTP status code for the response |
| `code` | Stable, machine-readable identifier (e.g., `'NOT_FOUND'`) |
| `details` | Structured context (e.g., array of field errors) |
| `isOperational` | Distinguishes expected failures from bugs |
| `name` | The error class name, set from `this.constructor.name` |

#### Syntax Rules

- `statusCode` should be a valid HTTP status code (e.g., `400`, `404`, `500`).
- `code` should be a stable string that clients can rely on; use uppercase snake case by convention.
- `details` can hold any structured data relevant to the error (arrays, objects).
- `isOperational` should default to `true` for expected errors and be `false` for programming bugs.

#### Constraints and Limitations

- `code` is not a standard property of the native `Error` object in all environments; setting it explicitly ensures consistency.
- Overloading properties with sensitive data can leak information if the error is exposed.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Error with `code` and `details`

```javascript
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message, details = []) {
    super(message || 'Validation failed', 400, 'VALIDATION_ERROR');
    this.details = details;
  }
}

app.post('/users', (req, res, next) => {
  const { email, name } = req.body;
  const errors = [];
  if (!email || !email.includes('@')) {
    errors.push({ field: 'email', message: 'Valid email required' });
  }
  if (!name || name.length < 2) {
    errors.push({ field: 'name', message: 'Name must be at least 2 characters' });
  }
  if (errors.length > 0) {
    throw new ValidationError('User validation failed', errors);
  }
  res.status(201).json({ created: true });
});

app.use((err, req, res, next) => {
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      error: err.message,
      code: err.code,
      ...(err.details && { details: err.details })
    });
  }
  res.status(500).json({ error: 'Internal Server Error', code: 'INTERNAL_ERROR' });
});
```

**Expected Output**

```
POST /users  { "email": "invalid", "name": "A" }
→ 400 {
  "error": "User validation failed",
  "code": "VALIDATION_ERROR",
  "details": [
    { "field": "email", "message": "Valid email required" },
    { "field": "name", "message": "Name must be at least 2 characters" }
  ]
}
```

**Why This Output Occurs**

- The `ValidationError` carries `statusCode: 400`, `code: 'VALIDATION_ERROR'`, and the `details` array. The handler includes all structured properties in the response because `isOperational` is true.

---

### Real-World Cases

- **API error contracts** — Clients switch on `code` to display localized error messages.
- **Form validation** — `details` array maps field names to specific validation failures.
- **Observability** — Logging `code` and `statusCode` enables aggregation and alerting by error type.

---

## Core Concept 4: Error Codes

### Definitions

**Core Definition**
Error codes are stable, machine-readable string or numeric identifiers attached to errors that allow clients and internal systems to identify the specific failure without parsing human-readable messages.

**Technical Definition**
The `code` property on an error carries a stable identifier (e.g., `'NOT_FOUND'`, `'VALIDATION_ERROR'`, `'INSUFFICIENT_FUNDS'`). Node.js documentation recommends using `error.code` to identify errors because `error.message` may change between versions. In custom error classes, the code is typically set in the constructor and inherited by subclasses, or overridden for more specific identification. Codes enable programmatic handling on the client side and structured logging on the server side. The `@fastify/create-error` package and similar tools support creating custom errors with codes for this purpose.

**Beginner-Friendly Explanation**
An error code is a short label for an error, like `'USER_NOT_FOUND'` or `'INVALID_EMAIL'`. Instead of relying on the error message—which might change or be translated—clients can check the code to know exactly what went wrong and how to respond.

---

### Purposes

- To provide a stable identifier for errors that does not change even if messages are updated.
- To enable client-side code to handle specific errors programmatically.
- To support structured logging and alerting by error category.
- To decouple error identification from human-readable messages.

---

### Syntax Rules and Structure

#### Complete General Syntax: Setting Error Codes

```javascript
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, 404, `${resource.toUpperCase()}_NOT_FOUND`);
  }
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `code` | Stable identifier (e.g., `'USER_NOT_FOUND'`) |
| Constructor default | Subclasses can derive codes from parameters |
| Inheritance | Subclasses inherit or override the base code |

#### Syntax Rules

- Use a stable naming convention, such as `UPPER_SNAKE_CASE`.
- Keep codes short but descriptive.
- Ensure codes are unique across the application.
- Document the codes available to API clients.

#### Constraints and Limitations

- `code` is not a standard property on all `Error` instances; setting it explicitly is required.
- Changing codes is a breaking change for clients; treat them as part of the API contract.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Domain-Specific Error Codes

```javascript
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

class PaymentError extends AppError {
  constructor(message, code) {
    super(message, 402, code);
  }
}

app.post('/pay', (req, res, next) => {
  const { amount, card } = req.body;
  if (!card || card.expired) {
    throw new PaymentError('Card has expired', 'CARD_EXPIRED');
  }
  if (amount > 10000) {
    throw new PaymentError('Amount exceeds limit', 'AMOUNT_EXCEEDS_LIMIT');
  }
  res.json({ paid: true });
});

app.use((err, req, res, next) => {
  res.status(err.statusCode || 500).json({
    error: err.message,
    code: err.code || 'INTERNAL_ERROR'
  });
});
```

**Expected Output**

```
POST /pay  { "amount": 500, "card": { "expired": true } }
→ 402 { "error": "Card has expired", "code": "CARD_EXPIRED" }

POST /pay  { "amount": 20000, "card": { "expired": false } }
→ 402 { "error": "Amount exceeds limit", "code": "AMOUNT_EXCEEDS_LIMIT" }
```

**Why This Output Occurs**

- The `PaymentError` carries the specific `code` for each failure condition, allowing the client to distinguish between an expired card and an amount limit.

---

### Real-World Cases

- **Payment processing** — Codes like `CARD_DECLINED`, `INSUFFICIENT_FUNDS`, `CARD_EXPIRED`.
- **Authentication** — Codes like `TOKEN_EXPIRED`, `INVALID_CREDENTIALS`, `MFA_REQUIRED`.
- **Resource operations** — Codes like `RESOURCE_NOT_FOUND`, `DUPLICATE_RESOURCE`.

---

## Core Concept 5: Error Messages

### Definitions

**Core Definition**
Error messages are the human-readable descriptions of errors, stored in the `message` property, that explain what went wrong in a way that is meaningful to developers or end users.

**Technical Definition**
The `message` property is set by calling `super(message)` in the custom error constructor. Messages should be clear, concise, and, for operational errors, safe to expose to clients. Programming errors should have detailed messages for server-side logging, but the centralized error handler should replace them with generic messages (`'Internal Server Error'`) before sending to clients. Node.js documentation notes that `error.message` may change between versions, so `error.code` should be used for programmatic identification. Messages should not contain sensitive information such as passwords, tokens, or internal paths.

**Beginner-Friendly Explanation**
The error message is the sentence that explains what went wrong. For errors that users will see, it should be helpful and safe—like “User not found” or “Invalid email address.” For bugs, the message can be detailed for developers, but the user should only see a generic “Something went wrong” message.

---

### Purposes

- To communicate the nature of the failure to developers for debugging and to clients for user feedback.
- To provide context that complements the machine-readable `code`.
- To support logging and alerting with descriptive error summaries.
- To avoid leaking sensitive information while still being useful.

---

### Syntax Rules and Structure

#### Complete General Syntax: Constructing Messages

```javascript
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message); // sets this.message
    // ...
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`${resource} with ID ${id} not found`, 404, `${resource.toUpperCase()}_NOT_FOUND`);
  }
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `super(message)` | Sets the native `message` property |
| Template literals | Build dynamic messages with identifiers |
| Default messages | Subclasses provide sensible defaults |

#### Syntax Rules

- Messages for operational errors should be safe to expose to clients.
- Messages for programming errors should be detailed for logs but masked before sending to clients.
- Do not include sensitive data (passwords, tokens, internal paths) in messages.
- Use the `code` property for programmatic identification, not the message.

#### Constraints and Limitations

- Messages may change between versions; they are not a stable API contract.
- Overly verbose messages can leak implementation details.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Safe Operational Messages vs. Masked Programming Errors

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

app.get('/safe', (req, res, next) => {
  throw new AppError('User not found', 404);
});

app.get('/bug', (req, res, next) => {
  throw new Error('Cannot read properties of undefined (reading "name")');
});

app.use((err, req, res, next) => {
  const status = err.isOperational ? err.statusCode : 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';

  console.error(`[${status}] ${err.message}`); // full message logged
  res.status(status).json({ error: message });
});
```

**Expected Output**

```
GET /safe → 404 { "error": "User not found" }
Console: [404] User not found

GET /bug  → 500 { "error": "Internal Server Error" }
Console: [500] Cannot read properties of undefined (reading "name")
```

**Why This Output Occurs**

- The operational error's message is safe and sent to the client; the programming error's message is logged server-side but replaced with a generic message in the response.

---

### Real-World Cases

- **User-facing validation** — Clear messages like “Email is required” for form errors.
- **Security-sensitive systems** — Generic messages for authentication failures to prevent enumeration.
- **Developer debugging** — Detailed messages in logs for programming errors.

---

## Core Concept 6: Custom Error Best Practices

### Definitions

**Core Definition**
Custom error best practices are the design and implementation patterns that keep error class hierarchies secure, consistent, maintainable, and effective for both debugging and client communication.

**Technical Definition**
Best practices include: extending the native `Error` object only once with a base `AppError` class and passing differentiating data as properties; setting `this.name` and using `Error.captureStackTrace` for clean stack traces; using `Object.setPrototypeOf(this, new.target.prototype)` to ensure `instanceof` works correctly; distinguishing operational errors from programming errors with `isOperational`; attaching stable `code` values for programmatic identification; keeping the error hierarchy shallow and well-documented; and centralizing error handling in a single four-argument middleware that inspects the error type and properties.

**Beginner-Friendly Explanation**
Best practices are the habits that keep your custom errors useful and safe. Create one base error class with common features. Use proper inheritance so `instanceof` works. Give each error a stable code and a safe message. And let one central handler decide how to respond based on the error type.

---

### Purposes

- To maintain a consistent, predictable error contract across the application.
- To prevent security vulnerabilities from leaked messages or incorrectly classified errors.
- To enable reliable `instanceof` checks and type-based handling.
- To keep the error hierarchy maintainable as the application grows.

---

### Syntax Rules and Structure

#### Complete General Syntax: Best-Practice Custom Error Setup

```javascript
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
    Object.setPrototypeOf(this, new.target.prototype);
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`${resource} with ID ${id} not found`, 404, `${resource.toUpperCase()}_NOT_FOUND`);
  }
}

app.use((err, req, res, next) => {
  const status = err.isOperational ? err.statusCode : 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';
  const code = err.isOperational ? err.code : 'INTERNAL_ERROR';

  console.error({ status, code, message: err.message, stack: err.stack });
  res.status(status).json({ error: message, code });
});
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Single base class | `AppError` with shared properties |
| Clean stack trace | `Error.captureStackTrace` |
| Correct `instanceof` | `Object.setPrototypeOf` |
| Stable codes | `code` property in constructor |
| Centralized handler | Four-argument middleware last |

#### Syntax Rules

- Extend `Error` only once with a base `AppError`; use properties for differentiation.
- Always set `this.name` and call `Error.captureStackTrace`.
- Use `Object.setPrototypeOf(this, new.target.prototype)` for reliable inheritance.
- Set `isOperational` to `true` for expected failures and `false` for bugs.
- Register the centralized error handler last with four arguments.

#### Constraints and Limitations

- Deep inheritance hierarchies become difficult to reason about; keep them shallow.
- Over-classifying bugs as operational errors can leak internal details to clients.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Best-Practice Implementation

```javascript
const express = require('express');
const app = express();
app.use(express.json());

class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
    Object.setPrototypeOf(this, new.target.prototype);
  }
}

class ValidationError extends AppError {
  constructor(message, details = []) {
    super(message || 'Validation failed', 400, 'VALIDATION_ERROR');
    this.details = details;
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`${resource} with ID ${id} not found`, 404, `${resource.toUpperCase()}_NOT_FOUND`);
  }
}

app.post('/users', (req, res, next) => {
  const { email } = req.body;
  if (!email) {
    throw new ValidationError('Email is required', [{ field: 'email', issue: 'required' }]);
  }
  res.status(201).json({ created: true });
});

app.get('/users/:id', (req, res, next) => {
  const user = null; // simulate not found
  if (!user) {
    throw new NotFoundError('User', req.params.id);
  }
  res.json(user);
});

app.use((err, req, res, next) => {
  const status = err.isOperational ? err.statusCode : 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';
  const code = err.isOperational ? err.code : 'INTERNAL_ERROR';
  const details = err.isOperational ? err.details : undefined;

  console.error({ status, code, message: err.message, stack: err.stack });

  res.status(status).json({ error: message, code, ...(details && { details }) });
});

app.listen(3000);
```

**Expected Output**

```
POST /users  { "email": "" }
→ 400 {
  "error": "Email is required",
  "code": "VALIDATION_ERROR",
  "details": [{ "field": "email", "issue": "required" }]
}

GET /users/42
→ 404 {
  "error": "User with ID 42 not found",
  "code": "USER_NOT_FOUND"
}
```

**Why This Output Occurs**

- The `ValidationError` and `NotFoundError` carry appropriate status codes, codes, and details. The centralized handler uses `isOperational` to decide whether to expose the message and details, and masks programming errors with a generic `500` and `INTERNAL_ERROR` code.

---

### Real-World Cases

- **Production APIs** — Consistent error contracts with stable codes and safe messages.
- **Multi-team codebases** — A shared base `AppError` ensures uniform handling across services.
- **Security audits** — Verification that programming errors are masked and no sensitive data leaks.

---

## References

- Express Custom Errors (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/9-express-error-handling/4-express-custom-errors.mdx
- Typed-Error Middleware with Express (Steve Kinney) — https://stevekinney.com/courses/full-stack-typescript/typed-error-middleware-with-express
- How to handle errors in Express (CoreUI) — https://coreui.io/answers/how-to-handle-errors-in-express/
- Express v5 Global Error Handling Boilerplate (GitHub) — https://github.com/Mohamed-Asem/express-v5-boilerplate
- JavaScript Error Constructors: Best Practices and Pitfalls (serpuniversity) — https://raw.githubusercontent.com/serpuniversity/learn-docs/refs/heads/main/javascript/JavaScript%20Constructor%20cant%20be%20used%20directly.md
- Node.js v26.0.0 Documentation: Errors (use `error.code` instead of `error.message`) — https://nodejs.com.tw
- Node.js Best Practices: Use only the built-in Error object (goldbergyoni) — https://raw.githubusercontent.com/goldbergyoni/nodebestpractices/master/sections/errorhandling/useonlythebuiltinerror.md
- `http-errors` README (status and statusCode properties) — https://github.com/mbtools/http-errors
- Error Handling in Node.js (mcollina/skills) — https://github.com/mcollina/skills/blob/main/skills/node/rules/error-handling.md
- Operational vs Programmer Errors (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/9-express-error-handling/0-express-error-basics.mdx
- AI-Skills: Error Handling Express (operational vs programming distinction) — https://github.com/MarcoNasi/AI-Skills/blob/main/backend/Error_Handling_Express.md