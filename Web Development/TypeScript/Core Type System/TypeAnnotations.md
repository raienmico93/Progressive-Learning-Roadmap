## Type Annotations

This section provides a comprehensive reference for TypeScript **type annotations** — the explicit syntactic mechanism for associating types with variables, function parameters, return values, and object properties. Type annotations are optional throughout TypeScript; when omitted, the compiler relies on **type inference**. When present, annotations serve as documentation, enforced constraints, and tooling signals. A central design principle is that annotations are **erased** at compile time and have no runtime effect.

---

## 1. Type Annotations (General)

### Definitions

**Core Definition.** A **type annotation** is an explicit syntactic construct that associates a TypeScript **type** with a declaration — a variable, parameter, return value, or property — using the colon syntax `: Type`.

**Technical Definition.** Type annotations are **type-level expressions** placed in designated syntactic positions. The compiler parses them, uses them during type checking, and **erases** them during emission. Annotations participate in assignability checking, contextual typing, and control-flow narrowing. They are distinct from **type assertions** (`as T`), which override checking rather than declare an expected type.

**Beginner-Friendly Explanation.** A type annotation is **you telling TypeScript what type something should be**. Instead of letting the compiler guess, you write `: string` after a variable or `: number` before a function body’s opening brace. These annotations help catch mistakes early and make your intentions clear, but they disappear completely when the code runs.

### Purposes

- **To declare the intended type explicitly**, overriding or supplementing inference.
- **To document public interfaces** for functions, classes, and modules.
- **To enforce constraints at boundaries** where values enter typed code.
- **To improve editor tooling** by providing precise type information for autocompletion and diagnostics.
- **To serve as executable specification** that the compiler verifies.

### Syntax Rules and Structure

Type annotations appear in several syntactic positions. The general form is:

```
declarationName: TypeExpression
```

**Component breakdown:**

- `declarationName` — the identifier being annotated (variable, parameter, property).
- `:` — the **annotation separator** (not an operator; it introduces the type).
- `TypeExpression` — any valid TypeScript type: primitive (`string`, `number`), object type, union (`A | B`), interface name, generic instantiation (`Array<string>`), function type (`(x: number) => string`), etc.

**Syntax rules.**

- A space is conventionally placed **after** the colon: `name: string`, not `name : string`.
- Annotations are **optional**; when omitted, inference determines the type.
- Annotations are **erased** and have no runtime representation.
- `import type` and `export type` guarantee that type-only imports are erased entirely.

**Constraints and limitations.**

- An annotation does **not** perform runtime validation; values can still enter via `any`, assertions, or untyped JavaScript.
- Over-annotating can duplicate information already available through inference and reduce flexibility.
- Some positions (e.g., object literal properties in expression context) do not accept annotations directly.

### Annotated Complete Step by Step Code Examples

#### Example 1: Annotation across declaration categories

```typescript
// example_general.ts
// Step 1: Variable annotation.
let username: string = "Alice";                 // ': string' annotates username

// Step 2: Parameter and return annotations.
function greet(name: string): string {          // name: string; return: string
  return `Hello, ${name}`;
}

// Step 3: Property annotation in an interface.
interface User {
  name: string;                                 // property annotation
  age: number;
}

// Step 4: Use the annotated constructs.
const user: User = { name: username, age: 30 };
console.log(greet(user.name));                  // Expected: Hello, Alice
console.log(user.age);                          // Expected: 30
```

**Expected Output:**
```
Hello, Alice
30
```

**Why this output occurs.** The annotations constrain `username` to `string`, `greet` to accept and return `string`, and `User` to have `name: string` and `age: number`. All assignments satisfy these constraints, so the program compiles and runs. The emitted JavaScript contains only the values and operations; the annotations are removed.

#### Example 2: Annotation vs inference

```typescript
// example_annotation_vs_inference.ts
// Step 1: Inferred type (no annotation).
let inferred = "hello";          // inferred: string

// Step 2: Annotated type.
let annotated: "hello" = "hello"; // literal type "hello"

// Step 3: Reassignment behavior.
inferred = "world";              // allowed (string)
// annotated = "world";          // Error: Type '"world"' is not assignable to type '"hello"'.

console.log(inferred);           // Expected: world
console.log(annotated);          // Expected: hello
```

