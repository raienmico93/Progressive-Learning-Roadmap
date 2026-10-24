# Assertion Functions in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** An **assertion function** is a TypeScript function with an `asserts` return type that **validates a runtime condition** and **narrows the type** of a value or parameter for the remainder of the enclosing scope, provided the function returns normally. If the condition fails, the function **throws an error** rather than returning, making it a runtime-verified contract between the function and the type system. Assertion functions were introduced in **TypeScript 3.7** and are the throwing counterpart to **type predicates** (`value is T`).

**Technical Definition.** An assertion function declares `asserts condition` or `asserts value is T` as its return type. When control flow passes through the assertion call without an exception, TypeScript's **control-flow analysis** applies the asserted narrowing to the remainder of the scope. Unlike type predicates, which return a boolean and must be used in a conditional, assertion functions **return `void` and throw** on failure, eliminating the need for `if` checks or `as` casts at every call site. Assertion functions are **erased** at compile time and emit no JavaScript beyond the function's own runtime check.

**Beginner-Friendly Explanation.** An assertion function is **a runtime check that screams if something is wrong and teaches TypeScript if everything is right**. You call `assertIsString(value)`, and the function actually checks whether `value` is a string. If it is not, the function **throws an error** and your program stops. If it is, the function returns normally, and TypeScript **trusts the result** for the rest of the scope—no `if` statement needed. It is like a bouncer at a club: check the ID at the door, and everyone inside is guaranteed to be of age.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Throws on failure** | If the condition is not met, the assertion function throws an error; it never returns normally. |
| **Narrows on success** | When the function returns without throwing, TypeScript narrows the type for the rest of the scope. |
| **Returns `void`** | Assertion functions do not return values; they either throw or return implicitly. |
| **Two forms** | `asserts condition` validates a boolean expression; `asserts value is T` narrows a value to a specific type. |
| **Eliminates `if` nesting** | Callers do not need to wrap subsequent code in `if` blocks or use `as` casts. |
| **Centralizes validation** | Shared preconditions can be reused across multiple functions. |
| **Compile-time trust** | TypeScript does **not** verify that the runtime check matches the assertion; a lying assertion is a correctness bug. |
| **TypeScript 3.7+** | Assertion functions require TypeScript 3.7 or later. |
| **Erasure** | Assertion signatures are compile-time only; the runtime check remains in the function body. |

---

### Prerequisites

- **TypeScript basics**: functions, parameters, return types, and `void`.
- **Union types** and **type narrowing**.
- **Type predicates** (`value is T`) for comparison.
- **Control flow**: `if`/`else`, `throw`, and early returns.
- **The `unknown` type** and runtime validation.
- **A TypeScript environment** (3.7+; `tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Type narrowing** — the broader mechanism that assertion functions serve.
- **Type predicates** — the boolean-returning counterpart to assertion functions.
- **Type assertions (`as`)** — the unsafe, compile-time-only alternative.
- **Runtime validation** — libraries like Zod and `io-ts` for external data.
- **Invariant checking** — `assertNever` for exhaustiveness and `assertDefined` for nullish checks.

---

## Core Concepts / Features

---

## 1. Assertion Functions

### Definitions

**Core Definition.** An **assertion function** is a function that **validates a condition at runtime and throws if it fails**, narrowing the type of its argument(s) for the remainder of the calling scope when it returns normally.

**Technical Definition.** Assertion functions use the `asserts` keyword in their return type position. They must be declared with an explicit type annotation (no arrow-function expression inference for the signature). They throw when validation fails and return `void` on success. Control-flow analysis treats the assertion as a **guarantee**: if the call completes, the condition holds, and the narrowed type applies to all subsequent statements in the scope.

**Beginner-Friendly Explanation.** An assertion function is **a check-and-throw helper that also narrows types**. It replaces repetitive `if (!x) throw ...` patterns with a single reusable call.

### Purposes

- **To centralize precondition checks** that would otherwise be duplicated across multiple functions.
- **To narrow types without `if` nesting** or `as` casts, keeping callers flat and readable.
- **To validate external data** (API responses, form inputs) before treating it as a trusted type.
- **To enforce invariants** such as non-null, non-empty, or specific-shape conditions.
- **To combine runtime validation with compile-time narrowing** in a single, reusable function.
- **To improve intent clarity** compared to scattered `if`-throw patterns.

### Syntax Rules and Structure

**General syntax (`asserts value is T`):**

```typescript
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Expected a string");
  }
}
```

**General syntax (`asserts condition`):**

```typescript
function assert(condition: unknown, message?: string): asserts condition {
  if (!condition) {
    throw new Error(message ?? "Assertion failed");
  }
}
```

**Component breakdown:**

- `function assertIsString(value: unknown)` — the assertion function declaration; the parameter is typically typed `unknown`.
- `: asserts value is string` — the **assertion signature**: if the function returns, `value` is a `string`.
- `throw new Error(...)` — the failure path; assertion functions must throw when the condition fails.
- Implicit return (or `return;`) — the success path; no value is returned.

**Syntax rules.**

- Assertion functions must return `void` (explicitly or implicitly); they cannot return values.
- The `asserts` keyword must appear in the return type annotation.
- The function must throw when the condition fails.
- The parameter being asserted must be a **parameter** of the function, not a property or expression.
- `asserts condition` asserts a boolean expression; `asserts value is T` narrows a value.

**Constraints and limitations.**

- Assertion functions **cannot be arrow functions** without an explicit assertion signature on a variable.
- TypeScript does **not verify** that the runtime check matches the assertion; a lying assertion is a silent bug.
- Assertion functions cannot narrow `this` properties.
- **Unions of assertion functions do not act as assertion functions** (TypeScript issue #59707).
- The asserted parameter must be a **binding** (variable, parameter, or property); you cannot assert arbitrary expressions.

### Annotated Complete Step by Step Code Examples

#### Example 1: `asserts value is T` — string validation

```typescript
// example_assert_string.ts
// Step 1: Declare the assertion function.
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error(`Expected string, got ${typeof value}`);
  }
}

