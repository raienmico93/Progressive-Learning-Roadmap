# Type Predicates in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **type predicate** is a user-defined function return type annotation of the form `parameterName is Type`. It is a compile-time contract between a boolean-returning function and the type system: when the function returns `true`, TypeScript **narrows the type of the argument** from a broader type to the specified narrower type within the guarded scope. Type predicates are the primary mechanism for encapsulating **custom runtime type checks** that the built-in guards (`typeof`, `instanceof`, `in`) cannot express.

**Technical Definition.** A type predicate function has the signature `function isX(value: unknown): value is X`. The `is` keyword separates a **parameter name** from the **target type**, and the function must return a `boolean`. TypeScript does **not verify** that the runtime implementation actually matches the predicate; the developer assumes full responsibility for correctness. Since **TypeScript 5.5**, the compiler **infers** type predicates automatically for functions that satisfy specific structural conditions (single `return`, no parameter mutation, and a boolean expression refining a parameter), reducing the need for explicit `is` annotations in simple cases.

**Beginner-Friendly Explanation.** A type predicate is **a function that teaches TypeScript what type a value really is**. You write `function isString(value: unknown): value is string`, and inside the function you check whether `value` is actually a string. When the function returns `true`, TypeScript **trusts you** and treats `value` as a string in the calling code. It is a safe alternative to `as` casts because the check actually happens at runtime—but TypeScript does not verify that your check is correct, so a lying predicate is worse than an assertion.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Compile-time narrowing** | The predicate narrows the argument's type in the caller when the function returns `true`. |
| **Parameter-scoped** | The predicate must reference a **parameter** of the function, not a property like `this.node`. |
| **Boolean return required** | The function must return a `boolean`; predicates cannot return other types. |
| **Runtime check trusted** | TypeScript does **not** verify that the runtime check matches the predicate. |
| **Inference (TS 5.5+)** | Simple single-return guards infer the predicate automatically without an explicit `is`. |
| **Explicit `is` for complex guards** | Multi-return guards and exported guards should use explicit `is` annotations. |
| **Erasure** | Type predicates are compile-time only and emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: union types, type aliases, and interfaces.
- **Control flow**: `if`/`else`, early returns, and `switch`.
- **The `unknown` type** and type narrowing.
- **Built-in type guards**: `typeof`, `instanceof`, and `in`.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support, ideally version 5.5+).

---

### Related Programming Areas

- **Type narrowing** — the broader mechanism that type predicates serve.
- **Type guards** — the runtime checks that drive narrowing.
- **Type assertions (`as`)** — the unsafe alternative that predicates replace.
- **Assertion functions (`asserts value is T`)** — the throwing counterpart to type predicates.
- **Runtime validation** — libraries like Zod and `io-ts` pair with predicates for external data.

---

## Core Concepts / Features

---

## 1. Type Predicates

### Definitions

**Core Definition.** A **type predicate** is a function return type annotation (`value is Type`) that narrows the argument's type when the function returns `true`.

**Technical Definition.** A type predicate function declares `parameterName is Type` as its return type. The `parameterName` must be a parameter of the function. When the function returns `true`, TypeScript narrows the argument from its declared type to `Type` within the guarded scope. Type predicates are **user-defined type guards** and are the mechanism behind `Array.prototype.filter`'s ability to remove `undefined` values.

**Beginner-Friendly Explanation.** A type predicate is **a reusable, named runtime check that also teaches TypeScript about the type**. Instead of repeating a complex check, you put it in a function and mark it with `is`.

### Purposes

- **To encapsulate complex runtime type checks** in a reusable function that narrows the caller's type.
- **To replace unsafe `as` assertions** with runtime-verified checks.
- **To validate external data** (API responses, `JSON.parse` results) before treating it as a trusted type.
- **To enable safe array filtering** that removes `null`, `undefined`, or invalid members from arrays.
- **To compose multiple guards** and apply them consistently across a codebase.

### Syntax Rules and Structure

**General syntax:**

```typescript
function isX(value: unknown): value is X {
  return /* boolean runtime check */;
}
```

**Component breakdown:**

- `function isX` — the guard function, conventionally named `isX`.
- `value: unknown` — the parameter being checked (often `unknown` for maximum applicability).
- `: value is X` — the **type predicate** return type.
- The body must return a `boolean`.

**Syntax rules.**

