# TypeScript Conditional Types: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Conditional types are a TypeScript type-level feature that selects one of two possible types based on whether a type relationship test succeeds. They use the syntax `T extends U ? X : Y`, which means "if type `T` is assignable to type `U`, the result is type `X`; otherwise, the result is type `Y`". Conditional types bring decision-making to the type level, enabling non-uniform type mappings that adapt based on their inputs.

**Technical Definition**
A conditional type `T extends U ? X : Y` is evaluated by the type checker as follows: the check `T extends U` asks whether `T` is assignable to `U` under TypeScript's structural subtyping rules. If the condition is definitely true (even under the most permissive instantiation of type variables), the conditional resolves to `X`. If it is definitely false, it resolves to `Y`. If the condition depends on unresolved type variables, the conditional type is **deferred** until those variables are instantiated. When the checked type `T` is a **naked type parameter** (a type parameter that appears alone, not wrapped in a tuple, array, or other type constructor), the conditional type becomes **distributive**: it automatically distributes over union members during instantiation. The `infer` keyword, used within the true branch of a conditional type, captures and extracts type information from the matched pattern, enabling type-level pattern matching.

**Beginner-Friendly Explanation**
Conditional types are like "if-else" statements, but for types instead of values. You write `T extends U ? X : Y`, which means "if type `T` fits into type `U`, use type `X`; otherwise, use type `Y`." For example, `type IsString<T> = T extends string ? "yes" : "no"` gives you `"yes"` for strings and `"no"` for everything else. The real power comes when you use them with generics—TypeScript can decide at compile time which type to use based on what you pass in. The `infer` keyword is like a "capture" mechanism: it lets you extract parts of a type that match a pattern, similar to how regular expressions capture groups in strings. Conditional types are considered an advanced feature but are the foundation for many of TypeScript's built-in utility types like `Exclude`, `Extract`, and `ReturnType`.

### Key Characteristics

- **Type-level ternary**: `T extends U ? X : Y` mirrors JavaScript's `condition ? a : b`, but operates on types.
- **Assignability check**: The `extends` keyword checks assignability, not inheritance—structural subtyping determines the result.
- **Distribution over unions**: Naked type parameters distribute over union members, applying the conditional to each member separately.
- **Deferred evaluation**: Conditionals involving unresolved type variables are deferred until instantiation.
- **`infer` for extraction**: The `infer` keyword captures type information within the matched pattern.
- **Tuple wrapping to disable distribution**: `[T] extends [U]` prevents distribution when whole-union comparison is needed.
- **Compile-time only**: Conditional types are erased at runtime; they exist only during type checking.

### Prerequisites

- Solid understanding of TypeScript generics (type parameters, constraints)
- Familiarity with union types and intersection types
- Understanding of structural typing and assignability
- Basic knowledge of utility types (`Exclude`, `Extract`, `ReturnType`)
- Familiarity with `keyof` and indexed access types

### Related Programming Areas

- **Type-Level Programming**: Conditional types are the foundation for type-level computation
- **Utility Types**: `Exclude`, `Extract`, `NonNullable`, `ReturnType`, `Parameters` are built on conditional types
- **Type Inference**: `infer` enables type extraction from complex structures
- **Distributive Type Transformations**: Filtering, mapping, and transforming unions
- **Type-Safe Library Design**: Building reusable, adaptive generic APIs

### Core Concepts / Features

1. Conditional Type Syntax (`T extends U ? X : Y`) and Ternary Type Evaluation
2. Type Relationships and Subtyping Checks at the Type Level
3. Distributive Conditional Types When Operating on Naked Union Types
4. Disabling Distributivity Using Tuple Wrapping (`[T] extends [U]`)
5. Pattern Matching and Type Extraction via the `infer` Keyword


## 1. Conditional Type Syntax (`T extends U ? X : Y`) and Ternary Type Evaluation

### Definitions

**Core Definition**
The conditional type syntax `T extends U ? X : Y` is a type-level ternary expression. If `T` is assignable to `U`, the type evaluates to `X`; otherwise, it evaluates to `Y`. This mirrors JavaScript's conditional (ternary) operator, but operates entirely at compile time on types.

**Technical Definition**
A conditional type is declared using the form `T extends U ? X : Y`, where `T` is the checked type, `U` is the constraint type, `X` is the true-branch type, and `Y` is the false-branch type. When TypeScript encounters a conditional type, it first determines whether the condition can be resolved: if the most permissive instantiation of `T` (with all type parameters replaced by `any`) is not assignable to `U`, the conditional resolves to `Y`. If it can be resolved, TypeScript replaces type variables with their inferences and checks assignability again. If the condition still depends on unresolved type variables, the conditional type is deferred until instantiation. Conditional types can be nested to create multi-branch decision trees.

**Beginner-Friendly Explanation**
The syntax `T extends U ? X : Y` is TypeScript's way of saying "if T can be assigned to U, use type X; otherwise, use type Y." It's just like the `? :` operator in JavaScript, but for types. For example, `type Message<T> = T extends string ? "It's a string" : "Not a string"` will give you `"It's a string"` when `T` is `string` and `"Not a string"` otherwise. You can nest them to handle multiple cases: `T extends string ? "string" : T extends number ? "number" : "other"`. The key thing to remember is that `extends` here means "is assignable to"—TypeScript checks structural compatibility, not inheritance. So a type with all the members of `U` will satisfy the check, even if it was never declared to extend `U`.

### Purposes

- To select a type based on whether another type satisfies a condition.
- To create non-uniform type mappings that adapt to their inputs.
- To implement multi-branch type-level decision trees (nested conditionals).
- To serve as the foundation for TypeScript's built-in utility types.
- To enable type-level validation and constraint checking.

### Syntax Rules and Structure

**General Syntax: Basic Conditional Type**

```typescript
type ConditionalType<T> = T extends U ? X : Y;
```