**Expected Output:**
```
world
hello
```

**Why this output occurs.** `inferred` is widened to `string` because no annotation is present. `annotated` preserves the literal type `"hello"` because the annotation specifies it. The annotation **narrows** the type beyond what inference would produce, enforcing a stricter constraint.

### Real-World Cases

- **Public API signatures**: Exported functions annotate parameters and return types to provide stable contracts.
- **Configuration objects**: Interfaces annotate expected configuration shapes, enabling editor completion and error detection.
- **Boundary validation**: Values from external sources are annotated as `unknown` and narrowed with validators.

---

## 2. Variable Annotations

### Definitions

**Core Definition.** A **variable annotation** is a type annotation placed on a variable declaration introduced by `let`, `const`, or `var`.

**Technical Definition.** In a variable statement `let x: T = expr`, the annotation `T` determines the variable’s type. If no annotation is present, the type is inferred from the initializer, or `any` if no initializer and no contextual information exists. Under `--noImplicitAny`, implicit `any` in variable declarations is an error. When an annotation is present, the initializer must be assignable to it.

**Beginner-Friendly Explanation.** A variable annotation is **you labeling a variable with its type**. You write `let age: number = 30` to say “this variable holds a number.” It is useful when you want to be explicit or when the initializer alone would infer a different type than you intend.

### Purposes

- **To override inference** when the initializer’s inferred type is too wide or not what you want.
- **To declare a variable before assignment** without initializing it immediately.
- **To document the intended type** for readers and tooling.
- **To enforce constraints on subsequent assignments**.

### Syntax Rules and Structure

```
let variableName: TypeExpression = initializer;
const constantName: TypeExpression = initializer;
```

**Component breakdown:**

- `let` / `const` / `var` — declaration keyword.
- `variableName` — identifier.
- `: TypeExpression` — the annotation.
- `= initializer` — optional; must be assignable to `TypeExpression`.

**Syntax rules.**

- Annotations are **optional**; inference is preferred for trivial initializers.
- When an annotation is present, the initializer is checked against it; assigning a value of an incompatible type is an error.
- `const` declarations with literal initializers and annotations preserve the annotated type.
- A variable declaration annotation may not use `typeof` to reference the variable being declared.

**Constraints and limitations.**

- Annotating trivially inferred types (e.g., `const x: boolean = true`) is discouraged because it adds no value.
- An annotation without an initializer requires definite assignment analysis under `strictPropertyInitialization` (for class fields) or results in `any` in non-strict configurations.
- Variable annotations do not provide runtime validation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit annotation and reassignment checking

```typescript
// example_variable.ts
// Step 1: Annotate a variable.
let score: number = 100;

// Step 2: Reassignment is checked.
score = 200;           // OK
// score = "high";     // Error: Type 'string' is not assignable to type 'number'.

// Step 3: Log.
console.log(score);    // Expected: 200
```

**Expected Output:**
```
200
```

**Why this output occurs.** The annotation `number` constrains all assignments. `200` is a `number` and is accepted. `"high"` would be rejected at compile time because `string` is not assignable to `number`. The emitted JavaScript contains only the assignments and the log.

#### Example 2: Annotation enabling declaration before assignment

```typescript
// example_variable_deferred.ts
// Step 1: Declare with annotation, no initializer.
let userId: string;

// Step 2: Assign later.
userId = "user-42";

console.log(userId);   // Expected: user-42
```

**Expected Output:**
```
user-42
```

**Why this output occurs.** The annotation `: string` declares the type without requiring an initializer. The later assignment is checked against `string`. Without the annotation, `userId` would be implicitly `any` under non-strict settings, losing type safety.

#### Example 3: Discouraged redundant annotation

```typescript
// example_variable_redundant.ts
// Discouraged: trivial inference makes the annotation redundant.
const isActive: boolean = true;
// Preferred:
const isActiveInferred = true;

console.log(isActive);          // Expected: true
console.log(isActiveInferred);  // Expected: true
```

**Expected Output:**
```
true
true
```