- The predicate parameter must be a **parameter** of the function, not a property or expression.
- The function must return a `boolean`.
- The predicate is erased at runtime; it produces no JavaScript beyond the function itself.
- Since TS 5.5, a single-return boolean function with no parameter mutation **infers** its predicate automatically.

**Constraints and limitations.**

- TypeScript does **not verify** that the runtime check matches the predicate. A lying predicate is a correctness bug that the compiler cannot catch.
- Predicates cannot narrow `this` properties like `this.node`; they only work on parameters.
- Inferred predicates only fire for functions with a **single return** and no branching.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit type predicate

```typescript
// example_predicate_explicit.ts
// Step 1: Declare a type predicate function.
function isString(value: unknown): value is string {
  return typeof value === "string";
}

// Step 2: Use it to narrow a union.
function process(value: string | number) {
  if (isString(value)) {
    console.log(value.toUpperCase()); // narrowed to string
  } else {
    console.log(value.toFixed(2)); // narrowed to number
  }
}

process("hello"); // Expected: HELLO
process(42);      // Expected: 42.00
```

**Setup.** Compile with `npx tsc example_predicate_explicit.ts --strict`.

**Expected Output:**
```
HELLO
42.00
```

**Why this output occurs.** The predicate `value is string` tells TypeScript that `isString` narrows its argument to `string` when it returns `true`. The `if` branch is narrowed to `string`, enabling `.toUpperCase()`; the `else` branch is narrowed to `number`, enabling `.toFixed()`.

---

#### Example 2: Inferred type predicate (TypeScript 5.5+)

```typescript
// example_predicate_inferred.ts
// Step 1: A simple single-return guard infers its predicate.
function isNonNullish<T>(value: T): value is NonNullable<T> {
  return value != null;
}

// Or without any annotation at all:
function isNumber(value: unknown) {
  return typeof value === "number";
}
// Inferred signature: (value: unknown) => value is number

const values: (string | null)[] = ["a", null, "b"];
const nonNull = values.filter(isNonNullish);

console.log(nonNull); // Expected: [ 'a', 'b' ]
```

**Expected Output:**
```
[ 'a', 'b' ]
```

**Why this output occurs.** TypeScript 5.5 infers the predicate `value is NonNullable<T>` from the single boolean return. `Array.prototype.filter` uses the predicate to produce `string[]` rather than `(string | null)[]`.

---

### Real-World Cases

- **API response validation**: `isUser(response): response is User`.
- **Array filtering**: `values.filter(isNotNull)` to remove `null`/`undefined`.
- **Configuration parsing**: `isConfig(value)` before treating environment variables as typed configuration.
- **Test utilities**: `isError(result)` for discriminated result types.

---

## 2. `is` Keyword

### Definitions

**Core Definition.** The **`is` keyword** is the syntactic token used in a type predicate return type to separate the **parameter name** from the **narrowed type** (`value is Type`).

**Technical Definition.** `is` appears only in the return type position of a function. It forms a **type predicate**, which the compiler interprets as a narrowing assertion. The left side must be a **parameter name**; the right side is any valid type. The `is` keyword has no runtime representation and is erased.

**Beginner-Friendly Explanation.** `is` is **the word that turns a boolean-returning function into a type guard**. Writing `value is string` as the return type tells TypeScript “when this function returns `true`, `value` is a `string`.”

### Purposes

- **To declare a type predicate explicitly**, making the narrowing contract visible to callers and tooling.
- **To document the guard's behavior** for exported or public API functions.
- **To satisfy TypeScript's type predicate syntax** for multi-return guards that cannot be inferred automatically.
- **To enable narrowing in callers** without requiring type assertions.

### Syntax Rules and Structure

**General syntax:**

```typescript
function isX(value: unknown): value is X {
  // runtime check
}
```

**Component breakdown:**

- `is` — the predicate keyword.
- `value` — the parameter name (must match a parameter).
- `X` — the narrowed type.

**Syntax rules.**

- `is` is used only in **return type** annotations.
- The left-hand side must be a **parameter name**, not a property or arbitrary expression.
- The function must return a `boolean`.
- Explicit `is` is optional for simple single-return guards (TS 5.5+ infers them).

**Constraints and limitations.**

- `is` cannot narrow `this` properties.
- A predicate that does not match the runtime check is a silent correctness bug.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit `is` for a generic guard

```typescript
// example_is_generic.ts
function isNonNull<T>(value: T | null): value is T {
  return value !== null;
}

const values: (string | null)[] = ["a", null, "b"];
const nonNull = values.filter(isNonNull);

console.log(nonNull); // Expected: [ 'a', 'b' ]
```

