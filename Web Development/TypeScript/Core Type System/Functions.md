# Functions in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **function** in TypeScript is a **callable value** that maps a tuple of input values (arguments) to an output value (return value), optionally producing side effects. TypeScript adds a **static type layer** over JavaScript functions, allowing the **parameter types**, **return type**, **arity**, and **calling convention** (call vs. construct) to be checked at compile time. Functions are **first-class values**: they can be assigned to variables, passed as arguments, returned from other functions, and stored in data structures.

**Technical Definition.** A function’s type consists of two parts: the **parameter types** and the **return type**. TypeScript provides **function type expressions** (`(p: T) => R`), **call signatures** (`{ (p: T): R }`) for callable objects with properties, and **construct signatures** (`{ new (p: T): R }`) for constructor values. Functions support **optional parameters** (`?`), **default parameters**, **rest parameters** (`...args: T[]`), **generics** (`<T>`), and **overloads** (multiple call signatures followed by an implementation). TypeScript’s function type checking is **structurally** based and is **erased** at compile time; the emitted JavaScript contains standard JavaScript functions.

**Beginner-Friendly Explanation.** A function is **a reusable block of code that takes inputs and produces an output**. TypeScript lets you **describe** what inputs a function accepts and what it returns, so the compiler can catch mistakes before you run the code. You can write functions in several styles—declarations, expressions, and arrow functions—and you can mark parameters as optional, give them defaults, or accept a variable number of arguments. When a function needs to behave differently depending on what it receives, you can write **overloads** to describe each calling form.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **First-class values** | Functions can be assigned, passed, and returned like any other value. |
| **Type = parameters + return** | A function’s type is determined by its parameter types and return type. |
| **Structural compatibility** | Functions are compatible if their parameter and return types are structurally compatible. |
| **Parameter variance** | Under `strictFunctionTypes`, function type parameters are contravariant; method parameters remain bivariant. |
| **Arity flexibility** | Functions with fewer parameters are assignable to types expecting more (extra arguments are ignored). |
| **Overload resolution** | Overloaded functions resolve calls against their overload signatures, not the implementation signature. |
| **`this` context** | `this` can be typed explicitly with a fake first parameter; arrow functions capture lexical `this`. |
| **Erasure** | All TypeScript-specific function annotations are erased and emit no JavaScript. |

---

### Prerequisites

- **JavaScript functions**: declarations, expressions, arrow functions, and `this`.
- **TypeScript basics**: type annotations, type aliases, and interfaces.
- **Primitive and object types**: `string`, `number`, `boolean`, and object shapes.
- **Generics fundamentals**: type parameters and constraints.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Functional programming**: higher-order functions, closures, and currying.
- **Callback APIs**: event handlers, array methods (`map`, `filter`, `reduce`), and asynchronous callbacks.
- **Class methods** and `this` binding.
- **Function overloads** and signature resolution.
- **Type-level programming**: conditional types, `Parameters<T>`, and `ReturnType<T>`.

---

## Core Concepts / Features

---

## 1. Function Types

### Definitions

**Core Definition.** A **function type** describes the **shape** of a function: the types of its parameters and the type of its return value. It is written using a **function type expression** (`(p: T) => R`) or a **call signature** within an object type.

**Technical Definition.** The function type expression `(a: string) => void` means “a function with one parameter, named `a`, of type `string`, that returns nothing.” Parameter names are **required** in function type expressions. A function type expression is semantically a **call signature** with no additional properties. When a function has properties, a **call signature** must be used inside an object type. Function types are **erased** at compile time.

**Beginner-Friendly Explanation.** A function type is **a description of what a function looks like**—what it takes and what it returns. `(name: string) => string` means “a function that takes a string and returns a string.” You use it to type variables, parameters, and properties that hold functions.

### Purposes

- **To name the contract of a callback or function value** so the compiler can check calls.
- **To enable higher-order functions** that accept or return functions with a known shape.
- **To document the intended calling convention** of an API.
- **To support structural compatibility** checks between different function values.

### Syntax Rules and Structure

**General syntax (function type expression):**

```
type Fn = (param1: Type1, param2: Type2) => ReturnType;
```

**General syntax (call signature in an object type):**

```
type CallableWithProps = {
  (param: Type): ReturnType;
  property: Type;
};
```

**Component breakdown:**