**Why this output occurs.** Both declarations produce the same runtime behavior. The annotation `: boolean` adds no information because `true` already infers `boolean`. Style guides recommend omitting such redundant annotations.

### Real-World Cases

- **Deferred initialization**: Variables populated inside conditionals or loops are annotated to maintain type safety.
- **Configuration constants**: `const DEFAULT_TIMEOUT: number = 5000` documents the intended type even when inference would suffice.
- **Migration**: Annotating variables as specific types during JavaScript-to-TypeScript migration prevents accidental widening to `any`.

---

## 3. Function Annotations

### Definitions

**Core Definition.** **Function annotations** are type annotations applied to a function’s **parameters** and **return type**, collectively defining the function’s **signature**.

**Technical Definition.** A function’s type consists of its parameter types and its return type. Annotations on parameters and the return position form the function signature. When annotations are omitted, parameter types may be inferred from default values or contextual typing, and the return type is inferred from return expressions. Function annotations are erased at compile time; they do not affect runtime arity or behavior.

**Beginner-Friendly Explanation.** Function annotations describe **what goes into a function and what comes out**. You annotate each parameter with the type of value it expects, and you annotate the return type with what the function produces. Together, they let TypeScript check that callers provide the right arguments and that the function body returns the right result.

### Purposes

- **To define a checkable contract** between the function and its callers.
- **To catch argument and return-type mismatches** at compile time.
- **To document the function’s interface** for readers and tooling.
- **To enable contextual typing** for callbacks passed as arguments.
- **To support function overloading** through multiple annotated signatures.

### Syntax Rules and Structure

```
function functionName(param1: Type1, param2: Type2): ReturnType {
  // body
}
```

**Component breakdown:**

- `functionName` — the function identifier.
- `param1: Type1` — parameter annotation; the parameter accepts values of `Type1`.
- `): ReturnType` — return type annotation; the function must return a value assignable to `ReturnType`.
- `{ ... }` — function body.

For arrow functions:

```
const fn = (param: Type): ReturnType => expression;
```

**Syntax rules.**

- Parameter annotations appear after the parameter name, separated by `:`.
- Return type annotations appear after the parameter list, before `=>` or `{`.
- Optional parameters use `?:` (e.g., `name?: string`) and automatically include `undefined` in their type.
- Rest parameters use `...args: Type[]` and must be typed as an array.
- Default parameters may omit the annotation when the default value clearly determines the type.

**Constraints and limitations.**

- Function declarations with bodies that always throw infer `void` unless explicitly annotated `never`; function expressions infer `never` in the same situation.
- Parameter bivariance applies to methods under default settings; `strictFunctionTypes` changes parameter checking for function types.
- Annotations do not enforce runtime argument counts or types.

### Annotated Complete Step by Step Code Examples

#### Example 1: Annotated function declaration

```typescript
// example_function.ts
// Step 1: Annotate parameters and return type.
function add(a: number, b: number): number {
  return a + b;
}

// Step 2: Call correctly.
console.log(add(2, 3));   // Expected: 5

// Step 3: Uncomment to see a compile-time error.
// console.log(add("2", "3"));
// Error: Argument of type 'string' is not assignable to parameter of type 'number'.
```

**Expected Output:**
```
5
```

**Why this output occurs.** The annotations constrain `a` and `b` to `number` and the return to `number`. The call `add(2, 3)` satisfies the contract. The string call would be rejected at compile time because `string` is not assignable to `number`. At runtime, only the addition and log remain.

#### Example 2: Arrow function annotations

```typescript
// example_arrow.ts
// Step 1: Annotate an arrow function.
const multiply = (x: number, y: number): number => x * y;

console.log(multiply(4, 5));   // Expected: 20
```

**Expected Output:**
```
20
```

**Why this output occurs.** The arrow function’s parameters and return are annotated identically to a function declaration. The compiler checks that the body’s expression type is assignable to the annotated return type.

### Real-World Cases

- **API client functions**: `fetchUser(id: string): Promise<User>` documents the expected input and output.
- **Event handlers**: Callback signatures are annotated to ensure compatibility with the event system.
- **Utility libraries**: Exported functions annotate signatures to provide stable, documented contracts.