// Step 2: Use it to narrow an unknown value.
function processUsername(input: unknown) {
  assertIsString(input); // Throws if input is not a string.
  // Step 3: After the call, input is narrowed to string.
  return input.trim().toLowerCase();
}

console.log(processUsername("JohnDoe")); // Expected: johndoe
// processUsername(123); // Throws: Expected string, got number
```

**Setup.** Compile with `npx tsc example_assert_string.ts --strict`.

**Expected Output:**
```
johndoe
```

**Why this output occurs.** `assertIsString` throws for non-string values and returns normally for strings. After the call, control-flow analysis narrows `input` from `unknown` to `string`, enabling `.trim()` and `.toLowerCase()` without `if` checks or casts.

---

#### Example 2: `asserts condition` — generic assertion

```typescript
// example_assert_condition.ts
function assert(condition: unknown, message?: string): asserts condition {
  if (!condition) {
    throw new Error(message ?? "Assertion failed");
  }
}

function range(from: unknown, to: unknown) {
  assert(typeof from === "number", "from must be a number");
  assert(typeof to === "number", "to must be a number");
  // Both from and to are narrowed to number.
  return from + to;
}

console.log(range(0, 5)); // Expected: 5
// range("a", "f"); // Throws: from must be a number
```

**Expected Output:**
```
5
```

**Why this output occurs.** The `asserts condition` signature tells TypeScript that if `assert` returns, the boolean expression passed as `condition` is true. Each `assert(typeof x === "number")` call narrows `x` to `number` for the rest of the scope.

---

#### Example 3: Generic assertion function

```typescript
// example_assert_generic.ts
function assertDefined<T>(
  value: T | null | undefined,
  name: string
): asserts value is T {
  if (value === null || value === undefined) {
    throw new Error(`${name} is required`);
  }
}

function processUser(user: { email: string } | null) {
  assertDefined(user, "user");
  // user is now narrowed to { email: string }.
  console.log(user.email);
}

