# `infer` Keyword in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** The **`infer` keyword** is a TypeScript type-level construct used **exclusively within the `extends` clause of conditional types** to **capture and extract type information** from a larger type structure. It performs **pattern matching for types**: the developer defines a type pattern containing `infer` placeholders, and TypeScript binds those placeholders to the corresponding types when the pattern matches.

**Technical Definition.** `infer` introduces a **type variable** within a conditional type's `extends` clause. When the checked type is assignable to the pattern, the `infer` variable is bound to the type at that structural position, and the **true branch** of the conditional can reference it. Since **TypeScript 4.7**, `infer X extends Y` constrains the inferred type to `Y`, combining binding and validation in a single step and eliminating post-extraction narrowing. `infer` is the mechanism behind built-in utility types such as `ReturnType`, `Parameters`, `InstanceType`, and `Awaited`.

**Beginner-Friendly Explanation.** `infer` is **“capture whatever type fits this spot.”** You write a pattern like `T extends (infer U)[] ? U : never`, which means: “If `T` is an array, grab its element type and call it `U`; otherwise, return `never`.” It is how TypeScript **looks inside a type** and pulls out the piece you care about—return types, parameter types, promise values, tuple elements, and even substrings from template literal types.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Conditional-only** | `infer` may appear **only** in the `extends` clause of a conditional type. |
| **Pattern matching** | It matches the structure of a type and captures the part at the `infer` position. |
| **True-branch scope** | The inferred variable is available in the **true branch** of the conditional. |
| **Multiple captures** | Several `infer` declarations may appear in one pattern. |
| **Constrained infer (TS 4.7+)** | `infer X extends Y` binds and validates the extracted type. |
| **Composability** | Works with arrays, promises, functions, tuples, template literal types, and objects. |
| **Erasure** | All `infer` constructs are compile-time only and emit no JavaScript. |
| **Performance-sensitive** | Deeply nested `infer` chains and large unions can slow compilation. |

---

### Prerequisites

- **TypeScript basics**: type aliases, unions, and interfaces.
- **Conditional types**: `T extends U ? X : Y` branching.
- **Generics fundamentals**: type parameters and constraints.
- **Template literal types** for string pattern extraction.
- **Indexed access types** (`T[K]`) and `keyof`.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Conditional types** — the only context where `infer` is valid.
- **Utility types** — `ReturnType`, `Parameters`, `InstanceType`, and `Awaited` are built with `infer`.
- **Template literal types** — extracting substrings from string patterns.
- **Mapped types** — often combined with `infer` for recursive transformations.
- **Type-level programming** — `infer` is the extraction primitive of TypeScript’s type system.

---

## Core Concepts / Features

---

## 1. `infer` Keyword

### Definitions

**Core Definition.** The **`infer` keyword** declares a **type variable inside a conditional type's pattern** and binds it to the type found at that position when the pattern matches.

**Technical Definition.** `infer` appears in the `extends` clause of a conditional type: `T extends Pattern<infer U> ? X : Y`. If `T` matches `Pattern`, `U` is inferred from the corresponding position and is available in the true branch `X`. If no match occurs, the false branch is used. `infer` cannot be used outside conditional types.

**Beginner-Friendly Explanation.** `infer` is **the capture group of a type-level regular expression**. You write a pattern, put `infer U` where you want to grab a type, and TypeScript fills in `U` when the pattern matches.

### Purposes

- **To extract inner types** from arrays, promises, functions, objects, and tuples.
- **To implement utility types** such as `ReturnType` and `Parameters` without manual extraction.
- **To pattern-match structured types** and derive new types from their components.
- **To enable type-level parsing** of template literal strings.
- **To avoid unsafe type assertions** by deriving types from verified patterns.

### Syntax Rules and Structure

**General syntax:**

```
T extends SomePattern<infer U> ? TrueBranch : FalseBranch
```

**Component breakdown:**

