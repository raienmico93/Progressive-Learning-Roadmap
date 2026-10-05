# TypeScript Type Aliases: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A type alias in TypeScript is a name given to any type using the `type` keyword. Type aliases allow developers to create reusable, descriptive names for complex types—primitives, objects, unions, intersections, tuples, functions, and recursive structures—without creating a new runtime entity. They are purely a compile-time construct.

**Technical Definition**
A type alias is a TypeScript declaration that binds an identifier to a type expression using the syntax `type AliasName = TypeExpression;`. Type aliases do not create new types in the nominal sense; they are transparent references to the underlying type. TypeScript resolves aliases structurally, meaning a value of the aliased type is fully compatible with the original type. Type aliases can represent any type: primitives, object literals, unions, intersections, tuples, function signatures, conditional types, mapped types, and recursive types. Unlike interfaces, type aliases cannot be extended or implemented directly, cannot participate in declaration merging, and can represent non-object types (unions, primitives). Type aliases support generic parameters.

**Beginner-Friendly Explanation**
A type alias is a nickname for a type. Instead of writing `{ id: number; name: string; email: string }` everywhere, you can write `type User = { id: number; name: string; email: string }` and then just use `User`. Type aliases make code more readable and easier to maintain—if the shape changes, you update it in one place. You can create aliases for anything: a simple string, a complex object, a union of options, a tuple, or a function signature. Unlike interfaces, type aliases can represent unions and primitives, but they can't be "extended" the way interfaces can.

### Key Characteristics

- **Transparent**: Type aliases are resolved structurally; they do not create distinct nominal types.
- **Universal**: Can alias any type—primitives, objects, unions, intersections, tuples, functions, and more.
- **Reusable**: Centralize complex type definitions for consistency and maintainability.
- **Generic-capable**: Support type parameters for parameterized types.
- **Recursive**: Can reference themselves in their own definition.
- **Compile-time only**: Erased during compilation; no runtime representation.
- **Not extensible**: Cannot be extended or implemented like interfaces (though intersections can approximate extension).

### Prerequisites

- Basic knowledge of TypeScript primitive types
- Familiarity with TypeScript object types and interfaces
- Understanding of union and intersection types
- Familiarity with generics (for generic type aliases)

### Related Programming Areas

- **Type Theory**: Type aliases correspond to type-level definitions
- **Generics**: Type aliases support type parameters, enabling reusable parameterized types
- **Discriminated Unions**: Union type aliases are fundamental to discriminated union patterns
- **Functional Programming**: Function type aliases and recursive types are common in functional programming
- **Data Modeling**: Type aliases are used extensively for domain modeling and API contracts

### Core Concepts / Features

1. Primitive Aliases
2. Object Aliases
3. Union and Intersection Aliases
4. Tuple Aliases
5. Function Aliases
6. Recursive Type Aliases (e.g., JSON Representation)


## 1. Primitive Aliases

### Definitions

**Core Definition**
A primitive alias is a type alias that gives a name to a primitive type (string, number, boolean, bigint, symbol, null, undefined) or a literal type. Primitive aliases serve as documentation and semantic markers, making code more readable by expressing intent.

**Technical Definition**
Primitive type aliases bind an identifier to a primitive type or literal type using `type AliasName = PrimitiveType;`. Because TypeScript's type system is structural, a primitive alias is entirely transparent—a value of type `UserId` (aliased to `string`) is fully interchangeable with any `string`. Primitive aliases do not provide nominal typing; to achieve nominal-like behavior, developers use "branded types" (intersection types with a phantom property). Primitive aliases are useful for self-documenting code, especially in domain modeling where a `UserId` and an `OrderId` might both be strings but have different semantics.

**Beginner-Friendly Explanation**
A primitive alias is a fancy name for a simple type. Instead of writing `string` everywhere, you can write `type UserId = string` and then use `UserId` in your code. This makes your code more readable—when someone sees `UserId`, they know it's a string that represents a user's ID. But here's the catch: TypeScript still treats `UserId` exactly like `string`. You can pass a `UserId` where a `string` is expected, and vice versa. If you want to prevent that (to make `UserId` distinct from `OrderId`), you need a "branded type," which is a more advanced technique.

### Purposes

- To provide semantic names for primitive types, improving code readability.
- To document the intended meaning of a primitive value in domain models.
- To centralize primitive type definitions for consistency across a codebase.
- To serve as a foundation for branded types that provide nominal-like safety.
- To make function signatures more expressive by naming their primitive types.

### Syntax Rules and Structure

**General Syntax: Primitive Alias**

```typescript
type AliasName = PrimitiveType;
```

**Component Breakdown**
- `type`: The keyword introducing the type alias.
- `AliasName`: The name of the alias (PascalCase by convention).
- `PrimitiveType`: The primitive type being aliased (`string`, `number`, `boolean`, etc.).

**General Syntax: Literal Type Alias**

```typescript
type Direction = "Up" | "Down" | "Left" | "Right";
```

**Component Breakdown**
- The alias represents a union of string literal types.

**General Syntax: Branded Type**

