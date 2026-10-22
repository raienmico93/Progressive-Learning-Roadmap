# Union Types in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **union type** is a type formed by combining two or more types with the vertical bar operator (`|`), representing a value that can be **one of several types**. It is the type-level equivalent of a logical **OR**: a value of type `A | B` is either an `A` or a `B`. Union types are a **type-space construct**—they exist only at compile time and are erased during compilation.

**Technical Definition.** In TypeScript's type grammar, a union type `T₁ | T₂ | … | Tₙ` denotes the **set-theoretic union** of its constituent types. A value is assignable to the union if it is assignable to **at least one** member. When reading from a union, only **properties common to all members** are accessible (the intersection of their members). **Control-flow analysis** narrows union types in conditional branches based on type guards (`typeof`, `instanceof`, `in`, equality checks, custom predicates), enabling safe access to member-specific properties. Since TypeScript 3.3, unions of callable types can be invoked if their parameters intersect to a valid call signature.

**Beginner-Friendly Explanation.** A union type says: **"This value can be one of these types."** `string | number` means the value is either a string or a number—nothing else. It is TypeScript's way of saying "OR" at the type level. When you have a union, TypeScript only lets you use properties that **every** member shares, unless you first **narrow** the type with a check (like `typeof value === "string"`) to tell the compiler which specific type it is in that branch.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **OR semantics** | A union `A \| B` accepts any value assignable to `A` or `B`. |
| **Common property access** | Only properties present on **every** member are accessible without narrowing. |
| **Narrowing required** | Accessing member-specific properties requires a type guard to narrow the union. |
| **Assignability** | A value assignable to one member is assignable to the union; the reverse is not true. |
| **Discriminated unions** | A common literal property (discriminant) enables automatic narrowing in `switch`/`if` branches. |
| **Exhaustiveness checking** | The `never` type can verify that all union members are handled. |
| **Performance consideration** | Union type checking is **quadratic**; very large unions (50+ members) cause measurable compilation slowdowns. |
| **Erasure** | Union types exist only at compile time and emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: type annotations, type aliases, and interfaces.
- **Primitive types**: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`.
- **Literal types**: `"hello"`, `42`, `true`.
- **Control flow**: `if`/`else`, `switch`, and logical operators.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Type narrowing and type guards** — the mechanism for working with unions safely.
- **Discriminated unions** — the primary pattern for modeling state and variants.
- **Intersection types** (`&`) — the dual of unions (AND semantics).
- **Template literal types** — unions of string literals generated from patterns.
- **Utility types** — `Exclude<T, U>`, `Extract<T, U>`, and `NonNullable<T>` operate on unions.
- **Performance optimization** — preferring base types over large unions.

---

## Core Concepts / Features

---

## 1. Union Types

### Definitions

**Core Definition.** A **union type** describes a value that can be **one of several types**, declared with the `|` operator. It is TypeScript's type-level OR.

**Technical Definition.** A union type `A | B | C` is the set of all values that belong to at least one of `A`, `B`, or `C`. The compiler permits assignment of a value to the union if it is assignable to any member. Reading from the union exposes only the **intersection of members' properties**. Control-flow analysis narrows the union in branches guarded by type checks. Union types are **erased** and have no runtime representation.

**Beginner-Friendly Explanation.** A union type is **a list of allowed types**. `string | number` means "a string OR a number." It is useful when a value can legitimately be more than one type, such as a function that accepts either a string or an array of strings.

### Purposes

- **To express that a value can be one of several types** without falling back to `any`.
- **To model real-world variability** such as "a command-line argument is a string, a string array, or a function."
- **To improve type inference** in arrays and collections with mixed value kinds.
- **To enable precise modeling of states and variants** through discriminated unions.
- **To support flexible APIs** that accept multiple input shapes.

### Syntax Rules and Structure

**General syntax:**

```
type UnionName = Type1 | Type2 | Type3;
```

**Inline union in a parameter:**

```
function padLeft(value: string, padding: string | number): string { ... }
```

**Component breakdown:**

- `Type1 | Type2 | Type3` — the union of constituent types, separated by `|`.
- The leading `|` in multi-line unions is optional and stylistic: `type T = | A | B | C;`.

**Syntax rules.**

- A union may contain any number of constituent types, including primitives, literals, object types, interfaces, type aliases, and other unions (which are flattened).
- `null` and `undefined` may be included: `string | null | undefined`.
- Literal types may be combined: `"idle" | "loading" | "success" | "error"`.
- Unions may be generic: `type Result<T> = T | Error`.
- Unions may be used inline in annotations, parameters, and return types.

**Constraints and limitations.**

- **Only common properties are accessible** without narrowing.
- **Assignability is one-way**: a member type is assignable to the union, but the union is not assignable to a member.
- **Unions of callable types** could not be invoked if parameter lists differed before TypeScript 3.3; since 3.3, parameters are intersected.
- **Very large unions** (50+ members) cause **quadratic** type-checking performance.
- **Interfaces cannot extend union types**; use type aliases for unions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic union type

```typescript
// example_union_basic.ts
// Step 1: Declare a union type.
type StringOrNumber = string | number;

