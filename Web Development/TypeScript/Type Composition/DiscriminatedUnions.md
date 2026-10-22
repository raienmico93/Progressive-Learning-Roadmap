# Discriminated Unions in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **discriminated union** (also called a **tagged union** or **algebraic data type**) is a union type in which every member shares a common **literal-typed property**—the **discriminant**—whose value uniquely identifies that member. The discriminant enables TypeScript's **control-flow analysis** to narrow the union to a specific member when the discriminant is checked, providing type-safe access to member-specific properties and enabling **exhaustiveness checking**.

**Technical Definition.** A discriminated union is a union of object types that each declare a property (conventionally named `kind`, `type`, `status`, or `tag`) whose type is a **unit type** (a string literal, numeric literal, `null`, or `undefined`). When the compiler encounters a check on this discriminant (via `switch`, `if`, or equality), it narrows the union in the guarded branch to the member whose discriminant matches. This narrowing is **flow-sensitive**: it propagates through `switch` cases, `if`/`else` branches, early returns, and logical operators. When all members are handled, the remaining type is `never`, which can be used to enforce **exhaustiveness**. Discriminated unions are **erased** at compile time and have no runtime representation.

**Beginner-Friendly Explanation.** A discriminated union is **a union where every member carries a label**. That label—the discriminant—is a fixed value like `"circle"` or `"square"`. When you check the label (e.g., `switch (shape.kind)`), TypeScript can tell exactly which shape you are working with, and it lets you safely use the properties that belong only to that shape. Think of it as a set of sealed envelopes: each envelope has a label on the outside, and you can only open the contents once you have checked the label. If you add a new kind of envelope and forget to handle it, TypeScript warns you.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Common discriminant** | Every member shares a property with a literal type that uniquely identifies it. |
| **Automatic narrowing** | Checking the discriminant narrows the union to the matching member. |
| **Exhaustiveness checking** | Assigning the residual type to `never` verifies that all members are handled. |
| **Invalid-state prevention** | Members carry only the fields relevant to their variant, making illegal combinations unrepresentable. |
| **`switch`-friendly** | `switch` on the discriminant is the idiomatic narrowing pattern. |
| **Erasure** | Discriminated unions exist only at compile time and emit no JavaScript. |
| **Literal-type requirement** | The discriminant must be a unit type (`"a"`, `42`, `null`, `undefined`); methods and non-literal types do not work as discriminants. |

---

### Prerequisites

- **TypeScript basics**: type aliases, interfaces, and union types.
- **Literal types**: `"idle"`, `42`, `true`.
- **Control flow**: `switch`, `if`/`else`, and early returns.
- **The `never` type** and type narrowing.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Union types** — discriminated unions are a specialized form of union.
- **Type narrowing and type guards** — the mechanism that makes discriminated unions work.
- **Algebraic data types (ADTs)** — the theoretical foundation (sum types).
- **State machines** — discriminated unions model mutually exclusive states.
- **Pattern matching** — `switch` on discriminants is TypeScript's closest analog.

---

## Core Concepts / Features

---

## 1. Discriminated Unions

### Definitions

**Core Definition.** A **discriminated union** is a union type whose members each declare a common literal-typed **discriminant** property, enabling the compiler to narrow the union to a specific member when the discriminant is checked.

**Technical Definition.** A discriminated union is a union of object types where every member has a property (the discriminant) whose type is a **unit type**. TypeScript uses control-flow analysis to narrow the union when the discriminant is compared to a literal. The narrowing is **exhaustive** when all members are handled, leaving `never`. Discriminated unions are the recommended way to model **variant data**—values that take one of several mutually exclusive shapes.

**Beginner-Friendly Explanation.** A discriminated union is **a union with a label on each member**. The label tells TypeScript which member you are working with when you check it. It is the best way to model data that can be one of several different things, like a shape that is either a circle or a square.

### Purposes

