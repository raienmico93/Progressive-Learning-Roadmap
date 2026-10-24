# Type Narrowing in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Type narrowing** is the process by which TypeScript’s **control-flow analysis** refines the apparent type of a value from a broader declared type to a more specific type within a guarded scope. TypeScript “overlays type analysis on JavaScript’s runtime control flow constructs like `if`/`else`, conditional ternaries, loops, truthiness checks, etc.”, and recognizes certain runtime checks as **type guards** that justify the refinement.

**Technical Definition.** Type narrowing is driven by **Control Flow Analysis (CFA)** : TypeScript “follows possible paths of execution that our programs can take” and, at each point, computes the narrowest type consistent with the guards that dominate that point. Built-in guards include `typeof`, `instanceof`, the `in` operator, truthiness checks, and equality checks; user-defined guards use type predicates. Narrowing is **flow-sensitive** and is **invalidated** by assignments or operations that may affect the value. When all union members are eliminated, the residual type is `never`, enabling **exhaustiveness checking**.

**Beginner-Friendly Explanation.** Type narrowing is **TypeScript following your logic to figure out what a value really is at each point in your code**. If a variable is declared as `string | number`, TypeScript does not know which one it is—until you check. When you write `if (typeof value === "string")`, TypeScript sees the check and, inside that block, treats `value` as a `string`. That lets you safely call string methods without unsafe casts. Narrowing is what makes unions **practical and safe**.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Flow-sensitive** | Narrowing depends on the control-flow path leading to a point, not just the declaration. |
| **Guard-driven** | Narrowing is triggered by recognized runtime checks (`typeof`, `instanceof`, `in`, truthiness, equality). |
| **Complementary narrowing** | The `else` branch (or code after an early return) is narrowed to the remaining members. |
| **Discriminant-based** | A common literal-typed property enables automatic narrowing across union variants. |
| **Reversible** | Assignments and operations that may mutate the value can widen the type back. |
| **`never` for exhaustiveness** | When all members are handled, the residual type is `never`, which can be used to enforce completeness. |
| **Compile-time only** | Narrowing is erased; the emitted JavaScript contains only the runtime checks you wrote. |

---

### Prerequisites

- **Union types** and **literal types**.
- **Control flow**: `if`/`else`, `switch`, ternaries, loops, and early returns.
- **JavaScript operators**: `typeof`, `instanceof`, `in`, `===`, `!==`, `==`, `!=`.
- **Type predicates** (`value is T`) for custom guards.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Type guards** — the runtime checks that drive narrowing.
- **Discriminated unions** — the primary pattern for structured variant narrowing.
- **Exhaustiveness checking** with `never`.
- **Control-flow analysis** in the TypeScript compiler.
- **Type assertions** (`as`) — the unsafe alternative narrowing replaces.

---

## Core Concepts / Features

---

## 1. Type Narrowing

### Definitions

**Core Definition.** **Type narrowing** refines a value’s type from a broader declared type to a more specific type within a guarded scope, based on runtime checks that TypeScript recognizes.

**Technical Definition.** TypeScript’s CFA tracks the **apparent type** of each identifier at each program point. Guards refine the apparent type in the guarded branch; assignments reset it to the declared type (or the assigned type). Narrowing is **syntactic** in that it recognizes specific expression forms (`typeof x === "..."`, `x instanceof C`, `"p" in x`, truthiness, equality), and **semantic** in that it respects JavaScript’s runtime coercion rules.

**Beginner-Friendly Explanation.** Type narrowing is **TypeScript becoming more specific about a value’s type based on your checks**. It is how you go from “this is a `string` or a `number`” to “inside this `if`, it is a `string`.”

### Purposes

- **To make union types usable safely** without type assertions.
- **To access member-specific properties** after verifying the member at runtime.
- **To eliminate impossible cases** and enable exhaustive handling.
- **To replace unsafe casts** with compiler-verified runtime checks.
- **To improve editor intelligence** by showing the narrowed type in each branch.

### Syntax Rules and Structure

Narrowing is not a single syntax; it is triggered by **guard expressions** in control-flow positions:

- `if (typeof x === "string") { ... }`
- `if (x instanceof Date) { ... }`
- `if ("prop" in x) { ... }`
- `if (x) { ... }` (truthiness)
- `if (x === "literal") { ... }` (equality)
- `switch (x.kind) { case "a": ... }` (discriminant)

**Syntax rules.**

- The guard must appear in a control-flow position (`if`, `while`, ternary, `&&`, `||`).
- Narrowing applies **inside** the guarded branch and in the **complementary** branch.
- Early returns and `throw` narrow the remaining code.
- Narrowing is **lost** if the value is reassigned or passed to a function that may mutate it.
- Destructuring can **break** discriminant narrowing; keep the object intact.

**Constraints and limitations.**

- Narrowing does not cross function boundaries unless the function is a recognized type guard.
- Object properties are not narrowed unless copied to a local variable (with caveats).
- `typeof null === "object"` requires explicit null handling.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic union narrowing

```typescript
// example_narrowing_basic.ts
function formatValue(value: string | number): string {
  // Step 1: Here, value is string | number.
  if (typeof value === "string") {
    // Step 2: Narrowed to string.
    return value.toUpperCase();
  }
  // Step 3: Narrowed to number.
  return value.toFixed(2);
}

console.log(formatValue("hello")); // Expected: HELLO
console.log(formatValue(3.14159)); // Expected: 3.14
```

**Setup.** Compile with `npx tsc example_narrowing_basic.ts --strict`.

**Expected Output:**
```
HELLO
3.14
```

**Why this output occurs.** The `typeof` guard narrows `value` to `string` in the `if` branch, enabling `.toUpperCase()`. The complementary branch narrows to `number`, enabling `.toFixed()`.

---

#### Example 2: Early return narrowing

```typescript
// example_narrowing_early_return.ts
function process(value: string | null): string {
  if (value === null) {
    return "default";
  }
  // Step 1: After the early return, value is narrowed to string.
  return value.toUpperCase();
}

console.log(process(null));      // Expected: default
console.log(process("hello"));   // Expected: HELLO
```

**Expected Output:**
```
default
HELLO
```

**Why this output occurs.** The `return` in the `null` branch removes `null` from the type for the rest of the function, leaving `string`.

---

### Real-World Cases

- **API response handling**: narrowing `unknown` responses to success or error variants.
- **Event handling**: narrowing `Event` to `MouseEvent` or `KeyboardEvent`.
- **Configuration parsing**: narrowing `string | number | boolean` environment values.

---

## 2. Control Flow Analysis

### Definitions

**Core Definition.** **Control Flow Analysis (CFA)** is the compiler subsystem that tracks the possible types of each identifier along every path through a program, enabling narrowing to be **flow-sensitive** rather than purely declaration-based.

**Technical Definition.** CFA builds a **control-flow graph** from JavaScript constructs (`if`, `switch`, loops, `return`, `throw`, `&&`, `||`, ternaries) and propagates type refinements through it. At each program point, the apparent type is the **intersection** of the declared type and all guards that dominate that point. Assignments update the apparent type. Since **TypeScript 4.4**, CFA also tracks **aliased conditions**—guards stored in `const` variables, `readonly` properties, or unmodified parameters—and **extracted discriminants**, allowing narrowing through these aliases.

**Beginner-Friendly Explanation.** Control flow analysis is **the engine that follows your code’s branches**. It understands that after an early return, the remaining code has fewer possibilities, and it remembers checks stored in variables (since TS 4.4).

### Purposes

- **To make narrowing flow-sensitive** so each branch reflects the checks that led to it.
- **To preserve narrowing through aliased conditions** and extracted discriminants.
- **To enable exhaustive handling** by tracking remaining union members.
- **To support complex logic** with loops, ternaries, and logical operators.

### Syntax Rules and Structure

CFA processes:

- **Branches**: `if`/`else`, `switch`, ternaries.
- **Jumps**: `return`, `throw`, `break`, `continue`.
- **Logical operators**: `&&`, `||`, `??`.
- **Assignments**: which reset narrowing.
- **Aliased conditions**: `const isString = typeof x === "string"`.
- **Extracted discriminants**: `const { kind } = shape`.

**Syntax rules.**

