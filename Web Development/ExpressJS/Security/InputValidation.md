# Input Validation — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Input validation** is the process of verifying that data received by an application conforms to the application’s expected structure, type, format, and business rules **before the data is used, stored, or passed to another component**. It is a **boundary control**: it operates at the point where untrusted data enters a trusted system and determines whether that data is permitted to proceed.

**Technical Definition.** Input validation enforces a **schema** — a declarative or imperative specification of allowable data — against every external input. The schema checks **syntax** (type, length, format, character set, structure) and **semantics** (range, consistency with other fields, and business meaning). Validation **rejects** non-conforming data rather than attempting to repair it. It must be performed **server-side**; client-side validation is a usability convenience that can be bypassed trivially. Validation is distinct from **sanitization** (which transforms data) and from **output encoding** (which neutralizes data at the point of use), and a complete defense uses all three in layers.

**Beginner-Friendly Explanation.** Input validation is **checking that what someone typed into your app is actually what you expected**. If a field is supposed to be a five-digit ZIP code, validation rejects `"90210; DROP TABLE users"` because it is not five digits. If a field is a date range, validation checks that the start date is before the end date. Validation is **the bouncer at the door**: if the data does not meet the rules, it does not get in. It must happen **on the server**, because anything that happens in the browser can be bypassed with `curl` or a proxy tool.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Server-side enforcement** | Validation must run on the server; client-side checks are usability aids, not security controls. |
| **Allowlist-oriented** | Define what is **allowed** and reject everything else, rather than trying to enumerate every malicious input. |
| **Syntax and semantics** | Check both the **form** (type, length, format) and the **meaning** (range, cross-field consistency) of the data. |
| **Boundary-focused** | Validate at every boundary where untrusted data enters: HTTP bodies, query parameters, headers, cookies, file uploads, and IPC. |
| **Reject, don’t repair** | The safest response to invalid input is rejection; sanitization is a separate, context-specific layer. |
| **Schema-driven** | Modern practice expresses validation rules as reusable schemas, often with automatic TypeScript type inference. |

---

### Prerequisites

- **HTTP fundamentals**: request bodies, query parameters, headers, and cookies.
- **Basic programming concepts**: types, objects, arrays, and functions.
- **Node.js and npm** (for the JavaScript library examples).
- **A code editor and a terminal** to run the examples.
- **Awareness of injection vulnerabilities** (SQL injection, XSS) that validation helps mitigate.

---

### Related Programming Areas

- **Security fundamentals**: input validation is a primary control against injection attacks.
- **API design**: request schemas are part of the API contract.
- **Sanitization and output encoding**: complementary layers that neutralize data at the point of use.
- **Type systems**: TypeScript-first validators like Zod provide both runtime validation and compile-time types.
- **Dependency management**: validation libraries are third-party dependencies that must be kept updated.

---

## Core Concepts / Features

---

## 1. Input Validation

### Definitions

**Core Definition.** **Input validation** is the technique of ensuring that **only properly formatted data enters a software component**, by checking incoming data against an explicit set of rules and rejecting anything that does not conform.

**Technical Definition.** Validation applies a **schema** to untrusted input. The schema specifies the **expected type** (string, number, boolean, object, array), **format** (email, URL, UUID, date), **length or range** (minimum and maximum), **allowed values** (enum, exact match), and **semantic constraints** (end date after start date, quantity not exceeding stock). Validation should be **allowlist-based**: the schema defines the acceptable set, and everything outside it is rejected. Validation must occur **after parsing but before business logic**, and the parser itself must be configured with size and depth limits to prevent resource exhaustion before validation can run.

**Beginner-Friendly Explanation.** Input validation is **the rules that decide what data is allowed in**. You write down what a valid username looks like (3–30 letters and numbers), what a valid email looks like (contains `@` and a domain), and what a valid age looks like (a whole number between 13 and 120). When data arrives, you check it against those rules. If it fails, you reject it with a clear error. If it passes, it goes into your application.

### Purposes

