# Conditional Types in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **conditional type** is a type-level construct that selects one of two possible types based on a **type relationship test** expressed with the `extends` keyword. Conditional types “help describe the relation between the types of inputs and outputs” and are the primary mechanism for **non-uniform type mappings**—type transformations that differ based on the shape of the input type.

**Technical Definition.** A conditional type has the form `T extends U ? X : Y`. When the type `T` is **assignable to** the type `U`, the conditional resolves to `X`; otherwise, it resolves to `Y`. Conditional types become **distributive** when the checked type is a **naked type parameter**: they automatically distribute over union members during instantiation. The `infer` keyword (TypeScript 2.8+) allows types to be **extracted** from within the structure being matched in the true branch.

**Beginner-Friendly Explanation.** A conditional type is **an “if statement” for types**. You write `T extends U ? X : Y`, which means: “if type `T` fits inside type `U`, use type `X`; otherwise, use type `Y`.” It lets TypeScript **choose a type based on another type**, just like a JavaScript `if` chooses a value based on a condition. The real power comes when `T` is a generic parameter: you can write a single type function that behaves differently depending on what type it receives.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Type-level branching** | Selects between two types based on assignability (`extends`). |
| **Non-uniform mapping** | Different inputs can produce structurally different outputs. |
| **Distributive by default** | Naked type parameters distribute over union members automatically. |
| **`infer` extraction** | Captures inner types (return types, element types) during matching. |
| **Composable** | Can be nested, combined with mapped types, and used in utility types. |
| **Deferred evaluation** | When type parameters are unresolved, the conditional type is deferred until instantiation. |
| **Erasure** | All conditional types are compile-time only and emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: type annotations, interfaces, type aliases, and union types.
- **Generics fundamentals**: type parameters and constraints (`<T extends U>`).
- **Union types** and **literal types**.
- **The `extends` keyword** in generic constraints and interface inheritance.
- **Indexed access types** (`T[K]`) and `keyof`.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Utility types** — `Exclude`, `Extract`, `NonNullable`, `ReturnType`, `Parameters`, and `Awaited` are conditional types.
- **Mapped types** — often combined with conditional types for property-level transformations.
- **Distributive conditional types** — the union-distribution behavior that enables filtering and mapping.
- **`infer` keyword** — type extraction and pattern matching.
- **Recursive type aliases** — conditional types enable recursive structures like `DeepPartial`.
- **Type-level programming** — conditional types are the control-flow construct of TypeScript’s type system.

---

## Core Concepts / Features

---

## 1. Conditional Types

### Definitions

**Core Definition.** A **conditional type** selects one of two types based on whether one type is assignable to another. It is the type-level equivalent of a ternary expression.

**Technical Definition.** `T extends U ? X : Y` resolves to `X` when `T` is assignable to `U`, and to `Y` otherwise. Conditional types are **deferred** when `T` or `U` contains unresolved type parameters; the compiler waits until the type arguments are known. When the checked type is a **naked type parameter**, the conditional type distributes over union members.

**Beginner-Friendly Explanation.** A conditional type is **“if this type is a kind of that type, use this; otherwise use that.”** For example, `IsString<T> = T extends string ? true : false` checks whether `T` is a string and returns `true` or `false`.

### Purposes

- **To express non-uniform type mappings** where different input types produce different output types.
- **To build utility types** that filter, extract, or transform other types.
- **To encode type-level logic** such as “if `T` is an array, use its element type.”
- **To replace overloads** with a single generic signature that adapts its return type.
- **To enforce conditional constraints** in generic APIs.

### Syntax Rules and Structure

**General syntax:**

```
T extends U ? X : Y
```

**Component breakdown:**

- `T` — the **checked type**.
- `extends` — the **assignability test** keyword.
- `U` — the **target type**.
- `? X` — the **true branch**, selected when `T` is assignable to `U`.
- `: Y` — the **false branch**, selected otherwise.

**Syntax rules.**

- Conditional types may appear anywhere a type is expected.
- They may be nested: `A extends B ? (C extends D ? X : Y) : Z`.
- They may be generic: `type F<T> = T extends U ? X : Y`.
- They may reference `infer` in the `extends` clause.
- They are **deferred** when type parameters are unresolved.

**Constraints and limitations.**

- Without generics, conditional types are of limited use because the condition is statically known.
- Conditional types cannot be used as **values** or at runtime.
- Deeply nested conditional types can produce complex error messages and hit recursion limits.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic conditional type