processUser({ email: "ada@example.com" }); // Expected: ada@example.com
```

**Expected Output:**
```
ada@example.com
```

**Why this output occurs.** The generic assertion `asserts value is T` removes `null` and `undefined` from `user`, narrowing it to the object type. The `name` parameter provides a descriptive error message.

---

### Real-World Cases

- **API response validation**: asserting that `response.data` matches a domain type before accessing fields.
- **DOM queries**: `assertDefined(document.getElementById("id"))` to narrow `HTMLElement | null` to `HTMLElement`.
- **Configuration parsing**: asserting that environment variables are defined before use.
- **Discriminated union handling**: `assertNever(value)` for exhaustiveness checking.
- **Form validation**: asserting that required fields are present and non-empty.

---

## 2. `asserts` Keyword

### Definitions

**Core Definition.** The **`asserts` keyword** is the syntactic marker in a function's return type that declares an **assertion signature**—a contract that the function throws if the asserted condition is false.

**Technical Definition.** `asserts` appears only in the return type position of a function declaration. It introduces either `asserts condition` (asserting a boolean expression) or `asserts parameterName is Type` (asserting a narrowed type for a parameter). The keyword has no runtime representation and is erased, but it changes how TypeScript's control-flow analysis treats calls to the function.

**Beginner-Friendly Explanation.** `asserts` is **the word that turns a throw-if-fail function into a type-narrowing tool**. Writing `asserts value is string` tells TypeScript “if this function returns, `value` is a string.”

### Purposes

- **To declare an assertion signature** explicitly, making the narrowing contract visible to callers.
- **To enable control-flow narrowing** after the assertion call without `if` checks.
- **To document the function's throwing behavior** in the type system.
- **To distinguish assertion functions from type predicates** (`is`) and plain boolean functions.

### Syntax Rules and Structure

**`asserts condition`:**

```typescript
function assert(condition: unknown): asserts condition { ... }
```

**`asserts value is T`:**

```typescript
function assertIsString(value: unknown): asserts value is string { ... }
```

**Component breakdown:**

- `asserts` — the keyword.
- `condition` — a boolean expression parameter.
- `value is T` — a type predicate-style narrowing for a parameter.

**Syntax rules.**

- `asserts` must be the **first token** of the return type annotation.
- The function must return `void` (or have no return type annotation; `void` is inferred).
- The function must throw when the assertion fails.
- `asserts condition` requires the condition to be a parameter (or an expression derived from parameters).
- `asserts value is T` requires `value` to be a parameter.

**Constraints and limitations.**

- `asserts` is **not a value**; it cannot be used in expressions.
- Assertion signatures are erased and have no runtime effect beyond the function body.
- TypeScript does not validate that the runtime check matches the assertion.

### Annotated Complete Step by Step Code Examples

#### Example 1: `asserts condition`

```typescript
// example_asserts_condition.ts
function assert(condition: unknown, message?: string): asserts condition {
  if (!condition) {
    throw new Error(message ?? "Assertion failed");
  }
}

const value: unknown = "hello";
assert(typeof value === "string");
console.log(value.toUpperCase()); // Expected: HELLO
```

**Expected Output:**
```
HELLO
```

**Why this output occurs.** The `asserts condition` signature tells TypeScript that if `assert` returns, the condition `typeof value === "string"` is true. The subsequent `.toUpperCase()` call is therefore type-safe.

---

#### Example 2: `asserts value is T`

```typescript
// example_asserts_value.ts
function assertIsObject(value: unknown): asserts value is object {
  if (typeof value !== "object" || value === null) {
    throw new Error("Value must be an object");
  }
}

const data: unknown = { status: "success" };
assertIsObject(data);
console.log(data); // Expected: { status: 'success' }
```

**Expected Output:**
```
{ status: 'success' }
```

**Why this output occurs.** The assertion narrows `data` from `unknown` to `object` after the call, enabling safe property access in subsequent code.

---

### Real-World Cases

- **Invariant helpers**: `assert(condition)` for general preconditions.
- **Type-specific guards**: `assertIsString`, `assertIsArray`, `assertIsDefined`.
- **Exhaustiveness**: `assertNever(value)` for unreachable branches.

---

## 3. Assertion Function Declaration

### Definitions

**Core Definition.** An **assertion function declaration** is the statement that defines an assertion function, including its parameter list and its `asserts` return type.

**Technical Definition.** Assertion functions must be **declared with an explicit assertion signature**; TypeScript does not infer `asserts` signatures for arrow-function expressions assigned to variables without annotations. The declaration may be **exported**, **generic**, and **overloaded**. It must return `void` and must throw on failure.

**Beginner-Friendly Explanation.** An assertion function declaration is **the full signature of your assertion function**. You must write the `asserts` return type explicitly—TypeScript will not guess it.

### Purposes

- **To define a reusable assertion** with a clear contract.
- **To export assertion functions** for use across modules.
- **To make assertion functions generic** for broader applicability.
- **To document the throwing behavior** in the public API.

### Syntax Rules and Structure

**Basic declaration:**

```typescript
function assertDefined<T>(value: T | null | undefined, name: string): asserts value is T {
  if (value === null || value === undefined) throw new Error(`${name} is required`);
}
```

**Declaration with explicit `void`:**

```typescript
function assert(condition: unknown): asserts condition { ... }
```

**Syntax rules.**

- The `asserts` keyword must appear in the return type annotation.
- The function must return `void` (explicitly or implicitly).
- The function must throw when the condition fails.
- Generic parameters may be used in the assertion.

**Constraints and limitations.**

- Arrow functions assigned to variables require an explicit assertion signature on the variable type.
- Assertion functions cannot be used as type predicates (`value is T`) directly; they are distinct signatures.

### Annotated Complete Step by Step Code Examples

#### Example 1: Exported generic assertion

```typescript
// example_declaration_exported.ts
export function assertDefined<T>(
  value: T | null | undefined,
  name: string
): asserts value is T {
  if (value === null || value === undefined) {
    throw new Error(`${name} must be defined`);
  }
}
```

```typescript
// consumer.ts
import { assertDefined } from "./example_declaration_exported";

