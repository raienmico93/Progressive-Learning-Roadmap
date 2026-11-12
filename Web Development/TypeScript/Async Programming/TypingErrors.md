# Typing Errors in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Typing errors** in TypeScript is the practice of applying **static types to thrown and caught errors**, ensuring that error handling code operates on well-understood error shapes rather than untyped values. Because JavaScript allows **any value** to be thrown—not just `Error` instances—TypeScript provides mechanisms to model, narrow, and handle error types safely across `try`/`catch` boundaries, custom error class hierarchies, and asynchronous rejection paths.

**Technical Definition.** In TypeScript, the `throw` statement accepts values of **any type**, but the type system cannot statically guarantee what will be thrown. Since **TypeScript 4.4**, the `useUnknownInCatchVariables` compiler flag (enabled by `strict`) types `catch` clause variables as **`unknown`** rather than `any`, forcing explicit narrowing before use. Custom error classes extend the built-in `Error` type, adding typed properties and discriminating fields. **Error causes** (`Error.cause`, ES2022) provide typed chaining. There is **no native typed-rejection mechanism** for promises; rejection reasons are typed `any` at the type level, so discipline and narrowing are essential.

**Beginner-Friendly Explanation.** In JavaScript, you can throw **anything**—an `Error`, a string, a number, even `null`. TypeScript cannot know in advance what a `try`/`catch` block will catch. To keep you safe, TypeScript's strict mode types caught errors as **`unknown`**, which forces you to **check what the error is** before using it. You can create **custom error classes** with typed properties, and you can **narrow** the `unknown` error to those classes with `instanceof` checks, so error handling becomes as type-safe as the rest of your code.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **`unknown` catch variables** | Under `strict`, `catch (e)` types `e` as `unknown`, requiring narrowing. |
| **Any value can be thrown** | JavaScript permits throwing strings, numbers, and `null`, not just `Error` instances. |
| **Custom error classes** | Extending `Error` lets you attach typed properties and discriminate error kinds. |
| **`Error.cause`** | ES2022 chaining provides typed root-cause information. |
| **No typed rejections** | Promise rejections are typed `any`; narrowing is required at `await`. |
| **`instanceof` narrowing** | The standard way to refine `unknown` errors to specific error classes. |
| **`never` for `throw`** | Functions that always throw return `never`, aiding exhaustiveness checks. |
| **Result types** | Discriminated unions (`{ ok: true; data } | { ok: false; error }`) are a typed alternative to exceptions. |

---

### Prerequisites

- **TypeScript basics**: type annotations, classes, and inheritance.
- **`try`/`catch`/`finally`** syntax.
- **The `unknown` type** and type narrowing.
- **`instanceof`** and custom error classes.
- **The `strict` compiler flag** (specifically `useUnknownInCatchVariables`).
- **A TypeScript environment** (4.4+ for `unknown` catch variables; 4.5+ for `Error.cause` typing).

---

### Related Programming Areas

- **Error handling** — `try`/`catch`, propagation, and recovery.
- **Type narrowing** — `instanceof`, discriminated unions, and user-defined guards.
- **Custom error hierarchies** — domain-specific error classes.
- **Async error handling** — rejection narrowing with `await` and `catch`.
- **Result types** — typed alternatives to exceptions for expected failures.
- **Never-throwing functions** — `assertNever` and exhaustiveness checks.

---

## Core Concepts / Features

---

## 1. Typing Errors

### Definitions

**Core Definition.** **Typing errors** is the practice of assigning, narrowing, and handling **types for thrown and caught values** in TypeScript, ensuring error handling code is type-safe.

**Technical Definition.** JavaScript's `throw` accepts any value, so TypeScript models catch variables as **`unknown`** under `strict` (since TS 4.4). Narrowing with `instanceof`, `typeof`, or custom type guards refines `unknown` to specific error types. Custom error classes extend `Error` and add typed properties. Async rejections are typed `any` at the promise level; `await` in a `try` block converts them to `unknown` catch variables. The `Error.cause` property provides typed chaining for wrapping errors.

**Beginner-Friendly Explanation.** Typing errors means **being explicit about what errors can occur and what shape they have**. You create custom error classes, throw them, and narrow caught errors to those classes before using their properties.

