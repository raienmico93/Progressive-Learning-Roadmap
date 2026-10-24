# `satisfies` Operator in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** The **`satisfies` operator** is a TypeScript 4.9+ expression-level operator that **validates an expression's type against a target type without changing the expression's inferred type**. It resolves the long-standing dilemma between **type annotations (`: T`)**, which validate structure but **widen literal types**, and **type assertions (`as T`)**, which preserve inferred types but **skip validation entirely**.

**Technical Definition.** The `satisfies` operator checks that the **type of the left-hand expression is assignable to the type on the right-hand side**, while leaving the **inferred type of the expression unchanged**. It performs full **structural validation**, including excess property checks for object literals, but the variable retains the **narrowest possible inferred type**—including literal types, readonly modifiers from `as const`, and tuple structures. It is **erased at compile time** and emits no runtime JavaScript.

**Beginner-Friendly Explanation.** `satisfies` is **the “check my work but don’t change my answer” operator**. You use it when you want TypeScript to **validate that a value matches a shape** (catching typos, missing fields, and wrong types) while **keeping the exact literal values** you wrote—so `"localhost"` stays `"localhost"` instead of becoming `string`. It is the missing middle ground between type annotations and type assertions.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Literal preservation** | The inferred type of the expression is **not widened**; literal types and readonly structures remain. |
| **Structural validation** | The expression must be assignable to the target type, including required properties and excess property checks for fresh literals. |
| **Expression-level operator** | Unlike annotations, `satisfies` applies to **values**, not declarations; it can be used in any expression position. |
| **Composability with `as const`** | `as const satisfies T` combines readonly literal preservation with structural validation. |
| **No runtime effect** | `satisfies` is erased and produces no JavaScript. |
| **TypeScript 4.9+** | The operator requires TypeScript 4.9 or later. |
| **Allows extra properties** | Unlike annotations, `satisfies` permits properties beyond the target type’s declaration, preserving them in the inferred type. |

---

### Prerequisites

- **TypeScript basics**: type annotations, type aliases, interfaces, and object literals.
- **Literal types**: `"hello"`, `42`, `true`.
- **Type inference and widening**: how `let`/`const` and annotations affect inferred types.
- **Type assertions (`as`)** : their purpose and their safety pitfalls.
- **`as const` assertions**: readonly and literal preservation.
- **A TypeScript environment** (4.9+; `tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Type annotations (`:`)** — the widening alternative that `satisfies` complements.
- **Type assertions (`as`)** — the unsafe alternative that `satisfies` replaces for validation.
- **`as const`** — the readonly literal-preservation assertion that pairs with `satisfies`.
- **Type inference and widening** — the behavior `satisfies` is designed to preserve.
- **Runtime validation** — libraries like Zod for external data, which `satisfies` does not replace.

---

## Core Concepts / Features

---

## 1. `satisfies` Operator

### Definitions

**Core Definition.** The **`satisfies` operator** validates that an expression matches a target type while **preserving the expression’s most specific inferred type**. It is a **compile-time check** that does not alter the value or its inferred type.

**Technical Definition.** The syntax `expression satisfies Type` checks that `expression` is assignable to `Type`. If the check passes, the result is `expression` with its **original inferred type**; if it fails, a compile-time error is reported at the offending property. The operator supports **excess property checks** for object literals, **missing property detection**, and **value-type conformance**, while retaining literal types, tuple structures, and readonly modifiers from `as const`.

**Beginner-Friendly Explanation.** `satisfies` means **“make sure this fits the shape, but remember exactly what I wrote.”** You use it when you want validation without losing the precision of your literal values.

### Purposes

- **To validate a value against a type without widening** its literal types.
- **To catch typos in object keys** while preserving exact key information.
- **To ensure an object has all required keys** of a type without losing extra properties.
- **To preserve literal types for downstream inference**, autocomplete, and type narrowing.
- **To replace unsafe `as` assertions** with compiler-verified structural checks.
- **To combine readonly literal preservation with validation** using `as const satisfies T`.

### Syntax Rules and Structure

**General syntax:**

```
expression satisfies Type
```

**Component breakdown:**

- `expression` — the value being validated (object literal, array, function, etc.).
- `satisfies` — the operator keyword.
- `Type` — the target type to validate against.
- The result is `expression` with its **inferred type**, not `Type`.

**Syntax rules.**

- The expression must be **assignable** to the target type.
- For **fresh object literals**, excess property checks apply: properties not present in `Type` cause errors unless `Type` has an index signature.
- The inferred type of the variable remains the **narrowest possible type** of the expression.
- `satisfies` may be combined with `as const`: `expression as const satisfies Type`.
- `satisfies` may be used in variable declarations, return expressions, and function arguments.

**Constraints and limitations.**

- `satisfies` performs **no runtime validation**; it is purely a compile-time check.
- It cannot validate **values constructed at runtime** from external input (use runtime validators like Zod).
- Excess properties are allowed if the target type has an index signature, which may weaken typo detection.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic object validation

```typescript
// example_satisfies_basic.ts
// Step 1: Define a target type.
type Colors = "red" | "green" | "blue";
type RGB = [number, number, number];

// Step 2: Use satisfies to validate the palette.
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  bleu: [0, 0, 255], // ❌ Typo caught by satisfies!
} satisfies Record<Colors, string | RGB>;