```typescript
// example_conditional_basic.ts
// Step 1: Define a conditional type.
type IsString<T> = T extends string ? true : false;

// Step 2: Apply it to different types.
type A = IsString<string>;   // true
type B = IsString<number>;   // false
type C = IsString<"hello">;  // true (literal extends string)

const a: A = true;
const b: B = false;
console.log(a, b); // Expected: true false
```

**Expected Output:**
```
true false
```

**Why this output occurs.** `string` is assignable to `string`, so `A` is `true`. `number` is not assignable to `string`, so `B` is `false`. The literal `"hello"` is a subtype of `string`, so `C` is also `true`.

---

#### Example 2: `Flatten` — array element extraction with a conditional type

```typescript
// example_conditional_flatten.ts
// Step 1: Conditional type that extracts element types from arrays.
type Flatten<T> = T extends any[] ? T[number] : T;

// Step 2: Apply to an array type.
type Str = Flatten<string[]>; // string

// Step 3: Apply to a non-array type.
type Num = Flatten<number>;   // number

const str: Str = "hello";
const num: Num = 42;
console.log(str, num); // Expected: hello 42
```

**Expected Output:**
```
hello 42
```

**Why this output occurs.** When `T` is `string[]`, the condition is true, and `T[number]` extracts the element type `string`. When `T` is `number`, the condition is false, and `T` is returned unchanged.

---

### Real-World Cases

- **API response adaptation**: returning different shapes based on whether a request succeeds or fails.
- **Type predicates**: `IsString<T>`, `IsArray<T>`, and similar type-level checks.
- **Utility type implementation**: `Exclude`, `Extract`, and `NonNullable` are conditional types.

---

## 2. `T extends U ? X : Y`

### Definitions

**Core Definition.** The syntax `T extends U ? X : Y` is the **concrete form** of a conditional type, where the `extends` keyword performs an assignability test and the ternary-like `? :` selects the result.

**Technical Definition.** The `extends` keyword in this context is **not inheritance**; it is an **assignability test**. `T extends U` is true if `T` is assignable to `U`. The result type is `X` when true and `Y` when false. The entire construct is a **type expression**, not a value expression.

**Beginner-Friendly Explanation.** `T extends U ? X : Y` is **the type-level equivalent of `condition ? valueIfTrue : valueIfFalse`**. You write it exactly like a JavaScript ternary, but with types instead of values.

### Purposes

- **To express type-level branching** in a single, readable expression.
- **To choose between two types** based on the shape of the input type.
- **To implement type-level decision trees** by nesting conditional types.
- **To encode constraints** that depend on the input type.

### Syntax Rules and Structure

```
CheckedType extends TargetType ? TrueBranch : FalseBranch
```

**Syntax rules.**

- The `extends` test uses **structural assignability**, not nominal inheritance.
- The true and false branches may be any types, including other conditional types.
- The condition may use `infer` to capture types.
- The syntax is identical for distributive and non-distributive conditional types.

**Constraints and limitations.**

- `T extends U` can be true for a wide range of types due to structural typing; ensure the condition matches your intent.
- Conditional types with unresolved type parameters defer evaluation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Concrete conditional type

```typescript
// example_ternary_form.ts
// Step 1: Define a conditional type with concrete types.
type Result = string extends number ? "yes" : "no";

// Step 2: The condition is false, so Result is "no".
const result: Result = "no";
console.log(result); // Expected: no
```

**Expected Output:**
```
no
```

**Why this output occurs.** `string` is not assignable to `number`, so the false branch `"no"` is selected.

---

#### Example 2: Nested conditional types

```typescript
// example_nested_conditional.ts
type TypeName<T> =
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  "object";

type A = TypeName<string>;  // "string"
type B = TypeName<number>;  // "number"
type C = TypeName<boolean>; // "boolean"
type D = TypeName<object>;  // "object"

const a: A = "string";
console.log(a); // Expected: string
```

**Expected Output:**
```
string
```

**Why this output occurs.** The conditional types are evaluated in order. `string` matches the first condition; `number` fails the first and matches the second; and so on.

---

### Real-World Cases

- **Type-level validation**: ensuring input types meet a condition before proceeding.
- **Discriminated type selection**: choosing a handler type based on an event name type.

---

## 3. Type Relationships

### Definitions

**Core Definition.** **Type relationships** are the assignability connections between types that conditional types test. The `extends` condition checks whether one type **is assignable to** another—whether every value of the first type is also a value of the second.