**Component Breakdown**
- `T`: The checked type (often a generic type parameter).
- `extends`: The assignability check.
- `U`: The constraint type to check against.
- `? X`: The type to use if the condition is true.
- `: Y`: The type to use if the condition is false.

**General Syntax: Nested Conditional Types**

```typescript
type TypeName<T> =
  T extends string ? "string"
  : T extends number ? "number"
  : T extends boolean ? "boolean"
  : T extends undefined ? "undefined"
  : T extends Function ? "function"
  : "object";
```

**Component Breakdown**
- Each `? :` pair adds another branch to the decision tree.
- TypeScript evaluates branches in order until one matches.

**General Syntax: Conditional Type in Function Signatures**

```typescript
function process<T>(value: T): T extends string ? string : number {
  // ...
}
```

**Component Breakdown**
- The return type is conditional on the input type `T`.

**Syntax Rules**

- The condition uses `extends` for assignability checking, not inheritance.
- Nested conditionals use chained `? :` pairs.
- Conditional types can be used in any type position: aliases, function returns, parameters, generic constraints.
- The true and false branches can be any types, including other conditional types.
- When the condition depends on unresolved type variables, the conditional is deferred.
- Conditional types distribute over naked type parameters (see Section 3).

**Constraints and Limitations**

- Conditional types cannot be used at runtime—they are purely compile-time.
- The `extends` check uses structural assignability, which can produce surprising results.
- Deeply nested conditionals can become difficult to read and maintain.
- The compiler may produce complex error messages for failed conditional type checks.
- Conditional types involving unresolved generics are deferred, which can make debugging difficult.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Conditional Type

```typescript
// Step 1: Define a basic conditional type.
type IsString<T> = T extends string ? "yes" : "no";

// Step 2: Use it with different types.
type Test1 = IsString<string>;   // "yes"
type Test2 = IsString<number>;   // "no"
type Test3 = IsString<"hello">;  // "yes" (literal extends string)

// Step 3: Use in a function.
function describeType<T>(value: T): IsString<T> {
  if (typeof value === "string") {
    return "yes" as IsString<T>;
  }
  return "no" as IsString<T>;
}

console.log(describeType("hello"));  // "yes"
console.log(describeType(42));       // "no"

// Step 4: Nested conditional types.
type TypeName<T> =
  T extends string ? "string"
  : T extends number ? "number"
  : T extends boolean ? "boolean"
  : T extends undefined ? "undefined"
  : T extends Function ? "function"
  : "object";

type T0 = TypeName<string>;       // "string"
type T1 = TypeName<"a">;          // "string"
type T2 = TypeName<true>;         // "boolean"
type T3 = TypeName<() => void>;   // "function"
type T4 = TypeName<string[]>;     // "object"

console.log("Type names extracted.");
```

**Expected Output:**
```
yes
no
Type names extracted.
```

**Why This Output Occurs:** The `IsString<T>` conditional type checks if `T` is assignable to `string`. The `TypeName<T>` type uses nested conditionals to classify types into categories. The literal `"a"` is assignable to `string`, so it returns `"string"`.

#### Example 2: Conditional Types in Practice

```typescript
// Step 1: Define a conditional type for flattening arrays.
type Flatten<T> = T extends Array<infer U> ? U : T;

// Step 2: Use it.
type A = Flatten<string[]>;       // string
type B = Flatten<number>;         // number
type C = Flatten<(string | number)[]>;  // string | number

console.log("Flatten works.");

// Step 3: Define a conditional type for unwrapping promises.
type Unwrap<T> = T extends Promise<infer U> ? U : T;

type D = Unwrap<Promise<string>>;  // string
type E = Unwrap<number>;           // number

// Step 4: Use in a function.
async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);
  return response.json() as Promise<T>;
}

type Data = Unwrap<ReturnType<typeof fetchData>>;
// Data = T (unresolved generic)

// Step 5: Conditional types with function types.
type ReturnTypeOf<T> = T extends (...args: any[]) => infer R ? R : never;

type F = ReturnTypeOf<() => string>;        // string
type G = ReturnTypeOf<(a: number) => boolean>;  // boolean

console.log("Conditional type patterns complete.");
```

**Expected Output:**
```
Flatten works.
Conditional type patterns complete.
```

**Why This Output Occurs:** The `Flatten<T>` conditional type extracts the element type from arrays. The `Unwrap<T>` type extracts the resolved value from promises. The `ReturnTypeOf<T>` type extracts the return type from function types. Each demonstrates how conditional types adapt to their inputs.

### Real-World Cases

**Case 1: API Response Types**
Conditional types distinguish between success and error responses, enabling type-safe handling of API results.

**Case 2: Form Validation**
Conditional types derive validation result types from form field types, ensuring type safety across validation logic.

**Case 3: State Management**
Conditional types model state transitions, ensuring that actions produce the correct next state type.

**Case 4: Utility Types**
TypeScript's built-in utility types (`Exclude`, `Extract`, `NonNullable`, `ReturnType`) are all implemented as conditional types.

**Case 5: Type-Safe Event Emitters**
Conditional types map event names to their payload types, enabling type-safe event handling.

**Case 6: Configuration Validation**
Conditional types validate that configuration objects satisfy required shapes, producing compile-time errors for invalid configurations.


## 2. Type Relationships and Subtyping Checks at the Type Level

### Definitions

**Core Definition**
The `extends` keyword in conditional types performs a **subtyping check**—it asks whether the checked type is assignable to the constraint type. This is a structural assignability check, not a nominal inheritance check. If every value of type `T` can be used where a value of type `U` is expected, then `T extends U` is true.