- `T` — the checked type.
- `SomePattern<infer U>` — the pattern containing the `infer` placeholder.
- `U` — the captured type variable, scoped to the true branch.
- `? TrueBranch` — selected when the pattern matches; may use `U`.
- `: FalseBranch` — selected otherwise.

**Syntax rules.**

- `infer` must appear **inside the `extends` clause** of a conditional type.
- The inferred variable is scoped to the **true branch**.
- Multiple `infer` declarations may appear in a single pattern.
- `infer` may not be used in the false branch.

**Constraints and limitations.**

- `infer` cannot be used outside conditional types.
- `infer` captures whatever the position allows, often widening to `string` or `unknown`; constrain with `infer X extends Y` when precision is needed.
- Deeply nested `infer` patterns can hit recursion limits.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic array element extraction

```typescript
// example_infer_basic.ts
// Step 1: Define a conditional type with infer.
type ArrayElement<T> = T extends (infer U)[] ? U : never;

// Step 2: Apply to different array types.
type A = ArrayElement<string[]>;   // string
type B = ArrayElement<number[]>;   // number
type C = ArrayElement<string>;     // never (not an array)

const a: A = "hello";
console.log(a); // Expected: hello
```

**Setup.** Compile with `npx tsc example_infer_basic.ts --strict`.

**Expected Output:**
```
hello
```

**Why this output occurs.** `string[]` matches the pattern `(infer U)[]` with `U = string`. `string` does not match, so `never` is returned.

---

### Real-World Cases

- **Utility type implementation**: `ReturnType<T>`, `Parameters<T>`, `InstanceType<T>`.
- **API response typing**: extracting the resolved value from `Promise<T>`.
- **Event system typing**: extracting payload types from handler signatures.

---

## 2. Infer Declaration

### Definitions

**Core Definition.** An **infer declaration** is the syntactic introduction of one or more `infer` type variables within a conditional type's pattern, optionally with **constraints** (`infer X extends Y`).

**Technical Definition.** An infer declaration appears as `infer U` or `infer U extends Constraint` inside the `extends` clause. Since TypeScript 4.7, `infer X extends Y` **binds the variable and asserts a constraint** in one step; if the extracted type does not satisfy `Y`, the conditional takes the false branch. This eliminates the need for nested conditionals that re-narrow the extracted type after the fact.

**Beginner-Friendly Explanation.** An infer declaration is **where you name the type you are extracting**. You write `infer U` to capture a type, or `infer U extends string` to capture it **only if it is a string**.

### Purposes

- **To name extracted types** for use in the true branch.
- **To constrain extracted types** so invalid matches are rejected automatically.
- **To capture multiple types** from different positions in a single pattern.
- **To simplify type-level pipelines** by replacing post-extraction narrowing with a single constrained `infer`.

### Syntax Rules and Structure

**Unconstrained infer:**

```
infer U
```

**Constrained infer (TS 4.7+):**

```
infer U extends Constraint
```

**Multiple infer declarations:**

```
T extends [infer First, infer Second, ...unknown[]] ? ... : never
```

**Component breakdown:**

- `infer` — the keyword.
- `U` — the type variable name.
- `extends Constraint` — optional constraint restricting the captured type.

**Syntax rules.**

- The constraint is checked **during matching**; if the extracted type does not satisfy it, the match fails.
- Constrained `infer` may be used with template literal types, tuple positions, and function signatures.
- Multiple infer declarations are scoped to the same true branch.

**Constraints and limitations.**

- Constrained `infer` requires TypeScript 4.7+.
- A constraint that is itself a union of non-literal kinds (`string | number | boolean`) may widen and defeat the purpose.
- Pre-4.7 codebases do not narrow with constrained `infer`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Constrained infer for numeric parsing

```typescript
// example_infer_constrained.ts
// ❌ Unconstrained: captures a string, not a number.
// type ToNumber<S extends string> = S extends `${infer N}` ? N : never;

// ✅ Constrained: captures a number literal directly.
type ToNumber<S extends string> = S extends `${infer N extends number}` ? N : never;

type A = ToNumber<"42">;    // 42 (number literal)
type B = ToNumber<"forty">; // never (constraint fails)

const a: A = 42;
console.log(a); // Expected: 42
```

