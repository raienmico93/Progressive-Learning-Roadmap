# Advanced Type Fundamentals in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Advanced type fundamentals** are the type-level operators and constructs that let TypeScript **express new types in terms of existing types**—generics, `keyof`, `typeof`, indexed access, conditional types, `infer`, mapped types, template literal types, and utility types—combined with value-level operators like `as const` and `satisfies` that preserve literal precision. They transform TypeScript from a simple annotation language into a **type-level programming language** capable of computing, filtering, and restructuring types at compile time.

**Technical Definition.** Advanced type manipulation is built on **type operators** that take types or values as input and produce new types. TypeScript’s type system “allows expressing types in terms of other types” through generics (parameterized types), `keyof` (key union extraction), `typeof` (value-to-type extraction), indexed access (`T["key"]`), conditional types (`T extends U ? X : Y` with `infer` for capture), mapped types (`{ [K in keyof T]: ... }` with optional key remapping via `as`), and template literal types (pattern-based string types). Utility types (`Partial`, `Pick`, `Omit`, `Record`, `Exclude`, `Extract`, `NonNullable`, `ReturnType`, `Parameters`) are themselves implemented with these operators, serving as reusable transformations.

**Beginner-Friendly Explanation.** Advanced types are **how you compute types instead of writing them by hand**. Instead of manually defining every variation of a type—`PartialUser`, `ReadonlyUser`, `UserUpdate`, and so on—you use operators that **generate** those variations from a single base type. Think of it as formulas for types: `Partial<T>` means “take any type `T` and make every property optional.” Conditional types let you say “if `T` is a string, return this; otherwise return that.” Mapped types let you iterate over properties. Template literal types let you build string types from patterns. Once you learn these, you can express constraints that were previously impossible to type safely.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Type-level computation** | Types can be computed from other types, not only declared manually. |
| **Generic parameterization** | Generics abstract over types while preserving relationships between inputs and outputs. |
| **Structural transformation** | Mapped types and utility types transform object shapes property by property. |
| **Conditional logic** | Conditional types and `infer` enable branching and extraction at the type level. |
| **Literal preservation** | `as const` and `satisfies` keep exact literal types instead of widening to primitives. |
| **Distributive behavior** | Conditional types distribute over naked type parameters, enabling powerful unions transformations. |
| **Compile-time only** | All advanced type constructs are erased and emit no JavaScript. |
| **Performance-sensitive** | Deep recursion, large unions, and excessive type computation can cause measurable compilation slowdowns. |

---

### Prerequisites

- **TypeScript basics**: type annotations, interfaces, type aliases, union and intersection types.
- **Generics fundamentals**: type parameters and constraints (`<T extends U>`).
- **Literal types**: `"hello"`, `42`, `true`.
- **Control flow**: `switch`, `if`/`else`, and type narrowing.
- **Utility type awareness**: familiarity with `Partial<T>` and `Record<K, V>` as examples.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Type-level programming** — conditional types, mapped types, and recursive type aliases.
- **Generic programming** — reusable abstractions with type parameters.
- **Runtime validation** — advanced types pair with type guards for external data.
- **Library API design** — utility types and template literal types shape ergonomic APIs.
- **Compiler performance** — type complexity affects build and IDE performance.

---

## Core Concepts / Features

---

## 1. Advanced Types

### Definitions

**Core Definition.** **Advanced types** are the type-level constructs beyond primitives and simple object shapes—generics, conditional types, mapped types, template literal types, and the operators `keyof`, `typeof`, `infer`, and indexed access—that enable **type computation and transformation**.

**Technical Definition.** TypeScript’s advanced type system is built on **type operators** that manipulate other types. Generics introduce type parameters; conditional types introduce branching; mapped types introduce iteration over property keys; template literal types introduce string pattern generation. These operators compose: a conditional type can inspect a mapped type, a mapped type can iterate over a `keyof` result, and template literal types can produce literal unions consumed by conditional types.

**Beginner-Friendly Explanation.** Advanced types are **the power tools of TypeScript’s type system**. They let you write types that **adapt** to other types instead of being fixed. You use them when basic interfaces and unions cannot express the constraint you need.

### Purposes