- **To prevent malformed data from corrupting application state** or causing unexpected exceptions.
- **To reduce the attack surface for injection attacks** by rejecting inputs that contain unexpected syntax.
- **To enforce business rules** such as date ranges, quantity limits, and required fields.
- **To provide clear, actionable error messages** to API consumers and users.
- **To protect downstream components** (databases, shells, template engines) from receiving data they cannot safely handle.

### Syntax Rules and Structure

**General pattern (imperative validation).**

```
if (!isValid(input)) {
  reject(input);
}
```

**General pattern (schema validation).**

```
schema = defineSchema({ field: rule1.rule2.rule3 });
result = schema.validate(input);
if (!result.success) reject(result.errors);
```

**Validation rule categories.**

| Category | What to Check | Example |
|---|---|---|
| **Type** | The expected JavaScript or data type | `string`, `number`, `boolean`, `object`, `array` |
| **Format** | The expected syntactic pattern | email, URL, UUID, ISO date |
| **Length / Range** | Minimum and maximum size or value | `min(3).max(30)`, `min(13).max(120)` |
| **Allowed values** | Exact membership in an enumerated set | `enum(["user", "admin"])` |
| **Required / Optional** | Presence and nullability | `.required()`, `.optional()` |
| **Semantic consistency** | Cross-field relationships | End date after start date |

**Constraints and limitations.**

- **Validation is not a substitute for output encoding**; data that passes validation may still be dangerous in a specific output context.
- **Validation is not a substitute for parameterized queries**; SQL injection must be prevented at the query layer, not by trying to filter out “bad” characters.
- **Regular expressions used for validation can be vulnerable to ReDoS**; complex patterns with nested quantifiers can be exploited to exhaust CPU.

### Annotated Complete Step by Step Code Examples

#### Example 1: Imperative validation in plain JavaScript

```javascript
// Step 1: Define a validation function for a user registration payload.
function validateUserInput(input) {
  const errors = [];

  // Step 2: Check type and presence.
  if (typeof input.username !== "string") {
    errors.push("username must be a string");
  } else {
    // Step 3: Check format and length (allowlist: letters, numbers, underscore).
    if (!/^[a-zA-Z0-9_]{3,30}$/.test(input.username)) {
      errors.push("username must be 3-30 letters, numbers, or underscores");
    }
  }

  // Step 4: Check email format.
  if (typeof input.email !== "string" || !/^[^@\s]+@[^@\s]+\.[^@\s]+$/.test(input.email)) {
    errors.push("email must be a valid email address");
  }

  // Step 5: Check semantic range.
  if (typeof input.age !== "number" || !Number.isInteger(input.age) || input.age < 13 || input.age > 120) {
    errors.push("age must be a whole number between 13 and 120");
  }

  return { valid: errors.length === 0, errors };
}

// Step 6: Use the validator.
const input = { username: "alice_99", email: "alice@example.com", age: 30 };
const result = validateUserInput(input);
console.log(result);
// Expected: { valid: true, errors: [] }
```

**Expected Output:**
```
{ valid: true, errors: [] }
```

**Why this output occurs.** Each field is checked against its type, format, and range rules. The username matches the allowlist regex, the email contains `@` and a domain, and the age is an integer within the allowed range. No errors are pushed, so `valid` is `true`.

---

#### Example 2: Schema-based validation with Zod

```typescript
// Step 1: Import Zod.
import { z } from "zod";

// Step 2: Define a schema with type, length, format, and range rules.
const UserSchema = z.object({
  username: z.string().min(3).max(30).regex(/^[a-zA-Z0-9_]+$/),
  email: z.string().email(),
  age: z.number().int().min(13).max(120).optional(),
});

// Step 3: Parse untrusted input.
const result = UserSchema.safeParse({
  username: "alice_99",
  email: "alice@example.com",
  age: 30,
});

// Step 4: Check the result.
if (result.success) {
  console.log("Valid:", result.data);
} else {
  console.error("Invalid:", result.error.flatten().fieldErrors);
}
// Expected: Valid: { username: 'alice_99', email: 'alice@example.com', age: 30 }
```

**Expected Output:**
```
Valid: { username: 'alice_99', email: 'alice@example.com', age: 30 }
```