- `(param1: Type1, param2: Type2)` — the parameter list with **named** parameters and types.
- `=>` — the return type separator (in function type expressions).
- `ReturnType` — the return type.
- In call signatures, `:` replaces `=>` between the parameter list and return type.

**Syntax rules.**

- Parameter names are **required** in function type expressions: `(string) => void` is a parameter named `string` of type `any`, not a string parameter.
- Parameter types may be omitted, in which case they are implicitly `any` (unless `noImplicitAny` is enabled).
- Return type may be omitted and inferred in declarations, but function type expressions require it.
- Call signatures are used when the callable also has properties.
- Function types may be generic: `<T>(arg: T) => T`.

**Constraints and limitations.**

- Function type expressions cannot describe callable objects with properties; use call signatures.
- Function type expressions are erased and provide no runtime reflection.
- Parameter names in function type expressions are documentation-only; they do not affect compatibility.

### Annotated Complete Step by Step Code Examples

#### Example 1: Function type expression

```typescript
// example_function_type.ts
// Step 1: Declare a function type.
type GreetFunction = (name: string) => string;

// Step 2: Assign a compatible function.
const greet: GreetFunction = (name) => `Hello, ${name}`;

// Step 3: Call the function.
console.log(greet("Ada")); // Expected: Hello, Ada

// ❌ Wrong parameter type.
// const bad: GreetFunction = (name: number) => `Hello, ${name}`;
// Error: number not assignable to string.
```

**Setup.** Compile with `npx tsc example_function_type.ts --strict`.

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The function type expression declares a parameter of type `string` and a return type of `string`. The assigned arrow function satisfies this contract. A function with a `number` parameter is rejected.

---

#### Example 2: Call signature for a callable with properties

```typescript
// example_call_signature.ts
// Step 1: Declare a callable object type with a property.
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};

// Step 2: Create a function and attach the property.
function myFunc(someArg: number): boolean {
  return someArg > 3;
}
myFunc.description = "default description";

// Step 3: Use it through the callable type.
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}

doSomething(myFunc); // Expected: default description returned true
```

**Expected Output:**
```
default description returned true
```

**Why this output occurs.** The call signature `(someArg: number): boolean` describes the callable part, and `description: string` describes the property. The function satisfies both. A pure function type expression cannot describe the `description` property.

---

### Real-World Cases

- **Callback types**: `type Callback = (error: Error | null, data: T) => void`.
- **Event handlers**: `type Handler = (event: MouseEvent) => void`.
- **Higher-order functions**: `function map<T, U>(arr: T[], fn: (item: T) => U): U[]`.

---

## 2. Function Declarations

### Definitions

**Core Definition.** A **function declaration** is a named function statement introduced with the `function` keyword. It is **hoisted** within its scope and can be called before its declaration appears.

**Technical Definition.** A function declaration has the form `function name(params): ReturnType { ... }`. It creates a named binding in the enclosing scope. TypeScript checks the parameter and return types. Function declarations are **hoisted** to the top of their containing scope and are available before their textual position.

**Beginner-Friendly Explanation.** A function declaration is **the normal way you write a named function**. You write `function add(x: number, y: number) { ... }` and can call `add` anywhere in the same scope, even before the declaration line.

### Purposes

- **To create a named, hoisted function** that can be called from anywhere in its scope.
- **To define the primary API of a module** with a clear, stable name.
- **To support recursion** by allowing a function to refer to itself by name.

### Syntax Rules and Structure

**General syntax:**

```
function functionName(param1: Type1, param2: Type2): ReturnType {
  // body
}
```

**Component breakdown:**

- `function` — declaration keyword.
- `functionName` — the function name.
- `(param1: Type1, param2: Type2)` — typed parameters.
- `: ReturnType` — optional return type annotation.
- `{ ... }` — function body.

**Syntax rules.**

- Function declarations are **hoisted**.
- Return type may be inferred from the body.
- Parameters may have optional, default, and rest forms.
- Function declarations may be generic.
- Function declarations may be overloaded (multiple signatures above the implementation).

**Constraints and limitations.**

- Function declarations are hoisted, which can lead to surprising behavior if a function is called before its dependencies are initialized.
- Function declarations cannot be conditionally redeclared in the same scope.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic function declaration

```typescript
// example_function_declaration.ts
// Step 1: Declare a function.
function add(x: number, y: number): number {
  return x + y;
}

// Step 2: Call it (hoisted).
console.log(add(2, 3)); // Expected: 5
```

