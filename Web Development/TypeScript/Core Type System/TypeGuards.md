# Type Guards in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **type guard** is a runtime expression that performs a check which TypeScript’s **control-flow analysis** recognizes as evidence about a value’s type, allowing the compiler to **narrow** that value from a broader type to a more specific type within the guarded scope. Type guards are the primary mechanism by which TypeScript reconciles its static type system with JavaScript’s dynamic runtime behavior.

**Technical Definition.** A type guard is “some expression that performs a runtime check that guarantees the type in some scope”. TypeScript recognizes several **built-in guard forms** — `typeof` checks, `instanceof` checks, `in` checks, truthiness checks, and equality checks — and also supports **user-defined guards** via functions whose return type is a **type predicate** of the form `parameterName is Type`. When a guard evaluates to `true`, the compiler narrows the union type of the guarded value to the subset compatible with the guard; when it evaluates to `false` (in an `else` branch or after an early return), the complementary narrowing applies. Type guards are **erased** at compile time — they are ordinary runtime expressions with no TypeScript-specific runtime machinery.

**Beginner-Friendly Explanation.** A type guard is **a runtime check that teaches TypeScript what type a value really is**. TypeScript cannot see inside your running program, so if a value is declared as `string | number`, it does not know which one it is. When you write `if (typeof value === "string")`, you are both checking at runtime **and** telling TypeScript: “inside this block, this value is a string.” TypeScript trusts that check and lets you use string-specific methods. Type guards are how you **safely work with union types** without resorting to unsafe type assertions.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Runtime + compile-time** | A type guard is a real runtime check that also informs the compiler’s static analysis. |
| **Control-flow sensitive** | Narrowing applies within the guarded branch and persists until the flow changes. |
| **Built-in forms** | `typeof`, `instanceof`, `in`, truthiness, and equality checks are recognized natively. |
| **User-defined guards** | Functions with a `value is Type` return type create reusable custom guards. |
| **Inference (TS 5.5+)** | Type predicates are automatically inferred for simple single-return guard functions. |
| **No runtime cost beyond the check** | Guards compile to ordinary JavaScript checks; no additional runtime machinery. |
| **Narrowing scope** | Narrowing affects variables and parameters, not object properties (unless copied to a local). |

---

### Prerequisites

- **TypeScript basics**: union types, type aliases, and interfaces.
- **Control flow**: `if`/`else`, `switch`, and early returns.
- **Primitive types** and **literal types**.
- **Classes and `instanceof`** for class-based narrowing.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Union type narrowing** and **discriminated unions**.
- **Type assertions** (`as`) — the unsafe alternative that type guards replace.
- **Assertion functions** (`asserts value is Type`) — the throwing counterpart to type predicates.
- **Runtime validation** for API responses and external data.
- **Control-flow analysis** in TypeScript’s compiler.

---

## Core Concepts / Features

---

## 1. Type Guards

### Definitions

**Core Definition.** A **type guard** is a runtime expression that TypeScript recognizes as narrowing a value’s type within a guarded scope.

**Technical Definition.** TypeScript’s compiler performs **control-flow analysis** over JavaScript control-flow constructs (`if`/`else`, ternaries, loops, truthiness checks) and identifies certain expressions as type guards. When a guard is true, the guarded value is narrowed to the intersection of its declared type and the type implied by the guard. Built-in guards include `typeof`, `instanceof`, `in`, equality checks, and truthiness checks. User-defined guards use type predicates. Narrowing is **flow-sensitive** and may be invalidated by assignments or function calls that could mutate the value.

**Beginner-Friendly Explanation.** A type guard is **any check that tells TypeScript “this value is actually this narrower type.”** You write a normal JavaScript check, and TypeScript recognizes it as a guard and adjusts its type knowledge inside the guarded block.

### Purposes

- **To safely access member-specific properties** on union values.
- **To replace unsafe type assertions** (`as`) with runtime-verified checks.
- **To enable exhaustiveness** in discriminated union handling.
- **To validate external data** before treating it as a trusted type.
- **To make narrowing logic reusable** across multiple call sites.

### Syntax Rules and Structure

Type guards have **no single syntax**; they are expressions recognized by the compiler. The recognized forms are:

1. **`typeof` guards**: `typeof x === "string"`.
2. **`instanceof` guards**: `x instanceof Date`.
3. **`in` guards**: `"prop" in x`.
4. **Truthiness guards**: `if (x) { ... }`.
5. **Equality guards**: `x === "literal"`, `x !== null`.
6. **User-defined guards**: `function isX(value: unknown): value is X`.

**Syntax rules.**

- The guard expression must appear in a **control-flow position** (`if`, `while`, ternary, logical operators) to trigger narrowing.
- Narrowing applies **inside** the guarded branch and, for `if`/`else`, in the **complementary** branch.
- Narrowing is lost if the value is reassigned or passed to a function that could mutate it.
- Destructuring can **lose narrowing** for discriminated unions; keep the object intact.

**Constraints and limitations.**

- Type guards narrow **variables and parameters**, not object properties, unless the property is copied to a local variable.
- Narrowing does not cross function boundaries unless the function is a recognized type guard.
- `typeof null` returns `"object"`, so null checks require explicit handling.

### Annotated Complete Step by Step Code Examples

#### Example 1: `typeof` guard on a union

```typescript
// example_guard_basic.ts
function format(value: string | number): string {
  if (typeof value === "string") {
    // Step 1: Inside this branch, value is narrowed to string.
    return value.toUpperCase();
  }
  // Step 2: Outside the branch, value is narrowed to number.
  return value.toFixed(2);
}

console.log(format("hello")); // Expected: HELLO
console.log(format(3.14159)); // Expected: 3.14
```

**Setup.** Compile with `npx tsc example_guard_basic.ts --strict`.

**Expected Output:**
```
HELLO
3.14
```

**Why this output occurs.** The `typeof value === "string"` check narrows `value` to `string` in the `if` branch, enabling `.toUpperCase()`. The complementary narrowing leaves `value` as `number` after the branch, enabling `.toFixed()`.

---

### Real-World Cases

- **API response handling**: narrowing `unknown` responses with custom guards before accessing fields.
- **Event handling**: narrowing `Event` to `MouseEvent` or `KeyboardEvent` with `instanceof`.
- **Form validation**: narrowing field values to strings, numbers, or arrays with `typeof` and `Array.isArray`.

---

## 2. `typeof`

### Definitions

**Core Definition.** The **`typeof` guard** uses the JavaScript `typeof` operator to narrow **primitive types** based on the string returned by `typeof`.

**Technical Definition.** `typeof` returns one of `"string"`, `"number"`, `"boolean"`, `"symbol"`, `"undefined"`, `"object"`, `"function"`, or `"bigint"`. TypeScript recognizes comparisons of `typeof x` against these string literals as type guards. `typeof` guards work only for **primitive types**; they cannot distinguish between different object shapes. `typeof null` returns `"object"`, so null checks require explicit `x !== null` or `x != null`.

**Beginner-Friendly Explanation.** `typeof` is **the guard for primitives**. It tells you whether a value is a string, number, boolean, and so on. It is simple and reliable for primitives, but it cannot tell you which class an object belongs to.

### Purposes

- **To narrow unions of primitive types** (`string | number`, `string | undefined`).
- **To distinguish primitives from objects or functions**.
- **To check for `undefined` or `function`** reliably.
- **To build reusable primitive guards** (`isString`, `isNumber`).

### Syntax Rules and Structure

**General syntax:**

```
typeof value === "string"
typeof value !== "undefined"
```

**Component breakdown:**

- `typeof value` — the runtime type string.
- `===` or `!==` — comparison with a string literal.
- Supported literals: `"string"`, `"number"`, `"boolean"`, `"symbol"`, `"undefined"`, `"object"`, `"function"`, `"bigint"`.

**Syntax rules.**

- Works only for **primitive** narrowing.
- Does **not** narrow to specific object shapes.
- `typeof null` returns `"object"`, requiring an explicit null check.
- `typeof` guards can be extracted into reusable functions (`isString`).

**Constraints and limitations.**

- Cannot distinguish between `Array`, `Date`, or custom classes.
- Cannot narrow interfaces or type aliases of object types.

### Annotated Complete Step by Step Code Examples

#### Example 1: `typeof` guard for primitives