---

## 4. Return Type Annotations

### Definitions

**Core Definition.** A **return type annotation** specifies the type of value a function **returns**, placed after the parameter list and before the function body.

**Technical Definition.** The return type annotation constrains all `return` statements in the function body: each returned expression must be assignable to the annotated type. If the function has no reachable `return` with a value, the return type must be `void` (or `undefined`). The special types `void` and `never` have specific semantics: `void` indicates no meaningful return value; `never` indicates the function never returns normally.

**Beginner-Friendly Explanation.** A return type annotation tells TypeScript **what the function gives back**. If you write `: number`, the function must return a number on every path that returns. If it returns a string, TypeScript flags the error.

### Purposes

- **To enforce that the function returns the intended type** on all code paths.
- **To document the function’s output** for callers.
- **To catch accidental changes** to return values during refactoring.
- **To express `void` for side-effect-only functions** and `never` for functions that always throw.

### Syntax Rules and Structure

```
function functionName(params): ReturnType {
  // body
}
```

**Component breakdown:**

- `): ReturnType` — the annotation; the colon is immediately after the closing parenthesis of the parameter list.
- `ReturnType` — any type expression, including `void` and `never`.

**Syntax rules.**

- Every `return` statement with a value must return an expression assignable to `ReturnType`.
- A function annotated `void` may use a bare `return;` but may not return a value (except `undefined`).
- A function annotated `never` must not have any reachable `return`; it must throw or loop forever.
- Without an annotation, the return type is inferred from return expressions; for function declarations that always throw, inference produces `void` for backward compatibility.

**Constraints and limitations.**

- `void` is not the same as `never`: `void` functions return normally with no value; `never` functions never return at all.
- Explicitly annotating `undefined` as a return type is allowed but often redundant under `strictNullChecks`.
- Return type annotations do not perform runtime checks.

### Annotated Complete Step by Step Code Examples

#### Example 1: Standard return annotation

```typescript
// example_return.ts
// Step 1: Annotate the return type.
function formatPrice(amount: number): string {
  return `$${amount.toFixed(2)}`;
}

console.log(formatPrice(19.5));   // Expected: $19.50

// Uncommenting the next line triggers a compile-time error.
// function bad(): string { return 42; }
// Error: Type 'number' is not assignable to type 'string'.
```

**Expected Output:**
```
$19.50
```

**Why this output occurs.** The annotation `: string` requires the function to return a `string`. The template literal is a string, so the contract is satisfied. A function returning a `number` where `string` is annotated would be rejected at compile time.

#### Example 2: `void` return annotation

```typescript
// example_return_void.ts
// Step 1: Annotate void for a side-effect-only function.
function logMessage(message: string): void {
  console.log(`[LOG] ${message}`);
}

// Step 2: Call and ignore the (non-existent) return value.
logMessage("Done");   // Expected: [LOG] Done
```

**Expected Output:**
```
[LOG] Done
```

**Why this output occurs.** `void` documents that the function returns no meaningful value. The compiler does not require a `return` statement. Callers cannot use the result in an expression, which prevents accidental reliance on `undefined`.

#### Example 3: `never` return annotation

```typescript
// example_return_never.ts
// Step 1: Annotate never for a function that always throws.
function fail(message: string): never {
  throw new Error(message);
}

// Step 2: Call in a try/catch.
try {
  fail("Something went wrong");
} catch (e) {
  console.log((e as Error).message);   // Expected: Something went wrong
}
```

**Expected Output:**
```
Something went wrong
```

**Why this output occurs.** `never` tells the compiler that `fail` never returns normally. The function body throws, satisfying this contract. If the function had a reachable `return`, the annotation would be an error. `never` is useful for exhaustiveness checks and for satisfying return-type requirements in unreachable code paths.

### Real-World Cases

- **API responses**: `async function fetchUser(): Promise<User>` documents the resolved type.
- **Exhaustiveness checks**: `assertNever(value: never): never` ensures all union cases are handled.
- **Event emitters**: `emit(event: string): void` signals that the return value is not meaningful.

---