// Step 3: The inferred type preserves literals.
const greenNormalized = palette.green.toUpperCase(); // ✅ Works: green is string
```

**Setup.** Compile with `npx tsc example_satisfies_basic.ts --strict` (TypeScript 4.9+).

**Expected behavior.** The `bleu` typo is caught at compile time because `Record<Colors, ...>` requires exactly the keys `"red"`, `"green"`, and `"blue"`. The inferred type of `palette.green` remains `string`, enabling `.toUpperCase()`.

**Why this occurs.** `satisfies` validates the object against `Record<Colors, string | RGB>` while preserving the specific inferred types of each property. Unlike an annotation, it does not widen `green` to `string | RGB`, so `.toUpperCase()` remains available.

---

#### Example 2: Preserving literal types

```typescript
// example_satisfies_literals.ts
interface ApiConfig {
  host: string;
  port: number;
}

// ❌ Annotation widens literal types.
const annotated: ApiConfig = { host: "localhost", port: 8080 };
// annotated.host is string; annotated.port is number.

// ✅ satisfies preserves literal types.
const validated = { host: "localhost", port: 8080 } satisfies ApiConfig;
// validated.host is "localhost"; validated.port is 8080.

type Environment = "localhost" | "staging" | "production";

const env: Environment = validated.host; // ✅ Works: "localhost" is assignable.
// const env2: Environment = annotated.host; // ❌ Error: string is not assignable.
```

**Expected behavior.** `validated.host` is the literal `"localhost"`, allowing assignment to the `Environment` union. The annotated version widens `host` to `string`, causing an error.

**Why this occurs.** `satisfies` validates the shape without overriding the inferred type. The annotation `: ApiConfig` explicitly widens the properties to their declared types.

---

#### Example 3: Allowing extra properties

```typescript
// example_satisfies_extra.ts
interface Animal {
  name: string;
  sound: string;
}

const dog = {
  name: "Buddy",
  sound: "Woof",
  breed: "Golden Retriever", // Extra property allowed with satisfies.
} satisfies Animal;

