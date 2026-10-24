# Type-Level Programming in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Type-level programming** is the practice of writing “code” that executes **entirely within TypeScript's compile-time type system**, using types as values and type operators as control flow. It transforms TypeScript from a simple type checker into a **Turing-complete meta-programming language** capable of computing, validating, and transforming types before any JavaScript runs. The type system becomes a programming language within a programming language.

**Technical Definition.** Type-level programming uses **generics as functions** (type parameters as arguments, return types as outputs), **conditional types as branching** (`extends ? :`), **recursive types as loops**, **`infer` as variable binding and destructuring**, **mapped types as iteration**, and **template literal types as string manipulation**. The “values” are **literal types** (`42`, `"hello"`), **tuples**, and **object types**. Compilation evaluates these programs; the results are **erased** and emit no JavaScript. TypeScript's instantiation depth limit (~50 levels) is the primary termination constraint.

**Beginner-Friendly Explanation.** Type-level programming is **writing programs that run when your code is compiled, not when it is executed**. Instead of computing values at runtime, you compute **types** at compile time. You use generics as functions: `type Double<T> = [T, T]` takes a type and returns a tuple. You use conditional types as `if` statements: `T extends string ? "text" : "other"`. You use recursion as loops. The result is that your types can **adapt, validate, and transform themselves** based on other types—like automatically deriving a getter type from a model type, or parsing route parameters from a URL string.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Compile-time execution** | All computation happens during compilation; runtime behavior is unchanged. |
| **Turing-complete** | TypeScript's type system can theoretically compute anything, within recursion limits. |
| **Generics as functions** | `<T>` parameters are inputs; the type expression is the output. |
| **Conditional types as branching** | `T extends U ? X : Y` is the type-level `if/else`. |
| **Recursion as loops** | Recursive type aliases iterate over tuples, strings, and unions. |
| **`infer` as destructuring** | Captures types from patterns; supports structural decomposition. |
| **Mapped types as iteration** | `[K in keyof T]` iterates over object properties. |
| **Template literals for strings** | Split, join, replace, and convert case at the type level. |
| **Erasure** | All computed types disappear at runtime. |
| **Performance-sensitive** | Deep recursion and large unions cause exponential compiler work and IDE lag. |

---

### Prerequisites

- **TypeScript basics**: type aliases, interfaces, unions, and intersections.
- **Generics**: type parameters, constraints (`extends`), and defaults.
- **Conditional types**: `T extends U ? X : Y` and `infer`.
- **Mapped types**: `[K in keyof T]`, modifiers, and key remapping.
- **Template literal types**: `` `${T}` `` patterns and intrinsic string utilities.
- **Recursive types**: self-referencing aliases and termination conditions.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Type-level computation** — arithmetic, boolean logic, and data structure manipulation at the type level.
- **Type-level algorithms** — sorting, searching, string parsing, and state machines.
- **Type-level string algorithms** — split, join, replace, and case conversion with template literals.
- **Recursive types** — trees, JSON, and deep transformations.
- **Utility types** — `Partial`, `Pick`, `Omit`, `ReturnType`, and `Parameters` are type-level programs.
- **Compiler performance** — recursion depth, union size, and instantiation cost.

---

## Core Concepts / Features

---

## 1. Type-Level Programming

### Definitions

**Core Definition.** **Type-level programming** is the discipline of using TypeScript's type system as a **programming language**, with types as values and type operators as control flow, to compute new types at compile time.

**Technical Definition.** The type system provides **data structures** (object and tuple types), **functions** (generic types), **conditionals** (conditional types with `extends`), **loops** (recursive types), and **basic algebra** (tuple-length arithmetic). Literal types (`42`, `"hello"`) serve as primitive values, enabling precise computation. The result is a **type-level program** that the compiler evaluates during instantiation.

**Beginner-Friendly Explanation.** Type-level programming is **treating types like data and writing programs that manipulate them**. You compute new types from existing ones using the same concepts as regular programming: functions, conditionals, loops, and data structures.

### Purposes

- **To make types adaptive** so they derive their shape from other types instead of being hardcoded.
- **To enforce complex invariants** that simple annotations cannot express.
- **To eliminate type duplication** by computing variants from a single source of truth.
- **To build type-safe libraries** with precise inference and autocomplete.
- **To model domain rules at compile time**, preventing invalid states and operations.

### Syntax Rules and Structure

**Type-level function (generic):**

```typescript
type Double<T> = [T, T];
type DoubleString = Double<string>; // [string, string]
```

**Type-level conditional:**

```typescript
type IsString<T> = T extends string ? true : false;
```