**Technical Definition.** Assignability is **structural**: `T` is assignable to `U` if every member required by `U` is present in `T` with a compatible type. Conditional types leverage this relationship to branch. The relationship is **one-directional**: `T extends U` does not imply `U extends T`.

**Beginner-Friendly Explanation.** Type relationships are **how TypeScript decides whether one type fits inside another**. `Dog extends Animal` is true because every `Dog` is also an `Animal` structurally. Conditional types ask this question at the type level.

### Purposes

- **To test whether a type satisfies a constraint** without instantiating it.
- **To filter unions** by removing types that do not satisfy a condition.
- **To extract types** that match a pattern using `infer`.
- **To build type-level assertions** that validate API contracts.

### Syntax Rules and Structure

```typescript
T extends U ? X : Y
```

**Syntax rules.**

- Assignability is **structural**, not nominal (unless `private`/`protected` members are present).
- Literal types are assignable to their base types (`"hello" extends string` is true).
- Union types distribute in conditional types when `T` is a naked type parameter.

**Constraints and limitations.**

- Structural assignability can produce surprising results; two unrelated types with the same shape may be compatible.
- Conditional types do not resolve overloads; they use the **last signature** for function types.

### Annotated Complete Step by Step Code Examples

#### Example 1: Assignability test

```typescript
// example_assignability.ts
interface Animal { live(): void }
interface Dog extends Animal { woof(): void }

type Example1 = Dog extends Animal ? number : string; // number
type Example2 = RegExp extends Animal ? number : string; // string

const e1: Example1 = 42;
const e2: Example2 = "not an animal";
console.log(e1, e2); // Expected: 42 not an animal
```

**Expected Output:**
```
42 not an animal
```

**Why this output occurs.** `Dog` is structurally assignable to `Animal` (it has all of `Animal`’s members), so `Example1` is `number`. `RegExp` does not have a `live` method, so `Example2` is `string`.

---

### Real-World Cases

- **Type compatibility checks**: validating that a component’s props extend a base props type.
- **Union filtering**: `Exclude<T, U>` removes members of `T` assignable to `U`.

---

## 4. Branching at the Type Level

### Definitions

**Core Definition.** **Branching at the type level** is the ability of conditional types to choose **different output types** based on a condition, creating a type-level decision tree.

**Technical Definition.** Each conditional type introduces a **binary branch**. Nested conditional types create a **decision tree** that resolves to a single type based on the input. Branching is **evaluated lazily** when type parameters are resolved, and **distributively** when the checked type is a naked type parameter.

**Beginner-Friendly Explanation.** Branching at the type level is **TypeScript choosing a type for you based on conditions**. You write a chain of `if/else` logic in types, and TypeScript picks the matching branch for each input type.

### Purposes

- **To implement type-level algorithms** such as type name resolution, filtering, and extraction.
- **To model conditional API behavior** where return types differ based on input flags.
- **To build recursive type utilities** like `DeepPartial` and `Awaited`.
- **To replace overloads** with a single generic signature that branches on type arguments.

### Syntax Rules and Structure

```typescript
type Branch<T> =
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  "other";
```

**Syntax rules.**

- Branching chains evaluate **in order**; the first matching branch wins.
- Each branch may be a complex type, including another conditional type.
- The final branch should provide a fallback for unmatched types.

**Constraints and limitations.**

- Deep chains can produce complex error messages and slow compilation.
- Order matters; more specific conditions should generally precede more general ones.

### Annotated Complete Step by Step Code Examples

#### Example 1: Type name resolution

```typescript
// example_branching.ts
type TypeName<T> =
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  T extends Function ? "function" :
  "object";

type T0 = TypeName<string>;           // "string"
type T1 = TypeName<"a">;              // "string" (literal extends string)
type T2 = TypeName<true>;             // "boolean"
type T3 = TypeName<() => void>;       // "function"
type T4 = TypeName<string[]>;         // "object"

const t0: T0 = "string";
console.log(t0); // Expected: string
```

**Expected Output:**
```
string
```

**Why this output occurs.** The branches are tested in order. `string` matches the first branch; `"a"` also matches because literal types extend their base types; `true` matches the boolean branch; a function type matches the function branch; and an array matches the fallback object branch.

---

### Real-World Cases

