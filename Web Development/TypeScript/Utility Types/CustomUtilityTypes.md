# Custom Utility Types in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **custom utility type** is a **user-defined generic type alias** that encapsulates a reusable **type-level transformation** not provided by TypeScript's built-in utility library. Custom utility types apply the same implementation patterns as built-ins—**mapped types**, **conditional types**, **`keyof`**, **indexed access** (`T[K]`), **`infer`**, and **template literal types**—but address project-specific needs such as deep transformations (`DeepPartial`, `DeepReadonly`), key remapping (`Getters<T>`, `Setters<T>`), string manipulations (`CamelCase<S>`, `KebabCase<S>`), and domain-specific derivations. They are **compile-time only**, **erased at runtime**, and **subject to the compiler's recursion and union-complexity limits**.

**Technical Definition.** Custom utility types are **type aliases with generic parameters** whose bodies compute new types using TypeScript's type operators. They follow the same structural patterns as built-ins: object utilities use **homomorphic mapped types** (`{ [K in keyof T]: ... }`) to preserve modifiers and tuple/array structure; union utilities use **distributive conditional types**; extraction utilities use **`infer`**; string utilities compose from the four **intrinsic string manipulation types**. Well-designed custom utilities are **small, focused, composable, and cached separately** by the compiler, whereas monolithic utilities with multiple conditional branches cause **quadratic type checking** and slow compilation.

**Beginner-Friendly Explanation.** Custom utility types are **utilities you write yourself** when TypeScript's built-in ones do not cover your needs. They let you build **project-specific type formulas**—like "make every nested property optional" (`DeepPartial`), "generate getter methods from property names" (`Getters`), or "convert snake_case strings to camelCase" (`CamelCase`). Once you write one, you can reuse it across your entire codebase, and it will stay synchronized with your base types automatically.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **User-defined** | Custom utilities are written by developers; they are not part of TypeScript's standard library. |
| **Generic and parameterized** | They accept type parameters, often with constraints (`K extends keyof T`) and defaults. |
| **Composable** | They can be chained with built-in utilities and other custom utilities. |
| **Domain-specific** | They encode project-specific transformations (deep immutability, getter generation, naming conventions). |
| **Erasure** | All custom utilities are compile-time only and emit no JavaScript. |
| **Performance-sensitive** | Deep recursion and monolithic mapped types cause exponential type expansion and IDE lag. |
| **Version-dependent** | Some patterns require specific TypeScript versions (recursive conditional types in 4.1+, `satisfies` in 4.9+, `infer extends` in 4.7+). |

---

### Prerequisites

- **TypeScript basics**: type aliases, interfaces, unions, and intersections.
- **Generics fundamentals**: type parameters, constraints, and defaults.
- **The `keyof` operator** and **indexed access types** (`T[K]`).
- **Mapped types**: `[K in keyof T]`, modifiers (`readonly`, `?`, `-readonly`, `-?`), and key remapping (`as`).
- **Conditional types** and the **`infer` keyword**.
- **Template literal types** and intrinsic string utilities.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Built-in utility types** — the standard library that custom utilities complement.
- **Mapped types** — the implementation mechanism for object utilities.
- **Conditional types** — the implementation mechanism for filtering and extraction utilities.
- **Recursive types** — the mechanism for deep transformations.
- **Type-level programming** — the broader discipline of computing types.
- **Compiler performance** — utility complexity affects build and IDE speed.

---

## Core Concepts / Features

---

## 1. Custom Utility Types

### Definitions

**Core Definition.** A **custom utility type** is a **reusable, generic type alias** that performs a type-level transformation not covered by TypeScript's built-in utilities.

**Technical Definition.** Custom utilities are **type aliases with generic parameters** that compute new types using mapped types, conditional types, `infer`, and template literal types. They may be **shallow** (operating on one level) or **recursive** (operating on nested structures). They are **erased** at compile time and are resolved during type checking.

**Beginner-Friendly Explanation.** A custom utility type is **a type formula you write yourself**. It takes one or more types as input and produces a new type as output, following the same patterns as `Partial<T>` and `ReturnType<T>`.

### Purposes

- **To fill gaps in TypeScript's built-in utilities** with project-specific transformations.
- **To eliminate type duplication** by deriving variants from a single source of truth.
- **To encode domain invariants** at the type level (e.g., "all nested properties are readonly").
- **To generate derived APIs** such as getter/setter types or event handler maps.
- **To implement type-level string algorithms** for naming conventions and path parsing.
- **To centralize complex type logic** in a reusable, tested function.

### Syntax Rules and Structure

**General syntax:**

