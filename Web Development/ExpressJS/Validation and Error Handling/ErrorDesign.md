# Error Design — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Error design is the deliberate architectural practice of structuring how an application communicates failures to its consumers — defining machine-readable error codes, human-readable messages, structured metadata, and the mapping of error conditions to HTTP status codes — so that clients can programmatically handle failures and users can understand them.

**Technical Definition:** Error design encompasses the schemas, conventions, and classification systems used to represent errors in API responses. It includes the assignment of deterministic machine-readable codes, the separation of human-readable messages from machine-readable details, the formatting of validation errors as field-level arrays, the correct distinction between 401 Unauthorized and 403 Forbidden, and the interception of low-level database rejections into user-safe HTTP responses. RFC 9457 (Problem Details for HTTP APIs) provides a standardised foundation: a problem detail is a structured format that carries machine-readable details of errors in an HTTP response, served as `application/problem+json`, with core members `type`, `title`, `status`, `detail`, and `instance`. The design of error responses is part of the API's contract — clients parse them, retry logic branches on them, and support engineers rely on them for debugging.

**Beginner-Friendly Explanation:** Error design is like designing the signage and alarm systems in a building. When something goes wrong, you want clear, consistent signals: a red light for "you can't enter" (401), an orange light for "you entered, but you can't go there" (403), and a yellow light for "there's a conflict" (409). Each signal should come with a standard code (so machines can react automatically) and a plain-language message (so humans understand what happened). Without a consistent design, every department inventing its own alarms creates chaos.

### Key Characteristics

- **Dual audience:** Every error response serves both machines (codes, structured details) and humans (messages, hints).
- **Deterministic codes:** Machine-readable codes are stable, versioned, and documented — clients can branch on them without parsing messages.
- **Status code discipline:** HTTP status codes carry the primary semantic; the error body carries the detail.
- **Schema safety:** Database errors are intercepted and mapped to safe HTTP responses without leaking schema details.
- **Validation specificity:** Validation failures return field-level arrays, not generic messages.
- **Consistency:** Every error across every endpoint follows the same envelope structure.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Classes, objects, and asynchronous programming.
- **Understanding of HTTP status codes:** 4xx and 5xx families.
- **A custom error class hierarchy** (e.g., `AppError` extending `Error`).

### Related Programming Areas

- **Error Handling Architecture:** The propagation and centralised middleware system that delivers errors to the formatter.
- **HTTP Responses:** Status codes and response bodies.
- **Input Validation:** Validation errors are a primary source of structured error responses.
- **Database Access:** Repository-level error interception and mapping.
- **API Documentation:** Error codes and schemas are documented in OpenAPI/Swagger specifications.

### Core Concepts

1. **Error Codes** — assigning deterministic, machine-readable internal application strings.
2. **Human-Readable Messages** — writing consumer-facing error alerts safe for user interfaces.
3. **Machine-Readable Details** — structuring metadata payloads for client applications.
4. **Validation Errors** — formatting arrays of nested schema validation failures.
5. **Authentication & Authorization Errors** — explicitly returning 401 vs. 403.
6. **Database Errors** — intercepting low-level engine rejections and mapping to safe HTTP codes.

---

## Core Concept 1: Error Codes

### Definitions

**Core Definition:** Error codes are deterministic, machine-readable strings assigned to each distinct error condition, allowing client applications to programmatically identify and handle specific failures without parsing human-language messages.

**Technical Definition:** Error codes follow a structured naming convention, typically of the form `PREFIX_CATEGORY_NUMBER`, where `PREFIX` identifies the application or service (e.g., `AUTH`, `PAY`, `USR`), `CATEGORY` identifies the error class (`VAL` for validation, `SYS` for system, `BIZ` for business, `NET` for network, `AUTH` for authentication), and `NUMBER` is a unique numeric identifier. Error categories map to HTTP status codes: `VAL` → 400, `BIZ` → 422, `AUTH` (001–099) → 401, `AUTH` (100–199) → 403, `SYS` → 500, and `NET` → 502/503/504. In TypeScript, error codes are often defined as string enums for compile-time safety.

**Beginner-Friendly Explanation:** Error codes are like the numeric codes on a hospital monitor — the doctor doesn't need to read a sentence to know that "E-401" means "heart rate alert." Similarly, a client application can say "if the error code is `AUTH_VAL_001`, show the email field in red" without needing to understand the English message.

