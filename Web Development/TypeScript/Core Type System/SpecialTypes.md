# TypeScript Special & Bottom Types: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
TypeScript's special and bottom types are a set of types that occupy unique positions in the type hierarchy and serve specialized roles in the type system. These types—`any`, `unknown`, `never`, `void`, `object`/`Object`/`{}`, `null`, and `undefined`—are not used to describe the shape of ordinary values but rather to handle edge cases: dynamic data, absence of values, impossible states, and the boundaries of the type system itself.

**Technical Definition**
In type theory, a "top type" is a type that is assignable from every other type, and a "bottom type" is a type that is assignable to every other type. TypeScript has two top types (`any` and `unknown`) and one bottom type (`never`). The `void` type represents the absence of a return value and behaves as a "unit type" for function returns. The `object` type (lowercase) represents all non-primitive values, while `Object` (uppercase) and `{}` represent the empty object type—which, counterintuitively, accepts all non-nullish values. With `strictNullChecks` enabled, `null` and `undefined` are no longer subtypes of every other type and must be handled explicitly. TypeScript 3.0 introduced `unknown` as a type-safe alternative to `any`.

**Beginner-Friendly Explanation**
TypeScript has some special types that don't describe what a value looks like but instead describe special situations. `any` means "turn off type checking for this." `unknown` means "I don't know what this is yet, so you need to check before using it." `never` means "this can't happen." `void` means "this function doesn't return anything." The `object` types are about what counts as an object. And `null`/`undefined` are about missing values. Understanding these types helps you write safer code and handle edge cases correctly.

### Key Characteristics

- **Top types** (`any`, `unknown`): Every type is assignable to them. `any` allows any operation; `unknown` requires narrowing before use.
- **Bottom type** (`never`): Nothing is assignable to `never` except `never` itself. Represents impossible values.
- **Unit type** (`void`): Represents the absence of a return value. Functions returning `void` can be used in callback positions where return values are ignored.
- **Object types** (`object`, `Object`, `{}`): `object` excludes primitives; `Object` and `{}` are structurally identical and accept all non-nullish values.
- **Nullish types** (`null`, `undefined`): With `strictNullChecks`, these are distinct types that require explicit handling.
- **Compile-time only**: All special types have no runtime representation; they are erased during compilation.

### Prerequisites

- Basic understanding of TypeScript's type system (primitives, unions, interfaces)
- Familiarity with `tsconfig.json` and compiler options
- Knowledge of variable declaration keywords (`let`, `const`, `var`)
- Understanding of union types and type narrowing

### Related Programming Areas

- **Type Theory**: Top types, bottom types, and lattice structures
- **Gradual Typing**: `any` as an escape hatch from static typing
- **Control Flow Analysis**: `never` and `unknown` in exhaustive checking
- **Null Safety**: `strictNullChecks` and null-aware type systems
- **API Design**: Choosing between `any`, `unknown`, and generics at boundaries

### Core Concepts / Features

The following core concepts are covered in this cheat sheet:

1. `any` — The Escape Hatch
2. `unknown` — The Type-Safe Alternative to `any`
3. `never` — The Bottom Type / Exhaustive Checking
4. `void` — Function Return Absence
5. `object` vs `Object` vs `{}` — The Empty Object Trap
6. `null` and `undefined` — Strict Null Checks
7. Differences Among Special Types
8. Appropriate and Inappropriate Use Cases

---

## 1. `any` — The Escape Hatch

### Definitions

**Core Definition**
`any` is a special type in TypeScript that disables all type checking for the value it describes. A value of type `any` can be assigned to any other type, and any other type can be assigned to it. Operations on `any` values are not checked at compile time, effectively making the value behave like an untyped JavaScript value.

**Technical Definition**
`any` is a "universal supertype" (top type) that encompasses every possible value in TypeScript, with the sole exception of `never`. It is bidirectionally assignable: any type is assignable to `any`, and `any` is assignable to any type. This bidirectional assignability means that `any` acts as a "black hole" in the type system—any type it touches becomes `any`, and type information is lost. The compiler treats `any` as "please turn off type checking for this thing". Under `noImplicitAny`, implicit `any` types (where no annotation is present and the compiler cannot infer a type) are treated as errors.

**Beginner-Friendly Explanation**
`any` is TypeScript's "off switch" for type checking. When you mark something as `any`, you're telling TypeScript: "Don't check this—I'll handle it." You can assign anything to it, call any method on it, and access any property. This is useful when you're migrating JavaScript code to TypeScript or working with truly dynamic data. But it's dangerous because you lose all the safety TypeScript provides. If you make a mistake with an `any` value, TypeScript won't catch it—you'll find out at runtime.

### Purposes

- To incrementally migrate a JavaScript codebase to TypeScript by temporarily disabling type checking on un-migrated modules.
- To work with third-party libraries that lack type definitions and cannot be easily typed.
- To bypass type system limitations when prototyping or experimenting with dynamic patterns.
- To handle truly dynamic runtime values where no type information is available or meaningful.
- To serve as a last-resort escape hatch when no other type accurately describes a value's behavior.

### Syntax Rules and Structure

**General Syntax: Explicit `any` Annotation**

```typescript
let variableName: any = expression;
```

**Component Breakdown**
- `variableName`: The variable being declared.
- `: any`: The explicit type annotation that disables type checking.
- `= expression`: The initializer (any value is assignable).

**General Syntax: `any` in Function Signatures**

```typescript
function functionName(param: any): any {
  return param;
}
```

**Component Breakdown**
- `param: any`: The parameter accepts any type.
- `): any`: The return type is `any`, disabling return type checking.

**Syntax Rules**

- `any` can be used in any type position: variable annotations, parameter types, return types, generic arguments, and type assertions.
- Any value can be assigned to a variable of type `any`.
- A variable of type `any` can be assigned to any other type without a type assertion.
- Property access, method calls, and index access on `any` values are not checked.
- `any` is assignable to every type except `never`.
- Under `noImplicitAny`, implicit `any` (where inference fails) is treated as an error.

**Constraints and Limitations**

