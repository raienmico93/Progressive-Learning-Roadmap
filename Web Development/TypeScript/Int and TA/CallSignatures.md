# Call Signatures in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **call signature** is a type-level construct within an object type that declares the object to be **callable as a function**, specifying the parameter types and return type of the call. It enables TypeScript to describe values that are both **function-like** and **object-like**—a pattern common in JavaScript where functions carry additional properties. Call signatures are declared **without a member name**, using the syntax `(params: Type): ReturnType` inside an interface or object type literal.

**Technical Definition.** In TypeScript's grammar, a call signature of the form `(p₁: T₁, p₂: T₂, …): R` within an object type makes the object type callable with arguments of types `T₁, T₂, …` and return type `R`. It is semantically related to **function type expressions** (`(p₁: T₁) => R`), but they are not identical: a function type expression describes **only a function**, while a call signature describes **a callable object that may also have properties, methods, index signatures, or construct signatures**. Multiple call signatures may appear in a single type, enabling **overload resolution**. A call signature may be **generic** (`<T>(arg: T): T`), may include a **`this` parameter**, and may be **optional** in the sense that the containing property may be optional. All call signatures are **erased** during compilation and have no runtime representation.

**Beginner-Friendly Explanation.** A call signature is **a way to say “this object can be called like a function.”** In JavaScript, functions are also objects—you can attach properties to them. TypeScript's function type expression (`(x: number) => string`) only describes the function part, not the properties. A call signature lets you describe **both**: “this thing is callable with these arguments, and it also has these properties.” It is like describing a Swiss Army knife: it has a blade (callable behavior) and a corkscrew (properties).

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Unnamed member** | Call signatures are declared without a member name inside an object type. |
| **Callable object description** | Describes objects that are callable **and** may have additional properties or methods. |
| **Overload support** | Multiple call signatures in one type act as overloads for the callable object. |
| **Generic support** | Call signatures may declare type parameters: `<T>(arg: T): T`. |
| **`this` parameter** | A fake first parameter that types the expected `this` context. |
| **Hybrid types** | Call signatures combine with property signatures to model function objects with metadata. |
| **Erasure** | Call signatures exist only at compile time and emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: interfaces, type aliases, and object types.
- **Function type expressions**: `(params) => ReturnType` syntax.
- **Object-oriented concepts**: methods, properties, and callable values.
- **Generics fundamentals**: type parameters and inference.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Hybrid types** — objects that are both callable and have properties.
- **Function overloading** — multiple call signatures for different argument shapes.
- **Callable interfaces** — interfaces consisting primarily of a call signature.
- **Constructor signatures** — the `new`-able counterpart to call signatures.
- **ESLint `prefer-function-type`** — a lint rule recommending function type expressions over single-call-signature interfaces.

---

## Core Concepts / Features

---

## 1. Call Signatures

### Definitions

**Core Definition.** A **call signature** is an unnamed member of an object type that declares the object to be callable with a specific parameter list and return type. It allows types to describe **callable values that also carry properties or other members**.

**Technical Definition.** A call signature of the form `(param: Type): ReturnType` inside an interface or type literal makes the containing type callable. It differs from a function type expression (`(param: Type) => ReturnType`) in that the call signature is a **member of an object type** and can coexist with property signatures, method signatures, index signatures, and construct signatures. Multiple call signatures in the same type enable **overload resolution**: the compiler selects the first matching signature for a given call. Call signatures may be generic, may include a `this` parameter, and are **erased** at compile time.

**Beginner-Friendly Explanation.** A call signature is **the rule that says “you can call this object with these arguments and get this result.”** It is how you describe something that behaves like a function but also has extra features—like a function that has a `description` property or a `version` number.

### Purposes

- **To describe callable objects that also have properties**, which function type expressions cannot express.
- **To enable type checking of function calls** on objects that carry additional metadata.
- **To support overloading** of callable objects with multiple call forms.
- **To model JavaScript APIs** where functions are augmented with properties (e.g., jQuery, Express middleware, test spies).
- **To provide a foundation for callable interfaces** and hybrid types.

### Syntax Rules and Structure

**General syntax (call signature inside an interface):**