```typescript
type Brand<T, B> = T & { readonly __brand: B };
type UserId = Brand<string, "UserId">;
```

**Component Breakdown**
- `Brand<T, B>`: A utility type that intersects `T` with a phantom brand property.
- `UserId`: A branded string type that is not interchangeable with plain `string`.

**Syntax Rules**

- Primitive aliases use the `type` keyword.
- The alias name must be a valid TypeScript identifier.
- Primitive aliases are transparent—they do not create new nominal types.
- Literal unions can be aliased to create named union types.
- Branded types use intersection with a phantom property for nominal-like behavior.
- Type aliases can be exported and imported like any other TypeScript declaration.

**Constraints and Limitations**

- Primitive aliases provide no additional type safety over the primitive type itself.
- Two primitive aliases for the same primitive type are fully interchangeable.
- Branded types require type assertions to construct from raw primitives.
- Branded types add a phantom property that must be handled in serialization.
- Type aliases cannot be extended or implemented like interfaces.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Primitive Alias

```typescript
// Step 1: Define primitive aliases.
type UserId = string;
type OrderId = string;
type Quantity = number;
type IsActive = boolean;

// Step 2: Use the aliases in functions.
function getUser(id: UserId): { id: UserId; name: string } {
  return { id, name: "Alice" };
}

function getOrder(id: OrderId): { id: OrderId; total: Quantity } {
  return { id, total: 100 };
}

// Step 3: Call the functions.
const user = getUser("user-123");
const order = getOrder("order-456");
console.log(user);   // { id: 'user-123', name: 'Alice' }
console.log(order);  // { id: 'order-456', total: 100 }

// Step 4: Aliases are transparent — no type safety gained.
const userId: UserId = "user-123";
const orderId: OrderId = userId;  // ✅ Allowed — both are strings!
console.log(orderId);  // "user-123"
```

**Expected Output:**
```
{ id: 'user-123', name: 'Alice' }
{ id: 'order-456', total: 100 }
user-123
```

**Why This Output Occurs:** The aliases `UserId` and `OrderId` are both just `string`. TypeScript allows `userId` to be assigned to `orderId` because they are structurally identical. The aliases provide documentation but no nominal type safety.

#### Example 2: Branded Types for Nominal Safety

```typescript
// Step 1: Define a brand utility type.
type Brand<T, B extends string> = T & { readonly __brand: B };

// Step 2: Create branded aliases.
type UserId = Brand<string, "UserId">;
type OrderId = Brand<string, "OrderId">;

// Step 3: Create branded values using type assertions.
function createUserId(id: string): UserId {
  return id as UserId;  // Assertion required
}

function createOrderId(id: string): OrderId {
  return id as OrderId;
}

// Step 4: Branded types are not interchangeable.
const userId = createUserId("user-123");
const orderId = createOrderId("order-456");

// const wrongId: UserId = orderId;  // ❌ Error: Type 'OrderId' is not assignable to type 'UserId'.

// Step 5: Branded types are still strings at runtime.
console.log(typeof userId);  // "string"
console.log(userId);  // "user-123"
```

**Expected Output:**
```
string
user-123
```

**Why This Output Occurs:** The `Brand` utility type intersects `string` with a phantom property `{ readonly __brand: "UserId" }`. This makes `UserId` structurally distinct from `OrderId` because the phantom property types differ. At runtime, the brand is erased, and the value is just a string.

### Real-World Cases

**Case 1: Domain Modeling**
Domain-driven design uses primitive aliases for entity IDs, monetary amounts, and other value objects to document their meaning and prevent confusion.

**Case 2: Branded Types for Safety**
Financial applications use branded types to distinguish between `USD` and `EUR` amounts (both numbers) or between `AccountId` and `TransactionId` (both strings), preventing accidental mixing.

**Case 3: API Contracts**
API type definitions use primitive aliases to document the meaning of string and number fields, improving API documentation and consumer understanding.

---

## 2. Object Aliases

### Definitions

**Core Definition**
An object alias is a type alias that gives a name to an object type. It describes the shape of an object—the properties it contains and their types—using the same syntax as an anonymous object type but with a name for reuse.

**Technical Definition**
An object type alias binds an identifier to an object type literal using `type AliasName = { ... };`. The alias is structurally identical to an interface with the same members, and values of the aliased type are fully compatible with values of an equivalent interface. Object aliases support all object type features: optional properties, readonly properties, index signatures, method signatures, and nested object types. Unlike interfaces, object aliases cannot be extended or implemented, but they can be combined with intersections to achieve similar effects. Object aliases can also be generic.

**Beginner-Friendly Explanation**
An object alias is a named blueprint for an object. Instead of writing `{ name: string; age: number }` repeatedly, you write `type Person = { name: string; age: number }` and use `Person` everywhere. It works exactly like an interface for most purposes—you can use it to type variables, function parameters, and return values. The main difference is that type aliases can't be "extended" like interfaces, but you can combine them with `&` (intersection) to achieve a similar result. Choose interfaces for public APIs and type aliases for unions and utility types.

### Purposes