```typescript
// example_typeof.ts
function processValue(value: string | number | boolean) {
  if (typeof value === "string") {
    return value.toUpperCase(); // narrowed to string
  }
  if (typeof value === "number") {
    return value.toFixed(2); // narrowed to number
  }
  return value ? "yes" : "no"; // narrowed to boolean
}

console.log(processValue("hello")); // Expected: HELLO
console.log(processValue(42)); // Expected: 42.00
console.log(processValue(true)); // Expected: yes
```

**Expected Output:**
```
HELLO
42.00
yes
```

**Why this output occurs.** Each `typeof` check narrows `value` to the corresponding primitive. The final branch handles the remaining `boolean` case.

---

### Real-World Cases

- **Environment variables**: `if (typeof process.env.PORT === "string")`.
- **Optional values**: `if (typeof value !== "undefined")`.
- **Callback detection**: `if (typeof callback === "function")`.

---

## 3. `instanceof`

### Definitions

**Core Definition.** The **`instanceof` guard** narrows a value to a **class instance type** by checking whether the value’s prototype chain includes the specified constructor.

**Technical Definition.** `x instanceof C` is true if `C.prototype` appears in the prototype chain of `x`. TypeScript recognizes this check as a type guard that narrows `x` to the type of instances constructed by `C`. `instanceof` works with **class constructors**, not with plain object types, interfaces, or type aliases, because those have no runtime representation.

**Beginner-Friendly Explanation.** `instanceof` is **the guard for class instances**. It tells you whether an object was created by a particular class (or a subclass). It is useful for narrowing `Error` types, DOM event types, and custom class hierarchies.

### Purposes

- **To narrow `Error` unions** to specific error subclasses.
- **To distinguish between class instances** in a hierarchy.
- **To narrow DOM events** to `MouseEvent`, `KeyboardEvent`, etc.
- **To build reusable class guards** (`isDate`, `isNetworkError`).

### Syntax Rules and Structure

**General syntax:**

```
value instanceof ClassName
```

**Component breakdown:**

- `value` — the value to check.
- `instanceof` — the operator.
- `ClassName` — a class constructor (not an interface or type alias).

**Syntax rules.**

- Works only with **classes** (values with a prototype).
- Narrows to the **instance type** of the class.
- Respects inheritance: a subclass instance is `instanceof` its base class.

**Constraints and limitations.**

- Cannot be used with interfaces or type aliases (they have no runtime value).
- Fails across realms (e.g., iframes) where prototypes differ.
- Does not work with plain object literals.

### Annotated Complete Step by Step Code Examples

#### Example 1: `instanceof` guard for errors

```typescript
// example_instanceof.ts
class NetworkError extends Error {
  constructor(message: string, public statusCode: number) {
    super(message);
  }
}

function handleError(error: Error) {
  if (error instanceof NetworkError) {
    console.error(`HTTP ${error.statusCode}: ${error.message}`); // narrowed to NetworkError
    return;
  }
  console.error(error.message); // narrowed to Error
}

handleError(new NetworkError("Not Found", 404));
// Expected: HTTP 404: Not Found
```

**Expected Output:**
```
HTTP 404: Not Found
```

**Why this output occurs.** The `instanceof NetworkError` check narrows `error` to `NetworkError` in the `if` branch, enabling access to `statusCode`. Outside the branch, `error` remains `Error`.

---

### Real-World Cases

- **Error handling**: `if (error instanceof ValidationError)`.
- **Event handling**: `if (event instanceof MouseEvent)`.
- **Built-in types**: `if (value instanceof Date)`.

---

## 4. `in`

### Definitions

**Core Definition.** The **`in` guard** narrows a value by checking whether a specified **property exists** on it. It is useful for distinguishing object types that differ by the presence of a property.

**Technical Definition.** `"prop" in value` returns `true` if `prop` exists on `value` or its prototype chain. TypeScript recognizes this as a type guard that narrows `value` to the union members that declare `prop`. `in` works with **object types** and is especially useful for discriminated unions where the discriminant is not a simple literal check.

**Beginner-Friendly Explanation.** `in` is **the guard for property presence**. It tells you whether an object has a particular property. It is useful when two object types differ by which properties they have.