const user: { email: string } | null = { email: "ada@example.com" };
assertDefined(user, "user");
console.log(user.email); // Expected: ada@example.com
```

**Expected Output:**
```
ada@example.com
```

**Why this output occurs.** The exported assertion function narrows `user` to the non-null type for all consumers. The explicit `asserts value is T` signature locks the public contract.

---

#### Example 2: Arrow function with explicit assertion type

```typescript
// example_declaration_arrow.ts
type AssertDefined = <T>(value: T | null | undefined) => asserts value is T;

const assertDefined: AssertDefined = (value) => {
  if (value == null) throw new Error("Value is required");
};

const name: string | null = "Ada";
assertDefined(name);
console.log(name.toUpperCase()); // Expected: ADA
```

**Expected Output:**
```
ADA
```

**Why this output occurs.** The type alias `AssertDefined` explicitly declares the assertion signature, allowing an arrow function to be used as an assertion function.

---

### Real-World Cases

- **Library utilities**: exporting `assertDefined`, `assertIsString`, and `assertNever`.
- **Test helpers**: `assertEqual` with runtime checks.
- **Validation modules**: shared precondition checks across services.

---

## 4. Assertion Function Usage

### Definitions

**Core Definition.** **Assertion function usage** refers to calling assertion functions in a scope to **validate preconditions and narrow types** for subsequent code.

**Technical Definition.** Assertion functions are called as **standalone statements** (not in conditions). After the call, TypeScript's control-flow analysis applies the asserted narrowing to the remainder of the scope. They are commonly used at **function entry** to validate parameters, and inside **callbacks** or **async functions** to validate external data.

**Beginner-Friendly Explanation.** You use assertion functions **as guard clauses at the top of a function**. Call the assertion, and everything after it is type-safe.

### Purposes

- **To validate function parameters** before using them.
- **To narrow `unknown` values** after parsing external data.
- **To enforce invariants** at the start of a function.
- **To eliminate repetitive `if`-throw patterns** across multiple functions.
- **To combine runtime checks with type narrowing** without `as` casts.

### Syntax Rules and Structure

**Function entry guard:**

```typescript
function process(user: User | null) {
  assertDefined(user, "user");
  // user is User here.
  console.log(user.email);
}
```

**Async data validation:**

```typescript
async function loadUser(id: string) {
  const data: unknown = await fetch(`/users/${id}`).then((r) => r.json());
  assertIsUser(data);
  console.log(data.name); // data is User
}
```

**Syntax rules.**

- Assertion calls are standalone statements; they cannot be used inside `if` conditions.
- The narrowed type applies from the call site to the end of the scope.
- Assertion functions may be composed: call multiple assertions in sequence.

**Constraints and limitations.**

- The narrowing does not persist across function boundaries unless the assertion is called inside the target function.
- Assertion functions in unions do not act as assertion functions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Parameter validation

```typescript
// example_usage_param.ts
interface Order {
  id: string;
  status: "pending" | "shipped";
}

function assertValidOrder(order: Order | null): asserts order is Order {
  if (!order) throw new Error("Order is required");
  if (order.status !== "pending") throw new Error("Order must be pending");
}