console.log(dog.breed); // ✅ Works: breed is preserved in the inferred type.
```

**Expected behavior.** `dog` is validated against `Animal`, but the extra `breed` property is preserved in the inferred type, enabling `dog.breed` access.

**Why this occurs.** `satisfies` allows excess properties when the expression is not a fresh literal assigned to an annotated variable. The inferred type includes all properties of the literal.

---

### Real-World Cases

- **Configuration objects**: validating API, server, or theme configuration while preserving literal host, port, and token values.
- **Route tables**: ensuring route paths start with `/` while keeping literal path types for autocomplete.
- **Palette and design tokens**: validating color values while preserving hex literal types.
- **Feature flag maps**: ensuring all flags are boolean while preserving exact flag names.

---

## 2. Satisfies Usage

### Definitions

**Core Definition.** **`satisfies` usage** refers to applying the operator in configuration objects, route maps, palette definitions, function maps, and other static data to validate structure while preserving precise types.

**Technical Definition.** `satisfies` is used in **variable declarations** (`const x = expr satisfies T`), **return expressions** (`return expr satisfies T`), and **combinations with `as const`** (`expr as const satisfies T`). It is especially valuable with `Record<K, V>` for key validation, arrays for element validation, and function maps for signature validation. It may also be combined with **branded types** to enforce additional invariants.

**Beginner-Friendly Explanation.** You use `satisfies` **wherever you have static data that must match a shape but whose exact values matter**—configs, routes, palettes, and lookup tables.

### Purposes

- **To validate configuration objects** against a schema while preserving literal values.
- **To validate route tables** and derive literal path unions with `typeof`.
- **To validate palettes and theme tokens** while preserving hex literal types.
- **To validate function maps** and handler signatures.
- **To combine with `as const`** for readonly, literal-preserving, validated configuration.
- **To combine with branded types** for validated configuration with proof of invariants.

### Syntax Rules and Structure

**Basic usage:**

```typescript
const config = { ... } satisfies Config;
```

**With `as const`:**

```typescript
const config = { ... } as const satisfies Config;
```

**With `Record`:**

```typescript
const routes = { ... } satisfies Record<string, `/${string}`>;
```

**With branded types:**

```typescript
const routes = { ... } satisfies Record<string, { method: HttpMethod; path: ValidatedPath }>;
```

**Syntax rules.**

- `as const satisfies T` applies readonly literal preservation before validation.
- `satisfies` may be used with `Record<K, V>` to enforce exact keys.
- `satisfies` may be used with array types to validate every element.
- Branded types can be combined with `satisfies` for runtime-validated invariants.

**Constraints and limitations.**

- `satisfies` does not validate **runtime input**; use runtime validators for external data.
- Excess properties are allowed unless the target type has no index signature and the expression is a fresh literal.

### Annotated Complete Step by Step Code Examples

#### Example 1: Configuration object with `satisfies`

```typescript
// example_usage_config.ts
interface ServerConfig {
  host: string;
  port: number;
  timeout: number;
}

const config = {
  host: "localhost",
  port: 8080,
  timeout: 3000,
} satisfies ServerConfig;

// config.host is "localhost" (literal)
// config.port is 8080 (literal)

console.log(config.host); // Expected: localhost
```

**Expected Output:**
```
localhost
```

**Why this output occurs.** `satisfies ServerConfig` validates the shape while preserving the literal types of `host`, `port`, and `timeout`. The configuration is validated but not widened.

---

#### Example 2: Route table with `as const satisfies`

```typescript
// example_usage_routes.ts
const ROUTES = {
  home: "/",
  user: "/user/:id",
  settings: "/settings",
} as const satisfies Record<string, `/${string}`>;

type RouteName = keyof typeof ROUTES; // "home" | "user" | "settings"
type RoutePath = (typeof ROUTES)[RouteName]; // "/" | "/user/:id" | "/settings"

const path: RoutePath = "/user/:id";
console.log(path); // Expected: /user/:id
```

**Expected Output:**
```
/user/:id
```

**Why this output occurs.** `as const` preserves the literal path values and makes the object readonly. `satisfies Record<string, `/${string}`>` validates that every value starts with `/`. The derived types `RouteName` and `RoutePath` retain precise literal information.

---

#### Example 3: Function map with `satisfies`

```typescript
// example_usage_functions.ts
type Handler = (body: unknown) => Promise<Response>;

const api = {
  GET: (query: Record<string, string>) => Promise.resolve(new Response("GET")),
  POST: (body: unknown) => Promise.resolve(new Response("POST")),
} satisfies Record<string, Handler>;

console.log(typeof api.GET); // Expected: function
```

**Expected Output:**
```
function
```

**Why this output occurs.** `satisfies Record<string, Handler>` validates that every property is a function matching the `Handler` signature, while preserving the specific parameter types of each handler.

---

#### Example 4: Branded types with `satisfies`

```typescript
// example_usage_branded.ts
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type ValidatedPath = string & { readonly __brand: "ValidatedPath" };

