# Method Signatures in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **method signature** is a type-level declaration that specifies the **name**, **parameter types**, and **return type** of a method within an object type, interface, or class. It defines the **callable contract** that conforming values must satisfy, without providing an implementation. Method signatures are the primary mechanism by which TypeScript describes the behavior of functions attached to objects and classes.

**Technical Definition.** In TypeScript's grammar, a method signature of the form `f<T₁, T₂, …>(p₁, p₂, …): R` is **equivalent** to a property declaration whose type is a callable object type with the same signature: `f: { <T₁, T₂, …>(p₁, p₂, …): R }`. TypeScript provides **two syntactic forms** for describing function members: **method shorthand** (`func(arg: string): number`) and **property function type** (`func: (arg: string) => number`). These are **semantically equivalent** for most purposes, but diverge in **variance behavior**: under `strictFunctionTypes`, property-style declarations receive **contravariant** parameter checking, while method-style declarations remain **bivariant**. Method signatures support **overloading** via multiple declarations with the same name, and may include an explicit **`this` parameter** to declare the expected `this` context. All method signatures are **erased** during compilation.

**Beginner-Friendly Explanation.** A method signature is **the contract for a function that belongs to an object or class**. It tells you what the method is called, what arguments it takes, and what it returns—without giving you the actual code. TypeScript uses these contracts to check that you are calling methods correctly and that classes implement the methods they promise. You can write method signatures in two styles: the "method style" (`greet(name: string): string`) and the "property style" (`greet: (name: string) => string`). They look different but do the same thing, with one subtle difference in how strictly TypeScript checks function parameters.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Two equivalent forms** | Method shorthand and property function type are semantically equivalent except for variance under `strictFunctionTypes`. |
| **Overload support** | A method may declare multiple signatures with the same name but different parameter lists. |
| **`this` parameter** | A fake first parameter that types the expected `this` context; erased at compile time. |
| **Bivariance vs. contravariance** | Method-style parameters are bivariant; property-style parameters are contravariant under `strictFunctionTypes`. |
| **Implementation signature invisibility** | In overloaded methods, the implementation signature cannot be called directly. |
| **Erasure** | All method signatures are compile-time only and emit no JavaScript. |
| **Optional and generic** | Methods may be optional (`?`), generic (`<T>`), and overloaded. |

---

### Prerequisites

- **TypeScript basics**: interfaces, type aliases, and object types.
- **Function types**: function type expressions and call signatures.
- **Classes**: methods, `this`, and access modifiers.
- **Compiler flags**: `strict`, `strictFunctionTypes`, `noImplicitThis`.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Object type design** in interfaces and type aliases.
- **Class method declarations** and inheritance.
- **Function overloading** and signature resolution.
- **Callback-style APIs** and `this` binding.
- **Variance and type compatibility** under `strictFunctionTypes`.

---

## Core Concepts / Features

---

## 1. Method Signatures

### Definitions

**Core Definition.** A **method signature** is a type-level declaration of a method's name, parameter types, and return type within an object type, interface, or class. It specifies the callable contract without providing an implementation.

**Technical Definition.** A method signature of the form `funcName<T>(params): ReturnType` is equivalent to a property whose type is a callable object type with the same signature. TypeScript supports two syntactic forms: **method shorthand** (`funcName(params): ReturnType`) and **property function type** (`funcName: (params) => ReturnType`). Under `strictFunctionTypes`, property-style declarations receive **contravariant** parameter checking, while method-style declarations remain **bivariant**. Method signatures may be optional (`funcName?()`), generic (`funcName<T>()`), and overloaded.

**Beginner-Friendly Explanation.** A method signature is **a description of what a method looks like**—its name, its arguments, and its return value. It is the type-level version of a function declaration. You use it in interfaces and object types to say "any object conforming to this type must have a method that looks like this."

### Purposes

- **To declare the callable contract** of methods on object types, interfaces, and classes.
- **To enable type checking of method calls**, ensuring arguments match parameter types and return values are used correctly.
- **To document the public API** of classes and objects through explicit method signatures.
- **To support polymorphism** by allowing objects with compatible method signatures to be used interchangeably.
- **To provide two equivalent syntaxes** (method shorthand and property function type) for different variance needs.

### Syntax Rules and Structure

**General syntax (method shorthand):**

```
interface InterfaceName {
  methodName(params: Type): ReturnType;
}
```

**General syntax (property function type):**