// Step 2: Assign values of either type.
let value: StringOrNumber = "hello";
console.log(value); // Expected: hello

value = 42;
console.log(value); // Expected: 42

// value = true; // Error: boolean is not assignable to string | number.
```

**Setup.** Compile with `npx tsc example_union_basic.ts --strict`.

**Expected Output:**
```
hello
42
```

**Why this output occurs.** `StringOrNumber` accepts any value that is a `string` or a `number`. Both assignments succeed. `true` is rejected because `boolean` is not a member of the union. At runtime, the union type is erased; only the values remain.

---

#### Example 2: Union in a function parameter

```typescript
// example_union_function.ts
function padLeft(value: string, padding: string | number): string {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  return padding + value;
}

console.log(padLeft("Hello", 4));      // Expected: "    Hello"
console.log(padLeft("Hello", ">> "));  // Expected: ">> Hello"
```

**Expected Output:**
```
    Hello
>> Hello
```

**Why this output occurs.** The `padding` parameter is `string | number`. The `typeof` check narrows `padding` to `number` in the first branch and `string` in the second. This allows the compiler to verify that `.join()` and string concatenation are valid in their respective branches.

---

#### Example 3: Common properties on unions

```typescript
// example_union_common.ts
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

function handlePet(pet: Bird | Fish) {
  pet.layEggs(); // ✅ OK: both Bird and Fish have layEggs
  // pet.fly();  // ❌ Error: only Bird has fly
  // pet.swim(); // ❌ Error: only Fish has swim
}

console.log("Only common properties are accessible");
```

**Expected Output:**
```
Only common properties are accessible
```

**Why this output occurs.** `Bird | Fish` only guarantees the properties present on **every** member. `layEggs` exists on both, so it is accessible. `fly` exists only on `Bird` and `swim` only on `Fish`, so they require narrowing.

---

### Real-World Cases

- **API parameters**: `function request(url: string, data: string | object): Promise<Response>`.
- **Configuration values**: `type LogLevel = "debug" | "info" | "warn" | "error"`.
- **State modeling**: `type Status = "idle" | "loading" | "success" | "error"`.
- **Command-line arguments**: `string | string[] | (() => string)`.

---

## 2. `|` Operator

### Definitions

**Core Definition.** The **`|` operator** is the syntactic token that combines two or more types into a union type. It is TypeScript's type-level analog of the logical OR.

**Technical Definition.** In TypeScript's grammar, the vertical bar `|` appears between type expressions in a union type. It has lower precedence than intersection (`&`) and is left-associative. Duplicate members are **collapsed** (e.g., `string | string` is `string`). Unions of literal types produce a finite set of permitted values.

**Beginner-Friendly Explanation.** The `|` symbol is **the "OR" of the type world**. You put it between types to say "this value can be the first type OR the second type OR the third type." It is the same symbol used for bitwise OR in JavaScript, but in TypeScript's type position it means type union.

### Purposes

- **To combine types into a union** concisely.
- **To create literal unions** such as `"idle" | "loading" | "success"`.
- **To model optional or nullable values** as `T | undefined` or `T | null`.
- **To flatten nested unions** through associativity.

### Syntax Rules and Structure

**General syntax:**

```
type Union = A | B | C;
```

**Component breakdown:**

- `A`, `B`, `C` — constituent types.
- `|` — the union operator.

**Syntax rules.**

- `|` has lower precedence than `&`; `A & B | C` is `(A & B) | C`.
- Unions are left-associative: `A | B | C` is `(A | B) | C`.
- Duplicate members are collapsed: `string | string` becomes `string`.
- Leading `|` in multi-line unions is stylistic and optional.

**Constraints and limitations.**

- `|` cannot be used to combine **values**; it is a type-level operator only.
- `|` cannot appear in index signature key types before TypeScript 4.4 (union key types are allowed since 4.4).
- `|` cannot be used to extend an interface; interfaces cannot extend union types.

### Annotated Complete Step by Step Code Examples

#### Example 1: Literal union with `|`

```typescript
// example_pipe_literal.ts
type LoadingState = "idle" | "loading" | "success" | "error";