- **To model mutually exclusive variants** with precise, type-safe shapes.
- **To enable automatic narrowing** without unsafe type assertions.
- **To enforce exhaustiveness** so no variant is forgotten when new ones are added.
- **To make invalid states unrepresentable** by giving each variant only its relevant fields.
- **To replace optional-field “god objects”** that allow illegal combinations.

### Syntax Rules and Structure

**General syntax:**

```
type UnionName =
  | { discriminant: "value1"; /* fields for variant 1 */ }
  | { discriminant: "value2"; /* fields for variant 2 */ }
  | { discriminant: "value3"; /* fields for variant 3 */ };
```

**Component breakdown:**

- `discriminant` — the shared property name (e.g., `kind`, `type`, `status`).
- `"value1"`, `"value2"`, `"value3"` — **literal types** that uniquely identify each member.
- Additional fields — variant-specific properties.

**Syntax rules.**

- The discriminant must be a **literal type** (string, number, `null`, or `undefined`).
- Every member must declare the discriminant property.
- The discriminant values must be **unique** across members.
- Members may have additional fields that differ per variant.
- The discriminant check may be performed with `switch`, `if`, or equality.
- `switch` on the discriminant is the idiomatic narrowing pattern.

**Constraints and limitations.**

- **Methods cannot be discriminants**; only literal-typed properties work.
- **Nested discriminants do not narrow automatically**; the discriminant must be at the same level.
- **Large unions** (50+ members) still incur quadratic checking costs; group them into categories if needed.
- Discriminated unions are **erased** and provide no runtime reflection.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic discriminated union with `switch`

```typescript
// example_du_basic.ts
// Step 1: Define each variant with a common discriminant.
interface Circle {
  kind: "circle";
  radius: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

interface Triangle {
  kind: "triangle";
  base: number;
  height: number;
}

// Step 2: Create the union.
type Shape = Circle | Rectangle | Triangle;

// Step 3: Narrow with switch on the discriminant.
function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2; // narrowed to Circle
    case "rectangle":
      return shape.width * shape.height; // narrowed to Rectangle
    case "triangle":
      return (shape.base * shape.height) / 2; // narrowed to Triangle
  }
}

// Step 4: Use the function.
console.log(calculateArea({ kind: "circle", radius: 1 }).toFixed(2)); // Expected: 3.14
console.log(calculateArea({ kind: "rectangle", width: 2, height: 3 })); // Expected: 6
console.log(calculateArea({ kind: "triangle", base: 4, height: 5 }));   // Expected: 10
```

**Setup.** Compile with `npx tsc example_du_basic.ts --strict`.

**Expected Output:**
```
3.14
6
10
```

**Why this output occurs.** The `kind` property is a literal-typed discriminant. Inside each `case`, TypeScript narrows `shape` to the corresponding interface, making `radius`, `width`/`height`, or `base`/`height` available. Without the discriminant, only properties common to all members would be accessible.

---

#### Example 2: Discriminated union with early returns

```typescript
// example_du_early_return.ts
type APIResponse =
  | { version: 0; msg: string }
  | { version: 1; message: string; status: number }
  | { error: string };

function handleResponse(response: APIResponse): void {
  if ("error" in response) {
    console.error(response.error);
    return;
  }

  if (response.version === 0) {
    console.log(response.msg);
  } else if (response.version === 1) {
    console.log(response.status, response.message);
  }
}

handleResponse({ error: "Not found" }); // Expected: Not found
handleResponse({ version: 0, msg: "OK" }); // Expected: OK
handleResponse({ version: 1, message: "Created", status: 201 }); // Expected: 201 Created
```

**Expected Output:**
```
Not found
OK
201 Created
```

**Why this output occurs.** The `in` check narrows the union by excluding the `error` member. After the early return, TypeScript knows `response` is one of the versioned members. The `version` checks then narrow to the specific member.

---

### Real-World Cases

- **React state management**: `type FetchState<T> = { status: "idle" } | { status: "loading" } | { status: "success"; data: T } | { status: "error"; error: Error }`.
- **Redux actions**: reducers switch on `action.type` to handle each action variant.
- **API responses**: success and error variants with different payloads.
- **Form field states**: `{ status: "valid"; value: string } | { status: "invalid"; errors: string[] }`.