- To provide reusable names for object shapes used in multiple places.
- To document the structure of domain entities, DTOs, and configuration objects.
- To enable type-safe access to object properties across a codebase.
- To serve as the foundation for discriminated unions and complex types.
- To centralize object shape definitions for easier maintenance.

### Syntax Rules and Structure

**General Syntax: Object Alias**

```typescript
type AliasName = {
  property1: Type1;
  property2?: Type2;
  readonly property3: Type3;
};
```

**Component Breakdown**
- `type AliasName`: The type alias declaration.
- `{ ... }`: The object type literal.
- Properties can have optional (`?`) and readonly modifiers.

**General Syntax: Generic Object Alias**

```typescript
type Container<T> = {
  value: T;
  timestamp: Date;
};
```

**Component Breakdown**
- `<T>`: The generic type parameter.
- `value: T`: The property uses the generic type.

**General Syntax: Object Alias with Index Signature**

```typescript
type Dictionary<T> = {
  [key: string]: T;
};
```

**Component Breakdown**
- `[key: string]: T`: Index signature for dynamic keys.

**Syntax Rules**

- Object aliases use the `type` keyword followed by the alias name and an object type literal.
- All object type features are supported: optional, readonly, index signatures, methods.
- Object aliases can be generic with type parameters.
- Object aliases can be combined with intersections (`&`) to emulate extension.
- Object aliases are structurally compatible with equivalent interfaces.
- Object aliases can be exported and imported.

**Constraints and Limitations**

- Object aliases cannot be extended or implemented like interfaces.
- Object aliases do not support declaration merging.
- Excess property checking applies to object literals assigned to object aliases.
- Recursive object aliases require careful definition to avoid circular references.
- Object aliases are erased at compile time.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Object Alias

```typescript
// Step 1: Define an object alias.
type User = {
  id: number;
  name: string;
  email: string;
  age?: number;
};

// Step 2: Use the alias in functions.
function createUser(id: number, name: string, email: string): User {
  return { id, name, email };
}

function printUser(user: User): void {
  const ageInfo = user.age !== undefined ? `, age ${user.age}` : "";
  console.log(`${user.name} (${user.email})${ageInfo}`);
}

// Step 3: Create and use users.
const alice = createUser(1, "Alice", "alice@example.com");
printUser(alice);  // "Alice (alice@example.com)"

const bob: User = { id: 2, name: "Bob", email: "bob@example.com", age: 30 };
printUser(bob);  // "Bob (bob@example.com), age 30"
```

**Expected Output:**
```
Alice (alice@example.com)
Bob (bob@example.com), age 30
```

**Why This Output Occurs:** The `User` type alias describes the required properties (`id`, `name`, `email`) and an optional property (`age`). The `printUser` function handles the optional `age` with a narrowing check.

#### Example 2: Generic Object Alias with Intersection

```typescript
// Step 1: Define a generic object alias.
type ApiResponse<T> = {
  data: T;
  status: number;
  message: string;
};

// Step 2: Define a base entity alias.
type Entity = {
  id: number;
  createdAt: Date;
};

// Step 3: Combine with intersection to emulate extension.
type UserEntity = Entity & {
  name: string;
  email: string;
};

// Step 4: Use the combined type.
const userResponse: ApiResponse<UserEntity> = {
  data: {
    id: 1,
    createdAt: new Date(),
    name: "Alice",
    email: "alice@example.com",
  },
  status: 200,
  message: "Success",
};

console.log(`Status: ${userResponse.status}`);
console.log(`User: ${userResponse.data.name} (${userResponse.data.email})`);
```

**Expected Output:**
```
Status: 200
User: Alice (alice@example.com)
```

**Why This Output Occurs:** The generic alias `ApiResponse<T>` wraps any data type with API metadata. The intersection `Entity & { name: string; email: string }` combines the base entity properties with additional user-specific properties, emulating interface extension.

### Real-World Cases

**Case 1: API Response Wrappers**
Generic object aliases like `ApiResponse<T>` or `PaginatedResponse<T>` wrap API data with metadata, providing consistent typing across all endpoints.

**Case 2: Domain Entities**
Object aliases define domain entities (User, Order, Product) with their properties, serving as the foundation for type-safe business logic.

**Case 3: Configuration Types**
Configuration object aliases define the shape of application settings, enabling type-safe configuration access and validation.

---

## 3. Union and Intersection Aliases

### Definitions

**Core Definition**
A union alias is a type alias that names a union of two or more types, meaning a value of that type can be any one of the constituent types. An intersection alias names an intersection of two or more types, meaning a value must satisfy all constituent types simultaneously.

**Technical Definition**
Union type aliases use the `|` operator to combine types: `type Status = "active" | "inactive" | "pending"`. Intersection type aliases use the `&` operator: `type Employee = Person & { employeeId: number }`. Union aliases are fundamental to discriminated unions, where a common literal property distinguishes between variants. Intersection aliases combine properties from multiple types into a single type. Type aliases are the only way to name union and intersection types (interfaces cannot represent unions). Both union and intersection aliases are transparent and structural.