let state: LoadingState = "idle";
console.log(state); // Expected: idle

state = "success";
console.log(state); // Expected: success

// state = "done"; // Error: "done" is not assignable to LoadingState.
```

**Expected Output:**
```
idle
success
```

**Why this output occurs.** The `|` operator combines four string literal types into a finite union. Only those four strings are assignable. The compiler rejects `"done"` because it is not a member.

---

#### Example 2: Precedence of `|` vs. `&`

```typescript
// example_pipe_precedence.ts
type A = { a: string };
type B = { b: number };
type C = { c: boolean };

// & binds tighter than |: (A & B) | C
type Combined = A & B | C;

const x: Combined = { a: "hello", b: 42 }; // ✅ Matches A & B
const y: Combined = { c: true };            // ✅ Matches C

console.log(x.a, x.b); // Expected: hello 42
console.log(y.c);      // Expected: true
```

**Expected Output:**
```
hello 42
true
```

**Why this output occurs.** `&` has higher precedence than `|`, so `A & B | C` is `(A & B) | C`. The first object satisfies `A & B`; the second satisfies `C`. The union accepts either.

---

### Real-World Cases

- **HTTP methods**: `type Method = "GET" | "POST" | "PUT" | "DELETE"`.
- **Nullable values**: `type MaybeString = string | null | undefined`.
- **Event names**: `type EventName = "click" | "focus" | "blur" | "submit"`.

---

## 3. Union Type Declaration

### Definitions

**Core Definition.** A **union type declaration** is a type alias or inline annotation that defines a union type by listing its constituent types separated by `|`.

**Technical Definition.** Union types may be declared as **named type aliases** (`type StringOrNumber = string | number`) or used **inline** in annotations, parameters, return types, and generic arguments. Interfaces **cannot** directly declare a union type; union types are the exclusive domain of type aliases and inline type expressions. Union types may be **generic** (`type Result<T> = T | Error`) and may reference other unions (which are flattened).

**Beginner-Friendly Explanation.** A union type declaration is **the statement that defines a union**. It either gives the union a name (using `type`) or uses it directly in a function signature. It lists the allowed types with `|` between them.

### Purposes

- **To name a union** for reuse across a codebase.
- **To define the allowed value set** for a parameter or property.
- **To parameterize unions** with generics for reusable result and option types.
- **To document the contract** of APIs that accept multiple input shapes.

### Syntax Rules and Structure

**General syntax (named union):**

```
type UnionName = Type1 | Type2 | Type3;
```

**General syntax (generic union):**

```
type Result<T> = T | Error;
```

**General syntax (inline union):**

```
function f(param: Type1 | Type2): ReturnType { ... }
```

**Component breakdown:**

- `type UnionName` — the named union alias.
- `= Type1 | Type2 | Type3` — the constituent types.
- Generic parameters may be used in any constituent.

**Syntax rules.**

- Union aliases may be **generic** and may use **constraints**.
- Unions may include **literal types**, **primitive types**, **object types**, **interfaces**, and **other unions**.
- `null` and `undefined` may be included explicitly.
- Unions may be **recursive** (through type aliases, since TS 3.7).

**Constraints and limitations.**

- **Interfaces cannot extend unions**; use type aliases.
- Union aliases are **closed** (cannot be declaration-merged).
- Very large unions impact compiler performance.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generic union alias

```typescript
// example_union_generic.ts
type Result<T> = T | Error;