### Purposes

- To assign deterministic, machine-readable internal application strings (e.g., `ERR_INSUFFICIENT_FUNDS`, `AUTH_TOKEN_EXPIRED`).
- To enable client applications to branch on specific error types without parsing messages.
- To provide a stable identifier for error documentation and support.
- To enable consistent monitoring and alerting based on error categories.

### Syntax Rules and Structure

#### Naming Convention

```
<PREFIX>_<CATEGORY>_<NUMBER>
```

| Component | Description | Examples |
|-----------|-------------|----------|
| `PREFIX` | Application or service identifier | `AUTH`, `PAY`, `USR`, `API` |
| `CATEGORY` | Error category | `VAL`, `BIZ`, `SYS`, `NET`, `AUTH` |
| `NUMBER` | Unique numeric identifier | `001`, `100`, `404` |

#### Examples

```
AUTH_VAL_001    → Authentication validation error (email required)
PAY_SYS_503     → Payment system unavailable
USR_BIZ_100     → User business rule violation (cannot return after 30 days)
API_NET_408     → API network timeout
AUTH_AUTH_201  → Token expired
```

#### Category-to-HTTP Mapping

| Category | Full Name | HTTP Status |
|----------|-----------|-------------|
| `VAL` | Validation | 400 Bad Request |
| `BIZ` | Business | 422 Unprocessable Entity |
| `AUTH` (001–099) | Authentication | 401 Unauthorized |
| `AUTH` (100–199) | Authorization | 403 Forbidden |
| `SYS` | System | 500 Internal Server Error |
| `NET` | Network | 502/503/504 Gateway errors |

#### TypeScript Enum Definition

```typescript
export enum ErrorCode {
  NOT_FOUND = 'NOT_FOUND',
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  UNAUTHORIZED = 'UNAUTHORIZED',
  FORBIDDEN = 'FORBIDDEN',
  INTERNAL_ERROR = 'INTERNAL_ERROR',
  INSUFFICIENT_FUNDS = 'ERR_INSUFFICIENT_FUNDS',
  TOKEN_EXPIRED = 'AUTH_TOKEN_EXPIRED'
}
```

**Rules:**
- Error codes must be **stable** — changing a code breaks clients that depend on it.
- Use **uppercase snake_case** for readability.
- Categorise by error **nature**, not by business type, to avoid proliferation and duplication.
- Document every code in the API specification.

**Constraints:**
- Numeric-only error codes are less readable but avoid translation issues; semantic codes are more debuggable but require namespacing to avoid collisions.
- Avoid reusing codes for different error conditions.

### Annotated Code Example

```typescript
// errors/errorCodes.ts
export enum ErrorCode {
  // Validation errors (VAL)
  VALIDATION_ERROR = 'VAL_001',
  INVALID_EMAIL = 'VAL_100',
  PASSWORD_TOO_SHORT = 'VAL_201',

  // Business errors (BIZ)
  INSUFFICIENT_FUNDS = 'BIZ_100',
  ORDER_ALREADY_CANCELLED = 'BIZ_001',

  // Authentication errors (AUTH)
  INVALID_CREDENTIALS = 'AUTH_001',
  TOKEN_EXPIRED = 'AUTH_201',
  INSUFFICIENT_PERMISSIONS = 'AUTH_100',

  // System errors (SYS)
  DATABASE_ERROR = 'SYS_500',
  INTERNAL_ERROR = 'SYS_999'
}
```

```typescript
// errors/AppError.ts
export class AppError extends Error {
  constructor(
    public readonly code: ErrorCode,
    message: string,
    public readonly statusCode: number,
    public readonly details?: unknown
  ) {
    super(message);
    this.name = 'AppError';
    Error.captureStackTrace(this, this.constructor);
  }
}
```

**Expected Output (error response):**
```json
{
  "success": false,
  "error": {
    "code": "BIZ_100",
    "message": "Insufficient funds for this transaction",
    "details": { "balance": 30, "required": 50 }
  }
}
```

**Why this output:** The error code `BIZ_100` is machine-readable and categorised as a business rule violation (HTTP 422). The client can branch on this code to display a specific UI message or trigger a top-up flow. The message is human-readable, and the details provide structured context.