## 5. Parameter Annotations

### Definitions

**Core Definition.** A **parameter annotation** specifies the type of a function parameter, constraining the arguments that callers may pass.

**Technical Definition.** Parameter annotations appear after each parameter name. They determine the parameter’s type for assignability checking of arguments at call sites and for type checking within the function body. TypeScript supports required parameters, optional parameters (`?:`), default parameters, and rest parameters, each with specific annotation rules.

**Beginner-Friendly Explanation.** A parameter annotation tells TypeScript **what kind of value a function expects for each input**. If you annotate `name: string`, callers must pass a string. Optional parameters are marked with `?` and may be omitted; rest parameters collect multiple arguments into an array.

### Purposes

- **To enforce that callers provide the correct argument types**.
- **To enable type checking within the function body** based on parameter types.
- **To document the function’s expected inputs**.
- **To support optional and rest parameter patterns** with type safety.

### Syntax Rules and Structure

```
function fn(required: Type, optional?: Type, defaultValue: Type = value, ...rest: Type[]) { ... }
```

**Component breakdown:**

- `required: Type` — a required parameter; callers must pass an argument assignable to `Type`.
- `optional?: Type` — an optional parameter; its type is `Type | undefined`, and it may be omitted.
- `defaultValue: Type = value` — a default parameter; the annotation may be omitted if the default clearly determines the type.
- `...rest: Type[]` — a rest parameter; must be an array type, and it collects remaining arguments.

**Syntax rules.**

- Required parameters must precede optional and default parameters in the parameter list.
- Optional parameters automatically include `undefined` in their type.
- Rest parameters must be last and must be typed as an array.
- Parameter annotations are erased at compile time.

**Constraints and limitations.**

- An optional parameter cannot be followed by a required parameter unless the required parameter has a default value.
- Parameter annotations do not enforce argument count at runtime.
- Method parameters are bivariant under default settings; `strictFunctionTypes` changes checking for function-type positions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Required and optional parameters

```typescript
// example_params.ts
// Step 1: Define a function with required and optional parameters.
function buildName(first: string, last?: string): string {
  return last ? `${first} ${last}` : first;
}

// Step 2: Call with one and two arguments.
console.log(buildName("Ada"));           // Expected: Ada
console.log(buildName("Ada", "Lovelace")); // Expected: Ada Lovelace
```

**Expected Output:**
```
Ada
Ada Lovelace
```

**Why this output occurs.** `first` is required; `last` is optional (`string | undefined`). The first call omits `last`, so the conditional returns `first`. The second call provides both, so the full name is returned. The compiler checks that arguments are assignable to the parameter types.

#### Example 2: Default and rest parameters

```typescript
// example_params_default_rest.ts
// Step 1: Default parameter (annotation optional).
function greet(name: string, greeting = "Hello"): string {
  return `${greeting}, ${name}`;
}

// Step 2: Rest parameter (must be an array type).
function sum(...numbers: number[]): number {
  return numbers.reduce((a, b) => a + b, 0);
}

console.log(greet("Ada"));            // Expected: Hello, Ada
console.log(greet("Ada", "Hi"));      // Expected: Hi, Ada
console.log(sum(1, 2, 3, 4));         // Expected: 10
```

**Expected Output:**
```
Hello, Ada
Hi, Ada
10
```

**Why this output occurs.** `greeting` has a default value and an inferred `string` type. When omitted, it defaults to `"Hello"`. `sum` uses a rest parameter `...numbers: number[]`, which collects all arguments into a `number` array. The `reduce` computes the total.

### Real-World Cases

- **Configuration options**: Functions accept optional parameters for settings with sensible defaults.
- **Variadic utilities**: `sum(...values: number[])` and `log(...messages: unknown[])` use rest parameters for flexible argument lists.
- **Callback signatures**: Higher-order functions annotate callback parameters to ensure compatibility.

---

## 6. Property Annotations

### Definitions

**Core Definition.** A **property annotation** specifies the type of a property in a class, interface, or type alias.

**Technical Definition.** Property annotations appear after the property name. In classes, they may be combined with `readonly`, optional (`?`), and access modifiers. In interfaces and type aliases, they define the required or optional shape of object types. Class property annotations are optional; without an annotation and initializer, the property is implicitly `any`.