- **To express reusable type transformations** that work across many input types.
- **To enforce complex invariants**—such as “the setter accepts a broader type than the getter returns” or “every key must have a corresponding handler.”
- **To derive types from values** so runtime constants and compile-time types stay in sync.
- **To build ergonomic library APIs** with precise inference and autocomplete.
- **To replace unsafe `any` casts** with computed, checked types.

### Syntax Rules and Structure

**Core type operators:**

| Operator | Form | Purpose |
|---|---|---|
| **Generic** | `<T>`, `<T extends U>` | Parameterize types |
| **`keyof`** | `keyof T` | Union of property keys |
| **`typeof`** | `typeof value` | Type of a value |
| **Indexed access** | `T["key"]` | Subset of a type |
| **Conditional** | `T extends U ? X : Y` | Type-level branching |
| **`infer`** | `T extends (infer U)[] ? U : never` | Capture a matched type |
| **Mapped** | `{ [K in keyof T]: ... }` | Iterate over properties |
| **Key remapping** | `{ [K in keyof T as NewKey]: ... }` | Rename properties |
| **Template literal** | `` `${T}-${U}` `` | Pattern-based string types |

**Syntax rules.**

- `keyof` produces a union of string (and number/symbol) literal keys.
- `typeof` in a type position refers to the type of a value; `typeof` in an expression position is the runtime operator.
- Conditional types **distribute** over naked type parameters: `T extends any ? ...` distributes over union members of `T`.
- `infer` may only appear inside the `extends` clause of a conditional type.
- Mapped types may use `readonly`/`?` modifiers and `-readonly`/`-?` to remove them.
- Template literal types cross-multiply unions in interpolated positions.

**Constraints and limitations.**

- Deeply recursive conditional or mapped types can hit compiler recursion limits.
- Large unions cause **quadratic** type checking; group or brand large sets.
- Conditional types can behave unexpectedly when type parameters are not naked (distributiveness is lost).

### Annotated Complete Step by Step Code Examples

#### Example 1: Conditional type with `infer`

```typescript
// example_conditional_infer.ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { id: 1, name: "John" };
}

type User = ReturnType<typeof getUser>;
// User = { id: number; name: string }

const user: User = { id: 1, name: "Ada" };
console.log(user.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The conditional type `T extends (...args: any[]) => infer R ? R : never` checks whether `T` is a function type and captures its return type in `R`. `typeof getUser` is the function type; `ReturnType` extracts `{ id: number; name: string }`.

---

#### Example 2: Mapped type with key remapping

```typescript
// example_mapped_remap.ts
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }

const getters: PersonGetters = {
  getName: () => "Ada",
  getAge: () => 36,
};

console.log(getters.getName()); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The mapped type iterates over `keyof Person`, remaps each key with a template literal type (`get${Capitalize<...>}`), and sets the value type to a function returning `T[K]`. The result is a type with getter methods for every property.

---

#### Example 3: Template literal type with union cross-multiplication

```typescript
// example_template_union.ts
type Color = "red" | "blue";
type Size = "small" | "large";

type Variant = `${Color}-${Size}`;
// "red-small" | "red-large" | "blue-small" | "blue-large"

const v: Variant = "blue-large";
console.log(v); // Expected: blue-large
```

**Expected Output:**
```
blue-large
```

**Why this output occurs.** The template literal type cross-multiplies the `Color` and `Size` unions, producing every combination. Only those exact strings are assignable to `Variant`.

---

### Real-World Cases

- **API client typing**: deriving request and response types from route definitions.
- **Form validation**: mapping field names to validation error types.
- **Event systems**: mapping event names to payload types with template literal types.
- **ORM models**: deriving create, update, and read types from a base entity type.

---

## 2. Type Manipulation

### Definitions

**Core Definition.** **Type manipulation** is the use of operators—`keyof`, `typeof`, indexed access, `as const`, and `satisfies`—to **inspect and derive types from existing types and values** without changing runtime behavior.

**Technical Definition.** Type manipulation focuses on **extracting and preserving information**. `keyof T` extracts the union of property keys. `typeof value` extracts the type of a runtime value. Indexed access `T[K]` extracts the type of a property. `as const` narrows values to literal types and makes structures readonly. `satisfies` validates a value against a type **without widening** its inferred literal types.