---

## 2. Discriminant Property

### Definitions

**Core Definition.** The **discriminant property** (also called the **tag**) is the shared property in a discriminated union whose **literal type** uniquely identifies each member. It is the property that TypeScript checks to narrow the union.

**Technical Definition.** The discriminant is a property of a **unit type**—a string literal (`"circle"`), numeric literal (`0`), `null`, or `undefined`. The compiler uses **control-flow analysis** on the discriminant to narrow the union. The discriminant must be **present on every member** and have a **unique value** per member. Property names are arbitrary; `kind`, `type`, `status`, `tag`, and `version` are common conventions.

**Beginner-Friendly Explanation.** The discriminant is **the label** on each member of the union. It is a property whose value is a fixed literal, like `"circle"`. When you check that label, TypeScript knows which member you have.

### Purposes

- **To uniquely identify each union member** at the type level.
- **To enable automatic narrowing** when the discriminant is checked.
- **To serve as the switch condition** for exhaustive handling.
- **To document the variants** clearly in the type definition.

### Syntax Rules and Structure

**General syntax:**

```
type Union =
  | { discriminant: "value1"; /* ... */ }
  | { discriminant: "value2"; /* ... */ };
```

**Component breakdown:**

- `discriminant` — the property name.
- `"value1"`, `"value2"` — literal type values.

**Syntax rules.**

- The discriminant must be a **literal type** (`string`, `number`, `null`, or `undefined`).
- Every member must declare the discriminant.
- Discriminant values must be **unique**.
- The discriminant may be a **numeric literal** (e.g., `version: 0`, `version: 1`).
- Methods, computed properties, and non-literal types cannot serve as discriminants.

**Constraints and limitations.**

- **Methods are not discriminants**; only literal-typed properties work.
- **Nested discriminants do not narrow**; the discriminant must be at the top level of each member.
- The discriminant property name should be consistent across members.

### Annotated Complete Step by Step Code Examples

#### Example 1: String literal discriminant

```typescript
// example_discriminant_string.ts
type TimingEvent =
  | { name: "start"; userStarted: boolean }
  | { name: "closed"; duration: number };

function handleEvent(event: TimingEvent) {
  switch (event.name) {
    case "start":
      console.log(event.userStarted); // narrowed to "start" variant
      break;
    case "closed":
      console.log(event.duration); // narrowed to "closed" variant
      break;
  }
}

handleEvent({ name: "start", userStarted: true }); // Expected: true
handleEvent({ name: "closed", duration: 100 });    // Expected: 100
```

**Expected Output:**
```
true
100
```

**Why this output occurs.** The `name` property is a string literal discriminant. The `switch` narrows `event` to the matching member, enabling access to `userStarted` or `duration` respectively.

---

#### Example 2: Numeric literal discriminant

```typescript
// example_discriminant_number.ts
type APIResponses =
  | { version: 0; msg: string }
  | { version: 1; message: string; status: number }
  | { error: string };

function handleResponse(response: APIResponses) {
  if ("error" in response) {
    console.error(response.error);
    return;
  }

  if (response.version === 0) {
    console.log(response.msg);
  } else if (response.version === 1) {
    console.log(response.status, response.message);
  }
}

handleResponse({ version: 0, msg: "OK" }); // Expected: OK
```

**Expected Output:**
```
OK
```

**Why this output occurs.** The `version` property is a numeric literal discriminant (`0` or `1`). The `error` member has no `version` property, so the `in` check excludes it. The `version` checks then narrow to the specific response variant.

---

### Real-World Cases

- **Event names**: `{ name: "start" } | { name: "closed" }`.
- **API versions**: `{ version: 0 } | { version: 1 }`.
- **Action types**: `{ type: "add" } | { type: "remove" }`.

---

## 3. Tagged Unions

### Definitions

**Core Definition.** **Tagged union** is a **synonym** for **discriminated union**. The term emphasizes the **tag** (the discriminant property) that labels each member. Both terms describe the same TypeScript pattern: a union whose members carry a literal-typed tag enabling narrowing.