```typescript
type UtilityName<T, /* additional params */> = /* type transformation */;
```

**Component breakdown:**

- `UtilityName` — the utility type name (PascalCase).
- `<T, ...>` — input type parameters, optionally constrained.
- The body is a type expression (mapped type, conditional type, etc.).

**Syntax rules.**

- Custom utilities are **type aliases**, not interfaces; they cannot be declaration-merged.
- They may have multiple type parameters (`Pick<T, K>` style).
- They may have constraints (`K extends keyof T`).
- They may have defaults (`B extends string = "Default"`).
- They compose with built-in utilities: `Partial<Omit<T, K>>`.

**Constraints and limitations.**

- Custom utilities **cannot create runtime values**.
- They cannot modify the original type; they produce a **new type**.
- Recursive utilities must **terminate** on base cases.
- Monolithic utilities with multiple conditional branches slow compilation significantly.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic custom utility — `Nullable<T>`

```typescript
// example_custom_nullable.ts
// Step 1: Declare a simple custom utility.
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

// Step 2: Apply it to an interface.
interface User {
  name: string;
  age: number;
}

type NullableUser = Nullable<User>;
// { name: string | null; age: number | null }

// Step 3: Use the derived type.
const user: NullableUser = { name: null, age: 36 };
console.log(user.name); // Expected: null
```

**Setup.** Compile with `npx tsc example_custom_nullable.ts --strict`.

**Expected Output:**
```
null
```

**Why this output occurs.** `Nullable<T>` iterates over every property of `T` and unions the value type with `null`. The mapped type is **homomorphic**, preserving any existing modifiers. The result is a type where every property may be null.

---

#### Example 2: Custom utility with a constraint — `PickByValue<T, V>`

```typescript
// example_custom_pick_by_value.ts
// Step 1: Declare a utility with a constrained type parameter.
type PickByValue<T, V> = {
  [K in keyof T as T[K] extends V ? K : never]: T[K];
};

// Step 2: Apply it to filter properties by value type.
interface User {
  id: string;
  name: string;
  age: number;
  active: boolean;
}

type StringProps = PickByValue<User, string>;
// { id: string; name: string }

type NumberProps = PickByValue<User, number>;
// { age: number }

const props: StringProps = { id: "u1", name: "Ada" };
console.log(props.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The mapped type uses **key remapping** (`as`) to filter keys: only keys whose value type extends `V` are kept. Non-matching keys map to `never`, removing them from the result.

---

### Real-World Cases

- **API payload derivation**: custom `CreateRequest<T>`, `UpdateRequest<T>`, `PublicResponse<T>`.
- **Deep transformations**: `DeepPartial<T>`, `DeepReadonly<T>`, `DeepRequired<T>`.
- **Naming conventions**: `Getters<T>`, `Setters<T>`, `EventHandlers<T>`.
- **String case conversion**: `CamelCase<S>`, `KebabCase<S>`, `SnakeCase<S>`.

---

## 2. Utility Type Declaration

### Definitions

**Core Definition.** A **utility type declaration** is the statement that defines a custom utility, including its generic parameters, optional constraints, and the type expression that computes the output.

**Technical Definition.** Declarations use `type Name<T> = ...` with optional constraints (`T extends U`) and defaults (`T = Default`). The body may be a **mapped type**, **conditional type**, **template literal type**, or a composition thereof. Declarations are **erased** and produce no runtime code.

**Beginner-Friendly Explanation.** A utility type declaration is **the line of code where you define your custom type formula**. You give it a name, generic parameters, and a body that computes the transformation.

### Purposes

- **To name and reuse type transformations** across a codebase.
- **To document the intended transformation** with a descriptive name.
- **To parameterize the transformation** with generic types and constraints.
- **To compose complex utilities** from smaller, focused declarations.
- **To export reusable type utilities** for other modules.

### Syntax Rules and Structure

**General syntax:**

```typescript
type UtilityName<T, K extends keyof T = keyof T> = /* body */;
```

**Component breakdown:**

- `UtilityName` — the utility name (PascalCase).
- `<T>` — the primary type parameter.
- `K extends keyof T` — optional constrained parameter.
- `= keyof T` — optional default value.
- `/* body */` — the type transformation expression.

**Syntax rules.**

- The declaration uses `type`, not `interface`.
- Generic parameters may be constrained and may have defaults.
- The body may reference all type parameters.
- Declarations are erased and produce no JavaScript.

**Constraints and limitations.**

- Recursive declarations must have terminating base cases.
- Complex declarations should be broken into smaller composable types.
- Recursive utilities without depth limits can hang the compiler.

### Annotated Complete Step by Step Code Examples

#### Example 1: Declaration with constraint and default

```typescript
// example_declaration_constraint_default.ts
// Step 1: Declare with a constrained parameter and a default.
type OptionalExcept<T, K extends keyof T = never> =
  Partial<T> & Pick<T, K>;