**Expected Output:**
```
5
```

**Why this output occurs.** The function declaration is hoisted, so `add` is available at the call site. The parameter and return types are checked.

---

### Real-World Cases

- **Utility functions**: `function formatDate(date: Date): string`.
- **Module APIs**: `export function createUser(input: CreateUserInput): User`.
- **Recursive algorithms**: `function factorial(n: number): number`.

---

## 3. Function Expressions

### Definitions

**Core Definition.** A **function expression** is a function value created as part of an expression, typically assigned to a variable or passed as an argument. It may be **named** or **anonymous**.

**Technical Definition.** A function expression has the form `const f = function (params): ReturnType { ... }` or `const f = function name(params): ReturnType { ... }`. It is **not hoisted**; the variable is initialized when the expression is evaluated. The optional internal name is scoped to the function body only. TypeScript checks the function expression against its contextual type.

**Beginner-Friendly Explanation.** A function expression is **a function written as a value** and assigned to a variable. Unlike declarations, it is not hoisted, so you cannot call it before the assignment line.

### Purposes

- **To create a function as a value** for assignment or passing as an argument.
- **To control hoisting behavior** by deferring availability until the expression is evaluated.
- **To name the function internally** for recursion or debugging while keeping the outer binding separate.

### Syntax Rules and Structure

**General syntax (anonymous):**

```
const fn = function (param: Type): ReturnType { ... };
```

**General syntax (named):**

```
const fn = function internalName(param: Type): ReturnType { ... };
```

**Component breakdown:**

- `const fn` — the variable binding.
- `function` — the function expression keyword.
- `internalName` (optional) — a name scoped to the function body.
- `(param: Type)` — parameters.
- `: ReturnType` — return type.

**Syntax rules.**

- Function expressions are **not hoisted**.
- The internal name is scoped to the function body.
- They may be generic.
- They may be used as arguments to higher-order functions.

**Constraints and limitations.**

- Cannot be called before the assignment is evaluated.
- Anonymous function expressions have no internal name for recursion (use the outer variable).

### Annotated Complete Step by Step Code Examples

#### Example 1: Named function expression

```typescript
// example_function_expression.ts
const factorial = function fact(n: number): number {
  return n <= 1 ? 1 : n * fact(n - 1);
};

console.log(factorial(5)); // Expected: 120

// fact(3); // Error: 'fact' is not defined in the outer scope.
```

**Expected Output:**
```
120
```

**Why this output occurs.** The internal name `fact` is available inside the function body for recursion, but not in the outer scope. The variable `factorial` is the only outer reference.

---

### Real-World Cases

- **Callbacks**: `array.map(function (item) { return item * 2; })`.
- **Configuration values**: `const handler = function onEvent(e: Event) { ... }`.
- **Conditional function definitions**: assigning different function expressions based on runtime conditions.

---

## 4. Arrow Functions

### Definitions

**Core Definition.** An **arrow function** is a concise function expression using `=>` syntax. Arrow functions **capture `this` lexically** from the surrounding scope and cannot have their own `this` binding.

**Technical Definition.** An arrow function has the form `(params) => expression` or `(params) => { statements }`. TypeScript checks arrow functions against their contextual type, inferring parameter types from context when possible. Arrow functions **cannot** have a `this` parameter and **cannot** be constructors. Their `this` is permanently bound to the enclosing lexical scope.

**Beginner-Friendly Explanation.** An arrow function is **a shorter way to write a function**. It looks like `(x) => x * 2`. The key difference is that arrow functions **do not have their own `this`**—they use the `this` from where they are defined. This makes them ideal for callbacks where you want to preserve the outer `this`.

### Purposes

- **To write concise function expressions**.
- **To preserve lexical `this`** in callbacks and event handlers.
- **To enable contextual typing** of parameters from the expected function type.
- **To avoid the dynamic `this` binding** of regular functions.

### Syntax Rules and Structure

**General syntax:**

```
const fn = (param: Type): ReturnType => expression;
const fn = (param: Type): ReturnType => { statements };
```

**Component breakdown:**

- `(param: Type)` — parameters.
- `=>` — arrow.
- `expression` or `{ statements }` — the body.
- `: ReturnType` — optional return type.

**Syntax rules.**