### Purposes

- **To distinguish object types by their properties**.
- **To narrow discriminated unions** when the discriminant is a property check.
- **To handle API responses** with success and error variants.
- **To check for optional properties** before accessing them.

### Syntax Rules and Structure

**General syntax:**

```
"propertyName" in value
```

**Component breakdown:**

- `"propertyName"` — the property name as a string literal.
- `in` — the operator.
- `value` — the object to check.

**Syntax rules.**

- The property name must be a **string literal** for TypeScript to narrow.
- Narrows to union members that declare the property.
- Works with **optional properties** as well as required ones.

**Constraints and limitations.**

- Does not narrow to types where the property is only conditionally present.
- Property existence does not guarantee the property’s type; further checks may be needed.
- Dynamic property names may not narrow.

### Annotated Complete Step by Step Code Examples

#### Example 1: `in` guard for discriminated object variants

```typescript
// example_in.ts
type Dog = { bark: () => void };
type Cat = { meow: () => void };

function makeSound(animal: Dog | Cat) {
  if ("bark" in animal) {
    animal.bark(); // narrowed to Dog
  } else {
    animal.meow(); // narrowed to Cat
  }
}

makeSound({ bark: () => console.log("Woof") }); // Expected: Woof
```

**Expected Output:**
```
Woof
```

**Why this output occurs.** The `"bark" in animal` check narrows `animal` to `Dog` in the `if` branch. The `else` branch narrows to `Cat` because `Dog` has been excluded.

---

### Real-World Cases

- **API responses**: `if ("error" in response) { ... } else { ... }`.
- **Plugin systems**: checking which optional capabilities an object provides.
- **Configuration objects**: detecting which configuration variant is present.

---

## 5. Custom Type Guards

### Definitions

**Core Definition.** A **custom type guard** is a user-defined function that returns a **type predicate** (`value is Type`), combining a runtime check with compile-time narrowing. It encapsulates narrowing logic that can be reused across multiple call sites.

**Technical Definition.** A custom type guard is declared as `function isX(value: unknown): value is X { ... }`. The return type uses the **type predicate syntax** `parameterName is Type`. When the function returns `true`, TypeScript narrows the argument to `Type` within the guarded scope. Since **TypeScript 5.5**, the compiler can **infer** type predicates for simple guard functions whose body is a single boolean return, eliminating the need for explicit `is` annotations in many cases. Explicit `is` annotations remain useful for guards with multiple returns or for documenting public API contracts.

**Beginner-Friendly Explanation.** A custom type guard is **a reusable check function that teaches TypeScript about types**. Instead of writing the same complex runtime check repeatedly, you put it in a function `isUser(value)` and TypeScript will narrow the value whenever the function returns true.

### Purposes

- **To reuse complex narrowing logic** across multiple call sites.
- **To replace unsafe type assertions** with runtime-verified checks.
- **To validate external data** before treating it as a trusted type.
- **To compose guards** for nested or compound conditions.
- **To document narrowing contracts** for public APIs via explicit `is` annotations.

### Syntax Rules and Structure

**General syntax:**

```
function isX(value: unknown): value is X {
  return /* boolean runtime check */;
}
```

**Component breakdown:**

- `function isX` — the guard function.
- `value: unknown` — the parameter to check (often `unknown`).
- `: value is X` — the **type predicate** return type.
- `return` — a boolean expression that performs the runtime check.

**Syntax rules.**

- The return type must be a **type predicate** (`value is X`).
- The predicate parameter must be a **parameter of the function**, not a property.
- The function must return a **boolean**.
- Since TS 5.5, a single boolean `return` body **infers** the predicate automatically.
- Explicit `is` annotations are recommended for **exported guards** and guards with **multiple returns**.

**Constraints and limitations.**

- A guard that **lies** (returns true when the value does not match) is worse than a type assertion because it hides bugs behind a safe-looking name.
- Type predicates narrow **parameters**, not properties like `this.node`.
- Inferred predicates only fire for **single-return** functions; multi-return guards need explicit annotations.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit custom type guard