```
interface InterfaceName {
  methodName: (params: Type) => ReturnType;
}
```

**Component breakdown:**

- `methodName` — the method identifier.
- `(params: Type)` — the parameter list with type annotations.
- `: ReturnType` — the return type (omitted for `void`-returning methods).
- `;` — terminates the signature in an interface or type alias.

**Syntax rules.**

- Method signatures may be **optional**: `methodName?(): void`.
- Method signatures may be **generic**: `methodName<T>(arg: T): T`.
- Method signatures may be **overloaded** by repeating the name with different parameter lists.
- The method shorthand form is generally preferred for interfaces unless `strictFunctionTypes` requires property-style variance.
- Methods may be `public`, `private`, or `protected` in classes; interfaces cannot declare private or protected methods.

**Constraints and limitations.**

- Method signatures are **erased** at runtime and have no JavaScript representation.
- Method-style parameters are **bivariant** by default, which is unsound but matches JavaScript callback patterns.
- Property-style parameters are **contravariant** under `strictFunctionTypes` and are safer for function properties.
- Interfaces cannot declare `private` or `protected` methods.

### Annotated Complete Step by Step Code Examples

#### Example 1: Method shorthand signature

```typescript
// example_method_shorthand.ts
// Step 1: Declare an interface with a method shorthand signature.
interface Greeter {
  greet(name: string): string;
}

// Step 2: Implement the interface with an object literal.
const greeter: Greeter = {
  greet(name) {
    return `Hello, ${name}`;
  },
};

// Step 3: Call the method; TypeScript checks the signature.
console.log(greeter.greet("Ada")); // Expected: Hello, Ada

// greeter.greet(42); // Error: number not assignable to string.
```

**Setup.** Compile with `npx tsc example_method_shorthand.ts --strict`.

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The `Greeter` interface requires a `greet` method that takes a `string` and returns a `string`. The object literal provides a compatible method. The compiler checks both the parameter and return types. At runtime, the method executes normally; the interface is erased.

---

#### Example 2: Property function type signature

```typescript
// example_method_property.ts
// Step 1: Declare an interface with a property function type.
interface Validator {
  validate: (value: string) => boolean;
}

// Step 2: Implement with an arrow function (property style).
const validator: Validator = {
  validate: (value) => value.length > 0,
};

console.log(validator.validate("hello")); // Expected: true
console.log(validator.validate(""));      // Expected: false
```

**Expected Output:**
```
true
false
```

**Why this output occurs.** The property function type declares `validate` as a property whose value is a function. The object provides a compatible arrow function. Under `strictFunctionTypes`, this form receives contravariant parameter checking, which is stricter than the method shorthand form.

---

#### Example 3: Optional method signature

```typescript
// example_method_optional.ts
interface Logger {
  log?: (message: string) => void;
}

function safeLog(logger: Logger, message: string) {
  // Must check for optionality before calling.
  logger.log?.(message);
}

safeLog({ log: (m) => console.log(m) }, "Hello"); // Expected: Hello
safeLog({}, "Ignored"); // No output — log is undefined.
```

**Expected Output:**
```
Hello
```

**Why this output occurs.** `log` is an optional method property. The optional chaining operator `?.` safely invokes it only if it is present. The second call produces no output because `log` is `undefined`.

---

### Real-World Cases

- **Service interfaces**: `interface ILogger { log(message: string): void; }`.
- **Validation contracts**: `interface Validator { validate(value: unknown): boolean; }`.
- **Event handlers**: `interface Handler { handle(event: Event): void; }`.
- **React props**: `interface ButtonProps { onClick: (event: MouseEvent) => void; }`.

---

## 2. Method Overloading

### Definitions

**Core Definition.** **Method overloading** is the technique of declaring **multiple method signatures with the same name** but different parameter lists, followed by a **single implementation signature** that handles all cases. It allows a method to be called in different ways while maintaining precise type checking for each call form.

**Technical Definition.** In TypeScript, overloaded methods consist of **two or more overload signatures** (without bodies) followed by **one implementation signature** (with a body). The implementation signature must be **compatible with all overload signatures** but is **not visible from the outside**—callers can only use the overload signatures. The compiler resolves calls to a **single overload** based on the argument types; it cannot resolve calls that match only the implementation signature or that require a union of multiple overloads. Overload signatures must appear **consecutively** above the implementation, and their order matters for resolution.