**Technical Definition.** In TypeScript's documentation, the terms “discriminated union,” “tagged union,” and “algebraic data type” are used interchangeably. The **tag** is the discriminant property. The pattern originates from typed functional languages (e.g., F#, Haskell) where tagged unions are a first-class language feature; in TypeScript, the pattern is emulated through union types and literal discriminants.

**Beginner-Friendly Explanation.** “Tagged union” is just **another name for a discriminated union**. The “tag” is the label (discriminant) on each member. If you hear “tagged union,” think “discriminated union.”

### Purposes

- **To provide a familiar term** for developers coming from languages with native tagged unions.
- **To emphasize the role of the tag** in narrowing.
- **To describe the same pattern** in algebraic-data-type terminology.

### Syntax Rules and Structure

The syntax is **identical** to discriminated unions: a union of object types with a common literal-typed tag property.

**Syntax rules.**

- Same as discriminated unions.
- The tag property name is arbitrary (`kind`, `type`, `tag`, `status`, etc.).

**Constraints and limitations.**

- Same as discriminated unions.
- The term “tagged union” is a synonym, not a distinct TypeScript feature.

### Annotated Complete Step by Step Code Examples

#### Example 1: Tagged union with a `tag` property

```typescript
// example_tagged_union.ts
type Notification =
  | { tag: "email"; address: string }
  | { tag: "sms"; phoneNumber: string }
  | { tag: "push"; deviceId: string };

function send(notification: Notification): void {
  switch (notification.tag) {
    case "email":
      console.log(`Email to ${notification.address}`); // narrowed
      break;
    case "sms":
      console.log(`SMS to ${notification.phoneNumber}`); // narrowed
      break;
    case "push":
      console.log(`Push to ${notification.deviceId}`); // narrowed
      break;
  }
}

send({ tag: "email", address: "ada@example.com" }); // Expected: Email to ada@example.com
```

**Expected Output:**
```
Email to ada@example.com
```

**Why this output occurs.** The `tag` property is the discriminant (tag). The `switch` narrows to the matching variant, enabling access to `address`. The pattern is identical to a discriminated union; the name “tagged union” emphasizes the tag property.

---

### Real-World Cases

- **Notification channels**: `{ tag: "email" } | { tag: "sms" } | { tag: "push" }`.
- **Command types**: `{ tag: "create" } | { tag: "update" } | { tag: "delete" }`.
- **Functional programming**: modeling sum types with tags.

---

## 4. Discriminated Union Patterns

### Definitions

**Core Definition.** **Discriminated union patterns** are recurring idioms for defining, narrowing, and handling discriminated unions in TypeScript, including exhaustive `switch` statements, `never`-based exhaustiveness checks, and state-machine modeling.

**Technical Definition.** The primary patterns are: **exhaustive `switch`** on the discriminant with a `never`-based `default` branch; **early-return narrowing** using `in` or equality checks; **state-machine modeling** with one variant per legal state; and **grouping large unions** into discriminated categories. These patterns leverage control-flow analysis to eliminate runtime errors and catch missing cases at compile time.

**Beginner-Friendly Explanation.** These are **the standard ways people use discriminated unions**. The most important is the exhaustive `switch`: handle every case, and put a `never` check in the `default` branch so TypeScript tells you if you forget one.

### Purposes

- **To handle every variant** without runtime surprises.
- **To make missing cases compile-time errors** when new variants are added.
- **To model state machines** where each state has exactly the right fields.
- **To replace optional-field objects** that allow illegal combinations.

### Syntax Rules and Structure

**Exhaustive `switch` pattern:**

```
function handle(value: Union): ReturnType {
  switch (value.discriminant) {
    case "a": /* narrowed to A */ break;
    case "b": /* narrowed to B */ break;
    default: {
      const unreachable: never = value;
      return unreachable;
    }
  }
}
```

**`assertNever` helper:**

```
function assertNever(value: never): never {
  throw new Error(`Unhandled: ${JSON.stringify(value)}`);
}
```

