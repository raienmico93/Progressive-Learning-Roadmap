# TypeScript Union Types: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A union type in TypeScript is a type that represents a value that can be one of several possible types. Union types are written using the pipe operator (`|`) between two or more types. A value of a union type must be assignable to at least one of the constituent types.

**Technical Definition**
A union type is a TypeScript type construct that denotes the set-theoretic union of its constituent types. Given types `A` and `B`, the union `A | B` contains all values that are members of `A` or members of `B` (or both). TypeScript enforces that operations on a union-typed value are only permitted if they are valid for every constituent type—this is the "common properties" rule. To access type-specific members, the value must first be narrowed to one of the constituent types via type guards (`typeof`, `instanceof`, `in`, equality checks, or user-defined type predicates). Discriminated unions (tagged unions) use a shared literal property to enable exhaustive pattern matching and compile-time exhaustiveness checking via the `never` type.

**Beginner-Friendly Explanation**
A union type says "this value can be one of these types." For example, `string | number` means the value is either a string or a number. Think of it like a multiple-choice question: the answer is one of the options, but you don't know which until you check. TypeScript makes you check (narrow) the type before using type-specific features, which prevents runtime errors. Discriminated unions are a special pattern where each option has a "tag" (like `kind: "circle"`) that tells you which option you're dealing with. This lets you write safe pattern-matching code and ensures you handle every possible case.

### Key Characteristics

- **Set-theoretic union**: A union type contains all values of its constituent types.
- **Common properties rule**: Only operations valid for *all* constituents are allowed without narrowing.
- **Narrowing required**: Type-specific access requires narrowing via type guards.
- **Discriminated unions**: A shared literal property enables pattern matching.
- **Exhaustiveness checking**: The `never` type ensures all cases are handled.
- **Structural**: Union members are compared structurally, not nominally.
- **Compile-time only**: Union types are erased at runtime.

### Prerequisites

- Basic knowledge of TypeScript primitive and object types
- Familiarity with interfaces and type aliases
- Understanding of type annotations and inference
- Familiarity with control flow (`if`, `switch`) and type guards

### Related Programming Areas

- **Type Theory**: Union types are sum types (coproducts) in type theory
- **Algebraic Data Types**: Discriminated unions correspond to tagged unions in ML-family languages
- **Pattern Matching**: Discriminated unions enable pattern-matching-style code
- **Control Flow Analysis**: Narrowing is the mechanism for exploiting union types
- **Functional Programming**: Unions model "either/or" data with `Either`/`Result` types

### Core Concepts / Features

1. Union Syntax (`|`) and Conceptual Meaning
2. Multiple Possible Primitive and Object Types
3. Discriminated Unions (Tagged Unions) for Pattern Matching
4. Union-Compatible Operations and the "Common Properties" Rule
5. Narrowing Unions via Type Guards (`typeof`, `instanceof`, `in`, User-Defined Predicates)
6. Exhaustiveness Checking Using the `never` Type


## 1. Union Syntax (`|`) and Conceptual Meaning

### Definitions

**Core Definition**
Union syntax uses the pipe operator (`|`) to combine two or more types into a single type that accepts values of any of the constituent types. The union type's conceptual meaning is "one of these types."

**Technical Definition**
The union type operator (`|`) is a binary type operator that constructs the union of its operands. `A | B` is the type whose values are exactly the values that are members of `A` or members of `B`. Union types are normalized by TypeScript: duplicate members are removed, subtypes are absorbed by supertypes (e.g., `string | "hello"` simplifies to `string`), and `never` is absorbed into any union (`T | never` simplifies to `T`). Unions can combine any type: primitives, literals, object types, function types, tuples, and other unions. Union types are one of TypeScript's most fundamental type constructs and are essential for modeling values that can take multiple forms.

**Beginner-Friendly Explanation**
The pipe symbol (`|`) is TypeScript's way of saying "or." When you write `string | number`, you're saying "this value is either a string or a number." It's like a multiple-choice question. The union type doesn't tell you which option you have—just the possibilities. You can combine more than two types: `string | number | boolean` is a valid union. Unions are useful for modeling values that come from different sources (like API responses that can be success or error) or that can take different forms.

### Purposes

- To model values that can take one of several distinct forms.
- To represent "either/or" data without resorting to `any`.
- To combine literal types into named categories (e.g., `type Status = "active" | "inactive"`).
- To support discriminated unions for pattern matching.
- To enable type-safe handling of dynamic or polymorphic data.

### Syntax Rules and Structure

**General Syntax: Union Type**

```typescript
type AliasName = Type1 | Type2 | Type3;
```

**Component Breakdown**
- `Type1 | Type2 | Type3`: The union of the constituent types.
- A value of `AliasName` is assignable to at least one of the types.

**General Syntax: Inline Union Annotation**

```typescript
let variable: Type1 | Type2 = value;
```

**Component Breakdown**
- The union can be used directly in variable, parameter, or return type positions.

**General Syntax: Union of Literal Types**

```typescript
type Direction = "Up" | "Down" | "Left" | "Right";
```

**Component Breakdown**
- Each constituent is a string (or numeric) literal type.

**Syntax Rules**

- The `|` operator joins two or more types into a union.
- Union members can be any valid type: primitives, objects, literals, functions, tuples, arrays.
- Duplicate members are removed (e.g., `string | string` becomes `string`).
- Subtypes are absorbed by supertypes (e.g., `string | "hello"` becomes `string`).
- `never` is absorbed (e.g., `T | never` becomes `T`).
- Union types can be nested: `(A | B) | C` is equivalent to `A | B | C`.
- Type aliases are often used to name union types for reuse.

**Constraints and Limitations**

- Union types are erased at runtime; they exist only in the type system.
- Only common properties of all members are accessible without narrowing.
- Excess property checking behavior with unions can be surprising (improved in TypeScript 3.5).
- Very large unions can slow down the compiler.
- Union types cannot have methods or implementations.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Union Types

```typescript
// Step 1: Declare a union type for an ID that can be a number or string.
type ID = number | string;

// Step 2: Create values of the union type.
const numericId: ID = 123;
const stringId: ID = "abc-123";
console.log(numericId);  // 123
console.log(stringId);   // "abc-123"

// Step 3: Both are assignable to the union type.
function printId(id: ID): void {
  console.log(`ID: ${id}`);
}

printId(456);       // "ID: 456"
printId("xyz-789"); // "ID: xyz-789"

// Step 4: Assigning other types is an error.
// const boolId: ID = true;  // ❌ Error: Type 'boolean' is not assignable to type 'ID'.

// Step 5: Narrowing is required to use type-specific methods.
function formatId(id: ID): string {
  if (typeof id === "number") {
    return id.toFixed(0);  // ✅ Number method
  }
  return id.toUpperCase();  // ✅ String method
}

console.log(formatId(42));        // "42"
console.log(formatId("abc-123")); // "ABC-123"
```

**Expected Output:**
```
123
abc-123
ID: 456
ID: xyz-789
42
ABC-123
```

**Why This Output Occurs:** The `ID` type accepts both `number` and `string`. Inside `formatId`, TypeScript narrows `id` based on the `typeof` check—`number` in the first branch and `string` after the return. Type-specific methods (`toFixed`, `toUpperCase`) are only accessible after narrowing.

#### Example 2: Union Normalization

```typescript
// Step 1: TypeScript normalizes union types.
type A = string | string;           // Simplifies to string
type B = string | "hello";          // Simplifies to string (string absorbs literal)
type C = number | never;            // Simplifies to number (never absorbed)
type D = "a" | "b" | "a";           // Simplifies to "a" | "b" (duplicates removed)

// Step 2: Verify normalization with assignments.
const a: A = "anything";            // ✅
const b: B = "world";               // ✅
const c: C = 42;                    // ✅
const d: D = "a";                   // ✅

// Step 3: Nested unions flatten.
type E = (string | number) | boolean;
type F = string | number | boolean;
// E and F are equivalent.

const e: E = true;
const f: F = true;
console.log(e === f);  // true

// Step 4: Literal unions preserve distinct literals.
type Status = "active" | "inactive" | "pending";
const status: Status = "active";
console.log(status);  // "active"
```

**Expected Output:**
```
true
active
```

**Why This Output Occurs:** TypeScript normalizes unions by removing duplicates, absorbing subtypes into supertypes, and eliminating `never`. This means `string | "hello"` is simply `string` because `"hello"` is a subtype of `string`. Literal unions like `Status` are preserved because none of the literals are subtypes of each other.

### Real-World Cases

**Case 1: API Response IDs**
APIs often return IDs as either numbers or strings depending on the backend. Union types model this: `type ID = number | string`.

**Case 2: Configuration Values**
Configuration values can be strings, numbers, booleans, or arrays. Union types model these possibilities: `type ConfigValue = string | number | boolean | string[]`.

**Case 3: Status Literals**
Status fields are naturally union types: `type Status = "active" | "inactive" | "pending" | "deleted"`.

---

## 2. Multiple Possible Primitive and Object Types

### Definitions

**Core Definition**
Union types can combine primitive types, object types, or both. When a union contains object types, TypeScript allows access only to properties common to all object members (the "common properties" rule). Primitive members provide their own set of allowed operations.

**Technical Definition**
A union type can include any combination of primitives (`string`, `number`, `boolean`, `bigint`, `symbol`, `null`, `undefined`) and object types (interfaces, classes, object literals, arrays, tuples, functions). When the union includes object types, member access is restricted to properties present on *all* object members with compatible types. If members have no common properties, accessing any property is a compile error. This restriction is a direct consequence of type soundness: since the value could be any member, only operations valid for all members are safe.

**Beginner-Friendly Explanation**
A union can mix primitives and objects. For example, `string | { name: string }` means the value is either a string or an object with a `name`. But TypeScript is strict: you can only access properties that *all* members have. If one member is a string and another is an object, there are no common properties, so you can't access anything until you narrow. This prevents runtime errors—you can't assume a value has a `name` property if it might be a string. Narrow first, then access.

### Purposes

- To model values that can be primitives or objects depending on context.
- To combine multiple object shapes into a single type.
- To enforce safe access by requiring narrowing before member access.
- To represent API responses that can be objects, arrays, or primitives.
- To model flexible configuration values with mixed types.

### Syntax Rules and Structure

**General Syntax: Primitive Union**

```typescript
type PrimitiveUnion = string | number | boolean | null | undefined;
```

**Component Breakdown**
- The union contains only primitive types.

**General Syntax: Object Union**

```typescript
type ObjectUnion =
  | { name: string; email: string }
  | { name: string; phone: string };
```

**Component Breakdown**
- Each member is an object type.
- Common properties (`name`) are accessible without narrowing.
- Unique properties (`email`, `phone`) require narrowing.

**General Syntax: Mixed Union**

```typescript
type MixedUnion = string | number | { name: string };
```

**Component Breakdown**
- The union mixes primitives and objects.

**Syntax Rules**

- Union members can be any combination of primitives and objects.
- Member access is restricted to properties present on all object members.
- If members have no common properties, no property access is allowed without narrowing.
- Type-specific operations (e.g., string methods, object methods) require narrowing.
- `null` and `undefined` in unions require narrowing under `strictNullChecks`.
- Object literals assigned to union types are subject to excess property checking (improved in TypeScript 3.5+).

**Constraints and Limitations**

- Accessing properties not common to all members is a compile error without narrowing.
- Excess property checking with object unions can be confusing (mitigated in TS 3.5+).
- Large unions of object types can become unwieldy; consider discriminated unions.
- Union types cannot be used with `keyof` in all cases (distributes over members).
- Type assertions can bypass narrowing but are unsafe.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Primitive Union Operations

```typescript
// Step 1: Define a union of primitives.
type Value = string | number | boolean;

// Step 2: Common operations across all primitives.
function stringify(value: Value): string {
  return String(value);  // String() works on all three
}

console.log(stringify("hello"));  // "hello"
console.log(stringify(42));       // "42"
console.log(stringify(true));     // "true"

// Step 3: Type-specific operations require narrowing.
function describe(value: Value): string {
  if (typeof value === "string") {
    return `String of length ${value.length}`;
  }
  if (typeof value === "number") {
    return `Number with value ${value.toFixed(2)}`;
  }
  return `Boolean: ${value}`;
}

console.log(describe("hello"));  // "String of length 5"
console.log(describe(3.14159));  // "Number with value 3.14"
console.log(describe(true));     // "Boolean: true"

// Step 4: Direct property access without narrowing fails.
// function bad(value: Value): number {
//   return value.length;  // ❌ Error: Property 'length' does not exist on type 'Value'.
// }
```

**Expected Output:**
```
hello
42
true
String of length 5
Number with value 3.14
Boolean: true
```

**Why This Output Occurs:** `String(value)` is a common operation that works on all primitives, so it's allowed without narrowing. Accessing `value.length` is only valid for strings, so it requires narrowing via `typeof`.

#### Example 2: Object Union with Common Properties

```typescript
// Step 1: Define a union of object types with a common property.
type Contact =
  | { name: string; email: string; phone?: never }
  | { name: string; phone: string; email?: never };

// Step 2: Access common properties directly.
function getContactName(contact: Contact): string {
  return contact.name;  // ✅ `name` is common to both members
}

// Step 3: Access unique properties requires narrowing.
function getContactInfo(contact: Contact): string {
  if ("email" in contact && contact.email) {
    return `Email: ${contact.email}`;
  }
  if ("phone" in contact && contact.phone) {
    return `Phone: ${contact.phone}`;
  }
  return "No contact info";
}

const emailContact: Contact = { name: "Alice", email: "alice@example.com" };
const phoneContact: Contact = { name: "Bob", phone: "555-0123" };

console.log(getContactName(emailContact));   // "Alice"
console.log(getContactName(phoneContact));   // "Bob"
console.log(getContactInfo(emailContact));   // "Email: alice@example.com"
console.log(getContactInfo(phoneContact));   // "Phone: 555-0123"

// Step 4: Accessing a non-common property fails without narrowing.
// function bad(contact: Contact): string {
//   return contact.email;  // ❌ Error: Property 'email' does not exist on type 'Contact'.
// }
```

**Expected Output:**
```
Alice
Bob
Email: alice@example.com
Phone: 555-0123
```

**Why This Output Occurs:** The `Contact` union has two members, both with `name`. The `name` property is common, so it's accessible directly. The `email` and `phone` properties are unique to one member each, so narrowing via the `in` operator is required. The `never` type on the mutually exclusive properties prevents both from being present.

### Real-World Cases

**Case 1: JSON Values**
`JSON.parse` returns values that can be primitives, arrays, or objects. Union types model this: `type Json = string | number | boolean | null | Json[] | { [key: string]: Json }`.

**Case 2: Form Field Values**
Form fields can be strings, numbers, booleans, or arrays of strings. Union types model these possibilities.

**Case 3: Configuration Schemas**
Configuration schemas allow values of different types depending on the setting. Union types capture this flexibility.

---

## 3. Discriminated Unions (Tagged Unions) for Pattern Matching

### Definitions

**Core Definition**
A discriminated union (also called a tagged union or algebraic data type) is a union type where each member has a common literal property (the "discriminant" or "tag") that uniquely identifies that member. This enables type-safe pattern matching and exhaustive checking.

**Technical Definition**
A discriminated union is a union of object types that share a common property whose type is a distinct literal type in each member. This literal property (the discriminant) allows TypeScript to narrow the union to a specific member based on the discriminant's value. Discriminated unions are TypeScript's primary mechanism for modeling algebraic data types and are typically combined with `switch` statements and exhaustiveness checking via `never`. The discriminant must be a literal type (string, number, or boolean literal) and must be unique across members for effective narrowing.

**Beginner-Friendly Explanation**
A discriminated union is a union where each option has a "tag" that says which option it is. For example, a `Shape` might be `{ kind: "circle", radius: number }` or `{ kind: "square", side: number }`. The `kind` property is the tag. When you check `shape.kind === "circle"`, TypeScript knows the shape is a circle and gives you access to `radius`. This is the TypeScript equivalent of pattern matching in languages like Rust or Haskell. It's one of the most powerful features for modeling complex data safely.

### Purposes

- To model algebraic data types (sum types) with distinct variants.
- To enable type-safe pattern matching based on a discriminant property.
- To support exhaustive checking via the `never` type.
- To replace error-prone flags and optional properties with explicit variants.
- To make state machines, API responses, and event systems type-safe.

### Syntax Rules and Structure

**General Syntax: Discriminated Union**

```typescript
type AliasName =
  | { kind: "variant1"; property1: Type1 }
  | { kind: "variant2"; property2: Type2 }
  | { kind: "variant3"; property3: Type3 };
```

**Component Breakdown**
- `kind`: The discriminant property (a literal type unique to each member).
- `"variant1"`, `"variant2"`, `"variant3"`: The discriminant values.
- Each member has its own unique properties.

**General Syntax: Pattern Matching with `switch`**

```typescript
function process(value: AliasName): ReturnType {
  switch (value.kind) {
    case "variant1":
      return use(value.property1);
    case "variant2":
      return use(value.property2);
    case "variant3":
      return use(value.property3);
  }
}
```

**Component Breakdown**
- `switch (value.kind)`: Matches on the discriminant.
- Each case narrows `value` to the corresponding member.

**General Syntax: Pattern Matching with `if`**

```typescript
if (value.kind === "variant1") {
  // value is narrowed to { kind: "variant1"; property1: Type1 }
}
```

**Component Breakdown**
- The equality check on the discriminant narrows the type.

**Syntax Rules**

- The discriminant property must have a literal type in each member.
- Discriminant values must be unique across members (for effective narrowing).
- The discriminant property name must be the same across all members.
- Common properties can appear in all members.
- Narrowing works with `===`, `!==`, `switch`, and the `in` operator.
- Exhaustiveness checking uses `never` in the `default` case.

**Constraints and Limitations**

- The discriminant must be a literal type (string, number, boolean literal).
- Discriminant values must be unique for reliable narrowing.
- Excess property checking with discriminated unions can be strict (provide only the matching member's properties).
- Adding a new variant requires updating all exhaustive consumers.
- Discriminated unions do not work well with computed discriminants (dynamic values).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Shape Discriminated Union

```typescript
// Step 1: Define a discriminated union.
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number }
  | { kind: "rectangle"; width: number; height: number };

// Step 2: Calculate area with pattern matching.
function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    case "rectangle":
      return shape.width * shape.height;
  }
}

// Step 3: Test each variant.
console.log(area({ kind: "circle", radius: 5 }).toFixed(2));       // "78.54"
console.log(area({ kind: "square", sideLength: 4 }));              // 16
console.log(area({ kind: "rectangle", width: 3, height: 6 }));     // 18

// Step 4: Discriminant narrowing works with `if` too.
function describe(shape: Shape): string {
  if (shape.kind === "circle") {
    return `Circle with radius ${shape.radius}`;
  }
  if (shape.kind === "square") {
    return `Square with side ${shape.sideLength}`;
  }
  return `Rectangle ${shape.width}x${shape.height}`;
}

console.log(describe({ kind: "circle", radius: 5 }));  // "Circle with radius 5"

// Step 5: Invalid combinations are caught.
// const invalid: Shape = { kind: "circle", sideLength: 4 };
// ❌ Error: Object literal may only specify known properties,
// and 'sideLength' does not exist in type '{ kind: "circle"; radius: number; }'.
```

**Expected Output:**
```
78.54
16
18
Circle with radius 5
```

**Why This Output Occurs:** The `kind` property is the discriminant. TypeScript narrows `shape` based on `shape.kind`, allowing access to variant-specific properties. The `switch` statement handles each case, and TypeScript verifies that all cases are covered.

#### Example 2: API Result Discriminated Union

```typescript
// Step 1: Define a discriminated union for API results.
type ApiResult<T> =
  | { status: "success"; data: T }
  | { status: "error"; error: string; code: number }
  | { status: "loading" };

// Step 2: Process results with pattern matching and exhaustiveness.
function handleResult<T>(result: ApiResult<T>): string {
  switch (result.status) {
    case "success":
      return `Data: ${JSON.stringify(result.data)}`;
    case "error":
      return `Error ${result.code}: ${result.error}`;
    case "loading":
      return "Loading...";
    default:
      // Exhaustiveness check
      const _exhaustive: never = result;
      return _exhaustive;
  }
}

// Step 3: Test each variant.
const success: ApiResult<{ name: string }> = { status: "success", data: { name: "Alice" } };
const error: ApiResult<{ name: string }> = { status: "error", error: "Not found", code: 404 };
const loading: ApiResult<{ name: string }> = { status: "loading" };

console.log(handleResult(success));  // 'Data: {"name":"Alice"}'
console.log(handleResult(error));    // "Error 404: Not found"
console.log(handleResult(loading));  // "Loading..."

// Step 4: Adding a new variant requires updating consumers.
// If we add { status: "cancelled"; reason: string }, the `handleResult`
// function will produce a compile error in the default case until the new
// variant is handled.
```

**Expected Output:**
```
Data: {"name":"Alice"}
Error 404: Not found
Loading...
```

**Why This Output Occurs:** The `ApiResult<T>` union has three variants distinguished by the `status` discriminant. The `switch` statement handles each variant, narrowing the type appropriately. The `never` check in the `default` case ensures all variants are handled.

### Real-World Cases

**Case 1: Redux Actions**
Redux actions are naturally discriminated unions: `{ type: "ADD_TODO"; payload: Todo } | { type: "REMOVE_TODO"; id: number }`. The `type` field is the discriminant.

**Case 2: State Machines**
State machines model states as discriminated unions: `{ state: "idle" } | { state: "loading"; progress: number } | { state: "success"; data: T } | { state: "error"; error: Error }`.

**Case 3: AST Nodes**
Compilers represent abstract syntax tree nodes as discriminated unions: `{ kind: "NumberLiteral"; value: number } | { kind: "BinaryExpression"; left: Node; right: Node; operator: string }`.

**Case 4: API Responses**
API responses are modeled as discriminated unions: success, error, loading, and possibly other states.

---

## 4. Union-Compatible Operations and the "Common Properties" Rule

### Definitions

**Core Definition**
The "common properties" rule states that when you have a value of a union type, you can only access properties and methods that are common to *all* members of the union. Type-specific members require narrowing first. This rule ensures type safety: since you don't know which member the value is, only operations valid for all members are safe.

**Technical Definition**
For a union type `T = T1 | T2 | ... | Tn`, member access `value.property` is allowed only if `property` exists on every `Ti` with a compatible type. The type of the accessed property is the union of its types across all members. Method calls follow the same rule: the method must exist on all members, and its parameters must accept arguments valid for all members. If the property exists on only some members, accessing it is a compile error (unless the value is narrowed). This rule applies recursively to nested property access.

**Beginner-Friendly Explanation**
The common properties rule is TypeScript's way of keeping you safe. If you have a value that could be a string or an object, you can't access `.length` because it only exists on strings—the value might be an object. You can only use operations that work for *everything* in the union. Once you narrow (check which type it is), you can use that type's specific features. This rule prevents the classic runtime error where you assume a value has a property it doesn't.

### Purposes

- To guarantee type safety by preventing access to members that may not exist.
- To encourage explicit narrowing before type-specific operations.
- To allow safe access to shared properties without narrowing.
- To model "interface-like" common behavior across union members.
- To catch potential runtime errors at compile time.

### Syntax Rules and Structure

**General Syntax: Common Property Access**

```typescript
type Union = { common: string; a: number } | { common: string; b: boolean };

function use(value: Union): string {
  return value.common;  // ✅ Allowed — `common` exists on both
}
```

**Component Breakdown**
- `value.common`: Accessible because both union members have `common`.

**General Syntax: Type-Specific Access Requires Narrowing**

```typescript
function use(value: Union): number | boolean {
  // return value.a;  // ❌ Error: Property 'a' does not exist on the second member.
  if ("a" in value) {
    return value.a;  // ✅ Narrowed to the first member
  }
  return value.b;    // ✅ Narrowed to the second member
}
```

**Component Breakdown**
- Type-specific properties require narrowing via type guards.

**Syntax Rules**

- Property access on a union requires the property to exist on all members.
- The accessed property's type is the union of its types across members.
- Method calls require the method on all members with compatible signatures.
- Index access follows the same rules (index signature must exist on all members).
- If no properties are common, no property access is allowed without narrowing.
- Narrowing (via `typeof`, `instanceof`, `in`, equality, custom predicates) unlocks member access.

**Constraints and Limitations**

- Common property types may be wider than desired (union of member types).
- Calling methods with union-typed parameters can be tricky (arguments must be valid for all members).
- The rule can be frustrating when members share a structural property but not its type.
- `keyof` on a union distributes, giving the union of keys (not the intersection).
- Widening of `this` in method calls within unions can be surprising.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Common Properties in Object Unions

```typescript
// Step 1: Define a union with a common property.
type Vehicle =
  | { type: "car"; wheels: 4; doors: number }
  | { type: "motorcycle"; wheels: 2; helmetRequired: boolean }
  | { type: "truck"; wheels: 6; payloadCapacity: number };

// Step 2: Access common properties directly.
function getWheels(vehicle: Vehicle): number {
  return vehicle.wheels;  // ✅ `wheels` exists on all members
}

// Step 3: Access the discriminant directly.
function getType(vehicle: Vehicle): string {
  return vehicle.type;  // ✅ `type` exists on all members
}

// Step 4: Type-specific properties require narrowing.
function describe(vehicle: Vehicle): string {
  // return vehicle.doors;  // ❌ Error: Property 'doors' does not exist on all members.
  switch (vehicle.type) {
    case "car":
      return `Car with ${vehicle.doors} doors`;  // ✅ Narrowed
    case "motorcycle":
      return `Motorcycle (helmet ${vehicle.helmetRequired ? "required" : "optional"})`;
    case "truck":
      return `Truck with ${vehicle.payloadCapacity}kg capacity`;
  }
}

console.log(getWheels({ type: "car", wheels: 4, doors: 4 }));  // 4
console.log(describe({ type: "motorcycle", wheels: 2, helmetRequired: true }));
// "Motorcycle (helmet required)"
```

**Expected Output:**
```
4
Motorcycle (helmet required)
```

**Why This Output Occurs:** The `wheels` and `type` properties are common to all three members, so they're accessible directly. The `doors`, `helmetRequired`, and `payloadCapacity` properties are unique to specific members, so narrowing is required. The `switch` on `type` narrows the union to the appropriate member.

#### Example 2: No Common Properties

```typescript
// Step 1: Define a union with no common properties.
type Mixed = string | number | { name: string };

// Step 2: No property access is allowed without narrowing.
// function bad(value: Mixed): void {
//   console.log(value.length);  // ❌ Error: Property 'length' does not exist on type 'Mixed'.
//   console.log(value.toFixed); // ❌ Error: Property 'toFixed' does not exist on type 'Mixed'.
// }

// Step 3: Narrowing unlocks member access.
function process(value: Mixed): string {
  if (typeof value === "string") {
    return `String: ${value.toUpperCase()}`;
  }
  if (typeof value === "number") {
    return `Number: ${value.toFixed(2)}`;
  }
  return `Object with name: ${value.name}`;
}

console.log(process("hello"));                   // "String: HELLO"
console.log(process(3.14159));                   // "Number: 3.14"
console.log(process({ name: "Alice" }));         // "Object with name: Alice"

// Step 4: Type assertions can bypass the rule (unsafe).
function unsafe(value: Mixed): number {
  return (value as string).length;  // ⚠️ Unsafe — may fail at runtime
}

console.log(unsafe("hello"));  // 5
// console.log(unsafe(42));    // Runtime: undefined (42.length is undefined)
```

**Expected Output:**
```
String: HELLO
Number: 3.14
Object with name: Alice
5
```

**Why This Output Occurs:** The `Mixed` union has no common properties across all members, so no property access is allowed without narrowing. The type guards (`typeof`) narrow the union to each member, enabling access to `toUpperCase`, `toFixed`, and `name`. The `as string` assertion bypasses the common properties rule but is unsafe (calling `.length` on a number returns `undefined`).

### Real-World Cases

**Case 1: Event Systems**
Events often have common properties (`type`, `timestamp`) and type-specific properties (`target`, `key`, `position`) that require narrowing.

**Case 2: Form Field Values**
Form fields share common properties (`name`, `required`) but have type-specific properties (`maxLength` for text, `step` for numbers).

**Case 3: Notification Types**
Notifications share `id`, `timestamp`, and `read` but have type-specific fields (`emailSubject`, `smsNumber`, `pushIcon`).

---

## 5. Narrowing Unions via Type Guards (`typeof`, `instanceof`, `in`, and User-Defined Predicates)

### Definitions

**Core Definition**
Narrowing is the process by which TypeScript refines a union type to a more specific member type based on runtime checks (type guards). Type guards are expressions that, when true, tell TypeScript that a value is of a specific type. TypeScript supports built-in guards (`typeof`, `instanceof`, `in`, equality) and user-defined type predicates.

**Technical Definition**
Type narrowing occurs through control flow analysis: TypeScript tracks the possible types of a value at each point in the program and refines them based on type guards. The built-in guards are:
- `typeof value === "string"` (and other `typeof` results)
- `value instanceof ClassName`
- `"property" in value`
- `value === literal` (equality with literals)
- `value == null` / `value != null` (nullish checks)
- Truthiness checks (`if (value)`)

User-defined type predicates are functions with the return type `value is TypeName`, which TypeScript uses for narrowing. The `asserts value is TypeName` return type (TypeScript 3.7+) provides assertion functions that narrow in the calling scope.

**Beginner-Friendly Explanation**
Narrowing is how TypeScript figures out which member of a union you're working with. A type guard is a check that tells TypeScript "in this branch, the value is this specific type." For example, `typeof value === "string"` narrows `string | number` to `string` inside the `if` block. Built-in guards cover most cases: `typeof` for primitives, `instanceof` for classes, `in` for object properties. For custom logic, you write a type predicate: a function that returns `value is TypeName`. Narrowing is what makes unions usable—without it, you couldn't access type-specific features.

### Purposes

- To refine union types to specific members based on runtime checks.
- To enable safe access to type-specific properties and methods.
- To support discriminated union pattern matching.
- To encapsulate complex narrowing logic in reusable predicates.
- To enable exhaustiveness checking with `never`.

### Syntax Rules and Structure

**General Syntax: `typeof` Guard**

```typescript
if (typeof value === "string") {
  // value is narrowed to string
}
```

**Component Breakdown**
- `typeof value`: The JavaScript `typeof` operator.
- `=== "string"`: The comparison against a `typeof` result.

**General Syntax: `instanceof` Guard**

```typescript
if (value instanceof ClassName) {
  // value is narrowed to ClassName
}
```

**Component Breakdown**
- `instanceof`: The JavaScript `instanceof` operator.
- `ClassName`: The class to check against.

**General Syntax: `in` Guard**

```typescript
if ("property" in value) {
  // value is narrowed to members that have `property`
}
```

**Component Breakdown**
- `"property" in value`: Checks if the property exists.
- Narrows to members that have the property.

**General Syntax: User-Defined Type Predicate**

```typescript
function isTypeName(value: unknown): value is TypeName {
  return /* boolean check */;
}

if (isTypeName(value)) {
  // value is narrowed to TypeName
}
```

**Component Breakdown**
- `value is TypeName`: The type predicate return type.
- The function returns a boolean, but TypeScript uses the predicate for narrowing.

**General Syntax: Assertion Function**

```typescript
function assertIsTypeName(value: unknown): asserts value is TypeName {
  if (!/* check */) throw new Error("Not TypeName");
}

assertIsTypeName(value);
// value is narrowed to TypeName after this call
```

**Component Breakdown**
- `asserts value is TypeName`: The assertion signature.
- The function throws if the assertion fails.

**Syntax Rules**

- `typeof` works with `"string"`, `"number"`, `"boolean"`, `"symbol"`, `"bigint"`, `"undefined"`, `"object"`, `"function"`.
- `instanceof` works with class constructors and built-in objects.
- `in` works with object properties (including optional ones).
- Equality checks with literals narrow to literal types.
- User-defined predicates use `value is Type` return type.
- Assertion functions use `asserts value is Type` return type.
- Narrowing respects control flow: `if`/`else`, `switch`, `return`, `throw`, loops.
- Narrowing does not persist across async boundaries (after `await`).

**Constraints and Limitations**

- `typeof null` returns `"object"`, which is a known JavaScript quirk.
- `instanceof` does not work across execution contexts (iframes, worker threads).
- User-defined predicates are unchecked—an incorrect predicate can cause runtime errors.
- Assertion functions must throw on failure (not return false).
- Narrowing is erased at runtime; it's purely a compile-time analysis.
- Complex nested narrowing may not work as expected.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Built-In Type Guards

```typescript
// Step 1: Define a union.
type Value = string | number | Date | { name: string; age: number };

// Step 2: Use typeof for primitives.
function process(value: Value): string {
  if (typeof value === "string") {
    return `String: ${value.toUpperCase()}`;
  }
  if (typeof value === "number") {
    return `Number: ${value.toFixed(2)}`;
  }
  // Step 3: Use instanceof for classes.
  if (value instanceof Date) {
    return `Date: ${value.toISOString()}`;
  }
  // Step 4: Use `in` for object properties.
  if ("name" in value) {
    return `Person: ${value.name} (${value.age})`;
  }
  return "Unknown";
}

console.log(process("hello"));                        // "String: HELLO"
console.log(process(3.14159));                        // "Number: 3.14"
console.log(process(new Date("2024-01-01")));         // "Date: 2024-01-01T00:00:00.000Z"
console.log(process({ name: "Alice", age: 30 }));     // "Person: Alice (30)"
```

**Expected Output:**
```
String: HELLO
Number: 3.14
Date: 2024-01-01T00:00:00.000Z
Person: Alice (30)
```

**Why This Output Occurs:** Each type guard narrows the union to the corresponding member. `typeof` narrows primitives, `instanceof` narrows class instances, and `in` narrows objects with the specified property. The order matters—TypeScript checks each guard in sequence.

#### Example 2: User-Defined Type Predicates

```typescript
// Step 1: Define interfaces for the union.
interface Cat {
  kind: "cat";
  meow(): string;
}

interface Dog {
  kind: "dog";
  bark(): string;
}

type Pet = Cat | Dog;

// Step 2: Define user-defined type predicates.
function isCat(pet: Pet): pet is Cat {
  return pet.kind === "cat";
}

function isDog(pet: Pet): pet is Dog {
  return pet.kind === "dog";
}

// Step 3: Use the predicates for narrowing.
function describe(pet: Pet): string {
  if (isCat(pet)) {
    return pet.meow();  // ✅ Narrowed to Cat
  }
  if (isDog(pet)) {
    return pet.bark();  // ✅ Narrowed to Dog
  }
  return "Unknown pet";
}

const cat: Pet = { kind: "cat", meow: () => "Meow!" };
const dog: Pet = { kind: "dog", bark: () => "Woof!" };

console.log(describe(cat));  // "Meow!"
console.log(describe(dog));  // "Woof!"

// Step 4: Assertion functions.
function assertIsCat(pet: Pet): asserts pet is Cat {
  if (pet.kind !== "cat") {
    throw new Error("Not a cat");
  }
}

const unknownPet: Pet = { kind: "cat", meow: () => "Meow!" };
assertIsCat(unknownPet);
console.log(unknownPet.meow());  // ✅ Narrowed after assertion
```

**Expected Output:**
```
Meow!
Woof!
Meow!
```

**Why This Output Occurs:** The user-defined predicates `isCat` and `isDog` use the `pet is Cat` return type to tell TypeScript how to narrow. The `assertIsCat` function uses `asserts pet is Cat` to narrow in the calling scope after the assertion. Both approaches enable safe access to type-specific methods.

### Real-World Cases

**Case 1: API Response Handling**
API responses are narrowed using discriminants or custom predicates to handle success, error, and loading states.

**Case 2: Event Delegation**
DOM event handlers use `instanceof` to narrow events to `MouseEvent`, `KeyboardEvent`, etc., accessing event-specific properties.

**Case 3: Redux Reducers**
Redux reducers use discriminated union narrowing via `switch (action.type)` to handle each action type safely.

---

## 6. Exhaustiveness Checking Using the `never` Type

### Definitions

**Core Definition**
Exhaustiveness checking is a compile-time technique that ensures all members of a union have been handled in a pattern-matching construct (typically a `switch` or `if`/`else` chain). It uses the `never` type, which is the bottom type that no value can inhabit, to catch missing cases.

**Technical Definition**
Exhaustiveness checking works by assigning the narrowed value to a variable of type `never` in the `default` case of a `switch` statement (or the final `else` branch). If all union members have been handled, TypeScript narrows the value to `never` in the default case, and the assignment succeeds. If a member is unhandled, its type remains in the default case, and the assignment to `never` fails with a compile error. This technique is often encapsulated in an `assertNever` helper function. Adding a new member to the union causes compile errors at every exhaustive consumer, ensuring that all consumers are updated.

**Beginner-Friendly Explanation**
Exhaustiveness checking ensures you've handled every possible case in a union. You add a `default` case that assigns the value to `never`. If you've handled all cases, TypeScript knows the value can't exist in the default case (its type is `never`), and the assignment is fine. If you forgot a case, the value still has a type in the default case, and TypeScript gives you an error. This is incredibly useful: when you add a new variant to a union, TypeScript tells you every place that needs updating. It's like a compiler-enforced checklist.

### Purposes

- To ensure all union members are handled in pattern-matching code.
- To catch missing cases at compile time when adding new union members.
- To document the intention that a union is fully handled.
- To make refactoring safe by flagging all exhaustive consumers.
- To provide a compile-time guarantee of completeness.

### Syntax Rules and Structure

**General Syntax: Exhaustiveness with `never`**

```typescript
type Union = { kind: "a"; a: number } | { kind: "b"; b: string } | { kind: "c"; c: boolean };

function process(value: Union): string {
  switch (value.kind) {
    case "a":
      return `A: ${value.a}`;
    case "b":
      return `B: ${value.b}`;
    case "c":
      return `C: ${value.c}`;
    default:
      const _exhaustive: never = value;
      throw new Error(`Unhandled: ${JSON.stringify(_exhaustive)}`);
  }
}
```

**Component Breakdown**
- `default`: The fallback case.
- `const _exhaustive: never = value`: Assigns `value` to `never`.
- If all cases are handled, `value` is `never` and the assignment succeeds.
- If a case is missing, `value` retains its type and the assignment fails.

**General Syntax: `assertNever` Helper**

```typescript
function assertNever(value: never): never {
  throw new Error(`Unhandled value: ${JSON.stringify(value)}`);
}

function process(value: Union): string {
  switch (value.kind) {
    case "a": return `A: ${value.a}`;
    case "b": return `B: ${value.b}`;
    case "c": return `C: ${value.c}`;
    default: return assertNever(value);
  }
}
```

**Component Breakdown**
- `assertNever(value: never): never`: A helper function that accepts only `never`.
- If all cases are handled, `value` is `never` and the call succeeds.
- If a case is missing, the call fails.

**Syntax Rules**

- The `default` case must assign the value to a variable of type `never`.
- All union members must be handled before the `default` case.
- The `assertNever` helper accepts only `never` and throws.
- Discriminated unions are required for effective exhaustiveness checking.
- Adding a new variant causes compile errors at all exhaustive consumers.
- The `never` type is assignable to every type, so `assertNever` works in any return position.

**Constraints and Limitations**

- Exhaustiveness checking requires a discriminant (discriminated union).
- Non-discriminated unions may not narrow to `never` in the default case.
- The `never` assignment pattern is the only reliable exhaustiveness check.
- Exhaustiveness checking does not work with runtime-dynamic unions.
- The compiler does not enforce exhaustiveness automatically—you must add the check.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Exhaustive Switch with `never`

```typescript
// Step 1: Define a discriminated union.
type PaymentMethod =
  | { type: "credit_card"; cardNumber: string; expiry: string }
  | { type: "paypal"; email: string }
  | { type: "bank_transfer"; accountNumber: string; routingNumber: string };

// Step 2: Process with exhaustiveness checking.
function processPayment(method: PaymentMethod): string {
  switch (method.type) {
    case "credit_card":
      return `Processing card ending in ${method.cardNumber.slice(-4)}`;
    case "paypal":
      return `Processing PayPal for ${method.email}`;
    case "bank_transfer":
      return `Processing bank transfer from ${method.accountNumber}`;
    default:
      // Exhaustiveness check — if a new payment method is added,
      // this line produces a compile error until it's handled.
      const _exhaustive: never = method;
      throw new Error(`Unhandled payment method: ${JSON.stringify(_exhaustive)}`);
  }
}

// Step 3: Test all cases.
console.log(processPayment({ type: "credit_card", cardNumber: "4111111111111234", expiry: "12/25" }));
// "Processing card ending in 1234"
console.log(processPayment({ type: "paypal", email: "alice@example.com" }));
// "Processing PayPal for alice@example.com"
console.log(processPayment({ type: "bank_transfer", accountNumber: "123456789", routingNumber: "987654321" }));
// "Processing bank transfer from 123456789"
```

**Expected Output:**
```
Processing card ending in 1234
Processing PayPal for alice@example.com
Processing bank transfer from 123456789
```

**Why This Output Occurs:** The `switch` handles all three `PaymentMethod` variants. In the `default` case, TypeScript narrows `method` to `never` because all cases are handled. The assignment to `_exhaustive: never` succeeds. If a fourth payment method were added, the `default` case would receive the new variant, and the assignment would fail with a compile error.

#### Example 2: Adding a New Variant Triggers Compile Errors

```typescript
// Step 1: Initial union.
type Status = "active" | "inactive";

function getLabel(status: Status): string {
  switch (status) {
    case "active": return "Active";
    case "inactive": return "Inactive";
    default: {
      const _exhaustive: never = status;
      return _exhaustive;
    }
  }
}

// Step 2: Add a new variant.
type Status2 = "active" | "inactive" | "pending";

function getLabel2(status: Status2): string {
  switch (status) {
    case "active": return "Active";
    case "inactive": return "Inactive";
    // case "pending": return "Pending";  // ← Uncomment to fix the error
    default: {
      const _exhaustive: never = status;
      // ❌ Error: Type 'Status2' is not assignable to type 'never'.
      //   Type '"pending"' is not assignable to type 'never'.
      return _exhaustive;
    }
  }
}

// Step 3: The compile error tells the developer exactly what's missing.
console.log("If you see this, all cases are handled.");
```

**Expected Output:** The code produces a compile error on the `_exhaustive: never` assignment, indicating that `"pending"` is not handled. This is the intended behavior.

**Why This Output Occurs:** When `"pending"` is added to the `Status2` union without a corresponding `case`, TypeScript narrows `status` to `"pending"` in the `default` case. Since `"pending"` is not `never`, the assignment fails. This forces the developer to handle the new variant.

### Real-World Cases

**Case 1: State Machines**
State machine transitions use exhaustiveness checking to ensure all states and events are handled, preventing runtime errors from unhandled states.

**Case 2: Redux Reducers**
Redux reducers use exhaustiveness checking to ensure all action types are handled, catching missing handlers when new actions are added.

**Case 3: AST Visitors**
Compiler AST visitors use exhaustiveness checking to ensure all node types are visited, catching missing visitors when new node types are added.

**Case 4: API Response Handling**
API response handlers use exhaustiveness checking to ensure all response variants (success, error, loading, cancelled) are handled.

---

## References

- TypeScript Handbook: Narrowing — https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- TypeScript Handbook: Unions and Intersection Types — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types
- TypeScript Handbook: Discriminated Unions — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook: Exhaustiveness Checking — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
- TypeScript 3.5 Release Notes (Improved excess property checks in union types) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-5.html
- TypeScript 3.7 Release Notes (Assertion Functions) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html
- TypeScript Playground: Narrowing — https://www.typescriptlang.org/play/typescript/language/type-guards.ts.html
- Effective TypeScript: Item 3 — Understanding Type Widening
- Effective TypeScript: Item 22 — Understand Type Narrowing
- MDN: typeof Operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof
- MDN: instanceof Operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof
- MDN: in Operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in
- Total TypeScript: Discriminated Unions — https://www.totaltypescript.com/discriminated-unions
- TypeScript ESLint: no-unnecessary-condition — https://typescript-eslint.io/rules/no-unnecessary-condition/
- TypeScript Wiki: FAQ — Why does `typeof null` return "object"? — https://github.com/microsoft/TypeScript/wiki/FAQ