**Expected Output:**
```
42
```

**Why this output occurs.** `infer N extends number` binds `N` to the numeric literal `42` for `"42"`. For `"forty"`, the extracted token is `"forty"`, which does not extend `number`, so the conditional takes the false branch and returns `never`.

---

#### Example 2: Multiple infer declarations in a tuple

```typescript
// example_infer_multiple.ts
type Pair<T> = T extends [infer First, infer Second] ? [First, Second] : never;

type A = Pair<[string, number]>; // [string, number]
type B = Pair<[string]>;         // never (too short)

const pair: A = ["hello", 42];
console.log(pair[1]); // Expected: 42
```

**Expected Output:**
```
42
```

**Why this output occurs.** The pattern matches a two-element tuple and captures both positions as `First` and `Second`. A one-element tuple does not match the two-element pattern.

---

### Real-World Cases

- **Route parameter parsing**: `infer Param extends string` for path segments.
- **API version extraction**: `infer Version extends number` from version strings.
- **Tuple unpacking**: capturing multiple positional types in one pattern.

---

## 3. Infer Usage

### Definitions

**Core Definition.** **Infer usage** refers to applying `infer` across different type structures—arrays, promises, functions, tuples, objects, and template literal strings—to **extract and derive types**.

**Technical Definition.** `infer` may be placed at any **structural position** in the `extends` pattern: element type of an array (`(infer U)[]`), resolved value of a promise (`Promise<infer U>`), return type of a function (`(...args: any[]) => infer R`), parameter tuple (`(...args: infer P) => any`), property type (`{ data: infer D }`), tuple position (`[infer First, ...unknown[]]`), and substring of a template literal (`` `${infer Prefix}:${infer Suffix}` ``). The captured type is available in the true branch.

**Beginner-Friendly Explanation.** You can use `infer` **wherever a type has an inner structure**. It works on arrays, promises, functions, objects, tuples, and even strings.

### Purposes

- **To extract return types and parameter types** from function signatures.
- **To unwrap promise and array types** to their inner values.
- **To parse template literal strings** into named parts.
- **To derive object property types** without indexed access.
- **To build recursive type parsers** for structured strings and nested types.

### Syntax Rules and Structure

**Array element:**

```typescript
type Element<T> = T extends (infer U)[] ? U : never;
```

**Promise value:**

```typescript
type Resolved<T> = T extends Promise<infer U> ? U : never;
```

**Function return type:**

```typescript
type Return<T> = T extends (...args: any[]) => infer R ? R : never;
```

**Parameter tuple:**

```typescript
type Params<T> = T extends (...args: infer P) => any ? P : never;
```

**Object property:**

```typescript
type Data<T> = T extends { data: infer D } ? D : never;
```

**Template literal extraction:**

```typescript
type Before<T> = T extends `${infer Prefix}:${string}` ? Prefix : never;
type After<T> = T extends `${string}:${infer Suffix}` ? Suffix : never;
```

**Syntax rules.**

- `infer` must be placed at a position where a type is expected in the pattern.
- The pattern must be a valid type expression.
- Captured types are available in the true branch only.

**Constraints and limitations.**

- Greedy template literal matching may capture more than intended; use explicit delimiters.
- Function overloads: `infer` uses the **last signature** for return type extraction.
- Deeply recursive extraction can hit instantiation depth limits.

### Annotated Complete Step by Step Code Examples

#### Example 1: Function return type and parameters

```typescript
// example_infer_function.ts
const greet = (name: string, age: number): string => `Hello, ${name}`;

type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type MyParameters<T> = T extends (...args: infer P) => any ? P : never;

type GreetReturn = MyReturnType<typeof greet>;   // string
type GreetParams = MyParameters<typeof greet>;   // [name: string, age: number]

const message: GreetReturn = "Hello, Ada";
console.log(message); // Expected: Hello, Ada
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The function type is matched against `(...args: any[]) => infer R`, capturing the return type `string`. It is also matched against `(...args: infer P) => any`, capturing the parameter tuple `[string, number]`.

---

#### Example 2: Promise unwrapping

```typescript
// example_infer_promise.ts
type Awaited<T> = T extends Promise<infer U> ? U : T;