function path<P extends `/${string}`>(p: P): P & ValidatedPath {
  return p as P & ValidatedPath;
}

const routes = {
  listUsers: { method: "GET", path: path("/users") },
  newUser: { method: "POST", path: path("/users") },
} satisfies Record<string, { method: HttpMethod; path: ValidatedPath }>;

console.log(routes.listUsers.method); // Expected: GET
```

**Expected Output:**
```
GET
```

**Why this output occurs.** `satisfies` validates the shape and the branded `ValidatedPath` constraint. The brand carries proof that the path was validated, while `satisfies` preserves the literal `"GET"` method type and the branded path type.

---

### Real-World Cases

- **Configuration files**: `const config = { ... } as const satisfies Config` for validated, readonly, literal-preserving configuration.
- **Route maps**: validating API route prefixes while deriving literal path unions.
- **Theme definitions**: validating color token values while preserving hex literals.
- **API client method maps**: validating handler signatures while preserving parameter types.

---

## 3. Satisfies Best Practices

### Definitions

**Core Definition.** **`satisfies` best practices** are guidelines for choosing between type annotations, type assertions, and `satisfies` based on whether the goal is **validation, literal preservation, or both**.

**Technical Definition.** Best practices center on **using `satisfies` for validation without widening**, **using annotations when the variable should be exactly the target type**, **using `as const satisfies` for readonly literal configuration**, and **never using `as` for validation**. The operator solves the “annotation widens, assertion skips validation” dilemma: it **validates structure while preserving the narrowest inferred type**.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to choose the right type tool. The big rules are: use `satisfies` when you want validation but need literal types; use `as const satisfies` when you also want readonly; and use annotations when you want the variable to be exactly the target type.

### Purposes

- **To preserve literal types** for downstream inference while validating structure.
- **To catch typos and missing properties** without widening.
- **To combine readonly preservation with validation** using `as const satisfies`.
- **To avoid unsafe `as` assertions** that skip validation.
- **To choose the right tool** for each use case based on whether widening or validation is desired.

### Syntax Rules and Structure

**Comparison table:**

| Approach | Validation | Literal Preservation | Readonly | Use Case |
|---|---|---|---|---|
| **`: Type` (annotation)** | ✅ Yes | ❌ No (widens) | ❌ No | Variable should be exactly `Type` |
| **`as Type` (assertion)** | ❌ No | ❌ No | ❌ No | Last resort; no runtime check |
| **`satisfies Type`** | ✅ Yes | ✅ Yes | ❌ No | Validation without widening |
| **`as const satisfies Type`** | ✅ Yes | ✅ Yes | ✅ Yes | Validated readonly literal config |

**Best-practice rules.**

1. **Use `satisfies` for validation without widening.** Prefer it over annotations when literal types matter for downstream inference.
2. **Use `as const satisfies` for readonly literal configuration.** This combines readonly preservation with structural validation.
3. **Use type annotations (`: Type`) when the variable should be exactly the target type.** Annotations define the contract for the variable’s lifetime and prevent reassignment to incompatible values.
4. **Never use `as` for validation.** Assertions skip runtime checks and can hide bugs; use `satisfies` instead.
5. **Do not use `satisfies` for runtime validation.** It is compile-time only; validate external input with runtime validators like Zod.
6. **Combine with `Record<K, V>` for exact-key validation.** `satisfies Record<Keys, Value>` catches typos and missing keys.
7. **Use `satisfies` with branded types for additional invariants.** Brands can carry proof of runtime validation through the type system.

**Component breakdown (type tool decision tree):**

```
Do you need to validate a value against a type?
  ├── Yes → Do literal types matter downstream?
  │        ├── Yes → Use `satisfies Type` (or `as const satisfies` for readonly).
  │        └── No → Use a type annotation (`: Type`).
  └── No → Do you have external proof the value is correct?
           ├── Yes → Use `as Type` (last resort).
           └── No → Do not use a type tool; validate at runtime.