- **Event handler typing**: returning different handler types based on event name.
- **API response typing**: choosing success or error types based on a status flag.
- **Type-level validation**: ensuring input types meet a condition before proceeding.

---

## 5. `infer` Keyword

### Definitions

**Core Definition.** The **`infer` keyword** introduces a **type variable within the `extends` clause** of a conditional type, capturing a type that is discovered during matching. It is TypeScript’s mechanism for **type pattern matching and extraction**.

**Technical Definition.** `infer` may only appear in the `extends` clause of a conditional type. It declares a type variable that is inferred from the matched type structure. The inferred variable is available in the **true branch** of the conditional type. Since TypeScript 4.7, `infer X extends Y` constrains the inferred type to `Y`, rejecting matches that do not satisfy the constraint.

**Beginner-Friendly Explanation.** `infer` is **“capture whatever type fits this position.”** You write `T extends (infer U)[] ? U : never` to say: “If `T` is an array, capture its element type as `U` and return it.” It is pattern matching for types.

### Purposes

- **To extract inner types** from arrays, promises, functions, tuples, and other structures.
- **To implement utility types** such as `ReturnType`, `Parameters`, `InstanceType`, and `Awaited`.
- **To pattern-match function signatures** and capture parameter or return types.
- **To constrain extracted types** with `infer X extends Y` (TypeScript 4.7+), ensuring validated extraction.
- **To build recursive type transformations** that unwrap nested structures.

### Syntax Rules and Structure

**General syntax:**

```typescript
type Extract<T> = T extends (infer U)[] ? U : never;
```

**Constrained infer (TS 4.7+):**

```typescript
type FirstString<T> = T extends [infer S extends string, ...unknown[]] ? S : never;
```

**Component breakdown:**

- `infer U` — introduces the captured type variable.
- The position in the pattern determines what is captured.
- The true branch may use `U`.
- `infer S extends string` — constrains `S` to `string`.

**Syntax rules.**

- `infer` may only appear in the `extends` clause of a conditional type.
- The inferred variable is scoped to the true branch.
- Multiple `infer` declarations are allowed in a single conditional type.
- `infer X extends Y` requires TypeScript 4.7 or later.

**Constraints and limitations.**

- `infer` cannot be used outside conditional types.
- `infer` constraints may reject matches that would otherwise succeed, which can be surprising.
- Deeply nested `infer` patterns can hit recursion limits.

### Annotated Complete Step by Step Code Examples

#### Example 1: Extracting an array element type with `infer`

```typescript
// example_infer_array.ts
type ElementType<T> = T extends (infer U)[] ? U : never;

type A = ElementType<string[]>; // string
type B = ElementType<number[]>; // number

const a: A = "hello";
const b: B = 42;
console.log(a, b); // Expected: hello 42
```

**Expected Output:**
```
hello 42
```

**Why this output occurs.** For `string[]`, the pattern `(infer U)[]` matches with `U = string`. For `number[]`, `U = number`.

---

#### Example 2: Extracting a return type with `infer`

```typescript
// example_infer_return.ts
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Fn = (x: number) => string;
type FnReturn = MyReturnType<Fn>; // string

const result: FnReturn = "hello";
console.log(result); // Expected: hello
```

**Expected Output:**
```
hello
```

**Why this output occurs.** The pattern `(...args: any[]) => infer R` matches function types and captures the return type as `R`. For `(x: number) => string`, `R` is `string`.

---

### Real-World Cases

- **`ReturnType<T>`** : `T extends (...args: any) => infer R ? R : any`.
- **`Parameters<T>`** : `T extends (...args: infer P) => any ? P : never`.
- **`Awaited<T>`** : unwrapping promise types with `infer`.
- **Route parameter extraction**: capturing path parameters from route strings.

---

## 6. Conditional Type Declaration

### Definitions

**Core Definition.** A **conditional type declaration** is a type alias that defines a conditional type, typically with generic parameters, using the `type` keyword.

**Technical Definition.** Conditional types are declared using `type Name<T> = T extends U ? X : Y`. They may be exported, reused, and composed with other types. They participate in **type-level programming** and are erased at compile time.

**Beginner-Friendly Explanation.** A conditional type declaration is **the statement that defines a conditional type**. You give it a name and generic parameters so it can be reused across your codebase.

### Purposes

- **To name and reuse conditional type logic** across multiple call sites.
- **To document the intended type transformation** with a descriptive name.
- **To compose complex type operations** from smaller, named conditional types.
- **To export utility types** for use by other modules.

