# Zod — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Zod is a TypeScript-first schema validation library with static type inference. You define a schema once, and Zod both validates data at runtime and automatically generates the matching TypeScript type.

**Technical Definition**
Zod is a TypeScript-first schema validation library with static type inference. To validate data, you must first define a schema — schemas represent types, from simple primitive values to complex nested objects and arrays. Schemas are constructed using type factories (`z.string()`, `z.number()`, `z.object()`, `z.array()`, etc.) and validated with `.parse()` or `.safeParse()`. Zod infers a static TypeScript type from each schema, extractable via `z.infer<typeof schema>` (an alias for `z.output`). When schemas include transformations, input and output types diverge and can be extracted separately with `z.input` and `z.output`. Zod schemas support composition (`.extend`, `.merge`, `.pick`, `.omit`, `.partial`), custom validation (`.refine`, `.superRefine`), and transformation (`.transform`, `.pipe`).

**Beginner-Friendly Explanation**
Zod is a tool that checks whether data is correct at runtime and automatically creates the matching TypeScript type for you. You describe what valid data looks like — “name is a string, email is a valid email, age is an optional positive number” — and Zod enforces those rules. If the data is good, you get back a fully typed, validated value. If not, Zod tells you exactly which fields failed and why. Because the type comes from the schema, you never have to write the same type twice.

---

### Key Characteristics

- **TypeScript-first** — Zod is designed from the ground up for TypeScript, with automatic type inference and full IDE autocomplete.
- **Single source of truth** — One schema provides both runtime validation and the compile-time TypeScript type; you never duplicate type definitions.
- **Type-safe parsing** — `.parse()` returns the schema's inferred output type, and TypeScript knows the exact shape of the result.
- **Rich composition** — `.extend`, `.merge`, `.pick`, `.omit`, `.partial`, `.required`, and `.deepPartial` make schemas reusable and DRY.
- **Custom validation and transformation** — `.refine`, `.superRefine`, `.transform`, and `.pipe` handle logic beyond built-in validators.
- **Structured errors** — `$ZodError` instances contain an `.issues` array with machine-readable codes and human-readable messages.
- **Express-friendly** — `safeParse()` returns a result object without throwing, making it ideal for middleware that must return `400` responses.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()` (for Express integration).
- TypeScript 5.0 or higher with `strict` mode enabled (for full type inference benefits).
- The `zod` package installed (`npm install zod`).
- Understanding of TypeScript types, generics, and discriminated unions.
- Familiarity with Express middleware and the `(req, res, next)` signature.

---

### Related Programming Areas

- **TypeScript** — Zod bridges runtime validation and compile-time types.
- **Express validation** — Zod schemas are wrapped in middleware to validate `req.body`, `req.query`, and `req.params`.
- **API contracts** — Schemas define and document the input and output shapes of an API.
- **Security** — Validation prevents malformed or malicious data from reaching business logic and databases.
- **Schema composition** — Reusable schema fragments reduce duplication across endpoints.
- **OpenAPI generation** — Zod schemas can be converted to OpenAPI specifications.

---

## Core Concept 1: Zod

### Definitions

**Core Definition**
Zod is the library — a TypeScript-first schema validation toolkit that provides runtime validation with static type inference.

**Technical Definition**
Zod exports a namespace of schema factories (`z.string()`, `z.number()`, `z.object()`, `z.array()`, `z.union()`, `z.enum()`, `z.literal()`, etc.) and utilities (`z.infer`, `z.output`, `z.input`, `z.coerce`, `z.preprocess`, `z.lazy`). It is available in two distributions: the full `zod` package (compiler and runtime) and `zod/mini` (a smaller, tree-shakable variant with a check-based API). Zod 4 unified error customization under a single `error` parameter, replacing the older `message` and `errorMap` parameters.

**Beginner-Friendly Explanation**
Zod is the package you install and import. It gives you all the tools you need: type factories to describe data, parsing methods to validate it, and type utilities to extract TypeScript types from schemas.

---

### Purposes

- To provide runtime validation that TypeScript's compile-time types cannot enforce.
- To eliminate the duplication between validation logic and TypeScript type definitions.
- To offer a composable, expressive API for describing data shapes.
- To integrate cleanly with Express, NestJS, and other Node.js frameworks.

---

### Syntax Rules and Structure

#### Complete General Syntax: Import and Basic Usage

```typescript
import { z } from 'zod';