**Beginner-Friendly Explanation.** Type manipulation is **getting type information out of things you already have**. You have an object value; `typeof` gives you its type. You have a type; `keyof` gives you its property names. `as const` freezes exact values so they are not widened. `satisfies` checks a value against a shape while keeping its exact literal types.

### Purposes

- **To derive types from runtime values** so values and types cannot drift apart.
- **To extract key and property types** for generic utilities and APIs.
- **To preserve literal precision** without losing structural validation.
- **To avoid unsafe assertions** by validating with `satisfies` instead of `as`.
- **To build single sources of truth** where a value defines both runtime data and compile-time types.

### Syntax Rules and Structure

**Key manipulation forms:**

```typescript
type Keys = keyof Person;                 // "name" | "age"
type NameType = Person["name"];           // string
type ValueType = (typeof config)[keyof typeof config];
```

**`as const`:**

```typescript
const ROUTES = { home: "/", about: "/about" } as const;
type Route = (typeof ROUTES)[keyof typeof ROUTES]; // "/" | "/about"
```

**`satisfies`:**

```typescript
const config = {
  apiUrl: "/api",
  timeout: 5000,
} satisfies Config;
// config.apiUrl remains "/api", not string
```

**Syntax rules.**

- `keyof T` produces a union of key literal types; for types with index signatures, it may include `string | number`.
- `typeof` in type position requires a value; `typeof` on a class gives the constructor type.
- Indexed access with a union of keys produces a union of property types.
- `as const` applies to literals, arrays, and objects; it makes properties readonly and preserves literal types.
- `satisfies` was introduced in TypeScript 4.9 and works with any type expression.

**Constraints and limitations.**

- `as const` makes structures readonly, which may be undesirable for mutable data.
- `satisfies` does not change the value's type; it only checks compatibility.
- `keyof` on a type with an index signature is broad (`string | number`) and loses specific key information.

### Annotated Complete Step by Step Code Examples

#### Example 1: `as const` + `typeof` + `keyof` for a route table

```typescript
// example_routes.ts
const ROUTES = {
  home: "/",
  about: "/about",
  contact: "/contact",
} as const;

type RoutePath = (typeof ROUTES)[keyof typeof ROUTES];
// "/" | "/about" | "/contact"

type RouteName = keyof typeof ROUTES;
// "home" | "about" | "contact"

function navigate(path: RoutePath) {
  console.log(`Navigating to ${path}`);
}

navigate(ROUTES.about); // Expected: Navigating to /about
// navigate("/unknown"); // ❌ Error: not assignable to RoutePath.
```

**Expected Output:**
```
Navigating to /about
```

**Why this output occurs.** `as const` preserves the literal string values and makes the object readonly. `(typeof ROUTES)[keyof typeof ROUTES]` extracts the union of values. `keyof typeof ROUTES` extracts the union of keys. The value and its derived types share a single source of truth.

---

#### Example 2: `satisfies` validates without widening

```typescript
// example_satisfies.ts
interface Config {
  apiUrl: string;
  timeout: number;
}

// ❌ Annotation widens literals.
const wide: Config = { apiUrl: "/api", timeout: 5000 };
// wide.apiUrl: string

// ✅ satisfies preserves literals and validates.
const narrow = { apiUrl: "/api", timeout: 5000 } satisfies Config;
// narrow.apiUrl: "/api"

console.log(narrow.apiUrl); // Expected: /api
```

**Expected Output:**
```
/api
```

**Why this output occurs.** The annotation `: Config` widens `"/api"` to `string`. `satisfies Config` checks the shape without changing the inferred type, so `narrow.apiUrl` remains the literal `"/api"`.

---

#### Example 3: Indexed access with `keyof`

```typescript
// example_indexed_access.ts
interface User {
  id: string;
  name: string;
  age: number;
}

type UserValue = User[keyof User]; // string | number
type NameType = User["name"];      // string

const values: UserValue[] = ["u1", "Ada", 36];
console.log(values.length); // Expected: 3
```

**Expected Output:**
```
3
```

**Why this output occurs.** `User[keyof User]` indexes `User` with the union `"id" | "name" | "age"`, producing `string | number`. Indexing with a single key produces that property's type.

---

### Real-World Cases

