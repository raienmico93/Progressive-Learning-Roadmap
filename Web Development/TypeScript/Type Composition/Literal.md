# TypeScript Literal Types: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A literal type in TypeScript is a type that represents a single, specific value rather than a general category. Instead of `string`, a literal type might be the specific string `"active"`. Instead of `number`, it might be `42`. Literal types enable TypeScript to enforce exact value constraints at compile time.

**Technical Definition**
Literal types are singleton types in TypeScript's type system: each literal type has exactly one value. TypeScript supports string literal types (`"hello"`), numeric literal types (`42`), boolean literal types (`true` / `false`), and bigint literal types (`42n`). Literal types are subtypes of their corresponding primitive types (`"hello"` is a subtype of `string`). When combined into unions (`"active" | "inactive"`), literal types model enumeration-like value constraints. Template literal types (TypeScript 4.1+) extend literal types to string patterns, enabling type-level string manipulation. Literal types are subject to widening: in mutable contexts, they are widened to their base primitive types; the `as const` assertion preserves literal types.

**Beginner-Friendly Explanation**
A literal type is a type that says "this exact value and nothing else." Instead of `string`, you can say `"active"`—meaning the value must be exactly the string `"active"`. Literal types let you write very precise types: `type Status = "active" | "inactive" | "pending"` says a status is one of these three exact strings. This catches typos at compile time and enables exhaustive checking. Template literal types go further, letting you define patterns like `` `user-${string}` `` to describe strings that match a specific format. Literal types are one of TypeScript's most powerful features for precise modeling.

### Key Characteristics

- **Singleton types**: Each literal type has exactly one value.
- **Subtypes of primitives**: `"hello"` is a subtype of `string`; `42` is a subtype of `number`.
- **Union-composable**: Literal unions model exact value sets.
- **Widening-aware**: Literal types widen to primitives in mutable contexts.
- **`as const` preserves**: Const assertions prevent widening.
- **Template literal types**: String patterns as types (TypeScript 4.1+).
- **Compile-time only**: Erased at runtime.

### Prerequisites

- Basic knowledge of TypeScript primitive types
- Familiarity with union types and type aliases
- Understanding of type inference and widening
- Basic familiarity with generics (for template literal types)

### Related Programming Areas

- **Type Theory**: Literal types are singleton types; literal unions are sum types
- **Enum Alternatives**: Literal unions often replace enums
- **Type-Level Programming**: Template literal types enable compile-time string manipulation
- **API Design**: Literal unions model exact API states
- **Configuration**: Literal types enforce exact configuration values

### Core Concepts / Features

1. String, Numeric, and Boolean Literal Syntax
2. Literal Unions for Exact Value Constraints
3. Configuration Modeling and API State Management
4. Template Literal Types for Dynamic String Patterns
5. Type Widening and Controlling It Using `as const` (Const Assertions)


## 1. String, Numeric, and Boolean Literal Syntax

### Definitions

**Core Definition**
Literal type syntax uses the literal value itself as a type annotation. A string literal type is written as `"value"`, a numeric literal type as `42`, and a boolean literal type as `true` or `false`. These types represent exactly that value.

**Technical Definition**
Literal types are singleton types in TypeScript. A variable annotated with a literal type can only hold that exact value (or a value assignable to it). Literal types are subtypes of their corresponding primitive types: `"hello"` extends `string`, `42` extends `number`, `true` extends `boolean`. This subtyping relationship means literal types are assignable to primitives, but not vice versa. Literal types can be used in any type position: variable annotations, function parameters, return types, generic arguments, and union members. Bigint literal types (`42n`) are also supported (TypeScript 3.2+).

**Beginner-Friendly Explanation**
A literal type is when you use the actual value as the type. Instead of `let x: string`, you write `let x: "hello"`—meaning `x` can only be `"hello"`. Literal types are the most precise types possible: they allow exactly one value. You can use them for strings, numbers, and booleans. They're most useful in unions (`"active" | "inactive"`) or as function parameters that must be specific values. Literal types are subtypes of their primitives, so a `"hello"` value can be used where a `string` is expected, but not vice versa.

### Purposes

- To constrain values to exact literals for precise type checking.
- To catch typos at compile time (e.g., `"actve"` vs `"active"`).
- To model enum-like value sets without runtime code.
- To enable discriminated unions with literal discriminants.
- To serve as the foundation for template literal types.

### Syntax Rules and Structure

**General Syntax: String Literal Type**

```typescript
let variable: "literalValue" = "literalValue";
```

**Component Breakdown**
- `"literalValue"`: The string literal type.
- The variable can only hold that exact string.

**General Syntax: Numeric Literal Type**

```typescript
let variable: 42 = 42;
```

**Component Breakdown**
- `42`: The numeric literal type.
- The variable can only hold the number `42`.