**Technical Definition**
The assignability check `T extends U` in a conditional type follows TypeScript's structural subtyping rules. A type `T` is assignable to `U` if every member of `U` is present in `T` with a compatible type. This means `T` can have more members than `U` (it's a subtype structurally), but not fewer. The check is performed at the type level: if the condition can be resolved to `true` for all instantiations, the conditional resolves to `X`; if it can be resolved to `false` for all instantiations, it resolves to `Y`; otherwise, it is deferred. This structural approach means that literal types are subtypes of their base types (`"hello" extends string` is true), object types with extra properties are subtypes of types with fewer properties, and union types are subtypes if all their members are subtypes.

**Beginner-Friendly Explanation**
The `extends` in conditional types is not about class inheritance—it's about whether one type can be used in place of another. For example, `"hello" extends string` is true because a string literal can be used wherever a string is expected. Similarly, `{ name: string; age: number } extends { name: string }` is true because an object with more properties can be used where an object with fewer is expected. This is called "structural subtyping" or "duck typing" at the type level. It means you can use conditional types to check for the presence of specific properties or members, not just nominal type relationships.

### Purposes

- To check whether one type is assignable to another at the type level.
- To implement type-level validation and constraint checking.
- To filter union types based on assignability to a constraint.
- To extract types that match (or don't match) a constraint.
- To enable type-safe conditional logic in generic code.

### Syntax Rules and Structure

**General Syntax: Assignability Check**

```typescript
type IsAssignable<T, U> = T extends U ? true : false;
```

**Component Breakdown**
- `T extends U`: The assignability check.
- Returns `true` if `T` is assignable to `U`, `false` otherwise.

**General Syntax: Structural Subtyping Examples**

```typescript
type A = { name: string; age: number } extends { name: string } ? true : false;
// true — more properties is assignable to fewer

type B = { name: string } extends { name: string; age: number } ? true : false;
// false — fewer properties is not assignable to more

type C = "hello" extends string ? true : false;
// true — literal is assignable to primitive

type D = string extends "hello" ? true : false;
// false — primitive is not assignable to literal
```

**Component Breakdown**
- The check follows structural subtyping rules.
- More properties in the source is allowed; fewer is not.

**General Syntax: Function Type Assignability**

```typescript
type F1 = (a: number, b: string) => void;
type F2 = (a: number) => void;
type F3 = (a: number, b: string, c: boolean) => void;

type Check1 = F1 extends F2 ? true : false;  // true (fewer params is assignable)
type Check2 = F2 extends F1 ? true : false;  // false (more params is not assignable)
```

**Component Breakdown**
- Function types are assignable if parameters are compatible (contravariance).
- A function with fewer parameters is assignable to one with more.

**Syntax Rules**

- The `extends` check in conditional types uses structural assignability.
- `T extends U` is true if every member of `U` is present in `T`.
- Literal types are assignable to their primitives; primitives are not assignable to literals.
- More properties in the source is assignable to fewer; fewer is not assignable to more.
- Function types follow contravariant parameter and covariant return type rules.
- Union types are assignable if all members are assignable.
- Intersection types are assignable if the source satisfies all constituents.

**Constraints and Limitations**

- Structural subtyping can produce surprising results when types have similar shapes.
- `extends` does not check nominal identity—two types with the same shape are compatible.
- The `any` type is assignable to everything and everything is assignable to `any`.
- `never` is assignable to everything, but nothing is assignable to `never`.
- `unknown` is only assignable to `any` and `unknown`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Structural Subtyping Checks

```typescript
// Step 1: Define a base type.
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// Step 2: Check assignability.
type Check1 = Dog extends Animal ? "yes" : "no";  // "yes"
type Check2 = Animal extends Dog ? "yes" : "no";  // "no"

// Step 3: Structural subtyping (no inheritance needed).
interface Cat {
  name: string;
  meow(): void;
}

type Check3 = Cat extends Animal ? "yes" : "no";  // "yes" (has name)
type Check4 = Animal extends Cat ? "yes" : "no";  // "no" (missing meow)

// Step 4: Literal vs. primitive.
type Check5 = "hello" extends string ? "yes" : "no";  // "yes"
type Check6 = string extends "hello" ? "yes" : "no";  // "no"

// Step 5: Union assignability.
type Check7 = ("a" | "b") extends string ? "yes" : "no";  // "yes"
type Check8 = string extends "a" | "b" ? "yes" : "no";     // "no"

console.log("Subtyping checks complete.");

// Step 6: Practical use — extracting types that match a constraint.
type ExtractStrings<T> = T extends string ? T : never;

type Mixed = "a" | 1 | "b" | 2 | true;
type OnlyStrings = ExtractStrings<Mixed>;  // "a" | "b"

const s: OnlyStrings = "a";
console.log(s);  // "a"
```

**Expected Output:**
```
Subtyping checks complete.
a
```

**Why This Output Occurs:** `Dog` extends `Animal` because it has all of `Animal`'s members. `Cat` structurally satisfies `Animal` because it has `name`, even though it doesn't explicitly extend `Animal`. Literals are assignable to primitives, but not vice versa. The `ExtractStrings` conditional type filters the union to only string members.

#### Example 2: Function Type Assignability

```typescript
// Step 1: Define function types with different parameter counts.
type F1 = (a: number) => void;
type F2 = (a: number, b: string) => void;
type F3 = (a: number, b: string, c: boolean) => void;

// Step 2: Check assignability (fewer params is assignable to more).
type Check1 = F1 extends F2 ? true : false;  // true
type Check2 = F2 extends F1 ? true : false;  // false
type Check3 = F1 extends F3 ? true : false;  // true

// Step 3: Return type covariance.
type R1 = () => string;
type R2 = () => string | number;

type Check4 = R1 extends R2 ? true : false;  // true (string is assignable to string | number)
type Check5 = R2 extends R1 ? true : false;  // false

// Step 4: Parameter contravariance.
type P1 = (a: string) => void;
type P2 = (a: string | number) => void;

type Check6 = P1 extends P2 ? true : false;  // false (string | number is not assignable to string)
type Check7 = P2 extends P1 ? true : false;  // true (string is assignable to string | number)

// Step 5: Use in a practical utility type.
type FunctionReturn<T> = T extends (...args: any[]) => infer R ? R : never;

type R3 = FunctionReturn<() => string>;           // string
type R4 = FunctionReturn<(a: number) => boolean>; // boolean

console.log("Function assignability complete.");
```

**Expected Output:**
```
Function assignability complete.
```

**Why This Output Occurs:** A function with fewer parameters is assignable to one with more (you can ignore extra parameters). Return types are covariant (a narrower return type is assignable to a wider one). Parameter types are contravariant (a wider parameter type is assignable to a narrower one). The `FunctionReturn` type extracts return types from function types.

### Real-World Cases

**Case 1: API Response Discrimination**
Conditional types check whether a response type has a `data` property (success) or an `error` property (failure), enabling type-safe handling.

**Case 2: Form Field Validation**
Conditional types check whether a form field value extends `string`, `number`, or `boolean`, enabling type-specific validation.

**Case 3: State Machine Transitions**
Conditional types check whether a state type satisfies the conditions for a transition, ensuring type-safe state changes.

**Case 4: Plugin Systems**
Conditional types check whether a plugin satisfies the required interface, enabling type-safe plugin registration.

**Case 5: Type-Safe Configuration**
Conditional types check whether a configuration object extends a required shape, producing compile-time errors for invalid configurations.

**Case 6: Event System Typing**
Conditional types check whether an event payload extends a specific shape, enabling type-safe event handling.


## 3. Distributive Conditional Types When Operating on Naked Union Types

### Definitions

**Core Definition**
A **distributive conditional type** is a conditional type where the checked type is a **naked type parameter**—a type parameter that appears alone, not wrapped in a tuple, array, or other type constructor. When a distributive conditional type is instantiated with a union type, the conditional is automatically distributed over the union members, applying the check to each member individually and unioning the results.

**Technical Definition**
Conditional types in which the checked type is a naked type parameter are called distributive conditional types. An instantiation of `T extends U ? X : Y` with the type argument `A | B | C` for `T` resolves to `(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)`. Within the true branch, references to `T` are resolved to the individual constituents of the union (i.e., `T` refers to each member separately). This distributive property is used by utility types like `Exclude<T, U>` and `Extract<T, U>` to filter unions. Distribution only occurs for naked type parameters—if the checked type is wrapped in a tuple (`[T]`), array (`T[]`), or other type constructor, distribution is disabled.

**Beginner-Friendly Explanation**
A distributive conditional type is one that "spreads out" over union types. If you have a conditional type `T extends string ? "yes" : "no"` and you pass in `string | number`, TypeScript applies the check to each member separately: `string` gets `"yes"` and `number` gets `"no"`, giving you `"yes" | "no"`. This is different from checking the union as a whole—if you checked `string | number extends string`, the answer would be `false` because the whole union isn't assignable to `string`. Distribution happens automatically when the checked type is a "naked" type parameter—a type parameter that appears alone. If you wrap it in a tuple (`[T] extends [string]`), distribution is turned off. This is the basis for utility types like `Exclude` and `Extract`.

### Purposes

- To filter union types based on whether each member satisfies a condition.
- To apply type transformations to each member of a union separately.
- To implement utility types like `Exclude<T, U>` and `Extract<T, U>`.
- To create type-level mappings that adapt to each union member.
- To enable fine-grained control over union type manipulation.

### Syntax Rules and Structure

**General Syntax: Distributive Conditional Type**

```typescript
type Distributive<T> = T extends U ? X : Y;
// When T is a naked type parameter, distribution occurs.
```

**Component Breakdown**
- `T`: A naked type parameter (appears alone).
- When `T` is instantiated with a union, the conditional distributes.

**General Syntax: `Exclude` Utility (Built on Distribution)**

```typescript
type Exclude<T, U> = T extends U ? never : T;
// Removes from T the members assignable to U.
```

**Component Breakdown**
- Each member of `T` is checked against `U`.
- Members assignable to `U` become `never` (removed from the union).

**General Syntax: `Extract` Utility (Built on Distribution)**

```typescript
type Extract<T, U> = T extends U ? T : never;
// Keeps from T only the members assignable to U.
```

**Component Breakdown**
- Each member of `T` is checked against `U`.
- Members not assignable to `U` become `never` (removed).

**General Syntax: `NonNullable` Utility (Built on Distribution)**

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;
// Removes null and undefined from T.
```

**Component Breakdown**
- Each member of `T` is checked against `null | undefined`.
- Nullish members become `never`.

**Syntax Rules**

- Distribution occurs when the checked type is a naked type parameter.
- A naked type parameter appears alone, not wrapped in `[]`, `{}`, `Array<>`, etc.
- Distribution applies during instantiation with a union type.
- Each union member is checked independently.
- The results are unioned together.
- Utility types like `Exclude`, `Extract`, and `NonNullable` rely on distribution.
- Distribution does not occur for non-naked types (e.g., `[T] extends [U]`).

**Constraints and Limitations**

- Distribution can produce surprising results when you want to check the union as a whole.
- Distribution over `never` produces `never` (not the true or false branch).
- Distribution can make conditional types behave differently than expected with unions.
- Distribution does not occur when the checked type is wrapped in a tuple, array, or object.
- Distribution can be disabled with tuple wrapping (see Section 4).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Understanding Distribution

```typescript
// Step 1: Define a distributive conditional type.
type IsString<T> = T extends string ? "yes" : "no";

// Step 2: Apply to a single type.
type A = IsString<string>;  // "yes"
type B = IsString<number>;  // "no"

// Step 3: Apply to a union — distribution occurs!
type C = IsString<string | number>;
// Distributes: ("yes") | ("no") = "yes" | "no"

type D = IsString<"hello" | "world" | 42>;
// Distributes: ("yes") | ("yes") | ("no") = "yes" | "no"

console.log("Distribution demonstrated.");

// Step 4: Practical use — filtering unions.
type ExtractStrings<T> = T extends string ? T : never;

type Mixed = "a" | 1 | "b" | 2 | true;
type OnlyStrings = ExtractStrings<Mixed>;  // "a" | "b"

const s: OnlyStrings = "a";
console.log(s);  // "a"

// Step 5: Excluding types.
type ExcludeNumbers<T> = T extends number ? never : T;

type NoNumbers = ExcludeNumbers<"a" | 1 | "b" | 2>;
// "a" | "b"

const n: NoNumbers = "b";
console.log(n);  // "b"

// Step 6: Built-in utility types.
type Test1 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type Test2 = Extract<"a" | "b" | "c", "a" | "c">;  // "a" | "c"
type Test3 = NonNullable<string | null | undefined>;  // string

console.log("Union filtering complete.");
```

**Expected Output:**
```
Distribution demonstrated.
a
b
Union filtering complete.
```

**Why This Output Occurs:** The `IsString<T>` conditional type distributes over the union `string | number`, producing `"yes" | "no"`. The `ExtractStrings` type filters the union to only string members. The `ExcludeNumbers` type removes number members. The built-in `Exclude`, `Extract`, and `NonNullable` types all use distribution.

#### Example 2: Distribution with Complex Unions

```typescript
// Step 1: Define a union of object types.
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "rectangle"; width: number; height: number };