type A = Awaited<Promise<string>>; // string
type B = Awaited<number>;          // number

const value: A = "resolved";
console.log(value); // Expected: resolved
```

**Expected Output:**
```
resolved
```

**Why this output occurs.** `Promise<string>` matches `Promise<infer U>` with `U = string`. A non-promise type does not match and is returned unchanged.

---

#### Example 3: Template literal extraction

```typescript
// example_infer_template.ts
type Before<T> = T extends `${infer Prefix}:${string}` ? Prefix : T;
type After<T> = T extends `${string}:${infer Suffix}` ? Suffix : T;

type BeforeTest = Before<"prefix:suffix">; // "prefix"
type AfterTest = After<"prefix:suffix">;   // "suffix"

const prefix: BeforeTest = "prefix";
console.log(prefix); // Expected: prefix
```

**Expected Output:**
```
prefix
```

**Why this output occurs.** The template literal pattern matches the colon delimiter, capturing everything before it as `Prefix` and everything after it as `Suffix`.

---

### Real-World Cases

- **Redux action typing**: using `ReturnType<typeof actionCreator>` to type reducers.
- **API client unwrapping**: `Awaited<ReturnType<typeof fetchUser>>`.
- **Route parsing**: extracting `:param` segments from route strings.
- **Event payload extraction**: deriving payload types from handler signatures.

---

## 4. Infer Best Practices

### Definitions

**Core Definition.** **Infer best practices** are guidelines for using `infer` in ways that maximize **precision, readability, and compiler performance**, while avoiding excessive nesting, unconstrained widening, and recursion pitfalls.

**Technical Definition.** Best practices center on **constraining `infer` with `extends`** for validated extraction (TypeScript 4.7+), **keeping `infer` patterns shallow and composable**, **avoiding deeply nested conditional chains**, **preferring iterative type aliases over recursive `infer` pyramids**, and **leveraging built-in utility types** before writing custom extraction logic. Performance guidance warns that deeply nested `infer` chains cause **quadratic type checking** and can hit instantiation depth limits.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep `infer` from becoming a performance or readability disaster. The big rules are: constrain `infer` when you need a specific type; keep patterns simple; break complex extraction into small named aliases; and use built-in utility types when they already do the job.

### Purposes

- **To prevent unsafe `as` casts** after type-level parsing by constraining extraction.
- **To maximize compiler performance** by avoiding deep recursion and nested conditionals.
- **To keep type definitions readable** with focused, single-purpose extraction aliases.
- **To ensure accurate extraction** by rejecting matches that do not satisfy constraints.
- **To leverage built-in utilities** instead of reinventing extraction logic.

### Syntax Rules and Structure

Best practices are conventions that interact with `infer` syntax. The core rules are:

1. **Constrain `infer` for validated extraction.** Use `infer X extends Y` when the extracted token must be a specific kind (number, string literal, key, tuple). This binds and validates in one step, eliminating post-extraction narrowing.
2. **Keep `infer` patterns shallow and composable.** Break complex extraction into small named aliases instead of deeply nested conditional pyramids.
3. **Prefer iterative aliases over recursive `infer` chains.** Compose `ElementType` applications rather than writing a single deeply nested conditional.
4. **Use built-in utility types first.** `ReturnType`, `Parameters`, `InstanceType`, and `Awaited` already handle common cases.
5. **Avoid excessive nesting.** Deeply nested `infer` patterns slow compilation and produce hard-to-read errors.
6. **Test with edge cases.** Verify behavior with `never`, `any`, `unknown`, unions, and non-matching types.
7. **Document complex extraction types.** Add comments explaining the pattern and fallback behavior.

**Component breakdown (infer decision tree):**

```
Do you need to extract an inner type?
  ├── Yes → Does a built-in utility type do it?
  │        ├── Yes → Use ReturnType / Parameters / Awaited / InstanceType.
  │        └── No → Write a conditional type with infer.
  │                 ├── Must the extracted type satisfy a constraint?
  │                 │   ├── Yes → Use `infer X extends Y`.
  │                 │   └── No → Use bare `infer X`, but consider downstream narrowing.
  │                 └── Keep the pattern shallow and composable.
  └── No → Use a simpler type.