// Step 2: Use the utility with and without the second argument.
interface User {
  id: string;
  name: string;
  email: string;
}

type UserUpdate = OptionalExcept<User, "id">;
// { id: string; name?: string; email?: string }

const update: UserUpdate = { id: "u1", name: "Ada" };
console.log(update.id); // Expected: u1
```

**Expected Output:**
```
u1
```

**Why this output occurs.** `OptionalExcept<T, K>` makes all properties optional via `Partial<T>` but restores the required properties listed in `K` via `Pick<T, K>`. The intersection combines both behaviors.

---

#### Example 2: Declaration with multiple type parameters

```typescript
// example_declaration_multi.ts
// Step 1: Declare a utility with two type parameters.
type Merge<A, B> = {
  [K in keyof A | keyof B]: K extends keyof B
    ? B[K]
    : K extends keyof A
      ? A[K]
      : never;
};

// Step 2: Merge two object types.
type A = { name: string; age: number };
type B = { age: string; active: boolean };

type Merged = Merge<A, B>;
// { name: string; age: string; active: boolean }

const merged: Merged = { name: "Ada", age: "36", active: true };
console.log(merged.age); // Expected: 36
```

**Expected Output:**
```
36
```

**Why this output occurs.** The mapped type iterates over the union of keys from both types. `B` takes precedence for overlapping keys (`age`), while `A` supplies non-overlapping keys (`name`).

---

### Real-World Cases

- **Library utilities**: exporting custom utilities from shared type packages.
- **Domain models**: declaring `CreateRequest<T>`, `UpdateRequest<T>`, and `Public<T>` from a base entity.
- **Form libraries**: declaring `FieldState<T>` utilities for validation states.

---

## 3. Utility Type Usage

### Definitions

**Core Definition.** **Utility type usage** refers to applying custom utilities in type annotations, generic constraints, function parameters, return types, and other type positions to derive and constrain types.

**Technical Definition.** Custom utilities are used wherever a type is expected. They are commonly applied to **variable annotations** (`const x: Partial<T>`), **function signatures** (`function update(id: string, patch: Partial<T>)`), **generic constraints** (`<T extends Partial<User>>`), and **compositions with other utilities** (`Readonly<Partial<T>>`).

**Beginner-Friendly Explanation.** You use a custom utility type **wherever you need the type transformation it provides**—as a variable type, function parameter, return type, or building block for other utilities.

### Purposes

- **To derive types at call sites** without manual duplication.
- **To constrain generic parameters** with custom type contracts.
- **To compose transformations** for complex derived types.
- **To document intent** with meaningful type names at usage sites.

### Syntax Rules and Structure

**Variable annotation:**

```typescript
const update: Partial<User> = { name: "Ada" };
```

**Function parameter:**

```typescript
function applyUpdate(id: string, patch: Partial<User>): void { ... }
```

**Generic constraint:**

```typescript
function process<T extends Partial<User>>(data: T): void { ... }
```

**Composition:**

```typescript
type ReadonlyUpdate = Readonly<Partial<User>>;
```

**Syntax rules.**

- Custom utilities may be used in any type position.
- They may be composed with built-in utilities.
- They may be used as generic constraints.

**Constraints and limitations.**

- Excessively chained utilities can produce complex error messages.
- Deep recursive utilities can slow compilation when used repeatedly.

### Annotated Complete Step by Step Code Examples

#### Example 1: Function parameter with custom utility

```typescript
// example_usage_param.ts
interface User {
  id: string;
  name: string;
  email: string;
}

type UserUpdate = Partial<Omit<User, "id">>;

function updateUser(id: string, patch: UserUpdate): void {
  console.log(`Updating ${id} with`, patch);
}

updateUser("u1", { name: "Ada" });
// Expected: Updating u1 with { name: 'Ada' }
```

**Expected Output:**
```
Updating u1 with { name: 'Ada' }
```

**Why this output occurs.** `UserUpdate` omits `id` and makes the remaining fields optional. The function accepts partial updates without allowing `id` to change.

---

#### Example 2: Generic constraint with custom utility

```typescript
// example_usage_constraint.ts
type WithId = { id: string };

function findById<T extends WithId>(items: T[], id: string): T | undefined {
  return items.find((item) => item.id === id);
}

const users = [
  { id: "u1", name: "Ada" },
  { id: "u2", name: "Grace" },
];