- **Configuration objects**: `as const` + `satisfies` for validated, literal-preserving config.
- **Route tables**: deriving path and name unions from a single route map.
- **Theme systems**: extracting design token keys and values from a `const` token object.
- **API route definitions**: deriving request and response types from route handler values.

---

## 3. Type Transformation

### Definitions

**Core Definition.** **Type transformation** is the use of **utility types**, **mapped types**, and **conditional types** to **create new types by structurally modifying existing ones**—making properties optional, readonly, picked, omitted, or conditionally included.

**Technical Definition.** Type transformation operators take an input type and produce a **new type with a modified structure**. Utility types (`Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record`, `Exclude`, `Extract`, `NonNullable`) are built-in transformations implemented with mapped and conditional types. Custom transformations use mapped types for property-level changes and conditional types for union-level filtering and extraction.

**Beginner-Friendly Explanation.** Type transformation is **reshaping a type into a new type**. You have a `User` type; `Partial<User>` makes every field optional. `Pick<User, "id" | "name">` keeps only those fields. `Exclude` removes union members. It is how you avoid writing the same shape five times.

### Purposes

- **To eliminate duplicated type definitions** by deriving variations from a base type.
- **To model update payloads** with `Partial<T>` for partial updates.
- **To restrict exposed fields** with `Pick<T, K>` or `Omit<T, K>`.
- **To enforce immutability** with `Readonly<T>`.
- **To filter unions** with `Exclude<T, U>` and `Extract<T, U>`.
- **To build precise API contracts** from a single domain model.

### Syntax Rules and Structure

**Built-in utility transformations:**

| Utility | Transformation |
|---|---|
| `Partial<T>` | All properties optional |
| `Required<T>` | All properties required |
| `Readonly<T>` | All properties readonly |
| `Pick<T, K>` | Only properties `K` |
| `Omit<T, K>` | All except properties `K` |
| `Record<K, V>` | Object with keys `K` and values `V` |
| `Exclude<T, U>` | Union members of `T` not assignable to `U` |
| `Extract<T, U>` | Union members of `T` assignable to `U` |
| `NonNullable<T>` | `T` without `null` and `undefined` |
| `ReturnType<T>` | Return type of a function type |
| `Parameters<T>` | Parameter tuple of a function type |

**Custom mapped transformation:**

```typescript
type Nullable<T> = { [K in keyof T]: T[K] | null };
type Mutable<T> = { -readonly [K in keyof T]: T[K] };
type Concrete<T> = { [K in keyof T]-?: T[K] };
```

**Syntax rules.**

- Mapped types use `[K in keyof T]` to iterate over keys.
- Modifiers `readonly` and `?` may be added or removed with `-readonly` and `-?`.
- Key remapping with `as` allows renaming or filtering keys.
- Conditional types distribute over naked type parameters.
- Utility types are erased and produce no runtime code.

**Constraints and limitations.**

- `Partial<T>` makes all properties optional, including nested ones? **No—it is shallow**; nested objects remain required.
- `Omit<T, K>` does not enforce that `K` exists on `T`; typos are silently accepted unless a stricter utility is used.
- Deep transformations (`DeepPartial`, `DeepReadonly`) require recursive mapped types and can hit recursion limits.

### Annotated Complete Step by Step Code Examples

#### Example 1: `Partial` and `Omit` for an update payload

```typescript
// example_update_payload.ts
interface User {
  id: string;
  name: string;
  email: string;
}

// ❌ Verbose manual type.
// interface UserUpdate {
//   name?: string;
//   email?: string;
// }

// ✅ Derived update payload: omit id, make rest optional.
type UserUpdate = Partial<Omit<User, "id">>;

const update: UserUpdate = { name: "Ada" };
console.log(update); // Expected: { name: 'Ada' }
```

**Expected Output:**
```
{ name: 'Ada' }
```

**Why this output occurs.** `Omit<User, "id">` removes `id`; `Partial` makes the remaining properties optional. The update payload cannot accidentally include or change `id`, and every other field is optional.

---

#### Example 2: `Pick` and `Record` for a public API

```typescript
// example_public_api.ts
interface User {
  id: string;
  name: string;
  email: string;
  passwordHash: string;
}

type PublicUser = Pick<User, "id" | "name">;
type UserMap = Record<string, PublicUser>;

const users: UserMap = {
  u1: { id: "u1", name: "Ada" },
  u2: { id: "u2", name: "Grace" },
};

console.log(Object.keys(users).length); // Expected: 2
```