### Real-World Cases

- **Payment processing:** `PAY_SYS_503` triggers a retry; `PAY_BIZ_100` (insufficient funds) does not.
- **Authentication:** `AUTH_201` (token expired) triggers a refresh; `AUTH_001` (invalid credentials) prompts re-login.
- **E-commerce:** `BIZ_001` (order already cancelled) prevents duplicate cancellation attempts.

---

## Core Concept 2: Human-Readable Messages

### Definitions

**Core Definition:** Human-readable messages are consumer-facing error explanations written in plain language, safe to display inside user interfaces, and intended to help end users understand what went wrong and what to do next.

**Technical Definition:** Human-readable messages are part of the error response body, distinct from the machine-readable code. They should describe the problem **specifically for the current occurrence** — not be a generic repetition of the HTTP status. Per RFC 9457, the `detail` member is "a human-readable explanation specific to this occurrence of the problem," while `title` is "a short, human-readable summary of the problem type" that should not change from occurrence to occurrence except for localisation purposes. Messages should be safe to display in user interfaces — no stack traces, no internal identifiers, no database schema details.

**Beginner-Friendly Explanation:** A human-readable message is what you'd tell a customer: "Your email address is already registered. Try logging in instead." It's not "Error 409: duplicate key value violates unique constraint 'users_email_key'." The first message helps the user; the second exposes your database internals and confuses everyone.

### Purposes

- To write consumer-facing error alerts safe to display inside user interfaces.
- To guide users toward resolution (e.g., "Check your email address format").
- To avoid exposing internal implementation details (stack traces, schema names).
- To support localisation by separating the translatable message from the machine code.

### Syntax Rules and Structure

```json
{
  "error": {
    "code": "VAL_100",
    "title": "Validation Error",
    "detail": "The email address you provided is not in a valid format. Please check and try again.",
    "field": "email"
  }
}
```

| Component | Breakdown |
|-----------|-----------|
| `title` | Short, stable summary of the problem type (e.g., "Validation Error"). |
| `detail` | Specific explanation for this occurrence. |
| `field` | The field that caused the error (if applicable). |

**Rules:**
- **Never** expose stack traces, SQL queries, or internal file paths in messages.
- Messages should be **actionable** — tell the user what to do next.
- The `title` should be **stable** across occurrences; the `detail` should be specific.
- Use the `detail` field for localised, user-facing text.

**Constraints:**
- Messages must not be used for programmatic branching — use codes for that.
- Avoid technical jargon (e.g., "unique constraint violation") in user-facing messages.

### Annotated Code Example

```typescript
// ✅ GOOD: User-facing message
{
  "code": "AUTH_001",
  "title": "Authentication Failed",
  "detail": "The email or password you entered is incorrect. Please try again."
}

// ❌ BAD: Leaks internal details
{
  "code": "500",
  "message": "TypeError: Cannot read property 'password' of undefined at UserService.login (/app/services/user.service.js:42:15)"
}
```

**Expected Output (user-facing):**
```json
{
  "code": "AUTH_001",
  "title": "Authentication Failed",
  "detail": "The email or password you entered is incorrect. Please try again."
}
```

**Why this output:** The good example gives the user a clear, actionable message without exposing that the server uses JavaScript, where the code lives, or what line number caused the error. The bad example leaks the stack trace and internal file path.

### Real-World Cases

- **Registration forms:** "This email is already registered. Try logging in instead."
- **Payment failures:** "Your card was declined. Please check your card details or use a different payment method."
- **Rate limiting:** "You've made too many requests. Please wait a minute and try again."

---

## Core Concept 3: Machine-Readable Details

### Definitions

**Core Definition:** Machine-readable details are structured metadata payloads included in error responses that client applications can parse dynamically to make programmatic decisions — such as retry logic, field highlighting, or error routing.

**Technical Definition:** Machine-readable details extend the error response beyond the code and message. They include structured fields such as `retryable` (boolean), `retryAfter` (seconds), `field` (the field that caused the error), `details` (an object with additional context), and `requestId` (for tracing). Google Cloud's error model uses a `details` map of key-value pairs for additional information. The Microsoft DMA API includes `errorDetailType` and `errorDetails` to enable programmatic troubleshooting. A `retryable` flag is particularly valuable — clients can decide whether to retry automatically without understanding the specific error code.