```

**Constraints and limitations.**

- `satisfies` does not replace runtime validation for untrusted data.
- `as const satisfies` makes the value readonly; use plain `satisfies` if mutation is needed.
- Excess properties are allowed with `satisfies` unless the target type has no index signature and the expression is a fresh literal.

### Annotated Complete Step by Step Code Examples

#### Example 1: `satisfies` vs. annotation

```typescript
// example_best_satisfies_vs_annotation.ts
interface Config {
  apiUrl: string;
  timeout: number;
}

// ❌ Annotation widens literals.
const wide: Config = { apiUrl: "/api", timeout: 5000 };
// wide.apiUrl is string.

// ✅ satisfies preserves literals.
const narrow = { apiUrl: "/api", timeout: 5000 } satisfies Config;
// narrow.apiUrl is "/api".

console.log(narrow.apiUrl); // Expected: /api
```

**Expected Output:**
```
/api
```

**Why this output occurs.** The annotation `: Config` widens `apiUrl` to `string`. `satisfies Config` validates the shape while preserving the literal `"/api"`.

---

#### Example 2: `as const satisfies` for readonly configuration

```typescript
// example_best_as_const_satisfies.ts
const ROUTES = {
  home: "/",
  user: "/user/:id",
} as const satisfies Record<string, `/${string}`>;

// ROUTES.home is "/" (literal), readonly.
// ROUTES.user is "/user/:id" (literal), readonly.

// ROUTES.home = "/other"; // ❌ Error: readonly.

console.log(ROUTES.home); // Expected: /
```

**Expected Output:**
```
/
```

**Why this output occurs.** `as const` preserves the literal path values and makes the object readonly. `satisfies` validates that every path starts with `/`. The combination provides both validation and readonly literal preservation.

---

#### Example 3: Avoiding `as` for validation

```typescript
// example_best_avoid_as.ts
interface User {
  id: string;
  name: string;
}

// ❌ as assertion: no validation, typo not caught.
// const bad = { id: "u1", nmae: "Ada" } as User;

// ✅ satisfies: typo caught at compile time.
const good = { id: "u1", name: "Ada" } satisfies User;

console.log(good.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `satisfies User` performs excess property checks and catches the `nmae` typo. `as User` would silently accept the typo and suppress the error.

---

### Real-World Cases

- **Code review guidelines**: preferring `satisfies` over `as` for config validation and over annotations when literal types matter.
- **Configuration management**: `as const satisfies` for validated, readonly, literal-preserving configuration.
- **Library design**: using `satisfies` to validate exported constants without widening their types.
- **Runtime vs. compile-time**: using runtime validators (Zod, io-ts) for external input and `satisfies` for static internal data.

---

## References

- Documentation - TypeScript 4.9: The `satisfies` Operator — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html#the-satisfies-operator
- TypeScript Playground - `satisfies` Operator Example — https://www.typescriptlang.org/play/typescript/language-extensions/satisfies-operator.ts.html
- Refine - How to Use the TypeScript `satisfies` Operator — https://refine.dev/blog/typescript-satisfies-operator/
- Convex - Understanding TypeScript's `satisfies` Operator — https://www.convex.dev/typescript/advanced/type-operators-manipulation/typescript-satisfies
- dot-skills - Use `satisfies` for Type Validation with Inference — https://github.com/pproenca/dot-skills/blob/master/skills/.curated/typescript/references/advanced-satisfies-operator.md
- dot-skills - Prefer `satisfies` Over `as` — https://github.com/ryoppippi/dotfiles/blob/master/agents/skills/typescript-style/SKILL.md
- dot-skills - Combine `satisfies` with Branded Types for Validated Configuration — https://raw.githubusercontent.com/pproenca/dot-skills/cf93c57cac89d6fc3e4194686000411567f5caf3/skills/.experimental/typescript-advanced-patterns/references/mod-satisfies-branded-config.md
- Educative - Validating Shape and Preserving Literals with `satisfies` — https://www.educative.io/courses/lta/learn-typescript/validating-shape-and-preserving-literals-with-satisfies
- TypeScript 4.9 Release Notes - `satisfies` Operator (Official) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html
- TypeScript Playground - `satisfies` with Configuration Objects — https://www.typescriptlang.org/play#example/satisfies-operator