**Beginner-Friendly Explanation**
A union alias says "this value can be one of these types." For example, `type Status = "active" | "inactive"` means a `Status` value is either `"active"` or `"inactive"`. A union is like a menu of options. An intersection alias says "this value must be all of these types at once." For example, `type Employee = Person & { employeeId: number }` means an `Employee` is a `Person` AND has an `employeeId`. Intersections are like combining two requirements. Union aliases are essential for discriminated unions, which are TypeScript's way of modeling "one of several shapes."

### Purposes

- To name unions of literal types for use as enum-like constructs.
- To create discriminated unions for modeling variants of a data structure.
- To combine multiple object types with intersections to emulate multiple inheritance.
- To express "one of several types" or "all of several types" clearly.
- To enable exhaustive checking with `never` in union aliases.

### Syntax Rules and Structure

**General Syntax: Union Alias**

```typescript
type AliasName = Type1 | Type2 | Type3;
```

**Component Breakdown**
- `Type1 | Type2 | Type3`: The union of types.
- A value of `AliasName` must be assignable to at least one of the constituent types.

**General Syntax: Intersection Alias**

```typescript
type AliasName = Type1 & Type2 & Type3;
```

**Component Breakdown**
- `Type1 & Type2 & Type3`: The intersection of types.
- A value of `AliasName` must be assignable to all constituent types.

**General Syntax: Discriminated Union Alias**

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number }
  | { kind: "rectangle"; width: number; height: number };
```

**Component Breakdown**
- Each union member has a literal `kind` property (the discriminant).
- Narrowing on `kind` narrows to the specific member.

**Syntax Rules**

- Union aliases use the `|` operator.
- Intersection aliases use the `&` operator.
- Union aliases can combine any types (primitives, literals, objects, tuples, functions).
- Intersection aliases combine object types by merging their properties.
- Discriminated unions require a common literal property (discriminant) in each member.
- Unions and intersections can be nested and combined.
- Type aliases are required for naming union types (interfaces cannot represent unions).

**Constraints and Limitations**

- Union types require narrowing before accessing type-specific properties.
- Intersection types can produce `never` if constituent types have conflicting property types.
- Excess property checking with union types can be confusing (improved in TypeScript 3.5).
- Intersections of primitives with objects can produce unexpected results.
- Union aliases cannot be extended like interfaces.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Union Alias for Status

```typescript
// Step 1: Define a union alias.
type Status = "active" | "inactive" | "pending";

// Step 2: Use the union in a function.
function getStatusMessage(status: Status): string {
  switch (status) {
    case "active": return "The account is active.";
    case "inactive": return "The account is inactive.";
    case "pending": return "The account is pending approval.";
    default:
      // Exhaustiveness check
      const _exhaustive: never = status;
      return _exhaustive;
  }
}

// Step 3: Call with valid and invalid values.
console.log(getStatusMessage("active"));    // "The account is active."
console.log(getStatusMessage("pending"));   // "The account is pending approval."
// getStatusMessage("deleted");  // ❌ Error: Argument of type '"deleted"' is not assignable to parameter of type 'Status'.
```

**Expected Output:**
```
The account is active.
The account is pending approval.
```

**Why This Output Occurs:** The union alias `Status` restricts the function parameter to the three literal values. The `switch` statement exhaustively handles all cases, and the `never` check ensures no case is missed.

#### Example 2: Intersection Alias for Combining Types

```typescript
// Step 1: Define base types.
type Timestamped = {
  createdAt: Date;
  updatedAt: Date;
};

type Identifiable = {
  id: number;
};

// Step 2: Combine with intersection.
type Entity = Identifiable & Timestamped;

type User = Entity & {
  name: string;
  email: string;
};

// Step 3: Create an object satisfying all intersections.
const user: User = {
  id: 1,
  createdAt: new Date("2024-01-01"),
  updatedAt: new Date("2024-06-15"),
  name: "Alice",
  email: "alice@example.com",
};