**Beginner-Friendly Explanation:** Machine-readable details are like the extra data on a shipping label — the barcode, the weight, the dimensions — that machines can scan and act on. A human reads the address; a machine reads the barcode. In an error response, the human reads the message, and the machine reads the `retryable` flag or the `field` name.

### Purposes

- To structure metadata payloads for client applications to parse dynamically.
- To enable retry logic based on a `retryable` flag or `retryAfter` hint.
- To identify the specific field that caused a validation error.
- To provide request IDs for tracing and support.
- To carry domain-specific context (e.g., `balance`, `required`) for error resolution.

### Syntax Rules and Structure

```json
{
  "error": {
    "code": "BIZ_100",
    "title": "Insufficient Funds",
    "detail": "Your current balance is insufficient for this transaction.",
    "retryable": false,
    "field": "amount",
    "details": {
      "balance": 30,
      "required": 50,
      "currency": "USD"
    },
    "requestId": "req_abc123",
    "timestamp": "2026-01-15T10:30:00Z"
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `retryable` | Boolean | Whether the client may retry the request. |
| `retryAfter` | Integer | Seconds to wait before retrying. |
| `field` | String | The field that caused the error. |
| `details` | Object | Domain-specific context. |
| `requestId` | String | Unique identifier for tracing. |
| `timestamp` | ISO 8601 | When the error occurred. |

**Rules:**
- The `retryable` flag should be `true` only for transient errors (network, timeout, rate limit).
- `retryAfter` should accompany 429 responses to prevent retry storms.
- `details` should be extensible — clients should ignore unknown keys.
- `requestId` should be logged server-side and included in responses for support.

**Constraints:**
- `details` must not leak sensitive information (PII, internal IDs).
- Overloading `details` with too many fields reduces clarity; keep it focused.

### Annotated Code Example

```typescript
// errors/AppError.ts
export class AppError extends Error {
  public readonly retryable: boolean;
  public readonly retryAfter?: number;
  public readonly field?: string;
  public readonly details?: Record<string, unknown>;

  constructor(options: {
    code: ErrorCode;
    message: string;
    statusCode: number;
    retryable?: boolean;
    retryAfter?: number;
    field?: string;
    details?: Record<string, unknown>;
  }) {
    super(options.message);
    this.name = 'AppError';
    this.code = options.code;
    this.statusCode = options.statusCode;
    this.retryable = options.retryable ?? false;
    this.retryAfter = options.retryAfter;
    this.field = options.field;
    this.details = options.details;
    Error.captureStackTrace(this, this.constructor);
  }
}
```

```typescript
// Example usage in a service
throw new AppError({
  code: ErrorCode.INSUFFICIENT_FUNDS,
  message: 'Your current balance is insufficient for this transaction.',
  statusCode: 422,
  retryable: false,
  field: 'amount',
  details: { balance: 30, required: 50, currency: 'USD' }
});
```

**Expected Output:**
```json
{
  "success": false,
  "error": {
    "code": "BIZ_100",
    "message": "Your current balance is insufficient for this transaction.",
    "retryable": false,
    "field": "amount",
    "details": { "balance": 30, "required": 50, "currency": "USD" }
  }
}
```

**Why this output:** The client can read `retryable: false` and decide not to retry. The `field: "amount"` tells the frontend to highlight the amount input. The `details` object provides the specific numbers needed to display a helpful message ("You need $20 more"). The `requestId` (omitted here for brevity) enables support tracing.

### Real-World Cases

- **Retry logic:** A mobile app reads `retryable: true` and `retryAfter: 60` to automatically retry after a rate-limit error.
- **Form validation:** The frontend reads `field` and `details` to highlight the correct input and show a specific error.
- **Support tracing:** A customer reports an issue; support looks up the `requestId` in logs.

---

## Core Concept 4: Validation Errors

### Definitions

**Core Definition:** Validation errors are structured error responses returned when request data fails schema validation, formatted as arrays of field-level objects that map specific field names to their explicit rejection reasons.

**Technical Definition:** Validation errors are typically returned with HTTP status 422 Unprocessable Entity when the request is syntactically correct but semantically invalid (e.g., a negative amount, an unsupported currency), or 400 Bad Request when the request cannot be parsed at all (e.g., malformed JSON). The error body contains an array of field-level errors, each with a `field` (or `path`) and a `message`. Libraries like express-validator provide a `validationResult(req)` function that extracts errors in a format like `{ msg, param, location }`. Zod's `error.flatten().fieldErrors` produces an object keyed by field name with arrays of messages.

**Beginner-Friendly Explanation:** Validation errors are like a teacher marking a test — they don't just say "you failed." They mark each wrong answer: "Question 3: wrong format. Question 7: missing." In an API, a validation error tells the client exactly which field failed and why, so the frontend can highlight the correct input and show the right message.

### Purposes

- To format arrays of nested schema validation failures (mapping specific field names to their explicit rejection reasons).
- To enable frontend forms to highlight the correct fields and display specific messages.
- To distinguish between syntax errors (400) and semantic errors (422).
- To provide machine-readable field paths for programmatic error routing.

### Syntax Rules and Structure

```json
{
  "success": false,
  "error": {
    "code": "VAL_001",
    "title": "Validation Error",
    "status": 422,
    "errors": [
      { "field": "email", "message": "Invalid email format" },
      { "field": "password", "message": "Password must be at least 8 characters" },
      { "field": "profile.age", "message": "Age must be a positive integer" }
    ]
  }
}
```

| Component | Breakdown |
|-----------|-----------|
| `errors` | Array of field-level error objects. |
| `field` | Dot-notation path to the invalid field. |
| `message` | Human-readable rejection reason for that field. |

**Rules:**
- Return **all** validation errors, not just the first.
- Use dot notation (`profile.age`) for nested fields.
- The `field` path must match the request body structure.
- Use 422 for semantic validation failures; 400 for malformed requests.

**Constraints:**
- Different validation libraries produce different error shapes; normalise them in middleware.
- Field paths should be consistent with the API's naming conventions.

### Annotated Code Example

```typescript
// middleware/validate.ts
import { ZodSchema } from 'zod';