- `any` completely bypasses type checking, eliminating TypeScript's primary benefit.
- Type information is lost when a value flows through an `any`-typed variable—it propagates like a virus.
- `any` cannot be assigned to `never` (the only exception to universal assignability).
- Using `any` extensively is considered an anti-pattern in production TypeScript codebases.
- The `no-explicit-any` ESLint rule can enforce avoiding explicit `any` usage.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `any` Usage and Type Safety Loss

```typescript
// Step 1: Declare a variable with any type.
let dynamicValue: any = "hello";

// Step 2: Assign a number — this is allowed because any accepts all types.
dynamicValue = 42;  // ✅ No error

// Step 3: Assign a boolean — also allowed.
dynamicValue = true;  // ✅ No error

// Step 4: Call a method that doesn't exist on the current value.
// TypeScript does NOT catch this error because the type is any.
dynamicValue.toUpperCase();  // ✅ Compiles, but ❌ crashes at runtime!

// Step 5: Assign to a typed variable without assertion.
let message: string = dynamicValue;  // ✅ No error (unsafe!)

console.log("This line may not be reached if runtime error occurred.");
```

**Expected Output:**
```
TypeError: dynamicValue.toUpperCase is not a function
```

**Why This Output Occurs:** The `any` type disables all type checking. TypeScript does not verify that `toUpperCase()` exists on a boolean value. At runtime, when `dynamicValue` holds a boolean (`true`), calling `toUpperCase()` throws a `TypeError`. This demonstrates the danger of `any`: type errors that would normally be caught at compile time become runtime crashes.

#### Example 2: `any` in Function Parameters (Migration Scenario)

```typescript
// Step 1: Define a function that accepts any type.
// This is useful during migration when a function's parameter
// types are not yet known.
function logValue(value: any): void {
  console.log("Value:", value);
  // No type checking on value — any operation is allowed.
}

// Step 2: Call with different types.
logValue(42);           // number
logValue("hello");      // string
logValue({ id: 1 });    // object
logValue([1, 2, 3]);    // array

// Step 3: The return type any propagates.
function getConfig(): any {
  return { theme: "dark", fontSize: 14 };
}

const config = getConfig();  // config is any
config.nonExistentProperty;  // ✅ No error (but undefined at runtime)
config.someMethod();         // ✅ No error (but crashes at runtime)
```

**Expected Output:**
```
Value: 42
Value: hello
Value: { id: 1 }
Value: [ 1, 2, 3 ]
```

**Why This Output Occurs:** The `logValue` function accepts `any`, so it can be called with any value. The `getConfig` function returns `any`, so the returned object's properties are not checked. Accessing `nonExistentProperty` returns `undefined` at runtime, and calling `someMethod()` would throw a runtime error. This is acceptable during migration but should be replaced with proper types when possible.

### Real-World Cases

**Case 1: JavaScript Migration**
When migrating a large JavaScript codebase to TypeScript, `any` can be used to annotate un-migrated modules temporarily, allowing the project to compile while migration proceeds incrementally. Each `any` annotation should be tracked and replaced with proper types over time.

**Case 2: Dynamic JSON Parsing**
`JSON.parse()` returns `any` because the shape of parsed JSON is not known at compile time. While this is convenient, `unknown` is a safer alternative that forces the developer to validate the parsed data before using it.

**Case 3: Third-Party Library Interop**
When working with JavaScript libraries that lack type definitions, `any` can be used as a temporary bridge. However, creating a `.d.ts` declaration file or using `unknown` with type guards is preferred for long-term maintainability.

---

## 2. `unknown` — The Type-Safe Alternative to `any`

### Definitions

**Core Definition**
`unknown` is a top type introduced in TypeScript 3.0 that is similar to `any` in that any value can be assigned to it, but unlike `any`, a value of type `unknown` cannot be used in any operation without first being narrowed to a more specific type. It is the type-safe counterpart of `any`.

**Technical Definition**
`unknown` is a top type (universal supertype) that is assignable from every type, but is only assignable to `unknown` and `any`. Unlike `any`, which permits any operation, `unknown` requires explicit type narrowing (via type guards, assertions, or type predicates) before the value can be used in any way. This makes `unknown` the safest way to represent a value whose type is not known at compile time. TypeScript 3.0 introduced `unknown` specifically to address the type safety problems caused by `any`.

**Beginner-Friendly Explanation**
`unknown` is like `any` but with a safety lock. You can put any value into an `unknown` variable, just like `any`. But before you can actually do anything with that value—call a method, access a property, or use it in an operation—you must first check what type it is. TypeScript forces you to narrow the type before using it. This prevents the runtime crashes that `any` allows. Think of `unknown` as saying: "I don't know what this is, but I'll check before using it."

### Purposes

- To safely represent values of unknown type without disabling type checking.
- To force developers to perform type validation before using dynamic data.
- To replace `any` in API boundaries where the input type is not known in advance.
- To enable type-safe handling of JSON-parsed data and external inputs.
- To provide a safer default type for generic parameters and function returns when the type is uncertain.

### Syntax Rules and Structure

**General Syntax: `unknown` Annotation**

```typescript
let variableName: unknown = expression;
```

**Component Breakdown**
- `variableName`: The variable being declared.
- `: unknown`: The type annotation for an unknown value.
- `= expression`: Any expression is assignable to `unknown`.

**General Syntax: Narrowing `unknown`**

```typescript
if (typeof variable === "string") {
  // variable is narrowed to string here
}
```

**Component Breakdown**
- `typeof variable === "string"`: A type guard that narrows `unknown` to `string`.
- Inside the block: `variable` is `string` and can be used as such.

**General Syntax: Type Assertion from `unknown`**

```typescript
const typedValue = unknownValue as SpecificType;
```

**Component Breakdown**
- `unknownValue`: A value of type `unknown`.
- `as SpecificType`: An assertion that tells TypeScript to treat the value as `SpecificType`.
- The assertion is unchecked at runtime—use with caution.

**Syntax Rules**

- Any value is assignable to `unknown`.
- `unknown` is assignable only to `unknown` and `any`.
- No operations are permitted on `unknown` values without narrowing (no property access, no method calls, no arithmetic, no function calls).
- Type narrowing works with `unknown` using all standard type guards (`typeof`, `instanceof`, `in`, custom predicates).
- Type assertions (`as`) can convert `unknown` to any other type.
- `unknown` is the default type parameter constraint in some generic contexts.

**Constraints and Limitations**

- `unknown` cannot be used directly in operations—it must be narrowed first.
- Type assertions from `unknown` are unchecked and can cause runtime errors if incorrect.
- In some complex generic scenarios, `unknown` may propagate further than desired.
- `unknown` is not a subtype of any specific type (except `any`).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Safe JSON Parsing with `unknown`

```typescript
// Step 1: Define a function that parses JSON and returns unknown.
// This forces the caller to validate the result before using it.
function parseJson(json: string): unknown {
  return JSON.parse(json);
}

// Step 2: Parse some JSON.
const result = parseJson('{ "name": "Alice", "age": 30 }');

// Step 3: Attempt to access properties directly — this is a compile error.
// console.log(result.name);  // ❌ Error: Object is of type 'unknown'.

// Step 4: Narrow the type using a type guard.
function isUser(value: unknown): value is { name: string; age: number } {
  return (
    typeof value === "object" &&
    value !== null &&
    "name" in value &&
    "age" in value &&
    typeof (value as any).name === "string" &&
    typeof (value as any).age === "number"
  );
}

// Step 5: Use the type guard to narrow and safely access properties.
if (isUser(result)) {
  // result is now narrowed to { name: string; age: number }
  console.log(`User: ${result.name}, Age: ${result.age}`);
} else {
  console.log("Invalid user data.");
}
```

**Expected Output:**
```
User: Alice, Age: 30
```

**Why This Output Occurs:** The `parseJson` function returns `unknown`, preventing the caller from assuming any structure. The `isUser` type predicate function validates the shape at runtime and tells TypeScript to narrow the type. Inside the `if` block, `result` is narrowed to the user type, allowing safe property access.

#### Example 2: `unknown` in Error Handling

```typescript
// Step 1: Define a function that processes errors of unknown type.
function handleError(error: unknown): string {
  // Step 2: Narrow using instanceof for Error objects.
  if (error instanceof Error) {
    return `Error: ${error.message}`;
  }

  // Step 3: Narrow using typeof for strings.
  if (typeof error === "string") {
    return `String error: ${error}`;
  }

  // Step 4: Handle all other cases.
  return "Unknown error occurred";
}

// Step 5: Test with different error types.
console.log(handleError(new Error("Something went wrong")));
console.log(handleError("A string error"));
console.log(handleError({ code: 500 }));
console.log(handleError(null));
```

**Expected Output:**
```
Error: Something went wrong
String error: A string error
Unknown error occurred
Unknown error occurred
```

**Why This Output Occurs:** The `unknown` type forces `handleError` to narrow the error before using it. The `instanceof Error` check narrows to `Error`, the `typeof` check narrows to `string`, and all other cases fall through to the default. This pattern is the recommended way to handle errors in TypeScript because `catch` clause variables are typed as `unknown` by default.

### Real-World Cases

**Case 1: API Response Validation**
When fetching data from an API, the response shape is not known at compile time. Using `unknown` for the parsed response forces developers to validate the shape with type guards before using it, preventing runtime errors from unexpected response structures.

**Case 2: Form Input Handling**
Form inputs from the DOM are inherently dynamic. Using `unknown` for input values forces validation logic to be written before the values are used in business logic, reducing the risk of type-related bugs.

**Case 3: Redux Action Payloads**
In Redux or similar state management libraries, action payloads can come from various sources. Using `unknown` for payload types forces reducers to validate and narrow the payload before processing it, improving type safety at the boundaries of the application.

---

## 3. `never` — The Bottom Type / Exhaustive Checking

### Definitions

**Core Definition**
`never` is TypeScript's bottom type—a type that represents values that can never occur. No value is assignable to `never` (except `never` itself), and `never` is assignable to every other type. It naturally arises in functions that never return (e.g., functions that always throw or contain infinite loops) and is used for exhaustiveness checking in discriminated unions.

**Technical Definition**
In type theory, the bottom type (often denoted as `⊥` or `never`) is the type that has no values. It is a subtype of every other type, meaning `never` is assignable to `string`, `number`, and every other type. However, no type is assignable to `never`. TypeScript uses `never` to represent code paths that are logically impossible, such as the `default` case in a fully exhaustive `switch` statement. Because `never` is only assignable to `never`, it can be used for compile-time exhaustiveness checking: if TypeScript does not error when assigning a value to `never`, then the value's type has not been fully narrowed, indicating a missing case.

**Beginner-Friendly Explanation**
`never` means "this can never happen." When a function always throws an error or loops forever, it never returns—so its return type is `never`. More importantly, `never` is used to make sure you've handled every possible case in a `switch` or `if/else` chain. If you write a function that should handle all possible types, you can add a `default` case that assigns the remaining value to `never`. If you forgot a case, TypeScript will give you an error because the remaining type isn't `never`. It's like a checklist that TypeScript verifies for you.

### Purposes

- To accurately model the return type of functions that never return (throw or loop forever).
- To enable compile-time exhaustiveness checking for discriminated unions and enums.
- To represent impossible code paths and unreachable states.
- To serve as the identity element for union types (`T | never` simplifies to `T`).
- To enable type-level programming patterns such as conditional type filtering.

### Syntax Rules and Structure

**General Syntax: `never` Return Type**

```typescript
function functionName(): never {
  throw new Error("This function never returns");
}
```

**Component Breakdown**
- `): never`: The return type annotation indicating the function never returns.
- The function body must throw an error or contain an infinite loop.

**General Syntax: Exhaustiveness Checking**

```typescript
function assertNever(value: never): never {
  throw new Error(`Unhandled value: ${value}`);
}

switch (unionValue) {
  case "A": /* ... */ break;
  case "B": /* ... */ break;
  default:
    assertNever(unionValue);  // ✅ Only compiles if all cases are handled
}
```

**Component Breakdown**
- `assertNever(value: never)`: A function that accepts only `never`.
- In the `default` case: If `unionValue` is not narrowed to `never`, the call is a compile error.

**Syntax Rules**

- `never` is assignable to every other type (it is the bottom type).
- No type is assignable to `never` except `never` itself.
- Functions with `never` return type must not have any reachable `return` statements.
- `T | never` simplifies to `T` in union types.
- `T & never` simplifies to `never` in intersection types.
- `never` is the default type for the `never` keyword in type-level conditional types.

**Constraints and Limitations**

- `never` cannot be used as a variable type for values that will be assigned (no value exists to assign).
- Functions that throw but are not annotated with `never` may have their return type inferred as `void` for backward compatibility with JavaScript class override patterns.
- `never` is erased at runtime; it has no runtime representation.
- Using `never` incorrectly (e.g., annotating a function that does return) causes compile errors.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Function That Never Returns

```typescript
// Step 1: Define a function that always throws.
// The return type is never because it never completes normally.
function throwError(message: string): never {
  throw new Error(message);
}

// Step 2: Define a function with an infinite loop.
// It also returns never because it never exits.
function infiniteLoop(): never {
  while (true) {
    // Do something forever
  }
}

// Step 3: Use the throwError function in a code path that should be unreachable.
function processValue(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  } else if (typeof value === "number") {
    return value.toFixed(2);
  }
  // This point is unreachable because value is never.
  // TypeScript allows the throwError call because it returns never.
  return throwError("Unexpected value type");
}
```

**Expected Output:** No output (the function either returns a value or throws an error). The `throwError` call is allowed because TypeScript understands that `never` is assignable to `string` (the declared return type of `processValue`).

**Why This Output Occurs:** The `throwError` function never returns, so its return type is `never`. Since `never` is assignable to every type, TypeScript allows `return throwError(...)` in a function declared to return `string`. This pattern is useful for handling logically unreachable code paths.

#### Example 2: Exhaustiveness Checking with `never`

```typescript
// Step 1: Define a discriminated union.
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number }
  | { kind: "rectangle"; width: number; height: number };

// Step 2: Define an assertNever function for exhaustiveness checking.
function assertNever(value: never): never {
  throw new Error(`Unhandled shape: ${JSON.stringify(value)}`);
}

// Step 3: Define a function that handles all shape kinds.
function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    case "rectangle":
      return shape.width * shape.height;
    default:
      // If all cases are handled, shape is narrowed to never here.
      // If a case is missing, TypeScript errors because shape is not never.
      return assertNever(shape);
  }
}

// Step 4: Test with each shape type.
console.log(calculateArea({ kind: "circle", radius: 5 }).toFixed(2));
console.log(calculateArea({ kind: "square", sideLength: 4 }));
console.log(calculateArea({ kind: "rectangle", width: 3, height: 6 }));
```

**Expected Output:**
```
78.54
16
18
```

**Why This Output Occurs:** The `switch` statement handles all three shape kinds. In the `default` case, TypeScript has narrowed `shape` to `never` because all possible `kind` values have been handled. The `assertNever` function accepts only `never`, so the call compiles. If a new shape kind were added to the union without adding a corresponding case, TypeScript would error on the `assertNever(shape)` call because `shape` would not be `never`.

### Real-World Cases

**Case 1: Redux Reducer Exhaustiveness**
Redux reducers typically use a `switch` statement on action type. Using `never` in the `default` case ensures that every action type is handled. If a new action type is added to the union without adding a case, the compiler catches the error immediately.

**Case 2: State Machine Validation**
State machines with discriminated unions can use `never` to verify that all state transitions are handled. The `assertNever` pattern ensures that no state is left unprocessed.

**Case 3: API Response Handling**
When handling API responses with multiple possible shapes (success, error, loading), `never` ensures that all response variants are processed. Adding a new response type without handling it triggers a compile error.

---

## 4. `void` — Function Return Absence

### Definitions

**Core Definition**
`void` is a special type in TypeScript that represents the absence of a return value from a function. It is used as the return type for functions that perform side effects (logging, mutation, etc.) but do not produce a meaningful value. Unlike `never`, which means a function never returns at all, `void` means a function returns but does not return a value.

**Technical Definition**
`void` is a "unit type" in type theory—it has exactly one value, which is `undefined` in TypeScript (when `strictNullChecks` is enabled, `undefined` is assignable to `void`). A function annotated with `void` as its return type may return `undefined` explicitly or have no `return` statement at all. Contextual typing with `void` does not force functions to not return a value; a function that returns a value can be assigned to a `void`-returning function type because the return value is simply ignored. TypeScript 5.1 introduced the ability for `undefined`-returning functions to have no return statement.

**Beginner-Friendly Explanation**
`void` means "this function doesn't return anything useful." It's used for functions that do something (like `console.log` or modify an array) but don't produce a value you'd want to use. If you write `function logMessage(): void`, you're saying: "This function returns nothing." You can still return `undefined` explicitly, or just not return anything at all. `void` is different from `never` because a `void` function finishes normally (it just doesn't give you a value), while a `never` function never finishes at all.

### Purposes

- To clearly document that a function is intended to be used only for its side effects.
- To enable callbacks and event handlers that ignore the return value of the function they invoke.
- To distinguish functions that return nothing from functions that never return.
- To allow functions that return values to be used in positions where the return value is ignored.
- To provide a type-safe way to specify that a function's return value is not meant to be consumed.

### Syntax Rules and Structure

**General Syntax: `void` Return Type**

```typescript
function functionName(): void {
  // No return statement, or return; or return undefined;
}
```

**Component Breakdown**
- `functionName`: The function name.
- `): void`: The return type annotation indicating no meaningful return value.
- The function body must not return a value (returning `undefined` is allowed).

**General Syntax: `void` in Callback Types**

```typescript
function functionName(callback: () => void): void {
  callback();
}
```

**Component Breakdown**
- `callback: () => void`: A callback that returns nothing.
- Functions that return values are assignable to `() => void` because the return value is ignored.

**Syntax Rules**

- `void` functions can return `undefined` (explicitly or implicitly).
- `undefined` is assignable to `void` under `strictNullChecks`.
- Functions returning any value are assignable to `void`-returning function types (return value is ignored).
- `void` is not assignable to `undefined` (the reverse is not true).
- `void` can be used in generic type positions (e.g., `Promise<void>`).
- A function with `void` return type cannot return a value (compile error).

**Constraints and Limitations**

- `void` is not the same as `any`—using `any` for callbacks loses type safety.
- `void` cannot be assigned from `undefined` in all contexts (e.g., `let x: void = undefined` is allowed, but `let y: undefined = voidValue` is not).
- `void`-returning function types accept functions that return values, which can lead to confusion about whether the return value is actually ignored.
- `void` has no runtime representation; it is purely a compile-time construct.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `void` Return Type for Side-Effect Functions

```typescript
// Step 1: Define a function that logs a message and returns nothing.
function logMessage(message: string): void {
  console.log(`[LOG] ${message}`);
  // No return statement — implicit return undefined.
}

// Step 2: Define a function that explicitly returns undefined.
function noOp(): void {
  return undefined;  // ✅ Explicit undefined return is allowed.
}

// Step 3: Define a function that returns a value — this is an error.
// function badVoid(): void {
//   return 42;  // ❌ Error: Type 'number' is not assignable to type 'void'.
// }

// Step 4: Call the void functions.
logMessage("Hello, TypeScript!");
noOp();
```

**Expected Output:**
```
[LOG] Hello, TypeScript!
```

**Why This Output Occurs:** The `logMessage` function performs a side effect (logging) and has no return statement, so its return type is `void`. The `noOp` function explicitly returns `undefined`, which is allowed because `undefined` is assignable to `void`. The commented-out `badVoid` function would cause a compile error because returning a number is not allowed when the return type is `void`.

#### Example 2: `void` in Callback Types

```typescript
// Step 1: Define a function that accepts a void-returning callback.
function processItems(items: number[], callback: (item: number) => void): void {
  for (const item of items) {
    callback(item);
  }
}

// Step 2: Pass a callback that returns a value.
// This is allowed because a function returning a value
// is assignable to a void-returning function type.
processItems([1, 2, 3], (item) => {
  console.log(item * 2);  // Returns void (console.log returns void)
});

// Step 3: Pass a callback that returns a number — still allowed!
processItems([4, 5, 6], (item) => {
  return item * 10;  // Return value is ignored by processItems
});

// Step 4: The return value of the callback is not used.
const result = processItems([7, 8], (item) => {
  return item + 100;  // Return value is ignored; result is void
});
console.log(result);  // undefined
```

**Expected Output:**
```
2
4
6
undefined
```

**Why This Output Occurs:** The `processItems` function accepts a callback of type `(item: number) => void`. Even though the second callback returns `item * 10`, the return value is ignored because the callback type specifies `void`. This is a deliberate TypeScript design decision: functions returning values are assignable to `void`-returning function types because the caller ignores the return value.

### Real-World Cases

**Case 1: Event Handlers in DOM**
DOM event handlers typically perform side effects (updating the DOM, logging, etc.) and do not return values. Using `void` for event handler return types clearly communicates that the handler's return value is ignored.

**Case 2: Array `forEach` Callbacks**
The `forEach` method accepts a callback of type `(value, index, array) => void`. This allows callbacks that return values (which are ignored) and callbacks that don't return anything.

**Case 3: Redux Dispatch Actions**
In Redux, `dispatch` returns the action itself, but many action creators are typed to return `void` when the return value is not meant to be used. This prevents callers from accidentally relying on the return value.

---

## 5. `object` vs `Object` vs `{}` — The Empty Object Trap

### Definitions

**Core Definition**
TypeScript has three confusingly similar types for representing objects: `object` (lowercase), `Object` (uppercase), and `{}` (empty object literal type). They have different meanings and accept different sets of values. `object` represents all non-primitive values. `Object` (uppercase) is the type of instances of the global `Object` class and accepts all non-nullish values (including primitives due to boxing). `{}` is the empty object type that also accepts all non-nullish values and is structurally identical to `Object` for assignment purposes.

**Technical Definition**
The lowercase `object` type was introduced in TypeScript 2.2 and represents any non-primitive value—that is, any value that is not `number`, `string`, `boolean`, `bigint`, `symbol`, `null`, or `undefined`. The uppercase `Object` type describes instances of the global `Object` class and includes all non-nullish values because primitives are automatically boxed to their wrapper objects when needed. The `{}` type—despite its appearance—does not mean "an object with no properties." Instead, it means "any value that is not `null` or `undefined`." Strings, numbers, booleans, and arrays all satisfy `{}`. The key difference between `Object` and `{}` is that `{}` does not enforce compatibility with `Object.prototype` properties, allowing objects with conflicting property types to be assigned.

**Beginner-Friendly Explanation**
These three types look similar but mean very different things. `object` (lowercase) means "any object, but not strings, numbers, or booleans." `Object` (uppercase) means "anything that has Object methods like `toString()`"—which includes strings and numbers because they get wrapped in objects. `{}` is the trickiest: it looks like "an empty object" but actually means "anything that's not null or undefined." So a string is assignable to `{}`. This is a common trap: developers use `{}` thinking it means "an object with no properties" but it actually accepts almost everything. Use `Record<string, never>` for truly empty objects and `object` for any non-primitive.

### Purposes

- To distinguish between primitive and non-primitive values using `object`.
- To represent values that inherit from `Object.prototype` using `Object`.
- To exclude only `null` and `undefined` from a type using `{}` (though this is rarely the intended meaning).
- To constrain generic type parameters to non-primitive types.
- To understand and avoid the common pitfalls of using `{}` and `Object` in type annotations.

### Syntax Rules and Structure

**General Syntax: `object` Type**

```typescript
let variableName: object = { key: "value" };
// let primitive: object = 42;  // ❌ Error: number is not assignable to object
```

**Component Breakdown**
- `: object`: The type annotation for any non-primitive value.
- Accepts: objects, arrays, functions, class instances.
- Rejects: `string`, `number`, `boolean`, `bigint`, `symbol`, `null`, `undefined`.

**General Syntax: `Object` Type**

```typescript
let variableName: Object = 42;  // ✅ Allowed (boxing)
let variableName2: Object = "hello";  // ✅ Allowed (boxing)
// let variableName3: Object = null;  // ❌ Error: null is not assignable to Object
```

**Component Breakdown**
- `: Object`: The type annotation for instances of the global Object class.
- Accepts: all non-nullish values (primitives are boxed).
- Rejects: `null` and `undefined`.

**General Syntax: `{}` Type**

```typescript
let variableName: {} = "hello";  // ✅ Allowed — string is not nullish
let variableName2: {} = 42;  // ✅ Allowed — number is not nullish
// let variableName3: {} = null;  // ❌ Error: null is not assignable to {}
```

**Component Breakdown**
- `: {}`: The empty object literal type (meaning "not nullish").
- Accepts: all non-nullish values (strings, numbers, booleans, objects, arrays, functions).
- Rejects: `null` and `undefined`.

**Syntax Rules**

- `object` accepts only non-primitive values (introduced in TypeScript 2.2).
- `Object` (uppercase) accepts all non-nullish values due to primitive boxing.
- `{}` accepts all non-nullish values and is structurally equivalent to `Object` for assignment.
- `object` is not assignable to `{}` in all cases (due to `Object.prototype` property conflicts).
- `null` and `undefined` are not assignable to any of the three types.
- For truly empty objects, use `Record<string, never>`.

**Constraints and Limitations**

- `{}` does not mean "empty object"—it means "any non-nullish value."
- `Object` and `{}` accept primitives, which is often surprising and unintended.
- Accessing properties on `object`, `Object`, or `{}` values is not allowed without narrowing.
- The TypeScript Do's and Don'ts guide recommends avoiding `Object` and `{}` in favor of `object` or more specific types.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `object` vs `Object` vs `{}` Assignment Behavior

```typescript
// Step 1: Test object (lowercase) — only non-primitives.
let obj1: object = { name: "Alice" };     // ✅ Object literal
let obj2: object = [1, 2, 3];             // ✅ Array
let obj3: object = () => {};              // ✅ Function
// let obj4: object = 42;                 // ❌ Error: number is not assignable to object
// let obj5: object = "hello";            // ❌ Error: string is not assignable to object

// Step 2: Test Object (uppercase) — all non-nullish values.
let obj6: Object = 42;                    // ✅ Allowed (primitive boxing)
let obj7: Object = "hello";               // ✅ Allowed (primitive boxing)
let obj8: Object = { name: "Bob" };       // ✅ Allowed
// let obj9: Object = null;               // ❌ Error: null is not assignable to Object
// let obj10: Object = undefined;         // ❌ Error: undefined is not assignable to Object

// Step 3: Test {} — all non-nullish values.
let obj11: {} = 42;                       // ✅ Allowed (surprising!)
let obj12: {} = "hello";                  // ✅ Allowed (surprising!)
let obj13: {} = true;                     // ✅ Allowed (surprising!)
let obj14: {} = { name: "Charlie" };      // ✅ Allowed
// let obj15: {} = null;                  // ❌ Error: null is not assignable to {}
// let obj16: {} = undefined;             // ❌ Error: undefined is not assignable to {}

console.log("All assignments compiled successfully.");
```

**Expected Output:**
```
All assignments compiled successfully.
```

**Why This Output Occurs:** The `object` type rejects primitives, so `42` and `"hello"` are not assignable. The `Object` type accepts primitives due to automatic boxing, so `42` and `"hello"` are assignable. The `{}` type also accepts primitives because it means "not nullish," not "empty object." All three types reject `null` and `undefined`.

#### Example 2: `object` vs `{}` in Type Guards

```typescript
// Step 1: Define a type guard using object.
function isObject(value: unknown): value is object {
  return typeof value === "object" && value !== null;
}

// Step 2: Define a type guard using {}.
function isNonNullish(value: unknown): value is {} {
  return value !== null && value !== undefined;
}

// Step 3: Test with various values.
const values: unknown[] = [42, "hello", { a: 1 }, [1, 2], null, undefined, true];

console.log("Using isObject (non-primitives only):");
for (const value of values) {
  if (isObject(value)) {
    console.log(`  object: ${JSON.stringify(value)}`);
  }
}

console.log("\nUsing isNonNullish (all non-nullish):");
for (const value of values) {
  if (isNonNullish(value)) {
    console.log(`  non-nullish: ${JSON.stringify(value)}`);
  }
}
```

**Expected Output:**
```
Using isObject (non-primitives only):
  object: {"a":1}
  object: [1,2]

Using isNonNullish (all non-nullish):
  non-nullish: 42
  non-nullish: "hello"
  non-nullish: {"a":1}
  non-nullish: [1,2]
  non-nullish: true
```

**Why This Output Occurs:** The `isObject` type guard uses `typeof value === "object"` and excludes `null`, so it only matches objects and arrays—not primitives. The `isNonNullish` type guard matches all values that are not `null` or `undefined`, including primitives. This demonstrates the practical difference: `object` is for non-primitives, while `{}` is for non-nullish values.

### Real-World Cases

**Case 1: Excluding Primitives in Generic Constraints**
When writing a generic function that should only accept object types (not primitives), use `<T extends object>` as the constraint. This prevents accidental use with `string`, `number`, or other primitives.

**Case 2: Validating Non-Nullish Values**
When a function should reject `null` and `undefined` but accept everything else, `{}` can be used. However, `unknown` with explicit `null`/`undefined` checks is often clearer and more maintainable.

**Case 3: Library API Design**
The TypeScript Do's and Don'ts guide recommends using `object` instead of `Object` or `{}` when you want to accept any object type. This makes the API's intent clearer and avoids the primitive boxing confusion.

---

## 6. `null` and `undefined` — Strict Null Checks

### Definitions

**Core Definition**
`null` and `undefined` are two special types in TypeScript (and JavaScript) that represent the absence of a value. `undefined` is the default value of uninitialized variables and missing properties. `null` is an explicit assignment that indicates "no value." Before TypeScript 2.0, `null` and `undefined` were assignable to every type, making them impossible to exclude from type checking. The `strictNullChecks` compiler option (introduced in TypeScript 2.0) makes them distinct types that must be handled explicitly.

**Technical Definition**
In non-strict mode (without `strictNullChecks`), `null` and `undefined` are considered subtypes of every other type, meaning `T` and `T | undefined` are synonymous and `null` is assignable to `string`, `number`, and all other types. With `strictNullChecks` enabled, `null` and `undefined` are not in the domain of every type and are only assignable to themselves and `any` (the one exception being that `undefined` is also assignable to `void`). This requires explicit handling of `null` and `undefined` values through type guards, narrowing, or optional chaining. The `strictNullChecks` option is automatically enabled when `strict: true` is set in `tsconfig.json`.

**Beginner-Friendly Explanation**
Before TypeScript 2.0, `null` and `undefined` could sneak into any type, leading to runtime errors like "Cannot read property of undefined." The `strictNullChecks` option fixes this by making `null` and `undefined` their own separate types. If a function returns a `string`, it can't return `null` unless you explicitly write `string | null`. This forces you to handle missing values before using them. It's like having a safety net: TypeScript won't let you forget to check if something might be `null` or `undefined`.

### Purposes

- To prevent runtime null pointer exceptions by catching them at compile time.
- To make the possibility of missing values explicit in function signatures and variable types.
- To enable precise type narrowing for `null` and `undefined` through control flow analysis.
- To distinguish between "value is absent" (`undefined`) and "value is explicitly empty" (`null`).
- To support optional chaining (`?.`) and nullish coalescing (`??`) operators with full type safety.

### Syntax Rules and Structure

**General Syntax: Union with `null` or `undefined`**

```typescript
let variableName: TypeName | null = null;
let variableName2: TypeName | undefined = undefined;
let variableName3: TypeName | null | undefined = null;
```

**Component Breakdown**
- `TypeName | null`: The type can be `TypeName` or `null`.
- `TypeName | undefined`: The type can be `TypeName` or `undefined`.
- `TypeName | null | undefined`: The type can be `TypeName`, `null`, or `undefined`.

**General Syntax: Optional Property (implicit `undefined`)**

```typescript
interface InterfaceName {
  optionalProperty?: TypeName;  // Implicitly TypeName | undefined
}
```

**Component Breakdown**
- `optionalProperty?`: The `?` suffix adds `undefined` to the property's type.
- The property may be absent or explicitly `undefined`.

**General Syntax: Nullish Coalescing**

```typescript
const result = value ?? defaultValue;
```

**Component Breakdown**
- `value ?? defaultValue`: If `value` is `null` or `undefined`, use `defaultValue`.
- Otherwise, use `value`.

**Syntax Rules**

- With `strictNullChecks`, `null` and `undefined` are only assignable to themselves and `any` (with `undefined` also assignable to `void`).
- Optional parameters and properties automatically include `undefined` in their types.
- The `!= null` check narrows out both `null` and `undefined` (loose inequality).
- The `=== null` check narrows out only `null`; use `=== undefined` for `undefined`.
- `strictNullChecks` is included in the `strict` compiler option.
- Definite assignment assertions (`!`) can bypass strict null checks but should be used sparingly.

**Constraints and Limitations**

- `strictNullChecks` is not enabled by default unless `strict: true` is set.
- The `!` non-null assertion operator is unchecked and can cause runtime errors if used incorrectly.
- `strictNullChecks` may require significant code changes when enabled in an existing codebase.
- Some JavaScript libraries return `null` where TypeScript expects a value, requiring assertion or validation.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `strictNullChecks` in Action

```typescript
// Step 1: With strictNullChecks enabled, a string cannot be null.
let name: string = "Alice";
// name = null;  // ❌ Error: Type 'null' is not assignable to type 'string'
// name = undefined;  // ❌ Error: Type 'undefined' is not assignable to type 'string'

// Step 2: To allow null, explicitly include it in the union.
let nullableName: string | null = "Bob";
nullableName = null;  // ✅ Allowed

// Step 3: To allow undefined, explicitly include it in the union.
let optionalName: string | undefined = "Charlie";
optionalName = undefined;  // ✅ Allowed

// Step 4: Use type narrowing to safely access nullable values.
function greet(name: string | null): string {
  if (name === null) {
    return "Hello, stranger!";
  }
  // After the null check, name is narrowed to string.
  return `Hello, ${name}!`;
}

console.log(greet("Dave"));   // "Hello, Dave!"
console.log(greet(null));     // "Hello, stranger!"
```

**Expected Output:**
```
Hello, Dave!
Hello, stranger!
```

**Why This Output Occurs:** With `strictNullChecks`, `string` does not include `null` or `undefined`. The `greet` function explicitly accepts `string | null` and uses a `null` check to narrow the type. Inside the `if` block, `name` is `null`; after the block, `name` is narrowed to `string`.

#### Example 2: Optional Properties and Nullish Coalescing

```typescript
// Step 1: Define an interface with optional properties.
interface UserProfile {
  name: string;
  email?: string;  // Implicitly string | undefined
  phone?: string;  // Implicitly string | undefined
}

// Step 2: Create a user with only the required property.
const user: UserProfile = { name: "Alice" };

// Step 3: Access optional properties safely.
// Direct access returns undefined, which is allowed with strictNullChecks.
console.log(user.email);  // undefined

// Step 4: Use nullish coalescing to provide defaults.
const email = user.email ?? "no-email@example.com";
const phone = user.phone ?? "no-phone";
console.log(email);  // "no-email@example.com"
console.log(phone);  // "no-phone"

// Step 5: Use optional chaining for nested access.
interface Company {
  name: string;
  address?: {
    street: string;
    city: string;
  };
}

const company: Company = { name: "Acme" };
const city = company.address?.city ?? "Unknown";
console.log(city);  // "Unknown"
```

**Expected Output:**
```
undefined
no-email@example.com
no-phone
Unknown
```

**Why This Output Occurs:** The optional properties `email` and `phone` are typed as `string | undefined`. Accessing them directly returns `undefined` when absent. The nullish coalescing operator (`??`) provides fallback values for `null` and `undefined`. Optional chaining (`?.`) safely accesses nested properties, returning `undefined` if any part of the chain is nullish.

### Real-World Cases

**Case 1: API Response Handling**
API responses often contain optional fields or fields that may be `null`. With `strictNullChecks`, developers must explicitly handle these cases, preventing runtime errors from unexpected `null` values.

**Case 2: Configuration Objects**
Configuration objects with optional settings benefit from `strictNullChecks` because it forces developers to provide defaults or handle missing values explicitly, rather than relying on runtime fallbacks.

**Case 3: React Props and State**
React components with optional props use `undefined` to represent "not provided." `strictNullChecks` ensures that components handle missing props gracefully, preventing "Cannot read property of undefined" errors.

---

## 7. Differences Among Special Types

### Comparison Table

| Type | Category | Assignable From | Assignable To | Operations Allowed | Primary Use Case |
|------|----------|-----------------|---------------|-------------------|------------------|
| `any` | Top type | Everything | Everything (except `never`) | All | Escape hatch, migration |
| `unknown` | Top type | Everything | `unknown`, `any` | None (requires narrowing) | Safe dynamic values |
| `never` | Bottom type | `never` only | Everything | N/A (no values) | Exhaustive checking, impossible states |
| `void` | Unit type | `undefined`, `any` | `void`, `any` | N/A (no meaningful value) | Functions with no return value |
| `object` | Object type | Non-primitives | `object`, `any` | Property access (after narrowing) | Non-primitive values |
| `Object` | Object type | All non-nullish | `Object`, `any` | Object methods | Object instances |
| `{}` | Object type | All non-nullish | `{}`, `any` | Property access (after narrowing) | "Not nullish" constraint |
| `null` | Nullish type | `null` only | `null`, `any` | N/A | Explicit absence of value |
| `undefined` | Nullish type | `undefined` only | `undefined`, `void`, `any` | N/A | Uninitialized/missing value |

### Key Distinctions

- **`any` vs `unknown`**: Both accept all values, but `any` allows any operation while `unknown` requires narrowing. `unknown` is the type-safe alternative.
- **`never` vs `void`**: `void` means "returns nothing"; `never` means "never returns." A `void` function completes; a `never` function does not.
- **`object` vs `Object` vs `{}`**: `object` excludes primitives; `Object` and `{}` accept all non-nullish values. `{}` is structurally equivalent to `Object` for assignment but does not enforce `Object.prototype` compatibility.
- **`null` vs `undefined`**: `null` is an explicit assignment of "no value"; `undefined` is the default for uninitialized variables. With `strictNullChecks`, they are distinct types.
- **`any` vs `unknown` vs `{}`**: All three accept many values, but `any` disables checking, `unknown` requires narrowing, and `{}` simply excludes `null`/`undefined`.

---

## 8. Appropriate and Inappropriate Use Cases

### `any` — Appropriate and Inappropriate

**Appropriate:**
- During migration from JavaScript to TypeScript, temporarily annotating un-migrated modules.
- When interfacing with third-party libraries that have no type definitions and cannot be typed.
- In prototype code where type safety is not yet a priority.
- When the dynamic nature of the value is essential and no type guard is feasible.

**Inappropriate:**
- In production code where type safety is expected.
- As a return type for functions in public APIs (use `unknown` instead).
- When a more specific type (union, generic, interface) could be used.
- As a default type for variables or parameters when the type is simply not yet known.

### `unknown` — Appropriate and Inappropriate

**Appropriate:**
- For function return types when the return value's type is not known (e.g., JSON parsing).
- For API boundaries where input validation is required.
- For error handling in `catch` blocks (catch variables are `unknown` by default).
- For generic defaults when the type should be determined by the caller.

**Inappropriate:**
- When the type is actually known and can be specified.
- When you need to perform operations without narrowing (use generics instead).
- In performance-critical code where narrowing overhead is a concern (though this is rare).

### `never` — Appropriate and Inappropriate

**Appropriate:**
- For functions that always throw or contain infinite loops.
- For exhaustiveness checking in `switch` statements on discriminated unions.
- For type-level programming to represent impossible states.
- For `assertNever` helper functions in exhaustiveness patterns.

**Inappropriate:**
- As a variable type for values that will be assigned (no value exists to assign).
- As a return type for functions that do return.
- When `void` is the intended meaning (function returns nothing vs. never returns).

### `void` — Appropriate and Inappropriate

**Appropriate:**
- For functions that perform side effects and do not return a value.
- For callback types where the return value is ignored.
- For event handlers and DOM event listeners.
- For generic type parameters like `Promise<void>`.

**Inappropriate:**
- When the function never returns (use `never`).
- When the function returns a value that callers should use (specify the return type).
- As a substitute for `undefined` in all contexts (use `undefined` when the value is meaningful).

### `object` vs `Object` vs `{}` — Appropriate and Inappropriate

**Appropriate:**
- `object`: When constraining to non-primitive types.
- `Object` (uppercase): Rarely appropriate; prefer `object` or a specific interface.
- `{}`: Rarely appropriate; use `Record<string, never>` for empty objects, or `unknown` with null checks.

**Inappropriate:**
- `object`: When primitives should be accepted.
- `Object`: When primitives should be excluded.
- `{}`: When the intent is "empty object" (use `Record<string, never>` instead).

### `null` and `undefined` — Appropriate and Inappropriate

**Appropriate:**
- `null`: When explicitly representing "no value" as an intentional assignment.
- `undefined`: For optional properties, uninitialized variables, and missing function arguments.

**Inappropriate:**
- Using `null` where `undefined` is the natural JavaScript default.
- Using `!` (non-null assertion) to bypass `strictNullChecks` without proper validation.
- Mixing `null` and `undefined` inconsistently in the same API.

---

## References

- TypeScript Handbook: Everyday Types — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
- TypeScript Handbook: TypeScript 2.0 Release Notes (Null- and undefined-aware types) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html
- TypeScript Handbook: TypeScript 3.0 Release Notes (Unknown) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html
- TypeScript Handbook: Do's and Don'ts — https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html
- TypeScript Playground: Unknown and Never — https://www.typescriptlang.org/play/typescript/primitives/unknown-and-never.ts.html
- TypeScript Playground: Any — https://www.typescriptlang.org/play/typescript/primitives/any.ts.html
- TypeScript for Functional Programmers — https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html
- Exploring TypeScript: object vs Object vs {} — https://exploringjs.com/ts/downloads/exploring-ts-screen-preview.pdf
- TypeScript 5.1 Release Notes (undefined-returning functions) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-1.html
- MDN: typeof Operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof
- MDN: instanceof Operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof
- MDN: in Operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in
- TypeScript ESLint: no-explicit-any — https://typescript-eslint.io/rules/no-explicit-any/
- Total TypeScript: The `any` Type — https://www.totaltypescript.com/the-any-type
- Marijn Haverbeke: TypeScript's unknown type and type variance — https://marijnhaverbeke.nl/blog/typescript-unknown-type.html