// Step 4: All properties are accessible.
console.log(`User ${user.id}: ${user.name}`);
console.log(`Created: ${user.createdAt.toISOString()}`);
console.log(`Updated: ${user.updatedAt.toISOString()}`);
```

**Expected Output:**
```
User 1: Alice
Created: 2024-01-01T00:00:00.000Z
Updated: 2024-06-15T00:00:00.000Z
```

**Why This Output Occurs:** The intersection `Identifiable & Timestamped` requires both `id` and the timestamp properties. The `User` type further intersects with `{ name: string; email: string }`. The resulting object must satisfy all constituent types simultaneously.

### Real-World Cases

**Case 1: API State Management**
Union aliases model loading states: `type RequestState<T> = { status: "loading" } | { status: "success"; data: T } | { status: "error"; error: Error }`.

**Case 2: Event Systems**
Union aliases model events: `type AppEvent = { type: "userLogin"; userId: string } | { type: "userLogout"; userId: string } | { type: "pageView"; path: string }`.

**Case 3: Mixin Patterns**
Intersection aliases combine mixin traits: `type Loggable = Timestamped & Identifiable & Serializable`.

---

## 4. Tuple Aliases

### Definitions

**Core Definition**
A tuple alias is a type alias that gives a name to a tuple type. It allows developers to reuse tuple definitions across the codebase, improving readability and maintainability for fixed-length, heterogeneous collections.

**Technical Definition**
Tuple aliases bind an identifier to a tuple type using `type AliasName = [Type1, Type2, ...];`. All tuple features are supported: optional elements, rest elements, named elements, and readonly tuples. Tuple aliases are particularly useful for typing function return values that return multiple values, React hooks, and coordinate-like data structures. Generic tuple aliases can be parameterized, and variadic tuple types (TypeScript 4.0+) enable powerful tuple manipulation.

**Beginner-Friendly Explanation**
A tuple alias is a name for a tuple type. Instead of writing `[number, number]` everywhere for a coordinate, you write `type Point = [number, number]` and use `Point`. Tuple aliases are especially useful when a function returns multiple values—you can name the return type to make it clear what each position means. For example, `type MinMax = [min: number, max: number]` documents that the first element is the minimum and the second is the maximum.

### Purposes

- To name tuple types used in multiple places for consistency.
- To document the meaning of each tuple position with named elements.
- To type function return values that return multiple values.
- To enable reusable tuple types for coordinates, ranges, and key-value pairs.
- To support generic tuple manipulation with variadic tuple types.

### Syntax Rules and Structure

**General Syntax: Tuple Alias**

```typescript
type AliasName = [Type1, Type2, Type3];
```

**Component Breakdown**
- `[Type1, Type2, Type3]`: The tuple type with per-position types.

**General Syntax: Tuple Alias with Named Elements**

```typescript
type Point = [x: number, y: number];
```

**Component Breakdown**
- `x`, `y`: Descriptive labels for each position (documentation only).

**General Syntax: Generic Tuple Alias**

```typescript
type Pair<T, U> = [first: T, second: U];
```

**Component Breakdown**
- `<T, U>`: Generic type parameters.
- `first: T`, `second: U`: Named elements using generic types.

**General Syntax: Rest Tuple Alias**

```typescript
type Command = [name: string, ...args: string[]];
```

**Component Breakdown**
- `...args: string[]`: Rest element allowing variable trailing arguments.

**Syntax Rules**

- Tuple aliases use the `type` keyword followed by the alias name and a tuple type.
- All tuple features are supported: optional, rest, named, and readonly elements.
- Tuple aliases can be generic with type parameters.
- Named elements are for documentation; they do not affect access or destructuring.
- Tuple aliases are structurally compatible with equivalent tuple literals.

**Constraints and Limitations**

- Tuple aliases cannot be extended or implemented.
- Named elements do not enforce property-style access (index access is still used).
- Optional elements must come after required elements.
- Rest elements must come last.
- Tuple aliases share all limitations of tuple types (fixed length, runtime mutability).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Tuple Aliases

```typescript
// Step 1: Define tuple aliases.
type Point = [x: number, y: number];
type MinMax = [min: number, max: number];
type KeyValue = [key: string, value: number];

// Step 2: Use the aliases in functions.
function distance(p1: Point, p2: Point): number {
  return Math.sqrt((p2[0] - p1[0]) ** 2 + (p2[1] - p1[1]) ** 2);
}

function findMinMax(numbers: number[]): MinMax {
  return [Math.min(...numbers), Math.max(...numbers)];
}

// Step 3: Call the functions.
const d = distance([0, 0], [3, 4]);
console.log(`Distance: ${d}`);  // 5

const [min, max] = findMinMax([5, 2, 8, 1, 9]);
console.log(`Min: ${min}, Max: ${max}`);  // "Min: 1, Max: 9"

// Step 4: Use KeyValue in an array.
const entries: KeyValue[] = [["a", 1], ["b", 2], ["c", 3]];
entries.forEach(([key, value]) => console.log(`${key} = ${value}`));
```

**Expected Output:**
```
Distance: 5
Min: 1, Max: 9
a = 1
b = 2
c = 3
```

**Why This Output Occurs:** The `Point` alias names a two-number tuple, the `MinMax` alias names a two-number tuple with descriptive labels, and the `KeyValue` alias names a `[string, number]` tuple. Named elements provide documentation in editor tooltips but do not affect destructuring.

#### Example 2: Generic Tuple Aliases

```typescript
// Step 1: Define a generic tuple alias for pairs.
type Pair<T, U> = [first: T, second: U];

// Step 2: Define specific aliases from the generic.
type StringNumberPair = Pair<string, number>;
type UserPair = Pair<{ name: string }, { name: string }>;

// Step 3: Use the aliases.
const entry: StringNumberPair = ["age", 30];
console.log(`${entry[0]}: ${entry[1]}`);  // "age: 30"

// Step 4: Generic tuple alias with rest elements.
type Command<T> = [name: string, ...args: T[]];

const stringCommand: Command<string> = ["echo", "hello", "world"];
const numberCommand: Command<number> = ["add", 1, 2, 3];

function executeCommand<T>(command: Command<T>): void {
  const [name, ...args] = command;
  console.log(`Executing ${name} with args: ${args.join(", ")}`);
}