**Beginner-Friendly Explanation.** Method overloading lets you write **several "menu options" for how a method can be called**, then one actual implementation that handles all of them. For example, a `makeDate` method might accept either a timestamp or a month/day/year combination. You write two signatures describing those two ways of calling it, and one implementation that checks what it received and does the right thing. Callers only see the menu options, not the implementation.

### Purposes

- **To support multiple calling conventions** for a single method with precise type checking.
- **To provide rich IntelliSense and documentation** for each overload form.
- **To model JavaScript APIs** that accept different argument shapes (e.g., `Date` constructors, array methods).
- **To maintain backward compatibility** while adding new call forms.

### Syntax Rules and Structure

**General syntax:**

```
function methodName(param1: Type1): ReturnType1;
function methodName(param1: Type1, param2: Type2): ReturnType2;
function methodName(...args: any[]): any {
  // implementation
}
```

**Component breakdown:**

- **Overload signatures** — one or more declarations without bodies, each with a different parameter list.
- **Implementation signature** — a function with a body that is compatible with all overloads.
- The implementation signature is **not callable** directly; only the overloads are visible to callers.

**Syntax rules.**

- Overload signatures must appear **consecutively** above the implementation signature.
- The implementation signature must be **compatible** with every overload signature (parameters and return types).
- Overloads must either **all be required** or **all be optional**—mixing is not allowed.
- The order of overloads matters: more specific signatures should generally come before more general ones.
- Use `function` declarations, not arrow functions, for overloads in object literals (overloads in object literals require a workaround).

**Constraints and limitations.**

- The compiler resolves a call to **a single overload**; it cannot resolve calls that would require a union of overloads.
- The implementation signature is **not visible** to callers; even if it accepts more parameters, those cannot be used at call sites.
- Overloads with identical parameter types and return types are **redundant** and should be collapsed.
- **Prefer union types over overloads** when the function signature is simple.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic method overloading

```typescript
// example_overload_basic.ts
// Step 1: Declare overload signatures (no bodies).
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;

// Step 2: Provide the implementation signature.
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}

// Step 3: Call with different argument forms.
const d1 = makeDate(12345678); // ✅ One argument
const d2 = makeDate(5, 5, 5);  // ✅ Three arguments

console.log(d1 instanceof Date); // Expected: true
console.log(d2 instanceof Date); // Expected: true

// const d3 = makeDate(1, 3); // ❌ Error: No overload expects 2 arguments.
```

**Expected Output:**
```
true
true
```

**Why this output occurs.** The two overload signatures define the valid call forms: one argument or three arguments. The implementation signature accepts one required and two optional parameters, but **cannot be called with two arguments** because no overload permits that. The compiler resolves `makeDate(12345678)` to the first overload and `makeDate(5, 5, 5)` to the second.

---

#### Example 2: Overloading with incompatible implementation signature

```typescript
// example_overload_incompatible.ts
function fn(x: boolean): void;
// ❌ Overload signature is not compatible with implementation.
// function fn(x: string): void;
// Error: This overload signature is not compatible with its implementation signature.

function fn(x: boolean) {}
console.log("Incompatible overload is rejected");
```

**Expected Output:**
```
Incompatible overload is rejected
```

**Why this output occurs.** The overload `fn(x: string): void` is incompatible with the implementation `fn(x: boolean)` because a `string` is not assignable to `boolean`. TypeScript rejects the overload at declaration time.

---

#### Example 3: Union types preferred over overloads

```typescript
// example_overload_union.ts
// ❌ Overloaded version — redundant and harder to maintain.
// function len(s: string): number;
// function len(arr: any[]): number;
// function len(x: any) { return x.length; }

// ✅ Union version — simpler and supports mixed-union calls.
function len(x: string | any[]): number {
  return x.length;
}

console.log(len("hello")); // Expected: 5
console.log(len([1, 2, 3])); // Expected: 3
```

**Expected Output:**
```
5
3
```

**Why this output occurs.** The union version is simpler and, unlike the overloaded version, **supports calls with a union of the parameter types** (`string | any[]`). The TypeScript Handbook explicitly recommends preferring union types over overloads when possible.

---

### Real-World Cases

- **Date constructors**: `makeDate(timestamp)` or `makeDate(m, d, y)`.
- **Array methods**: `Array.prototype.concat` with different argument shapes.
- **Event emitters**: `emit(event: string)` or `emit(event: string, payload: T)`.
- **Utility functions**: `createElement(tag: string)` or `createElement(tag: string, props: object)`.