**Component breakdown:**

- `switch (value.discriminant)` — narrows the union.
- `case "a"`, `case "b"` — handled variants.
- `default` — should receive `never` if all variants are handled.
- `assertNever` — throws if an unhandled variant reaches `default`.

**Syntax rules.**

- The `default` branch should assign the value to `never` or call `assertNever`.
- Adding a new variant without handling it causes a compile-time error in every non-exhaustive `switch`.
- Early returns can narrow the remaining type without a `switch`.
- Group large unions into discriminated categories to keep outer unions small.

**Constraints and limitations.**

- `switch` exhaustiveness only works if the discriminant is a literal type.
- Nested discriminants do not narrow automatically; the discriminant must be at the same level.
- The `never` check must be in a `default` branch (or a final `else`).

### Annotated Complete Step by Step Code Examples

#### Example 1: Exhaustive `switch` with `assertNever`

```typescript
// example_pattern_exhaustive.ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function assertNever(value: never): never {
  throw new Error(`Unhandled: ${JSON.stringify(value)}`);
}

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    default:
      return assertNever(shape); // ✅ compiles: all variants handled
  }
}

console.log(area({ kind: "circle", radius: 1 }).toFixed(2)); // Expected: 3.14
```

**Expected Output:**
```
3.14
```

**Why this output occurs.** After handling both `"circle"` and `"square"`, `shape` in the `default` branch has type `never`. `assertNever` accepts `never`, so the code compiles. If a new `"triangle"` variant were added, `shape` in `default` would be `{ kind: "triangle" }`, which is not assignable to `never`, causing a compile-time error.

---

#### Example 2: State machine with discriminated unions

```typescript
// example_pattern_state_machine.ts
type Job =
  | { kind: "pending" }
  | { kind: "running"; startedAt: Date }
  | { kind: "done"; startedAt: Date; result: Uint8Array }
  | { kind: "failed"; startedAt: Date; error: string };

function describe(job: Job): string {
  switch (job.kind) {
    case "pending":
      return "queued";
    case "running":
      return `running since ${job.startedAt.toISOString()}`;
    case "done":
      return "finished";
    case "failed":
      return job.error;
    default: {
      const unreachable: never = job;
      return unreachable;
    }
  }
}

console.log(describe({ kind: "pending" })); // Expected: queued
```

**Expected Output:**
```
queued
```

**Why this output occurs.** Each state carries exactly the fields relevant to it—`startedAt` only appears in non-pending states, `result` only in `done`, and `error` only in `failed`. Invalid states (e.g., `pending` with a `result`) are unrepresentable. The `never` check ensures exhaustiveness.

---

#### Example 3: Early-return narrowing with `in`

```typescript
// example_pattern_early_return.ts
type APIResponse =
  | { version: 0; msg: string }
  | { version: 1; message: string; status: number }
  | { error: string };

function handle(response: APIResponse): void {
  if ("error" in response) {
    console.error(response.error);
    return; // Excludes the error member from the rest of the function.
  }

  if (response.version === 0) {
    console.log(response.msg);
  } else {
    console.log(response.status, response.message);
  }
}

handle({ error: "Not found" }); // Expected: Not found
```

**Expected Output:**
```
Not found
```

**Why this output occurs.** The `in` check narrows the union, and the early `return` excludes the `error` member. The remaining code only deals with the versioned members, which are then narrowed by `version` checks.

---

### Real-World Cases

- **React `useFetch`**: `type FetchState<T> = { status: "idle" } | { status: "loading" } | { status: "success"; data: T } | { status: "error"; error: Error }` with exhaustive rendering.
- **Redux reducers**: `switch (action.type)` with `assertNever` in the default case.
- **Job queues**: `pending | running | done | failed` with state-specific fields.
- **Form validation**: `{ status: "valid"; value: string } | { status: "invalid"; errors: string[] }`.

---

## 5. Discriminated Union Best Practices

### Definitions