executeCommand(stringCommand);  // "Executing echo with args: hello, world"
executeCommand(numberCommand);  // "Executing add with args: 1, 2, 3"
```

**Expected Output:**
```
age: 30
Executing echo with args: hello, world
Executing add with args: 1, 2, 3
```

**Why This Output Occurs:** The generic `Pair<T, U>` alias enables type-safe pairs of any two types. The `Command<T>` alias combines a fixed string name with a rest parameter of type `T[]`, enabling type-safe variadic commands.

### Real-World Cases

**Case 1: React `useState`**
React's `useState` returns a tuple `[value, setValue]`. Libraries often define named tuple aliases like `type StateHook<T> = [T, (value: T) => void]`.

**Case 2: Coordinate Systems**
Game development and mapping applications use tuple aliases like `type Position = [x: number, y: number]` or `type Position3D = [x: number, y: number, z: number]`.

**Case 3: Database Query Results**
Query functions return tuples like `type QueryResult<T> = [rows: T[], count: number]`, documenting the meaning of each element.

---

## 5. Function Aliases

### Definitions

**Core Definition**
A function alias is a type alias that names a function type. It describes the function's parameter types and return type without providing an implementation. Function aliases are used to type callbacks, higher-order functions, event handlers, and any value that is a function.

**Technical Definition**
Function type aliases use the syntax `type AliasName = (param1: Type1, param2: Type2) => ReturnType;`. They can include optional parameters, rest parameters, generic parameters, and overloaded signatures (via intersection). Function aliases are structurally typed: any function with compatible parameters and return type is assignable to the alias. They are commonly used for callback parameters, event listeners, and functional programming patterns like `map`, `filter`, and `reduce`.

**Beginner-Friendly Explanation**
A function alias is a name for a function's signature. Instead of writing `(a: number, b: number) => number` everywhere for a callback, you write `type MathOperation = (a: number, b: number) => number`. Function aliases make your code more readable and consistent. They're used for callbacks, event handlers, and any place where you need to describe "a function that takes these arguments and returns this type." You can also make them generic, so one alias can describe many similar functions.

### Purposes

- To name callback signatures for reuse across multiple functions.
- To document event handler and listener signatures.
- To type higher-order functions that accept or return functions.
- To enable generic function signatures with type parameters.
- To provide consistent typing for functional programming utilities.

### Syntax Rules and Structure

**General Syntax: Function Alias**

```typescript
type AliasName = (param1: Type1, param2: Type2) => ReturnType;
```

**Component Breakdown**
- `(param1: Type1, param2: Type2)`: The parameter list with types.
- `=> ReturnType`: The return type.
- The alias names the entire function type.

**General Syntax: Generic Function Alias**

```typescript
type Mapper<T, U> = (item: T, index: number) => U;
```

**Component Breakdown**
- `<T, U>`: Generic type parameters.
- `(item: T, index: number) => U`: A function from `T` to `U`.

**General Syntax: Function Alias with Rest Parameters**

```typescript
type Logger = (level: string, ...messages: string[]) => void;
```

**Component Breakdown**
- `...messages: string[]`: Rest parameter.

**General Syntax: Function Alias with Optional Parameters**

```typescript
type Formatter = (value: number, precision?: number) => string;
```

**Component Breakdown**
- `precision?: number`: Optional parameter.

**Syntax Rules**

- Function aliases use arrow function syntax for the type.
- Parameters can be optional (`?`), rest (`...`), or have default values (though defaults are not part of the type).
- Return type can be any type, including `void`, `never`, or another function type.
- Generic type parameters are supported.
- Overloaded function types can be expressed via intersection of function aliases.
- Functions are structurally typed: any compatible function is assignable.

**Constraints and Limitations**

- Function aliases cannot have implementations (they are types only).
- Overloaded function types require intersection types.
- `this` parameter typing requires special syntax.
- Function aliases do not support declaration merging.
- Parameter names in function aliases are documentation only (they do not affect compatibility).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Function Aliases

```typescript
// Step 1: Define function aliases.
type MathOperation = (a: number, b: number) => number;
type Predicate<T> = (value: T) => boolean;
type Comparator<T> = (a: T, b: T) => number;

// Step 2: Use the aliases in functions.
function calculate(a: number, b: number, operation: MathOperation): number {
  return operation(a, b);
}

function filterArray<T>(array: T[], predicate: Predicate<T>): T[] {
  return array.filter(predicate);
}

// Step 3: Call with compatible functions.
const add: MathOperation = (a, b) => a + b;
const multiply: MathOperation = (a, b) => a * b;

console.log(calculate(5, 3, add));       // 8
console.log(calculate(5, 3, multiply));  // 15

const isEven: Predicate<number> = (n) => n % 2 === 0;
console.log(filterArray([1, 2, 3, 4, 5, 6], isEven));  // [2, 4, 6]