- CFA is automatic; no syntax is required beyond the guards themselves.
- Aliased conditions narrow only if the alias is a `const`, a `readonly` property, or an unmodified parameter.
- Extracted discriminants narrow the original object only when the discriminant is extracted into a `const`.
- Reassignment invalidates narrowing.

**Constraints and limitations.**

- Narrowing across separate variables is not tracked (TypeScript does not know that two variables are correlated).
- Nested discriminants do not narrow automatically; the discriminant must be at the same level.
- Mutable aliases (`let`) do not preserve narrowing.

### Annotated Complete Step by Step Code Examples

#### Example 1: Aliased condition narrowing (TS 4.4+)

```typescript
// example_cfa_alias.ts
function process(value: unknown): string {
  // Step 1: Store the guard in a const alias.
  const isString = typeof value === "string";

  if (isString) {
    // Step 2: Narrowed to string through the alias.
    return value.toUpperCase();
  }
  return String(value);
}

console.log(process("hello")); // Expected: HELLO
console.log(process(42));      // Expected: 42
```

**Expected Output:**
```
HELLO
42
```

**Why this output occurs.** Since TypeScript 4.4, CFA tracks the aliased condition `isString` and narrows `value` accordingly, even though the guard is not inline.

---

#### Example 2: Extracted discriminant narrowing (TS 4.4+)

```typescript
// example_cfa_discriminant.ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number };

function area(shape: Shape): number {
  // Step 1: Extract the discriminant into a const.
  const { kind } = shape;

  if (kind === "circle") {
    // Step 2: Narrowed to circle via the extracted discriminant.
    return Math.PI * shape.radius ** 2;
  }
  return shape.sideLength ** 2;
}

console.log(area({ kind: "circle", radius: 1 }).toFixed(2)); // Expected: 3.14
```

**Expected Output:**
```
3.14
```

**Why this output occurs.** TypeScript 4.4+ CFA tracks extracted discriminants, so checking `kind` narrows the original `shape` object.

---

### Real-World Cases

- **Reducers**: narrowing action types through aliased discriminants.
- **Form validation**: tracking multiple guard aliases for complex conditions.
- **State machines**: narrowing state through extracted `status` fields.

---

## 3. Discriminated Unions

### Definitions

**Core Definition.** A **discriminated union** is a union of object types that share a common **literal-typed discriminant property** (e.g., `kind`, `type`, `status`). Checking the discriminant narrows the union to the matching member.

**Technical Definition.** Each union member declares the discriminant with a **unique literal type**. CFA narrows to the member whose discriminant matches the checked literal. Discriminated unions enable **exhaustiveness checking** with `never`: after all members are handled, the residual type is `never`.

**Beginner-Friendly Explanation.** A discriminated union is **a union where each variant has a tag**. The tag tells TypeScript which variant you have when you check it. `switch (shape.kind)` is the idiomatic way to handle each variant.

### Purposes

- **To model mutually exclusive variants** with precise, type-safe shapes.
- **To enable automatic narrowing** without custom guards.
- **To enforce exhaustiveness** so no variant is missed.
- **To make invalid states unrepresentable** by giving each variant only its fields.

### Syntax Rules and Structure

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number };
```

**Syntax rules.**

- The discriminant must be a **literal type** (`"circle"`, `42`, `null`, `undefined`).
- Every member must declare the discriminant with a **unique value**.
- `switch` on the discriminant is the idiomatic narrowing pattern.
- `never` in the `default` branch enforces exhaustiveness.
- Since TS 4.4, extracted discriminants (`const { kind } = shape`) narrow the original object.
- Since TS 4.6, destructured discriminated unions can narrow, but **destructuring can still break correlation** in some cases.

**Constraints and limitations.**

- Methods cannot be discriminants; only literal-typed properties work.
- Nested discriminants do not narrow automatically.
- Destructuring the discriminant and other fields can lose correlation; keep the object intact when possible.

### Annotated Complete Step by Step Code Examples

#### Example 1: Discriminated union with exhaustive switch

```typescript
// example_discriminated_union.ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number };

function assertNever(value: never): never {
  throw new Error(`Unhandled: ${JSON.stringify(value)}`);
}

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      return assertNever(shape); // ✅ Exhaustive.
  }
}