**General Syntax: Boolean Literal Type**

```typescript
let variable: true = true;
```

**Component Breakdown**
- `true` (or `false`): The boolean literal type.

**General Syntax: Bigint Literal Type**

```typescript
let variable: 42n = 42n;
```

**Component Breakdown**
- `42n`: The bigint literal type (TypeScript 3.2+).

**Syntax Rules**

- Literal types use the literal value as the type annotation.
- String literals use double or single quotes.
- Numeric literals use decimal, hexadecimal, octal, or binary notation.
- Boolean literals are `true` and `false`.
- Bigint literals use the `n` suffix.
- Literal types are subtypes of their primitives.
- Literal types can be used in any type position.
- Literal types are subject to widening in mutable contexts.

**Constraints and Limitations**

- Literal types are erased at runtime; they exist only in the type system.
- Without `as const`, `let` variables widen literal types to primitives.
- Literal types cannot be used with `instanceof` or runtime type checks.
- Very large literal unions can slow down the compiler.
- Literal types do not work well with computed values.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Literal Types

```typescript
// Step 1: Declare variables with literal types.
let direction: "north" = "north";
// direction = "south";  // ❌ Error: Type '"south"' is not assignable to type '"north"'.

let answer: 42 = 42;
// answer = 43;  // ❌ Error: Type '43' is not assignable to type '42'.

let isReady: true = true;
// isReady = false;  // ❌ Error: Type 'false' is not assignable to type 'true'.

// Step 2: Literal types are subtypes of primitives.
let broadString: string = direction;   // ✅ Literal assignable to primitive
let broadNumber: number = answer;      // ✅
let broadBoolean: boolean = isReady;   // ✅

// Step 3: Primitives are NOT assignable to literal types.
// let narrowString: "north" = broadString;
// ❌ Error: Type 'string' is not assignable to type '"north"'.

// Step 4: Use literal types in functions.
function setDirection(dir: "north" | "south" | "east" | "west"): void {
  console.log(`Direction set to ${dir}`);
}

setDirection("north");  // ✅
setDirection("east");   // ✅
// setDirection("up");  // ❌ Error: Argument of type '"up"' is not assignable to parameter of type '"north" | "south" | "east" | "west"'.
```

**Expected Output:**
```
Direction set to north
Direction set to east
```

**Why This Output Occurs:** Literal types constrain variables to exact values. `direction` can only be `"north"`, `answer` can only be `42`, and `isReady` can only be `true`. The `setDirection` function accepts only the four literal directions, rejecting `"up"`.

#### Example 2: Literal Types with Widening

```typescript
// Step 1: `let` widens literal types.
let widened = "hello";  // Inferred as string (widened)
// widened = "world";   // ✅ Allowed (string)

// Step 2: `const` preserves literal types.
const preserved = "hello";  // Inferred as "hello" (literal)
// preserved = "world";     // ❌ Error: Cannot assign to 'preserved' because it is a constant.

// Step 3: `const` with object literals widens properties.
const config = { status: "active" };  // status: string (widened)
config.status = "inactive";           // ✅ Allowed

// Step 4: `as const` preserves literal types in objects.
const constConfig = { status: "active" } as const;
// constConfig.status = "inactive";  // ❌ Error: Cannot assign to 'status' because it is a read-only property.

// Step 5: Function parameters can use literal types.
function logLevel(level: "debug" | "info" | "warn" | "error"): void {
  console.log(`[${level.toUpperCase()}]`);
}

logLevel("info");   // "[INFO]"
logLevel("error");  // "[ERROR]"
// logLevel("verbose");  // ❌ Error: Argument of type '"verbose"' is not assignable to parameter of type '"debug" | "info" | "warn" | "error"'.
```

**Expected Output:**
```
[INFO]
[ERROR]
```

**Why This Output Occurs:** The `let` variable `widened` is inferred as `string` (widened), so it can be reassigned to any string. The `const` variable `preserved` is inferred as the literal `"hello"`, so it cannot be reassigned. The `const config` object has its `status` property widened to `string`, while `constConfig` with `as const` preserves the literal type `"active"`.

### Real-World Cases

**Case 1: HTTP Methods**
HTTP methods are naturally literal types: `type HttpMethod = "GET" | "POST" | "PUT" | "DELETE" | "PATCH"`.

**Case 2: Log Levels**
Log levels are literal types: `type LogLevel = "debug" | "info" | "warn" | "error"`.

**Case 3: Environment Names**
Environment names are literal types: `type Environment = "development" | "staging" | "production"`.

**Case 4: Direction Vectors**
Direction vectors use literal types: `type Direction = "north" | "south" | "east" | "west"`.

---

## 2. Literal Unions for Exact Value Constraints

### Definitions