```

**Constraints and limitations.**

- Constrained `infer` requires TypeScript 4.7+.
- Deep recursion and large unions can cause compiler slowdowns.
- `infer` cannot extract types outside conditional types.

### Annotated Complete Step by Step Code Examples

#### Example 1: Constrained infer over post-narrowing

```typescript
// example_best_constrained.ts
// ❌ Extract then re-narrow: harder to read, still typed as string.
// type ToNumber<S extends string> = S extends `${infer N}` ? N extends `${number}` ? N : never : never;

// ✅ Constrain during extraction: directly typed as number.
type ToNumber<S extends string> = S extends `${infer N extends number}` ? N : never;

type A = ToNumber<"42">;    // 42 (number literal)
type B = ToNumber<"forty">; // never

const a: A = 42;
console.log(a); // Expected: 42
```

**Expected Output:**
```
42
```

**Why this output occurs.** `infer N extends number` binds `N` to `42` and validates the constraint in one step. The unconstrained version would capture `"42"` as a string and require additional conditionals to narrow it.

---

#### Example 2: Composable iterative aliases over deep nesting

```typescript
// example_best_composable.ts
// ❌ Deeply nested infer pyramid — hard to read and slow.
// type DeepArray<T> = T extends Array<infer U>
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

**Why this output occurs.** Each `ElementType` application unwraps one level. The iterative composition is easier to read and avoids the performance cost of a single deeply nested conditional.

---

#### Example 3: Prefer built-in utilities

```typescript
// example_best_builtin.ts
const greet = (name: string): string => `Hello, ${name}`;

// ❌ Custom implementation duplicates built-in behavior.
// type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

// ✅ Use the built-in.
type GreetReturn = ReturnType<typeof greet>; // string

const message: GreetReturn = "Hello";
console.log(message); // Expected: Hello
```

**Expected Output:**
```
Hello
```

**Why this output occurs.** `ReturnType` is already implemented with `infer` and optimized by the compiler. Reimplementing it adds maintenance burden without benefit.

---

### Real-World Cases

- **Library utility types**: using constrained `infer` for route parameters, version numbers, and API tokens.
- **Large codebases**: splitting complex extraction into small composable aliases to improve caching and compile time.
- **Type-safe parsers**: using `infer X extends Y` to validate extracted tokens without post-narrowing.
- **Code review**: rejecting deeply nested `infer` chains in favor of iterative aliases.

---

## References

- Using `infer` to Extract Types Within Conditional Types — mcollina/skills - https://github.com/mcollina/skills/blob/main/skills/typescript-magician/rules/infer-keyword.md
- Understanding the `infer` Keyword in Advanced TypeScript Types — Educative - https://www.educative.io/courses/advanced-typescript-masterclass/the-infer-keyword
- Constrain `infer` with `extends` for Validated Extraction — pproenca/dot-skills - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-advanced-patterns/references/tlp-infer-extends-constraints.md
- TypeScript Performance Best Practices — NickCrew/Claude-Cortex - https://github.com/NickCrew/Claude-Cortex/blob/main/skills/typescript-advanced-patterns/references/performance-best-practices.md
- TypeScript “infer” Tips — KINTO Technologies Blog - https://blog.kinto-technologies.com/posts/2024-12-02-typescript-infer/
- TypeScript Documentation — Conditional Types - https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript 4.7 Release Notes — `extends` Constraints on `infer` Type Variables - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-7.html
- TypeScript Documentation — Utility Types (`ReturnType`, `Parameters`, `Awaited`) - https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Documentation — Template Literal Types - https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html