function parseJSON<T>(input: string): Result<T> {
  try {
    return JSON.parse(input) as T;
  } catch (e) {
    return e instanceof Error ? e : new Error("Unknown error");
  }
}

const result = parseJSON<{ name: string }>('{"name":"Ada"}');
if (!(result instanceof Error)) {
  console.log(result.name); // Expected: Ada
}
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `Result<T>` is a generic union of `T` and `Error`. The function returns either the parsed value or an `Error`. The `instanceof Error` check narrows the union, allowing safe access to `result.name` in the success branch.

---

#### Example 2: Union with `null` and `undefined`

```typescript
// example_union_nullable.ts
type NullableString = string | null | undefined;

function getLength(value: NullableString): number {
  if (value == null) {
    return 0;
  }
  return value.length;
}

console.log(getLength("hello")); // Expected: 5
console.log(getLength(null));    // Expected: 0
console.log(getLength(undefined)); // Expected: 0
```

**Expected Output:**
```
5
0
0
```

**Why this output occurs.** `NullableString` includes `string`, `null`, and `undefined`. The `value == null` check narrows the union to `string` in the `else` branch (since `==` matches both `null` and `undefined`), allowing `.length` access.

---

### Real-World Cases

- **Result types**: `type Result<T, E = Error> = { ok: true; data: T } | { ok: false; error: E }`.
- **Option types**: `type Option<T> = T | null`.
- **API parameters**: `function request(url: string, body: string | object | FormData)`.

---

## 4. Union Type Narrowing

### Definitions

**Core Definition.** **Union type narrowing** is the process by which TypeScript's **control-flow analysis** refines a union type to a **more specific member type** within a conditional branch, based on a **type guard** check.

**Technical Definition.** Narrowing occurs when the compiler can prove that a value belongs to a specific member of a union based on runtime checks. Supported guards include `typeof` checks (primitives), `instanceof` checks (classes and built-ins), `in` checks (property existence), equality checks (`===`, `!==`, `==`, `!=`), truthiness checks, and **user-defined type predicates** (`value is Type`). Narrowing propagates through `if`/`else`, `switch`, ternary expressions, logical operators, and early returns. The `never` type can be used for **exhaustiveness checking**: if all members are handled, the remaining value has type `never`.

**Beginner-Friendly Explanation.** Narrowing is **how TypeScript figures out which specific type a union value actually is** inside a conditional block. When you write `if (typeof value === "string")`, TypeScript knows that inside that `if`, `value` is a string—not a number. This lets you safely use string-specific methods. Narrowing is what makes unions practical: without it, you could never access type-specific properties.

### Purposes

- **To safely access member-specific properties** on union values.
- **To handle each variant** of a discriminated union in a type-safe way.
- **To enable exhaustiveness checking** so no union member is forgotten.
- **To model state machines** and variant data with compile-time safety.
- **To replace unsafe type assertions** with checked narrowing.

### Syntax Rules and Structure

**Common narrowing mechanisms:**

| Guard | Syntax | Narrows |
|---|---|---|
| `typeof` | `typeof x === "string"` | Primitive types |
| `instanceof` | `x instanceof Date` | Class instances |
| `in` | `"prop" in x` | Object types with the property |
| Equality | `x === "literal"` | Literal types |
| Truthiness | `if (x)` | Removes `null`, `undefined`, `""`, `0`, `false` |
| Type predicate | `function isT(x): x is T` | Custom narrowing |
| Discriminant | `switch (x.kind)` | Discriminated unions |

**Syntax rules.**

- Narrowing applies within the branch where the guard is true; the complementary branch narrows to the remaining members.
- `switch` on a discriminant property narrows each `case` block.
- Early returns and `throw` statements narrow the remaining code.
- User-defined type predicates use the `parameterName is Type` return type syntax.