export function validateBody(schema: ZodSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      const fieldErrors = result.error.flatten().fieldErrors;
      const errors = Object.entries(fieldErrors).flatMap(
        ([field, messages]) =>
          messages.map((message) => ({ field, message }))
      );

      return res.status(422).json({
        success: false,
        error: {
          code: 'VAL_001',
          title: 'Validation Error',
          status: 422,
          errors
        }
      });
    }
    req.valid = result.data;
    next();
  };
}
```

```typescript
// Example Zod schema
const CreateUserSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email format'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  profile: z.object({
    age: z.number().int().positive('Age must be a positive integer')
  })
});
```

**Expected Output (for invalid request):**
```json
{
  "success": false,
  "error": {
    "code": "VAL_001",
    "title": "Validation Error",
    "status": 422,
    "errors": [
      { "field": "email", "message": "Invalid email format" },
      { "field": "password", "message": "Password must be at least 8 characters" },
      { "field": "profile.age", "message": "Age must be a positive integer" }
    ]
  }
}
```

**Why this output:** The middleware parses the request body against the Zod schema. If validation fails, `flatten().fieldErrors` produces an object keyed by field name. The middleware transforms this into a flat array of `{ field, message }` objects, which the frontend can iterate to highlight inputs and display messages.

### Real-World Cases

- **User registration:** Email format, password strength, and required fields.
- **E-commerce checkout:** Shipping address fields, payment card details, and quantity bounds.
- **Profile updates:** Nested fields like `address.zipCode` and `preferences.theme`.

---

## Core Concept 5: Authentication & Authorization Errors

### Definitions

**Core Definition:** Authentication errors (401 Unauthorized) indicate that the client has not proven its identity; authorization errors (403 Forbidden) indicate that the client's identity is known but it lacks the necessary privileges to access the resource.

**Technical Definition:** Per RFC 9110 and common practice, **401 Unauthorized** means the request lacks valid authentication credentials for the target resource — the client has not proven who it is. The server should include a `WWW-Authenticate` header giving the client instructions on how to authenticate. **403 Forbidden** means the server understood the request but refuses to fulfill it — the identity is known, but access is denied. Re-authenticating will not help. The key distinction: 401 = "Who are you?" (authentication); 403 = "I know who you are, but you can't do this" (authorization).

**Beginner-Friendly Explanation:** 401 is like a security guard saying "I don't know who you are — show me your ID." 403 is like the guard saying "I know who you are, but you're not on the VIP list — step aside." One prompts you to log in; the other tells you that logging in again won't help.

### Purposes

- To explicitly return 401 Unauthorized (identity unverified) vs. 403 Forbidden (identity known but lacks privileges).
- To include a `WWW-Authenticate` header on 401 responses to guide clients.
- To prevent information leakage by returning 404 for resources the user cannot access (hiding existence).
- To enable clients to distinguish between "re-authenticate" and "you don't have permission" flows.

### Syntax Rules and Structure

| Scenario | Status | Header | Meaning |
|----------|--------|--------|---------|
| No credentials provided | 401 | `WWW-Authenticate: Bearer` | Client hasn't authenticated. |
| Expired/invalid token | 401 | `WWW-Authenticate: Bearer error="invalid_token"` | Credentials are bad. |
| Valid credentials, insufficient role | 403 | — | Identity known, access denied. |
| Resource exists but user lacks access | 404 | — | Hide existence (security through obscurity). |

**Rules:**
- **401** must include a `WWW-Authenticate` header.
- **403** should not include `WWW-Authenticate` — re-authenticating won't help.
- For token refresh workflows: return 401 for expired access tokens (client should refresh); return 403 for invalid or revoked refresh tokens.
- Consider returning 404 instead of 403 when revealing existence would leak information.

**Constraints:**
- Some clients and browsers handle 401 and 403 differently; test both flows.
- Never return 403 for an unauthenticated request — that's a 401.

### Annotated Code Example

```typescript
// middleware/auth.ts
import jwt from 'jsonwebtoken';