console.log(findById(users, "u2")?.name); // Expected: Grace
```

**Expected Output:**
```
Grace
```

**Why this output occurs.** The constraint `T extends WithId` ensures that `item.id` exists on every element. The return type `T | undefined` preserves the specific element type, not just `WithId`.

---

### Real-World Cases

- **API clients**: `update(id, patch: Partial<Entity>)` for PATCH endpoints.
- **Form handlers**: `values: Partial<FormValues>` for partial form state.
- **Generic utilities**: `groupBy<T extends WithId>(items: T[])` preserving element types.

---

## 4. Custom Utility Type Best Practices

### Definitions

**Core Definition.** **Custom utility type best practices** are guidelines for writing utility types that are **correct, terminating, readable, and compiler-friendly**, avoiding exponential type expansion, unreadable errors, and duplication of built-in utilities.

**Technical Definition.** Best practices center on **preferring built-in utilities**, **composing small focused utilities**, **adding depth limits to recursive utilities**, **keeping unions small**, **using descriptive names**, and **measuring compiler performance**. Overly complex mapped types with multiple conditional branches slow compilation significantly; breaking them into smaller, focused utilities reduces type computation by **50–80%** in applicable cases. Deeply nested conditional types and large template literal unions cause **quadratic type checking** and IDE lag.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep custom utilities fast and maintainable. The big rules are: use built-in utilities first; break complex transformations into small pieces; add depth limits to recursive utilities; and do not re-implement the entire logic layer in the type system.

### Purposes

- **To maximize compiler performance** by avoiding monolithic mapped types and deep recursion.
- **To keep type definitions readable** with small, focused, composable utilities.
- **To prevent duplication** by deriving variants from a single source of truth.
- **To ensure utility types terminate** on all inputs.
- **To choose the right tool** between built-in utilities, custom utilities, and runtime code.

### Syntax Rules and Structure

Best practices are conventions that interact with utility type syntax. The core rules are:

1. **Prefer built-in utilities.** Use `Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record`, `Exclude`, `Extract`, `NonNullable`, `ReturnType`, `Parameters`, and `Awaited` before writing custom utilities.
2. **Compose small utilities.** Break complex transformations into smaller, focused types and compose them. Each small utility is cached separately by the compiler, reducing type computation by 50–80%.
3. **Add depth limits to recursive utilities.** Unbounded recursion can hang the compiler. Use a depth counter (`Depth extends number[] = []`) and stop at a reasonable depth (e.g., 5–10).
4. **Keep unions and tuples small.** Union checking is quadratic; large unions cause measurable slowdowns.
5. **Use descriptive names.** Name utilities after what they do: `DeepPartial`, `Getters`, `PickByValue`, `OptionalExcept`.
6. **Avoid re-implementing the entire logic layer in the type system.** If the transformation is only needed at runtime, write runtime code.
7. **Measure compiler performance.** Watch for editor lag and `tsc` slowdowns; they are signs that utility types are too expensive.
8. **Document complex utilities.** Add comments explaining the transformation and any depth limits or key remapping behavior.

**Component breakdown (custom utility decision tree):**

```
Do you need a type transformation?
  ├── Yes → Can a built-in utility do it?
  │        ├── Yes → Use Partial / Pick / Omit / ReturnType / Parameters / Awaited.
  │        └── No → Write a custom utility.
  │                 ├── Does it recurse? → Add a depth limit and correct base cases.
  │                 ├── Is it complex? → Break it into small composable utilities.
  │                 └── Keep unions small and types readable.
  └── No → Is the transformation only needed at runtime?
           ├── Yes → Write runtime code.
           └── No → Use simple types.
```

**Constraints and limitations.**

- Even well-structured utilities can be slow if unions are huge or recursion is deep.
- Custom utilities cannot replace runtime validation for external data.
- Deep recursion beyond ~50 levels causes "Type instantiation is excessively deep" errors.

### Annotated Complete Step by Step Code Examples

#### Example 1: Composable utilities over monolithic transformations

```typescript
// example_best_composable.ts
// ❌ Monolithic: multiple nested conditionals in one mapped type.
// type Complex<T> = {
//   [K in keyof T]: T[K] extends Function
//     ? T[K]
//     : T[K] extends object
//       ? Complex<T[K]>
//       : T[K] extends number
//         ? string
//         : T[K];
// };

// ✅ Composable: small, cacheable utilities.
type TransformValue<T> = T extends Date ? string : T extends number ? string : T;
type TransformObject<T> = { [K in keyof T]: TransformProperty<T[K]> };
type TransformProperty<T> = T extends Function
  ? T
  : T extends object
    ? TransformObject<T>
    : TransformValue<T>;