### Purposes

- **To prevent unsafe property access** on caught errors.
- **To discriminate between error kinds** for different handling paths.
- **To attach typed metadata** (status codes, field names, request IDs) to errors.
- **To chain errors** with typed causes for root-cause analysis.
- **To ensure exhaustive error handling** with `never` and discriminated unions.
- **To document the error contract** of functions and APIs.

### Syntax Rules and Structure

**Basic `try`/`catch` with `unknown`:**

```typescript
try {
  await fetchUser(id);
} catch (error: unknown) {
  if (error instanceof Error) {
    console.error(error.message);
  }
}
```

**Custom error class:**

```typescript
class ValidationError extends Error {
  constructor(public field: string, message: string) {
    super(message);
    this.name = "ValidationError";
  }
}
```

**Narrowing with `instanceof`:**

```typescript
catch (error: unknown) {
  if (error instanceof ValidationError) {
    console.error(error.field, error.message);
  } else if (error instanceof Error) {
    console.error(error.message);
  }
}
```

**Component breakdown:**

- `catch (error: unknown)` — the catch variable is `unknown` under `strict`.
- `error instanceof Error` — narrows to the base `Error` type.
- `error instanceof ValidationError` — narrows to the custom type with `field` and `message`.
- `error.message` — accessible only after narrowing.

**Syntax rules.**

- `throw` accepts any value; TypeScript does not restrict the thrown type.
- Catch variables are `unknown` under `strict`; explicit annotations of `any` are possible but discouraged.
- Narrow with `instanceof Error` before accessing `.message` or `.stack`.
- Custom error classes should set `this.name` and, if targeting ES5, fix the prototype chain.
- `Error.cause` is available in ES2022 targets; TypeScript types it when the `lib` includes `ES2022.Error`.

**Constraints and limitations.**

- TypeScript cannot type **rejection reasons** at the promise level; they are `any`.
- Non-`Error` thrown values are common; always handle the `unknown` fallback.
- Custom error classes with additional properties may not serialize correctly across some boundaries (e.g., workers).

### Annotated Complete Step by Step Code Examples

#### Example 1: Narrowing `unknown` catch variables

```typescript
// example_typing_errors_basic.ts
class ValidationError extends Error {
  constructor(public field: string, message: string) {
    super(message);
    this.name = "ValidationError";
  }
}

function validate(value: string): void {
  if (value.length < 3) {
    throw new ValidationError("value", "Too short");
  }
}

try {
  validate("ab");
} catch (error: unknown) {
  if (error instanceof ValidationError) {
    console.error(`Invalid ${error.field}: ${error.message}`);
    // Expected: Invalid value: Too short
  } else if (error instanceof Error) {
    console.error(error.message);
  } else {
    console.error("Unknown error:", error);
  }
}
```

**Setup.** Compile with `npx tsc example_typing_errors_basic.ts --strict`.

**Expected Output:**
```
Invalid value: Too short
```

**Why this output occurs.** The catch variable is `unknown`. The `instanceof ValidationError` check narrows it to the custom error type, giving access to `field` and `message`. The `instanceof Error` fallback handles other errors, and the final `else` handles non-`Error` thrown values.

---

#### Example 2: Non-`Error` thrown values

```typescript
// example_typing_errors_non_error.ts
function risky(value: number): void {
  if (value === 0) throw "zero is not allowed";
  if (value < 0) throw 42;
}

try {
  risky(0);
} catch (error: unknown) {
  if (error instanceof Error) {
    console.error(error.message);
  } else if (typeof error === "string") {
    console.error(`String error: ${error}`); // Expected: String error: zero is not allowed
  } else {
    console.error(`Non-error thrown: ${String(error)}`);
  }
}
```

**Expected Output:**
```
String error: zero is not allowed
```

**Why this output occurs.** The `typeof error === "string"` check narrows the `unknown` error to `string`, demonstrating that not all thrown values are `Error` instances.

---

#### Example 3: Error causes

```typescript
// example_typing_errors_cause.ts
class ApiError extends Error {
  constructor(message: string, public statusCode: number, options?: ErrorOptions) {
    super(message, options);
    this.name = "ApiError";
  }
}

function fetchData(): never {
  try {
    throw new Error("Network failure");
  } catch (error: unknown) {
    throw new ApiError("API request failed", 500, { cause: error });
  }
}

try {
  fetchData();
} catch (error: unknown) {
  if (error instanceof ApiError) {
    console.error(error.statusCode); // Expected: 500
    if (error.cause instanceof Error) {
      console.error(error.cause.message); // Expected: Network failure
    }
  }
}
```

**Expected Output:**
```
500
Network failure
```

**Why this output occurs.** The `cause` option attaches the original error to the `ApiError`. TypeScript types `error.cause` as `unknown` when targeting ES2022, requiring narrowing before accessing `.message`.

---

### Real-World Cases

- **API clients**: custom `ApiError` with status codes and request IDs.
- **Validation**: `ValidationError` with field names and messages.
- **Infrastructure**: `DatabaseError`, `TimeoutError`, and `NetworkError` classes.
- **Error chaining**: wrapping low-level errors with domain-level causes.

---

## 2. `try`/`catch`

### Definitions

**Core Definition.** **`try`/`catch`** is the JavaScript statement for handling exceptions, where `try` contains code that may throw and `catch` handles the thrown value.

**Technical Definition.** In TypeScript, `try`/`catch` blocks participate in **control-flow analysis**. The `catch` clause variable is typed `unknown` under `strict`. `finally` runs regardless of outcome and does not affect the caught type. The `throw` statement has type `never`, which makes functions that always throw return `never` and aids exhaustiveness checking.

**Beginner-Friendly Explanation.** `try`/`catch` is **how you handle errors**. You put risky code in `try`; if it throws, the error is caught in `catch`, where you can check what it is and respond.

### Purposes

- **To catch synchronous and asynchronous errors** at the appropriate level.
- **To recover from expected failures** and continue execution.
- **To centralize error handling** in a single place.
- **To guarantee cleanup** with `finally`.
- **To narrow error types** for safe property access.

### Syntax Rules and Structure

**Basic form:**

```typescript
try {
  // risky code
} catch (error: unknown) {
  // handle error
} finally {
  // cleanup
}
```

**Rethrowing:**

```typescript
try {
  await save();
} catch (error: unknown) {
  logger.error(error);
  throw error; // rethrow
}
```

**Syntax rules.**

- The `catch` clause variable is `unknown` under `strict`.
- The `finally` clause runs regardless of whether `try` succeeded or threw.
- The `throw` statement has type `never`.
- `catch` without a variable (`catch {}`) is allowed and ignores the error.

**Constraints and limitations.**

- `try`/`catch` adds runtime overhead and should be scoped narrowly.
- Errors thrown in `finally` override earlier errors.
- `try`/`catch` does not catch promise rejections unless the promise is awaited.

### Annotated Complete Step by Step Code Examples

#### Example 1: `try`/`catch`/`finally`

```typescript
// example_try_catch_finally.ts
function parseJSON(input: string): unknown {
  return JSON.parse(input);
}

function safeParse(input: string): unknown {
  try {
    return parseJSON(input);
  } catch (error: unknown) {
    if (error instanceof SyntaxError) {
      console.error("Invalid JSON:", error.message);
      // Expected: Invalid JSON: Unexpected token ...
    }
    return null;
  } finally {
    console.log("parse attempt finished"); // Expected: parse attempt finished
  }
}

safeParse("{invalid}");
```

**Expected Output (approximate):**
```
Invalid JSON: Unexpected token ...
parse attempt finished
```

**Why this output occurs.** The `catch` narrows the `unknown` error to `SyntaxError` and logs its message. The `finally` block runs regardless of whether parsing succeeded.

---

#### Example 2: Rethrowing with typed errors

```typescript
// example_try_catch_rethrow.ts
class AppError extends Error {
  constructor(message: string, public code: string) {
    super(message);
    this.name = "AppError";
  }
}

async function loadUser(id: string): Promise<{ id: string; name: string }> {
  if (id === "missing") throw new AppError("User not found", "USER_NOT_FOUND");
  return { id, name: "Ada" };
}

async function main() {
  try {
    await loadUser("missing");
  } catch (error: unknown) {
    if (error instanceof AppError) {
      console.error(error.code); // Expected: USER_NOT_FOUND
    }
    throw error; // rethrow
  }
}

main().catch((error: unknown) => {
  if (error instanceof Error) console.error("Final:", error.message);
  // Expected: Final: User not found
});
```

**Expected Output:**
```
USER_NOT_FOUND
Final: User not found
```

**Why this output occurs.** The inner `catch` narrows the error to `AppError` and logs its `code`, then rethrows. The outer `.catch` catches the rethrown error and logs its message.

---

### Real-World Cases

- **API clients**: catching `ApiError` and showing user-friendly messages.
- **Parsing**: catching `SyntaxError` from `JSON.parse`.
- **Database transactions**: `finally` blocks for rollback or cleanup.
- **Middleware**: centralized error handling in Express or Fastify.

---

## 3. Error Types

### Definitions

**Core Definition.** **Error types** are the classes and interfaces that model thrown errors, including the built-in `Error` hierarchy and custom error classes with domain-specific properties.

**Technical Definition.** The built-in `Error` class provides `message`, `name`, `stack`, and (in ES2022) `cause`. Custom error classes extend `Error` and add typed properties. TypeScript narrows `unknown` errors to these classes using `instanceof`. Discriminated error unions (`{ kind: "notFound" } | { kind: "validation"; field: string }`) provide an alternative to class hierarchies for **result-based error handling**.

**Beginner-Friendly Explanation.** Error types are **the kinds of errors your code can throw**. You create custom classes for different error categories—validation, network, not-found—so you can tell them apart and handle each appropriately.

### Purposes

- **To distinguish between error categories** for targeted handling.
- **To attach typed metadata** (status codes, field names, retry flags).
- **To document the error contract** of a function.
- **To enable exhaustive error handling** with discriminated unions.
- **To chain errors** with typed causes.

### Syntax Rules and Structure

**Custom error class:**

```typescript
class NotFoundError extends Error {
  readonly kind = "notFound";
  constructor(public resource: string, id: string) {
    super(`${resource} ${id} not found`);
    this.name = "NotFoundError";
  }
}
```

**Discriminated error union:**

```typescript
type AppError =
  | { kind: "notFound"; resource: string }
  | { kind: "validation"; field: string; message: string }
  | { kind: "unknown"; error: Error };
```

**Narrowing:**

```typescript
if (error instanceof NotFoundError) { ... }
else if (error instanceof Error) { ... }
```

**Syntax rules.**

- Custom error classes should set `this.name` and, if targeting ES5, fix `Object.setPrototypeOf(this, new.target.prototype)`.
- Discriminated error unions use a `kind` discriminant for narrowing.
- `Error.cause` is typed `unknown` and requires narrowing.

**Constraints and limitations.**

- Class-based errors do not serialize well across worker or network boundaries.
- Discriminated unions require converting caught errors into result values.

### Annotated Complete Step by Step Code Examples

#### Example 1: Custom error hierarchy

```typescript
// example_error_types_hierarchy.ts
abstract class DomainError extends Error {
  abstract readonly kind: string;
  constructor(message: string) {
    super(message);
    this.name = new.target.name;
  }
}

class NotFoundError extends DomainError {
  readonly kind = "notFound";
  constructor(public resource: string, id: string) {
    super(`${resource} ${id} not found`);
  }
}

class ValidationError extends DomainError {
  readonly kind = "validation";
  constructor(public field: string, message: string) {
    super(message);
  }
}

function handle(error: unknown): string {
  if (error instanceof NotFoundError) return `Missing ${error.resource}`;
  if (error instanceof ValidationError) return `Invalid ${error.field}`;
  if (error instanceof DomainError) return error.message;
  return "Unknown error";
}

console.log(handle(new NotFoundError("User", "u1"))); // Expected: Missing User
console.log(handle(new ValidationError("email", "Invalid"))); // Expected: Invalid email
```

**Expected Output:**
```
Missing User
Invalid email
```

**Why this output occurs.** The `instanceof` checks narrow the `unknown` error to each custom class, enabling access to `resource` and `field`. The hierarchy provides a base `DomainError` for shared behavior.

---

#### Example 2: Discriminated error union (result type)

```typescript
// example_error_types_result.ts
type Result<T, E> =
  | { ok: true; value: T }
  | { ok: false; error: E };

type FetchError =
  | { kind: "notFound"; id: string }
  | { kind: "network"; retryable: boolean };

async function fetchUser(id: string): Promise<Result<{ name: string }, FetchError>> {
  if (id === "missing") return { ok: false, error: { kind: "notFound", id } };
  return { ok: true, value: { name: "Ada" } };
}

async function main() {
  const result = await fetchUser("missing");
  if (!result.ok) {
    switch (result.error.kind) {
      case "notFound":
        console.error(`User ${result.error.id} not found`); // Expected: User missing not found
        break;
      case "network":
        console.error(`Network error, retryable: ${result.error.retryable}`);
        break;
    }
  }
}

main();
```

**Expected Output:**
```
User missing not found
```

**Why this output occurs.** The `Result` type makes errors explicit in the return type, and the `kind` discriminant enables exhaustive narrowing. This avoids `try`/`catch` for expected failures.

---

### Real-World Cases

- **API clients**: `ApiError` with status codes and retry flags.
- **Validation**: `ValidationError` with field-level details.
- **Domain logic**: `Result<T, E>` for expected failures like not-found.
- **Infrastructure**: `DatabaseError`, `TimeoutError`, `RateLimitError`.

---

## 4. Error Handling Best Practices

### Definitions

**Core Definition.** **Error handling best practices** are guidelines for typing and handling errors in TypeScript in ways that maximize **safety, clarity, and maintainability**, while avoiding unsafe casts, swallowed errors, and unhandled rejections.

**Technical Definition.** Best practices center on **narrowing `unknown` catch variables**, **using custom error classes with typed metadata**, **always handling the non-`Error` case**, **rethrowing after logging**, **using result types for expected failures**, **enabling `useUnknownInCatchVariables`**, and **avoiding empty `catch` blocks**. They also include **`assertNever` for exhaustive error handling** and **typed causes** for error chaining.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep error handling safe and clear. The big rules are: always narrow `unknown` errors before using them; handle non-`Error` thrown values; do not swallow errors silently; and use result types for expected failures instead of exceptions.

### Purposes

- **To prevent unsafe property access** on caught errors.
- **To ensure all error kinds are handled** exhaustively.
- **To preserve error context** with causes and rethrowing.
- **To distinguish expected failures** from unexpected exceptions.
- **To keep error handling readable** and centralized.

### Syntax Rules and Structure

**Best-practice rules:**

1. **Narrow `unknown` catch variables.** Use `instanceof Error` or custom classes before accessing properties.
2. **Handle non-`Error` thrown values.** JavaScript allows throwing anything; always include an `else` branch.
3. **Use custom error classes with typed metadata.** Extend `Error` and add discriminating properties.
4. **Rethrow after logging** when the error cannot be handled locally. Do not swallow errors silently.
5. **Use result types for expected failures.** Discriminated unions (`Result<T, E>`) make expected errors explicit in the type system.
6. **Use `assertNever` for exhaustive error handling.** Ensure all error kinds are handled in `switch` statements.
7. **Use `Error.cause` for chaining.** Attach root causes when wrapping errors.
8. **Avoid empty `catch` blocks.** If you intentionally ignore an error, add a comment explaining why.
9. **Enable `useUnknownInCatchVariables`** (part of `strict`) to enforce narrowing.

**Component breakdown (error handling decision tree):**

```
Is the failure expected or unexpected?
  ├── Expected → Use a result type (`Result<T, E>`) with discriminated errors.
  └── Unexpected → Use `try`/`catch` with `unknown` narrowing.
                    ├── Can you recover? → Handle and continue.
                    └── Cannot recover? → Log, attach cause, and rethrow.
```

**Constraints and limitations.**

- Result types require discipline to check `ok` at every call site.
- Custom error classes add runtime overhead for class creation.
- `Error.cause` requires ES2022 target for full typing.

### Annotated Complete Step by Step Code Examples

#### Example 1: Narrowing `unknown` and handling non-`Error` values

```typescript
// example_best_narrowing.ts
class ApiError extends Error {
  constructor(message: string, public status: number) {
    super(message);
    this.name = "ApiError";
  }
}

function handle(error: unknown): string {
  if (error instanceof ApiError) return `API ${error.status}: ${error.message}`;
  if (error instanceof Error) return `Error: ${error.message}`;
  return `Non-error thrown: ${String(error)}`;
}

console.log(handle(new ApiError("Not Found", 404))); // Expected: API 404: Not Found
console.log(handle("oops")); // Expected: Non-error thrown: oops
```

**Expected Output:**
```
API 404: Not Found
Non-error thrown: oops
```

**Why this output occurs.** The function narrows the `unknown` error to `ApiError`, then `Error`, then falls back to `String(error)` for non-`Error` values.

---

#### Example 2: Result type for expected failures

```typescript
// example_best_result.ts
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E };

type ParseError = { kind: "syntax"; message: string } | { kind: "empty" };

function parseNumber(input: string): Result<number, ParseError> {
  if (input.trim() === "") return { ok: false, error: { kind: "empty" } };
  const n = Number(input);
  if (Number.isNaN(n)) return { ok: false, error: { kind: "syntax", message: "Not a number" } };
  return { ok: true, value: n };
}

const result = parseNumber("");
if (!result.ok) {
  switch (result.error.kind) {
    case "empty":
      console.log("Input is empty"); // Expected: Input is empty
      break;
    case "syntax":
      console.log(result.error.message);
      break;
  }
}
```

**Expected Output:**
```
Input is empty
```

**Why this output occurs.** The `Result` type makes the failure explicit in the return type, and the `kind` discriminant enables exhaustive handling without `try`/`catch`.

---

#### Example 3: Rethrowing with causes

```typescript
// example_best_rethrow.ts
class AppError extends Error {
  constructor(message: string, options?: ErrorOptions) {
    super(message, options);
    this.name = "AppError";
  }
}

async function fetchData(): Promise<string> {
  try {
    throw new Error("Network failure");
  } catch (error: unknown) {
    throw new AppError("Failed to fetch data", { cause: error });
  }
}

fetchData().catch((error: unknown) => {
  if (error instanceof AppError) {
    console.error(error.message); // Expected: Failed to fetch data
    if (error.cause instanceof Error) {
      console.error("Caused by:", error.cause.message); // Expected: Caused by: Network failure
    }
  }
});
```

**Expected Output:**
```
Failed to fetch data
Caused by: Network failure
```

**Why this output occurs.** The inner `catch` wraps the original error with `cause`, preserving the root cause. The outer `catch` narrows the `AppError` and its `cause` before accessing messages.

---

### Real-World Cases

- **API clients**: narrowing `ApiError` for status-specific handling.
- **Validation**: result types for expected field validation failures.
- **Infrastructure**: wrapping low-level errors with domain-level causes.
- **Code review**: rejecting empty `catch` blocks and untyped error property access.

---

## References

- TypeScript 4.4 Release Notes — Defaulting to the `unknown` Type in Catch Variables — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html#defaulting-to-the-unknown-type-in-catch-variables
- TypeScript 4.5 Release Notes — `Error.cause` Typing — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html
- TypeScript Handbook — Narrowing (`instanceof` and `unknown`) — https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- TSConfig Reference — `useUnknownInCatchVariables` — https://www.typescriptlang.org/tsconfig/useUnknownInCatchVariables.html
- MDN — `try...catch` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch
- MDN — `Error` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error
- MDN — `Error.cause` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/cause
- pproenca/dot-skills — Narrow `catch` Bindings Instead of Assuming `Error` — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-refactor/references/narrow-catch-bindings.md
- pproenca/dot-skills — Narrow `unknown` Errors with `instanceof` — https://github.com/pproenca/dot-skills/blob/master/skills/.curated/typescript/references/safety-error-narrowing.md
- pproenca/dot-skills — Model Expected Failures as Result Types — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.curated/typescript/references/pattern-result-types.md
- TypeScript ESLint — `use-unknown-in-catch-callback-variable` Rule — https://typescript-eslint.io/rules/use-unknown-in-catch-callback-variable/
- Total TypeScript — Typing Errors — https://www.totaltypescript.com/books/total-typescript-essentials
- TypeScript ESLint — `no-empty-object-type` Rule (for error shapes) — https://typescript-eslint.io/rules/no-empty-object-type/