- Arrow functions **capture `this` lexically**.
- They **cannot** have a `this` parameter.
- They **cannot** be used with `new`.
- They may be generic: `<T>(arg: T): T => arg`.
- Single-expression bodies may omit `{}` and `return`.

**Constraints and limitations.**

- Arrow functions cannot be constructors.
- They cannot be used where a `this` binding is required.
- Named arrow functions are not possible; use a `const` variable for a name.

### Annotated Complete Step by Step Code Examples

#### Example 1: Lexical `this` in a callback

```typescript
// example_arrow_this.ts
class Counter {
  count = 0;

  // ✅ Arrow function preserves `this` from the class instance.
  increment = (): void => {
    this.count++;
  };
}

const counter = new Counter();
const inc = counter.increment;
inc();
console.log(counter.count); // Expected: 1
```

**Expected Output:**
```
1
```

**Why this output occurs.** The arrow function captures `this` from the `Counter` instance at definition time. Even when detached and called as `inc()`, it still refers to the original instance.

---

#### Example 2: Contextual typing of arrow parameters

```typescript
// example_arrow_contextual.ts
const names = ["Ada", "Grace", "Alan"];

// names is string[], so name is inferred as string.
const lengths = names.map((name) => name.length);

console.log(lengths); // Expected: [ 3, 5, 4 ]
```

**Expected Output:**
```
[ 3, 5, 4 ]
```

**Why this output occurs.** The array method `map` provides a contextual type `(value: string, index: number, array: string[]) => U`. The arrow parameter `name` is inferred as `string` from this context.

---

### Real-World Cases

- **Array callbacks**: `items.map((item) => item.id)`.
- **React event handlers**: `onClick={() => setCount(c => c + 1)}`.
- **Preserving `this` in class methods**: `method = () => { this.property++; }`.

---

## 5. Parameters

### Definitions

**Core Definition.** **Parameters** are the named inputs declared in a function signature. TypeScript allows parameters to be **required**, **optional** (`?`), **defaulted** (`= value`), or **rest** (`...args`).

**Technical Definition.** Required parameters must be supplied by the caller. Optional parameters (`param?: Type`) may be omitted and are typed as `Type | undefined`. Default parameters (`param: Type = value`) are optional and default to the specified value when `undefined` is passed. Rest parameters (`...args: Type[]`) collect any number of trailing arguments into an array. Optional and default parameters must follow required parameters.

**Beginner-Friendly Explanation.** Parameters are **the inputs a function accepts**. You can mark some as optional, give them default values, or allow a variable number of arguments with the rest syntax.

### Purposes

- **To declare the input contract** of a function.
- **To support flexible calling conventions** with optional and default values.
- **To accept variable numbers of arguments** with rest parameters.
- **To enable type checking** of arguments at every call site.

### Syntax Rules and Structure

**General syntax:**

```
function f(required: Type, optional?: Type, defaulted: Type = value, ...rest: Type[]) { ... }
```

**Component breakdown:**

- `required: Type` — must be supplied.
- `optional?: Type` — may be omitted; type is `Type | undefined`.
- `defaulted: Type = value` — optional; defaults to `value`.
- `...rest: Type[]` — collects trailing arguments into an array.

**Syntax rules.**

- Required parameters must precede optional and default parameters.
- Rest parameters must be last.
- Optional parameters may be combined with defaults (defaults take precedence).
- Default values are evaluated at call time, not at definition time.

**Constraints and limitations.**

- Optional parameters are not the same as default parameters; optional parameters are `undefined` when omitted, while defaults supply a value.
- Rest parameters must be an array type.

### Annotated Complete Step by Step Code Examples

#### Example 1: Required, optional, and default parameters

```typescript
// example_parameters.ts
function greet(name: string, greeting: string = "Hello", punctuation?: string): string {
  return `${greeting}, ${name}${punctuation ?? "!"}`;
}

console.log(greet("Ada"));                   // Expected: Hello, Ada!
console.log(greet("Ada", "Hi"));             // Expected: Hi, Ada!
console.log(greet("Ada", "Hey", "?"));       // Expected: Hey, Ada?
```

**Expected Output:**
```
Hello, Ada!
Hi, Ada!
Hey, Ada?
```

**Why this output occurs.** `name` is required. `greeting` has a default. `punctuation` is optional and defaults via `??` in the body. The compiler checks each call against the parameter types.