```typescript
// example_custom_guard.ts
interface User {
  id: string;
  name: string;
}

function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value &&
    typeof (value as User).id === "string" &&
    typeof (value as User).name === "string"
  );
}

function processValue(value: unknown) {
  if (isUser(value)) {
    console.log(value.name); // narrowed to User
  }
}

processValue({ id: "u1", name: "Ada" }); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The `isUser` guard performs a runtime shape check and returns a type predicate. When it returns `true`, TypeScript narrows `value` to `User`, enabling access to `name`.

---

#### Example 2: Inferred type predicate (TS 5.5+)

```typescript
// example_inferred_guard.ts
// With TS 5.5+, the predicate is inferred from the single return.
function isString(value: unknown) {
  return typeof value === "string";
}

function processValue(value: string | number) {
  if (isString(value)) {
    console.log(value.toUpperCase()); // narrowed to string
  }
}

processValue("hello"); // Expected: HELLO
```

**Expected Output:**
```
HELLO
```

**Why this output occurs.** The single boolean return body `typeof value === "string"` is analyzed by control-flow analysis, and the type predicate `value is string` is inferred automatically. No explicit `is` annotation is required.

---

### Real-World Cases

- **API validation**: `isApiResponse(value)` before accessing `data` or `error`.
- **Configuration parsing**: `isConfig(value)` for environment variables.
- **Form data**: `isFormData(value)` for `FormData` instances.

---

## 6. Type Predicate

### Definitions

**Core Definition.** A **type predicate** is the return type annotation of a custom type guard, written as `parameterName is Type`. It declares that when the function returns `true`, the parameter has the specified type.

**Technical Definition.** Type predicates use the syntax `x is T` in the return type position. They are a special form of return type that the compiler interprets as a narrowing assertion. The parameter named in the predicate must be a **parameter of the function**. Type predicates are erased at runtime; they are a compile-time contract.

**Beginner-Friendly Explanation.** A type predicate is **the “if this function returns true, the argument is this type” annotation**. It is how you tell TypeScript that a custom check function narrows the type.

### Purposes

- **To declare the narrowing behavior** of a custom type guard.
- **To document the guard’s contract** for consumers and tooling.
- **To enable safe narrowing** in callers without type assertions.
- **To support reusable guards** across a codebase.

### Syntax Rules and Structure

**General syntax:**

```
function isX(value: unknown): value is X {
  // runtime check
}
```

**Component breakdown:**

- `value` — the parameter being narrowed.
- `is` — the predicate keyword.
- `X` — the target type.

**Syntax rules.**

- The predicate parameter must be a **parameter** of the function.
- The function must return a **boolean**.
- The predicate is erased at runtime.
- Since TS 5.5, predicates are inferred for simple single-return guards.

**Constraints and limitations.**

- Predicates cannot narrow `this` properties directly.
- A predicate that is inconsistent with the runtime check is a bug that the compiler cannot catch.

### Annotated Complete Step by Step Code Examples

#### Example 1: Type predicate with `is`

```typescript
// example_type_predicate.ts
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function processValue(value: string | number) {
  if (isString(value)) {
    console.log(value.toUpperCase()); // narrowed to string
  }
}

processValue("hello"); // Expected: HELLO
```

**Expected Output:**
```
HELLO
```

**Why this output occurs.** The `value is string` predicate tells TypeScript that `isString` narrows its argument to `string`. The caller’s `value` is narrowed accordingly.

---

### Real-World Cases

- **Exported guards**: `export function isUser(value: unknown): value is User`.
- **Library APIs**: documenting narrowing contracts for consumers.
- **Multi-return guards**: explicit predicates for guards that branch.

---

## 7. `is` Keyword

### Definitions

**Core Definition.** The **`is` keyword** is the syntactic token used in a **type predicate** to separate the parameter name from the narrowed type (`value is Type`). It is the language construct that turns a boolean-returning function into a recognized type guard.

**Technical Definition.** The `is` keyword appears only in **return type positions** of functions. It forms a **type predicate** of the form `parameter is Type`. The compiler uses this predicate to narrow the parameter in callers when the function returns `true`. The `is` keyword has no runtime representation and is erased.

**Beginner-Friendly Explanation.** The `is` keyword is **what makes a function a type guard**. You write `value is string` as the return type, and TypeScript treats the function as a narrowing check.

### Purposes

- **To declare a type predicate** explicitly.
- **To enable narrowing** in callers of custom guards.
- **To document the guard’s contract** clearly.
- **To override inferred predicates** when the automatic inference is not what you want.

### Syntax Rules and Structure

**General syntax:**

```
function isX(value: unknown): value is X
```

**Component breakdown:**

- `value` — the parameter.
- `is` — the predicate keyword.
- `X` — the narrowed type.

**Syntax rules.**

- The `is` keyword is used only in **return type** annotations.
- The left side must be a **parameter name**.
- The function must return a boolean.
- Explicit `is` is optional for simple single-return guards (TS 5.5+ infers it).

**Constraints and limitations.**

- Cannot be used outside return type positions.
- Cannot narrow `this` properties.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit `is` keyword

```typescript
// example_is_keyword.ts
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