**Type-level loop (recursion):**

```typescript
type BuildTuple<N extends number, T extends unknown[] = []> =
  T["length"] extends N ? T : BuildTuple<N, [...T, unknown]>;
```

**Type-level data structure (object/tuple):**

```typescript
type User = { name: string; age: number };
type Name = User["name"]; // string
```

**Syntax rules.**

- Generics are **functions**: `type F<T> = ...` maps input types to output types.
- Conditional types are **branches**: `T extends U ? X : Y`.
- Recursive aliases are **loops**, with a **base case** that terminates.
- `infer` captures types from patterns for use in the true branch.
- Mapped types iterate over keys; template literals iterate over string patterns.

**Constraints and limitations.**

- The type system is **Turing-complete but resource-limited**: recursion depth is capped (~50 levels).
- There is **no mutable state**; everything is purely functional.
- Type-level programs cannot produce **runtime values**.
- Complex programs can cause **exponential type expansion** and compiler hangs.

### Annotated Complete Step by Step Code Examples

#### Example 1: Type-level `Add` with tuple lengths

```typescript
// example_type_level_add.ts
type BuildTuple<N extends number, T extends unknown[] = []> =
  T["length"] extends N ? T : BuildTuple<N, [...T, unknown]>;

type Add<A extends number, B extends number> =
  [...BuildTuple<A>, ...BuildTuple<B>]["length"];

type Sum = Add<3, 4>; // 7

const sum: Sum = 7;
console.log(sum); // Expected: 7
```

**Expected Output:**
```
7
```

**Why this output occurs.** `BuildTuple<3>` produces `[unknown, unknown, unknown]`; `BuildTuple<4>` produces a four-element tuple. Concatenating them produces a seven-element tuple, and reading `["length"]` yields the literal type `7`.

---

#### Example 2: Type-level boolean logic

```typescript
// example_type_level_logic.ts
type And<A extends boolean, B extends boolean> =
  A extends true ? (B extends true ? true : false) : false;

type Or<A extends boolean, B extends boolean> =
  A extends true ? true : B extends true ? true : false;

type Not<A extends boolean> = A extends true ? false : true;

type Test1 = And<true, true>;   // true
type Test2 = Or<false, true>;   // true
type Test3 = Not<true>;         // false

const t1: Test1 = true;
console.log(t1); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** The conditional types implement boolean logic. `And<true, true>` checks whether both operands extend `true`; `Or` checks whether either extends `true`; `Not` inverts the value.

---

### Real-World Cases

- **Utility types**: `Partial<T>`, `Required<T>`, `Readonly<T>`, and `Pick<T, K>` are type-level programs.
- **API type derivation**: computing request and response types from route definitions.
- **Type-safe event systems**: deriving handler names from event unions.

---

## 2. Type-Level Computation

### Definitions

**Core Definition.** **Type-level computation** is the use of type operators to **calculate new types**—arithmetic on numeric literals, boolean logic, string manipulation, and data structure transformations—at compile time.

**Technical Definition.** Type-level computation leverages **literal types** as values and **tuple lengths** as numeric representations. Arithmetic uses `BuildTuple` and `Concat` to emulate addition and subtraction. Boolean logic uses nested conditional types. String manipulation uses **template literal types** with `infer` for splitting and recursion for iteration. Object manipulation uses **mapped types** and **indexed access**.

**Beginner-Friendly Explanation.** Type-level computation is **doing math and logic with types instead of values**. You can compute `Add<3, 4>` to get `7`, or split a string type into characters, or merge two object types.

### Purposes

- **To compute numeric literals** for array sizes, offsets, and counts.
- **To manipulate string literal types** for naming conventions and path parsing.
- **To transform object shapes** by merging, picking, or omitting properties.
- **To build type-level data structures** such as tuples, linked lists, and trees.
- **To enable compile-time validation** of arithmetic and string constraints.

### Syntax Rules and Structure

**Numeric addition via tuples:**

```typescript
type Add<A extends number, B extends number> =
  [...BuildTuple<A>, ...BuildTuple<B>]["length"];
```

**Numeric subtraction via tuple matching:**

```typescript
type Sub<A extends number, B extends number> =
  BuildTuple<A> extends [...BuildTuple<B>, ...infer U] ? U["length"] : 0;
```

**String splitting:**

```typescript
type Split<S extends string, D extends string> =
  S extends `${infer Head}${D}${infer Rest}` ? [Head, ...Split<Rest, D>] : [S];