```
interface InterfaceName {
  (param1: Type1, param2: Type2): ReturnType;
  // additional properties or signatures
}
```

**Component breakdown:**

- `(param1: Type1, param2: Type2)` — the parameter list; **no member name** precedes it.
- `: ReturnType` — the return type, separated by a **colon** (not `=>`).
- `;` — terminates the call signature.
- Additional members (properties, methods, index signatures) may follow.

**Syntax rules.**

- A call signature has **no member name**; the parameter list itself is the signature.
- The return type separator is `:` — not `=>` (unlike function type expressions).
- Multiple call signatures may appear in one type; they act as overloads.
- Call signatures may be **generic**: `<T>(arg: T): T`.
- Call signatures may include a **`this` parameter**: `(this: Type, arg: string): void`.
- Call signatures may coexist with property signatures, method signatures, and construct signatures.

**Constraints and limitations.**

- A single call signature with no other members is often better expressed as a **function type expression** (`(param) => ReturnType`); ESLint's `prefer-function-type` rule flags this.
- Call signatures are **erased** and cannot be inspected at runtime.
- The order of multiple call signatures affects overload resolution; more specific signatures should generally come first.
- Classes cannot directly **implement** a call signature; callable interfaces require workarounds (see Callable Interfaces below).

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic call signature with properties (hybrid type)

```typescript
// example_call_signature_basic.ts
// Step 1: Declare an interface with a call signature and a property.
interface DescribableFunction {
  description: string;
  (someArg: number): boolean;
}

// Step 2: Create a function and attach the property.
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}

function myFunc(someArg: number): boolean {
  return someArg > 3;
}
myFunc.description = "default description";

// Step 3: Call the function through the interface type.
doSomething(myFunc); // Expected: default description returned true
```

**Setup.** Compile with `npx tsc example_call_signature_basic.ts --strict`.

**Expected Output:**
```
default description returned true
```

**Why this output occurs.** The interface declares both a call signature (`(someArg: number): boolean`) and a property (`description: string`). The function `myFunc` satisfies the call signature, and the attached `description` property satisfies the property signature. The compiler checks both aspects. This is the **hybrid type** pattern described in the TypeScript Handbook.

---

#### Example 2: Multiple call signatures (overloads)

```typescript
// example_call_signature_overload.ts
interface StringOrNumberParser {
  (input: string): number;
  (input: number): string;
}

const parse: StringOrNumberParser = (input: string | number): any => {
  return typeof input === "string" ? Number(input) : String(input);
};

console.log(parse("42")); // Expected: 42
console.log(parse(42));   // Expected: "42"
```

**Expected Output:**
```
42
"42"
```

**Why this output occurs.** The interface declares two call signatures, acting as overloads. The compiler resolves `parse("42")` to the first signature and `parse(42)` to the second. The implementation signature is not visible to callers; only the overloads are. This pattern is useful for APIs that accept different argument shapes.

---

#### Example 3: Generic call signature

```typescript
// example_call_signature_generic.ts
interface Identity {
  <T>(arg: T): T;
}

const identity: Identity = <T>(arg: T): T => arg;

console.log(identity("hello")); // Expected: hello
console.log(identity(42));      // Expected: 42
```

**Expected Output:**
```
hello
42
```

**Why this output occurs.** The call signature declares a generic type parameter `T`, making the callable object generic. Each call infers `T` from the argument, preserving the input type in the return type.

---

### Real-World Cases

- **jQuery-style APIs**: `$(selector)` is callable and has properties like `$.ajax`.
- **Test spies**: Jest/Vitest mocks are callable and carry properties like `.mock.calls`.
- **Express middleware**: middleware functions can have properties attached for configuration.
- **Event emitters**: callable objects that also expose event-related methods.

---

## 2. Callable Interfaces

### Definitions

**Core Definition.** A **callable interface** is an interface whose primary or sole purpose is to describe a **callable value** via one or more call signatures. It may contain only a call signature (a pure function contract) or combine call signatures with properties and methods (a hybrid type).