**Why this output occurs.** Zod’s `safeParse` returns a discriminated union: `success: true` with `data`, or `success: false` with `error`. The input satisfies every rule, so the data is returned with its inferred TypeScript type. Zod derives the static type `z.infer<typeof UserSchema>` automatically, eliminating the need for a separate interface.

---

### Real-World Cases

- **User registration and login**: validating usernames, emails, passwords, and MFA codes.
- **E-commerce checkout**: validating quantities, prices, addresses, and payment tokens.
- **File uploads**: validating file type, size, and content-type before accepting the file.
- **API gateways**: enforcing request schemas before forwarding to backend services.

---

## 2. Sanitization

### Definitions

**Core Definition.** **Sanitization** (also called **input cleaning** or **output encoding**, depending on context) is the process of **transforming data so that it is safe to use in a specific interpreter context** — escaping, encoding, or removing characters that could otherwise be interpreted as executable syntax.

**Technical Definition.** Sanitization operates **after validation** (or in place of it for free-form fields where strict validation would be impractical). It neutralizes dangerous characters for a **specific sink**: SQL (escaping quotes and parameterizing), HTML (encoding `<`, `>`, `&`, `"`), shell (escaping metacharacters), XML (escaping entities), or LDAP (escaping filter characters). Sanitization is **context-dependent**: a string safe for a database query may be dangerous when rendered in HTML or passed to a shell command. OWASP recommends **validation over sanitization** where possible, because sanitization can inadvertently break legitimate data and must be applied at every point of use.

**Beginner-Friendly Explanation.** Sanitization is **cleaning up data so it cannot do harm in a particular place**. Validation says “is this data allowed?” Sanitization says “given that this data is allowed, how do I make it safe for this specific destination?” If you are putting a comment into an HTML page, sanitization encodes `<` so it displays as `&lt;` instead of starting a tag. If you are putting a value into a SQL query, sanitization is not enough — you must use **parameterized queries** so the value is never treated as SQL syntax at all.

### Purposes

- **To neutralize dangerous characters** that could be interpreted as executable syntax by a downstream interpreter.
- **To complement validation** for free-form fields (comments, notes, search boxes) where strict allowlists would be too restrictive.
- **To provide defense in depth** when validation cannot guarantee safety for every possible output context.
- **To support safe rendering of user-generated content** in HTML, XML, and other markup formats.

### Syntax Rules and Structure

**General pattern (HTML encoding).**

```
encoded = escapeHtml(rawInput);
```

**General pattern (SQL parameterization — the preferred approach).**

```
query = "SELECT * FROM users WHERE name = ?";
params = [userInput];
```

**Sanitization contexts and techniques.**

| Context | Technique | Example |
|---|---|---|
| **HTML output** | HTML entity encoding | `<` → `&lt;`, `&` → `&amp;` |
| **SQL** | Parameterized queries (not string escaping) | `WHERE name = ?` with bound parameter |
| **Shell** | Argument escaping or avoiding shell entirely | Avoid `exec()` with user input; use `execFile` with argument arrays |
| **XML** | XML entity encoding | `<` → `&lt;` within XML content |
| **URL** | Percent-encoding | Space → `%20`, `&` → `%26` |

**Constraints and limitations.**

- **Sanitization is context-specific**; a value sanitized for HTML may be dangerous in SQL.
- **Sanitization can corrupt legitimate data**; for example, stripping apostrophes breaks names like O’Brien.
- **Sanitization is not a substitute for parameterized queries**; SQL injection must be prevented at the query layer.
- **Sanitization is not a substitute for output encoding**; the same data may be rendered in multiple contexts.

### Annotated Complete Step by Step Code Examples

#### Example 1: HTML encoding before rendering user content

```javascript
// Step 1: Define an HTML encoder.
function escapeHtml(str) {
  return str
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#39;");
}

// Step 2: Simulate user-generated content containing a script tag.
const userComment = '<script>alert("xss")</script>Hello';

// Step 3: Encode before inserting into HTML.
const safeComment = escapeHtml(userComment);

// Step 4: Render the safe version.
console.log(safeComment);
// Expected: &lt;script&gt;alert(&quot;xss&quot;)&lt;/script&gt;Hello
```

**Expected Output:**
```
&lt;script&gt;alert(&quot;xss&quot;)&lt;/script&gt;Hello
```