```

**Object merging:**

```typescript
type Merge<A, B> = { [K in keyof A | keyof B]: K extends keyof B ? B[K] : K extends keyof A ? A[K] : never };
```

**Syntax rules.**

- Numeric operations use tuple lengths and recursion.
- String operations use template literal types with `infer`.
- Object operations use mapped types and key remapping.
- All computations terminate on base cases (empty tuple, non-matching string, primitive).

**Constraints and limitations.**

- Numeric operations are **slow** for large numbers due to recursive tuple building.
- String operations require **literal** string types; widened `string` cannot be parsed.
- Object operations lose **modifiers** unless homomorphic mapped types are used.

### Annotated Complete Step by Step Code Examples

#### Example 1: Type-level string split and join

```typescript
// example_compute_string.ts
type Split<S extends string, D extends string> =
  S extends `${infer Head}${D}${infer Rest}` ? [Head, ...Split<Rest, D>] : [S];

type Join<T extends string[], D extends string = ""> =
  T extends [] ? "" :
  T extends [infer Head extends string] ? Head :
  T extends [infer Head extends string, ...infer Tail extends string[]]
    ? `${Head}${D}${Join<Tail, D>}` : never;

type Parts = Split<"a,b,c", ",">; // ["a", "b", "c"]
type Joined = Join<Parts, "-">;   // "a-b-c"

const joined: Joined = "a-b-c";
console.log(joined); // Expected: a-b-c
```

**Expected Output:**
```
a-b-c
```

**Why this output occurs.** `Split` recursively matches the delimiter and captures segments; `Join` recursively concatenates with the new delimiter. The result is a type-level string transformation.

---

#### Example 2: Type-level object merge

```typescript
// example_compute_object.ts
type Merge<A, B> = {
  [K in keyof A | keyof B]: K extends keyof B ? B[K] : K extends keyof A ? A[K] : never;
};

type A = { name: string; age: number };
type B = { age: string; active: boolean };
type C = Merge<A, B>;
// { name: string; age: string; active: boolean }

const c: C = { name: "Ada", age: "36", active: true };
console.log(c.age); // Expected: 36
```

**Expected Output:**
```
36
```

**Why this output occurs.** The mapped type iterates over the union of keys from both objects. `B` takes precedence for overlapping keys (`age`), while `A` supplies non-overlapping keys (`name`).

---

### Real-World Cases

- **Route parameter extraction**: splitting `/users/:id/posts/:postId` into parameter names.
- **CSS class generation**: combining color and size unions into class name unions.
- **API response merging**: combining base response fields with endpoint-specific fields.

---

## 3. Type-Level Algorithms

### Definitions

**Core Definition.** **Type-level algorithms** are complete computational procedures—sorting, searching, parsing, and state machines—implemented entirely with TypeScript types.

**Technical Definition.** Type-level algorithms use **recursion**, **conditional types**, **`infer`**, and **tuple manipulation** to implement algorithmic logic. Sorting algorithms (bubble sort, insertion sort) compare and reorder tuple elements. String algorithms (split, join, replace, case conversion) are the **four primitives** that compose into more complex transformations. State machines use object types to map states to their allowed transitions.

**Beginner-Friendly Explanation.** Type-level algorithms are **real algorithms—sorting, searching, parsing—written in the type system**. They run at compile time and produce precise types as output.

### Purposes

- **To sort tuple types** for deterministic ordering in type-level processing.
- **To search for types** within unions or tuples.
- **To parse strings** into structured type information (route params, query strings).
- **To implement type-level state machines** that enforce valid transitions.
- **To build type-level interpreters** for DSLs and configuration languages.

### Syntax Rules and Structure

**Type-level string algorithms (four primitives):**

```typescript
type Split<S extends string, D extends string> = ...;
type Join<T extends string[], D extends string> = ...;
type Replace<S extends string, From extends string, To extends string> = ...;
type CamelCase<S extends string> = ...;
```

**Type-level state machine:**

```typescript
type TrafficLight = {
  red: { next: "green"; wait: 60 };
  green: { next: "yellow"; wait: 45 };
  yellow: { next: "red"; wait: 5 };
};

type NextState<Current extends keyof TrafficLight> = TrafficLight[Current]["next"];
type State1 = NextState<"red">; // "green"
```

**Type-level sorting (conceptual):**

```typescript
type BubbleSort<T extends any[]> = ...; // recursive comparison and swap
```

**Syntax rules.**

- String algorithms compose from **split**, **join**, **replace**, and **case conversion**.
- State machines use object types and indexed access.
- Sorting uses recursive tuple comparison and reconstruction.
- All algorithms require **base cases** for termination.

**Constraints and limitations.**

- Sorting and searching are **expensive** on large tuples; keep tuples small.
- String algorithms require **literal** string types.
- State machines are **static**; they cannot handle dynamic runtime state.

### Annotated Complete Step by Step Code Examples

#### Example 1: Type-level `Replace`

```typescript
// example_algo_replace.ts
type Replace<S extends string, From extends string, To extends string> =
  S extends `${infer L}${From}${infer R}` ? `${L}${To}${Replace<R, From, To>}` : S;