---

### Real-World Cases

- **Option bags**: `function createUser(name: string, options?: CreateOptions)`.
- **Configurable functions**: `function log(message: string, level: LogLevel = "info")`.
- **Math utilities**: `function round(n: number, digits: number = 0)`.

---

## 6. Return Types

### Definitions

**Core Definition.** The **return type** of a function is the type of the value it produces. It may be **explicitly annotated** or **inferred** by the compiler from the function body.

**Technical Definition.** A return type annotation appears after the parameter list: `function f(): ReturnType { ... }`. If omitted, TypeScript infers the return type from the return statements. Functions that do not return a value have return type `void`. Functions that never return (throw or infinite loop) have return type `never`. Return type inference works on generic functions and propagates through overloads.

**Beginner-Friendly Explanation.** The return type is **what the function gives back**. You can write it explicitly (`: number`) or let TypeScript figure it out from the `return` statements. If a function returns nothing, its type is `void`.

### Purposes

- **To document the output contract** of a function.
- **To catch return type mismatches** at compile time.
- **To enable return type inference** for concise code.
- **To distinguish `void` and `never`** for functions that do not return or never return.

### Syntax Rules and Structure

**General syntax:**

```
function f(params): ReturnType { ... }
const f = (params): ReturnType => ...;
```

**Component breakdown:**

- `: ReturnType` — the annotation, placed after the parameter list.

**Syntax rules.**

- Return types may be inferred from the body.
- `void` indicates no useful return value.
- `never` indicates the function never returns (throws or infinite loop).
- Generic return types may relate to parameter types: `function first<T>(arr: T[]): T | undefined`.

**Constraints and limitations.**

- Inference may produce a wider type than desired; annotate to narrow.
- `void` functions can still return a value; the value is ignored by callers.

### Annotated Complete Step by Step Code Examples

#### Example 1: Inferred vs. explicit return type

```typescript
// example_return_type.ts
// Inferred: number
function add(x: number, y: number) {
  return x + y;
}

// Explicit: number
function subtract(x: number, y: number): number {
  return x - y;
}

// void: no return value
function log(message: string): void {
  console.log(message);
}

// never: always throws
function fail(message: string): never {
  throw new Error(message);
}

console.log(add(2, 3)); // Expected: 5
console.log(subtract(5, 2)); // Expected: 3
```

**Expected Output:**
```
5
3
```

**Why this output occurs.** `add` infers `number` from `x + y`. `subtract` explicitly annotates `number`. `log` returns `void`. `fail` returns `never` because it always throws.

---

### Real-World Cases

- **API functions**: `function getUser(id: string): Promise<User>`.
- **Validation functions**: `function isValid(input: unknown): input is User`.
- **Assertion functions**: `function assert(value: unknown): asserts value`.

---

## 7. Optional Parameters

### Definitions

**Core Definition.** An **optional parameter** is a parameter marked with `?` after its name, indicating that the caller may **omit** it. Its type includes `undefined` when omitted.

**Technical Definition.** The syntax `param?: Type` makes the parameter optional. Optional parameters must follow required parameters. Reading an optional parameter yields `Type | undefined`. Under `exactOptionalPropertyTypes`, the distinction between omitted and explicitly `undefined` is preserved.

**Beginner-Friendly Explanation.** An optional parameter is **an input you do not have to provide**. `function greet(name: string, title?: string)` can be called with just `name`, or with both `name` and `title`.

### Purposes

- **To support call sites that do not need every parameter**.
- **To model APIs with optional configuration**.
- **To reduce the burden on callers** when a parameter is often omitted.

### Syntax Rules and Structure

**General syntax:**

```
function f(required: Type, optional?: Type) { ... }
```

**Component breakdown:**

- `optional?: Type` — the optional parameter.

**Syntax rules.**

- Optional parameters must follow required parameters.
- They may be combined with defaults.
- Reading yields `Type | undefined`.

**Constraints and limitations.**

- Optional parameters do not enforce “at least one of a group.”
- Callers may pass `undefined` explicitly.

### Annotated Complete Step by Step Code Examples

#### Example 1: Optional parameter

```typescript
// example_optional_param.ts
function buildName(first: string, last?: string): string {
  return last ? `${first} ${last}` : first;
}

console.log(buildName("Ada"));          // Expected: Ada
console.log(buildName("Ada", "Lovelace")); // Expected: Ada Lovelace
```