**Expected Output:**
```
2
```

**Why this output occurs.** `Pick` exposes only `id` and `name`, hiding `email` and `passwordHash`. `Record<string, PublicUser>` types a map keyed by user ID.

---

#### Example 3: `Exclude` and `Extract` for union filtering

```typescript
// example_union_filter.ts
type AllEvents = "click" | "focus" | "blur" | "submit" | "reset";
type MouseEvents = Extract<AllEvents, "click" | "focus" | "blur">;
// "click" | "focus" | "blur"

type NonMouseEvents = Exclude<AllEvents, MouseEvents>;
// "submit" | "reset"

function handleMouse(event: MouseEvents) {
  console.log(event);
}

handleMouse("click"); // Expected: click
// handleMouse("submit"); // ❌ Error: not assignable to MouseEvents.
```

**Expected Output:**
```
click
```

**Why this output occurs.** `Extract` keeps only the union members assignable to the second argument. `Exclude` removes them. The result is two disjoint unions, each type-safe.

---

### Real-World Cases

- **DTO derivation**: creating `CreateUser`, `UpdateUser`, and `PublicUser` from a base `User`.
- **API response shaping**: omitting sensitive fields with `Omit`.
- **Permission models**: filtering allowed actions with `Exclude`.
- **Form state**: using `Partial` for dirty-field tracking.

---

## 4. Advanced Type Best Practices

### Definitions

**Core Definition.** **Advanced type best practices** are guidelines for using TypeScript’s type-level operators in ways that maximize **safety, readability, and compiler performance**, while avoiding excessive complexity, recursive explosions, and unsafe widening.

**Technical Definition.** Best practices center on **keeping types simple and composable**, **using type aliases for reusable patterns**, **leveraging inference instead of over-annotating**, **avoiding expensive type computation on every use**, **optimizing union sizes**, and **using `satisfies` instead of `as` for validation**. Performance guidance warns that **union checking is quadratic** and unions with 50+ members cause measurable compilation slowdowns, while deeply recursive conditional types can hit recursion limits and freeze IDEs.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep advanced types from becoming a performance or readability disaster. The big rules are: keep types simple and composable; use aliases to avoid repetition; let TypeScript infer when it can; avoid huge unions and deeply nested conditional types; and use `satisfies` instead of `as` when you want validation without losing precision.

### Purposes

- **To maximize compiler performance** by avoiding expensive type computations.
- **To keep advanced types readable** and maintainable for teams.
- **To preserve type safety** by using `satisfies` and precise utilities instead of assertions.
- **To prevent recursion and instantiation-depth errors** in complex types.
- **To avoid quadratic union checking** that slows builds and IDEs.

### Syntax Rules and Structure

Best practices are conventions that interact with advanced type syntax. The core rules are:

1. **Keep types simple and composable.** Avoid deeply nested conditional types; break complex transformations into smaller named types. A recursively nested `Complex<T>` is harder to read and slower than iterative `ElementType` aliases.
2. **Use type aliases for reusability.** Define `ID`, `Timestamp`, and `Optional<T>` once and reuse them.
3. **Leverage inference.** Let TypeScript infer types when possible; over-annotation can widen or obscure precise types.
4. **Avoid type computation overhead on every use.** Precompute expensive transformations once and reuse the alias.
5. **Optimize union types.** Union checking is **quadratic**; unions with 50+ members cause slowdowns. Group large unions into discriminated categories or use branded types.
6. **Use `satisfies` instead of `as` for validation.** `satisfies` validates without widening; `as` suppresses checks.
7. **Prefer precise index signatures over `any`.** Use `Record<K, V>` with known keys, not `[key: string]: any`.
8. **Watch recursion depth.** Deep recursive mapped or conditional types can trigger “Type instantiation is excessively deep” errors and IDE lag.

**Component breakdown (advanced type decision tree):**