console.log(area({ kind: "circle", radius: 1 }).toFixed(2)); // Expected: 3.14
console.log(area({ kind: "square", sideLength: 2 }));         // Expected: 4
```

**Expected Output:**
```
3.14
4
```

**Why this output occurs.** The `kind` discriminant narrows `shape` in each `case`. The `default` branch receives `never` because all members are handled; `assertNever` accepts `never`. Adding a new member without handling it causes a compile-time error.

---

#### Example 2: Extracted discriminant narrowing (TS 4.4+)

```typescript
// example_discriminated_extracted.ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number };

function area(shape: Shape): number {
  const { kind } = shape;
  if (kind === "circle") {
    return Math.PI * shape.radius ** 2;
  }
  return shape.sideLength ** 2;
}

console.log(area({ kind: "circle", radius: 1 }).toFixed(2)); // Expected: 3.14
```

**Expected Output:**
```
3.14
```

**Why this output occurs.** TypeScript 4.4+ CFA tracks the extracted discriminant `kind` and narrows the original `shape` object accordingly.

---

### Real-World Cases

- **React state**: `FetchState<T>` with `status: "idle" | "loading" | "success" | "error"`.
- **Redux actions**: `switch (action.type)` with `assertNever` in the default.
- **Job queues**: `pending | running | done | failed` with state-specific fields.
- **API responses**: success and error variants with different payloads.

---

## 4. Narrowing with `typeof`

### Definitions

**Core Definition.** The **`typeof` guard** narrows **primitive types** by comparing the result of the JavaScript `typeof` operator to a string literal.

**Technical Definition.** `typeof` returns `"string"`, `"number"`, `"boolean"`, `"symbol"`, `"undefined"`, `"object"`, `"function"`, or `"bigint"`. TypeScript recognizes comparisons against these literals as type guards that narrow to the corresponding primitive type. `typeof null` returns `"object"`, so null checks require explicit handling.

**Beginner-Friendly Explanation.** `typeof` is **the guard for primitives**. It tells you whether a value is a string, number, boolean, and so on.

### Purposes

- **To narrow unions of primitive types** (`string | number`, `string | undefined`).
- **To distinguish primitives from objects or functions**.
- **To check for `undefined` or `function`** reliably.
- **To build reusable primitive guards** (`isString`, `isNumber`).

### Syntax Rules and Structure

```typescript
typeof value === "string"
typeof value !== "undefined"
```

**Syntax rules.**

- Works only for **primitive** narrowing.
- Does **not** narrow to specific object shapes.
- `typeof null` returns `"object"`; handle null explicitly.
- Can be extracted into reusable guard functions.

**Constraints and limitations.**

- Cannot distinguish between `Array`, `Date`, or custom classes.
- Cannot narrow interfaces or type aliases of object types.

### Annotated Complete Step by Step Code Examples

#### Example 1: `typeof` guard for primitives

```typescript
// example_typeof_guard.ts
function format(value: string | number | boolean): string {
  if (typeof value === "string") {
    return value.trim();
  }
  if (typeof value === "number") {
    return value.toLocaleString();
  }
  return value ? "yes" : "no";
}

console.log(format("  hello  ")); // Expected: hello
console.log(format(1234.5));      // Expected: 1,234.5
console.log(format(true));        // Expected: yes
```

**Expected Output:**
```
hello
1,234.5
yes
```

**Why this output occurs.** Each `typeof` check narrows `value` to the corresponding primitive, enabling the appropriate methods.

---

### Real-World Cases

- **Environment variables**: `typeof process.env.PORT === "string"`.
- **Optional values**: `typeof value !== "undefined"`.
- **Callback detection**: `typeof callback === "function"`.

---

## 5. Narrowing with `instanceof`

### Definitions

**Core Definition.** The **`instanceof` guard** narrows a value to a **class instance type** by checking whether the value’s prototype chain contains the specified constructor’s prototype.

**Technical Definition.** `x instanceof C` is true if `C.prototype` is in the prototype chain of `x`. TypeScript narrows `x` to the instance type of `C`. `instanceof` works with **class constructors**, not with interfaces or type aliases (which have no runtime representation).

**Beginner-Friendly Explanation.** `instanceof` is **the guard for class instances**. It tells you whether an object was created by a particular class.

### Purposes

- **To narrow `Error` unions** to specific error subclasses.
- **To distinguish class instances** in a hierarchy.
- **To narrow DOM events** to `MouseEvent`, `KeyboardEvent`, etc.
- **To build reusable class guards** (`isDate`, `isNetworkError`).

### Syntax Rules and Structure

```typescript
value instanceof ClassName
```

**Syntax rules.**

- Works only with **classes** (values with a prototype).
- Narrows to the **instance type** of the class.
- Respects inheritance: a subclass instance is `instanceof` its base class.

**Constraints and limitations.**

- Cannot be used with interfaces or type aliases.
- Fails across realms (e.g., iframes) where prototypes differ.
- Does not work with plain object literals.

### Annotated Complete Step by Step Code Examples

#### Example 1: `instanceof` guard for errors

```typescript
// example_instanceof_guard.ts
class ApiError extends Error {
  constructor(message: string, public statusCode: number) {
    super(message);
  }
}