**Why this output occurs.** The `value is T` predicate tells `filter` that `isNonNull` removes `null` values. The result is `string[]` rather than `(string | null)[]`.

---

### Real-World Cases

- **Generic guards**: `isNonNull<T>(value: T | null): value is T`.
- **Filtering arrays**: `array.filter(isString)`.
- **Exported type contracts**: documenting custom guards.

---

## 8. Reusable Narrowing Logic

### Definitions

**Core Definition.** **Reusable narrowing logic** is the practice of extracting type guard checks into **standalone functions** that can be called from multiple sites, reducing duplication and ensuring consistent runtime validation.

**Technical Definition.** Custom type guards are the primary vehicle for reusable narrowing. By defining `isUser`, `isString`, `isNotNull`, etc., the same runtime check and type predicate are applied consistently. Since TypeScript 5.5, simple guards are inferred automatically, making extraction easier. Reusable guards compose: `isStringArray` can call `isString` for each element, and `isNotNull` can be used in `filter` calls. Libraries like `is-kit` provide combinators (`and`, `or`, `not`) for composing guards.

**Beginner-Friendly Explanation.** Reusable narrowing logic means **writing your type checks once and using them everywhere**. Instead of repeating a complex `typeof`/`in` check in ten places, you put it in a function like `isUser` and call that function. It keeps the checks consistent and your code shorter.

### Purposes

- **To eliminate duplicated runtime checks** across a codebase.
- **To ensure consistent validation** of the same type in multiple places.
- **To compose complex guards** from simpler ones.
- **To make guards testable** in isolation.
- **To document validation logic** in a named function.

### Syntax Rules and Structure

**General syntax (reusable guard):**

```
function isX(value: unknown): value is X {
  return /* shared runtime check */;
}
```

**Composition pattern:**

```
function isXAndY(value: unknown): value is X & Y {
  return isX(value) && isY(value);
}
```

**Component breakdown:**

- The guard function encapsulates the check.
- Composition uses logical operators (`&&`, `||`, `!`).

**Syntax rules.**

- Reusable guards should accept `unknown` for maximum applicability.
- Compose guards with `&&` and `||`; TypeScript narrows within the composition.
- Inferred predicates (TS 5.5+) reduce boilerplate for simple guards.
- Explicit `is` annotations are recommended for exported or multi-return guards.

**Constraints and limitations.**

- Over-extraction can make simple checks harder to read; extract when there is genuine reuse.
- Guard composition may lose narrowing in complex expressions; keep compositions simple.
- A lying reusable guard is more dangerous than a local lie because it affects more call sites.

### Annotated Complete Step by Step Code Examples

#### Example 1: Reusable guard for arrays

```typescript
// example_reusable_guard.ts
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isStringArray(value: unknown): value is string[] {
  return Array.isArray(value) && value.every(isString);
}

function processValue(value: unknown) {
  if (isStringArray(value)) {
    console.log(value.join(", ")); // narrowed to string[]
  }
}

processValue(["a", "b", "c"]); // Expected: a, b, c
```

**Expected Output:**
```
a, b, c
```

**Why this output occurs.** `isStringArray` reuses `isString` for each element via `every`, and its own type predicate narrows `value` to `string[]`. The runtime check is defined once and reused.

---

#### Example 2: Reusable `isNotNull` guard