### Syntax Rules and Structure

```typescript
type ConditionalType<T> = T extends U ? X : Y;
```

**Syntax rules.**

- The declaration uses `type` and may include generic parameters with constraints.
- The conditional type may be nested or composed with other type operators.
- Conditional type aliases are erased and produce no runtime code.

**Constraints and limitations.**

- Deeply nested conditional type declarations can produce complex error messages.
- Conditional types with unresolved parameters may defer evaluation in confusing ways.

### Annotated Complete Step by Step Code Examples

#### Example 1: Declaring a reusable conditional type

```typescript
// example_conditional_declaration.ts
type IsArray<T> = T extends any[] ? true : false;

type A = IsArray<string[]>; // true
type B = IsArray<string>;   // false

const a: A = true;
const b: B = false;
console.log(a, b); // Expected: true false
```

**Expected Output:**
```
true false
```

**Why this output occurs.** The conditional type checks whether `T` is an array. `string[]` matches; `string` does not.

---

### Real-World Cases

- **Library utility types**: `Exclude`, `Extract`, and `NonNullable` are declared as conditional types.
- **API type adapters**: declaring response types that vary by status.

---

## 7. Conditional Type Usage

### Definitions

**Core Definition.** **Conditional type usage** refers to applying conditional types in type annotations, generic constraints, return types, and other type positions to achieve **type-level branching and extraction**.

**Technical Definition.** Conditional types may be used wherever a type is expected. They are commonly used as **return types** of generic functions, **generic constraints**, **mapped type value types**, and **utility type arguments**.

**Beginner-Friendly Explanation.** Conditional type usage is **using conditional types in your code**. You apply them to function return types, generic parameters, and other type positions to make types adapt to their inputs.

### Purposes

- **To make function return types depend on argument types**.
- **To filter and transform unions** in generic code.
- **To implement type-safe APIs** that adapt to input types.
- **To compose type utilities** from smaller conditional types.

### Syntax Rules and Structure

**Function return type:**

```typescript
function createLabel<T extends string | number>(idOrName: T): T extends string ? NameLabel : IdLabel { ... }
```

**Generic constraint:**

```typescript
function process<T extends string | number>(value: T extends string ? string : number) { ... }
```

**Mapped type:**

```typescript
type Nullable<T> = { [K in keyof T]: T[K] extends null ? never : T[K] };
```

**Syntax rules.**

- Conditional types may be used in any type position.
- They may reference generic parameters and `infer` variables.
- They are erased and produce no runtime code.

**Constraints and limitations.**

- Conditional types in return positions may require assertions in the implementation because the compiler cannot always verify the relationship.
- Deferred conditional types can produce less precise error messages.

### Annotated Complete Step by Step Code Examples

#### Example 1: Conditional return type

```typescript
// example_conditional_return.ts
interface IdLabel { id: number }
interface NameLabel { name: string }

function createLabel<T extends string | number>(
  idOrName: T
): T extends string ? NameLabel : IdLabel {
  throw "unimplemented";
}

const nameLabel = createLabel("Ada");   // NameLabel
const idLabel = createLabel(42);        // IdLabel

console.log(typeof nameLabel); // Expected: object (NameLabel)
```

**Expected Output:**
```
object
```

**Why this output occurs.** The return type is conditional on `T`. For `"Ada"`, `T` is `string`, so the return type is `NameLabel`. For `42`, `T` is `number`, so the return type is `IdLabel`.

---

### Real-World Cases

- **Overload replacement**: a single generic function with a conditional return type instead of multiple overloads.
- **API clients**: returning different response types based on request parameters.

---

## 8. Distributive Conditional Types

### Definitions

**Core Definition.** A **distributive conditional type** is a conditional type in which the checked type is a **naked type parameter**. When instantiated with a union, the conditional type **distributes over each union member**, applying the conditional to each and unioning the results.

**Technical Definition.** “Conditional types in which the checked type is a naked type parameter are called distributive conditional types. Distributive conditional types are automatically distributed over union types during instantiation”. To **prevent distribution**, wrap both sides of `extends` in tuple types: `[T] extends [U] ? X : Y`.

**Beginner-Friendly Explanation.** Distributive conditional types are **conditional types that automatically apply to each member of a union**. If you have `ToArray<string | number>`, it becomes `ToArray<string> | ToArray<number>`, which is `string[] | number[]`. This is usually what you want, but you can turn it off with the tuple trick.