**Constraints and limitations.**

- Narrowing is **flow-sensitive**; it does not persist across function boundaries unless the type predicate is used.
- **Nested discriminants** do not narrow automatically; the discriminant must be at the same level.
- **Large unions** (11+ members) may have different nullability narrowing behavior.
- **Unions of callable types** have restrictions on invocation (only callable since TS 3.3 under certain conditions).

### Annotated Complete Step by Step Code Examples

#### Example 1: `typeof` narrowing

```typescript
// example_narrow_typeof.ts
function format(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase(); // Narrowed to string
  }
  return value.toFixed(2); // Narrowed to number
}

console.log(format("hello")); // Expected: HELLO
console.log(format(3.14159)); // Expected: 3.14
```

**Expected Output:**
```
HELLO
3.14
```

**Why this output occurs.** The `typeof value === "string"` check narrows `value` to `string` in the `if` branch (enabling `.toUpperCase()`) and to `number` in the `else` branch (enabling `.toFixed()`).

---

#### Example 2: `in` narrowing

```typescript
// example_narrow_in.ts
interface Admin {
  role: "admin";
  permissions: string[];
}

interface User {
  role: "user";
  email: string;
}

function describe(person: Admin | User) {
  if ("permissions" in person) {
    console.log(person.permissions.length); // Narrowed to Admin
  } else {
    console.log(person.email); // Narrowed to User
  }
}

describe({ role: "admin", permissions: ["read", "write"] }); // Expected: 2
describe({ role: "user", email: "ada@example.com" });          // Expected: ada@example.com
```

**Expected Output:**
```
2
ada@example.com
```

**Why this output occurs.** The `"permissions" in person` check narrows `person` to `Admin` in the `if` branch, where `permissions` is available. The `else` branch narrows to `User`, where `email` is available.

---

#### Example 3: Discriminated union narrowing

```typescript
// example_narrow_discriminated.ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2; // Narrowed to circle
    case "square":
      return shape.side ** 2; // Narrowed to square
  }
}

console.log(area({ kind: "circle", radius: 1 }).toFixed(2)); // Expected: 3.14
console.log(area({ kind: "square", side: 2 }));               // Expected: 4
```

**Expected Output:**
```
3.14
4
```

**Why this output occurs.** The `kind` discriminant is a literal type in each union member. The `switch (shape.kind)` narrows `shape` to the corresponding member in each `case`, enabling access to `radius` or `side` respectively.

---

#### Example 4: Exhaustiveness checking with `never`

```typescript
// example_narrow_never.ts
type Status = "idle" | "loading" | "success" | "error";

function assertNever(value: never): never {
  throw new Error(`Unhandled case: ${JSON.stringify(value)}`);
}

function handleStatus(status: Status): string {
  switch (status) {
    case "idle": return "Ready";
    case "loading": return "Loading...";
    case "success": return "Done";
    case "error": return "Failed";
    default: return assertNever(status); // ✅ Compiles: all members handled
  }
}

console.log(handleStatus("idle")); // Expected: Ready
```

**Expected Output:**
```
Ready
```

**Why this output occurs.** After handling all four members, the `status` in the `default` branch has type `never`. `assertNever` accepts `never`, so the code compiles. If a new member were added to `Status` and not handled, the `default` branch would receive a non-`never` type and the compiler would error, forcing exhaustive handling.

---

### Real-World Cases

- **State machines**: `switch (state.status)` with exhaustive handling of `"idle"`, `"loading"`, `"success"`, `"error"`.
- **API response handling**: `if ("error" in response) { ... } else { ... }`.
- **Form validation**: `if (typeof field.value === "string") { ... }`.
- **Error handling**: `if (e instanceof ValidationError) { ... }`.

---

## 5. Union Type Best Practices

### Definitions

**Core Definition.** **Union type best practices** are guidelines for designing and using unions in ways that maximize **type safety, compiler performance, and maintainability**, while avoiding the pitfalls of overly large unions, improper narrowing, and modeling invalid states.