**Core Definition**
A literal union is a union of two or more literal types, representing a set of exact values that a variable can hold. Literal unions are TypeScript's primary mechanism for modeling exact value constraints without runtime enums.

**Technical Definition**
A literal union type is a union whose members are all literal types: `type Status = "active" | "inactive" | "pending"`. Values of the union type must be exactly one of the listed literals. Literal unions are subtypes of their corresponding primitives (`"active" | "inactive"` extends `string`). They can be combined with other types via unions and intersections. Literal unions are often used as enum alternatives because they produce no runtime code, work with structural typing, and enable exhaustive checking via `never`. TypeScript 5.0+ treats enums as union types of their members, bringing enums and literal unions closer together.

**Beginner-Friendly Explanation**
A literal union is a list of exact values. Instead of saying "a string," you say "one of these three strings." For example, `type Status = "active" | "inactive" | "pending"` says a status must be exactly one of those three values. Literal unions are great for modeling states, modes, or categories. They're often used instead of enums because they're simpler, produce no runtime code, and work naturally with TypeScript's type system. If you try to assign a value that's not in the list, TypeScript catches it at compile time.

### Purposes

- To constrain values to an exact set of allowed literals.
- To model enum-like value sets without runtime code.
- To enable exhaustive checking via `never` in `switch` statements.
- To serve as discriminants in discriminated unions.
- To provide type-safe configuration options and API states.

### Syntax Rules and Structure

**General Syntax: Literal Union**

```typescript
type AliasName = "value1" | "value2" | "value3";
```

**Component Breakdown**
- `"value1" | "value2" | "value3"`: The union of literal types.
- Values must be exactly one of the literals.

**General Syntax: Mixed Literal Union**

```typescript
type Mixed = "auto" | 0 | 1 | true;
```

**Component Breakdown**
- Literal unions can mix string, number, and boolean literals.

**General Syntax: Literal Union as Discriminant**

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };
```

**Component Breakdown**
- The literal type `"circle"` / `"square"` serves as the discriminant.

**Syntax Rules**

- Literal unions use the `|` operator between literal types.
- All members must be literal types (or types that can be narrowed to literals).
- Literal unions are subtypes of the union of their primitive types.
- Literal unions can be mixed (string, number, boolean literals together).
- Literal unions can be used as discriminants in discriminated unions.
- Exhaustiveness checking works with literal unions via `never`.

**Constraints and Limitations**

- Literal unions can become unwieldy if they have many members.
- Adding a new member requires updating all exhaustive consumers.
- Literal unions do not exist at runtime (unlike enums).
- Literal unions cannot be iterated at runtime.
- Widening can accidentally broaden literal unions in some contexts.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Literal Union for Status

```typescript
// Step 1: Define a literal union.
type Status = "active" | "inactive" | "pending";

// Step 2: Use it in a function.
function getStatusMessage(status: Status): string {
  switch (status) {
    case "active": return "The account is active.";
    case "inactive": return "The account is inactive.";
    case "pending": return "The account is pending approval.";
    default:
      const _exhaustive: never = status;
      return _exhaustive;
  }
}

// Step 3: Call with valid and invalid values.
console.log(getStatusMessage("active"));    // "The account is active."
console.log(getStatusMessage("pending"));   // "The account is pending approval."
// getStatusMessage("deleted");  // ❌ Error: Argument of type '"deleted"' is not assignable to parameter of type 'Status'.

// Step 4: Use as a discriminant.
interface User {
  name: string;
  status: Status;
}

const alice: User = { name: "Alice", status: "active" };
console.log(`${alice.name}: ${alice.status}`);  // "Alice: active"

// Step 5: Literal unions are assignable to primitives.
const statusString: string = alice.status;  // ✅
console.log(statusString.toUpperCase());    // "ACTIVE"
```

**Expected Output:**
```
The account is active.
The account is pending approval.
Alice: active
ACTIVE
```

**Why This Output Occurs:** The `Status` literal union constrains values to exactly three strings. The `switch` statement handles all three, with the `never` check ensuring exhaustiveness. The literal union is assignable to `string`, enabling primitive operations after assignment.

#### Example 2: Mixed Literal Unions

```typescript
// Step 1: Define a mixed literal union.
type ConfigValue = "auto" | "none" | 0 | 1 | true | false;

// Step 2: Use it in a configuration function.
function setConfig(name: string, value: ConfigValue): void {
  console.log(`Setting ${name} = ${JSON.stringify(value)}`);
}

setConfig("mode", "auto");       // 'Setting mode = "auto"'
setConfig("retries", 0);         // 'Setting retries = 0'
setConfig("enabled", true);      // 'Setting enabled = true'
// setConfig("mode", "custom");  // ❌ Error: '"custom"' is not assignable to ConfigValue.