**Core Definition.** **Discriminated union best practices** are guidelines for designing and handling discriminated unions in ways that maximize **type safety, exhaustiveness, and maintainability**, while avoiding the pitfalls of missing cases, optional-field “god objects,” and large flat unions that degrade compiler performance.

**Technical Definition.** Best practices center on **modeling one variant per legal state**, **always using exhaustive `switch` with `never` in the default branch**, **preferring discriminated unions over optional-field objects**, **using literal unions rather than enums** for zero-runtime closed sets, **grouping large unions into discriminated categories** to avoid quadratic checking, and **keeping the discriminant at the top level** of each member.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to get the most out of discriminated unions. The big rules are: make invalid states impossible; always handle every case with a `never` check; do not use optional fields for mutually exclusive states; and group huge unions into smaller categories.

### Purposes

- **To make invalid states unrepresentable**, eliminating a class of runtime bugs.
- **To catch missing cases at compile time** when new variants are added.
- **To avoid quadratic type-checking** on large unions.
- **To keep code readable and maintainable** with clear variant structures.
- **To replace optional-field objects** that force readers to check which fields are set.

### Syntax Rules and Structure

Best practices are conventions that interact with discriminated union syntax. The core rules are:

1. **Model one variant per legal state.** Each member should carry exactly the fields relevant to that state. Avoid optional fields that can coexist illegally.
2. **Always use exhaustive `switch` with `never` in the default branch.** This ensures compile-time errors when new variants are added but not handled.
3. **Prefer discriminated unions over optional-field objects.** Optional fields allow impossible combinations (e.g., `status: "error"` with no `error` field); discriminated unions make each state explicit.
4. **Use literal unions, not `enum`.** String literal unions have zero runtime cost, serialize as themselves, and avoid the pitfalls of numeric enums.
5. **Group large unions into discriminated categories.** Outer unions with 50+ members cause quadratic checking; group them into 3–5 categories, each with its own discriminated union.
6. **Keep the discriminant at the top level.** Nested discriminants do not narrow automatically; the discriminant must be at the same level as the union members.
7. **Use unique discriminant values.** Duplicate discriminant values break narrowing and make the union ambiguous.
8. **Use an `assertNever` helper.** Encapsulate the `never` check in a reusable function for cleaner code.

**Component breakdown (discriminated union design decision tree):**

```
Do you have mutually exclusive states with different fields?
  ├── Yes → Use a discriminated union.
  │        ├── Add a literal-typed discriminant to each variant.
  │        ├── Keep only variant-specific fields in each member.
  │        └── Handle with an exhaustive switch + never default.
  └── No → Use a simple union or optional fields (carefully).
```

**Constraints and limitations.**

- Discriminated unions require literal-typed discriminants; methods and computed properties do not work.
- Nested discriminants do not narrow automatically.
- Large unions still incur performance costs; grouping mitigates but does not eliminate them.
- Exhaustiveness checking requires a `default` branch and a `never`-typed variable or `assertNever`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Modeling invalid states as impossible

```typescript
// example_best_invalid_states.ts
// ❌ Optional fields allow impossible combinations.
// interface Job {
//   status: "pending" | "running" | "done" | "failed";
//   startedAt?: Date; // should be set iff status !== "pending"
//   result?: Uint8Array; // should be set iff status === "done"
//   error?: string; // should be set iff status === "failed"
// }

// ✅ Discriminated union: one variant per legal state.
type Job =
  | { kind: "pending" }
  | { kind: "running"; startedAt: Date }
  | { kind: "done"; startedAt: Date; result: Uint8Array }
  | { kind: "failed"; startedAt: Date; error: string };

function describe(job: Job): string {
  switch (job.kind) {
    case "pending":
      return "queued";
    case "running":
      return `running since ${job.startedAt.toISOString()}`;
    case "done":
      return "finished";
    case "failed":
      return job.error;
    default: {
      const unreachable: never = job;
      return unreachable;
    }
  }
}

console.log(describe({ kind: "pending" })); // Expected: queued
```

**Expected Output:**
```
queued
```