// Step 2: Distribution with object types.
type GetKind<T> = T extends { kind: infer K } ? K : never;

type ShapeKinds = GetKind<Shape>;
// Distributes: "circle" | "square" | "rectangle"

console.log("Shape kinds extracted.");

// Step 3: Filter shapes by kind.
type CirclesOnly = Extract<Shape, { kind: "circle" }>;
// { kind: "circle"; radius: number }

const circle: CirclesOnly = { kind: "circle", radius: 5 };
console.log(circle.radius);  // 5

// Step 4: Distribution with nested unions.
type Nested = { type: "a"; data: string } | { type: "b"; data: number } | { type: "c" };

type HasData<T> = T extends { data: infer D } ? D : never;

type DataTypes = HasData<Nested>;
// string | number

// Step 5: Distribution over never.
type NeverTest = IsString<never>;
// never (distribution over never produces never)

console.log("Complex distribution complete.");
```

**Expected Output:**
```
Shape kinds extracted.
5
Complex distribution complete.
```

**Why This Output Occurs:** The `GetKind<T>` type distributes over the `Shape` union, extracting the `kind` literal from each member. `Extract` filters to only circle shapes. The `HasData<T>` type distributes over `Nested`, extracting the `data` type from each member. Distribution over `never` produces `never`.

### Real-World Cases

**Case 1: Redux Action Filtering**
Distribution filters a union of action types to only those that match a specific handler.

**Case 2: API Response Handling**
Distribution separates success and error response types from a union of possible responses.

**Case 3: Form Field Validation**
Distribution applies validation rules to each field type in a union of form fields.

**Case 4: Event System Typing**
Distribution filters event types to those with specific payload shapes.

**Case 5: State Management**
Distribution extracts specific state slice types from a union of possible states.

**Case 6: Database Query Results**
Distribution filters query result types based on whether they have specific properties.


## 4. Disabling Distributivity Using Tuple Wrapping (`[T] extends [U]`)

### Definitions

**Core Definition**
Tuple wrapping is a technique to **disable distributivity** in conditional types. By wrapping both the checked type and the constraint type in single-element tuples (`[T] extends [U]`), the conditional type treats the union as a whole instead of distributing over each member. This is essential when you need to compare entire unions or check if a union is assignable to a type as a single entity.

**Technical Definition**
The distribution of conditional types is triggered when the checked type is a **naked type parameter**. Wrapping `T` in a tuple (`[T]`) makes it no longer naked—`[T]` is a tuple type whose element type is `T`, not a union. When the checked type is `[T]` (a tuple) rather than `T` (a naked parameter), the conditional type does not distribute. The same applies to the constraint side: `[U]` is a tuple containing `U`, not `U` itself. The discipline is to know which behavior you need and to wrap one or both sides in a single-element tuple when you want to compare the whole union as one type instead of element-by-element. Wrapping both sides is the standard approach for disabling distribution.

**Beginner-Friendly Explanation**
Normally, conditional types distribute over unions—they check each member separately. But sometimes you want to check the union as a whole. For example, you might want to know "is this entire union assignable to `string`?" If you use `T extends string`, distribution would check each member separately. But if you wrap it in a tuple—`[T] extends [string]`—TypeScript checks the whole union at once. This is because `[T]` is a tuple containing `T`, not a union of `T`s. So `[string | number]` is not assignable to `[string]` because the tuple's element is a union, not a single string. This trick is essential for writing correct equality checks and whole-union comparisons in TypeScript.

### Purposes

- To disable distribution when comparing entire unions.
- To implement type equality checks that work correctly on unions.
- To check whether a whole union is assignable to a type.
- To prevent the "surprising" behavior of distributive conditionals in equality checks.
- To handle `never` correctly (distribution over `never` produces `never`).

### Syntax Rules and Structure

**General Syntax: Disabling Distribution**

```typescript
type NonDistributive<T, U> = [T] extends [U] ? X : Y;
```

**Component Breakdown**
- `[T]`: Wraps the checked type in a tuple.
- `[U]`: Wraps the constraint type in a tuple.
- The conditional no longer distributes over unions in `T`.

**General Syntax: Equality Check**

```typescript
type Equals<A, B> = [A] extends [B] ? ([B] extends [A] ? true : false) : false;
```

**Component Breakdown**
- Both sides are wrapped to compare as wholes.
- Two assignability checks in both directions for equality.

**General Syntax: Comparing Union to Type**

```typescript
type IsWholeUnionAssignable<T, U> = [T] extends [U] ? true : false;