**Beginner-Friendly Explanation.** A property annotation tells TypeScript **what type a property holds**. In a class, you write `name: string` to say the property must be a string. In an interface, you write `age?: number` to say the property is optional. `readonly` prevents reassignment after initialization.

### Purposes

- **To define the shape of objects** in interfaces and type aliases.
- **To enforce property types in classes**.
- **To mark properties as optional** with `?`.
- **To enforce immutability** with `readonly`.
- **To document the structure of configuration objects and data models**.

### Syntax Rules and Structure

```
interface InterfaceName {
  required: Type;
  optional?: Type;
  readonly immutable: Type;
}

class ClassName {
  property: Type;
  optional?: Type;
  readonly immutable: Type;
}
```

**Component breakdown:**

- `required: Type` — a required property with the specified type.
- `optional?: Type` — an optional property; its type includes `undefined`.
- `readonly immutable: Type` — a property that cannot be reassigned after initialization.

**Syntax rules.**

- Optional properties use `?:` and automatically include `undefined`.
- `readonly` properties may be assigned at declaration or in the constructor but not afterward.
- A getter without a setter is automatically `readonly`.
- Class properties without annotations or initializers are implicitly `any` unless `strictPropertyInitialization` is enabled.

**Constraints and limitations.**

- `readonly` prevents reassignment but does not provide deep immutability.
- Optional properties may require `undefined` checks before use.
- Property annotations do not perform runtime validation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Interface property annotations

```typescript
// example_properties_interface.ts
// Step 1: Define an interface with required and optional properties.
interface Product {
  id: number;
  name: string;
  discount?: number;   // optional
}

// Step 2: Create values.
const full: Product = { id: 1, name: "Keyboard", discount: 10 };
const minimal: Product = { id: 2, name: "Mouse" };

console.log(full.discount);      // Expected: 10
console.log(minimal.discount);   // Expected: undefined
```

**Expected Output:**
```
10
undefined
```

**Why this output occurs.** `discount` is optional, so both objects satisfy `Product`. Accessing `minimal.discount` yields `undefined`, and its static type is `number | undefined`, requiring a check before arithmetic.

#### Example 2: Class property annotations with `readonly`

```typescript
// example_properties_class.ts
class User {
  readonly id: number;
  name: string;
  email?: string;

  constructor(id: number, name: string) {
    this.id = id;        // readonly assignment allowed in constructor
    this.name = name;
  }
}

const user = new User(1, "Ada");
console.log(user.name);          // Expected: Ada
// user.id = 2;                  // Error: Cannot assign to 'id' because it is a read-only property.
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `id` is `readonly`, so reassignment outside the constructor is a compile-time error. `email` is optional and may be omitted. The constructor assigns `id` and `name`; `email` remains `undefined`.

### Real-World Cases

- **Data models**: Interfaces annotate API response shapes with required and optional fields.
- **Configuration**: `readonly` properties prevent accidental mutation of configuration objects.
- **Class design**: Property annotations document and enforce the shape of class instances.

---

## 7. Type Annotation Best Practices

### Definitions

**Core Definition.** **Type annotation best practices** are conventions for deciding **when and how** to use type annotations to maximize safety, readability, and tooling effectiveness, while avoiding redundancy and over-annotation.

**Technical Definition.** Best practices include: relying on inference for trivially determined types; annotating **function signatures** (parameters and returns) for public APIs; avoiding redundant annotations on local variables; using `readonly` and `?` appropriately; and preferring precise types over wide types. Google’s TypeScript Style Guide recommends omitting annotations when inference is obvious and adding them for complex expressions and public interfaces.

**Beginner-Friendly Explanation.** Best practices help you decide **when to write annotations and when to let TypeScript guess**. The general rule: annotate things that are part of your code’s public surface (function parameters, return types, exported interfaces), and let inference handle obvious local variables. Avoid annotating `const x: boolean = true` — TypeScript already knows.

### Purposes

- **To maximize safety** by annotating the places where mistakes are most likely (boundaries, public APIs).
- **To reduce verbosity** by letting inference handle trivial cases.
- **To improve readability** by documenting complex signatures.
- **To align with official guidance** and team conventions.

### Syntax Rules and Structure

**Key best-practice rules:**

1. **Omit annotations for trivially inferred types**: `const x = 15` (not `const x: number = 15`).
2. **Annotate function parameters and return types** on public/exported functions.
3. **Annotate structural types explicitly** for object literals and interfaces.
4. **Use lowercase primitives** (`string`, `number`, `boolean`, `symbol`, `object`) rather than boxed types.
5. **Prefer `unknown` over `any`** when a type is genuinely unknown.
6. **Use `readonly` for immutable properties** and `?` for genuinely optional properties.
7. **Avoid type assertions (`as T`)** unless you have runtime evidence; they bypass checking.
8. **Annotate return types of complex functions** to catch accidental return-type changes and improve documentation.

**Constraints and limitations.**

- Inference may produce `any` in non-strict configurations; `--noImplicitAny` and `--strict` mitigate this.
- Over-annotation of local variables can obscure rather than clarify.
- Annotations do not replace runtime validation at trust boundaries.

### Annotated Complete Step by Step Code Examples

#### Example 1: Omit redundant annotations

```typescript
// example_best_omit.ts
// Recommended: let inference handle obvious types.
const count = 42;                 // inferred: number
const label = "items";            // inferred: string
const isReady = true;             // inferred: boolean

// Discouraged: redundant annotations add noise.
const redundantCount: number = 42;
const redundantLabel: string = "items";

console.log(count, label, isReady);
// Expected: 42 items true
```

**Expected Output:**
```
42 items true
```

**Why this output occurs.** Both versions produce identical runtime behavior. The unannotated version is preferred because inference already determines the types unambiguously.

#### Example 2: Annotate function signatures

```typescript
// example_best_function.ts
// Recommended: annotate parameters and return type.
export function calculateTotal(
  prices: number[],
  taxRate: number
): number {
  const subtotal = prices.reduce((a, b) => a + b, 0);
  return subtotal * (1 + taxRate);
}

console.log(calculateTotal([10, 20], 0.1));
// Expected: 33
```

**Expected Output:**
```
33
```

**Why this output occurs.** The annotations document the expected input and output types. Callers receive clear errors if they pass the wrong types. The return annotation ensures the function returns a `number`, catching accidental changes during refactoring.

#### Example 3: Annotate structural types explicitly

```typescript
// example_best_structural.ts
// Recommended: explicitly annotate the object shape.
interface Point {
  x: number;
  y: number;
}

const origin: Point = { x: 0, y: 0 };

console.log(origin.x, origin.y);
// Expected: 0 0
```

**Expected Output:**
```
0 0
```

**Why this output occurs.** The interface explicitly declares the structural type, catching field mismatches at the declaration site rather than at usage sites.

### Real-World Cases

- **Public libraries**: Exported functions annotate signatures, while internal local variables rely on inference.
- **Strict codebases**: `noImplicitAny` and explicit structural types prevent `any` leakage.
- **Team style guides**: Google’s TypeScript Style Guide codifies when to omit and when to require annotations.

---

## References

- TypeScript Handbook — Everyday Types - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
- TypeScript Handbook — More on Functions - https://www.typescriptlang.org/docs/handbook/2/functions.html
- TypeScript Handbook — Classes - https://www.typescriptlang.org/docs/handbook/2/classes.html 
- TypeScript Handbook — Do’s and Don’ts - https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html 
- Google TypeScript Style Guide — Type System - https://google.github.io/styleguide/tsguide.html 
- TypeScript Book — Type Annotations - https://github.com/gibbok/typescript-book#type-annotations 
- W3Schools — TypeScript Functions (Optional, Default, Rest Parameters) - https://www.w3schools.com/typescript/typescript_functions.php 
- Learning TypeScript — Functions (Return Types, `void`, `never`) - https://www.learningtypescript.com/functions 
- TypeScript Documentation — Type Compatibility (Function Parameter Bivariance) - https://www.typescriptlang.org/docs/handbook/type-compatibility.html