type Result = Replace<"foo_bar_baz", "_", "-">; // "foo-bar-baz"

const result: Result = "foo-bar-baz";
console.log(result); // Expected: foo-bar-baz
```

**Expected Output:**
```
foo-bar-baz
```

**Why this output occurs.** The algorithm recursively matches the `From` substring, replaces it with `To`, and recurses on the remainder until no match remains.

---

#### Example 2: Type-level state machine

```typescript
// example_algo_state_machine.ts
type TrafficLight = {
  red: { next: "green"; wait: 60 };
  green: { next: "yellow"; wait: 45 };
  yellow: { next: "red"; wait: 5 };
};

type NextState<Current extends keyof TrafficLight> = TrafficLight[Current]["next"];

type S1 = NextState<"red">;   // "green"
type S2 = NextState<S1>;      // "yellow"
type S3 = NextState<S2>;      // "red"

const state: S1 = "green";
console.log(state); // Expected: green
```

**Expected Output:**
```
green
```

**Why this output occurs.** The state machine is an object type mapping each state to its transition metadata. `NextState` uses indexed access to compute the successor state type.

---

### Real-World Cases

- **Route parsing**: extracting `:id` parameters from `/users/:id/posts/:postId`.
- **Event handler generation**: `on${Capitalize<Event>}` for typed event names.
- **Type-safe state machines**: enforcing legal transitions in UI wizards and workflows.
- **Configuration validation**: parsing and validating string patterns at compile time.

---

## 4. Type-Level Programming Best Practices

### Definitions

**Core Definition.** **Type-level programming best practices** are guidelines for writing type-level programs that are **correct, terminating, and compiler-friendly**, avoiding exponential expansion, depth-limit errors, and unreadable type errors.

**Technical Definition.** Best practices center on **depth-limited recursion**, **tail-recursive patterns**, **correct base cases**, **preferring built-in utilities**, **keeping unions small**, and **avoiding type-level computation when runtime code would suffice**. Recursion beyond ~50 levels causes exponential compiler work and “Type instantiation is excessively deep” errors. Deeply nested conditional types and large template literal unions cause **quadratic type checking** and IDE lag.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced type-level programmers use** to keep their type programs from freezing the compiler. The big rules are: add depth limits; use tail recursion; keep unions small; and do not compute at the type level what you could compute at runtime.

### Purposes

- **To prevent compiler hangs** and exponential type expansion.
- **To avoid “excessively deep” instantiation errors** on recursive types.
- **To keep IDE responsiveness** by limiting type computation.
- **To ensure type-level algorithms terminate** on all inputs.
- **To choose the right tool** between type-level computation and runtime code.

### Syntax Rules and Structure

Best practices are conventions that interact with type-level syntax. The core rules are:

1. **Add explicit depth limits.** Use a depth counter (`Depth extends number[] = []`) and stop at a reasonable depth (e.g., 5–10) to prevent exponential expansion.
2. **Use tail-recursive patterns.** Structure recursion so the recursive call is the **final result**; this avoids deep stack growth in the type checker.
3. **Get the base cases right.** Terminate on primitives, empty tuples, and non-matching strings. Homomorphic mapped types over primitives return the primitive unchanged.
4. **Prefer built-in utility types.** `Partial`, `Pick`, `ReturnType`, `Parameters`, and `Awaited` are optimized type-level programs.
5. **Keep unions and tuples small.** Union checking is quadratic; large unions and tuples cause slowdowns.
6. **Avoid type-level computation for runtime-only concerns.** If the transformation is not needed by downstream types, do not reify it at the type level.
7. **Use `satisfies` and `as const` for inference control** instead of complex conditional types when validation and literal preservation are the goals.
8. **Test with edge cases.** Verify behavior with `never`, `any`, `unknown`, empty tuples, and non-matching strings.

**Component breakdown (type-level programming decision tree):**

```
Do you need to compute a new type from existing types?
  ├── Yes → Can a built-in utility type do it?
  │        ├── Yes → Use Partial / Pick / ReturnType / Parameters.
  │        └── No → Write a type-level program.
  │                 ├── Does it recurse? → Add a depth limit and correct base cases.
  │                 ├── Does it use strings? → Use template literal primitives.
  │                 └── Keep it small and composable.
  └── No → Is the computation only needed at runtime?
           ├── Yes → Write runtime code, not type-level code.
           └── No → Use simple types.