**Expected Output:**
```
[ 'a', 'b' ]
```

**Why this output occurs.** The `value is T` predicate tells `filter` that `isNonNull` removes `null` values, producing `string[]` from `(string | null)[]`.

---

### Real-World Cases

- **Exported guards**: `export function isUser(value: unknown): value is User`.
- **Multi-return guards**: guards that branch across several `return` statements need explicit `is`.
- **Generic guards**: `isNonNull<T>(value: T | null): value is T`.

---

## 3. Type Predicate Declaration

### Definitions

**Core Definition.** A **type predicate declaration** is the function signature that declares the predicate, including the parameter list and the `parameterName is Type` return type.

**Technical Definition.** The declaration may be an explicit signature or inferred by the compiler (TS 5.5+). Explicit declarations are required for **multi-return** guards, **exported** guards, and cases where inference is undesirable. Inferred declarations require a **single return**, **no parameter mutation**, and a **boolean expression tied to a parameter refinement**.

**Beginner-Friendly Explanation.** A type predicate declaration is **the signature of your guard function**. You can write it explicitly with `is`, or let TypeScript 5.5 infer it from a simple function body.

### Purposes

- **To define the narrowing contract** of a guard function.
- **To decide between explicit `is` and inferred predicates** based on complexity and API surface.
- **To ensure the guard is reusable** across multiple call sites.
- **To document the intended validation logic** in a named function.

### Syntax Rules and Structure

**Explicit declaration:**

```typescript
function isX(value: unknown): value is X { ... }
```

**Inferred declaration (TS 5.5+):**

```typescript
function isX(value: unknown) {
  return /* single boolean return refining a parameter */;
}
```

**Conditions for inference (TS 5.5+):**

- No explicit return type or predicate annotation.
- A **single `return` statement** with no implicit returns.
- No mutation of the parameter.
- The returned expression is a **boolean** tied to a refinement on the parameter.

**Syntax rules.**

- Explicit `is` is required for multi-return guards and exported guards.
- Inferred predicates are available only in TypeScript 5.5+.
- Inferred predicates do not fire for arrow functions with expression bodies? Actually, they do for single-return arrow functions.

**Constraints and limitations.**

- Inferred predicates may incur a performance cost during type checking.
- Multi-return guards and guards with early exits need explicit `is`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit declaration for an exported guard

```typescript
// example_declaration_exported.ts
// ✅ Explicit is documents the public contract.
export function isUser(value: unknown): value is { id: string; name: string } {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value &&
    typeof (value as { id: unknown }).id === "string" &&
    typeof (value as { name: unknown }).name === "string"
  );
}
```

```typescript
// consumer.ts
import { isUser } from "./example_declaration_exported";

function handle(data: unknown) {
  if (isUser(data)) {
    console.log(data.name); // narrowed to { id: string; name: string }
  }
}

handle({ id: "u1", name: "Ada" }); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The explicit `value is { id: string; name: string }` annotation locks the public contract of the exported guard. Consumers get narrowing without depending on inference behavior.

---

#### Example 2: Inferred declaration for a simple local guard

```typescript
// example_declaration_inferred.ts
// ✅ No annotation needed: TS 5.5+ infers the predicate.
function isString(value: unknown) {
  return typeof value === "string";
}

function process(value: string | number) {
  if (isString(value)) {
    console.log(value.toUpperCase());
  }
}