---

## 3. `this` Parameter

### Definitions

**Core Definition.** The **`this` parameter** is a **fake first parameter** in a function or method signature that declares the **expected type of `this`** inside the function body. It allows TypeScript to check that the function is called with the correct `this` context.

**Technical Definition.** JavaScript does not allow a parameter named `this`, so TypeScript uses that syntax space to let you **declare the type for `this`** in the function body. The `this` parameter appears **first in the parameter list**, is **erased** during compilation, and does not count as a real argument. It is especially useful for **callback-style APIs** where another object controls when your function is called. Under `noImplicitThis`, TypeScript warns when `this` is used without an explicit type and cannot be inferred.

**Beginner-Friendly Explanation.** The `this` parameter lets you **tell TypeScript what `this` should be** when a function runs. Normally, `this` is whatever object the function is called on, and it can change unexpectedly. By declaring `this: User`, you force TypeScript to check that the function is only called with a `User` as `this`. It is like a safety label: "this function expects to be run inside a `User`."

### Purposes

- **To type the `this` context explicitly** in functions and methods.
- **To prevent `this`-related runtime errors** in callbacks and detached methods.
- **To enable safe callback-style APIs** where the caller controls the `this` binding.
- **To satisfy `noImplicitThis`** by providing an explicit type for `this`.

### Syntax Rules and Structure

**General syntax:**

```
function functionName(this: Type, param: ParamType): ReturnType {
  // this is typed as Type
}
```

**Component breakdown:**

- `this: Type` — the **fake first parameter** declaring the type of `this`.
- `param: ParamType` — the real parameters that follow.
- The `this` parameter is **not counted** as an argument at call sites.

**Syntax rules.**

- The `this` parameter must be the **first parameter** in the list.
- It is **erased** at compile time and has no runtime effect.
- The `this` parameter is **not visible** to callers as an argument.
- For callback APIs, use `function` expressions, **not arrow functions**, to get the `this` behavior.
- Under `noImplicitThis`, `this` must be explicitly typed or inferable from context.

**Constraints and limitations.**

- Arrow functions capture the **lexical** `this` and cannot have a `this` parameter.
- The `this` parameter does not change the runtime behavior of `this`; it only adds compile-time checking.
- TypeScript infers `this` via control-flow analysis in many cases, so an explicit `this` parameter is not always necessary.

### Annotated Complete Step by Step Code Examples

#### Example 1: `this` parameter in a callback API

```typescript
// example_this_callback.ts
interface User {
  id: number;
  admin: boolean;
}

interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();

// ✅ Correct: function expression with explicit this: User.
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});

function getDB(): DB {
  return {
    filterUsers(filter) {
      return [
        { id: 1, admin: true },
        { id: 2, admin: false },
      ].filter((user) => filter.call(user));
    },
  };
}

console.log(admins.length); // Expected: 1
```

**Expected Output:**
```
1
```

**Why this output occurs.** The `filter` parameter is typed as `(this: User) => boolean`, declaring that `this` will be a `User` when the callback is invoked. The function expression uses `this.admin`, which the compiler can check because of the `this` parameter. The `filter.call(user)` binding ensures the correct `this` at runtime.

---

#### Example 2: `this` parameter with `noImplicitThis`

```typescript
// example_this_noImplicit.ts
// Compile with: npx tsc example_this_noImplicit.ts --strict --noImplicitThis

function greet(this: { name: string }): string {
  return `Hello, ${this.name}`;
}

const obj = { name: "Ada", greet };
console.log(obj.greet()); // Expected: Hello, Ada

// greet(); // Error: The 'this' context of type 'void' is not assignable to method's 'this' of type '{ name: string }'.
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The `this: { name: string }` parameter tells TypeScript that `greet` must be called with an object that has a `name` string. Calling `obj.greet()` satisfies this requirement. Calling `greet()` without a proper `this` is rejected at compile time under `noImplicitThis`.

---

#### Example 3: Arrow functions capture lexical `this`

```typescript
// example_this_arrow.ts
interface Counter {
  count: number;
  increment: () => void;
}

const counter: Counter = {
  count: 0,
  // ❌ Arrow function captures lexical this, not the object.
  increment: () => {
    // this.count++; // Error: 'this' implicitly has type 'any'.
  },
};

// ✅ Use a function expression with this parameter.
const safeCounter = {
  count: 0,
  increment: function (this: { count: number }) {
    this.count++;
  },
};