// Step 3: Narrowing works with mixed literal unions.
function describeValue(value: ConfigValue): string {
  if (typeof value === "string") {
    return `String config: ${value}`;
  }
  if (typeof value === "number") {
    return `Numeric config: ${value}`;
  }
  return `Boolean config: ${value}`;
}

console.log(describeValue("auto"));   // "String config: auto"
console.log(describeValue(1));        // "Numeric config: 1"
console.log(describeValue(false));    // "Boolean config: false"
```

**Expected Output:**
```
Setting mode = "auto"
Setting retries = 0
Setting enabled = true
String config: auto
Numeric config: 1
Boolean config: false
```

**Why This Output Occurs:** The mixed literal union accepts specific strings, numbers, and booleans. The `typeof` narrowing works because literal types are subtypes of their primitives, so `typeof value === "string"` narrows to `"auto" | "none"`.

### Real-World Cases

**Case 1: Redux Action Types**
Redux action types are literal unions: `type ActionType = "ADD_TODO" | "REMOVE_TODO" | "TOGGLE_TODO"`.

**Case 2: API Status Codes**
API status categories are literal unions: `type ApiStatus = "success" | "error" | "loading" | "idle"`.

**Case 3: Theme Variants**
UI theme variants are literal unions: `type Theme = "light" | "dark" | "system"`.

**Case 4: Sort Directions**
Sort directions are literal unions: `type SortDirection = "asc" | "desc"`.

---

## 3. Configuration Modeling and API State Management

### Definitions

**Core Definition**
Literal unions are extensively used to model configuration options and API states. They provide exact value constraints that prevent invalid configurations and enable exhaustive handling of API states (loading, success, error, etc.).

**Technical Definition**
Configuration modeling with literal unions defines the allowed values for each configuration option as a literal union. API state management uses discriminated unions (often with literal discriminants) to model the possible states of an API request: idle, loading, success, error, etc. These patterns provide compile-time guarantees that invalid states cannot be represented and that all states are handled. They are foundational to type-safe state management in React, Redux, and other frameworks.

**Beginner-Friendly Explanation**
Literal unions are perfect for configuration and API states. For configuration, you can say `type LogLevel = "debug" | "info" | "warn" | "error"` and TypeScript ensures only valid log levels are used. For API states, you model each state explicitly: `{ status: "loading" }`, `{ status: "success"; data: T }`, `{ status: "error"; error: string }`. This makes invalid states impossible—you can't have a "success" state without data, or a "loading" state with an error. This pattern is called a discriminated union or state machine, and it's one of TypeScript's most powerful features for building reliable applications.

### Purposes

- To enforce exact configuration values and prevent typos.
- To model all possible API states explicitly.
- To make invalid states unrepresentable.
- To enable exhaustive handling of all states.
- To provide type-safe state transitions.

### Syntax Rules and Structure

**General Syntax: Configuration Model**

```typescript
type Config = {
  logLevel: "debug" | "info" | "warn" | "error";
  environment: "development" | "staging" | "production";
  mode: "auto" | "manual";
};
```

**Component Breakdown**
- Each configuration option is a literal union.

**General Syntax: API State Model**

```typescript
type ApiState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: string };
```

**Component Breakdown**
- `status` is the discriminant literal.
- Each state carries its own data.

**General Syntax: Exhaustive State Handler**

```typescript
function handleState<T>(state: ApiState<T>): string {
  switch (state.status) {
    case "idle": return "Idle";
    case "loading": return "Loading...";
    case "success": return `Data: ${JSON.stringify(state.data)}`;
    case "error": return `Error: ${state.error}`;
  }
}
```

**Component Breakdown**
- The `switch` on `status` narrows to each state.
- Each case handles the specific state's data.

**Syntax Rules**

- Configuration options use literal unions for allowed values.
- API states use discriminated unions with literal discriminants.
- Each state variant carries its own data.
- Narrowing on the discriminant unlocks variant-specific properties.
- Exhaustiveness checking ensures all states are handled.
- Invalid state combinations are impossible (compile errors).

**Constraints and Limitations**

- Adding a new state requires updating all exhaustive consumers.
- Literal unions for configuration must be maintained as options change.
- State transitions must be explicitly modeled (TypeScript doesn't enforce them).
- Runtime validation is still needed for external data (APIs, user input).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Configuration Modeling

```typescript
// Step 1: Define configuration with literal unions.
interface AppConfig {
  logLevel: "debug" | "info" | "warn" | "error";
  environment: "development" | "staging" | "production";
  theme: "light" | "dark" | "system";
  cacheMode: "none" | "memory" | "disk";
}

// Step 2: Create a valid configuration.
const config: AppConfig = {
  logLevel: "info",
  environment: "production",
  theme: "dark",
  cacheMode: "disk",
};

console.log(`Environment: ${config.environment}`);
console.log(`Log Level: ${config.logLevel}`);

// Step 3: Invalid values are rejected.
// const badConfig: AppConfig = {
//   logLevel: "verbose",  // ❌ Error: '"verbose"' is not assignable to type '"debug" | "info" | "warn" | "error"'.
//   environment: "prod",
//   theme: "dark",
//   cacheMode: "disk",
// };

// Step 4: Configuration validation with exhaustiveness.
function getLogLevelPriority(level: AppConfig["logLevel"]): number {
  switch (level) {
    case "debug": return 0;
    case "info": return 1;
    case "warn": return 2;
    case "error": return 3;
  }
}

console.log(`Priority of info: ${getLogLevelPriority("info")}`);  // 1
```

**Expected Output:**
```
Environment: production
Log Level: info
Priority of info: 1
```

**Why This Output Occurs:** The `AppConfig` interface uses literal unions for each option, constraining values to specific sets. The `getLogLevelPriority` function exhaustively handles all log levels. Invalid values like `"verbose"` are caught at compile time.

#### Example 2: API State Management

```typescript
// Step 1: Define an API state discriminated union.
type ApiState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T; timestamp: Date }
  | { status: "error"; error: string; code: number };

// Step 2: Define a function that handles all states.
function renderState<T>(state: ApiState<T>): string {
  switch (state.status) {
    case "idle":
      return "No data loaded.";
    case "loading":
      return "Loading...";
    case "success":
      return `Loaded ${JSON.stringify(state.data)} at ${state.timestamp.toISOString()}`;
    case "error":
      return `Error ${state.code}: ${state.error}`;
  }
}

// Step 3: Create different states.
const idle: ApiState<string> = { status: "idle" };
const loading: ApiState<string> = { status: "loading" };
const success: ApiState<string> = { status: "success", data: "Hello", timestamp: new Date() };
const error: ApiState<string> = { status: "error", error: "Not found", code: 404 };

console.log(renderState(idle));     // "No data loaded."
console.log(renderState(loading));  // "Loading..."
console.log(renderState(success));  // "Loaded "Hello" at ..."
console.log(renderState(error));    // "Error 404: Not found"

// Step 4: Invalid states are impossible.
// const invalid: ApiState<string> = { status: "success" };
// ❌ Error: Property 'data' is missing in type '{ status: "success"; }'.

// Step 5: State transitions are type-safe.
function transitionToSuccess<T>(data: T): ApiState<T> {
  return { status: "success", data, timestamp: new Date() };
}

const newState = transitionToSuccess(42);
console.log(renderState(newState));  // "Loaded 42 at ..."
```

**Expected Output:**
```
No data loaded.
Loading...
Loaded "Hello" at ...
Error 404: Not found
Loaded 42 at ...
```

**Why This Output Occurs:** The `ApiState<T>` union models four mutually exclusive states. The `status` discriminant narrows to each state, enabling access to state-specific data. Invalid states (like `success` without `data`) are compile errors.

### Real-World Cases

**Case 1: React Query / SWR**
Data-fetching libraries model state as discriminated unions: idle, loading, success, error.

**Case 2: Redux State Machines**
Redux state is modeled as discriminated unions with action types as discriminants.

**Case 3: Form Validation**
Form validation states use literal unions: `"pristine" | "valid" | "invalid" | "submitting"`.

**Case 4: Authentication States**
Authentication uses discriminated unions: `{ status: "unauthenticated" } | { status: "authenticating" } | { status: "authenticated"; user: User }`.

---

## 4. Template Literal Types for Dynamic String Patterns

### Definitions

**Core Definition**
Template literal types (introduced in TypeScript 4.1) are literal types constructed from template literal syntax. They allow you to define types based on string patterns, such as `` `user-${string}` `` (any string starting with `"user-"`) or `` `${number}px` `` (any string ending with `"px"` and starting with a number).

**Technical Definition**
Template literal types extend literal types to string patterns using the template literal syntax. A template literal type is written as `` `prefix${Type}suffix` `` where `Type` is a type that can be interpolated. When the interpolated type is a union of literals, the template literal type expands to a union of all combinations. Template literal types support intrinsic string manipulation utilities: `Uppercase<T>`, `Lowercase<T>`, `Capitalize<T>`, and `Uncapitalize<T>`. They also support conditional type inference (`infer`) for pattern matching and extraction. Template literal types enable type-level string manipulation, pattern-based APIs, and type-safe string formatting.

**Beginner-Friendly Explanation**
Template literal types let you use string patterns as types. Instead of saying "any string," you can say `` `user-${string}` ``—meaning "any string that starts with `user-`." Or `` `${number}px` ``—meaning "a number followed by `px`." You can also use built-in utilities: `Uppercase<"hello">` gives `"HELLO"`, and `Capitalize<"hello">` gives `"Hello"`. Template literal types are powerful for creating type-safe APIs—like event names (`on${Capitalize<string>}`), CSS values (`${number}px`), or route patterns. They bring type-level string manipulation to TypeScript.

### Purposes

- To define types for strings that match specific patterns.
- To create type-safe event names, routes, and CSS values.
- To enable type-level string manipulation with `Uppercase`, `Lowercase`, `Capitalize`, `Uncapitalize`.
- To extract parts of strings using pattern matching with `infer`.
- To enforce naming conventions at compile time.

### Syntax Rules and Structure

**General Syntax: Template Literal Type**

```typescript
type AliasName = `prefix${Type}suffix`;
```

**Component Breakdown**
- `` `prefix${Type}suffix` ``: The template literal type.
- `prefix` / `suffix`: Literal strings.
- `${Type}`: The interpolated type (string, number, boolean, or union of literals).

**General Syntax: Union Expansion**

```typescript
type Result = `user-${"active" | "inactive"}`;
// Result = "user-active" | "user-inactive"
```

**Component Breakdown**
- Interpolating a union expands to a union of all combinations.

**General Syntax: Intrinsic String Utilities**

```typescript
type Upper = Uppercase<"hello">;    // "HELLO"
type Lower = Lowercase<"HELLO">;    // "hello"
type Cap = Capitalize<"hello">;     // "Hello"
type Uncap = Uncapitalize<"Hello">; // "hello"
```

**Component Breakdown**
- TypeScript provides four intrinsic string manipulation utilities.

**General Syntax: Pattern Matching with `infer`**

```typescript
type ExtractId<T> = T extends `user-${infer Id}` ? Id : never;
type Id = ExtractId<"user-123">;  // "123"
```

**Component Breakdown**
- `infer Id`: Captures the matched portion of the string.

**Syntax Rules**

- Template literal types use backticks and `${}` interpolation.
- Interpolated types must be assignable to `string | number | boolean | bigint | null | undefined`.
- Union interpolation expands to a union of all combinations.
- Template literal types can be nested.
- Intrinsic utilities (`Uppercase`, etc.) work on literal types.
- `infer` enables pattern extraction.
- Template literal types can be used in any type position.

**Constraints and Limitations**

- Union expansion can produce very large unions (combinatorial explosion).
- Template literal types are compile-time only; no runtime behavior.
- `infer` works only in conditional types.
- Intrinsic utilities do not work with generic types (only concrete literals).
- Template literal types cannot be used with `instanceof` or runtime checks.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Template Literal Types

```typescript
// Step 1: Define a template literal type for CSS units.
type CssUnit = "px" | "em" | "rem" | "%";
type CssValue = `${number}${CssUnit}`;

// Step 2: Use it.
const width: CssValue = "100px";   // ✅
const height: CssValue = "2.5rem"; // ✅
const margin: CssValue = "50%";    // ✅
// const invalid: CssValue = "100pt";  // ❌ Error: '"100pt"' is not assignable to CssValue.

// Step 3: Template literal types with unions.
type EventName = `on${Capitalize<"click" | "focus" | "blur">}`;
// EventName = "onClick" | "onFocus" | "onBlur"

function addListener(event: EventName, handler: () => void): void {
  console.log(`Adding listener for ${event}`);
}

addListener("onClick", () => {});  // ✅
addListener("onFocus", () => {});  // ✅
// addListener("onHover", () => {});  // ❌ Error: '"onHover"' is not assignable to EventName.

// Step 4: Pattern extraction with infer.
type ExtractRouteParam<T> = T extends `/users/${infer Id}` ? Id : never;
type UserId = ExtractRouteParam<"/users/123">;  // "123"
type NotFound = ExtractRouteParam<"/posts/456">; // never

const userId: UserId = "123";  // ✅
console.log(userId);  // "123"
```

**Expected Output:**
```
Adding listener for onClick
Adding listener for onFocus
123
```

**Why This Output Occurs:** The `CssValue` template literal type requires a number followed by a CSS unit. The `EventName` type uses `Capitalize` to create camelCase event names. The `ExtractRouteParam` type uses `infer` to extract the ID portion of a route pattern.

#### Example 2: Advanced Pattern Matching

```typescript
// Step 1: Define a type that extracts the prefix from a string.
type GetPrefix<T extends string> = T extends `${infer Prefix}-${string}` ? Prefix : never;

type P1 = GetPrefix<"user-123">;    // "user"
type P2 = GetPrefix<"order-456">;   // "order"
type P3 = GetPrefix<"noDash">;      // never

// Step 2: Define a type-safe API for prefixed IDs.
type PrefixedId<Prefix extends string> = `${Prefix}-${string}`;

function parseId<Prefix extends string>(
  id: PrefixedId<Prefix>,
  prefix: Prefix
): string {
  return id.slice(prefix.length + 1);
}

const userId: PrefixedId<"user"> = "user-123";  // ✅
const orderId: PrefixedId<"order"> = "order-456"; // ✅
// const invalid: PrefixedId<"user"> = "order-123";
// ❌ Error: '"order-123"' is not assignable to type '`user-${string}`'.

console.log(parseId(userId, "user"));    // "123"
console.log(parseId(orderId, "order"));  // "456"

// Step 3: Combining template literal types with mapped types.
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number; }

const getters: PersonGetters = {
  getName: () => "Alice",
  getAge: () => 30,
};

console.log(getters.getName());  // "Alice"
console.log(getters.getAge());   // 30
```

**Expected Output:**
```
123
456
Alice
30
```

**Why This Output Occurs:** The `GetPrefix` type uses `infer` to extract the portion before the dash. The `PrefixedId<Prefix>` type enforces the prefix pattern. The `Getters<T>` mapped type uses template literal types to transform property names into getter names.

### Real-World Cases

**Case 1: Type-Safe Event Emitters**
Event emitters use template literal types to type event names: `on${Capitalize<EventName>}`.

**Case 2: CSS-in-JS**
CSS-in-JS libraries use template literal types to type CSS values: `` `${number}px` ``, `` `${number}rem` ``.

**Case 3: Route Patterns**
Routing libraries use template literal types to type route parameters: `` `/users/${string}` ``.

**Case 4: i18n Keys**
Internationalization libraries use template literal types to type translation keys: `` `home.${string}` ``.

**Case 5: Getters and Setters**
Mapped types with template literal types generate getters and setters for object properties.

---

## 5. Type Widening and Controlling It Using `as const` (Const Assertions)

### Definitions

**Core Definition**
Type widening is TypeScript's process of broadening literal types to their corresponding primitive types in mutable contexts. The `as const` assertion (const assertion) prevents widening by preserving literal types and making objects deeply readonly. Const assertions, introduced in TypeScript 3.4, are the primary tool for controlling widening.

**Technical Definition**
Type widening occurs when TypeScript infers a type for a value in a mutable context: `let x = "hello"` infers `x` as `string` (widened from `"hello"`), while `const x = "hello"` infers `x` as `"hello"` (not widened). Object properties are widened even when assigned to `const` variables (`const obj = { status: "active" }` infers `status: string`). The `as const` assertion prevents widening for the entire expression, preserving all literal types and making all properties `readonly`. Const assertions can be applied to literals, object literals, and array literals. The `satisfies` operator (TypeScript 4.9+) provides an alternative that validates types without widening.

**Beginner-Friendly Explanation**
Type widening is TypeScript's way of being flexible: when you write `let status = "active"`, TypeScript assumes you might change `status` later, so it widens the type to `string`. This is usually helpful, but sometimes you want TypeScript to remember the exact value. The `as const` assertion tells TypeScript: "Don't widen this—keep the literal types." It also makes everything readonly. For example, `const config = { status: "active" } as const` gives `config` the type `{ readonly status: "active" }` instead of `{ status: string }`. This is essential for discriminated unions and type-level programming.

### Purposes

- To understand when and why TypeScript widens literal types.
- To prevent widening using `as const` for precise types.
- To create deeply readonly objects and arrays with `as const`.
- To preserve literal types for discriminated unions and template literal types.
- To control the trade-off between flexibility and precision.

### Syntax Rules and Structure

**General Syntax: Widening (Default)**

```typescript
let variable = "literal";  // Widened to string
const object = { prop: "literal" };  // prop widened to string
```

**Component Breakdown**
- `let` variables and object properties are widened to primitives.

**General Syntax: `as const` Assertion**

```typescript
const variable = "literal" as const;  // Literal type preserved
const object = { prop: "literal" } as const;  // prop: "literal", readonly
```

**Component Breakdown**
- `as const`: Prevents widening and makes the expression deeply readonly.

**General Syntax: `satisfies` Operator (TypeScript 4.9+)**

```typescript
const config = {
  status: "active",
} satisfies { status: string };
// config.status: "active" (preserved) while validated against the type
```

**Component Breakdown**
- `satisfies`: Validates against a type without widening.

**Syntax Rules**

- `let` and `var` declarations widen literal types to primitives.
- `const` declarations do not widen primitive literals but do widen object properties.
- Object and array literal properties are widened even under `const`.
- `as const` prevents widening and makes the expression deeply readonly.
- `as const` can be applied to literals, object literals, and array literals.
- `satisfies` validates a type without widening (TypeScript 4.9+).
- Widening also occurs in function return types unless annotated.

**Constraints and Limitations**

- `as const` makes the entire expression readonly, which may be too restrictive.
- `as const` cannot be combined with type annotations in the same declaration.
- `satisfies` requires TypeScript 4.9+.
- Widening behavior differs between `let`/`var` and `const`.
- Widening can be surprising in complex expressions.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Widening in Different Contexts

```typescript
// Step 1: `let` widens literals.
let a = "hello";        // string (widened)
let b = 42;             // number (widened)
let c = true;           // boolean (widened)

// Step 2: `const` preserves primitive literals.
const d = "hello";      // "hello" (preserved)
const e = 42;           // 42 (preserved)
const f = true;         // true (preserved)

// Step 3: `const` with object literals widens properties.
const obj = {
  status : "active",
  count  : 0,
};
// obj.status : string (widened!)
// obj.count  : number (widened!)

obj.status = "inactive";  // ✅ Allowed (widened)

// Step 4: `as const` preserves everything.
const constObj = {
  status : "active",
  count  : 0,
} as const;
// constObj.status : "active" (preserved)
// constObj.count  : 0 (preserved)

// constObj.status = "inactive";  // ❌ Error: Cannot assign to 'status' because it is a read-only property.

// Step 5: `satisfies` validates without widening.
const validated = {
  status : "active",
  count  : 0,
} satisfies { status: string; count: number };
// validated.status : "active" (preserved!)
// validated.count  : 0 (preserved!)

console.log(validated.status);  // "active"
```

**Expected Output:**
```
active
```

**Why This Output Occurs:** `let` and object properties are widened to primitives. `const` preserves primitive literals but not object properties. `as const` preserves everything and makes it readonly. `satisfies` validates against a type while preserving the inferred literal types.

#### Example 2: `as const` for Discriminated Unions

```typescript
// Step 1: Without as const, the discriminant is widened.
function createActionWithoutConst(type: string) {
  return { type };  // type: string (widened)
}

const action1 = createActionWithoutConst("ADD");
// action1: { type: string } — no literal preservation

// Step 2: With as const, the discriminant is preserved.
function createActionWithConst<T extends string>(type: T) {
  return { type } as const;  // type: T (preserved)
}

const action2 = createActionWithConst("ADD");
// action2: { readonly type: "ADD" }

// Step 3: Use with discriminated unions.
type Action =
  | { type: "ADD"; payload: string }
  | { type: "REMOVE"; id: number };

function createAddAction(payload: string): Action {
  return { type: "ADD", payload };  // ✅ Contextual typing narrows type
}

const addAction = createAddAction("New item");
console.log(addAction.type);  // "ADD"

// Step 4: as const for configuration objects.
const config = {
  apiUrl: "https://api.example.com",
  methods: ["GET", "POST"],
  timeout: 5000,
} as const;

// config.methods: readonly ["GET", "POST"]
// config.timeout: 5000
// config.apiUrl: "https://api.example.com"

type ApiUrl  = typeof config.apiUrl;           // "https://api.example.com"
type Methods = typeof config.methods[number];  // "GET" | "POST"

function request(url: ApiUrl, method: Methods): void {
  console.log(`${method} ${url}`);
}

request(config.apiUrl, config.methods[0]);  // "GET https://api.example.com"
// request("https://other.com", "GET");     // ❌ Error: not assignable to ApiUrl.
```

**Expected Output:**
```
ADD
GET https://api.example.com
```

**Why This Output Occurs:** Without `as const`, the action's `type` is widened to `string`, losing the literal. With `as const`, the literal is preserved. For the configuration object, `as const` preserves the literal API URL and the tuple of HTTP methods, enabling type-safe `request` calls.

### Real-World Cases

**Case 1: Redux Action Creators**
Redux action creators use `as const` to preserve action type literals for discriminated union narrowing.

**Case 2: Configuration Constants**
Application configuration uses `as const` to preserve exact values for type-safe consumption.

**Case 3: Route Definitions**
Route definitions use `as const` to preserve path literals and parameter names.

**Case 4: Design Tokens**
Design systems use `as const` to preserve color names, spacing values, and typography scales as literal types.

---

## References

- TypeScript Handbook: Literal Types — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types
- TypeScript Handbook: Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- TypeScript 4.1 Release Notes (Template Literal Types) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html
- TypeScript 3.4 Release Notes (Const Assertions) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html
- TypeScript 4.9 Release Notes (`satisfies` Operator) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html
- TypeScript Handbook: Narrowing (Literal Types) — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#literal-types
- TypeScript Playground: Literal Types — https://www.typescriptlang.org/play/typescript/primitives/literal-types.ts.html
- Effective TypeScript: Item 3 — Understanding Type Widening
- Effective TypeScript: Item 41 — Understand Evolving any
- Total TypeScript: Literal Types — https://www.totaltypescript.com/literal-types
- Total TypeScript: Template Literal Types — https://www.totaltypescript.com/template-literal-types
- MDN: Template Literals — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals
- TypeScript ESLint: no-unnecessary-type-assertion — https://typescript-eslint.io/rules/no-unnecessary-type-assertion/