**Technical Definition.** Best practices center on **using discriminated unions for structured variants**, **avoiding large flat unions** (50+ members cause quadratic checking), **grouping moderately large unions** (20–50 members) into discriminated categories, **using exhaustive checks with `never`**, **preferring literal unions over enums for zero-runtime closed sets**, **narrowing with type guards instead of assertions**, and **modeling states so that invalid combinations are impossible**.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep unions from becoming a mess. The big rules are: use a **discriminator** (a common literal property) when you have structured variants; do not create enormous flat unions—group them or use branded types; use `switch` with `never` to ensure you handle every case; and model your data so that impossible states cannot be represented.

### Purposes

- **To maximize type safety** by making invalid states unrepresentable.
- **To ensure exhaustiveness** so no union member is forgotten when new members are added.
- **To maintain compiler performance** by avoiding quadratic checking on large unions.
- **To improve readability** by using discriminants and clear naming.
- **To enable safe narrowing** without type assertions.

### Syntax Rules and Structure

Best practices are conventions that interact with union syntax. The core rules are:

1. **Use discriminated unions for structured variants.** Add a common literal property (e.g., `kind`, `type`, `status`) to each member. Switch on it to narrow automatically.
2. **Use `switch` with `never` for exhaustiveness.** A `default` branch calling `assertNever(value: never)` ensures all members are handled. Add this check whenever a new member is added.
3. **Avoid flat unions with 50+ members.** Union checking is quadratic; large unions cause compilation slowdowns and IDE lag. For 20–50 members, group them into discriminated categories; for 50+, use a branded string type with runtime validation.
4. **Prefer literal unions over `enum` for closed sets.** `type Status = "idle" | "loading" | "success"` has zero runtime cost and better tree-shaking than numeric enums.
5. **Narrow with type guards, not assertions.** Use `typeof`, `instanceof`, `in`, and custom predicates (`value is T`) instead of `as` casts.
6. **Model states so invalid combinations are impossible.** Use discriminated unions to represent mutually exclusive states, rather than optional properties that can coexist illegally.
7. **Name union members clearly.** Use descriptive literal values and interface names that explain each variant.

**Component breakdown (union design decision tree):**

```
Are the variants structured objects with a common discriminant?
  ├── Yes → Use a discriminated union (switch on the discriminant).
  └── No → Is the union small (< 20 literal members)?
           ├── Yes → Use a flat literal union.
           └── No → Is it 20–50 members?
                    ├── Yes → Group into discriminated categories.
                    └── No (50+) → Use a branded string type with runtime validation.
```

**Constraints and limitations.**

- Discriminated unions require a common literal property on every member.
- Nested discriminants do not narrow automatically; the discriminant must be at the same level.
- Exhaustiveness checking with `never` requires a `default` branch and a helper function.
- Large unions may still be unavoidable for generated types; mitigate with grouping or branding.

### Annotated Complete Step by Step Code Examples

#### Example 1: Discriminated union with exhaustive switch

```typescript
// example_best_discriminated.ts
type FetchState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

function assertNever(value: never): never {
  throw new Error(`Unhandled: ${JSON.stringify(value)}`);
}

function render<T>(state: FetchState<T>): string {
  switch (state.status) {
    case "idle": return "Ready";
    case "loading": return "Loading...";
    case "success": return `Data: ${JSON.stringify(state.data)}`;
    case "error": return `Error: ${state.error.message}`;
    default: return assertNever(state);
  }
}

console.log(render({ status: "idle" }));    // Expected: Ready
console.log(render({ status: "success", data: { name: "Ada" } })); // Expected: Data: {"name":"Ada"}
```

**Expected Output:**
```
Ready
Data: {"name":"Ada"}
```

**Why this output occurs.** The `status` discriminant narrows the union in each `case`. The `assertNever` call in the `default` branch ensures that adding a new member without handling it causes a compile-time error. This is the recommended pattern for exhaustive state handling.

---

#### Example 2: Grouping large unions into discriminated categories