function processOrder(order: Order | null) {
  assertValidOrder(order);
  // order is narrowed to Order here.
  console.log(order.id);
}

processOrder({ id: "o1", status: "pending" }); // Expected: o1
```

**Expected Output:**
```
o1
```

**Why this output occurs.** The assertion validates that `order` is non-null and pending, narrowing `order` to `Order` for the rest of `processOrder`.

---

#### Example 2: Async data validation

```typescript
// example_usage_async.ts
function assertIsUser(value: unknown): asserts value is { id: string; name: string } {
  if (
    typeof value !== "object" ||
    value === null ||
    !("id" in value) ||
    !("name" in value)
  ) {
    throw new Error("Invalid user");
  }
}

async function loadUser() {
  const data: unknown = JSON.parse('{"id":"u1","name":"Ada"}');
  assertIsUser(data);
  console.log(data.name); // Expected: Ada
}

loadUser();
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The assertion validates the shape of the parsed JSON and narrows `data` to the user type, enabling safe access to `name`.

---

### Real-World Cases

- **Service methods**: validating that dependencies are present at function entry.
- **API clients**: asserting response shapes before mapping to domain models.
- **Form handlers**: asserting that required fields are non-empty.
- **Exhaustiveness checks**: `assertNever` in `default` branches.

---

## 5. Assertion Function Best Practices

### Definitions

**Core Definition.** **Assertion function best practices** are guidelines for writing assertions that are **accurate, reusable, and safe**, avoiding lying assertions, over-validation, and misuse of the throwing contract.

**Technical Definition.** Best practices center on **ensuring the runtime check matches the assertion**, **using assertions for shared preconditions**, **preferring them over repetitive `if`-throw patterns**, **keeping assertions pure and focused**, **using explicit `asserts` signatures for exported functions**, and **distinguishing assertion functions from type predicates** based on whether the caller needs conditional handling.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep assertion functions reliable. The big rule is: **an assertion function that lies is worse than an `as` cast** because the bug hides behind a trustworthy name. Keep assertions honest, focused, and tested.

### Purposes

- **To prevent lying assertions** that silently narrow to incorrect types.
- **To centralize validation logic** and reduce duplication across functions.
- **To keep assertions pure, focused, and testable**.
- **To choose between assertion functions and type predicates** appropriately.
- **To ensure assertions are used for preconditions, not for control-flow branching**.

### Syntax Rules and Structure

Best practices are conventions that interact with assertion syntax. The core rules are:

1. **Ensure the runtime check establishes every promised property.** A predicate `asserts value is User` must verify all required fields, not just one.
2. **Use assertion functions for shared preconditions.** Extract repeated `if`-throw checks into a reusable assertion function.
3. **Prefer assertion functions over type predicates for guard-clause style.** When the caller should **not proceed** on failure, use an assertion function; when the caller needs to **branch** on the result, use a type predicate.
4. **Keep assertion functions pure and focused.** One assertion should check one logical condition; compose multiple assertions rather than writing a monolithic checker.
5. **Use explicit `asserts` signatures for exported assertion functions.** Do not rely on inference; assertion signatures are never inferred.
6. **Test assertion functions with both accepted and rejected values.** TypeScript cannot catch lying assertions.
7. **Avoid over-asserting.** Do not assert conditions that are already guaranteed by the type system; use `satisfies` or type annotations instead.
8. **Use `assertNever` for exhaustiveness.** In `default` branches of discriminated union switches, call `assertNever(value)` to catch unhandled cases.

**Component breakdown (assertion decision tree):**

```
Do you need to validate a runtime condition?
  ├── Yes → Should the caller branch on the result?
  │        ├── Yes → Use a type predicate (`value is T`).
  │        └── No → Should the caller stop execution on failure?
  │                 ├── Yes → Use an assertion function (`asserts value is T`).
  │                 └── No → Use a plain boolean function.
  └── No → Use `satisfies` or a type annotation.
```

**Constraints and limitations.**

- Assertion functions that throw are not suitable for expected, recoverable failures; use type predicates with `if`/`else` for recoverable branching.
- Overusing assertions can make error handling implicit and harder to trace.
- Assertion functions cannot narrow `this` properties.

### Annotated Complete Step by Step Code Examples

#### Example 1: Extract repeated `if`-throw checks