```typescript
// example_is_not_null.ts
function isNotNull<T>(value: T | null): value is T {
  return value !== null;
}

const users: (string | null)[] = ["Ada", null, "Grace"];
const valid = users.filter(isNotNull);

console.log(valid); // Expected: [ 'Ada', 'Grace' ]
```

**Expected Output:**
```
[ 'Ada', 'Grace' ]
```

**Why this output occurs.** `isNotNull` is a reusable generic guard that removes `null` from the type. `filter` uses the predicate to produce `string[]`.

---

### Real-World Cases

- **Validation libraries**: reusable guards for `isEmail`, `isUrl`, `isUUID`.
- **API clients**: `isApiResponse`, `isErrorResponse` shared across endpoints.
- **Form handling**: `isFormData`, `isFile` guards reused across components.

---

## 9. Type Guard Best Practices

### Definitions

**Core Definition.** **Type guard best practices** are guidelines for writing guards that are **accurate, reusable, and safe**, avoiding the pitfalls of lying guards, unnecessary assertions, and narrowing that does not survive refactoring.

**Technical Definition.** Best practices center on **replacing type assertions with runtime-verified guards**, **naming guards `isX` or `hasX`** so their purpose is clear, **ensuring the runtime check actually matches the predicate** (a lying guard is worse than an assertion), **using inferred predicates for simple single-return guards** (TS 5.5+), **adding explicit `is` annotations for exported or multi-return guards**, **keeping the guarded object intact for discriminated union narrowing**, and **using `unknown` at boundaries and narrowing with guards**.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep type guards trustworthy. The big rules are: use guards instead of `as`; make sure the guard actually checks what it claims; name guards clearly; and do not destructure an object before narrowing a discriminated union.

### Purposes

- **To eliminate unsafe type assertions** and their hidden bugs.
- **To ensure guards are accurate** and do not lie about runtime checks.
- **To maximize narrowing effectiveness** across control flow and refactoring.
- **To make guards self-documenting** with clear names and contracts.
- **To leverage TypeScript 5.5+ inference** to reduce boilerplate.

### Syntax Rules and Structure

Best practices are conventions that interact with guard syntax. The core rules are:

1. **Replace type assertions with guards.** `as` silences the compiler without runtime verification; guards verify at runtime.
2. **Name guards `isX` or `hasX`.** A lying guard is worse than `as` because the bug hides behind a safe-sounding name.
3. **Make the runtime check match the predicate.** If the guard says `value is User`, the body must actually verify the `User` shape.
4. **Use inferred predicates for simple single-return guards.** TS 5.5+ infers `value is T` automatically, keeping the predicate in sync with the implementation.
5. **Add explicit `is` annotations for exported or multi-return guards.** Inference only fires on single returns; explicit annotations document the public contract.
6. **Keep the guarded object intact for discriminated union narrowing.** Destructuring can lose narrowing; access properties through the object (`response.ok` rather than `const { ok } = response`).
7. **Use `unknown` at trust boundaries and narrow with guards.** Avoid `any`; use `unknown` plus guards for external data.
8. **Prefer guards over assertion functions for conditional handling.** Use type predicates when the caller decides what to do with invalid values; use assertion functions when invalid input is a programming error.

**Component breakdown (guard decision tree):**

```
Do you need to narrow a value's type at runtime?
  ├── Is it a primitive? → Use typeof.
  ├── Is it a class instance? → Use instanceof.
  ├── Does it differ by a property? → Use in.
  └── Is it a complex shape? → Write a custom guard (isX).
       ├── Single return? → Let TS 5.5+ infer the predicate.
       └── Multiple returns or exported? → Add explicit `value is Type`.
```

**Constraints and limitations.**

- Inferred predicates only work for single-return functions with a boolean return.
- `instanceof` does not work with interfaces or type aliases.
- `in` does not narrow when the property name is dynamic.
- Guards do not survive reassignment or mutation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Guard instead of assertion