**Expected Output:**
```
Ada
Ada Lovelace
```

**Why this output occurs.** `last` is optional and typed as `string | undefined`. The ternary check narrows it before use.

---

### Real-World Cases

- **Option bags**: `function configure(options?: Config)`.
- **Logging**: `function log(message: string, level?: LogLevel)`.

---

## 8. Default Parameters

### Definitions

**Core Definition.** A **default parameter** is a parameter with a default value (`param: Type = value`). When the argument is `undefined` or omitted, the default value is used.

**Technical Definition.** Default parameters are evaluated **at call time**, not at definition time. They are optional and have the declared type. A default parameter may be followed by required parameters, but the required parameters cannot be supplied without providing the defaulted one (or passing `undefined`).

**Beginner-Friendly Explanation.** A default parameter is **an input with a backup value**. If the caller does not provide it, the default is used automatically.

### Purposes

- **To provide sensible fallback values** for omitted arguments.
- **To reduce the need for explicit `undefined` checks** in the body.
- **To simplify function signatures** by combining optionality and a value.

### Syntax Rules and Structure

**General syntax:**

```
function f(param: Type = value) { ... }
```

**Component breakdown:**

- `param: Type = value` — the default parameter.

**Syntax rules.**

- Default values are evaluated at call time.
- Default parameters are optional.
- They may precede required parameters, but callers must supply the required ones explicitly.

**Constraints and limitations.**

- Default values cannot reference other parameters declared later.
- Default expressions are evaluated on every call where the argument is `undefined`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Default parameter

```typescript
// example_default_param.ts
function greet(name: string, greeting: string = "Hello"): string {
  return `${greeting}, ${name}`;
}

console.log(greet("Ada"));       // Expected: Hello, Ada
console.log(greet("Ada", "Hi")); // Expected: Hi, Ada
```

**Expected Output:**
```
Hello, Ada
Hi, Ada
```

**Why this output occurs.** `greeting` defaults to `"Hello"` when omitted. When supplied, the argument overrides the default.

---

### Real-World Cases

- **Configuration defaults**: `function createServer(port: number = 3000)`.
- **Formatting**: `function format(n: number, digits: number = 2)`.

---

## 9. Rest Parameters

### Definitions

**Core Definition.** A **rest parameter** (`...args: Type[]`) collects **any number of trailing arguments** into an array. It must be the **last** parameter.

**Technical Definition.** The rest parameter receives all arguments from its position onward as an array. TypeScript checks that each argument is assignable to the array’s element type. Rest parameters may be combined with required, optional, and default parameters, provided the rest parameter is last.

**Beginner-Friendly Explanation.** A rest parameter is **a way to accept a variable number of arguments**. `function sum(...numbers: number[])` can be called with any number of number arguments.

### Purposes

- **To accept variable-length argument lists**.
- **To forward arguments** to another function.
- **To implement variadic utilities** like `sum`, `max`, and `log`.

### Syntax Rules and Structure

**General syntax:**

```
function f(...args: Type[]) { ... }
```

**Component breakdown:**

- `...args: Type[]` — the rest parameter.

**Syntax rules.**

- Must be last in the parameter list.
- Type must be an array type.
- May be combined with other parameter forms, as long as it is last.

**Constraints and limitations.**

- Only one rest parameter is allowed per function.
- The rest parameter is always an array, even when zero arguments are passed.

### Annotated Complete Step by Step Code Examples

#### Example 1: Rest parameter for summation

```typescript
// example_rest_param.ts
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum());          // Expected: 0
console.log(sum(1, 2, 3));   // Expected: 6
console.log(sum(10, 20));    // Expected: 30
```

**Expected Output:**
```
0
6
30
```

**Why this output occurs.** The rest parameter collects all arguments into `numbers`. `reduce` sums them. With no arguments, the array is empty, and the initial value `0` is returned.

---

### Real-World Cases

- **Math utilities**: `function max(...values: number[])`.
- **Logging**: `function log(...messages: string[])`.
- **Decorators and wrappers**: forwarding arguments with `fn(...args)`.

---

## 10. Function Overloads

### Definitions

**Core Definition.** **Function overloads** are multiple **overload signatures** (without bodies) followed by a single **implementation signature** (with a body). They allow a function to be called in different ways while maintaining precise type checking for each call form.