```
Do you need to derive a new type from an existing one?
  ├── Yes → Can a built-in utility type do it?
  │        ├── Yes → Use Partial / Pick / Omit / Record / Exclude / Extract.
  │        └── No → Write a mapped or conditional type.
  │                 ├── Keep it simple and composable.
  │                 └── Avoid deep recursion and large unions.
  └── No → Do you need to validate a value without widening?
           ├── Yes → Use satisfies.
           └── No → Use a plain type annotation.
```

**Constraints and limitations.**

- Even well-structured advanced types can be slow if unions are huge.
- `satisfies` does not replace runtime validation for external data.
- Recursive types require clear base cases and depth limits.

### Annotated Complete Step by Step Code Examples

#### Example 1: Composable type aliases over deep nesting

```typescript
// example_best_composable.ts
// ❌ Deeply nested conditional type — hard to read and slow.
// type Complex<T> = T extends Array<infer U>
//   ? U extends Array<infer V>
//     ? V extends Array<infer W>
//       ? W
//       : never
//     : never
//   : never;

// ✅ Composable iterative aliases.
type ElementType<T> = T extends (infer E)[] ? E : T;
type Deep1<T> = ElementType<T>;
type Deep2<T> = ElementType<Deep1<T>>;

type Result = Deep2<string[][]>; // string
const value: Result = "hello";
console.log(value); // Expected: hello
```

**Expected Output:**
```
hello
```

**Why this output occurs.** The iterative `ElementType` alias is easier to read and compose than a single deeply nested conditional. Each step is named and reusable.

---

#### Example 2: Optimizing a large union with discriminated categories

```typescript
// example_best_large_union.ts
// ❌ Large flat union — quadratic checking.
// type Event = "user_login" | "user_logout" | ... (100+ members);

// ✅ Group into discriminated categories.
type UserEvent = { category: "user"; action: "login" | "logout" };
type PageEvent = { category: "page"; action: "view" | "leave" };

type AppEvent = UserEvent | PageEvent;

function handle(event: AppEvent) {
  switch (event.category) {
    case "user": console.log(event.action); break;
    case "page": console.log(event.action); break;
  }
}

handle({ category: "user", action: "login" }); // Expected: login
```

**Expected Output:**
```
login
```

**Why this output occurs.** The outer union has only two members, avoiding quadratic checking. Each category narrows to a small action union.

---

#### Example 3: `satisfies` over `as` for validation

```typescript
// example_best_satisfies.ts
interface Config {
  apiUrl: string;
  timeout: number;
}

// ❌ as assertion: no validation, widens type.
// const bad = { apiUrl: "/api", timeout: 5000 } as Config;

// ✅ satisfies: validates and preserves literals.
const config = { apiUrl: "/api", timeout: 5000 } satisfies Config;
console.log(config.apiUrl); // Expected: /api
```

**Expected Output:**
```
/api
```

**Why this output occurs.** `satisfies Config` checks the shape while preserving `apiUrl` as the literal `"/api"`. `as Config` would widen it to `string` and skip the structural check.

---

### Real-World Cases

- **Library utility types**: composing `Partial<Omit<T, K>>` for update payloads instead of manual duplicates.
- **Large generated APIs**: grouping GraphQL or database enum unions into discriminated categories to avoid quadratic checking.
- **Configuration validation**: `satisfies` for config objects that must match a schema while preserving literal route and token values.
- **Performance-sensitive codebases**: precomputing complex type transformations once and reusing the aliases.

---

## References

- TypeScript Handbook — Creating Types from Types - https://www.typescriptlang.org/docs/handbook/2/types-from-types.html
- TypeScript Handbook — Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook — Conditional Types - https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Handbook — Mapped Types - https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- TypeScript Handbook — Template Literal Types - https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- TypeScript Handbook — Utility Types - https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript 4.9 Release Notes — `satisfies` Operator - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html
- TypeScript 3.4 Release Notes — `const` Assertions - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html
- Performance Best Practices — Claude-Cortex TypeScript Advanced Patterns - https://github.com/NickCrew/Claude-Cortex/blob/main/skills/typescript-advanced-patterns/references/performance-best-practices.md
- Avoid Large Union Types — pproenca/dot-skills - https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.curated/typescript/references/type-avoid-large-unions.md
- TypeScript Advanced Types Skill — svssdeva/agentic-skills - https://github.com/svssdeva/agentic-skills/blob/main/typescript/typescript-advanced-types/SKILL.md