**Technical Definition.** A callable interface is an interface that includes at least one call signature. When it contains **only** a single call signature, it is semantically equivalent to a function type expression, and ESLint's `prefer-function-type` rule recommends using the function type form instead. When it contains **multiple call signatures** or **additional members**, it cannot be expressed as a function type expression and is the appropriate tool. Callable interfaces may be **generic**, may include **`this` parameters**, and may be **implemented by functions** (but not directly by classes without workarounds).

**Beginner-Friendly Explanation.** A callable interface is **an interface that describes something you can call**. If it only describes a function, a function type expression is usually cleaner. If it describes a function **with extras**—like properties or multiple call forms—a callable interface is the right tool.

### Purposes

- **To name a function-like contract** that will be reused across a codebase.
- **To combine callable behavior with additional properties or methods** (hybrid types).
- **To declare multiple call signatures** (overloads) on a single named interface.
- **To provide a foundation for dependency injection** of function-like services.
- **To document the callable contract** of APIs that are both functions and objects.

### Syntax Rules and Structure

**General syntax (pure callable interface):**

```
interface CallableInterface {
  (param: Type): ReturnType;
}
```

**General syntax (hybrid callable interface):**

```
interface HybridInterface {
  (param: Type): ReturnType;
  property: Type;
  method(): void;
}
```

**Component breakdown:**

- `(param: Type): ReturnType` — the call signature.
- `property: Type` — additional properties.
- `method(): void` — additional methods.

**Syntax rules.**

- A callable interface may contain one or more call signatures.
- When it contains **only one call signature and no other members**, prefer a function type expression for brevity (ESLint `prefer-function-type`).
- When it contains **multiple call signatures** or **additional members**, an interface or type alias is appropriate.
- Callable interfaces may be implemented by **function expressions**, not by classes directly (classes require workarounds such as interface-class merging or a `Callable` helper).
- Generic callable interfaces are supported.

**Constraints and limitations.**

- Classes **cannot** directly `implements` a call signature; the compiler reports that the class provides no match for the signature.
- Pure single-signature callable interfaces are considered stylistic noise by the `prefer-function-type` rule.
- Callable interfaces are erased and provide no runtime reflection.

### Annotated Complete Step by Step Code Examples

#### Example 1: Pure callable interface (and why a function type is preferred)

```typescript
// example_callable_pure.ts
// ❌ A single-call-signature interface (prefer-function-type flags this).
interface Greeter {
  (name: string): string;
}

// ✅ A function type expression is more succinct for a pure function.
type GreeterFn = (name: string) => string;

const greet: GreeterFn = (name) => `Hello, ${name}`;

console.log(greet("Ada")); // Expected: Hello, Ada
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The interface `Greeter` and the type alias `GreeterFn` are **semantically equivalent**. ESLint's `prefer-function-type` rule recommends the function type form because it is more succinct and avoids the false implication that the type has object-like behavior.

---

#### Example 2: Callable interface implemented by a function

```typescript
// example_callable_function.ts
interface ReturnString {
  (): string;
}

declare const foo: ReturnString;

const bar = foo();

console.log(typeof bar); // Expected: string
```

**Expected Output:**
```
string
```

**Why this output occurs.** The callable interface `ReturnString` describes a function that takes no arguments and returns a string. A function value can be assigned to it. This is the basic use case for callable interfaces: naming a function contract.

---

#### Example 3: Hybrid callable interface (multiple members)

```typescript
// example_callable_hybrid.ts
interface Counter {
  (start: number): string;
  interval: number;
  reset(): void;
}

function getCounter(): Counter {
  const counter = ((start: number) => `Count: ${start}`) as Counter;
  counter.interval = 1000;
  counter.reset = () => console.log("Reset");
  return counter;
}

const c = getCounter();
console.log(c(10));      // Expected: Count: 10
console.log(c.interval); // Expected: 1000
c.reset();               // Expected: Reset
```

**Expected Output:**
```
Count: 10
1000
Reset
```

**Why this output occurs.** The `Counter` interface declares a call signature **plus** a property (`interval`) and a method (`reset`). A function type expression cannot describe this combination. The implementation uses a type assertion to attach the properties, which is the standard pattern for hybrid types.

---

#### Example 4: Callable interface with a class workaround

```typescript
// example_callable_class.ts
// Callable interfaces cannot be implemented by classes directly.
// Workaround: interface-class merging with a Callable helper.