**Why this output occurs.** The `<`, `>`, `"`, and `'` characters are replaced with their HTML entity equivalents. The browser renders the encoded string as **inert text** rather than executing it as a script. This is **output encoding**, the correct defense for HTML rendering contexts.

---

#### Example 2: Parameterized query instead of string concatenation

```javascript
// ❌ VULNERABLE: string concatenation.
// const query = `SELECT * FROM users WHERE name = '${userInput}'`;

// ✅ SAFE: parameterized query.
const query = "SELECT * FROM users WHERE name = ?";
const params = [userInput];

db.execute(query, params, (err, rows) => {
  // The database treats userInput as data, never as SQL syntax.
});
```

**Expected Output (conceptual):**
```
Input: "alice' OR '1'='1"
→ The query returns only rows where name exactly equals "alice' OR '1'='1" (no rows).
→ The injection payload is treated as a literal string, not as SQL.
```

**Why this output occurs.** Parameterization separates the **query template** from the **data**. The database engine parses the template first and binds the parameter as a value. The input cannot alter the query structure, regardless of which characters it contains. This is the definitive defense against SQL injection and is **superior to sanitization** for SQL contexts.

---

### Real-World Cases

- **Comment sections and forums**: HTML-encoding user comments before rendering.
- **Search boxes**: parameterizing search queries and escaping wildcard characters where necessary.
- **File upload filenames**: sanitizing filenames to prevent path traversal (`../`) and null-byte injection.
- **Logging**: encoding user data before writing to log files to prevent log injection.

---

## 3. Validation Libraries

### Definitions

**Core Definition.** **Validation libraries** are reusable software packages that provide **declarative schema definition, data parsing, and error reporting** for input validation, reducing the need to write ad-hoc validation logic manually.

**Technical Definition.** Modern validation libraries follow a **schema-first** model: the developer declares a schema (types, constraints, transformations), and the library provides a `parse` or `validate` method that returns either the validated data or a structured error object. Libraries differ in **type-system integration** (Zod and Valibot infer TypeScript types from schemas; Joi and Yup are JavaScript-first), **middleware integration** (express-validator integrates directly with Express request objects), and **ecosystem focus** (Joi is backend-oriented, Yup is form-oriented, Zod is full-stack TypeScript-oriented).

**Beginner-Friendly Explanation.** Validation libraries are **toolkits that save you from writing the same validation code over and over**. Instead of manually checking `typeof input.name === "string" && input.name.length >= 3` for every field, you write a schema once — `z.string().min(3)` — and the library does the checking. The best libraries also give you **automatic TypeScript types**, so your code and your validation rules stay in sync.

### Purposes

- **To reduce boilerplate** by expressing validation rules declaratively.
- **To provide consistent, structured error messages** across endpoints.
- **To integrate validation with the type system** (especially in TypeScript) so that validated data is automatically typed.
- **To centralize validation rules** for reuse across multiple endpoints and services.
- **To reduce the risk of validation gaps** by making rules explicit and reviewable.

### Syntax Rules and Structure

**Comparison of major libraries.**

| Library | Approach | TypeScript Support | Best For |
|---|---|---|---|
| **Zod** | TypeScript-first schema; `z.object`, `z.string().min()` | **Excellent** (schema infers types) | Full-stack TypeScript, React, modern Node.js |
| **Joi** | Schema-based; `Joi.object({ ... })` | Good (but not type-inferring) | Backend-heavy Node.js with complex conditional rules |
| **express-validator** | Middleware chain; `body("email").isEmail()` | Good (middleware-based) | Simple Express APIs with route-level validation |
| **Yup** | Schema-based; similar to Joi | Good | React forms and frontend validation |

**Zod syntax pattern.**

```typescript
const schema = z.object({
  field: z.string().min(3).max(30),
});
const result = schema.safeParse(input);
```

**Joi syntax pattern.**

```javascript
const schema = Joi.object({
  field: Joi.string().min(3).max(30).required(),
});
const { error, value } = schema.validate(input);
```

**express-validator syntax pattern.**

```javascript
app.post("/route", [
  body("email").isEmail().normalizeEmail(),
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
  // ...
});
```