```typescript
// example_best_extract.ts
// ❌ Repeated if-throw in every function.
function processOrder(order: Order | null) {
  if (!order) throw new Error("Order is required");
  sendConfirmation(order); // narrowed, but duplicated
}

function shipOrder(order: Order | null) {
  if (!order) throw new Error("Order is required");
  createShipment(order); // same check duplicated
}

// ✅ Reusable assertion function.
function assertDefined<T>(value: T | null | undefined, name: string): asserts value is T {
  if (value === null || value === undefined) {
    throw new Error(`${name} is required`);
  }
}

function processOrderGood(order: Order | null) {
  assertDefined(order, "order");
  sendConfirmation(order); // narrowed to Order
}

function shipOrderGood(order: Order | null) {
  assertDefined(order, "order");
  createShipment(order); // same narrowing, no duplication
}
```

**Why this is better.** The assertion function centralizes the null check and narrowing, eliminating duplicated validation and keeping the callers flat and readable.

---

#### Example 2: Assertion function vs. type predicate

```typescript
// example_best_predicate_vs_assert.ts
// Type predicate: caller branches on the result.
function isString(value: unknown): value is string {
  return typeof value === "string";
}

// Assertion function: caller stops execution on failure.
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") throw new Error("Not a string");
}

// Use the predicate when you need if/else.
function processPredicate(value: unknown) {
  if (isString(value)) {
    console.log(value.toUpperCase());
  } else {
    console.log("Not a string");
  }
}

// Use the assertion when failure is fatal.
function processAssertion(value: unknown) {
  assertIsString(value);
  console.log(value.toUpperCase()); // no if needed
}

processAssertion("hello"); // Expected: HELLO
```

**Expected Output:**
```
HELLO
```

**Why this distinction matters.** The predicate allows recoverable branching; the assertion enforces a precondition and throws on failure. Choosing the right tool depends on whether the caller needs to handle the failure case.

---

#### Example 3: `assertNever` for exhaustiveness

```typescript
// example_best_assert_never.ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function assertNever(value: never): never {
  throw new Error(`Unhandled: ${JSON.stringify(value)}`);
}

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.radius ** 2;
    case "square": return shape.side ** 2;
    default: return assertNever(shape);
  }
}

console.log(area({ kind: "circle", radius: 1 }).toFixed(2)); // Expected: 3.14
```

**Expected Output:**
```
3.14
```

**Why this output occurs.** `assertNever` accepts `never`, ensuring that all union members are handled. Adding a new variant without handling it causes a compile-time error because `shape` in `default` is no longer `never`.

---

### Real-World Cases

- **Shared preconditions**: `assertDefined(user, "user")` across multiple service methods.
- **API validation**: `assertIsUser(response)` before mapping to domain models.
- **Exhaustiveness**: `assertNever` in reducers and state machines.
- **Test helpers**: assertion functions for expected error conditions.

---

## References

- Assertion Functions (TypeScript 3.7 Release Notes) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions
- Assertion Functions (TypeScript Handbook, Narrowing) — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#assertion-functions
- Assert | TypeScript Guide by Convex — https://www.convex.dev/typescript/advanced/advanced-concepts/typescript-assert
- TypeScript Asserts: Syntax, Usage, and Examples (Mimo) — https://mimo.org/glossary/typescript/asserts
- Use Assertion Functions for Validation (pproenca/dot-skills) — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.curated/typescript/references/safety-assertion-functions.md
- Use Assertion Functions for Precondition Checks (pproenca/dot-skills) — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-refactor/references/narrow-assertion-functions.md
- What is the difference between "asserts value is type" and "value is type"? (Stack Overflow) — https://stackoverflow.com/questions/70931239
- Narrow the unknown Type with TypeScript's Assertion Functions (egghead.io) — https://egghead.io/lessons/typescript-narrow-the-unknown-type-with-typescript-s-assertion-functions
- TypeScript 3.7 Assertion Signatures (GitHub / microsoft/TypeScript-Website) — https://github.com/microsoft/TypeScript-Website
- Validate Types with Brands and Assertions (Total TypeScript) — https://www.totaltypescript.com/workshops/advanced-typescript-patterns/type-predicates-and-assertion-functions/combining-brands-with-assertion-functions/solution
- Unions of Assertion Functions Do Not Act as Assertion Functions (microsoft/TypeScript Issue #59707) — https://github.com/microsoft/TypeScript/issues/59707