// Step 4: Inline functions also work.
console.log(calculate(10, 2, (a, b) => a / b));  // 5
```

**Expected Output:**
```
8
15
[ 2, 4, 6 ]
5
```

**Why This Output Occurs:** The `MathOperation` alias names a function type `(a: number, b: number) => number`. Any function with that signature is assignable, whether named (`add`, `multiply`) or inline. The generic `Predicate<T>` alias works for any element type.

#### Example 2: Generic Function Aliases and Higher-Order Functions

```typescript
// Step 1: Define generic function aliases.
type Mapper<T, U> = (item: T, index: number, array: T[]) => U;
type Reducer<T, U> = (accumulator: U, item: T, index: number) => U;

// Step 2: Use them in higher-order functions.
function transform<T, U>(array: T[], mapper: Mapper<T, U>): U[] {
  return array.map(mapper);
}

function fold<T, U>(array: T[], reducer: Reducer<T, U>, initial: U): U {
  return array.reduce(reducer, initial);
}

// Step 3: Call with typed functions.
const numbers = [1, 2, 3, 4, 5];

const doubled = transform(numbers, (n) => n * 2);
console.log(doubled);  // [2, 4, 6, 8, 10]

const strings = transform(numbers, (n, i) => `Item ${i}: ${n}`);
console.log(strings);  // ["Item 0: 1", "Item 1: 2", ...]

const sum = fold(numbers, (acc, n) => acc + n, 0);
console.log(`Sum: ${sum}`);  // 15

// Step 4: Function aliases compose.
type Transformer<T> = (input: T) => T;
const identity: Transformer<number> = (x) => x;
const double: Transformer<number> = (x) => x * 2;
const composed: Transformer<number> = (x) => double(identity(x));
console.log(composed(21));  // 42
```

**Expected Output:**
```
[ 2, 4, 6, 8, 10 ]
[ 'Item 0: 1', 'Item 1: 2', 'Item 2: 3', 'Item 3: 4', 'Item 4: 5' ]
Sum: 15
42
```

**Why This Output Occurs:** The generic `Mapper<T, U>` and `Reducer<T, U>` aliases describe common functional programming signatures. The `transform` and `fold` functions accept these aliases as parameters, enabling type-safe higher-order functions. Inline arrow functions are contextually typed by the aliases.

### Real-World Cases

**Case 1: Event Handlers**
Event handler aliases like `type ClickHandler = (event: MouseEvent) => void` provide consistent typing for DOM event listeners.

**Case 2: Middleware Functions**
Express.js and similar frameworks use middleware aliases like `type Middleware = (req: Request, res: Response, next: NextFunction) => void`.

**Case 3: Redux Action Creators**
Action creator aliases like `type ActionCreator<P> = (payload: P) => Action<P>` provide consistent typing for Redux action creators.

---

## 6. Recursive Type Aliases (e.g., JSON Representation)

### Definitions

**Core Definition**
A recursive type alias is a type alias that references itself in its own definition. Recursive types are used to model self-referential data structures like trees, linked lists, nested JSON, and file systems. TypeScript allows recursive type aliases as long as the recursion occurs within an object type, array type, or tuple type (not directly at the top level).

**Technical Definition**
TypeScript supports recursive type aliases with restrictions: a type alias can reference itself as long as the reference is not at the "top level" of the type (i.e., not immediately resolved during type evaluation). The recursion must be "guarded" by an object type, array type, or tuple type. This restriction prevents infinite type expansion. The classic example is a JSON type: `type Json = string | number | boolean | null | Json[] | { [key: string]: Json };`. Recursive types are essential for modeling hierarchical data like trees, ASTs, and nested configurations.

**Beginner-Friendly Explanation**
A recursive type alias is a type that refers to itself. Think of a tree: a tree has a value and a list of child trees, and each child is also a tree. In TypeScript, you can write `type Tree = { value: number; children: Tree[] }`. The `Tree` type refers to itself inside the definition. This works because the self-reference is inside an object type (`{ ... }`). You can't write `type X = X` (direct self-reference), but you can write `type X = { next: X }` (guarded self-reference). Recursive types are used for JSON, trees, linked lists, and any nested data structure.

### Purposes

- To model hierarchical data structures like trees and nested objects.
- To represent JSON values with full type safety.
- To define linked lists, binary trees, and other recursive data structures.
- To type abstract syntax trees (ASTs) in compilers and interpreters.
- To model file systems, organizational charts, and other nested hierarchies.

### Syntax Rules and Structure

**General Syntax: Recursive Type Alias**

```typescript
type AliasName = {
  value: Type;
  children: AliasName[];
};
```

**Component Breakdown**
- `AliasName` references itself inside the object type.
- The recursion is guarded by the object type and array type.

**General Syntax: JSON Type Alias**

```typescript
type Json =
  | string
  | number
  | boolean
  | null
  | Json[]
  | { [key: string]: Json };