**Constraints and limitations.**

- **Zod’s `parse` throws on failure**; use `safeParse` to return a result object instead of throwing.
- **Joi values are optional by default**; use `.required()` to enforce presence.
- **express-validator requires calling `validationResult(req)`** to retrieve errors; it does not throw automatically.
- **All libraries introduce a dependency** that must be kept updated for security and compatibility.

### Annotated Complete Step by Step Code Examples

#### Example 1: Zod schema with safe parsing and error formatting

```typescript
// Step 1: Import Zod.
import { z } from "zod";

// Step 2: Define a reusable schema.
const SignupSchema = z.object({
  username: z.string().min(3, "Username must be at least 3 characters").max(30),
  email: z.string().email("Invalid email format"),
  password: z.string().min(8, "Password must be at least 8 characters"),
});

// Step 3: Validate untrusted input.
const result = SignupSchema.safeParse({
  username: "ab",
  email: "not-an-email",
  password: "short",
});

// Step 4: Handle the result.
if (!result.success) {
  console.log(result.error.flatten().fieldErrors);
}
// Expected:
// {
//   username: ['Username must be at least 3 characters'],
//   email: ['Invalid email format'],
//   password: ['Password must be at least 8 characters']
// }
```

**Expected Output:**
```json
{
  "username": ["Username must be at least 3 characters"],
  "email": ["Invalid email format"],
  "password": ["Password must be at least 8 characters"]
}
```

**Why this output occurs.** `safeParse` returns `success: false` with a `ZodError`. The `flatten()` method converts the error into a field-keyed object, making it easy to return a structured `400 Bad Request` response. Each field’s message is the one defined in the schema, providing clear feedback to the client.

---

#### Example 2: Joi schema with conditional rules

```javascript
// Step 1: Import Joi.
const Joi = require("joi");

// Step 2: Define a schema with cross-field constraints.
const schema = Joi.object({
  username: Joi.string().alphanum().min(3).max(30).required(),
  password: Joi.string().pattern(new RegExp("^[a-zA-Z0-9]{3,30}$")),
  repeat_password: Joi.ref("password"),
  birth_year: Joi.number().integer().min(1900).max(2013),
})
  .with("username", "birth_year") // username requires birth_year
  .xor("password", "access_token"); // exactly one of password or access_token

// Step 3: Validate.
const { error, value } = schema.validate({
  username: "alice",
  password: "secret123",
  repeat_password: "secret123",
  birth_year: 1990,
});

// Step 4: Check the result.
if (error) {
  console.error(error.details);
} else {
  console.log("Valid:", value);
}
// Expected: Valid: { username: 'alice', password: 'secret123', repeat_password: 'secret123', birth_year: 1990 }
```

**Expected Output:**
```
Valid: { username: 'alice', password: 'secret123', repeat_password: 'secret123', birth_year: 1990 }
```

**Why this output occurs.** Joi evaluates the field-level rules and the cross-field constraints (`.with()` and `.xor()`). The input satisfies every constraint, so `error` is `undefined` and `value` contains the validated data. If `password` and `repeat_password` did not match, or if `birth_year` were missing, Joi would return a `ValidationError` with detailed messages.

---

#### Example 3: express-validator middleware chain

```javascript
// Step 1: Import the middleware functions.
const { body, validationResult } = require("express-validator");

// Step 2: Define a route with a validation chain.
app.post(
  "/api/users",
  [
    body("username")
      .isLength({ min: 3 })
      .withMessage("Username must be at least 3 characters")
      .isAlphanumeric()
      .withMessage("Username must contain only letters and numbers"),
    body("email")
      .isEmail()
      .withMessage("Must provide a valid email address")
      .normalizeEmail(),
    body("password")
      .isLength({ min: 6 })
      .withMessage("Password must be at least 6 characters"),
  ],
  (req, res) => {
    // Step 3: Collect validation errors.
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    // Step 4: Process valid data.
    res.status(201).json({ message: "User registered", user: req.body });
  }
);
```