export function requireAuth(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401)
      .set('WWW-Authenticate', 'Bearer realm="api"')
      .json({
        success: false,
        error: {
          code: 'AUTH_001',
          title: 'Authentication Required',
          detail: 'No authentication credentials were provided.'
        }
      });
  }

  const token = authHeader.split(' ')[1];
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401)
        .set('WWW-Authenticate', 'Bearer error="invalid_token", error_description="Token expired"')
        .json({
          success: false,
          error: {
            code: 'AUTH_201',
            title: 'Token Expired',
            detail: 'Your session has expired. Please refresh your token.'
          }
        });
    }
    return res.status(401).json({
      success: false,
      error: {
        code: 'AUTH_002',
        title: 'Invalid Token',
        detail: 'The authentication token is invalid.'
      }
    });
  }
}

export function requireRole(...roles: string[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({
        error: { code: 'AUTH_001', title: 'Authentication Required' }
      });
    }
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        success: false,
        error: {
          code: 'AUTH_100',
          title: 'Insufficient Permissions',
          detail: `Your role (${req.user.role}) does not allow access to this resource.`
        }
      });
    }
    next();
  };
}
```

**Expected Output (no token → 401):**
```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="api"

{"success": false, "error": { "code": "AUTH_001", "title": "Authentication Required", "detail": "No authentication credentials were provided." }}
```

**Expected Output (valid token, insufficient role → 403):**
```
HTTP/1.1 403 Forbidden