```

**Component Breakdown**
- `Json` references itself in the array and object alternatives.
- The recursion is guarded by array and object types.

**General Syntax: Linked List Alias**

```typescript
type ListNode<T> = {
  value: T;
  next: ListNode<T> | null;
};
```

**Component Breakdown**
- `next` references `ListNode<T>` or `null` (terminator).

**Syntax Rules**

- Recursive type aliases must have the self-reference guarded by an object type, array type, or tuple type.
- Direct self-reference (`type X = X`) is a compile error.
- Recursion can be mutual (two types referencing each other).
- Generic recursive aliases are supported.
- Recursive aliases are transparent and structural.
- TypeScript 3.7+ relaxed some recursion restrictions for recursive type aliases.

**Constraints and Limitations**

- Direct self-reference is not allowed.
- Recursive types can cause performance issues with deeply nested data.
- Recursive types cannot be extended or implemented.
- Excessively deep recursion may cause compiler stack overflow.
- Recursive types in unions require careful narrowing.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: JSON Representation

```typescript
// Step 1: Define the recursive JSON type.
type Json =
  | string
  | number
  | boolean
  | null
  | Json[]
  | { [key: string]: Json };

// Step 2: Create JSON values.
const jsonString: Json = "hello";
const jsonNumber: Json = 42;
const jsonArray: Json = [1, "two", true, null];
const jsonObject: Json = {
  name: "Alice",
  age: 30,
  hobbies: ["reading", "coding"],
  address: {
    street: "123 Main St",
    city: "Springfield",
  },
};

// Step 3: Define a function that processes JSON.
function stringify(value: Json): string {
  return JSON.stringify(value);
}

console.log(stringify(jsonString));  // '"hello"'
console.log(stringify(jsonObject));  // '{"name":"Alice","age":30,...}'

// Step 4: Type-safe JSON access requires narrowing.
function getStringValue(json: Json, key: string): string | null {
  if (typeof json === "object" && json !== null && !Array.isArray(json)) {
    const value = json[key];
    if (typeof value === "string") {
      return value;
    }
  }
  return null;
}

console.log(getStringValue(jsonObject, "name"));  // "Alice"
console.log(getStringValue(jsonObject, "age"));   // null (not a string)
```

**Expected Output:**
```
"hello"
{"name":"Alice","age":30,"hobbies":["reading","coding"],"address":{"street":"123 Main St","city":"Springfield"}}
Alice
null
```

**Why This Output Occurs:** The `Json` type recursively describes all valid JSON values. The `stringify` function accepts any JSON value. The `getStringValue` function narrows the JSON to an object, then narrows the property value to a string.

#### Example 2: Tree Structure

```typescript
// Step 1: Define a recursive tree type.
type TreeNode = {
  value: number;
  left?: TreeNode;
  right?: TreeNode;
};

// Step 2: Create a binary tree.
const tree: TreeNode = {
  value: 10,
  left: {
    value: 5,
    left: { value: 3 },
    right: { value: 7 },
  },
  right: {
    value: 15,
    right: { value: 20 },
  },
};

// Step 3: Recursively traverse the tree.
function sumTree(node: TreeNode | undefined): number {
  if (!node) return 0;
  return node.value + sumTree(node.left) + sumTree(node.right);
}

console.log(`Sum: ${sumTree(tree)}`);  // 10 + 5 + 3 + 7 + 15 + 20 = 60

// Step 4: Find a value in the tree.
function findValue(node: TreeNode | undefined, target: number): boolean {
  if (!node) return false;
  if (node.value === target) return true;
  return findValue(node.left, target) || findValue(node.right, target);
}

console.log(`Find 7: ${findValue(tree, 7)}`);    // true
console.log(`Find 12: ${findValue(tree, 12)}`);  // false
```

**Expected Output:**
```
Sum: 60
Find 7: true
Find 12: false
```

**Why This Output Occurs:** The `TreeNode` type references itself in the `left` and `right` properties, which are optional. The `sumTree` and `findValue` functions recursively traverse the tree, handling `undefined` at the leaves.

### Real-World Cases

**Case 1: JSON Parsing and Validation**
The `Json` type is used to type `JSON.parse()` results, forcing validation before accessing nested properties.

**Case 2: Abstract Syntax Trees**
Compilers and interpreters use recursive types to represent ASTs, where each node can contain child nodes of the same type.

**Case 3: File System Modeling**
File systems are naturally recursive: a directory contains files and subdirectories, each of which is itself a directory or file.

---

## References

- TypeScript Handbook: Type Aliases — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases
- TypeScript Handbook: Object Types — https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Handbook: Unions and Intersections — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types
- TypeScript Handbook: Tuple Types — https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types
- TypeScript Handbook: More on Functions — https://www.typescriptlang.org/docs/handbook/2/functions.html
- TypeScript 3.7 Release Notes (Recursive Type Aliases) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html
- TypeScript 4.0 Release Notes (Variadic Tuple Types) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-0.html
- TypeScript Playground: Type Aliases — https://www.typescriptlang.org/play/typescript/primitives/type-aliases.ts.html
- Effective TypeScript: Item 13 — Know the Differences Between type and interface
- Total TypeScript: Type Aliases vs Interfaces — https://www.totaltypescript.com/type-vs-interface
- MDN: JSON — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON
- TypeScript ESLint: consistent-type-definitions — https://typescript-eslint.io/rules/consistent-type-definitions/