export interface Callable<T extends unknown[] = unknown[], R = unknown> {
  (...args: T): R;
}

export class Callable<T extends unknown[] = unknown[], R = unknown>
  extends Function {
  public constructor(func: (...args: T) => R) {
    super();
    return Object.setPrototypeOf(func, new.target.prototype) as typeof this;
  }
}

class Test extends Callable<[number, number], number> {
  public multiplier = 2;

  public constructor() {
    super((a, b) => (a + b) * this.multiplier);
  }
}

const test = new Test();
console.log(test(1, 2)); // Expected: 6

test.multiplier = 3;
console.log(test(1, 2)); // Expected: 9
```

**Expected Output:**
```
6
9
```

**Why this output occurs.** A class cannot directly `implements` a call signature. The `Callable` helper class extends `Function` and merges with a same-named interface that provides the call signature. The constructor uses `Object.setPrototypeOf` to make the instance callable. This pattern is a workaround and has performance implications.

---

### Real-World Cases

- **Function contracts**: naming a callback type that is used in multiple places.
- **Hybrid APIs**: callable objects with metadata, configuration, or helper methods.
- **Dependency injection**: function-like services typed by a callable interface.
- **Test utilities**: callable mocks with tracking properties.

---

## 3. Call Signature Best Practices

### Definitions

**Core Definition.** **Call signature best practices** are guidelines for using call signatures and callable interfaces in TypeScript in ways that maximize **clarity, safety, and maintainability**, while avoiding stylistic noise and unnecessary complexity.

**Technical Definition.** Best practices center on the **`prefer-function-type` principle**: when a type consists of a **single call signature and no other members**, prefer a **function type expression** (`(param: Type) => ReturnType`) over an interface or object type literal with a call signature. Use callable interfaces or hybrid types only when the type has **multiple call signatures** (overloads), **additional properties or methods**, or is **extended by other types**. For hybrid types, document the callable contract clearly and consider whether a plain object with a method would be simpler. Avoid callable interfaces when a class implementation is required, because classes cannot directly implement call signatures.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep call signatures from becoming noise. The big rule is: **if it is just a function, use a function type**. Use a callable interface only when the thing is **more than a function**—when it has properties, methods, or multiple call forms. And remember that classes cannot directly implement callable interfaces, so plan accordingly.

### Purposes

- **To reduce visual noise** by preferring function type expressions for pure functions.
- **To ensure the type accurately reflects runtime behavior** (callable + properties).
- **To avoid compiler and lint warnings** from `prefer-function-type`.
- **To choose the simplest correct construct** for each scenario.
- **To plan for class implementation** when callable behavior is needed on class instances.

### Syntax Rules and Structure

Best practices are conventions that interact with call signature syntax. The core rules are:

1. **Prefer function type expressions for single-signature callables.** Use `type Greet = (name: string) => string` instead of `interface Greet { (name: string): string }`. ESLint's `prefer-function-type` rule enforces this.
2. **Use callable interfaces or hybrid types only when necessary.** Appropriate when the type has **multiple call signatures**, **additional members**, or is **intended for extension**.
3. **Document hybrid types clearly.** When a type is both callable and has properties, explain the dual nature in comments or naming.
4. **Avoid callable interfaces for class implementations.** Classes cannot directly `implements` call signatures; use function types for function properties on classes instead.
5. **Place specific overloads first.** When multiple call signatures exist, order them from most specific to most general.
6. **Use generics for type-preserving callables.** Prefer `<T>(arg: T): T` over `(arg: any) => any` when the return type depends on the input type.
7. **Consider `this` parameters for callback APIs.** Declare `this: Type` when the callable's `this` context is controlled by the caller.

**Component breakdown (call signature vs. function type decision tree):**

```
Does the type have only ONE call signature and NO other members?
  ├── Yes → Use a function type expression ((p: T) => R).
  └── No → Does it have multiple call signatures or additional members?
           ├── Yes → Use a callable interface or hybrid type.
           └── No → Re-evaluate; function type is likely sufficient.