**Expected Output (invalid request):**
```json
{
  "errors": [
    { "value": "j$", "msg": "Username must be at least 3 characters", "param": "username", "location": "body" },
    { "value": "j$", "msg": "Username must contain only letters and numbers", "param": "username", "location": "body" },
    { "value": "notanemail", "msg": "Must provide a valid email address", "param": "email", "location": "body" },
    { "value": "123", "msg": "Password must be at least 6 characters", "param": "password", "location": "body" }
  ]
}
```

**Why this output occurs.** Each `body()` call creates a **validation chain** that runs as Express middleware. `validationResult(req)` collects the errors from the chains. The route handler checks `errors.isEmpty()` and returns a `400` with the structured error array if any rule failed.

---

### Real-World Cases

- **Zod**: widely used in TypeScript full-stack applications, including tRPC and React Hook Form integrations.
- **Joi**: used in backend Node.js services requiring complex conditional and cross-field validation.
- **express-validator**: the standard choice for Express APIs needing route-level validation without a separate schema layer.
- **Yup**: common in React form libraries such as Formik.

---

## 4. Input Validation Best Practices

### Definitions

**Core Definition.** **Input validation best practices** are empirically established guidelines for designing and implementing validation that is **secure, maintainable, and complete**, derived from OWASP, CWE, and industry experience.

**Technical Definition.** Best practices center on **server-side enforcement**, **allowlist validation**, **syntax and semantic checking**, **defense in depth** (validation plus parameterization plus output encoding), **parser safety** (size and depth limits before parsing), **ReDoS avoidance**, and **consistent error handling**. Validation must be applied to **all input sources** — not just form bodies, but also query parameters, headers, cookies, and file uploads.

**Beginner-Friendly Explanation.** Best practices are **the habits that make validation actually work**. Validate on the server, always. Prefer allowlists (“only these characters are allowed”) over denylists (“block these bad characters”). Check both the form and the meaning of the data. Use parameterized queries and output encoding in addition to validation, because validation alone cannot stop injection in every context. Set size limits on requests before parsing them. And test your validation with malicious inputs to make sure it holds.

### Purposes

- **To ensure validation cannot be bypassed** by client-side manipulation.
- **To cover all input sources** rather than only the most obvious ones.
- **To prevent resource-exhaustion attacks** (ReDoS, large payloads) that occur before validation runs.
- **To provide defense in depth** so that a failure in one layer does not lead to a vulnerability.
- **To make validation maintainable** through schemas, constants, and reusable rules.

### Syntax Rules and Structure

**Best-practice checklist.**

| Area | Best Practice | Why It Matters |
|---|---|---|
| **Server-side** | Validate on the server, not just the client | Client-side checks are trivially bypassed |
| **Allowlist** | Define allowed values/characters; reject everything else | Denylists are bypassable with encoding and case variations |
| **Syntax + semantics** | Check type, length, format **and** range, cross-field consistency | Syntactically valid data can still be semantically invalid |
| **All sources** | Validate bodies, query parameters, headers, cookies, file uploads | Any untrusted input is an attack vector |
| **Parser limits** | Set request size limits and maximum nesting depth before parsing | A parser can be exhausted before validation runs |
| **ReDoS awareness** | Avoid regex patterns with nested quantifiers; test with tools | Poorly designed regexes can cause denial of service |
| **Defense in depth** | Pair validation with parameterized queries and output encoding | Validation alone does not prevent all injection |
| **Reject, don’t repair** | Reject invalid input rather than trying to sanitize it | Sanitization can silently corrupt data and create new bypasses |
| **Structured errors** | Return field-level, machine-readable error objects | Clients need actionable feedback to correct requests |

**Constraints and limitations.**

- **Validation is not a complete security solution**; it must be combined with other controls.
- **Overly strict allowlists can break legitimate use cases** (e.g., names with apostrophes, international characters); balance security with usability.
- **Validation libraries add dependencies** that must be kept updated.

### Annotated Complete Step by Step Code Examples

#### Example 1: Defense-in-depth validation pipeline