type Test = IsWholeUnionAssignable<string | number, string>;
// false — the union is not assignable to string
```

**Component Breakdown**
- Without wrapping, distribution would produce `true | false` (boolean).
- With wrapping, the whole union is checked: `false`.

**General Syntax: Handling `never`**

```typescript
type Wrap<T> = [T] extends [never] ? null : { value: T };

type Test1 = Wrap<never>;        // null
type Test2 = Wrap<string>;       // { value: string }
type Test3 = Wrap<string | never>;  // { value: string } (never absorbed)
```

**Component Breakdown**
- Without wrapping, `never extends never ? null : { value: never }` distributes to `never`.
- With wrapping, `[never] extends [never]` is `true`, so the result is `null`.

**Syntax Rules**

- Wrap the checked type in a single-element tuple to disable distribution.
- Wrapping both sides (`[T] extends [U]`) is the standard approach.
- Distribution is disabled because `[T]` is a tuple, not a naked type parameter.
- This works for equality checks, whole-union assignability, and `never` handling.
- The technique is essential for writing correct type-level utilities.
- Distribution should be kept when per-element evaluation is desired (e.g., `Exclude`, `Extract`).

**Constraints and Limitations**

- Wrapping changes the comparison from union-wide to tuple-wide.
- The tuple wrapper adds a level of nesting that can affect complex type manipulations.
- Overusing tuple wrapping can make types harder to read.
- Distribution is sometimes desired (e.g., filtering unions)—know when to disable it.
- The technique does not work for binary type-level operators that take exactly one type.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Disabling Distribution for Equality

```typescript
// Step 1: Compare distributive vs. non-distributive behavior.
type Distributive<T> = T extends string ? "yes" : "no";
type NonDistributive<T> = [T] extends [string] ? "yes" : "no";

// Step 2: Test with a union.
type A = Distributive<string | number>;     // "yes" | "no" (distributed!)
type B = NonDistributive<string | number>;  // "no" (whole union check)

console.log("Distribution behavior compared.");

// Step 3: Type equality check with tuple wrapping.
type Equals<A, B> = [A] extends [B] ? ([B] extends [A] ? true : false) : false;

type Test1 = Equals<string, string>;           // true
type Test2 = Equals<string, number>;           // false
type Test3 = Equals<string | number, string | number>;  // true
type Test4 = Equals<string | number, string>;  // false (whole union comparison)

console.log("Equality checks complete.");

// Step 4: Whole-union assignability.
type IsAssignableTo<T, U> = [T] extends [U] ? true : false;

type Check1 = IsAssignableTo<string, string | number>;  // true
type Check2 = IsAssignableTo<string | number, string>;  // false
type Check3 = IsAssignableTo<"a" | "b", string>;        // true
type Check4 = IsAssignableTo<string, "a" | "b">;        // false

console.log("Assignability checks complete.");

// Step 5: Distribution-free `never` handling.
type Wrap<T> = [T] extends [never] ? null : { value: T };

type W1 = Wrap<never>;       // null
type W2 = Wrap<string>;      // { value: string }
type W3 = Wrap<string | never>;  // { value: string }

console.log("Never handling complete.");
```

**Expected Output:**
```
Distribution behavior compared.
Equality checks complete.
Assignability checks complete.
Never handling complete.
```

**Why This Output Occurs:** Without tuple wrapping, `Distributive<string | number>` produces `"yes" | "no"`. With wrapping, `NonDistributive<string | number>` checks the whole union and produces `"no"`. The `Equals` type correctly compares unions as wholes. The `Wrap` type handles `never` correctly with wrapping (returning `null` instead of `never`).

#### Example 2: Practical Use Cases

```typescript
// Step 1: Type-safe array checker (whole-union comparison).
type IsArray<T> = [T] extends [readonly unknown[]] ? true : false;

type A1 = IsArray<string[]>;          // true
type A2 = IsArray<string>;            // false
type A3 = IsArray<string | number[]>; // false (whole union is not an array)

console.log("Array checks complete.");

// Step 2: Tuple detection.
type IsTuple<T> = [T] extends [readonly unknown[]]
  ? number extends T["length"]
    ? false  // regular array
    : true   // tuple
  : false;

type T1 = IsTuple<[string, number]>;  // true
type T2 = IsTuple<string[]>;          // false
type T3 = IsTuple<string | [number]>; // false (whole union check)

console.log("Tuple checks complete.");

// Step 3: Exact type matching.
type StrictEqual<A, B> = [A] extends [B] ? ([B] extends [A] ? true : false) : false;

type S1 = StrictEqual<{ name: string }, { name: string }>;  // true
type S2 = StrictEqual<{ name: string }, { name: string; age: number }>;  // false

// Step 4: Conditional distribution for filtering (distribution desired).
type FilterNumbers<T> = T extends number ? never : T;
type Filtered = FilterNumbers<"a" | 1 | "b" | 2>;  // "a" | "b"

console.log("Practical type utilities complete.");
```

**Expected Output:**
```
Array checks complete.
Tuple checks complete.
Practical type utilities complete.
```

**Why This Output Occurs:** The `IsArray<T>` type uses tuple wrapping to check the whole union—`string | number[]` is not an array, so it returns `false`. Without wrapping, it would distribute and return `true | false`. The `IsTuple<T>` type uses tuple wrapping and `T["length"]` to detect tuples. Distribution is kept for `FilterNumbers` because per-element evaluation is desired.

### Real-World Cases

**Case 1: Type Equality Utilities**
Libraries like `type-fest` use tuple wrapping in `Equals<A, B>` to implement exact type equality checks.

**Case 2: Array and Tuple Detection**
Utility types use tuple wrapping to detect whether a type is an array or tuple as a whole, not per-member.

**Case 3: Conditional Type Guards**
Type guards use tuple wrapping to check whole-union assignability without distribution.

**Case 4: `never` Handling**
Utilities that need to handle `never` specially (e.g., optional type wrappers) use tuple wrapping to avoid distribution collapsing to `never`.

**Case 5: API Response Discrimination**
API response handlers use tuple wrapping to check whether a response union as a whole is assignable to a success or error type.

**Case 6: Form Validation**
Form validation utilities use tuple wrapping to check whole-union assignability of form field types.

### References

- TypeScript Handbook: Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Handbook: Distributive Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types
- TypeScript 2.8 Release Notes: Conditional Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html
- TypeScript Handbook: Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- Control Distribution with the `[T] extends [U]` Tuple Trick — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-advanced-patterns/references/tlp-distributive-conditional-control.md
- Stack Overflow: Why does TypeScript distribute conditional types over union types only when the checked type is a naked type parameter? — https://stackoverflow.com/questions/79898352/why-does-typescript-distribute-conditional-types-over-union-types-only-when-the
- Stack Overflow: How to avoid distributive conditional types — https://stackoverflow.com/q/70789029
- TypeScript PR #21316: Conditional types — https://github.com/Microsoft/TypeScript/pull/21316
- Total TypeScript: Conditional Types — https://www.totaltypescript.com/workshops/type-transformations/conditional-types-and-infer/conditional-types
- TypeScript Deep Dive: Conditional Types — https://basarat.gitbook.io/typescript/type-system/conditional-types
- MDN: Conditional (ternary) operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_operator


## 5. Pattern Matching and Type Extraction via the `infer` Keyword

### Definitions

**Core Definition**
The `infer` keyword is used within the true branch of a conditional type to **capture** and **extract** type information from the matched pattern. It declares a type variable that TypeScript infers based on the structure being checked, enabling type-level pattern matching similar to regular expression capture groups.

**Technical Definition**
The `infer` keyword appears in the constraint of a conditional type: `T extends SomePattern<infer U> ? U : never`. When the pattern matches, TypeScript infers the type variable `U` from the corresponding position in `T` and makes it available in the true branch. Multiple `infer` declarations can appear in a single conditional type, enabling extraction of multiple type components. `infer` works with arrays (`(infer U)[]`), promises (`Promise<infer U>`), functions (`(...args: infer P) => infer R`), objects (`{ data: infer D }`), and template literals (`` `${infer Prefix}:${infer Suffix}` ``). The `infer` keyword is the foundation for utility types like `ReturnType`, `Parameters`, and `Awaited`.

**Beginner-Friendly Explanation**
The `infer` keyword is like a "capture" mechanism for types. It lets you say "if this type matches this pattern, capture the part that varies as a new type variable." For example, `T extends Array<infer U> ? U : never` captures the element type of an array—if `T` is `string[]`, then `U` is `string`. You can use `infer` with arrays, promises, functions, objects, and even template literals. Multiple `infer` declarations let you extract multiple parts at once. This is how TypeScript's built-in utility types like `ReturnType<T>` and `Parameters<T>` work under the hood. It's like having a type-level regular expression engine.

### Purposes

- To extract type information from complex structures (arrays, promises, functions, objects).
- To implement type-level pattern matching for type transformations.
- To create utility types that derive types from existing types.
- To enable type extraction from template literal strings.
- To build type-level parsers and transformers.

### Syntax Rules and Structure

**General Syntax: Basic `infer` Extraction**

```typescript
type ExtractType<T> = T extends Pattern<infer U> ? U : never;
```

**Component Breakdown**
- `Pattern<infer U>`: The pattern to match, with `U` capturing the type.
- `? U`: The captured type is returned in the true branch.

**General Syntax: Array Element Extraction**

```typescript
type ArrayElement<T> = T extends (infer U)[] ? U : never;
```

**Component Breakdown**
- `(infer U)[]`: Matches array types and captures the element type.

**General Syntax: Promise Value Extraction**

```typescript
type PromiseValue<T> = T extends Promise<infer U> ? U : never;
```

**Component Breakdown**
- `Promise<infer U>`: Matches promise types and captures the resolved value.

**General Syntax: Function Return Type Extraction**

```typescript
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
```

**Component Breakdown**
- `(...args: any[]) => infer R`: Matches function types and captures the return type.

**General Syntax: Function Parameter Extraction**

```typescript
type MyParameters<T> = T extends (...args: infer P) => any ? P : never;
```

**Component Breakdown**
- `(...args: infer P) => any`: Matches function types and captures the parameter tuple.

**General Syntax: Template Literal Extraction**

```typescript
type RemovePrefix<T> = T extends `prefix:${infer Rest}` ? Rest : T;
```

**Component Breakdown**
- `` `prefix:${infer Rest}` ``: Matches template literal types and captures the suffix.

**Syntax Rules**

- `infer` is only allowed within the constraint of a conditional type.
- Multiple `infer` declarations can appear in a single conditional type.
- The inferred type variable is available in the true branch.
- `infer` can capture types from arrays, tuples, promises, functions, objects, and template literals.
- The captured type can be used in the true branch for further processing.
- `infer X extends Y` (TypeScript 4.7+) combines capture and constraint checking.

**Constraints and Limitations**

- `infer` cannot be used outside a conditional type's true branch.
- The pattern must be structurally matchable by TypeScript's inference engine.
- `infer` cannot capture multiple occurrences of the same type variable in some cases.
- Complex nested `infer` patterns can produce `unknown` or `never` unexpectedly.
- Distribution affects `infer` when the checked type is a naked union.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `infer` Extraction Patterns

```typescript
// Step 1: Extract array element type.
type ArrayElement<T> = T extends (infer U)[] ? U : never;

type Test1 = ArrayElement<string[]>;       // string
type Test2 = ArrayElement<number[]>;       // number
type Test3 = ArrayElement<(string | number)[]>;  // string | number
type Test4 = ArrayElement<string>;         // never