```typescript
// example_best_grouping.ts
// ❌ 200+ flat union members cause quadratic checking.
// type AnalyticsEvent = 'page_view' | 'button_click' | ...;

// ✅ Group into discriminated categories.
type UserEvent = { category: "user"; action: "login" | "logout" | "signup" };
type PageEvent = { category: "page"; action: "view" | "scroll" | "leave" };
type FormEvent = { category: "form"; action: "submit" | "validate" | "reset" };

type AppEvent = UserEvent | PageEvent | FormEvent;

function handleEvent(event: AppEvent) {
  switch (event.category) {
    case "user":
      console.log(`User action: ${event.action}`); // Narrowed to UserEvent
      break;
    case "page":
      console.log(`Page action: ${event.action}`); // Narrowed to PageEvent
      break;
    case "form":
      console.log(`Form action: ${event.action}`); // Narrowed to FormEvent
      break;
  }
}

handleEvent({ category: "user", action: "login" }); // Expected: User action: login
```

**Expected Output:**
```
User action: login
```

**Why this output occurs.** Instead of a flat union of 200+ string literals, the events are grouped into three discriminated categories. The outer union has only **3 members**, avoiding quadratic checking, while each category narrows to its specific `action` union.

---

#### Example 3: Modeling invalid states as impossible

```typescript
// example_best_invalid_states.ts
// ❌ Invalid states are representable: loading AND error can coexist.
// interface State {
//   loading: boolean;
//   error?: string;
//   data?: string;
// }

// ✅ Discriminated union makes invalid states impossible.
type State =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: string }
  | { status: "error"; error: string };

function display(state: State): string {
  switch (state.status) {
    case "idle": return "Waiting";
    case "loading": return "Loading...";
    case "success": return state.data; // data only exists on success
    case "error": return state.error;  // error only exists on error
  }
}

console.log(display({ status: "success", data: "Hello" })); // Expected: Hello
```

**Expected Output:**
```
Hello
```

**Why this output occurs.** The discriminated union ensures that `data` exists only in the `success` state and `error` only in the `error` state. It is **impossible** to represent a state where both `loading` and `error` are true. The compiler enforces this at every use site.

---

### Real-World Cases

- **React state management**: `type State<T> = { status: "idle" } | { status: "loading" } | { status: "success"; data: T } | { status: "error"; error: Error }`.
- **API responses**: `type ApiResponse<T> = { ok: true; data: T } | { ok: false; error: string }`.
- **Form validation**: `type FieldState = { status: "valid"; value: string } | { status: "invalid"; errors: string[] }`.
- **Analytics events**: grouping large event unions into discriminated categories to avoid performance degradation.

---

## References

- TypeScript Handbook — Everyday Types: Union Types - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types
- TypeScript Handbook — Unions and Intersection Types (Deprecated) - https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html
- TypeScript 1.4 Release Notes — Union Types - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-4.html
- TypeScript 3.3 Release Notes — Improved Behavior for Calling Union Types - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-3.html
- TypeScript Playground — Discriminate Types - https://www.typescriptlang.org/play/typescript/meta-types/discriminate-types.ts.html
- TypeScript Handbook — Narrowing - https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- TypeScript Handbook — The `never` Type (Exhaustiveness Checking) - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#the-never-type
- Understanding Union Types and Type Guards in TypeScript — Educative - https://www.educative.io/courses/rails-for-front-end-development-essential-tools/lta/union-types-and-type-guards
- Unions, Intersections, and Type Guards — Steve Kinney - https://stevekinney.com/courses/react-typescript/typescript-unions-intersections-guards
- Discriminated Unions — Steve Kinney - https://stevekinney.com/courses/react-typescript/typescript-discriminated-unions
- Avoid Large Union Types — pproenca/dot-skills - https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.curated/typescript/references/type-avoid-large-unions.md
- Common Keys of Unions of Objects — Total TypeScript - https://www.totaltypescript.com/common-keys-of-unions
- Exhaustive Checks — pproenca/dot-skills - https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.curated/typescript/references/safety-exhaustive-checks.md
- TypeScript Performance — Preferring Base Types Over Unions - https://github.com/microsoft/TypeScript/wiki/Performance#preferring-base-types-over-unions