function handleError(error: Error): void {
  if (error instanceof ApiError) {
    console.log(`HTTP ${error.statusCode}: ${error.message}`);
    return;
  }
  console.log(`Unknown: ${error.message}`);
}

handleError(new ApiError("Not Found", 404));
// Expected: HTTP 404: Not Found
```

**Expected Output:**
```
HTTP 404: Not Found
```

**Why this output occurs.** The `instanceof ApiError` check narrows `error` to `ApiError`, enabling access to `statusCode`.

---

### Real-World Cases

- **Error handling**: `if (error instanceof ValidationError)`.
- **Event handling**: `if (event instanceof MouseEvent)`.
- **Built-in types**: `if (value instanceof Date)`.

---

## 6. Narrowing with `in`

### Definitions

**Core Definition.** The **`in` guard** narrows a value by checking whether a specified **property exists** on it (or its prototype chain). It is useful for distinguishing object types that differ by the presence of a property.

**Technical Definition.** `"prop" in value` returns `true` if `prop` exists. TypeScript narrows `value` to the union members that have the property (required or optional) in the `true` branch, and to those that lack it or have it as optional in the `false` branch.

**Beginner-Friendly Explanation.** `in` is **the guard for property presence**. It tells you whether an object has a particular property.

### Purposes

- **To distinguish object types by their properties**.
- **To narrow discriminated unions** when the discriminant is a property check.
- **To handle API responses** with success and error variants.
- **To check for optional properties** before accessing them.

### Syntax Rules and Structure

```typescript
"propertyName" in value
```

**Syntax rules.**

- The property name must be a **string literal** for narrowing.
- Narrows to union members that declare the property.
- Optional properties appear in **both** branches.

**Constraints and limitations.**

- Does not narrow to types where the property is only conditionally present.
- Property existence does not guarantee the property’s type.
- Dynamic property names may not narrow.

### Annotated Complete Step by Step Code Examples

#### Example 1: `in` guard for object variants

```typescript
// example_in_guard.ts
interface Car { drive(): void; wheels: number }
interface Boat { sail(): void; rudder: boolean }

function operate(vehicle: Car | Boat): void {
  if ("wheels" in vehicle) {
    vehicle.drive();
    console.log(`Car with ${vehicle.wheels} wheels`);
  } else {
    vehicle.sail();
    console.log(`Boat with rudder: ${vehicle.rudder}`);
  }
}