```

**Constraints and limitations.**

- Even depth-limited recursion can be expensive on wide unions.
- Tail recursion is not always expressible in TypeScript's conditional type syntax.
- Type-level programs cannot handle dynamic runtime input.

### Annotated Complete Step by Step Code Examples

#### Example 1: Depth-limited recursive type

```typescript
// example_best_depth_limit.ts
// ❌ Unbounded recursion can hang the compiler.
// type DeepReadonly<T> = { readonly [K in keyof T]: DeepReadonly<T[K]> };

// ✅ Depth-limited recursion.
type DeepReadonly<T, Depth extends number[] = []> =
  Depth["length"] extends 5
    ? T
    : T extends object
      ? { readonly [K in keyof T]: DeepReadonly<T[K], [...Depth, 1]> }
      : T;

interface Config {
  a: { b: { c: { d: string } } };
}

const config: DeepReadonly<Config> = { a: { b: { c: { d: "value" } } } };
// config.a.b.c.d = "other"; // ❌ Error: readonly.
console.log(config.a.b.c.d); // Expected: value
```

**Expected Output:**
```
value
```

**Why this output occurs.** The depth counter stops recursion at five levels, preventing exponential expansion while still transforming nested properties. All levels are marked `readonly`.

---

#### Example 2: Prefer runtime code for runtime-only transformations

```typescript
// example_best_runtime.ts
// ❌ Type-level string transformation with no downstream type benefit.
// type Shout<S extends string> = `${Uppercase<S>}!`;

// ✅ Runtime function does the same job more simply.
function shout(s: string): string {
  return `${s.toUpperCase()}!`;
}

console.log(shout("hello")); // Expected: HELLO!
```

**Expected Output:**
```
HELLO!
```

**Why this output occurs.** The transformation is not needed by any downstream type, so a runtime function is simpler, faster, and easier to maintain than a type-level program.

---

#### Example 3: Composable string primitives

```typescript
// example_best_composable.ts
type Split<S extends string, D extends string> =
  S extends `${infer H}${D}${infer R}` ? [H, ...Split<R, D>] : [S];

type Join<T extends string[], D extends string = ""> =
  T extends [] ? "" :
  T extends [infer H extends string] ? H :
  T extends [infer H extends string, ...infer R extends string[]]
    ? `${H}${D}${Join<R, D>}` : never;

type CamelCase<S extends string> =
  Split<S, "_"> extends [infer H extends string, ...infer R extends string[]]
    ? `${H}${Join<{ [K in keyof R]: Capitalize<R[K] & string> }, "">}`
    : S;

type Result = CamelCase<"last_login_at">; // "lastLoginAt"

const result: Result = "lastLoginAt";
console.log(result); // Expected: lastLoginAt
```

**Expected Output:**
```
lastLoginAt
```

**Why this output occurs.** `Split` breaks the snake_case string on `_`; `Capitalize` transforms each segment; `Join` concatenates them. The four base string primitives compose into a general camelCase converter.

---

### Real-World Cases

- **Library type utilities**: using `ts-toolbelt` or `type-fest` instead of writing custom recursive types.
- **Large codebases**: depth-limiting deep transformations to prevent compiler slowdowns.
- **Route parsing**: composing split and join primitives for path parameter extraction.
- **Code review**: rejecting unbounded recursive types and type-level programs that duplicate runtime logic.

---

## References

- Type-Level Programming | React with TypeScript | Steve Kinney — https://stevekinney.com/courses/react-typescript/typescript-type-level-programming
- TypeScript Types as a Programming Language — https://marmelab.com/blog/2025/12/04/typescript-type-as-a-programming-language.html
- Developing type-level algorithms in TypeScript — https://softwaremill.com/developing-type-level-algorithms-in-typescript/
- Build Type-Level String Algorithms with Recursive Template Literals — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-advanced-patterns/references/tlp-type-level-string-algorithms.md
- Type-Level Programming Patterns — https://github.com/ahmad-ubaidillah/aizen-gate/blob/main/aizen-gate/skills-reference/.agent/skills/typescript-expert/sub-skills/type-level-programming-patterns.md
- TypeScript Documentation — Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Documentation — Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- TypeScript Documentation — Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- TypeScript Documentation — Recursive Type References — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html
- TypeScript 4.1 Release Notes — Recursive Conditional Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html