// Step 2: Extract promise value.
type PromiseValue<T> = T extends Promise<infer U> ? U : never;

type Test5 = PromiseValue<Promise<string>>;  // string
type Test6 = PromiseValue<Promise<number>>;  // number

// Step 3: Extract function return type.
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Test7 = MyReturnType<() => string>;           // string
type Test8 = MyReturnType<(a: number) => boolean>; // boolean

// Step 4: Extract function parameter types.
type MyParameters<T> = T extends (...args: infer P) => any ? P : never;

type Test9 = MyParameters<(a: string, b: number) => void>;
// [a: string, b: number]

// Step 5: Extract object property type.
type GetData<T> = T extends { data: infer D } ? D : never;

type Test10 = GetData<{ data: string }>;      // string
type Test11 = GetData<{ data: number[] }>;    // number[]
type Test12 = GetData<{ other: string }>;     // never

console.log("Infer extraction complete.");
```

**Expected Output:**
```
Infer extraction complete.
```

**Why This Output Occurs:** Each conditional type uses `infer` to capture a specific part of the matched type. `ArrayElement` extracts element types, `PromiseValue` extracts promise values, `MyReturnType` extracts return types, `MyParameters` extracts parameter tuples, and `GetData` extracts object property types.

#### Example 2: Advanced `infer` Patterns

```typescript
// Step 1: Template literal extraction.
type RemovePrefix<T> = T extends `prefix:${infer Rest}` ? Rest : T;

type Test1 = RemovePrefix<"prefix:value">;  // "value"
type Test2 = RemovePrefix<"other">;         // "other"

// Step 2: Multiple captures in template literals.
type ParseRoute<T> = T extends `${infer Start}:${infer Param}/${infer Rest}`
  ? { start: Start; param: Param; rest: ParseRoute<Rest> }
  : T extends `${infer Start}:${infer Param}`
  ? { start: Start; param: Param }
  : T;

type Route = ParseRoute<"/users/:id/posts/:postId">;
// Nested structure with extracted params

// Step 3: Nested infer for tuples.
type FirstElement<T> = T extends [infer First, ...infer Rest] ? First : never;
type RestElements<T> = T extends [infer First, ...infer Rest] ? Rest : never;

type Test3 = FirstElement<[string, number, boolean]>;  // string
type Test4 = RestElements<[string, number, boolean]>;  // [number, boolean]

// Step 4: Combining infer with distribution.
type ExtractNumbers<T> = T extends Array<infer U>
  ? U extends number ? U : never
  : never;

type Mixed = [1, 2, "a", 3, "b"];
type OnlyNumbers = ExtractNumbers<Mixed>;  // 1 | 2 | 3

// Step 5: Constrained infer (TypeScript 4.7+).
type ToNumber<S extends string> = S extends `${infer N extends number}` ? N : never;

type Test5 = ToNumber<"42">;    // 42
type Test6 = ToNumber<"abc">;   // never

console.log("Advanced infer patterns complete.");
```

**Expected Output:**
```
Advanced infer patterns complete.
```

**Why This Output Occurs:** The `RemovePrefix` type extracts the suffix after `"prefix:"`. The `ParseRoute` type uses multiple `infer` captures to parse route patterns. The `FirstElement` and `RestElements` types extract tuple components. The `ExtractNumbers` type combines `infer` with distribution to filter numbers from a union. The constrained `infer N extends number` extracts numeric literals.

### Real-World Cases

**Case 1: `ReturnType<T>` Utility**
`ReturnType<T>` is implemented as `T extends (...args: any[]) => infer R ? R : never`, extracting function return types.

**Case 2: `Parameters<T>` Utility**
`Parameters<T>` is implemented as `T extends (...args: infer P) => any ? P : never`, extracting function parameter tuples.

**Case 3: `Awaited<T>` Utility**
`Awaited<T>` uses recursive `infer` to unwrap promise chains, extracting the final resolved value.

**Case 4: Route Parsing**
Routing libraries use template literal `infer` to extract path parameters from route patterns.

**Case 5: API Response Unwrapping**
API clients use `infer` to extract data types from generic response wrappers like `ApiResponse<T>`.

**Case 6: Form Field Extraction**
Form libraries use `infer` to extract field value types from form schema types.

**Case 7: State Management**
State management libraries use `infer` to extract state slice types from action types.

**Case 8: Test Utilities**
Test utilities use `infer` to extract mock function return types and parameter types.

**Case 9: Type-Safe Event Emitters**
Event emitters use `infer` to extract event payload types from event definition types.

**Case 10: GraphQL Response Typing**
GraphQL clients use `infer` to extract data types from query result types.

### References

- TypeScript Handbook: Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Handbook: Infer Type Operators — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types
- TypeScript 2.8 Release Notes: Conditional Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html
- TypeScript 4.7 Release Notes: `infer` `extends` Constraints — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-7.html
- TypeScript Handbook: Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- The `infer` Keyword — https://github.com/mcollina/skills/blob/main/skills/typescript-magician/rules/infer-keyword.md
- Total TypeScript: `infer` — https://www.totaltypescript.com/workshops/type-transformations/conditional-types-and-infer/infer/solution
- Convex TypeScript Guide: `infer` — https://www.convex.dev/typescript/advanced/type-operators-manipulation/infer
- TypeScript Deep Dive: Conditional Types — https://basarat.gitbook.io/typescript/type-system/conditional-types
- MDN: Template Literals — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals
- TypeScript PR #21316: Conditional Types — https://github.com/Microsoft/TypeScript/pull/21316
- TypeScript 4.7 Release Notes: `infer` `extends` Constraints — https://devblogs.microsoft.com/typescript/announcing-typescript-4-7/#infer-extends-constraints
- Effective TypeScript: Item 14 — Use Type Operations and Generics to Avoid Repeating Yourself