```typescript
import { z } from "zod";

// Layer 1: Request size limit (configure before parsing).
app.use(express.json({ limit: "100kb" }));

// Layer 2: Schema validation (syntax and semantics).
const CreateOrderSchema = z.object({
  productId: z.string().uuid("Invalid product ID"),
  quantity: z.number().int().min(1).max(100),
  notes: z.string().max(500).optional(),
});

app.post("/orders", (req, res) => {
  // Layer 3: Validate the body.
  const result = CreateOrderSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ errors: result.error.flatten().fieldErrors });
  }

  // Layer 4: Authorization check (separate from validation).
  if (!req.user.canOrder(result.data.productId)) {
    return res.status(403).json({ error: "Forbidden" });
  }

  // Layer 5: Parameterized database query.
  db.query("INSERT INTO orders (product_id, quantity, notes) VALUES (?, ?, ?)", [
    result.data.productId,
    result.data.quantity,
    result.data.notes ?? null,
  ]);

  res.status(201).json({ success: true });
});
```

**Expected Output (conceptual):**
```
Valid request with correct permissions → 201 Created.
Invalid body → 400 Bad Request with field-level errors.
Valid body but unauthorized → 403 Forbidden.
SQL injection payload in notes → stored as inert text (parameterized query).
```

**Why this output occurs.** Five independent layers protect the endpoint: a size limit prevents large-payload attacks; Zod validates type, format, and range; authorization is checked separately (validation does not imply permission); and the database query is parameterized so user input never becomes SQL syntax. This is **defense in depth** applied to input handling.

---

#### Example 2: Allowlist validation with a safe regex

```javascript
// ✅ Allowlist: only lowercase letters, numbers, and underscores.
const USERNAME_PATTERN = /^[a-z0-9_]{3,16}$/;

function isValidUsername(input) {
  return typeof input === "string" && USERNAME_PATTERN.test(input);
}

// Test cases.
console.log(isValidUsername("alice_99")); // true
console.log(isValidUsername("ALICE")); // false (uppercase not allowed)
console.log(isValidUsername("alice!")); // false (special character)
console.log(isValidUsername("alice<script>")); // false (contains < and >)
```

**Expected Output:**
```
true
false
false
false
```

**Why this output occurs.** The regex defines the **allowlist**: only lowercase letters, numbers, and underscores, with a length of 3–16. Any input outside this set is rejected. The pattern is **anchored** (`^` and `$`) and uses a **bounded quantifier** (`{3,16}`) to avoid catastrophic backtracking. This is the recommended allowlist approach for username validation.

---

### Real-World Cases

- **OWASP Proactive Controls**: C3 (Validate All Input and Handle Exceptions) is a foundational control in the OWASP Top 10 Proactive Controls.
- **CWE-20**: Improper Input Validation is a root cause of injection, XSS, and path traversal vulnerabilities.
- **PCI DSS and HIPAA**: require input validation as part of secure application development.
- **Modern TypeScript stacks**: use Zod schemas shared between client and server to enforce validation at every boundary.

---

## References

- OWASP Input Validation Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- OWASP Top 10 Proactive Controls: C5 — Validate All Inputs - https://top10proactive.owasp.org/archive/2018/c5-validate-inputs/
- OWASP Top 10 Proactive Controls V2 (PDF) — Input Validation - https://wiki.owasp.org/images/9/9b/OWASP_Top_10_Proactive_Controls_V2.pdf
- OWASP Input Sanitization vs Validation (Safeguard.sh) - https://safeguard.sh/resources/blog/what-is-input-sanitization
- OWASP Top 10 Security Checklist (2026) - https://github.com/boshu2/agentops/blob/0c30caf72fd280c8611756dad4373d01832a7921/skills/security/references/owasp-checklist.md
- Zod Documentation — Schemas and Parsing - https://zod.dev/
- Zod Validation — Core Examples - https://github.com/agents-inc/skills/blob/707467a78f96fb7d684e55e1f6281e541a736fd6/src/skills/web-forms-zod-validation/examples/core.md
- Joi API v17.13.8 - https://joi.dev/api/17.x.x
- express-validator — Getting Started - https://express-validator.github.io/docs/6.13.0/
- Express API Validation (Compile-N-Run) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/10-express-rest-api/9-express-api-validation.mdx
- OWASP Regular Expression Denial of Service (ReDoS) - https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS
- CWE-20: Improper Input Validation - https://cwe.mitre.org/data/definitions/20.html
- MDN Web Docs — Client-side form validation - https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation