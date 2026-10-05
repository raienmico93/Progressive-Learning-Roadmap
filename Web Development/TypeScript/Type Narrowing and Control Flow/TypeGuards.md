# TypeScript User-Defined Type Guards: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
User-defined type guards are functions that perform runtime validation and inform TypeScript's type checker that a value satisfies a specific type. They bridge the gap between runtime data (which TypeScript cannot statically verify) and compile-time type safety by encoding runtime checks as type predicates.

**Technical Definition**
User-defined type guards are functions whose return type is annotated as a type predicate (`parameterName is Type`). When the function returns `true`, TypeScript's control flow analysis narrows the parameter to the predicate type within the guarded branch. Assertion functions use the related `asserts parameterName is Type` or `asserts condition` return type to narrow types after the call, throwing on failure. Type guards are the only mechanism for narrowing to custom interfaces and types that lack built-in runtime guards (since `instanceof` only works with classes). They enable validation of external data (API responses, JSON, user input) at the boundary of the type system, converting `unknown` into verified types.

**Beginner-Friendly Explanation**
A user-defined type guard is a function you write to check if a value is a specific type. Instead of just returning `true` or `false`, you annotate it with `value is Type`, which tells TypeScript: "If I return true, this value is definitely of this type." This lets you write custom validation logic for your own interfaces and types, which built-in guards like `typeof` and `instanceof` can't handle. Type guards are essential for validating external data (like API responses) and converting untyped `unknown` data into safely typed values. Assertion functions are similar but instead of returning a boolean, they throw an error if the value doesn't match—narrowing the type for the rest of the scope.

### Key Characteristics

- **Runtime validation**: Type guards perform actual runtime checks on values.
- **Compile-time narrowing**: They inform TypeScript's type checker about the narrowed type.
- **Custom interfaces**: The only way to narrow to custom interfaces (since `instanceof` requires classes).
- **Boundary validation**: Essential for validating external data at the edges of the type system.
- **Composable**: Type guards can be combined and reused to build complex validation logic.
- **Two forms**: Type predicates (`value is Type`) return booleans; assertion functions (`asserts value is Type`) throw on failure.
- **TS 5.5+ inference**: Type predicates are automatically inferred for simple boolean-returning functions.

### Prerequisites

- Basic knowledge of union types and narrowing
- Familiarity with built-in type guards (`typeof`, `instanceof`, `in`)
- Understanding of TypeScript's type system and structural typing
- Basic familiarity with generics (for reusable guards)

### Related Programming Areas

- **Control Flow Analysis**: The engine that enables narrowing
- **Runtime Validation**: Schema validation libraries (Zod, io-ts) vs. hand-rolled type guards
- **API Boundary Validation**: Validating external data at system boundaries
- **Design Patterns**: Strategy, template method, and assertion patterns
- **Testing**: Type guards in test assertions and fixture validation

### Core Concepts / Features

1. Custom Validation Functions and Runtime-to-Compile-Time Type Safety
2. Assertion Functions (`asserts condition`, `asserts value is Type`)
3. Reusable Narrowing Logic and Higher-Order Guard Functions
4. Narrowing with Inline Callbacks and Array Filtering (`.filter(isDefined)`)


## 1. Custom Validation Functions and Runtime-to-Compile-Time Type Safety

### Definitions

**Core Definition**
Custom validation functions are user-written functions that check whether a runtime value satisfies a specific type. When annotated with a type predicate (`value is Type`), they enable TypeScript to narrow the value's type from a broader type (or `unknown`) to the specific type within the guarded branch.

**Technical Definition**
A custom validation function with a type predicate has the signature `function isType(value: unknown): value is SpecificType`. The function performs runtime checks (using `typeof`, `instanceof`, `in`, property access, and custom logic) and returns a boolean. The type predicate annotation tells TypeScript that a `true` return guarantees the value is `SpecificType`. TypeScript's control flow analysis uses this information to narrow the type in the calling scope. This is the primary mechanism for narrowing to custom interfaces, since `instanceof` requires classes and `typeof` only works with primitives. TypeScript 5.5+ automatically infers type predicates for functions that meet specific conditions: no explicit return type annotation, a single `return` statement, no parameter mutation, and a boolean expression tied to a parameter refinement.

**Beginner-Friendly Explanation**
A custom validation function is a function you write to check if something is the right type. Instead of just saying "this returns a boolean," you write `value is User`, which tells TypeScript: "If I return true, the value is definitely a User." This is how you validate your own types—like checking that an API response has a `name` property that's a string and an `id` property that's a number. Without type guards, you'd have to use unsafe `as` casts. With type guards, TypeScript verifies the type for you based on your runtime checks. TypeScript 5.5+ is smart enough to infer these predicates automatically for simple functions, but explicit annotations are still recommended for exported or complex guards.

### Purposes

- To validate external data (API responses, JSON, user input) against expected types.
- To narrow `unknown` or union types to custom interfaces and types.
- To replace unsafe type assertions (`as`) with checked runtime validation.
- To encapsulate complex validation logic in reusable functions.
- To enable type-safe processing of dynamic data at system boundaries.

### Syntax Rules and Structure

**General Syntax: Type Predicate Function**

```typescript
function isTypeName(value: unknown): value is TypeName {
  return /* runtime check */;
}
```

**Component Breakdown**
- `value: unknown`: The parameter being validated.
- `value is TypeName`: The type predicate return type.
- The function returns a boolean, but TypeScript uses the predicate for narrowing.

**General Syntax: Type Predicate with Generics**

```typescript
function isDefined<T>(value: T | null | undefined): value is T {
  return value !== null && value !== undefined;
}
```

**Component Breakdown**
- `<T>`: The generic type parameter.
- `value is T`: The predicate that preserves the generic type.

**General Syntax: Inferred Type Predicate (TypeScript 5.5+)**

```typescript
// No explicit return type annotation — TypeScript infers `value is string`
function isString(value: unknown) {
  return typeof value === "string";
}
```

**Component Breakdown**
- TS 5.5+ infers the type predicate automatically under specific conditions.

**Syntax Rules**

- The type predicate syntax is `parameterName is Type`.
- `parameterName` must be a parameter of the function.
- The function must return a boolean.
- The predicate type must be assignable to the parameter type.
- Type predicates are unchecked by the compiler—the developer is responsible for correctness.
- Generic type predicates can preserve type relationships.
- TypeScript 5.5+ infers predicates for simple boolean-returning functions.
- Type predicates work with `Array.filter()` to narrow array element types.

**Constraints and Limitations**

- Type predicates are unchecked: an incorrect predicate causes runtime errors.
- The parameter name must match exactly.
- Type predicates cannot be used with `this` parameters.
- Predicates on `unknown` require explicit property checks.
- TypeScript 5.5+ inference has specific conditions (single return, no mutation).
- Excessive `as` casts inside predicates may hide bugs.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Type Guard for an Interface

```typescript
// Step 1: Define the interface to validate.
interface User {
  id: number;
  name: string;
  email: string;
}

// Step 2: Write a custom type guard for the interface.
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    typeof (value as { id: unknown }).id === "number" &&
    "name" in value &&
    typeof (value as { name: unknown }).name === "string" &&
    "email" in value &&
    typeof (value as { email: unknown }).email === "string"
  );
}

// Step 3: Use the type guard to validate unknown data.
const apiResponse: unknown = { id: 1, name: "Alice", email: "alice@example.com" };

if (isUser(apiResponse)) {
  // apiResponse is narrowed to User
  console.log(`${apiResponse.name} (${apiResponse.email})`);
  // Accessing properties is type-safe
} else {
  console.log("Invalid user data");
}

// Step 4: Type predicate enables safe processing.
function processUser(data: unknown): string {
  if (isUser(data)) {
    return `User: ${data.name}`;
  }
  return "Not a user";
}

console.log(processUser({ id: 2, name: "Bob", email: "bob@example.com" }));
// "User: Bob"
console.log(processUser({ id: "3", name: "Charlie" }));
// "Not a user"
```

**Expected Output:**
```
Alice (alice@example.com)
User: Bob
Not a user
```

**Why This Output Occurs:** The `isUser` type guard checks all required properties with their types. When it returns `true`, TypeScript narrows `apiResponse` from `unknown` to `User`, enabling safe property access. The invalid object fails the check because `id` is a string and `email` is missing.

#### Example 2: TypeScript 5.5+ Inferred Type Predicates

```typescript
// Step 1: Without explicit predicate — TS 5.5+ infers it.
function isString(value: unknown) {
  return typeof value === "string";
}

// Step 2: The inferred type predicate works for narrowing.
function process(value: unknown): string {
  if (isString(value)) {
    // value is narrowed to string (inferred predicate)
    return value.toUpperCase();
  }
  return String(value);
}

console.log(process("hello"));  // "HELLO"
console.log(process(42));       // "42"

// Step 3: Inferred predicate for generic functions.
function isNonNullish<T>(value: T | null | undefined) {
  return value != null;
}

const values: (string | null | undefined)[] = ["a", null, "b", undefined, "c"];
const defined = values.filter(isNonNullish);
// defined: string[] (narrowed by inferred predicate)

console.log(defined);  // ["a", "b", "c"]

// Step 4: Explicit predicates are still recommended for exported guards.
export function isValidUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value
  );
}
```

**Expected Output:**
```
HELLO
42
[ 'a', 'b', 'c' ]
```

**Why This Output Occurs:** TypeScript 5.5+ infers the type predicate for `isString` because it has a single boolean return statement that refines the parameter. The `filter(isNonNullish)` call narrows the array to `string[]` because the inferred predicate tells TypeScript that `null` and `undefined` are excluded. The explicit `isValidUser` is exported and documented as a contract.

### Real-World Cases

**Case 1: API Response Validation**
APIs return `unknown` data from `fetch` or `JSON.parse`. Custom type guards validate the shape before using it, preventing runtime errors from unexpected response structures.

**Case 2: Configuration File Validation**
Configuration files (JSON, YAML) are parsed into `unknown` and validated with type guards before the application uses them.

**Case 3: Form Input Validation**
Form inputs from the DOM or external sources are validated with custom guards to ensure they match expected types before processing.

**Case 4: Local Storage Deserialization**
Data retrieved from `localStorage` is parsed from strings and validated with type guards before use.


## 2. Assertion Functions (`asserts condition`, `asserts value is Type`)

### Definitions

**Core Definition**
Assertion functions are functions that validate a condition or type at runtime and throw an error if the assertion fails. They use the `asserts` return type syntax (`asserts condition` or `asserts value is Type`) to tell TypeScript that if the function returns normally (without throwing), the asserted condition is true and the type is narrowed for the rest of the scope.

**Technical Definition**
Assertion functions (introduced in TypeScript 3.7) use the `asserts` keyword in their return type. Two forms exist: `asserts condition` asserts that a boolean condition is true, and `asserts parameterName is Type` asserts that a parameter is of a specific type. Both forms must throw on failure. After an assertion function call, TypeScript narrows the type in the calling scope for the remainder of that scope. This is more concise than manual `if (!condition) throw` patterns and produces more readable code. Assertion functions are particularly useful for shared preconditions across multiple functions—the assertion logic is written once and reused, with narrowing applied automatically.

**Beginner-Friendly Explanation**
An assertion function is like a type guard that throws an error instead of returning `false`. You write `asserts value is User`, and if the value isn't a User, your function throws an error. If it doesn't throw, TypeScript knows the value is a User for the rest of the function. This is cleaner than writing `if (!isUser(value)) throw new Error(...)` everywhere. Assertion functions are great for preconditions—checks that must pass before your code can continue. You write the assertion once and reuse it, and TypeScript narrows the type automatically after the call. There's also `asserts condition` for asserting that a boolean condition is true.

### Purposes

- To validate preconditions and narrow types in a single function call.
- To replace repetitive `if (!condition) throw` patterns with reusable assertions.
- To narrow types after a function call without explicit `if` blocks.
- To enforce invariants across multiple functions with shared assertion logic.
- To provide clear error messages when assertions fail.

### Syntax Rules and Structure

**General Syntax: `asserts value is Type`**

```typescript
function assertIsType(value: unknown): asserts value is Type {
  if (!/* check */) {
    throw new Error("Not the expected type");
  }
}

assertIsType(data);
// data is narrowed to Type for the rest of the scope
```

**Component Breakdown**
- `asserts value is Type`: The assertion return type.
- The function must throw on failure.
- After the call, `value` is narrowed to `Type`.

**General Syntax: `asserts condition`**

```typescript
function assert(condition: unknown, message?: string): asserts condition {
  if (!condition) {
    throw new Error(message ?? "Assertion failed");
  }
}

assert(value !== null, "Value is required");
// value is narrowed to exclude null
```

**Component Breakdown**
- `asserts condition`: Asserts the condition is truthy.
- After the call, TypeScript narrows based on the condition.

**General Syntax: Generic Assertion Function**

```typescript
function assertDefined<T>(value: T | null | undefined, name: string): asserts value is T {
  if (value === null || value === undefined) {
    throw new Error(`${name} is required`);
  }
}
```

**Component Breakdown**
- `<T>`: Generic type parameter.
- `asserts value is T`: Preserves the generic type after assertion.

**Syntax Rules**

- Assertion functions use `asserts parameterName is Type` or `asserts condition`.
- The function must throw on failure (not return `false`).
- After the call, the type is narrowed for the rest of the scope.
- Assertion functions have a `void` return type (they return nothing).
- They can be used in any scope: functions, methods, blocks.
- They work with discriminated unions and generics.
- Assertion functions are erased at compile time; the runtime behavior is the throw.

**Constraints and Limitations**

- Assertion functions must throw—they cannot return `false`.
- They cannot be used with `this` parameters.
- They do not work with `async` functions (narrowing does not persist across `await`).
- Assertion functions cannot be used with destructured parameters directly.
- They must be called as statements (not in expressions).
- Some environments (e.g., test frameworks) may prefer type predicates over assertions.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `asserts value is Type`

```typescript
// Step 1: Define an assertion function for a custom type.
interface User {
  id: number;
  name: string;
}

function assertIsUser(value: unknown): asserts value is User {
  if (
    typeof value !== "object" ||
    value === null ||
    !("id" in value) ||
    typeof (value as { id: unknown }).id !== "number" ||
    !("name" in value) ||
    typeof (value as { name: unknown }).name !== "string"
  ) {
    throw new Error("Value is not a User");
  }
}

// Step 2: Use the assertion to narrow a type.
function processData(data: unknown): string {
  assertIsUser(data);
  // data is narrowed to User for the rest of this function
  return `User ${data.id}: ${data.name}`;
}

console.log(processData({ id: 1, name: "Alice" }));
// "User 1: Alice"

// Step 3: Assertion throws on invalid data.
try {
  processData({ id: "1", name: "Bob" });
} catch (e) {
  console.log((e as Error).message);  // "Value is not a User"
}

// Step 4: Shared preconditions across functions.
function createUser(data: unknown): void {
  assertIsUser(data);
  console.log(`Creating user: ${data.name}`);
}

function updateUser(data: unknown): void {
  assertIsUser(data);
  console.log(`Updating user: ${data.name}`);
}

createUser({ id: 2, name: "Charlie" });  // "Creating user: Charlie"
updateUser({ id: 3, name: "Dave" });     // "Updating user: Dave"
```

**Expected Output:**
```
User 1: Alice
Value is not a User
Creating user: Charlie
Updating user: Dave
```

**Why This Output Occurs:** The `assertIsUser` function validates the shape and throws on failure. After the call, TypeScript narrows `data` to `User` for the rest of the function. The same assertion is reused across `createUser` and `updateUser`, eliminating duplicated check-and-throw patterns.

#### Example 2: `asserts condition`

```typescript
// Step 1: Define a generic assertion for conditions.
function assert(condition: unknown, message?: string): asserts condition {
  if (!condition) {
    throw new Error(message ?? "Assertion failed");
  }
}

// Step 2: Use `asserts condition` to narrow nullish values.
function processValue(value: string | null | undefined): string {
  assert(value != null, "Value is required");
  // value is narrowed to string
  return value.toUpperCase();
}

console.log(processValue("hello"));  // "HELLO"

try {
  processValue(null);
} catch (e) {
  console.log((e as Error).message);  // "Value is required"
}

// Step 3: Generic assertion function.
function assertDefined<T>(value: T | null | undefined, name: string): asserts value is T {
  if (value === null || value === undefined) {
    throw new Error(`${name} is required`);
  }
}

function processOrder(order: { id: number; items: string[] } | null): void {
  assertDefined(order, "order");
  // order is narrowed to { id: number; items: string[] }
  console.log(`Processing order ${order.id} with ${order.items.length} items`);
}

processOrder({ id: 1, items: ["item1", "item2"] });
// "Processing order 1 with 2 items"

// Step 4: Assertions in loops and nested scopes.
function validateAll(items: (string | null)[]): string[] {
  const result: string[] = [];
  for (const item of items) {
    assert(item != null, "Item is null");
    result.push(item.toUpperCase());  // item is narrowed to string
  }
  return result;
}

console.log(validateAll(["a", "b", "c"]));  // ["A", "B", "C"]
```

**Expected Output:**
```
HELLO
Value is required
Processing order 1 with 2 items
[ 'A', 'B', 'C' ]
```

**Why This Output Occurs:** The `assert` function with `asserts condition` narrows based on the condition. The `assertDefined` generic function preserves the type `T` after the assertion. Assertions work in loops and nested scopes, narrowing types within each iteration.

### Real-World Cases

**Case 1: Precondition Checks in Service Methods**
Service methods use assertion functions to validate inputs before processing, ensuring that null/undefined values are caught early with descriptive errors.

**Case 2: Express Middleware Validation**
Express middleware uses assertion functions to validate request bodies, narrowing `req.body` from `any` to typed objects for downstream handlers.

**Case 3: Test Fixtures**
Test utilities use assertion functions to validate test data, throwing descriptive errors when fixtures don't match expected shapes.

**Case 4: Configuration Loading**
Configuration loaders use assertion functions to validate that required settings are present, throwing clear errors at startup rather than failing later.


## 3. Reusable Narrowing Logic and Higher-Order Guard Functions

### Definitions

**Core Definition**
Reusable narrowing logic refers to the practice of extracting type guard logic into reusable, composable functions. Higher-order guard functions are functions that take type guards as arguments or return type guards, enabling the composition of complex validation from simpler pieces.

**Technical Definition**
Higher-order type guard functions operate on type predicates as first-class values. A function like `combine<T, U>(guard1: (x: unknown) => x is T, guard2: (x: unknown) => x is U): (x: unknown) => x is T & U` takes two type guards and returns a new guard that narrows to the intersection of both types. TypeScript supports higher-order type guard functions through generic type parameters and the `x is T` predicate syntax. The `Array.filter()` method's type definitions use this pattern: the overload `filter<S extends T>(predicate: (value: T) => value is S): S[]` narrows the array element type. Library utilities like `type-guard-helpers` and `is-kit` provide pre-built combinators for composing guards.

**Beginner-Friendly Explanation**
A higher-order type guard is a function that takes other type guards and combines them. For example, you could write a `combine` function that takes an `isString` guard and an `isNotEmpty` guard and returns a new guard that checks both. This lets you build complex validation from small, reusable pieces. Instead of writing one giant type guard for every combination, you compose small guards. TypeScript's `Array.filter()` uses this pattern: when you pass a type guard to `filter`, it narrows the array's element type. Higher-order guards make your validation logic DRY (Don't Repeat Yourself) and easier to test.

### Purposes

- To compose complex type guards from simpler, reusable pieces.
- To avoid duplicating validation logic across multiple guards.
- To create domain-specific guard combinators (AND, OR, NOT).
- To enable type-safe array filtering with narrowed element types.
- To build validation libraries and utilities with composable guards.

### Syntax Rules and Structure

**General Syntax: Higher-Order Guard Combinator**

```typescript
function combine<T, U>(
  guard1: (value: unknown) => value is T,
  guard2: (value: unknown) => value is U
): (value: unknown) => value is T & U {
  return (value: unknown): value is T & U => {
    return guard1(value) && guard2(value);
  };
}
```

**Component Breakdown**
- `guard1` / `guard2`: Type guard parameters.
- Returns a new type guard with the intersection predicate.

**General Syntax: Generic Guard Composer**

```typescript
function and<T, U>(
  first: (value: T) => value is U,
  second: (value: T) => value is T
): (value: T) => value is U {
  return (value: T): value is U => first(value) && second(value);
}
```

**Component Breakdown**
- Composes two guards with AND semantics.

**General Syntax: `Array.filter` with Type Guard**

```typescript
const numbers = [1, null, 2, undefined, 3].filter(isDefined);
// numbers: number[]
```

**Component Breakdown**
- The type guard `isDefined` narrows the array element type.

**Syntax Rules**

- Higher-order guards take type guard functions as parameters.
- They return new type guard functions with composed predicates.
- Generic type parameters preserve type relationships.
- `Array.filter` has a type guard overload that narrows the array type.
- Guard combinators can implement AND (`&&`), OR (`||`), and NOT (`!`) logic.
- Type predicates in returned guards must be compatible with the composed types.

**Constraints and Limitations**

- TypeScript does not always infer type predicates through higher-order functions.
- Generic composition can become complex; explicit annotations may be needed.
- Some combinators require explicit `is` annotations to work correctly.
- The `Array.filter` type guard overload only works when the predicate is a type guard.
- TypeScript 5.5+ improves inference for simple cases but not all complex compositions.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Higher-Order Guard Combinator

```typescript
// Step 1: Define basic type guards.
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNonEmpty(value: string): value is string {
  return value.length > 0;
}

// Step 2: Define a combinator that combines two guards.
function and<T, U extends T>(
  guard1: (value: unknown) => value is T,
  guard2: (value: T) => value is U
): (value: unknown) => value is U {
  return (value: unknown): value is U => guard1(value) && guard2(value);
}

// Step 3: Create a composed guard.
const isNonEmptyString = and(isString, isNonEmpty);
// isNonEmptyString: (value: unknown) => value is string

// Step 4: Use the composed guard.
function process(value: unknown): string {
  if (isNonEmptyString(value)) {
    return `Non-empty string: ${value.toUpperCase()}`;
  }
  return "Not a non-empty string";
}

console.log(process("hello"));  // "Non-empty string: HELLO"
console.log(process(""));       // "Not a non-empty string"
console.log(process(42));       // "Not a non-empty string"

// Step 5: Compose guards for objects.
interface HasId { id: number; }
interface HasName { name: string; }
type Entity = HasId & HasName;

function hasId(value: unknown): value is HasId {
  return typeof value === "object" && value !== null && "id" in value;
}

function hasName(value: unknown): value is HasName {
  return typeof value === "object" && value !== null && "name" in value;
}

const isEntity = and(hasId, hasName);
// isEntity: (value: unknown) => value is HasId & HasName

const entity: unknown = { id: 1, name: "Alice" };
if (isEntity(entity)) {
  console.log(`${entity.id}: ${entity.name}`);  // "1: Alice"
}
```

**Expected Output:**
```
Non-empty string: HELLO
Not a non-empty string
Not a non-empty string
1: Alice
```

**Why This Output Occurs:** The `and` combinator takes two type guards and returns a new guard that requires both to pass. `isNonEmptyString` combines `isString` and `isNonEmpty` to narrow to non-empty strings. `isEntity` combines `hasId` and `hasName` to narrow to the intersection `HasId & HasName`.

#### Example 2: `Array.filter` with Type Guards

```typescript
// Step 1: Define a type guard for defined values.
function isDefined<T>(value: T | null | undefined): value is T {
  return value !== null && value !== undefined;
}

// Step 2: Use it with Array.filter to narrow arrays.
const mixed: (string | null | undefined)[] = ["a", null, "b", undefined, "c"];
const defined = mixed.filter(isDefined);
// defined: string[]

console.log(defined);  // ["a", "b", "c"]

// Step 3: Combine with other operations.
const lengths = defined.map((s) => s.length);
console.log(lengths);  // [1, 1, 1]

// Step 4: Filter objects by type.
interface Cat { kind: "cat"; meow(): void; }
interface Dog { kind: "dog"; bark(): void; }
type Pet = Cat | Dog;

const pets: Pet[] = [
  { kind: "cat", meow: () => console.log("Meow") },
  { kind: "dog", bark: () => console.log("Woof") },
];

function isCat(pet: Pet): pet is Cat {
  return pet.kind === "cat";
}

const cats = pets.filter(isCat);
// cats: Cat[]
cats.forEach((cat) => cat.meow());  // "Meow"

// Step 5: With TypeScript 5.5+, inline predicates are inferred.
const numbers = [1, null, 2, undefined, 3].filter((n) => n !== null && n !== undefined);
// numbers: number[] (inferred by TS 5.5+)
console.log(numbers);  // [1, 2, 3]
```

**Expected Output:**
```
[ 'a', 'b', 'c' ]
[ 1, 1, 1 ]
Meow
[ 1, 2, 3 ]
```

**Why This Output Occurs:** The `isDefined` type guard narrows the array from `(string | null | undefined)[]` to `string[]`. The `isCat` guard narrows `Pet[]` to `Cat[]`. TypeScript 5.5+ infers the inline predicate for the last example, narrowing to `number[]`.

### Real-World Cases

**Case 1: Validation Libraries**
Libraries like `type-guard-helpers` and `is-kit` provide combinators for composing guards, enabling complex validation from simple pieces.

**Case 2: Array Processing**
Data processing pipelines use `filter(isDefined)` to remove nullish values and narrow array element types, eliminating runtime null checks.

**Case 3: Domain Validation**
Domain validation composes guards for different aspects of an entity (e.g., `isValidEmail` + `isNonEmptyString`) into a complete entity validator.

**Case 4: API Response Filtering**
API responses containing arrays of mixed types are filtered with type guards to produce typed subsets (e.g., filtering errors from a result array).


## 4. Narrowing with Inline Callbacks and Array Filtering (`.filter(isDefined)`)

### Definitions

**Core Definition**
Inline callback narrowing refers to using type guards as callbacks in array methods like `.filter()`, `.find()`, and `.some()`. When a type guard is passed to `.filter()`, TypeScript narrows the resulting array's element type based on the guard's predicate. The `.filter(isDefined)` pattern is the most common use case, removing `null` and `undefined` from arrays while narrowing the element type.

**Technical Definition**
`Array.prototype.filter` has a type guard overload: `filter<S extends T>(predicate: (value: T) => value is S): S[]`. When a type guard is passed as the predicate, TypeScript narrows the array from `T[]` to `S[]`, where `S` is the predicate type. This enables type-safe processing of arrays after filtering. The `isDefined` guard (`value is T`) is the canonical example: it removes nullish values and narrows the array. Before TypeScript 5.5, inline arrow functions with boolean returns were not inferred as type guards, requiring explicit named guards. TypeScript 5.5+ infers type predicates for simple inline callbacks that meet the inference conditions.

**Beginner-Friendly Explanation**
When you have an array with some `null` or `undefined` values, you can filter them out using `.filter(isDefined)`. This not only removes the nullish values but also tells TypeScript that the remaining array has no nullish values—so you don't need to check for `null` every time you access an element. Before TypeScript 5.5, you had to write a named `isDefined` function for this to work. Now, TypeScript can infer it for simple inline callbacks. This pattern is extremely common for processing API responses, form data, and any array that might contain nullish values.

### Purposes

- To remove nullish values from arrays and narrow the element type.
- To filter arrays by type, producing typed subsets.
- To enable type-safe processing of filtered arrays without null checks.
- To work with API responses that contain arrays of mixed types.
- To leverage TypeScript 5.5+ inference for inline callbacks.

### Syntax Rules and Structure

**General Syntax: `isDefined` Guard**

```typescript
function isDefined<T>(value: T | null | undefined): value is T {
  return value !== null && value !== undefined;
}

const defined = mixedArray.filter(isDefined);
// defined: T[]
```

**Component Breakdown**
- `value is T`: The predicate that excludes `null` and `undefined`.
- `.filter(isDefined)`: Narrows the array element type.

**General Syntax: Type-Specific Filter**

```typescript
function isCat(pet: Pet): pet is Cat {
  return pet.kind === "cat";
}

const cats = pets.filter(isCat);
// cats: Cat[]
```

**Component Breakdown**
- The guard narrows the array to the specific type.

**General Syntax: Inline Callback (TypeScript 5.5+)**

```typescript
const numbers = [1, null, 2, undefined, 3].filter((n) => n !== null && n !== undefined);
// numbers: number[] (TS 5.5+ inferred predicate)
```

**Component Breakdown**
- TypeScript 5.5+ infers the predicate for simple inline callbacks.

**Syntax Rules**

- `Array.filter` has a type guard overload that narrows the element type.
- The predicate must be a type guard (`value is T`).
- Named guards are recommended for reuse and clarity.
- TypeScript 5.5+ infers predicates for simple inline callbacks.
- The `isDefined` pattern is the canonical example for removing nullish values.
- Filtering with type guards produces a new array with the narrowed type.

**Constraints and Limitations**

- Inline callbacks are not always inferred as type guards (pre-5.5).
- TypeScript 5.5+ inference has specific conditions (single return, no mutation).
- Complex inline callbacks may not be inferred.
- `Array.filter` with a non-guard callback returns `T[]` (no narrowing).
- The `isDefined` guard does not preserve `null` or `undefined` intentionally.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `isDefined` Pattern

```typescript
// Step 1: Define the isDefined type guard.
function isDefined<T>(value: T | null | undefined): value is T {
  return value !== null && value !== undefined;
}

// Step 2: Filter an array with nullish values.
const rawData: (string | null | undefined)[] = ["a", null, "b", undefined, "c", null];

const cleaned = rawData.filter(isDefined);
// cleaned: string[]

console.log(cleaned);  // ["a", "b", "c"]
console.log(cleaned.map((s) => s.toUpperCase()));  // ["A", "B", "C"]

// Step 3: Works with objects.
interface User { name: string; }
const users: (User | null)[] = [
  { name: "Alice" },
  null,
  { name: "Bob" },
  null,
];

const validUsers = users.filter(isDefined);
// validUsers: User[]
validUsers.forEach((user) => console.log(user.name));
// "Alice"
// "Bob"

// Step 4: Combine with other operations.
const numbers: (number | undefined)[] = [1, undefined, 2, undefined, 3];
const doubled = numbers
  .filter(isDefined)
  .map((n) => n * 2);
console.log(doubled);  // [2, 4, 6]

// Step 5: TypeScript 5.5+ infers inline predicates.
const inferred = rawData.filter((item) => item !== null && item !== undefined);
// inferred: string[] (TS 5.5+)
console.log(inferred);  // ["a", "b", "c"]
```

**Expected Output:**
```
[ 'a', 'b', 'c' ]
[ 'A', 'B', 'C' ]
Alice
Bob
[ 2, 4, 6 ]
[ 'a', 'b', 'c' ]
```

**Why This Output Occurs:** The `isDefined` type guard removes `null` and `undefined` from the array, narrowing the element type to `string` or `User`. The `filter` method's type guard overload produces a narrowed array. TypeScript 5.5+ infers the inline predicate for the last example.

#### Example 2: Type-Specific Filtering

```typescript
// Step 1: Define a discriminated union.
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "rectangle"; width: number; height: number };

// Step 2: Define type guards for each variant.
function isCircle(shape: Shape): shape is { kind: "circle"; radius: number } {
  return shape.kind === "circle";
}

function isSquare(shape: Shape): shape is { kind: "square"; side: number } {
  return shape.kind === "square";
}

// Step 3: Filter the array by type.
const shapes: Shape[] = [
  { kind: "circle", radius: 5 },
  { kind: "square", side: 4 },
  { kind: "circle", radius: 3 },
  { kind: "rectangle", width: 2, height: 6 },
];

const circles = shapes.filter(isCircle);
// circles: { kind: "circle"; radius: number }[]
circles.forEach((c) => console.log(`Circle area: ${(Math.PI * c.radius ** 2).toFixed(2)}`));

const squares = shapes.filter(isSquare);
// squares: { kind: "square"; side: number }[]
squares.forEach((s) => console.log(`Square area: ${s.side ** 2}`));

// Step 4: Combine filters for complex processing.
const roundShapes = shapes.filter((s) => isCircle(s) || isSquare(s));
// roundShapes: Shape[] (union narrowed to circle | square)
console.log(`Round shapes: ${roundShapes.length}`);  // 3

// Step 5: Extract specific data after filtering.
const radii = shapes.filter(isCircle).map((c) => c.radius);
console.log(radii);  // [5, 3]
```

**Expected Output:**
```
Circle area: 78.54
Circle area: 28.27
Square area: 16
Round shapes: 3
[ 5, 3 ]
```

**Why This Output Occurs:** The `isCircle` and `isSquare` guards narrow the `Shape[]` array to the specific variant arrays. After filtering, the narrowed array's element type has the variant-specific properties (`radius`, `side`) accessible without further narrowing.

### Real-World Cases

**Case 1: API Response Processing**
API responses often contain arrays with optional or nullish elements. `.filter(isDefined)` removes nullish values and narrows the array for safe processing.

**Case 2: Form Data Cleanup**
Form data may contain empty or undefined fields. `.filter(isDefined)` cleans the data before validation or submission.

**Case 3: Event Handling**
Event listeners receive arrays of events with mixed types. Filtering with type guards produces typed event arrays for specific handlers.

**Case 4: Database Query Results**
Query results may contain null values for optional fields. Filtering with type guards produces clean, typed arrays for downstream processing.

**Case 5: React State Management**
React state often contains optional arrays. `.filter(isDefined)` cleans the state before rendering, ensuring type-safe access to elements.

---

## References

- TypeScript Handbook: Narrowing — https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- TypeScript Handbook: User-Defined Type Guards — https://www.typescriptlang.org/docs/handbook/advanced-types.html#user-defined-type-guards
- TypeScript 3.7 Release Notes (Assertion Functions) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html
- TypeScript 5.5 Release Notes (Inferred Type Predicates) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-5.html
- TypeScript 4.4 Release Notes (Control Flow Analysis of Aliased Conditions) — https://devblogs.microsoft.com/typescript/announcing-typescript-4-4/
- TypeScript 5.5 Release Notes (Inferred Type Predicates) — https://devblogs.microsoft.com/typescript/announcing-typescript-5-5/
- Total TypeScript: Inferred Type Predicates — https://www.totaltypescript.com/simplified-type-guards-with-typescript-5-5
- type-guard-helpers Documentation — https://www.npmjs.com/package/type-guard-helpers
- is-kit Documentation — https://www.npmjs.com/package/is-kit
- TypeScript ESLint: no-unnecessary-condition — https://typescript-eslint.io/rules/no-unnecessary-condition/
- MDN: Array.prototype.filter() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter
- MDN: typeof Operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof
- MDN: instanceof Operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof
- MDN: in Operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in
- Assertion Functions — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions
- Convex TypeScript Guide: Assert — https://www.convex.dev/typescript/assert
- Steve Kinney: Type Narrowing and Control Flow — https://stevekinney.com/courses/react-typescript/typescript-type-narrowing-control-flow