// Define a schema
const User = z.object({
  id: z.number(),
  name: z.string(),
  email: z.email(),
  age: z.number().optional()
});

// Validate with .parse() (throws on failure)
const user = User.parse(data);

// Validate with .safeParse() (returns result object)
const result = User.safeParse(data);
if (!result.success) {
  console.log(result.error.issues);
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `import { z } from 'zod'` | Imports the Zod namespace |
| `z.object({ ... })` | Creates an object schema |
| `z.string()`, `z.number()` | Primitive type factories |
| `.parse(data)` | Validates and returns the typed output (throws on failure) |
| `.safeParse(data)` | Validates and returns `{ success, data, error }` |

#### Syntax Rules

- Schemas are created using type factories; they are immutable and can be reused.
- `.parse()` throws a `$ZodError` on failure; `.safeParse()` never throws and should be preferred in Express middleware.
- Use `z.infer<typeof schema>` to extract the TypeScript type from a schema.
- Zod 4 accepts an `error` parameter for custom messages on virtually every API; the v3 `message` parameter is deprecated.

#### Constraints and Limitations

- Zod is runtime validation; it adds a small per-request overhead compared to no validation.
- `.parse()` throwing means uncaught errors can crash Express handlers unless wrapped or replaced with `.safeParse()`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Minimal Zod Schema and Validation

```typescript
import { z } from 'zod';

// Define a simple object schema
const Player = z.object({
  username: z.string(),
  xp: z.number()
});

// Valid data
const valid = Player.parse({ username: 'billie', xp: 100 });
console.log(valid);
// { username: 'billie', xp: 100 }

// Invalid data
const invalid = Player.safeParse({ username: 'billie', xp: '100' });
console.log(invalid.success); // false
console.log(invalid.error.issues);
// [{ code: 'invalid_type', expected: 'number', received: 'string', path: ['xp'], message: '...' }]
```

**Step-by-Step Explanation**

1. **Schema definition** — `z.object()` composes the `username` and `xp` fields into a validated shape.
2. **Valid parsing** — `.parse()` returns the typed value on success.
3. **Invalid parsing** — `.safeParse()` returns `success: false` and a structured `error.issues` array.

**Expected Output**

```
{ username: 'billie', xp: 100 }
false
[{ code: 'invalid_type', expected: 'number', received: 'string', path: ['xp'], ... }]
```

**Why This Output Occurs**

- Zod validates the runtime types against the schema; valid data is returned as-is, invalid data produces an issue array with the failing path.

---

### Real-World Cases

- **Express API request validation** — Validating `req.body` before business logic executes.
- **Shared schemas across client and server** — Reusing the same Zod schema in React forms and Express handlers.
- **Database output parsing** — Validating data returned from a database before using it.

---

## Core Concept 2: Schemas

### Definitions

**Core Definition**
A Zod schema is an immutable object that describes the expected type and constraints of a value, used for both runtime validation and TypeScript type inference.

**Technical Definition**
Schemas are created by Zod's type factories and represent types from primitives (`z.string()`, `z.number()`, `z.boolean()`, `z.bigint()`, `z.symbol()`, `z.undefined()`, `z.null()`) to complex structures (`z.object()`, `z.array()`, `z.union()`, `z.enum()`, `z.literal()`, `z.tuple()`, `z.record()`, `z.map()`, `z.set()`, `z.date()`). Each schema supports chainable methods for constraints (`.min()`, `.max()`, `.email()`, `.regex()`), modifiers (`.optional()`, `.nullable()`, `.default()`), and composition. Coercion variants (`z.coerce.string()`, `z.coerce.number()`) convert input values before validation.

**Beginner-Friendly Explanation**
A schema is the blueprint for your data. It says “this field is a string, that field is a number between 1 and 100, this other field is optional.” You build a schema once and reuse it wherever that shape of data appears.

---

### Purposes

- To describe the exact shape and constraints of data in a reusable, declarative form.
- To support primitives, objects, arrays, unions, enums, literals, and nested structures.
- To provide default values, optionality, nullability, and type coercion.
- To serve as the input to `.parse()`, `.safeParse()`, and type inference.

---

### Syntax Rules and Structure

#### Complete General Syntax: Primitive and Object Schemas

```typescript
import { z } from 'zod';

// Primitives
const str = z.string();
const num = z.number();
const bool = z.boolean();
const date = z.date();

// Coercion (converts input before validating)
const coercedNum = z.coerce.number(); // Number(input)
const coercedBool = z.coerce.boolean(); // Boolean(input)

// Object schema
const User = z.object({
  id: z.number(),
  name: z.string().min(1).max(100),
  email: z.email(),
  age: z.number().int().positive().optional(),
  role: z.enum(['admin', 'editor', 'viewer']).default('viewer')
});
```

**Component Breakdown**

| Schema Type | Common Rules |
|-------------|-------------|
| `z.string()` | `.min()`, `.max()`, `.length()`, `.regex()`, `.startsWith()`, `.endsWith()`, `.trim()`, `.toLowerCase()` |
| `z.number()` | `.int()`, `.min()`, `.max()`, `.positive()`, `.negative()`, `.nonnegative()` |
| `z.object()` | `.extend()`, `.merge()`, `.pick()`, `.omit()`, `.partial()`, `.required()`, `.strict()` |
| `z.array()` | `.min()`, `.max()`, `.length()`, `.nonempty()` |
| `z.enum()` | `.exclude()`, `.extract()` |
| `z.coerce.*` | Coerces input using built-in constructors before validation |

#### Syntax Rules

- Object keys are required by default; use `.optional()` for optional fields.
- `.default(value)` applies when the input is `undefined`.
- `.nullable()` allows `null`; `.nullish()` allows both `null` and `undefined`.
- `z.coerce.boolean()` uses JavaScript truthiness, so `"false"` becomes `true`; use `.transform()` or `z.pipe()` for precise boolean parsing.

#### Constraints and Limitations

- Empty strings are allowed by default in `z.string()`; use `.min(1)` or `.nonempty()` to reject them.
- `z.object()` strips unknown keys by default; use `.strict()` to reject them or `.passthrough()` to preserve them.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Nested Object and Array Schema

```typescript
import { z } from 'zod';

const OrderSchema = z.object({
  orderId: z.uuid(),
  items: z.array(
    z.object({
      productId: z.number().int().positive(),
      quantity: z.number().int().min(1)
    })
  ).min(1),
  shipping: z.object({
    street: z.string(),
    city: z.string(),
    zip: z.string().regex(/^\d{5}$/)
  })
});

const validOrder = {
  orderId: '123e4567-e89b-12d3-a456-426614174000',
  items: [{ productId: 1, quantity: 2 }],
  shipping: { street: 'Main St', city: 'Springfield', zip: '12345' }
};

console.log(OrderSchema.safeParse(validOrder).success); // true
```

**Expected Output**

```
true
```

**Why This Output Occurs**

- All nested fields satisfy the schema; `z.array().min(1)` ensures at least one item, and the regex constrains the ZIP code format.

---

### Real-World Cases

- **E-commerce orders** — Nested item arrays and shipping addresses.
- **User profiles** — Optional fields, enums for roles, and defaults.
- **Configuration schemas** — Validating environment variables with coercion.

---

## Core Concept 3: Type Inference

### Definitions

**Core Definition**
Type inference is Zod's ability to automatically generate a TypeScript type from a schema, eliminating the need to write and maintain separate type definitions.

**Technical Definition**
Zod infers a static type from each schema definition. The `z.infer<typeof schema>` utility extracts the **output** type (after transformations), and is identical to `z.output`. When schemas include transformations, the **input** type (before transformation) differs and can be extracted with `z.input`. Refinements can narrow the output type using type predicates. Recursive types are supported via `z.lazy()` with an explicit type annotation.

**Beginner-Friendly Explanation**
Type inference means you write the schema once, and TypeScript automatically knows the type. You do not have to write `interface User { name: string; age?: number }` separately — `z.infer<typeof UserSchema>` gives you that type for free.

---

### Purposes

- To eliminate duplication between validation logic and TypeScript type definitions.
- To ensure that the type used by handlers and services always matches the validation schema.
- To support separate input and output types for schemas with transformations.
- To enable type narrowing through refinements.

---

### Syntax Rules and Structure

#### Complete General Syntax: `z.infer`, `z.input`, `z.output`

```typescript
import { z } from 'zod';

const User = z.object({
  id: z.number(),
  name: z.string(),
  email: z.email(),
  age: z.number().optional()
});

// Output type (after transforms) — same as z.infer
type User = z.infer<typeof User>;
// { id: number; name: string; email: string; age?: number }

// Input type (before transforms)
type UserInput = z.input<typeof User>;

// Output type explicitly
type UserOutput = z.output<typeof User>;
```

**Component Breakdown**

| Utility | Extracts |
|---------|----------|
| `z.infer<typeof S>` | Output type; alias for `z.output` |
| `z.input<typeof S>` | Input type (before transformations) |
| `z.output<typeof S>` | Output type (after transformations) |

#### Syntax Rules

- Use `z.infer<typeof schema>` for most cases; it gives the output type.
- Use `z.input<typeof schema>` when the schema transforms data (e.g., string to number) and you need the pre-transform type.
- Use a type predicate with `.refine()` to narrow the output type.

#### Constraints and Limitations

- `z.infer` reflects the **output** type; if the schema transforms, the inferred type may not match the raw input.
- Recursive schemas require an explicit type annotation with `z.lazy()`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Input vs Output Types with Transformation

```typescript
import { z } from 'zod';

const FormSchema = z.object({
  name: z.string(),
  age: z.string().transform(Number),
  subscribe: z.string().transform((val) => val === 'true')
});

type FormInput = z.input<typeof FormSchema>;
// { name: string; age: string; subscribe: string }

type FormOutput = z.output<typeof FormSchema>;
// { name: string; age: number; subscribe: boolean }

const raw: FormInput = { name: 'Alice', age: '30', subscribe: 'true' };
const parsed: FormOutput = FormSchema.parse(raw);
console.log(parsed);
// { name: 'Alice', age: 30, subscribe: true }
```

**Expected Output**

```
{ name: 'Alice', age: 30, subscribe: true }
```

**Why This Output Occurs**

- The `.transform()` calls convert the string inputs to a number and a boolean; `z.output` reflects the post-transform types.

---

### Real-World Cases

- **Form handling** — `z.input` types the raw string form data, `z.output` types the parsed values.
- **API contracts** — Sharing `z.infer` types between frontend and backend.
- **Type narrowing** — Using `.refine()` with type predicates to narrow output types.

---

## Core Concept 4: Schema Composition

### Definitions

**Core Definition**
Schema composition is the practice of building complex schemas by extending, merging, picking, omitting, or partially applying existing schemas, rather than defining each schema from scratch.

**Technical Definition**
Zod object schemas support `.extend(shape)` to add or overwrite fields, `.merge(other)` as shorthand for `A.extend(B.shape)`, `.pick(keys)` and `.omit(keys)` to select or remove fields, `.partial()` and `.deepPartial()` to make fields optional, `.required()` to make fields mandatory, and `.passthrough()` to preserve unknown keys. When two schemas share keys, `.merge()` gives precedence to the merged schema's fields. `z.intersection(A, B)` combines two schemas, and `z.union()` represents alternatives.

**Beginner-Friendly Explanation**
Composition lets you avoid copy-pasting schemas. You define a base schema, then derive variations: add fields with `.extend()`, remove fields with `.omit()`, make everything optional with `.partial()`. This keeps your schemas DRY and consistent.

---

### Purposes

- To reuse common schema fragments across multiple endpoints.
- To derive create, update, and response schemas from a single base schema.
- To keep schemas maintainable as the application grows.
- To avoid drift between related schemas.

---

### Syntax Rules and Structure

#### Complete General Syntax: Composition Methods

```typescript
import { z } from 'zod';

const BaseUser = z.object({
  id: z.number(),
  name: z.string(),
  email: z.email()
});

// Add fields
const UserWithRole = BaseUser.extend({
  role: z.enum(['admin', 'user'])
});

// Merge two schemas
const HasID = z.object({ id: z.number() });
const Teacher = BaseUser.merge(HasID);

// Pick specific fields
const JustName = BaseUser.pick({ name: true });

// Omit specific fields
const NoID = BaseUser.omit({ id: true });

// Make all fields optional
const PartialUser = BaseUser.partial();

// Make all fields required
const RequiredUser = PartialUser.required();
```

**Component Breakdown**

| Method | Effect |
|--------|--------|
| `.extend(shape)` | Adds or overwrites fields |
| `.merge(other)` | Equivalent to `A.extend(B.shape)`; B wins on conflicts |
| `.pick(keys)` | Keeps only the specified keys |
| `.omit(keys)` | Removes the specified keys |
| `.partial()` | Makes all fields optional |
| `.deepPartial()` | Makes all nested fields optional |
| `.required()` | Makes all fields required |
| `.passthrough()` | Preserves unknown keys instead of stripping them |

#### Syntax Rules

- `.extend()` returns a new schema; the original is unchanged.
- `.merge()` is not a pure intersection; overlapping keys are overridden by the merged schema.
- `.partial()` is shallow; use `.deepPartial()` for nested optionality.

#### Constraints and Limitations

- `.deepPartial()` only works as expected in hierarchies of objects, arrays, and tuples.
- Conflicting rules for the same key in `.merge()` may produce surprising results.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Create vs Update Schemas from a Base

```typescript
import { z } from 'zod';

// Base schema: all fields optional by default
const BaseUser = z.object({
  name: z.string().min(1),
  email: z.email(),
  password: z.string().min(8)
});

// Create: all fields required
const CreateUser = BaseUser.required();

// Update: all fields optional
const UpdateUser = BaseUser.partial();

// Response: omit password
const UserResponse = BaseUser.omit({ password: true });

console.log(CreateUser.safeParse({}).success); // false
console.log(UpdateUser.safeParse({}).success); // true
console.log(UserResponse.parse({ name: 'A', email: 'a@b.com', password: 'secret123' }));
// { name: 'A', email: 'a@b.com' }
```

**Expected Output**

```
false
true
{ name: 'A', email: 'a@b.com' }
```

**Why This Output Occurs**

- `.required()` enforces all fields, `.partial()` makes them optional, and `.omit()` removes the password from the response shape.

---

### Real-World Cases

- **CRUD APIs** — Base schema with `required()` for create and `partial()` for update.
- **Response serialization** — Omitting sensitive fields like passwords and tokens.
- **Multi-tier resources** — Extending a base entity schema with role-specific fields.

---

## Core Concept 5: Custom Validators

### Definitions

**Core Definition**
Custom validators are user-defined validation functions attached to a schema via `.refine()` or `.superRefine()`, and transformations attached via `.transform()` or `.pipe()`, enabling logic beyond Zod's built-in rules.

**Technical Definition**
`.refine(check, params?)` adds a custom validation that returns a truthy value to pass, or `false` to fail. It can also throw or use a type predicate. `.superRefine((val, ctx) => ...)` is the more versatile form; it receives a context object with `ctx.addIssue()` for registering one or more issues, and supports any `ZodIssueCode`. Refinements can be asynchronous, in which case `.parseAsync()` must be used. `.transform(fn)` changes the value after validation; transforms and refinements can be interleaved. `.pipe(schema)` chains a schema after a transform.

**Beginner-Friendly Explanation**
Custom validators let you add your own rules. Use `.refine()` for a simple yes/no check (like “passwords must match”). Use `.superRefine()` when you need to report multiple issues or control the error codes. Use `.transform()` to change the data after validation (like trimming a string or converting a string to a number).

---

### Purposes

- To enforce application-specific rules not covered by built-in validators.
- To perform cross-field validation (e.g., password confirmation).
- To report multiple issues from a single validation pass with `.superRefine()`.
- To transform data after validation with `.transform()` and `.pipe()`.

---

### Syntax Rules and Structure

#### Complete General Syntax: `.refine()`, `.superRefine()`, `.transform()`

```typescript
import { z } from 'zod';

// Simple refinement
const passwordForm = z.object({
  password: z.string(),
  confirm: z.string()
}).refine((data) => data.password === data.confirm, {
  message: "Passwords don't match",
  path: ["confirm"] // path of error
});

// Multiple issues with superRefine
const Strings = z.array(z.string()).superRefine((val, ctx) => {
  if (val.length > 3) {
    ctx.addIssue({
      code: z.ZodIssueCode.too_big,
      maximum: 3,
      type: "array",
      inclusive: true,
      message: "Too many items"
    });
  }
  if (val.length !== new Set(val).size) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: "No duplicates allowed."
    });
  }
});

// Transform after validation
const stringToNumber = z.string().transform((val) => Number(val));
```

**Component Breakdown**

| Method | Purpose |
|--------|---------|
| `.refine(fn, params?)` | Simple custom validation; returns truthy to pass |
| `.superRefine(fn)` | Advanced validation; can add multiple issues with `ctx.addIssue()` |
| `.transform(fn)` | Changes the value after validation |
| `.pipe(schema)` | Chains another schema after a transform |

#### Syntax Rules

- `.refine()` should return a truthy value for valid data; returning `false` or a falsy value fails validation.
- `.superRefine()` must use `ctx.addIssue()` to report failures; returning `false` is not sufficient.
- Async refinements require `.parseAsync()`.
- Use the `fatal: true` flag with `return z.NEVER` to abort early inside `.superRefine()`.

#### Constraints and Limitations

- `.refine()` can only report one issue per call; use `.superRefine()` for multiple issues.
- Transformations change the output type; use `z.output` to see the post-transform type.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Password Confirmation with `.refine()`

```typescript
import { z } from 'zod';

const passwordForm = z.object({
  password: z.string().min(8),
  confirm: z.string()
}).refine((data) => data.password === data.confirm, {
  message: "Passwords don't match",
  path: ["confirm"]
});

const result = passwordForm.safeParse({
  password: 'secret123',
  confirm: 'different'
});

console.log(result.success); // false
console.log(result.error.issues[0]);
// { code: 'custom', path: ['confirm'], message: "Passwords don't match" }
```

**Expected Output**

```
false
{ code: 'custom', path: ['confirm'], message: "Passwords don't match" }
```

**Why This Output Occurs**

- The refinement compares the two fields and attaches the error to the `confirm` path.

---

#### Example 2: Multiple Issues with `.superRefine()`

```typescript
const schema = z.array(z.string()).superRefine((val, ctx) => {
  if (val.length > 3) {
    ctx.addIssue({ code: z.ZodIssueCode.too_big, maximum: 3, type: 'array', inclusive: true, message: 'Too many items' });
  }
  if (val.length !== new Set(val).size) {
    ctx.addIssue({ code: z.ZodIssueCode.custom, message: 'No duplicates allowed.' });
  }
});

const result = schema.safeParse(['a', 'a', 'b', 'c', 'd']);
console.log(result.error.issues.length); // 2
console.log(result.error.issues.map(i => i.message));
// ['Too many items', 'No duplicates allowed.']
```

**Expected Output**

```
2
['Too many items', 'No duplicates allowed.']
```

**Why This Output Occurs**

- Both conditions fail, so `.superRefine()` registers two issues in a single pass.

---

### Real-World Cases

- **Password confirmation** — Cross-field matching with `.refine()`.
- **Array uniqueness and limits** — Multiple constraints with `.superRefine()`.
- **Data normalization** — Trimming, parsing, or converting values with `.transform()`.

---

## Core Concept 6: Error Messages

### Definitions

**Core Definition**
Zod error messages are the human-readable strings attached to validation issues, customizable per schema, per parse, or globally through error maps.

**Technical Definition**
Validation errors are surfaced as instances of `$ZodError`, containing an `.issues` array. Each issue has a `message` property and structured metadata (e.g., `code`, `path`, `expected`, `received`). In Zod 4, virtually every API accepts an optional `error` parameter, which can be a string or an error map function. Schema-level messages have the highest precedence, followed by per-parse error maps, then global error maps. Zod provides built-in locales for internationalization, and errors can be formatted with `z.flattenError()` (flat object for forms), `z.treeifyError()` (nested structure), or `z.prettifyError()` (human-readable string).

**Beginner-Friendly Explanation**
Zod error messages tell you what went wrong. You can customize them for each field, for each parse call, or globally. You can also format errors differently depending on whether you need a flat list for a form, a nested tree for debugging, or a pretty string for logs.

---

### Purposes

- To provide clear, user-friendly feedback for invalid input.
- To customize messages per field, per parse, or globally.
- To format errors for different consumers (forms, logs, APIs).
- To support internationalization through built-in locales.

---

### Syntax Rules and Structure

#### Complete General Syntax: Error Customization and Formatting

```typescript
import { z } from 'zod';

// Per-field custom message
const schema = z.object({
  email: z.email({ error: 'Please enter a valid email address' }),
  password: z.string().min(8, { error: 'Password must be at least 8 characters' })
});

// Per-parse error map
const result = schema.safeParse(data, {
  error: (issue) => {
    if (issue.code === 'invalid_type') {
      return 'This field has the wrong type';
    }
    return undefined; // fall back to default
  }
});

// Global error map
z.config({
  customError: (issue) => {
    if (issue.code === 'too_small') {
      return `Must be at least ${issue.minimum} characters`;
    }
  }
});

// Formatting
if (!result.success) {
  const flat = z.flattenError(result.error); // { formErrors, fieldErrors }
  const pretty = z.prettifyError(result.error); // human-readable string
  const tree = z.treeifyError(result.error); // nested structure
}
```

**Component Breakdown**

| Mechanism | Scope | Example |
|-----------|-------|---------|
| `{ error: '...' }` | Single schema/field | `z.string().min(3, { error: 'Too short' })` |
| Per-parse `error` | Single parse call | `.safeParse(data, { error: fn })` |
| `z.config({ customError })` | Global | Applies to all schemas |
| `z.flattenError()` | Formatting | Flat object for forms |
| `z.prettifyError()` | Formatting | Human-readable string |
| `z.treeifyError()` | Formatting | Nested error tree |

#### Syntax Rules

- In Zod 4, use `{ error: '...' }` instead of `{ message: '...' }`; the `message` parameter is deprecated.
- Error maps receive an `issue` object; return `undefined` to fall back to the next error map in the precedence chain.
- Global error maps have lower precedence than schema-level and per-parse messages.

#### Constraints and Limitations

- Zod 3 used separate `message` and `errorMap` parameters; Zod 4 unified them under `error`.
- Global error maps affect all validations; use them carefully in libraries.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Per-Field Custom Messages and Flattened Errors

```typescript
import { z } from 'zod';

const schema = z.object({
  email: z.email({ error: 'Invalid email format' }),
  password: z.string().min(8, { error: 'Password too short' })
});

const result = schema.safeParse({ email: 'invalid', password: '123' });

if (!result.success) {
  console.log(z.flattenError(result.error));
  // {
  //   formErrors: [],
  //   fieldErrors: {
  //     email: ['Invalid email format'],
  //     password: ['Password too short']
  //   }
  // }
}
```

**Expected Output**

```typescript
{
  formErrors: [],
  fieldErrors: {
    email: ['Invalid email format'],
    password: ['Password too short']
  }
}
```

**Why This Output Occurs**

- `z.flattenError()` converts the issue array into a form-friendly structure with field-level error arrays.

---

### Real-World Cases

- **Form validation** — `z.flattenError()` produces field-level errors for UI display.
- **API error responses** — Custom messages and structured issue codes for clients.
- **Internationalization** — Using Zod's built-in locales for translated messages.

---

## Core Concept 7: Zod Best Practices

### Definitions

**Core Definition**
Zod best practices are the design and implementation patterns that keep Zod schemas maintainable, type-safe, performant, and secure across a TypeScript Express application.

**Technical Definition**
Best practices include: defining schemas close to where they are used and extracting them to shared modules when reused; composing schemas with `.extend`, `.merge`, `.pick`, `.omit`, and `.partial` to avoid duplication; deriving all TypeScript types with `z.infer` instead of manually duplicating types; using `.safeParse()` in Express middleware to avoid throwing; using `z.coerce` or `.transform()` for query parameters and form data; stripping or rejecting unknown keys to prevent mass assignment; and structuring schemas by domain for maintainability.

**Beginner-Friendly Explanation**
Best practices are the habits that keep Zod schemas clean and safe. Define schemas once and reuse them, derive types automatically, use `safeParse` in Express so errors become `400` responses, and coerce types for query strings.

---

### Purposes

- To keep schemas DRY through composition and reuse.
- To ensure that runtime validation and TypeScript types never drift apart.
- To integrate validation safely into Express without crashing on invalid input.
- To prevent mass assignment and handle query string coercion correctly.

---

### Syntax Rules and Structure

#### Complete General Syntax: Express Middleware Best Practice

```typescript
import { z } from 'zod';
import { Request, Response, NextFunction } from 'express';

function validateBody<T extends z.ZodTypeAny>(schema: T) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({
        error: 'Validation failed',
        details: z.flattenError(result.error).fieldErrors
      });
    }
    req.body = result.data; // typed as z.infer<T>
    next();
  };
}

const createUserSchema = z.object({
  username: z.string().min(3).max(20),
  email: z.email(),
  password: z.string().min(8),
  age: z.number().int().positive().optional()
});

app.post('/users', validateBody(createUserSchema), (req, res) => {
  // req.body is now typed as z.infer<typeof createUserSchema>
  res.status(201).json({ user: req.body });
});
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Safe parsing | `.safeParse()` instead of `.parse()` |
| Typed body | `req.body = result.data` |
| Structured errors | `z.flattenError(result.error)` |
| Reusable middleware | `validateBody(schema)` factory |

#### Syntax Rules

- **Use `.safeParse()` in Express** — `.parse()` throws and can crash handlers if not caught.
- **Use `z.coerce` for query parameters** — query strings are always strings; `z.coerce.number()` converts them.
- **Derive types with `z.infer`** — never duplicate types manually.
- **Compose schemas** — define base schemas and derive variants with `.extend`, `.partial`, and `.omit`.
- **Strip unknown keys** — `z.object()` strips by default; use `.strict()` to reject them explicitly.
- **Organize by domain** — modularize schemas by feature and extract shared schemas to a `schemas/` folder.

#### Constraints and Limitations

- `z.coerce.boolean()` uses truthiness; `"false"` becomes `true`. Use `.transform()` or `z.pipe()` for precise boolean parsing.
- Global error maps affect all schemas; avoid them in libraries.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Express Validation Pipeline

```typescript
import express from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json());

// Base schema
const BaseUser = z.object({
  username: z.string().min(3).max(20),
  email: z.email(),
  password: z.string().min(8)
});

// Create: all required
const CreateUser = BaseUser;

// Update: all optional
const UpdateUser = BaseUser.partial();

// Response: omit password
const UserResponse = BaseUser.omit({ password: true });

// Validation middleware
function validate<T extends z.ZodTypeAny>(schema: T) {
  return (req: express.Request, res: express.Response, next: express.NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({
        error: 'Validation failed',
        details: z.flattenError(result.error).fieldErrors
      });
    }
    req.body = result.data;
    next();
  };
}

app.post('/users', validate(CreateUser), (req, res) => {
  const user = UserResponse.parse(req.body); // strip password for response
  res.status(201).json({ user });
});

app.patch('/users/:id', validate(UpdateUser), (req, res) => {
  res.json({ updated: req.body });
});
```

**Expected Output**

```
POST /users  { "username": "alice", "email": "alice@example.com", "password": "secret123" }
→ 201 { "user": { "username": "alice", "email": "alice@example.com" } }

POST /users  { "username": "a", "email": "invalid", "password": "123" }
→ 400 {
  "error": "Validation failed",
  "details": {
    "username": ["Too small: expected string to have >=3 characters"],
    "email": ["Invalid email address"],
    "password": ["Too small: expected string to have >=8 characters"]
  }
}

PATCH /users/1  { "username": "alice2" }
→ 200 { "updated": { "username": "alice2" } }
```

**Why This Output Occurs**

- The `validate` middleware uses `.safeParse()` and returns `400` with flattened field errors on failure. The response schema strips the password before sending the user object.

---

### Real-World Cases

- **Production APIs** — Reusable middleware, composed schemas, and safe parsing.
- **Full-stack TypeScript** — Sharing `z.infer` types between React and Express.
- **Query parameter validation** — Using `z.coerce` to convert strings to numbers.

---

## References

- Zod Official Documentation — https://zod.dev/
- Zod Full Documentation (llms-full.txt) — https://zod.dev/llms-full.txt
- Type Inference — Zod — https://mintlify.wiki/colinhacks/zod/concepts/type-inference
- Customizing Errors — Zod — https://zod.dev/error-customization
- Formatting Errors — Zod — https://zod.dev/error-formatting
- Schema Composition (`.extend`, `.merge`, `.pick`, `.omit`, `.partial`) — GitHub — https://github.com/cameronraysmith/zod
- Refinements and `.superRefine()` — Zod — https://mintlify.wiki/colinhacks/zod/concepts/refinements
- Using Zod with Express — Steve Kinney — https://stevekinney.com/courses/full-stack-typescript/using-zod-with-express
- Structuring Zod Schemas Efficiently — Steve Kinney — https://stevekinney.com/courses/full-stack-typescript/structuring-zod-schemas-efficiently
- Zod v4 Migration Guide (error parameter unification) — https://zod.dev/v4
- express-zod-api — https://www.npmjs.com/package/express-zod-api
- zod-express-validator — https://www.npmjs.com/package/zod-express-validator