{"success": false, "error": { "code": "AUTH_100", "title": "Insufficient Permissions", "detail": "Your role (viewer) does not allow access to this resource." }}
```

**Why this output:** The 401 response includes the `WWW-Authenticate` header, guiding the client to provide credentials. The 403 response does not — re-authenticating won't grant the viewer role admin permissions. The error codes (`AUTH_001` vs. `AUTH_100`) allow the client to distinguish between the two flows programmatically.

### Real-World Cases

- **API authentication:** A mobile app receives 401 and navigates to the login screen; receives 403 and shows "You don't have permission."
- **Token refresh:** The client receives 401 with `error="invalid_token"` and uses its refresh token to obtain a new access token.
- **SaaS role management:** A "viewer" user tries to delete a document and receives 403; an "admin" succeeds.

---

## Core Concept 6: Database Errors

### Definitions

**Core Definition:** Database error interception is the practice of catching low-level database engine rejections — such as unique constraint violations, foreign key failures, and connection errors — at the repository layer and mapping them to safe, user-facing HTTP status codes without leaking schema details.

**Technical Definition:** Database drivers throw errors with engine-specific codes (e.g., PostgreSQL `23505` for unique violation, MySQL `ER_DUP_ENTRY`, MongoDB `11000` for duplicate key). These raw errors must never be exposed to clients — they reveal table names, column names, constraint names, and query structure. Instead, the repository or a centralised error handler maps them to HTTP statuses: unique constraint violations → **409 Conflict**, foreign key failures → **400 Bad Request** or **422**, connection failures → **503 Service Unavailable**, and serialisation failures → **409** or **503**. Libraries like `ds-express-errors` automate this mapping for MongoDB, Prisma, Mongoose, Sequelize, and Joi.

**Beginner-Friendly Explanation:** Database errors are like a mechanic telling you "the carburettor flange is misaligned." You don't need to know that — you need to know "your car won't start, and it needs to go to the shop." Similarly, when a database rejects a duplicate email, the client shouldn't see "duplicate key value violates unique constraint 'users_email_key'." They should see "This email is already registered" with a 409 status.

### Purposes

- To intercept and map low-level database engine rejections (e.g., unique constraint violations, foreign key failures) into user-safe HTTP status codes (409 Conflict) without leaking schema details.
- To provide consistent, predictable error responses regardless of the database engine.
- To prevent information disclosure through error messages.
- To automate mapping across multiple ORMs and databases.

### Syntax Rules and Structure

#### Common Database Error Mappings

| Database Error | PostgreSQL Code | MySQL Code | MongoDB Code | HTTP Status |
|----------------|----------------|-----------|--------------|-------------|
| Unique violation | `23505` | `ER_DUP_ENTRY` | `11000` | 409 Conflict |
| Foreign key violation | `23503` | `ER_NO_REFERENCED_ROW` | — | 400 / 422 |
| Not null violation | `23502` | `ER_BAD_NULL_ERROR` | — | 400 |
| Check violation | `23514` | — | — | 422 |
| Connection error | `08*` | — | — | 503 |
| Serialisation failure | `40001` | — | — | 409 / 503 |

#### Mapping Pattern

```typescript
function mapDatabaseError(err: unknown): AppError | null {
  // PostgreSQL unique violation
  if (err.code === '23505') {
    return new AppError({
      code: ErrorCode.CONFLICT,
      message: 'A resource with these details already exists.',
      statusCode: 409,
      details: { constraint: sanitiseConstraint(err.constraint) }
    });
  }

  // PostgreSQL foreign key violation
  if (err.code === '23503') {
    return new AppError({
      code: ErrorCode.INVALID_REFERENCE,
      message: 'The referenced resource does not exist.',
      statusCode: 422
    });
  }

  // MongoDB duplicate key
  if (err.code === 11000) {
    return new AppError({
      code: ErrorCode.CONFLICT,
      message: 'A resource with these details already exists.',
      statusCode: 409
    });
  }

  // Connection error
  if (err.code?.startsWith('08')) {
    return new AppError({
      code: ErrorCode.SERVICE_UNAVAILABLE,
      message: 'The service is temporarily unavailable. Please try again later.',
      statusCode: 503,
      retryable: true,
      retryAfter: 30
    });
  }

  return null; // Unknown database error → fall through to 500
}
```

**Rules:**
- **Never** expose raw database error messages, stack traces, or schema details to clients.
- Map unique constraint violations to **409 Conflict**, not 400 — the request is syntactically valid but conflicts with existing state.
- Map connection failures to **503** with a `retryable: true` flag.
- Sanitise any constraint names that are included in details (e.g., strip the table prefix).

**Constraints:**
- Different ORMs expose errors differently; centralise the mapping in one place.
- Some ORMs wrap the raw database error in their own class; access `err.original` or `err.parent` to get the underlying code.

### Annotated Code Example

```typescript
// errors/mapDatabaseError.ts
import { AppError } from './AppError';
import { ErrorCode } from './errorCodes';