**Technical Definition.** Overload signatures must appear **consecutively** above the implementation. The implementation signature must be **compatible with all overloads** but is **not visible from the outside**—callers can only use the overload signatures. The compiler resolves a call to a **single overload** based on the argument types; it cannot resolve calls that match only the implementation signature or that require a union of overloads. The TypeScript Handbook recommends **preferring union types over overloads when possible**.

**Beginner-Friendly Explanation.** Function overloads let you **declare several “menu options” for how a function can be called**, then write one implementation that handles all of them. Callers only see the menu options. If you find yourself writing overloads with the same parameter count and return type, you can usually replace them with a union type.

### Purposes

- **To support multiple calling conventions** for a single function with precise type checking.
- **To provide rich IntelliSense and documentation** for each call form.
- **To model JavaScript APIs** with genuinely different argument shapes.
- **To maintain backward compatibility** while adding new call forms.

### Syntax Rules and Structure

**General syntax:**

```
function f(a: Type1): ReturnType1;
function f(a: Type1, b: Type2): ReturnType2;
function f(...args: any[]): any {
  // implementation
}
```

**Component breakdown:**

- **Overload signatures** — one or more declarations without bodies.
- **Implementation signature** — a function with a body that is compatible with all overloads.

**Syntax rules.**

- Overload signatures must be **consecutive** above the implementation.
- The implementation signature must be **compatible** with all overloads.
- Overloads must either **all be required** or **all be optional**—mixing is not allowed.
- The order matters: more specific signatures should generally come first.
- Use `function` declarations, not arrow functions, for overloads.

**Constraints and limitations.**

- The implementation signature is **not callable**; only overloads are visible.
- The compiler cannot resolve calls that require a **union of overloads**.
- Overloads with identical parameter types and return types are **redundant**.
- **Prefer union types over overloads** when the signature can be expressed with a union.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic overload

```typescript
// example_overload_basic.ts
// Step 1: Overload signatures (no bodies).
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;

// Step 2: Implementation signature.
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}

// Step 3: Calls.
const d1 = makeDate(12345678); // ✅ One argument
const d2 = makeDate(5, 5, 5);  // ✅ Three arguments

console.log(d1 instanceof Date); // Expected: true
console.log(d2 instanceof Date); // Expected: true

// makeDate(1, 3); // ❌ Error: No overload expects 2 arguments.
```

**Expected Output:**
```
true
true
```

**Why this output occurs.** The overloads define valid call forms: one or three arguments. The implementation accepts one required and two optional parameters, but **cannot be called with two arguments** because no overload permits it.

---

#### Example 2: Union type preferred over overloads

```typescript
// example_union_over_overload.ts
// ❌ Redundant overloads.
// function len(s: string): number;
// function len(arr: any[]): number;

// ✅ Union version: simpler and supports union-typed calls.
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

**Why this output occurs.** The union version is simpler and, unlike the overloaded version, **supports calls with a union-typed argument**. The TypeScript Handbook explicitly recommends union types over overloads when possible.

---

### Real-World Cases

- **Date construction**: `makeDate(timestamp)` or `makeDate(m, d, y)`.
- **Array methods**: `concat` with different argument shapes.
- **Event emitters**: `emit(event: string)` or `emit(event: string, payload: T)`.

---

## 11. Function Type Best Practices

### Definitions

**Core Definition.** **Function type best practices** are guidelines for writing and typing functions in TypeScript in ways that maximize **type safety, clarity, and maintainability**, while avoiding common pitfalls such as `any` leakage, unnecessary overloads, and implicit `any` parameters.

**Technical Definition.** Best practices center on **avoiding `any`** and using `unknown` with narrowing for untyped data, **annotating return types on public APIs**, **preferring union types over overloads when possible**, **using the fewest type parameters necessary**, **typing `this` explicitly in callbacks**, and **enabling `strict` mode** to catch implicit `any` and nullability issues.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep functions safe and readable. The big rules are: do not use `any` (use `unknown` and narrow); annotate return types on public functions; prefer union types over overloads; and do not add unnecessary type parameters.

### Purposes

- **To maximize type safety** by avoiding `any` and implicit `any`.
- **To make public APIs clear and stable** with explicit return types.
- **To reduce complexity** by preferring union types over overloads.
- **To catch binding errors** with explicit `this` typing.
- **To enforce strictness** with `strict` mode and lint rules.

### Syntax Rules and Structure

Best practices are conventions that interact with function syntax. The core rules are:

1. **Never use `any`.** Use `unknown` for untyped data and narrow with type guards. Accept `any` only at third-party integration boundaries with a documented comment.
2. **Use `unknown` over `any` for untyped input.** `unknown` forces narrowing before use.
3. **Annotate return types on public functions.** Explicit return types document the contract and prevent accidental widening.
4. **Prefer union types over overloads when possible.** Union versions support union-typed calls and are simpler.
5. **Use as few type parameters as possible.** If a type parameter appears only once, reconsider whether it is needed.
6. **Type `this` explicitly in callbacks.** Use `function (this: Type) { ... }` to catch binding errors.
7. **Enable `strict` mode.** `strict` catches implicit `any`, nullability bugs, and unsafe `this`.

**Component breakdown (function typing decision tree):**

```
Does the function accept untyped data?
  ├── Yes → Use unknown, then narrow.
  └── No → Does the function have multiple call forms?
           ├── Yes → Can a union type express them?
           │        ├── Yes → Use a union type.
           │        └── No → Use overloads (consecutive, specific-first).
           └── No → Use a single signature with precise parameter and return types.