### Purposes

- **To filter unions** by removing members that do not satisfy a condition (`Exclude`).
- **To extract union members** that match a condition (`Extract`).
- **To transform each union member independently**.
- **To build type-level mapping functions** over unions.

### Syntax Rules and Structure

**Distributive form:**

```typescript
type ToArray<T> = T extends any ? T[] : never;
type Result = ToArray<string | number>; // string[] | number[]
```

**Non-distributive form (tuple trick):**

```typescript
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;
type Result2 = ToArrayNonDist<string | number>; // (string | number)[]
```

**Syntax rules.**

- Distribution occurs only when the checked type is a **naked type parameter** (not wrapped in a tuple, array, or other construct).
- The distribution applies to **union types**; non-union types are unaffected.
- The tuple trick `[T] extends [U]` prevents distribution by wrapping both sides.

**Constraints and limitations.**

- Distribution can be surprising when you expect a single union type.
- Deeply distributed conditional types can cause performance issues with large unions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Distributive filtering

```typescript
// example_distributive.ts
type Exclude<T, U> = T extends U ? never : T;

type Result = Exclude<"a" | "b" | "c", "a">;
// Distributes: ("a" extends "a" ? never : "a") | ("b" extends "a" ? never : "b") | ...
// Result: "b" | "c"

const result: Result = "b";
console.log(result); // Expected: b
```

**Expected Output:**
```
b
```

**Why this output occurs.** The conditional type distributes over the union `"a" | "b" | "c"`. `"a"` matches `"a"` and becomes `never`; `"b"` and `"c"` do not match and remain. The result is `"b" | "c"`.

---

#### Example 2: Preventing distribution with the tuple trick

```typescript
// example_non_distributive.ts
type ToArray<T> = T extends any ? T[] : never;
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type A = ToArray<string | number>;        // string[] | number[]
type B = ToArrayNonDist<string | number>; // (string | number)[]

const b: B = ["hello", 42];
console.log(b); // Expected: [ 'hello', 42 ]
```

**Expected Output:**
```
[ 'hello', 42 ]
```

**Why this output occurs.** `ToArray` distributes, producing `string[] | number[]`. `ToArrayNonDist` wraps `T` in a tuple, preventing distribution, so the result is `(string | number)[]`.

---

### Real-World Cases

- **Union filtering**: `Exclude<T, U>` and `Extract<T, U>` rely on distribution.
- **Type-level mapping**: transforming each member of a union independently.
- **API response narrowing**: filtering response types by status.

---

## 9. Conditional Type Best Practices

### Definitions

**Core Definition.** **Conditional type best practices** are guidelines for using conditional types in ways that maximize **type safety, readability, and compiler performance**, while avoiding excessive complexity, unnecessary distribution, and deferred-evaluation surprises.

**Technical Definition.** Best practices center on **preferring built-in utility types** (`Exclude`, `Extract`, `NonNullable`, `ReturnType`, `Parameters`) over custom implementations, **understanding distribution** and using the tuple trick when distribution is unwanted, **using `infer` with constrained patterns**, **providing fallback branches**, **keeping conditional types shallow and composable**, and **documenting complex conditional types**.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep conditional types safe and maintainable. The big rules are: use built-in utility types when you can; remember that union types distribute; wrap in tuples to stop distribution; always provide a fallback branch; and keep conditional types simple and readable.

### Purposes

- **To maximize type safety** by using well-tested utility types and constrained `infer`.
- **To avoid unexpected distribution** by using the tuple trick.
- **To keep conditional types readable** by composing named aliases.
- **To prevent compiler performance issues** from deep nesting and large unions.
- **To provide robust fallbacks** for unmatched types.

### Syntax Rules and Structure

Best practices are conventions that interact with conditional type syntax. The core rules are:

1. **Prefer built-in utility types.** `Exclude`, `Extract`, `NonNullable`, `ReturnType`, and `Parameters` are already implemented and optimized.
2. **Understand distribution.** Naked type parameters distribute over unions. Use `[T] extends [U]` to prevent distribution when needed.
3. **Always provide a fallback branch.** The false branch should handle unmatched types gracefully (often `never` or the original type).
4. **Constrain `infer` patterns.** Use `infer X extends Y` to validate extracted types (TypeScript 4.7+).
5. **Keep conditional types shallow and composable.** Break complex logic into named aliases rather than deeply nested conditionals.
6. **Document complex conditional types.** Add comments explaining the intended transformation and fallback behavior.
7. **Test edge cases.** Verify behavior with unions, literals, `never`, `any`, and `unknown`.
8. **Avoid overusing conditional types** when simpler types (unions, overloads) would suffice.