**Why this output occurs.** Each state carries only its relevant fields. The invalid state `{ kind: "pending", result: ... }` is rejected by the compiler. The `never` check ensures exhaustiveness.

---

#### Example 2: Exhaustive `switch` with `assertNever`

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
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    default:
      return assertNever(shape); // ✅ compiles: all variants handled
  }
}

console.log(area({ kind: "circle", radius: 1 }).toFixed(2)); // Expected: 3.14
```

**Expected Output:**
```
3.14
```

**Why this output occurs.** The `assertNever` helper ensures that if a new variant is added and not handled, the `default` branch receives a non-`never` type and the compiler reports an error. This is the recommended exhaustiveness pattern.

---

#### Example 3: Grouping large unions into discriminated categories

```typescript
// example_best_grouping.ts
// ❌ 200+ flat union members cause quadratic checking.
// ✅ Group into discriminated categories.
type UserEvent = { category: "user"; action: "login" | "logout" | "signup" };
type PageEvent = { category: "page"; action: "view" | "scroll" | "leave" };
type FormEvent = { category: "form"; action: "submit" | "validate" | "reset" };

type AppEvent = UserEvent | PageEvent | FormEvent;

function handleEvent(event: AppEvent): void {
  switch (event.category) {
    case "user":
      console.log(`User action: ${event.action}`); // narrowed to UserEvent
      break;
    case "page":
      console.log(`Page action: ${event.action}`); // narrowed to PageEvent
      break;
    case "form":
      console.log(`Form action: ${event.action}`); // narrowed to FormEvent
      break;
  }
}

handleEvent({ category: "user", action: "login" }); // Expected: User action: login
```

**Expected Output:**
```
User action: login
```

**Why this output occurs.** The outer union has only three members, avoiding quadratic checking. Each category narrows to its specific `action` union. This pattern keeps compiler performance reasonable while preserving type safety.

---

### Real-World Cases

- **React data fetching**: `FetchState<T>` with exhaustive rendering in components.
- **Redux reducers**: `switch (action.type)` with `assertNever` in the default case.
- **Job/queue systems**: `pending | running | done | failed` with state-specific fields.
- **API response handling**: success and error variants with exhaustive handling.

---

## References

- TypeScript Playground — Discriminate Types (Official Example) - https://www.typescriptlang.org/play/typescript/meta-types/discriminate-types.ts.html
- TypeScript Handbook — Narrowing: Discriminated Unions - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook — The `never` Type (Exhaustiveness Checking) - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#the-never-type
- TypeScript 2.0 Release Notes — Tagged (Discriminated) Union Types - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html
- OneUptime Blog — How to Handle Discriminated Unions in TypeScript - https://github.com/OneUptime/blog/blob/master/posts/2026-01-24-typescript-discriminated-unions/README.md
- Steve Kinney — Discriminated Unions (React with TypeScript) - https://stevekinney.com/courses/react-typescript/typescript-discriminated-unions
- Total TypeScript — Narrowing a Discriminated Union with a Switch - https://www.totaltypescript.com/workshops/typescript-pro-essentials/unions-and-narrowing/narrowing-a-discriminated-union-with-a-switch
- praxion — TypeScript Representation Patterns: Discriminated Unions with Exhaustiveness - https://github.com/francisco-perez-sorrosal/praxion/blob/main/skills/data-structure-design/references/typescript-patterns.md
- langwatch — Type Safety Rules: Exhaustive Switch Statements - https://github.com/langwatch/langwatch/blob/5698c57f401be8b06379b5cd10efca0c94a554d6/.cursor/rules/type-safety.mdc
- Stack Overflow — Discriminant Must Be a Literal Type (Methods Not Supported) - https://stackoverflow.com/questions/79660841/how-to-use-function-as-a-type-narrowing-hint-in-tagged-union
- Stack Overflow — Nested Discriminants Do Not Narrow - https://stackoverflow.com/questions/77854556/narrowing-a-discriminated-union-based-on-a-nested-discriminated-union
- TypeScript Handbook — Unions and Intersection Types (Discriminating Unions, Legacy) - https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html#discriminating-unions