```

**Constraints and limitations.**

- `prefer-function-type` is a **stylistic** rule; teams may disable it if they prefer interfaces for consistency.
- Hybrid types require type assertions or helper functions to attach properties to functions.
- Callable interfaces cannot be directly implemented by classes.
- Overload order affects resolution; incorrect ordering can make specific signatures unreachable.

### Annotated Complete Step by Step Code Examples

#### Example 1: Prefer function type over single call signature

```typescript
// example_best_function_type.ts
// ❌ Interface with a single call signature (flagged by prefer-function-type).
// interface Greet {
//   (name: string): string;
// }

// ✅ Function type expression — more succinct.
type Greet = (name: string) => string;

const greet: Greet = (name) => `Hello, ${name}`;
console.log(greet("Ada")); // Expected: Hello, Ada
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The function type expression is shorter, has no unnecessary interface declaration, and clearly communicates that `Greet` is a function type. ESLint's `prefer-function-type` rule recommends this form for single-signature callables.

---

#### Example 2: Callable interface for multiple signatures (appropriate use)

```typescript
// example_best_multi_signature.ts
// ✅ Callable interface for multiple call signatures.
interface Parser {
  (input: string): number;
  (input: number): string;
}

const parse: Parser = (input: string | number): any =>
  typeof input === "string" ? Number(input) : String(input);

console.log(parse("42")); // Expected: 42
console.log(parse(42));   // Expected: "42"
```

**Expected Output:**
```
42
"42"
```

**Why this output occurs.** A function type expression cannot express two different call signatures. The callable interface is the correct tool for overloaded callable objects.

---

#### Example 3: Hybrid type with documented dual nature

```typescript
// example_best_hybrid.ts
// ✅ Hybrid type: callable AND has properties.
// The callable returns a formatted string; `version` provides metadata.
interface VersionedFormatter {
  (value: number): string;
  version: string;
}

const formatter: VersionedFormatter = Object.assign(
  (value: number) => `v${value}`,
  { version: "1.0.0" }
);

console.log(formatter(2));        // Expected: v2
console.log(formatter.version);  // Expected: 1.0.0
```

**Expected Output:**
```
v2
1.0.0
```

**Why this output occurs.** The hybrid type combines a call signature with a `version` property. `Object.assign` attaches the property to the function. The interface documents both aspects clearly. This is an appropriate use of a callable interface because a function type expression cannot describe the property.

---

### Real-World Cases

- **Function type aliases**: the standard way to name callback types.
- **Overloaded APIs**: callable interfaces for functions with multiple call forms.
- **Hybrid libraries**: callable objects with metadata, configuration, or helper methods.
- **Code review**: enforcing `prefer-function-type` to keep type definitions concise.

---

## References

- TypeScript Handbook — More on Functions: Call Signatures - https://www.typescriptlang.org/docs/handbook/2/functions.html#call-signatures
- TypeScript Handbook — More on Functions: Construct Signatures (Combining Call and Construct Signatures) - https://www.typescriptlang.org/docs/handbook/2/functions.html#construct-signatures
- TypeScript Handbook — Object Types (Interfaces with Call Signatures) - https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript ESLint — `prefer-function-type` Rule - https://typescript-eslint.io/rules/prefer-function-type/
- Biome — `useShorthandFunctionType` Rule - https://biomejs.dev/linter/rules/use-shorthand-function-type/
- Stack Overflow — Can a TypeScript class implement a callable interface? - https://stackoverflow.com/questions/66375915
- GeeksforGeeks — What are Hybrid Types in TypeScript? - https://origin.geeksforgeeks.org/what-are-hybrid-types-in-typescript/
- TypeScript Playground — Call Signatures Example - https://www.typescriptlang.org/play/typescript/language-extensions/functions.ts.html
- TypeScript Language Specification — Call Signatures - https://raw.githubusercontent.com/microsoft/TypeScript/refs/tags/v2.3.3/doc/spec.md
- TypeScript 3.3 Release Notes — Improved Behavior for Calling Union Types (Call Signature Resolution) - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-3.html