**Component breakdown (conditional type decision tree):**

```
Do you need to transform types based on a condition?
  ├── Yes → Can a built-in utility type do it?
  │        ├── Yes → Use Exclude / Extract / NonNullable / ReturnType.
  │        └── No → Write a conditional type.
  │                 ├── Does it involve a union? → Decide on distribution.
  │                 │   ├── Distributive → Use a naked type parameter.
  │                 │   └── Non-distributive → Wrap in tuples: [T] extends [U].
  │                 ├── Need to extract a type? → Use infer (constrain with extends).
  │                 └── Provide a fallback branch.
  └── No → Use simpler types.
```

**Constraints and limitations.**

- Deeply nested conditional types can produce complex error messages and slow compilation.
- Distribution can be surprising; always test with unions.
- `infer` constraints may reject matches that would otherwise succeed.

### Annotated Complete Step by Step Code Examples

#### Example 1: Prefer built-in utility types

```typescript
// example_best_builtin.ts
// ❌ Custom implementation duplicates built-in behavior.
// type MyExclude<T, U> = T extends U ? never : T;

// ✅ Use the built-in.
type Result = Exclude<"a" | "b" | "c", "a">; // "b" | "c"

const result: Result = "c";
console.log(result); // Expected: c
```

**Expected Output:**
```
c
```

**Why this output occurs.** The built-in `Exclude` implements the same distributive conditional logic and is optimized by the compiler. Reimplementing it adds maintenance burden without benefit.

---

#### Example 2: Prevent unintended distribution

```typescript
// example_best_distribution.ts
// ❌ Distributive: unexpected for a single union type.
type WrapDist<T> = T extends any ? T[] : never;
type A = WrapDist<string | number>; // string[] | number[]

// ✅ Non-distributive: wraps the whole union.
type WrapNonDist<T> = [T] extends [any] ? T[] : never;
type B = WrapNonDist<string | number>; // (string | number)[]

const b: B = ["hello", 42];
console.log(b); // Expected: [ 'hello', 42 ]
```

**Expected Output:**
```
[ 'hello', 42 ]
```

**Why this output occurs.** The tuple trick `[T] extends [any]` prevents the conditional type from distributing, so the union is wrapped as a whole.

---

#### Example 3: Constrained `infer` for validated extraction

```typescript
// example_best_infer.ts
// ✅ Constrained infer: only captures string first elements.
type FirstString<T> = T extends [infer S extends string, ...unknown[]] ? S : never;

type A = FirstString<["hello", 42]>; // "hello"
type B = FirstString<[42, "hello"]>; // never

const a: A = "hello";
console.log(a); // Expected: hello
```

**Expected Output:**
```
hello
```

**Why this output occurs.** The constraint `S extends string` ensures that only a string first element is captured. Non-string first elements cause the match to fail and return `never`.

---

### Real-World Cases

- **Library utility types**: implementing `Exclude`, `Extract`, and `NonNullable` correctly with distribution.
- **API response typing**: using non-distributive wrapping when a union should be handled as a single type.
- **Type extraction**: using constrained `infer` for route parameters, function returns, and promise values.

---

## References

- Documentation - Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- Documentation - TypeScript 2.8 (Conditional Types) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html
- Documentation - TypeScript 4.7 (`extends` Constraints on `infer` Type Variables) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-7.html
- TypeScript Playground - Conditional Types — https://www.typescriptlang.org/play/typescript/meta-types/conditional-types.ts.html
- How to Handle Conditional Types in TypeScript — https://github.com/OneUptime/blog/tree/master/posts/2026-01-24-handle-conditional-types-typescript
- TypeScript Conditional Types - GeeksforGeeks — https://origin.geeksforgeeks.org/typescript/typescript-conditional-types/
- TypeScript "infer" Tips — https://blog.kinto-technologies.com/posts/typescript-infer-tips/
- mcollina/skills - Infer Keyword Best Practices — https://github.com/mcollina/skills/blob/main/skills/typescript-magician/rules/infer-keyword.md
- Advanced TypeScript Type Patterns (Distributive Gotcha) — https://cdn.jsdelivr.net/npm/@typescript-eslint/type-utils@latest/dist/index.d.ts