```typescript
// example_best_guard_over_assertion.ts
interface Order {
  total: number;
}

// ❌ Assertion trusts the developer, not the runtime.
function handleSuccessBad(payload: unknown) {
  const order = payload as Order; // Unsafe — no runtime check.
  console.log(order.total);
}

// ✅ Guard verifies at runtime.
function isOrder(value: unknown): value is Order {
  return (
    typeof value === "object" &&
    value !== null &&
    "total" in value &&
    typeof (value as Order).total === "number"
  );
}

function handleSuccessGood(payload: unknown) {
  if (!isOrder(payload)) throw new Error("Invalid payload");
  console.log(payload.total); // narrowed to Order.
}

handleSuccessGood({ total: 100 }); // Expected: 100
```

**Expected Output:**
```
100
```

**Why this output occurs.** The guard `isOrder` performs the runtime check that the assertion skips. When the guard returns `true`, `payload` is narrowed to `Order` and `total` is safe to access.

---

#### Example 2: Inferred predicate keeps implementation and type in sync

```typescript
// example_best_inferred.ts
// TS 5.5+ infers the predicate from the single return.
function isString(value: unknown) {
  return typeof value === "string";
}

// The inferred signature is (value: unknown) => value is string.
function processValue(value: string | number) {
  if (isString(value)) {
    console.log(value.toUpperCase()); // narrowed to string.
  }
}

processValue("hello"); // Expected: HELLO
```

**Expected Output:**
```
HELLO
```

**Why this output occurs.** The inferred predicate is derived from the function body, so changing the body automatically updates the narrowing behavior. This avoids the “out-of-sync” problem where the `is` annotation and the implementation disagree.

---

#### Example 3: Preserving narrowing with discriminated unions

```typescript
// example_best_discriminated.ts
type Response =
  | { ok: true; data: string }
  | { ok: false; error: string };

// ❌ Destructuring loses narrowing.
function handleBad(response: Response) {
  const { ok } = response;
  if (ok) {
    // response.data is not narrowed here.
  }
}

// ✅ Keep the object intact.
function handleGood(response: Response) {
  if (response.ok) {
    console.log(response.data); // narrowed to success variant.
  }
}

handleGood({ ok: true, data: "Hello" }); // Expected: Hello
```

**Expected Output:**
```
Hello
```

**Why this output occurs.** Destructuring `ok` into a separate variable loses the connection to `response`; TypeScript cannot narrow the original object based on the copied property. Accessing `response.ok` directly preserves the discriminant link.

---

### Real-World Cases

- **API clients**: custom guards for response validation instead of `as` casts.
- **Form libraries**: reusable field guards for `string`, `number`, and `File` values.
- **State management**: guards for action payloads before reducer logic.
- **Test utilities**: guards for fixture data shape validation.

---

## References

- TypeScript Handbook — Narrowing - https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- TypeScript Handbook — Advanced Types (Type Guards and Differentiating Types, User-Defined Type Guards) - https://www.typescriptlang.org/docs/handbook/advanced-types.html
- TypeScript 5.5 Release Notes — Inferred Type Predicates - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-5.html
- TypeScript 4.4 Release Notes — Control Flow Analysis of Aliased Conditions and Discriminants - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html
- TypeScript 5.6 Documentation — Official TypeScript Handbook - https://www.typescriptlang.org/docs/
- Microsoft/TypeScript Wiki — FAQ: TypeScript Types Are Erased - https://github.com/microsoft/TypeScript/wiki/FAQ
- Total TypeScript — Type Predicate Inference: The TS 5.5 Feature No One Expected - https://www.totaltypescript.com/type-predicate-inference
- Total TypeScript — Reusable Type Guards - https://www.totaltypescript.com/workshops/typescript-pro-essentials/unions-and-narrowing/reusable-type-guards
- oakoss/agent-skills — Type Guards and Narrowing - https://github.com/oakoss/agent-skills/blob/main/skills/typescript-patterns/references/type-guards.md
- pproenca/dot-skills — Write Custom Type Guards Instead of Type Assertions - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-refactor/references/narrow-custom-type-guards.md
- pproenca/dot-skills — Use Type Guards for Runtime Type Checking - https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.curated/typescript/references/safety-use-type-guards.md
- pproenca/dot-skills — Avoid Type and Non-Null Assertions - https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/ts-google/references/error-avoid-assertions.md
- W3Schools — TypeScript Type Guards - https://www.w3schools.com/typescript/typescript_type_guards.php
- is-kit — Reusable Type Guard Toolkit - https://www.npmjs.com/package/is-kit