process("hello"); // Expected: HELLO
```

**Expected Output:**
```
HELLO
```

**Why this output occurs.** The single boolean return `typeof value === "string"` is analyzed by control-flow analysis, and the predicate `value is string` is inferred automatically.

---

### Real-World Cases

- **Exported library guards**: explicit `is` for stable public contracts.
- **Local helpers**: inferred predicates for simple one-line checks.
- **Multi-return guards**: explicit `is` for guards that branch.

---

## 4. Type Predicate Usage

### Definitions

**Core Definition.** **Type predicate usage** refers to calling predicate functions in conditional contexts (`if`, `filter`, `&&`, ternaries) to narrow types and enable safe member access.

**Technical Definition.** Predicate functions are used as **guards in control-flow positions**. They are especially valuable with `Array.prototype.filter`, which uses the predicate to produce a narrowed array type. They can be **composed** with logical operators and other guards to build complex narrowing conditions.

**Beginner-Friendly Explanation.** You use a type predicate **wherever you need to check a type at runtime and have TypeScript trust the result**. The most common place is `array.filter(isSomething)` to remove invalid elements.

### Purposes

- **To narrow union types** before accessing member-specific properties.
- **To filter arrays** and remove `null`, `undefined`, or invalid members while preserving precise element types.
- **To validate external data** at API boundaries before use.
- **To compose guards** with `&&`, `||`, and nested conditions.
- **To replace unsafe assertions** in production code.

### Syntax Rules and Structure

**Conditional narrowing:**

```typescript
if (isUser(data)) {
  console.log(data.name);
}
```

**Array filtering:**

```typescript
const valid = values.filter(isNotNull);
```

**Composition:**

```typescript
if (isUser(data) && data.age >= 18) { ... }
```

**Syntax rules.**

- Predicates may be used in any boolean context.
- `filter` with a predicate narrows the resulting array type.
- Predicates may be combined with logical operators and other guards.

**Constraints and limitations.**

- Predicates only narrow **parameters**; they cannot narrow arbitrary properties without a wrapper.
- Predicates do not narrow across function boundaries unless the function is itself a recognized guard.

### Annotated Complete Step by Step Code Examples

#### Example 1: Array filtering with a predicate

```typescript
// example_usage_filter.ts
function isNotNull<T>(value: T | null | undefined): value is T {
  return value != null;
}

const values: (string | null | undefined)[] = ["a", null, "b", undefined];
const clean: string[] = values.filter(isNotNull);

console.log(clean); // Expected: [ 'a', 'b' ]
```

**Expected Output:**
```
[ 'a', 'b' ]
```

**Why this output occurs.** `filter` uses the predicate to narrow the array from `(string | null | undefined)[]` to `string[]`.

---

#### Example 2: Composing guards

```typescript
// example_usage_compose.ts
interface User {
  id: string;
  age: number;
}

function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "age" in value
  );
}

function isAdult(user: User): boolean {
  return user.age >= 18;
}

function process(data: unknown) {
  if (isUser(data) && isAdult(data)) {
    console.log(data.id); // narrowed to User
  }
}

process({ id: "u1", age: 20 }); // Expected: u1
```

**Expected Output:**
```
u1
```

**Why this output occurs.** The `isUser` predicate narrows `data` to `User`, enabling access to `data.age` in `isAdult`. The composed condition preserves narrowing.

---

### Real-World Cases

- **API clients**: `if (isApiError(response)) { ... } else { ... }`.
- **Form validation**: `values.filter(isValidField)`.
- **Configuration loading**: `if (isConfig(raw)) { useConfig(raw); }`.

---

## 5. Type Predicate Best Practices

### Definitions

**Core Definition.** **Type predicate best practices** are guidelines for writing predicates that are **accurate, reusable, and safe**, avoiding the severe pitfall of **lying predicates** that silently corrupt type safety.

**Technical Definition.** Best practices center on **replacing assertions with predicates**, **keeping predicates pure and simple**, **using descriptive names**, **reserving explicit `is` for multi-return and exported guards**, **testing predicates with accepted and rejected values**, and **understanding that TypeScript does not verify the runtime check matches the predicate**. A predicate is a **contract of trust**: the runtime check must establish every property promised by the signature.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep type predicates from becoming dangerous. The big rule is: **a lying predicate is worse than an assertion** because the bug hides behind a safe-sounding name. Keep predicates simple, test them, and use explicit `is` when the function is exported or branches.

### Purposes

- **To prevent lying predicates** that silently narrow to incorrect types.
- **To replace unsafe `as` assertions** with runtime-verified checks.
- **To keep predicates readable and testable** with focused, pure implementations.
- **To choose between inferred and explicit predicates** appropriately.
- **To ensure predicates establish every property promised by their signature**.

### Syntax Rules and Structure

Best practices are conventions that interact with predicate syntax. The core rules are:

1. **Replace assertions with predicates.** `as` silences the compiler without runtime verification; predicates verify at runtime.
2. **Keep predicates pure and simple.** Focus on checking one specific aspect; avoid side effects.
3. **Use descriptive names.** Name guards `isX` or `hasX` so their purpose is clear.
4. **Reserve explicit `is` for multi-return guards and exported guards.** Inference only fires for single-return functions; exported guards should lock their public contract with an explicit annotation.
5. **Ensure the runtime check establishes every promised property.** A predicate that claims `value is User` must verify all required fields, not just one.
6. **Test predicates with both accepted and rejected values.** TypeScript will not catch a predicate that returns `true` for invalid data.
7. **Prefer inferred predicates for simple local guards.** Let TS 5.5+ infer the predicate for single-return functions to keep the code concise.
8. **Avoid blind casts in production code.** Use predicates, refactor the type surface, or throw errors for invalid states.

**Component breakdown (predicate decision tree):**

```
Do you need to narrow a type at runtime?
  ├── Yes → Can a built-in guard (typeof, instanceof, in) express it?
  │        ├── Yes → Use the built-in guard.
  │        └── No → Write a custom predicate.
  │                 ├── Is the body a single boolean return? → Let TS 5.5+ infer.
  │                 └── Does it have multiple returns or is it exported? → Use explicit `is`.
  └── No → Use a type assertion only if you have external proof.