safeCounter.increment();
console.log(safeCounter.count); // Expected: 1
```

**Expected Output:**
```
1
```

**Why this output occurs.** Arrow functions capture the **lexical** `this` and cannot have a `this` parameter. The function expression with `this: { count: number }` correctly types the `this` context and allows mutation of `count`.

---

### Real-World Cases

- **Callback APIs**: `db.filterUsers(function (this: User) { ... })`.
- **Event handlers**: `function (this: HTMLElement, event: Event) { ... }`.
- **Detached methods**: binding a method to a specific object with compile-time safety.
- **Fluent APIs**: methods that return `this` for chaining.

---

## 4. Method Signature Best Practices

### Definitions

**Core Definition.** **Method signature best practices** are guidelines for declaring and using method signatures in TypeScript in ways that maximize **type safety, consistency, and maintainability**, while avoiding common pitfalls such as unnecessary overloads, unreachable signatures, and inconsistent syntax.

**Technical Definition.** Best practices center on **preferring union types over overloads when possible**, **choosing property-style signatures for stricter checking under `strictFunctionTypes`**, **keeping overloads consecutive and correctly ordered**, **using `this` parameters for callback APIs**, **avoiding unreachable or redundant overloads**, and **enforcing consistency with lint rules** such as `method-signature-style` and `useConsistentMethodSignatures`. The TypeScript Handbook explicitly recommends: "Always prefer parameters with union types instead of overloads when possible".

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep method signatures clean and safe. The big rules are: use union types instead of overloads when you can; use property-style signatures if you want stricter type checking; keep overloaded signatures together and in the right order; and use `this` parameters when you need to type callbacks.

### Purposes

- **To maximize type safety** by choosing the strictest appropriate signature style.
- **To reduce unnecessary complexity** by preferring union types over overloads.
- **To prevent unreachable or redundant overloads** that confuse callers and the compiler.
- **To maintain consistency** across a codebase with lint rules.
- **To type `this` contexts correctly** in callbacks and detached functions.

### Syntax Rules and Structure

Best practices are conventions that interact with method signature syntax. The core rules are:

1. **Prefer union types over overloads.** Use `function len(x: string | any[]): number` instead of two overloads. Union versions support calls with union-typed arguments, which overloads cannot resolve.
2. **Choose property-style signatures for stricter checking.** Under `strictFunctionTypes`, property-style function members receive contravariant parameter checking, while method-style members remain bivariant.
3. **Keep overloads consecutive.** Group overloaded signatures together above the implementation. ESLint's `adjacent-overload-signatures` rule enforces this.
4. **Order overloads from most specific to most general.** The compiler resolves calls to the first matching overload, so specific signatures should come first.
5. **Avoid unreachable overloads.** If an overload's parameter types are subsumed by an earlier overload, it can never be selected. Collapse identical-parameter overloads into a single signature with a union return type.
6. **Use `this` parameters for callback APIs.** When a function's `this` context is controlled by the caller, declare it explicitly to catch binding errors at compile time.
7. **Be consistent.** Use Biome's `useConsistentMethodSignatures` or TypeScript ESLint's `method-signature-style` to enforce either method-style or property-style across the codebase.

**Component breakdown (method signature decision tree):**

```
Does the function accept different argument shapes?
  ├── No → Use a single signature (method or property style).
  └── Yes → Can a union type express the variation?
            ├── Yes → Use a union type (preferred).
            └── No → Use overloads (consecutive, specific-first).
                     └── Does the function rely on a specific this context?
                         ├── Yes → Add a this parameter (function expression).
                         └── No → Proceed.
```

**Constraints and limitations.**

- Overloads with identical parameter types and return types are redundant and should be collapsed.
- Method-style signatures are bivariant and less strict than property-style under `strictFunctionTypes`.
- `this` parameters cannot be used with arrow functions.
- Union types may not always express complex overload relationships; overloads are still appropriate for genuinely different call forms.

### Annotated Complete Step by Step Code Examples

#### Example 1: Union types over overloads

```typescript
// example_best_union.ts
// ❌ Overloaded version — cannot resolve union-typed calls.
// function len(s: string): number;
// function len(arr: any[]): number;
// function len(x: any) { return x.length; }

// ✅ Union version — simpler and supports union-typed calls.
function len(x: string | any[]): number {
  return x.length;
}