interface Data {
  createdAt: Date;
  count: number;
  name: string;
}

type Transformed = TransformObject<Data>;
// { createdAt: string; count: string; name: string }

const data: Transformed = { createdAt: "2025-01-01", count: "42", name: "Ada" };
console.log(data.count); // Expected: 42
```

**Expected Output:**
```
42
```

**Why this output occurs.** Each small utility is cached separately by the compiler, making the transformation easier to debug and faster to compile than a single monolithic mapped type.

---

#### Example 2: Depth-limited recursive utility

```typescript
// example_best_depth_limit.ts
// ❌ Unbounded recursion can hang the compiler.
// type DeepPartial<T> = { [K in keyof T]?: DeepPartial<T[K]> };

// ✅ Depth-limited recursion.
type DeepPartial<T, Depth extends number[] = []> =
  Depth["length"] extends 5
    ? T
    : T extends object
      ? { [K in keyof T]?: DeepPartial<T[K], [...Depth, 1]> }
      : T;

interface Config {
  a: { b: { c: { d: string } } };
}

const partial: DeepPartial<Config> = { a: { b: {} } };
console.log(partial.a?.b); // Expected: {}
```

**Expected Output:**
```
{}
```

**Why this output occurs.** The depth counter stops recursion at five levels, preventing exponential expansion while still allowing meaningful partial updates.

---

#### Example 3: Prefer runtime code for runtime-only transformations

```typescript
// example_best_runtime.ts
// ❌ Type-level string transformation with no downstream type benefit.
// type Shout<S extends string> = `${Uppercase<S>}!`;

// ✅ Runtime function is simpler and faster.
function shout(s: string): string {
  return `${s.toUpperCase()}!`;
}

console.log(shout("hello")); // Expected: HELLO!
```

**Expected Output:**
```
HELLO!
```

**Why this output occurs.** The transformation is not needed by any downstream type, so runtime code is simpler and avoids unnecessary type computation.

---

#### Example 4: Descriptive names and documentation

```typescript
// example_best_naming.ts
/**
 * Makes every property of T optional except the keys in K.
 * Useful for update payloads where the identifier must remain required.
 */
type OptionalExcept<T, K extends keyof T> = Partial<T> & Pick<T, K>;

interface User {
  id: string;
  name: string;
  email: string;
}

type UserUpdate = OptionalExcept<User, "id">;
// { id: string; name?: string; email?: string }

const update: UserUpdate = { id: "u1", name: "Ada" };
console.log(update.id); // Expected: u1
```

**Expected Output:**
```
u1
```

**Why this output occurs.** The utility is documented with a JSDoc comment explaining its purpose and behavior. The name `OptionalExcept` clearly communicates the transformation.

---

### Real-World Cases

- **Large codebases**: composing small utilities instead of monolithic mapped types to reduce compilation time.
- **Configuration management**: depth-limited `DeepPartial` for patch payloads.
- **Code review**: rejecting unbounded recursive utilities and type-level code that duplicates runtime logic.
- **Library authoring**: documenting and exporting custom utilities with clear names and constraints.

---

## References

- TypeScript Documentation — Utility Types (Building Custom Utilities) — https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Documentation — Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- TypeScript Documentation — Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Documentation — Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- TypeScript Documentation — Type Manipulation — https://www.typescriptlang.org/docs/handbook/2/types-from-types.html
- Convex — TypeScript Utility Types (Custom Utilities) — https://www.convex.dev/typescript/advanced/utility-types-mapped-types/typescript-utility-types
- Total TypeScript — Building Custom Utility Types — https://www.totaltypescript.com/workshops/type-transformations/type-helpers
- Simplify Complex Mapped Types — pproenca/dot-skills — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.curated/typescript/references/type-simplify-mapped-types.md
- Performance Best Practices — NickCrew/Claude-Cortex — https://github.com/NickCrew/Claude-Cortex/blob/main/skills/typescript-advanced-patterns/references/performance-best-practices.md
- pproenca/dot-skills — Use Recursive Conditional Types for Structural Transformations — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-advanced-patterns/references/tlp-recursive-conditional-types.md
- pproenca/dot-skills — Limit Type Recursion Depth — https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.curated/typescript/references/type-limit-recursion-depth.md
- ts-essentials — A Collection of Essential TypeScript Types — https://www.npmjs.com/package/ts-essentials
- type-fest — A Collection of Essential TypeScript Types — https://www.npmjs.com/package/type-fest
- ts-toolbelt — A TypeScript Type Utility Library — https://www.npmjs.com/package/ts-toolbelt