export function mapDatabaseError(err: unknown): AppError {
  const pgCode = (err as any).code;

  // Unique violation (PostgreSQL 23505, MongoDB 11000)
  if (pgCode === '23505' || pgCode === 11000) {
    return new AppError({
      code: ErrorCode.CONFLICT,
      message: 'A resource with these details already exists.',
      statusCode: 409,
      // Do NOT include the raw constraint name
      details: { field: extractField(err) }
    });
  }

  // Foreign key violation
  if (pgCode === '23503') {
    return new AppError({
      code: ErrorCode.INVALID_REFERENCE,
      message: 'The referenced resource does not exist.',
      statusCode: 422
    });
  }

  // Connection error
  if (typeof pgCode === 'string' && pgCode.startsWith('08')) {
    return new AppError({
      code: ErrorCode.SERVICE_UNAVAILABLE,
      message: 'The service is temporarily unavailable. Please try again later.',
      statusCode: 503,
      retryable: true,
      retryAfter: 30
    });
  }

  // Unknown database error — log and return generic 500
  console.error('Unmapped database error:', err);
  return new AppError({
    code: ErrorCode.INTERNAL_ERROR,
    message: 'An unexpected error occurred.',
    statusCode: 500
  });
}
```

```typescript
// repositories/user.repository.ts
class UserRepository {
  async create(data: CreateUserDto) {
    try {
      return await this.db.query(
        'INSERT INTO users (email, name) VALUES ($1, $2) RETURNING *',
        [data.email, data.name]
      );
    } catch (err) {
      throw mapDatabaseError(err); // Map to safe AppError
    }
  }
}
```

**Expected Output (for duplicate email):**
```json
{
  "success": false,
  "error": {
    "code": "CONFLICT",
    "message": "A resource with these details already exists.",
    "statusCode": 409,
    "details": { "field": "email" }
  }
}
```

**Expected Output (for database down):**
```json
{
  "success": false,
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "The service is temporarily unavailable. Please try again later.",
    "statusCode": 503,
    "retryable": true,
    "retryAfter": 30
  }
}
```

**Why this output:** The repository catches the raw PostgreSQL error and passes it to `mapDatabaseError`. The unique violation (`23505`) is mapped to 409 Conflict with a sanitised field name (no constraint name). The connection error (`08*`) is mapped to 503 with `retryable: true`, telling the client it can retry after 30 seconds.

### Real-World Cases

- **User registration:** Duplicate email → 409 Conflict with "This email is already registered."
- **Order processing:** Foreign key violation (product doesn't exist) → 422 with "The referenced product does not exist."
- **Database maintenance:** Connection error → 503 with `retryable: true`, allowing the client to retry after 30 seconds.
- **Concurrent edits:** Serialisation failure → 409 Conflict, prompting the client to re-fetch and retry.

---

## References

- RFC 9457 — Problem Details for HTTP APIs — https://www.rfc-editor.org/rfc/rfc9457
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- RFC 7807 — Problem Details for HTTP APIs (obsoleted by RFC 9457) — https://datatracker.ietf.org/doc/html/rfc7807
- MDN — HTTP 401 Unauthorized — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401
- MDN — HTTP 403 Forbidden — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403
- MDN — HTTP 409 Conflict — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/409
- Express.js — Error Handling — https://expressjs.com/en/guide/error-handling.html
- Apidog — REST API Error Handling Best Practices: Status Codes, RFC 9457, and Retryable Errors — https://apidog.com/blog/rest-api-error-handling-best-practices/
- Stack Overflow — Best practices for structuring REST API error responses — https://stackoverflow.com/questions/79939339/
- Steve Kinney — Global Error Types in Express — https://stevekinney.com/courses/full-stack-typescript/global-error-types-in-express
- Steve Kinney — Typed-Error Middleware with Express — https://stevekinney.com/courses/full-stack-typescript/typed-error-middleware-with-express
- ds-express-errors — Centralised Error Handling for Express.js — https://ds-express-errors.dev/
- CyberPanel — 401 vs 403: Key Differences — https://cyberpanel.net/blog/401-vs-403
- Error Code Guide — Universal Dev Standards — https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/asiaostrich/universal-dev-standards/error-code-guide/SKILL.md
- Google Cloud — Datastream Error — https://docs.cloud.google.com/datastream/docs/reference/rest/v1/Error
- PostgREST — Error Source and HTTP Status Codes — https://postgrest.org/en/stable/errors.html
- express-validator — Result Processing — https://express-validator.github.io/docs/validation-result-api
- Zod — Error Formatting — https://zod.dev/error-formatting
- OWASP — REST Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html