const input: string | any[] = Math.random() > 0.5 ? "hello" : [1, 2, 3];
console.log(len(input)); // Expected: 5 or 3
```

**Expected Output (approximate):**
```
5
```

**Why this output occurs.** The union version can be called with `string | any[]`, while the overloaded version would reject such a call because the compiler cannot resolve it to a single overload. The TypeScript Handbook explicitly recommends union types over overloads when possible.

---

#### Example 2: Property-style signatures for stricter checking

```typescript
// example_best_property_style.ts
// ✅ Property-style signatures receive contravariant checking.
interface Emitter {
  emit: (event: Event) => void;
}

// ✅ Method-style signatures are bivariant.
interface MethodEmitter {
  emit(event: Event): void;
}

// Under strictFunctionTypes, property-style is stricter.
console.log("Property-style signatures are stricter under strictFunctionTypes");
```

**Expected Output:**
```
Property-style signatures are stricter under strictFunctionTypes
```

**Why this output occurs.** Under `strictFunctionTypes`, property-style function members are checked **contravariantly**, while method-style members remain **bivariant**. This makes property-style signatures safer for function properties where the parameter types matter.

---

#### Example 3: Consecutive overloads and `this` parameter

```typescript
// example_best_consecutive_this.ts
interface EventBus {
  on(event: "click", handler: (this: HTMLElement) => void): void;
  on(event: "keydown", handler: (this: HTMLElement) => void): void;
  on(event: string, handler: (this: any) => void): void;
}

const bus: EventBus = {
  on(event: string, handler: (this: any) => void) {
    console.log(`Registered: ${event}`);
  },
};

bus.on("click", function (this: HTMLElement) {
  console.log("Clicked");
}); // Expected: Registered: click
```

**Expected Output:**
```
Registered: click
```

**Why this output occurs.** The overloads are **consecutive** and **specific-first** (the literal `"click"` and `"keydown"` overloads come before the general `string` overload). The `this` parameter types the callback's `this` context as `HTMLElement`, catching binding errors at compile time. The implementation signature is compatible with all overloads.

---

### Real-World Cases

- **API client methods**: `get(url: string): Promise<Response>` or `get(url: string, options: Options): Promise<Response>`.
- **Event emitters**: consecutive overloads for specific event names with typed payloads.
- **Utility functions**: `len(x: string | any[]): number` instead of overloads.
- **Callback APIs**: `filterUsers(filter: (this: User) => boolean): User[]` with explicit `this` typing.

---

## References

- TypeScript Handbook — More on Functions: Function Overloads - https://www.typescriptlang.org/docs/handbook/2/functions.html#function-overloads
- TypeScript Handbook — More on Functions: Declaring `this` in a Function - https://www.typescriptlang.org/docs/handbook/2/functions.html#declaring-this-in-a-function
- TypeScript Handbook — More on Functions: Writing Good Overloads - https://www.typescriptlang.org/docs/handbook/2/functions.html#writing-good-overloads
- TypeScript Language Specification — Method Signatures (v2.3.3) - https://raw.githubusercontent.com/microsoft/TypeScript/refs/tags/v2.3.3/doc/spec.md
- TypeScript ESLint — `method-signature-style` Rule - https://app.unpkg.com/@typescript-eslint/eslint-plugin@3.0.2/files/docs/rules/method-signature-style.md
- Biome — `useConsistentMethodSignatures` Rule - https://biomejs.dev/linter/rules/use-consistent-method-signatures/
- Sentry — Node.js "No overload matches this call" Error with TypeScript - https://sentry.io/answers/node-js-no-overload-matches-this-call-error-with-typescript/
- CodeQL — Unreachable Method Overloads - https://codeql.github.com/
- Google TypeScript Style Guide — Method Signatures and Overloads - https://google.github.io/styleguide/tsguide.html
- TSConfig Reference — `strictFunctionTypes` - https://www.typescriptlang.org/tsconfig/strictFunctionTypes.html
- TSConfig Reference — `noImplicitThis` - https://www.typescriptlang.org/tsconfig/noImplicitThis.html
- TypeScript Handbook — Classes (`this` and Methods) - https://www.typescriptlang.org/docs/handbook/2/classes.html
- TypeScript 2.0 Release Notes — `this` Parameter - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html
- Microsoft/TypeScript PR #18654 — `strictFunctionTypes` and Method Bivariance - https://github.com/microsoft/TypeScript/pull/18654