```

**Constraints and limitations.**

- Inferred predicates may have a performance cost during type checking.
- Predicates cannot narrow `this` properties.
- A lying predicate is a correctness bug that the compiler cannot detect.

### Annotated Complete Step by Step Code Examples

#### Example 1: Predicate instead of assertion

```typescript
// example_best_predicate.ts
interface Order {
  total: number;
}

// ❌ Assertion: trusts the developer, not the runtime.
function handleBad(payload: unknown) {
  const order = payload as Order;
  console.log(order.total); // may crash at runtime
}

// ✅ Predicate: verifies at runtime.
function isOrder(value: unknown): value is Order {
  return (
    typeof value === "object" &&
    value !== null &&
    "total" in value &&
    typeof (value as Order).total === "number"
  );
}

function handleGood(payload: unknown) {
  if (!isOrder(payload)) throw new Error("Invalid payload");
  console.log(payload.total); // narrowed to Order
}

handleGood({ total: 100 }); // Expected: 100
```

**Expected Output:**
```
100
```

**Why this output occurs.** The predicate verifies the runtime shape before narrowing. The assertion version would compile but crash if `payload` lacked `total`.

---

#### Example 2: Explicit `is` for an exported multi-return guard

```typescript
// example_best_exported.ts
// ✅ Explicit is documents the public contract.
export function isStringOrNumber(value: unknown): value is string | number {
  if (typeof value === "string") return true;
  if (typeof value === "number") return true;
  return false;
}
```

```typescript
// consumer.ts
import { isStringOrNumber } from "./example_best_exported";

function process(value: unknown) {
  if (isStringOrNumber(value)) {
    console.log(typeof value); // "string" | "number"
  }
}

process("hello"); // Expected: string
```

**Expected Output:**
```
string
```

**Why this output occurs.** The guard has multiple returns, so inference does not apply. The explicit `value is string | number` locks the public contract for consumers.

---

### Real-World Cases

- **API boundaries**: replacing `as User` with `isUser(response)`.
- **Library exports**: explicit `is` annotations for stable public guards.
- **Local helpers**: inferred predicates for simple one-line checks.
- **Code review**: rejecting predicates that verify only a subset of required fields.

---

## References

- Documentation - Narrowing: Using Type Predicates — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates
- Documentation - TypeScript 5.5: Inferred Type Predicates — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-5.html#inferred-type-predicates
- Playground Example - Type Guards and Type Predicates — https://www.typescriptlang.org/play/typescript/language-extensions/type-guards.ts.html
- Documentation - Advanced Types: User-Defined Type Guards — https://www.typescriptlang.org/docs/handbook/advanced-types.html
- pproenca/dot-skills - Write Custom Type Guards Instead of Type Assertions — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-refactor/references/narrow-custom-type-guards.md
- pproenca/dot-skills - Use Type Guards for Runtime Type Checking — https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.curated/typescript/references/safety-use-type-guards.md
- Prisma Next - TypeScript Patterns: Type Predicates — https://github.com/prisma/prisma-next/blob/main/docs/reference/typescript-patterns.md
- DEV Community - Type Guards Using Type Predicates in TypeScript: Best Practices — https://dev.to/type-guards-using-type-predicates-in-typescript
- Total TypeScript - Type Guards and Predicates — https://www.totaltypescript.com/workshops/typescript-pro-essentials/unions-and-narrowing/type-predicates
- TypeScript Documentation - Utility Types (`NonNullable`) — https://www.typescriptlang.org/docs/handbook/utility-types.html#nonnullabletype