operate({ drive: () => console.log("Driving"), wheels: 4 });
// Expected:
// Driving
// Car with 4 wheels
```

**Expected Output:**
```
Driving
Car with 4 wheels
```

**Why this output occurs.** The `"wheels" in vehicle` check narrows `vehicle` to `Car` in the `if` branch and `Boat` in the `else` branch.

---

### Real-World Cases

- **API responses**: `if ("error" in response) { ... } else { ... }`.
- **Plugin systems**: checking which optional capabilities an object provides.
- **Configuration objects**: detecting which configuration variant is present.

---

## 7. Narrowing with Truthiness

### Definitions

**Core Definition.** **Truthiness narrowing** uses JavaScript’s coercion of values to booleans to narrow types: falsy values (`0`, `NaN`, `""`, `0n`, `null`, `undefined`, `false`) are removed in the `true` branch, and truthy values are removed in the `false` branch.

**Technical Definition.** When a value is used in a boolean context (`if`, `&&`, `||`, `!`), TypeScript narrows based on which values are truthy or falsy. In the `true` branch, falsy constituents are removed; in the `false` branch, truthy constituents are removed. Boolean negations (`!`) filter out the negated branch.

**Beginner-Friendly Explanation.** Truthiness narrowing is **using `if (value)` to check that a value exists**. It is useful for guarding against `null` and `undefined`, but it can be error-prone with primitives because `0` and `""` are also falsy.

### Purposes

- **To guard against `null` and `undefined`** concisely.
- **To narrow optional values** before use.
- **To filter arrays** with `.filter(Boolean)` (with caveats).
- **To remove falsy members from unions**.

### Syntax Rules and Structure

```typescript
if (value) { ... }
if (!value) { ... }
```

**Syntax rules.**

- Falsy values: `0`, `NaN`, `""`, `0n`, `null`, `undefined`, `false`.
- In the `true` branch, falsy types are removed.
- In the `false` branch, truthy types are removed.
- `!` reverses the branch.

**Constraints and limitations.**

- Truthiness checks on primitives can be **error-prone**: `0` and `""` are falsy, so `if (value)` may skip legitimate values.
- Prefer explicit `!= null` checks when only nullish values should be excluded.

### Annotated Complete Step by Step Code Examples

#### Example 1: Truthiness guard for nullish values

```typescript
// example_truthiness.ts
function greet(user: { name: string } | null | undefined): string {
  if (!user) {
    return "No user";
  }
  return `Hello, ${user.name}`;
}

console.log(greet(null));                   // Expected: No user
console.log(greet({ name: "Ada" }));        // Expected: Hello, Ada
```

**Expected Output:**
```
No user
Hello, Ada
```

**Why this output occurs.** The `!user` check removes `null` and `undefined` in the `false` branch, narrowing `user` to the object type.

---

#### Example 2: The empty-string pitfall

```typescript
// example_truthiness_pitfall.ts
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    // ✅ Narrowed to string[].
    for (const s of strs) console.log(s);
  } else if (typeof strs === "string") {
    // ⚠️ Empty string was excluded by the truthiness check!
    console.log(strs);
  }
}

printAll([ "a", "b" ]); // Expected: a, b
printAll("");           // Expected: (nothing — empty string is falsy)
```

**Expected Output:**
```
a
b
```

**Why this output occurs.** `if (strs && ...)` excludes `""` because it is falsy. An explicit `strs !== null` check would preserve the empty string.

---

### Real-World Cases

- **Optional props**: `if (user) { ... }`.
- **Array filtering**: `values.filter(Boolean)` (careful with `0` and `""`).
- **Default parameters**: truthiness fallbacks (prefer `??` for nullish-only fallback).

---

## 8. Narrowing with Equality

### Definitions

**Core Definition.** **Equality narrowing** uses `===`, `!==`, `==`, or `!=` comparisons to narrow types. Comparing a value to a literal narrows it to that literal; comparing two variables narrows both to their common type.

**Technical Definition.** When two values are compared with `===` or `!==`, TypeScript narrows both operands to their **common type** in the true branch. Literal comparisons narrow to the literal type. Loose equality (`== null`) narrows out both `null` and `undefined`. `switch` statements on literal values also narrow.

**Beginner-Friendly Explanation.** Equality narrowing is **using `===` to check a value against a literal or another variable**. `if (x === "hello")` narrows `x` to `"hello"`. `x == null` narrows out both `null` and `undefined`.

### Purposes

- **To narrow to literal types** with `===`.
- **To narrow out `null` and `undefined`** with `!= null`.
- **To narrow two variables** to their common type with `x === y`.
- **To handle `switch` cases** on discriminated unions.

### Syntax Rules and Structure

```typescript
x === "literal"
x !== null
x == null   // removes null and undefined
x === y
```

**Syntax rules.**

- `===` and `!==` narrow to the compared literal or common type.
- `== null` and `!= null` narrow out both `null` and `undefined`.
- `switch` on literals narrows each `case`.
- Loose equality with other values is narrowed correctly but should be used deliberately.

**Constraints and limitations.**

- Comparing unrelated types may produce `never`.
- `==` with non-null values has complex coercion rules; prefer `===` except for `null` checks.

### Annotated Complete Step by Step Code Examples

#### Example 1: Equality narrowing with literals

```typescript
// example_equality_literal.ts
type Status = "idle" | "loading" | "success" | "error";

function handle(status: Status): string {
  if (status === "success") {
    return "Done";
  }
  if (status === "error") {
    return "Failed";
  }
  return status; // narrowed to "idle" | "loading"
}

console.log(handle("success")); // Expected: Done
```

**Expected Output:**
```
Done
```

**Why this output occurs.** The `=== "success"` check narrows `status` to the literal `"success"` in the branch.

---

#### Example 2: `== null` narrowing

```typescript
// example_equality_null.ts
function process(value: string | null | undefined): string {
  if (value == null) {
    return "missing";
  }
  return value.toUpperCase(); // narrowed to string
}

console.log(process(null));      // Expected: missing
console.log(process(undefined)); // Expected: missing
console.log(process("hello"));   // Expected: HELLO
```

**Expected Output:**
```
missing
missing
HELLO
```

**Why this output occurs.** `value == null` removes both `null` and `undefined`, leaving `string` after the early return.

---

### Real-World Cases

- **Discriminated unions**: `switch (action.type)`.
- **Nullish checks**: `if (value != null) { ... }`.
- **Literal options**: `if (mode === "dark") { ... }`.

---

## 9. Narrowing Best Practices

### Definitions

**Core Definition.** **Narrowing best practices** are guidelines for using type guards and control flow to achieve **safe, predictable narrowing**, avoiding the pitfalls of lost narrowing, unsafe assertions, and error-prone truthiness checks.

**Technical Definition.** Best practices center on **preferring narrowing over assertions**, **keeping discriminated objects intact**, **using explicit nullish checks**, **leveraging aliased conditions and extracted discriminants (TS 4.4+)** , **using `never` for exhaustiveness**, and **understanding the limits of CFA** (separate variables, nested discriminants).

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep narrowing reliable. The big rules are: use guards instead of `as`; do not destructure a discriminated union before narrowing; use `!= null` instead of truthiness when you only mean nullish; and use `never` to catch missing cases.

### Purposes

- **To avoid unsafe type assertions** and their hidden bugs.
- **To preserve narrowing** across refactoring and destructuring.
- **To ensure nullish handling is precise**.
- **To enforce exhaustiveness** with `never`.
- **To leverage modern CFA features** for cleaner code.

### Syntax Rules and Structure

Best practices are conventions that interact with narrowing syntax. The core rules are:

1. **Prefer narrowing over assertions.** `as` silences the compiler without runtime verification; guards verify at runtime.
2. **Do not destructure discriminants before narrowing.** Destructuring can break the correlation between the discriminant and the rest of the object; access `response.ok` directly.
3. **Use `!= null` for nullish checks.** Truthiness also excludes `0`, `""`, and `false`; `!= null` removes only `null` and `undefined`.
4. **Use aliased conditions and extracted discriminants (TS 4.4+).** `const isString = typeof x === "string"` and `const { kind } = shape` narrow correctly.
5. **Use `never` for exhaustiveness.** Assign the residual value to `never` or call `assertNever` in the `default` branch.
6. **Keep discriminants at the top level.** Nested discriminants do not narrow automatically.
7. **Use `unknown` at boundaries and narrow with guards.** Avoid `any`; validate external data with custom guards.

**Component breakdown (narrowing decision tree):**

```
Do you need to narrow a union?
  ├── Primitive? → typeof.
  ├── Class instance? → instanceof.
  ├── Property presence? → in.
  ├── Discriminated union? → switch on discriminant.
  ├── Nullish only? → != null.
  └── Complex shape? → custom type guard (value is T).
```

**Constraints and limitations.**

- CFA does not track correlations between separate variables.
- Destructuring can lose discriminant correlation in some cases.
- Truthiness narrowing is error-prone with primitives.
- Nested discriminants require custom guards or restructuring.

### Annotated Complete Step by Step Code Examples

#### Example 1: Preserve discriminant narrowing

```typescript
// example_best_preserve.ts
type Response =
  | { ok: true; data: string }
  | { ok: false; error: string };

// ❌ Destructuring can lose narrowing.
function handleBad(response: Response) {
  const { ok } = response;
  if (ok) {
    // response.data may not be narrowed here.
  }
}

// ✅ Keep the object intact.
function handleGood(response: Response) {
  if (response.ok) {
    console.log(response.data); // narrowed.
  } else {
    console.log(response.error); // narrowed.
  }
}

handleGood({ ok: true, data: "Hello" }); // Expected: Hello
```

**Expected Output:**
```
Hello
```

**Why this output occurs.** Accessing `response.ok` directly preserves the discriminant link; destructuring `ok` into a separate variable can break the correlation.

---

#### Example 2: Use `!= null` instead of truthiness

```typescript
// example_best_nullish.ts
function process(value: string | null | undefined): string {
  if (value != null) {
    return value.toUpperCase();
  }
  return "missing";
}

console.log(process(""));    // Expected: "" (empty string preserved)
console.log(process(null));  // Expected: missing
```

**Expected Output:**
```
(missing line only, because "" prints empty)
missing
```

**Why this output occurs.** `!= null` removes only `null` and `undefined`, preserving the empty string. A truthiness check would have excluded `""`.

---

#### Example 3: Exhaustiveness with `never`

```typescript
// example_best_never.ts
type Status = "idle" | "loading" | "success" | "error";

function assertNever(value: never): never {
  throw new Error(`Unhandled: ${value}`);
}

function handle(status: Status): string {
  switch (status) {
    case "idle": return "Ready";
    case "loading": return "Loading...";
    case "success": return "Done";
    case "error": return "Failed";
    default: return assertNever(status);
  }
}

console.log(handle("idle")); // Expected: Ready
```

**Expected Output:**
```
Ready
```

**Why this output occurs.** After all four cases, `status` in `default` has type `never`. `assertNever` accepts `never`, ensuring exhaustiveness; adding a new status without handling it causes a compile-time error.

---

### Real-World Cases

- **API clients**: narrowing `unknown` responses with custom guards instead of `as`.
- **React components**: accessing discriminated union props without destructuring before narrowing.
- **Reducers**: exhaustive `switch` with `assertNever`.
- **Form validation**: explicit nullish checks before accessing field values.

---

## References

- TypeScript Handbook — Narrowing - https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- TypeScript 4.4 Release Notes — Control Flow Analysis of Aliased Conditions and Discriminants - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html
- TypeScript 5.5 Release Notes — Inferred Type Predicates - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-5.html
- TypeScript 4.6 Release Notes — Control Flow Analysis for Destructured Discriminated Unions - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-6.html
- TypeScript Playground — Type Guards - https://www.typescriptlang.org/play/typescript/language-extensions/type-guards.ts.html
- TypeScript Playground — Type Widening and Narrowing - https://www.typescriptlang.org/play/typescript/meta-types/type-widening-and-narrowing.ts.html
- TypeScript Playground — Discriminate Types - https://www.typescriptlang.org/play/typescript/meta-types/discriminate-types.ts.html
- Steve Kinney — Type Narrowing and Control Flow - https://stevekinney.com/courses/react-typescript/typescript-type-narrowing-control-flow
- OneUptime Blog — How to Handle Type Narrowing in TypeScript - https://github.com/oneuptime/blog/tree/master/posts/2026-01-24-typescript-type-narrowing
- Cory House — In modern TypeScript, there are at least a dozen ways to narrow types - https://coryhouse.dev/2024/12/31/tweet-1874227746135572941/
- Educative — Discriminated Unions and Exhaustive Checks - https://www.educative.io/courses/lta/learn-typescript/discriminated-unions-and-exhaustive-checks
- pproenca/dot-skills — Eliminate `as` Casts with Proper Narrowing Chains - https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/typescript-refactor/references/narrow-eliminate-as-casts.md
- Total TypeScript — Narrowing a Discriminated Union with a Switch - https://www.totaltypescript.com/workshops/typescript-pro-essentials/unions-and-narrowing/narrowing-a-discriminated-union-with-a-switch