```

**Constraints and limitations.**

- `unknown` requires narrowing and may add verbosity; use it deliberately.
- Explicit return types may duplicate inference; annotate public APIs, but allow inference for simple local functions.
- Overloads are still necessary for genuinely different call shapes; do not force unions where they do not fit.

### Annotated Complete Step by Step Code Examples

#### Example 1: `unknown` instead of `any`

```typescript
// example_best_unknown.ts
// ❌ `any` disables safety.
// function parse(input: any) { return input.value; }

// ✅ `unknown` forces narrowing.
function parse(input: unknown): string | undefined {
  if (typeof input === "object" && input !== null && "value" in input) {
    return String((input as { value: unknown }).value);
  }
  return undefined;
}

console.log(parse({ value: 42 })); // Expected: 42
console.log(parse("not an object")); // Expected: undefined
```

**Expected Output:**
```
42
undefined
```

**Why this output occurs.** `unknown` requires a runtime check before accessing properties. The `typeof` and `in` checks narrow `input` to an object with a `value` property, making the access safe.

---

#### Example 2: Union type over overloads

```typescript
// example_best_union.ts
// ❌ Redundant overloads.
// function len(s: string): number;
// function len(arr: any[]): number;

// ✅ Union version.
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

**Why this output occurs.** The union version accepts a `string | any[]` argument, while the overloaded version would reject it because the compiler cannot resolve it to a single overload.

---

### Real-World Cases

- **API client functions**: `function request(url: string, body: unknown): Promise<unknown>`.
- **Callbacks**: `function onEvent(this: HTMLElement, event: Event)`.
- **Utility functions**: `function first<T>(arr: T[]): T | undefined`.

---

## References

- TypeScript Handbook — More on Functions (Function Types, Call Signatures, Construct Signatures, Generics, Optional Parameters, Overloads, `this`) - https://www.typescriptlang.org/docs/handbook/2/functions.html
- TypeScript Handbook — Functions (Legacy) - https://www.typescriptlang.org/docs/handbook/functions.html
- TypeScript Handbook — Do's and Don'ts (Optional Parameters, `void` Return) - https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html
- TypeScript Handbook — Declaration Reference: Function Module (Overloads) - https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-function-d-ts.html
- TypeScript 3.3 Release Notes — Improved Behavior for Calling Union Types - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-3.html
- TypeScript ESLint — `no-explicit-any` Rule - https://typescript-eslint.io/rules/no-explicit-any/
- TypeScript ESLint — `explicit-function-return-type` Rule - https://typescript-eslint.io/rules/explicit-function-return-type/
- GitHub — TypeScript Strict Rules (Avoid `any`, Use `unknown`, Explicit Return Types) - https://github.com/0xMassi/claude-skills/blob/main/typescript-strict/SKILL.md
- GitHub — TypeScript Typing Rules (Avoid `any`) - https://github.com/slior/dialectic/blob/7afcc4ca/.cursor/rules/typescript-typing.mdc
- MDN — Arrow Function Expressions (`this` Binding) - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions
- MDN — Default Parameters - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters
- MDN — Rest Parameters - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters