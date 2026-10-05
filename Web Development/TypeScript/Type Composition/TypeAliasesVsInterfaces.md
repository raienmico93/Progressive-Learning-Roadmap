# TypeScript Type Aliases versus Interfaces: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Type aliases and interfaces are TypeScript's two primary mechanisms for naming types. Type aliases (`type Name = ...`) assign a name to any type expression, while interfaces (`interface Name { ... }`) declare named object types with a specific syntax. Both can describe the shape of objects, but they differ in what they can represent, how they can be extended, and how they behave in edge cases.

**Technical Definition**
A type alias is a declaration that binds an identifier to any type expression using the `type` keyword. It is fully transparent—the alias is resolved to the underlying type during type checking. An interface is a declaration that creates a named object type with a specific grammar and participates in the type-space namespace. Interfaces support declaration merging (multiple declarations with the same name are combined), can be extended with `extends`, and can be implemented by classes via `implements`. Type aliases cannot be merged, but can represent unions, intersections, primitives, tuples, conditional types, and mapped types—none of which interfaces can express directly. Both are erased at compile time and are compared structurally for compatibility. The choice between them is primarily about expressive power, extension semantics, and API design intent.

**Beginner-Friendly Explanation**
Both type aliases and interfaces let you name a type. If you want to describe an object with a `name` and `email`, you can write either `type User = { name: string; email: string }` or `interface User { name: string; email: string }`. For most everyday object shapes, they're interchangeable. The differences show up in edge cases: type aliases can represent *anything*—unions, primitives, tuples, complex computed types—while interfaces can only describe objects (including functions and constructors, which are objects). Interfaces can be "merged" (declared twice and combined) and "extended" with an `extends` keyword; type aliases use intersections (`&`) for extension and cannot be merged. Modern TypeScript developers often prefer type aliases for flexibility and interfaces for public API contracts. Neither is universally better—the right choice depends on what you're modeling.

### Key Characteristics

- **Structural compatibility**: Both are compared structurally in TypeScript's type system.
- **Expressive power**: Type aliases can represent any type; interfaces can only represent object types.
- **Extension**: Interfaces use `extends`; type aliases use intersections (`&`).
- **Declaration merging**: Only interfaces support merging; type aliases do not.
- **`implements`**: Classes can implement interfaces; classes cannot `implements` a type alias directly (though they can satisfy one structurally).
- **Recursion**: Both support recursion, but with different constraints and capabilities.
- **Excess property checking**: Both enforce it for object literals, with subtle differences.
- **Compile-time only**: Both are erased at runtime.

### Prerequisites

- Basic knowledge of TypeScript object types
- Familiarity with unions, intersections, and generics
- Understanding of type inference and structural typing
- Familiarity with classes and `implements`

### Related Programming Areas

- **API Design**: Choosing between type aliases and interfaces for public contracts
- **Module Augmentation**: Interfaces enable augmenting third-party types
- **Type-Level Programming**: Type aliases are essential for conditional and mapped types
- **Structural Typing**: Both rely on structural compatibility
- **Declaration Files**: `.d.ts` files commonly use both, with different conventions

### Core Concepts / Features

1. Structural Similarities (Defining Shapes of Objects)
2. Core Differences (Unions/Primitives vs. Objects/Classes)
3. Extension Capabilities (`extends` Keyword vs. Intersection Operators)
4. Declaration Merging (Unique to Interfaces)
5. Recursive Type Definitions (Allowed in Aliases via Conditional/Mapped Properties)
6. Excess Property Checking Behavior Differences
7. Choosing the Appropriate Abstraction (API Design Guidelines and Performance Considerations)


## 1. Structural Similarities (Defining Shapes of Objects)

### Definitions

**Core Definition**
Type aliases and interfaces are structurally similar when defining object shapes. Both can describe properties, methods, optional members, readonly members, index signatures, call signatures, and construct signatures. For simple object types, they are entirely interchangeable.

**Technical Definition**
Both type aliases and interfaces participate in TypeScript's structural type system. Two types—whether declared as aliases or interfaces—are compatible if their members are compatible, regardless of declaration form. When a type alias and an interface have identical members, they are mutually assignable. Both support property modifiers (`?`, `readonly`), method syntax, call signatures, construct signatures, and index signatures. Both are erased at compile time and produce no runtime code. For most object-shape use cases, the choice between them is a matter of style and additional features.

**Beginner-Friendly Explanation**
When you're just describing an object with some properties, type aliases and interfaces work the same way. `type User = { name: string }` and `interface User { name: string }` are basically identical. TypeScript treats them the same when checking compatibility. So for everyday object shapes, you can use either. The differences only matter when you need features that one supports but the other doesn't, like declaration merging or union types.

### Purposes

- To define object shapes with properties and methods.
- To enforce type safety for object literals and function parameters.
- To serve as contracts for classes via `implements`.
- To document the expected structure of data.
- To enable structural compatibility across different declarations.

### Syntax Rules and Structure

**General Syntax: Type Alias for Object Shape**

```typescript
type AliasName = {
  property1: Type1;
  property2?: Type2;
  readonly property3: Type3;
  method(param: ParamType): ReturnType;
};
```

**Component Breakdown**
- `type AliasName = { ... }`: Type alias declaration.
- Members use the same syntax as interfaces.

**General Syntax: Interface for Object Shape**

```typescript
interface InterfaceName {
  property1: Type1;
  property2?: Type2;
  readonly property3: Type3;
  method(param: ParamType): ReturnType;
}
```

**Component Breakdown**
- `interface InterfaceName { ... }`: Interface declaration.
- Members use identical syntax to type alias object literals.

**Syntax Rules**

- Both support optional (`?`) and readonly properties.
- Both support method declarations and function property syntax.
- Both support index signatures (`[key: string]: T`).
- Both support call and construct signatures.
- Both are structurally compared for compatibility.
- Both can be generic.
- Both are erased at compile time.

**Constraints and Limitations**

- Interfaces cannot represent unions or primitives (type aliases can).
- Type aliases cannot participate in declaration merging.
- Interfaces cannot be extended by interfaces with incompatible members (conflict errors).
- Both suffer from the same structural compatibility pitfalls.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Interchangeable Object Shapes

```typescript
// Step 1: Define the same shape as a type alias and an interface.
type UserType = {
  id: number;
  name: string;
  email?: string;
  readonly createdAt: Date;
};

interface UserInterface {
  id: number;
  name: string;
  email?: string;
  readonly createdAt: Date;
}

// Step 2: Create values of each.
const user1: UserType = {
  id: 1,
  name: "Alice",
  createdAt: new Date(),
};

const user2: UserInterface = {
  id: 2,
  name: "Bob",
  email: "bob@example.com",
  createdAt: new Date(),
};

// Step 3: They are mutually assignable.
const asInterface: UserInterface = user1;  // ✅
const asAlias: UserType = user2;            // ✅

console.log(asInterface.name);  // "Alice"
console.log(asAlias.email);     // "bob@example.com"

// Step 4: Both work in function signatures.
function greet(user: UserType): string {
  return `Hello, ${user.name}`;
}

console.log(greet(user2));  // "Hello, Bob"

function greet2(user: UserInterface): string {
  return `Hi, ${user.name}`;
}

console.log(greet2(user1));  // "Hi, Alice"
```

**Expected Output:**
```
Alice
bob@example.com
Hello, Bob
Hi, Alice
```

**Why This Output Occurs:** `UserType` and `UserInterface` have identical members, so they are structurally compatible. Values of either type are assignable to the other. Functions accepting one accept the other.

### Real-World Cases

**Case 1: Simple Data Models**
For simple data models with no union or primitive needs, either type aliases or interfaces work. Teams often standardize on one for consistency.

**Case 2: Function Parameter Types**
Inline or aliased object types work equally well for function parameters, with the choice often driven by code style.

**Case 3: Configuration Objects**
Configuration objects benefit from either form; interfaces are often preferred for public APIs, while type aliases are preferred for internal types.

---

## 2. Core Differences (Unions/Primitives vs. Objects/Classes)

### Definitions

**Core Definition**
The most significant difference between type aliases and interfaces is expressive power. Type aliases can name any type—unions, intersections, primitives, tuples, functions, conditional types, and mapped types. Interfaces can only name object types (including functions, constructors, and arrays, which are objects).

**Technical Definition**
Type aliases are general-purpose type-level bindings: `type X = ...` can be any type expression. Interfaces are restricted to object type declarations: they cannot represent union types, primitive types, tuple types, or conditional/mapped types directly. An interface is fundamentally an object type declaration with members. This restriction means interfaces cannot be used to define, for example, a union of string literals (`type Status = "active" | "inactive"`) or a tuple type (`type Point = [number, number]`). Type aliases are required for these. Conversely, interfaces can be extended, merged, and implemented by classes in ways that type aliases cannot.

**Beginner-Friendly Explanation**
Type aliases can name *any* type—unions, primitives, tuples, anything. Interfaces can only name object types. So if you want to define `type Status = "active" | "inactive"`, you *must* use a type alias. There's no way to write that as an interface. Similarly, `type Point = [number, number]` (a tuple) requires a type alias. Interfaces are specifically for describing object shapes. This is the biggest practical difference: type aliases are more versatile; interfaces are more specialized.

### Purposes

- To understand when a type alias is required versus when an interface works.
- To choose the right abstraction based on the kind of type being modeled.
- To leverage type aliases for unions, tuples, and computed types.
- To leverage interfaces for object contracts and class implementation.
- To design APIs that use the most appropriate abstraction for each type.

### Syntax Rules and Structure

**General Syntax: Type Alias for Union**

```typescript
type Status = "active" | "inactive" | "pending";
```

**Component Breakdown**
- Cannot be expressed as an interface.

**General Syntax: Type Alias for Tuple**

```typescript
type Point = [number, number];
```

**Component Breakdown**
- Cannot be expressed as an interface.

**General Syntax: Type Alias for Primitive**

```typescript
type UserId = string;
```

**Component Breakdown**
- Cannot be expressed as an interface.

**General Syntax: Interface for Object**

```typescript
interface User {
  name: string;
  email: string;
}
```

**Component Breakdown**
- Interfaces are restricted to object types.

**Syntax Rules**

- Type aliases can represent any type expression.
- Interfaces can only represent object types (including functions and constructors).
- Type aliases are required for unions, primitives, tuples, and conditional/mapped types.
- Interfaces are preferred for object shapes that need extension or merging.
- Both can be generic.
- Both support constraints (`extends` for generics).

**Constraints and Limitations**

- Interfaces cannot represent non-object types.
- Type aliases cannot be merged.
- Type aliases cannot be extended with `extends` (only intersected).
- Interfaces cannot be used with conditional or mapped types directly (though interfaces can be transformed by them).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Type Aliases for Non-Object Types

```typescript
// Step 1: Type alias for a union — interfaces cannot do this.
type Status = "active" | "inactive" | "pending";

// Step 2: Type alias for a tuple — interfaces cannot do this.
type Point = [x: number, y: number];

// Step 3: Type alias for a primitive — interfaces cannot do this.
type UserId = string;

// Step 4: Type alias for a function type — interfaces can do this too.
type Comparator<T> = (a: T, b: T) => number;

// Step 5: Use the aliases.
const status: Status = "active";
const point: Point = [10, 20];
const userId: UserId = "user-123";
const compareNumbers: Comparator<number> = (a, b) => a - b;

console.log(status);              // "active"
console.log(point);               // [10, 20]
console.log(userId);              // "user-123"
console.log(compareNumbers(5, 3)); // 2

// Step 6: Interfaces cannot replace these aliases.
// interface Status { }  // Cannot express "active" | "inactive" | "pending"
// interface Point { }   // Cannot express a tuple
// interface UserId { }  // Cannot express a primitive
```

**Expected Output:**
```
active
[ 10, 20 ]
user-123
2
```

**Why This Output Occurs:** The type aliases represent types that interfaces cannot express: a union of literals, a tuple, a primitive, and a function type. Only type aliases can bind names to these type expressions.

#### Example 2: Interfaces for Object Contracts

```typescript
// Step 1: Interface for an object contract.
interface Repository<T> {
  findById(id: number): T | undefined;
  save(entity: T): void;
  delete(id: number): void;
}

// Step 2: A class implements the interface.
class UserRepository implements Repository<{ id: number; name: string }> {
  private users: Map<number, { id: number; name: string }> = new Map();

  findById(id: number) {
    return this.users.get(id);
  }

  save(entity: { id: number; name: string }) {
    this.users.set(entity.id, entity);
  }

  delete(id: number) {
    this.users.delete(id);
  }
}

// Step 3: Use the interface as a contract.
const repo: Repository<{ id: number; name: string }> = new UserRepository();
repo.save({ id: 1, name: "Alice" });
console.log(repo.findById(1));  // { id: 1, name: 'Alice' }

// Step 4: A type alias cannot be used with `implements` in the same way.
type RepositoryType<T> = {
  findById(id: number): T | undefined;
  save(entity: T): void;
};

// class BadRepository implements RepositoryType<{ id: number }> { }
// ❌ Error: A class can only implement an object type or intersection of
// object types with statically known members.
// (Type aliases CAN be implemented if they are pure object types, but
// interfaces are idiomatic for this purpose.)
```

**Expected Output:**
```
{ id: 1, name: 'Alice' }
```

**Why This Output Occurs:** The `Repository<T>` interface defines a contract that `UserRepository` implements. Interfaces are idiomatic for class contracts because they clearly express intent and support declaration merging for augmentation.

### Real-World Cases

**Case 1: Union Types for State**
State modeling requires type aliases for union types (e.g., `type State = Loading | Success | Error`).

**Case 2: Tuple Types for Return Values**
Functions returning multiple values use tuple type aliases (e.g., `type MinMax = [number, number]`).

**Case 3: Object Contracts for Classes**
Classes implementing contracts use interfaces for clarity and mergeability (e.g., `interface Logger`).

---

## 3. Extension Capabilities (`extends` Keyword vs. Intersection Operators)

### Definitions

**Core Definition**
Interfaces extend other interfaces using the `extends` keyword, creating a subtype relationship. Type aliases extend other types using the intersection operator (`&`), creating a new type that combines all members. Both achieve similar results but with different semantics.

**Technical Definition**
The `extends` clause in an interface declaration creates a subtype relationship: the child interface is assignable to each parent. It supports multiple parents and can extend classes (inheriting instance members). The intersection operator (`&`) on type aliases creates a new type that has all members of the intersected types; the result is assignable to each constituent. Both approaches produce types with merged members. The key differences are: (1) interfaces can only extend interfaces/classes (not unions or primitives), while type aliases can intersect any types; (2) `extends` detects conflicts at the declaration site with specific error messages, while `&` computes the intersection (which may produce `never`); (3) interfaces support declaration merging of extended members, while intersections do not.

**Beginner-Friendly Explanation**
Interfaces extend using the `extends` keyword: `interface Child extends Parent { }`. Type aliases combine using the intersection operator: `type Child = Parent & { }`. Both give you a type with all members from both. The difference is subtle: `extends` is more restrictive (you can only extend interfaces and classes) but gives clearer error messages; `&` is more flexible (you can intersect anything) but can produce confusing `never` types if there are conflicts. For simple object extension, both work. For complex composition, intersections are more powerful.

### Purposes

- To build type hierarchies through extension or intersection.
- To reuse common members across related types.
- To compose types from multiple sources.
- To choose the right extension mechanism for the use case.
- To understand conflict resolution differences.

### Syntax Rules and Structure

**General Syntax: Interface Extension**

```typescript
interface Parent1 { a: string; }
interface Parent2 { b: number; }
interface Child extends Parent1, Parent2 {
  c: boolean;
}
// Child: { a: string; b: number; c: boolean; }
```

**Component Breakdown**
- `extends Parent1, Parent2`: Multiple inheritance of interfaces.
- The child has all parent members plus its own.

**General Syntax: Type Alias Intersection**

```typescript
type Parent1 = { a: string };
type Parent2 = { b: number };
type Child = Parent1 & Parent2 & {
  c: boolean;
};
// Child: { a: string; b: number; c: boolean; }
```

**Component Breakdown**
- `&` composes types into a single type.

**General Syntax: Extending a Class**

```typescript
class BaseClass {
  baseMethod(): void { }
}

interface Extended extends BaseClass {
  extraMethod(): void;
}
```

**Component Breakdown**
- Interfaces can extend classes, inheriting instance members.

**Syntax Rules**

- Interfaces use `extends` for extension; type aliases use `&`.
- Interfaces can extend multiple parents (comma-separated).
- Type aliases can intersect multiple types.
- Interfaces can extend classes (instance members only).
- Type aliases can intersect with anything (including unions, with care).
- Interface extension detects conflicts at declaration; intersections compute them.
- Interfaces support declaration merging of extended members; intersections do not.

**Constraints and Limitations**

- Interfaces cannot extend union types or primitives.
- Type aliases cannot use `extends` (only intersections).
- Intersections of conflicting primitives produce `never`, which can be confusing.
- Interface extension errors are usually clearer than intersection errors.
- Type aliases with intersections cannot be extended further with `extends` (only more intersections).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Interface Extension vs. Type Alias Intersection

```typescript
// Step 1: Define base types.
interface HasId { id: number; }
interface HasTimestamps { createdAt: Date; updatedAt: Date; }

// Step 2: Extend with interfaces.
interface Entity extends HasId, HasTimestamps {
  name: string;
}

const entity: Entity = {
  id: 1,
  createdAt: new Date(),
  updatedAt: new Date(),
  name: "Alice",
};
console.log(entity.name);  // "Alice"

// Step 3: Compose with type aliases and intersections.
type AliasEntity = HasId & HasTimestamps & {
  name: string;
};

const aliasEntity: AliasEntity = {
  id: 2,
  createdAt: new Date(),
  updatedAt: new Date(),
  name: "Bob",
};
console.log(aliasEntity.name);  // "Bob"

// Step 4: Both are structurally compatible.
const compatible1: Entity = aliasEntity;  // ✅
const compatible2: AliasEntity = entity;  // ✅
console.log(compatible1.name);  // "Bob"
console.log(compatible2.name);  // "Alice"

// Step 5: Interface extension errors are clearer for conflicts.
// interface BadEntity extends HasId {
//   id: string;  // ❌ Error: Interface 'BadEntity' incorrectly extends 'HasId'.
// }
// vs. type alias:
type BadAlias = HasId & { id: string };
// id: number & string = never — the error appears later when using the type.
```

**Expected Output:**
```
Alice
Bob
Bob
Alice
```

**Why This Output Occurs:** Both `Entity` (via `extends`) and `AliasEntity` (via `&`) produce types with `id`, `createdAt`, `updatedAt`, and `name`. The resulting types are structurally compatible. Interface extension produces clearer error messages for conflicts; intersection produces `never` for conflicting primitives.

#### Example 2: Type Alias Intersection with Unions

```typescript
// Step 1: Interfaces cannot extend unions.
type Status = "active" | "inactive";

// interface StatusEntity extends Status { }  // ❌ Error: An interface can only extend an object type.

// Step 2: Type aliases can intersect with unions.
type StatusObject = { status: Status } & { timestamp: Date };

const obj: StatusObject = {
  status: "active",
  timestamp: new Date(),
};
console.log(obj.status);  // "active"

// Step 3: Intersect with a union directly (with parentheses).
type Combined = ({ kind: "a"; a: number } | { kind: "b"; b: string }) & { id: number };

const combinedA: Combined = { kind: "a", a: 1, id: 1 };
const combinedB: Combined = { kind: "b", b: "x", id: 2 };

console.log(combinedA);  // { kind: 'a', a: 1, id: 1 }
console.log(combinedB);  // { kind: 'b', b: 'x', id: 2 }

// Step 4: The union distributes over the intersection.
// Combined is equivalent to:
// ({ kind: "a"; a: number; id: number }) | ({ kind: "b"; b: string; id: number })

// Step 5: Interfaces cannot express this pattern directly.
// interface Combined extends ... { }  // Cannot extend a union.
```

**Expected Output:**
```
active
{ kind: 'a', a: 1, id: 1 }
{ kind: 'b', b: 'x', id: 2 }
```

**Why This Output Occurs:** The `StatusObject` type intersects an object with a union-containing object. The `Combined` type intersects a union with an object, distributing the `id` property across all union members. Interfaces cannot express either pattern because they cannot extend unions.

### Real-World Cases

**Case 1: Domain Entity Hierarchies**
Domain entities use `extends` for clear hierarchy: `Entity` → `User` → `AdminUser`.

**Case 2: Mixin Composition**
Mixins use intersections: `Timestamped & Serializable & Loggable`.

**Case 3: Union Distribution**
Intersections with unions are used to add common properties to all union members: `(ActionA | ActionB) & { timestamp: Date }`.

---

## 4. Declaration Merging (Unique to Interfaces)

### Definitions

**Core Definition**
Declaration merging is a TypeScript feature where multiple declarations with the same name in the same scope are combined into a single declaration. Only interfaces support declaration merging; type aliases with the same name produce a duplicate identifier error.

**Technical Definition**
Interface declaration merging combines all members of multiple interface declarations with the same name. Non-function members must be unique (identical types are allowed; conflicting types are errors). Function members with the same name become overloads. Declaration merging is used for module augmentation (adding properties to third-party library types), global augmentation (adding to `Window`, `Array`, etc.), and splitting large interfaces across files. Type aliases cannot be merged—declaring two type aliases with the same name is an error. This makes interfaces uniquely suited for extensibility patterns where multiple sources contribute to a single type.

**Beginner-Friendly Explanation**
Declaration merging means you can declare the same interface multiple times, and TypeScript combines them. This is unique to interfaces—you can't do it with type aliases. If you declare `interface User { name: string }` in one file and `interface User { email: string }` in another (same scope), TypeScript creates a single `User` interface with both `name` and `email`. This is used to extend library types without modifying them: you can add properties to `Window` or `Express.Request` by declaring the interface again. Type aliases don't support this—if you try to declare the same type alias twice, you get an error.

### Purposes

- To extend third-party library types without modifying their source.
- To split large interfaces across multiple files for organization.
- To enable plugin systems where plugins contribute additional members.
- To augment global types (`Window`, `Array`, `Document`).
- To provide a mechanism for library authors to allow consumer extension.

### Syntax Rules and Structure

**General Syntax: Interface Merging**

```typescript
// File 1
interface User {
  name: string;
}

// File 2 (same scope)
interface User {
  email: string;
}

// Merged: interface User { name: string; email: string; }
```

**Component Breakdown**
- Multiple declarations with the same name are merged.

**General Syntax: Module Augmentation**

```typescript
import "some-library";

declare module "some-library" {
  interface LibraryInterface {
    newProperty: string;
  }
}
```

**Component Breakdown**
- Augments the library's interface without modifying the source.

**General Syntax: Global Augmentation**

```typescript
declare global {
  interface Window {
    __APP_VERSION__: string;
  }
}

export {};
```

**Component Breakdown**
- Augments global interfaces like `Window`.

**Syntax Rules**

- Interfaces with the same name in the same scope are merged.
- Non-function members must have unique names (or identical types).
- Function members with the same name become overloads.
- Interfaces can merge with classes and namespaces.
- Type aliases cannot be merged (duplicate identifiers are errors).
- Module augmentation requires `declare module` inside a module file.
- Global augmentation requires `declare global` and `export {}`.

**Constraints and Limitations**

- Non-function members with conflicting types cause compile errors.
- Merging cannot be prevented (all declarations merge).
- Module augmentation cannot add new top-level exports, only modify existing ones.
- Global augmentation is discouraged for application code.
- Declaration merging can make types harder to understand.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Interface Merging

```typescript
// Step 1: First declaration.
interface Config {
  apiUrl: string;
}

// Step 2: Second declaration — merges with the first.
interface Config {
  timeout: number;
  retries?: number;
}

// Step 3: The merged interface has all members.
const config: Config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
};

console.log(config.apiUrl);   // "https://api.example.com"
console.log(config.timeout);  // 5000
console.log(config.retries);  // 3

// Step 4: Function members merge as overloads.
interface Calculator {
  add(a: number, b: number): number;
}

interface Calculator {
  add(a: string, b: string): string;
}

const calc: Calculator = {
  add(a: any, b: any) { return a + b; },
};

console.log(calc.add(1, 2));      // 3
console.log(calc.add("a", "b"));  // "ab"

// Step 5: Type aliases cannot be merged.
// type AliasConfig = { apiUrl: string };
// type AliasConfig = { timeout: number };
// ❌ Error: Duplicate identifier 'AliasConfig'.
```

**Expected Output:**
```
https://api.example.com
5000
3
3
ab
```

**Why This Output Occurs:** The two `Config` interface declarations merge into one with `apiUrl`, `timeout`, and `retries?`. The two `Calculator` declarations merge, and `add` becomes an overloaded function. Type aliases with the same name produce a duplicate identifier error.

#### Example 2: Module Augmentation

```typescript
// Step 1: Assume a library defines an interface.
// library.d.ts:
// export interface Config {
//   apiUrl: string;
// }

// Step 2: Augment the library's interface.
// augment.ts:
import { Config } from "some-library";

declare module "some-library" {
  interface Config {
    retries?: number;
    timeout?: number;
  }
}

// Step 3: Now Config has the additional properties.
const config: Config = {
  apiUrl: "https://api.example.com",
  retries: 3,
  timeout: 5000,
};

console.log(config.apiUrl);   // "https://api.example.com"
console.log(config.retries);  // 3

// Step 4: Global augmentation.
declare global {
  interface Window {
    __APP_VERSION__: string;
  }
}
export {};

window.__APP_VERSION__ = "1.0.0";
console.log(window.__APP_VERSION__);  // "1.0.0"
```

**Expected Output:**
```
https://api.example.com
3
1.0.0
```

**Why This Output Occurs:** Module augmentation adds `retries?` and `timeout?` to the library's `Config` interface without modifying the library. Global augmentation adds `__APP_VERSION__` to the global `Window` interface. Both use declaration merging, which is unique to interfaces.

### Real-World Cases

**Case 1: Express Request Augmentation**
Express middleware augments `Request` with `user`, `session`, etc., via declaration merging.

**Case 2: Passport User Augmentation**
Passport augments the `Express.User` interface to include application-specific user properties.

**Case 3: Plugin Systems**
Plugin systems use declaration merging to let plugins add properties to shared context interfaces.

**Case 4: Global Type Augmentation**
Libraries augment `Window` to add browser extension APIs or application globals.

---

## 5. Recursive Type Definitions (Allowed in Aliases via Conditional/Mapped Properties)

### Definitions

**Core Definition**
Both type aliases and interfaces support recursive type definitions (types that reference themselves). However, type aliases have unique recursive capabilities through conditional and mapped types that interfaces cannot express. Type aliases can be recursive through unions, intersections, conditionals, and mapped types, enabling type-level algorithms and complex recursive structures.

**Technical Definition**
TypeScript allows recursive type aliases with restrictions: the recursion must be "guarded" by an object type, array type, tuple type, or conditional type. Direct self-reference (`type X = X`) is an error. TypeScript 3.7 relaxed some restrictions, allowing recursive type aliases through conditional types and other constructs. Interfaces support recursion through object properties (e.g., `interface Tree { children: Tree[] }`). Type aliases can additionally express recursion through unions (`type Json = string | number | Json[] | { [key: string]: Json }`), conditional types (`type Flatten<T> = T extends Array<infer U> ? Flatten<U> : T`), and mapped types. This makes type aliases uniquely suited for type-level programming.

**Beginner-Friendly Explanation**
Both type aliases and interfaces can be recursive—a type can refer to itself. For example, a `Tree` interface can have `children: Tree[]`. But type aliases can do more: they can be recursive through unions (like `Json = string | number | Json[]`), through conditional types (like `Flatten<T>`), and through mapped types. This makes type aliases essential for type-level algorithms. Interfaces are limited to recursion through object properties. So if you need complex recursive types, type aliases are the tool.

### Purposes

- To model recursive data structures like trees, linked lists, and JSON.
- To implement type-level algorithms using conditional and mapped types.
- To express recursive unions and intersections.
- To define recursive utility types.
- To enable advanced type transformations.

### Syntax Rules and Structure

**General Syntax: Recursive Type Alias via Union**

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

**General Syntax: Recursive Type Alias via Conditional Type**

```typescript
type Flatten<T> = T extends Array<infer U> ? Flatten<U> : T;
```

**Component Breakdown**
- `Flatten<T>` recursively flattens nested arrays.

**General Syntax: Recursive Type Alias via Mapped Type**

```typescript
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};
```

**Component Breakdown**
- `DeepReadonly<T>` recursively makes all properties readonly.

**General Syntax: Recursive Interface**

```typescript
interface Tree {
  value: number;
  children: Tree[];
}
```

**Component Breakdown**
- Recursion via object property.

**Syntax Rules**

- Both type aliases and interfaces support recursion.
- Type aliases can recurse through unions, intersections, conditionals, and mapped types.
- Interfaces can recurse through object properties.
- Direct self-reference (`type X = X`) is an error.
- Recursion must be guarded (by object, array, tuple, or conditional).
- TypeScript 3.7+ relaxed some recursion restrictions.
- Deeply recursive types can cause compiler performance issues.

**Constraints and Limitations**

- Direct self-reference is not allowed.
- Interfaces cannot express recursive unions or conditional types.
- Recursive type aliases can cause "Type instantiation is excessively deep" errors.
- Deep recursion may cause compiler stack overflow.
- Recursive types in unions require careful narrowing.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Recursive JSON Type (Type Alias)

```typescript
// Step 1: Define a recursive JSON type alias.
type Json =
  | string
  | number
  | boolean
  | null
  | Json[]
  | { [key: string]: Json };

// Step 2: Create JSON values.
const jsonObject: Json = {
  name: "Alice",
  age: 30,
  hobbies: ["reading", "coding"],
  address: {
    street: "123 Main St",
    city: "Springfield",
    coordinates: { lat: 39.78, lng: -89.65 },
  },
};

// Step 3: Recursive processing.
function countKeys(value: Json): number {
  if (typeof value !== "object" || value === null) return 0;
  if (Array.isArray(value)) {
    return value.reduce((sum, item) => sum + countKeys(item), 0);
  }
  return Object.keys(value).length +
    Object.values(value).reduce((sum, v) => sum + countKeys(v), 0);
}

console.log(countKeys(jsonObject));  // 9

// Step 4: Interfaces cannot express this union-based recursion.
// interface Json { }  // Cannot represent string | number | Json[] | { ... }
```

**Expected Output:**
```
9
```

**Why This Output Occurs:** The `Json` type alias recursively describes all valid JSON values. The `countKeys` function recursively traverses the structure. Interfaces cannot express this because they cannot represent the union of primitives and objects.

#### Example 2: Recursive Conditional Type

```typescript
// Step 1: Define a recursive conditional type.
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object
    ? T[K] extends Function
      ? T[K]
      : DeepReadonly<T[K]>
    : T[K];
};

// Step 2: Apply to a nested object type.
interface Config {
  api: {
    url: string;
    headers: {
      auth: string;
      contentType: string;
    };
  };
  timeout: number;
}

type ReadonlyConfig = DeepReadonly<Config>;

// Step 3: All nested properties are readonly.
const config: ReadonlyConfig = {
  api: {
    url: "https://api.example.com",
    headers: {
      auth: "token",
      contentType: "application/json",
    },
  },
  timeout: 5000,
};

// config.api.url = "https://other.com";
// ❌ Error: Cannot assign to 'url' because it is a read-only property.

console.log(config.api.url);  // "https://api.example.com"

// Step 4: Interfaces cannot express this transformation.
// There is no interface syntax for "recursively make all properties readonly".
```

**Expected Output:**
```
https://api.example.com
```

**Why This Output Occurs:** The `DeepReadonly<T>` type alias recursively maps over all properties, making them readonly at every level. This is a type-level algorithm that interfaces cannot express because interfaces have no mechanism for recursive transformation.

### Real-World Cases

**Case 1: JSON and API Data**
Recursive `Json` type aliases model arbitrary JSON data from APIs.

**Case 2: Deep Utility Types**
`DeepPartial<T>`, `DeepReadonly<T>`, and `DeepRequired<T>` are recursive conditional type aliases.

**Case 3: Tree Structures**
File systems, organizational charts, and ASTs use recursive types (both interfaces and aliases).

**Case 4: Type-Level Algorithms**
Flatten, tuple manipulation, and string parsing use recursive conditional type aliases.

---

## 6. Excess Property Checking Behavior Differences

### Definitions

**Core Definition**
Excess property checking is a TypeScript feature that flags extra properties in object literals assigned to typed targets. Both type aliases and interfaces enforce excess property checking, but the behavior can differ subtly due to the way each is resolved and the presence of index signatures.

**Technical Definition**
Excess property checking applies to "fresh" object literals in assignment contexts. When the target type is an interface or a type alias representing an object type, excess properties on direct object literals produce errors. However, subtle differences arise: (1) interfaces with index signatures allow any excess properties, while type aliases with index signatures behave the same way; (2) type aliases representing unions of object types have different excess property checking behavior (improved in TypeScript 3.5); (3) type aliases representing intersections may allow properties from any constituent; (4) interfaces that are augmented (via declaration merging) may have additional properties that affect excess property checking. In practice, the differences are minor, but edge cases exist.

**Beginner-Friendly Explanation**
Excess property checking catches typos: if you write `{ name: "Alice", agge: 30 }` when the type expects `age`, TypeScript flags `agge` as an error. Both type aliases and interfaces do this. But there are subtle differences in edge cases: unions and intersections of object types behave differently, and index signatures change the rules. For example, if the target type has an index signature (`[key: string]: T`), *any* extra property is allowed. In general, the differences are small and rarely matter in practice, but it's worth knowing they exist.

### Purposes

- To understand when excess property checking applies and when it doesn't.
- To diagnose unexpected excess property errors.
- To design types that allow or disallow excess properties intentionally.
- To understand differences between type aliases and interfaces in edge cases.
- To use index signatures and unions appropriately.

### Syntax Rules and Structure

**General Syntax: Excess Property Check with Interface**

```typescript
interface Config {
  apiUrl: string;
  timeout?: number;
}

const config: Config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  // extra: "value",  // ❌ Error: 'extra' does not exist in type 'Config'.
};
```

**Component Breakdown**
- Extra properties on direct object literals are errors.

**General Syntax: Excess Property Check with Type Alias**

```typescript
type Config = {
  apiUrl: string;
  timeout?: number;
};

const config: Config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  // extra: "value",  // ❌ Error: 'extra' does not exist in type 'Config'.
};
```

**Component Breakdown**
- Same behavior as interfaces for simple object types.

**General Syntax: Index Signature Bypasses Excess Check**

```typescript
interface WithIndex {
  [key: string]: string;
}

const withIndex: WithIndex = {
  anything: "goes",   // ✅ Allowed
  evenExtra: "stuff", // ✅ Allowed
};
```

**Component Breakdown**
- Index signatures allow any property names.

**Syntax Rules**

- Excess property checking applies to direct object literals.
- It applies to both interfaces and type aliases.
- Variables bypass excess property checking (they're not "fresh").
- Index signatures allow any properties, bypassing excess checking.
- Union types of object types have special excess checking behavior (TS 3.5+).
- Intersection types allow properties from any constituent.
- Weak types (all optional properties) have additional checking rules.

**Constraints and Limitations**

- Excess property checking is easily bypassed by assigning to an intermediate variable.
- Union type excess checking can be confusing (mitigated in TS 3.5+).
- Index signatures disable excess checking entirely.
- Excess checking does not apply to nested properties in some cases.
- Type assertions bypass excess checking.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Excess Property Check with Index Signatures

```typescript
// Step 1: Interface without index signature — strict.
interface StrictConfig {
  apiUrl: string;
  timeout?: number;
}

const strict: StrictConfig = {
  apiUrl: "https://api.example.com",
  // extra: "nope",  // ❌ Error: 'extra' does not exist in type 'StrictConfig'.
};

// Step 2: Interface with index signature — allows extras.
interface LooseConfig {
  apiUrl: string;
  [key: string]: string | number | undefined;
}

const loose: LooseConfig = {
  apiUrl: "https://api.example.com",
  extra: "allowed",   // ✅ Allowed by index signature
  another: 42,        // ✅ Allowed
};

console.log(loose.extra);  // "allowed"

// Step 3: Type alias with index signature — same behavior.
type LooseAlias = {
  apiUrl: string;
  [key: string]: string | number | undefined;
};

const looseAlias: LooseAlias = {
  apiUrl: "https://api.example.com",
  extra: "allowed",  // ✅
};

console.log(looseAlias.extra);  // "allowed"
```

**Expected Output:**
```
allowed
allowed
```

**Why This Output Occurs:** The `StrictConfig` interface has no index signature, so excess properties are errors. The `LooseConfig` interface and `LooseAlias` type alias both have index signatures, which allow any property names. The behavior is the same for interfaces and type aliases with index signatures.

#### Example 2: Union Type Excess Checking

```typescript
// Step 1: Define a union of object types.
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

// Step 2: Direct object literal with matching discriminant.
const circle: Shape = { kind: "circle", radius: 5 };  // ✅
const square: Shape = { kind: "square", side: 4 };    // ✅

// Step 3: Extra properties not in the matched member are errors.
// const invalid: Shape = { kind: "circle", radius: 5, side: 4 };
// ❌ Error: Object literal may only specify known properties,
// and 'side' does not exist in type '{ kind: "circle"; radius: number; }'.

// Step 4: Assigning via variable bypasses the check.
const circleWithSide = { kind: "circle", radius: 5, side: 4 };
const viaVariable: Shape = circleWithSide;  // ✅ Allowed (variable, not literal)

// Step 5: Interfaces behave the same way for discriminated unions.
interface Circle { kind: "circle"; radius: number; }
interface Square { kind: "square"; side: number; }
type ShapeUnion = Circle | Square;

const circle2: ShapeUnion = { kind: "circle", radius: 5 };  // ✅
// const invalid2: ShapeUnion = { kind: "circle", radius: 5, side: 4 };
// ❌ Error: same as type alias version.
```

**Expected Output:** No runtime output (compile-time behavior only). The valid assignments compile; the invalid one produces a compile error.

**Why This Output Occurs:** Excess property checking works on discriminated unions by checking the object literal against the member selected by the discriminant. Extra properties not in that member are errors. Both type aliases and interfaces behave the same way for discriminated unions. Assigning via a variable bypasses the check.

### Real-World Cases

**Case 1: Configuration Objects**
Configuration objects benefit from excess property checking to catch typos in option names. Using interfaces or type aliases without index signatures enforces this.

**Case 2: API Response Validation**
API response types use excess property checking to catch unexpected fields. In practice, response data often includes extra fields, so a variable-based approach or index signature is common.

**Case 3: React Props**
React component props use excess property checking to catch typos in prop names (e.g., `onclick` vs `onClick`). Both interfaces and type aliases work.

---

## 7. Choosing the Appropriate Abstraction (API Design Guidelines and Performance Considerations)

### Definitions

**Core Definition**
Choosing between type aliases and interfaces involves weighing expressive power, extension mechanisms, API design intent, team conventions, and compiler performance. There is no universal rule—the choice depends on context. Modern TypeScript community guidance generally recommends type aliases for most cases, with interfaces for public API contracts and declaration merging.

**Technical Definition**
The choice between type aliases and interfaces affects: (1) what types can be represented (interfaces are objects-only; aliases are universal); (2) extension mechanisms (`extends` vs. `&`); (3) declaration merging (interfaces only); (4) error message clarity (interfaces often produce clearer errors); (5) compiler performance (interfaces are generally faster for large object types because they're checked more efficiently; type aliases with complex conditional/mapped types can be slower); (6) API evolution (interfaces can be augmented without breaking changes; type aliases cannot). Community conventions vary: some style guides prefer interfaces for objects and type aliases for everything else; others prefer type aliases for everything. The TypeScript team has stated no strong preference, though the Handbook notes that interfaces are more extensible.

**Beginner-Friendly Explanation**
Choosing between type aliases and interfaces is a design decision. Here's a simple rule of thumb: use type aliases for unions, primitives, tuples, and anything that isn't a plain object. Use interfaces for object shapes that will be implemented by classes, extended by multiple sources, or augmented by third parties. For everything else, either works—pick one and be consistent. Performance-wise, interfaces are slightly faster for very large object types, but the difference rarely matters. The most important consideration is: does the type need to be extended or merged? If yes, use an interface. Otherwise, type aliases are more flexible.

### Purposes

- To choose the abstraction that best fits the type's purpose.
- To follow team and community conventions consistently.
- To optimize compiler performance for large codebases.
- To design APIs that are extensible and maintainable.
- To leverage the unique strengths of each construct.

### Syntax Rules and Structure

**General Syntax: Decision Guidelines**

```typescript
// Use type aliases for:
type Union = "a" | "b";                     // Unions
type Tuple = [number, string];              // Tuples
type Primitive = string;                    // Primitives
type Function_ = (x: number) => string;     // Functions
type Conditional<T> = T extends string ? 1 : 0;  // Conditional types
type Mapped<T> = { [K in keyof T]: T[K] }; // Mapped types

// Use interfaces for:
interface PublicApi {                        // Public API contracts
  method(): void;
}

interface ExtensibleConfig {                 // Types that need merging
  apiUrl: string;
}

interface Implementable {                    // Class contracts
  execute(): void;
}
```

**Component Breakdown**
- Type aliases: unions, tuples, primitives, functions, computed types.
- Interfaces: public APIs, extensible/mergeable types, class contracts.

**Syntax Rules**

- Use type aliases for non-object types (unions, primitives, tuples).
- Use interfaces for object types that need extension or merging.
- Use interfaces for public API contracts (better error messages, mergeable).
- Use type aliases for complex computed types (conditional, mapped, template literal).
- Be consistent within a codebase or module.
- Consider compiler performance for very large object types.

**Constraints and Limitations**

- No universal rule—context matters.
- Team conventions may override general guidelines.
- Performance differences are usually negligible.
- Migrating between the two is possible but can be disruptive.
- Both have unique features that the other lacks.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Applying the Guidelines

```typescript
// ===== USE TYPE ALIASES =====

// Unions
type Status = "active" | "inactive" | "pending";

// Tuples
type Coordinate = [latitude: number, longitude: number];

// Primitives
type UserId = string;
type OrderId = string;

// Functions
type EventHandler = (event: Event) => void;

// Conditional types
type IsArray<T> = T extends any[] ? true : false;

// Mapped types
type PartialDeep<T> = {
  [K in keyof T]?: T[K] extends object ? PartialDeep<T[K]> : T[K];
};

// ===== USE INTERFACES =====

// Public API contracts
interface UserService {
  findById(id: UserId): Promise<User | null>;
  save(user: User): Promise<void>;
}

interface User {
  id: UserId;
  name: string;
  email: string;
}

// Class contracts
interface Repository<T> {
  find(id: string): Promise<T | null>;
  save(entity: T): Promise<void>;
}

class PostgresUserRepository implements Repository<User> {
  async find(id: string): Promise<User | null> { return null; }
  async save(user: User): Promise<void> { }
}

// Augmentable types
interface Window {
  __APP_VERSION__: string;
}

// ===== EITHER WORKS =====

// Simple object shapes
type PointA = { x: number; y: number };
interface PointB { x: number; y: number; }

// Function parameter types
function distance1(p: PointA): number { return Math.sqrt(p.x ** 2 + p.y ** 2); }
function distance2(p: PointB): number { return Math.sqrt(p.x ** 2 + p.y ** 2); }

console.log(distance1({ x: 3, y: 4 }));  // 5
console.log(distance2({ x: 6, y: 8 }));  // 10
```

**Expected Output:**
```
5
10
```

**Why This Output Occurs:** Type aliases are used for unions, tuples, primitives, functions, and computed types—none of which interfaces can express. Interfaces are used for public API contracts, class contracts, and augmentable types. For simple object shapes, both work, and the choice is stylistic.

#### Example 2: Extending Third-Party Types

```typescript
// Step 1: Library defines an interface (mergeable).
// library.d.ts:
// export interface Request {
//   url: string;
//   method: string;
// }

// Step 2: Application augments the interface.
declare module "some-library" {
  interface Request {
    authToken?: string;
    userId?: number;
  }
}

// Step 3: Now Request has the additional properties.
const request: import("some-library").Request = {
  url: "/api/users",
  method: "GET",
  authToken: "token-123",
  userId: 1,
};

console.log(request.authToken);  // "token-123"

// Step 4: If the library had used a type alias, augmentation would be impossible.
// type Request = { url: string; method: string };
// type Request = { authToken?: string };  // ❌ Error: Duplicate identifier.

// Step 5: For internal types, type aliases are often preferred.
type InternalRequest = {
  url: string;
  method: string;
  authToken?: string;
};

const internal: InternalRequest = {
  url: "/api/users",
  method: "GET",
  authToken: "token-456",
};

console.log(internal.authToken);  // "token-456"
```

**Expected Output:**
```
token-123
token-456
```

**Why This Output Occurs:** The library's `Request` interface can be augmented via declaration merging (unique to interfaces). The internal `InternalRequest` type alias is used for application-specific types where merging is not needed. This illustrates the choice: use interfaces for library types that consumers may augment; use type aliases for internal types.

### Real-World Cases

**Case 1: Public Libraries**
Public libraries use interfaces for their public API contracts (to allow consumer augmentation) and type aliases for unions, tuples, and computed types.

**Case 2: Application Code**
Application code often uses type aliases throughout for consistency, with interfaces reserved for class contracts and augmentable types.

**Case 3: Framework Design**
Frameworks (React, Angular, Vue) use both: interfaces for component props and lifecycle contracts, type aliases for unions and utility types.

**Case 4: Performance-Critical Code**
For very large object types, interfaces may perform slightly better because the compiler resolves them more directly. Type aliases with deep conditional/mapped types can slow compilation.

---

## References

- TypeScript Handbook: Type Aliases — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases
- TypeScript Handbook: Interfaces — https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Handbook: Differences Between Type Aliases and Interfaces — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces
- TypeScript Handbook: Declaration Merging — https://www.typescriptlang.org/docs/handbook/declaration-merging.html
- TypeScript 3.7 Release Notes (Recursive Type Aliases) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html
- TypeScript 3.5 Release Notes (Improved Excess Property Checks) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-5.html
- TypeScript 4.9 Release Notes (`satisfies` Operator) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html
- TypeScript Wiki: FAQ — Interfaces vs Type Aliases — https://github.com/microsoft/TypeScript/wiki/FAQ
- Effective TypeScript: Item 13 — Know the Differences Between type and interface
- Effective TypeScript: Item 14 — Use Type Operations and Generics to Avoid Repeating Yourself
- TypeScript ESLint: consistent-type-definitions — https://typescript-eslint.io/rules/consistent-type-definitions/
- Total TypeScript: Type vs Interface — https://www.totaltypescript.com/type-vs-interface
- Matt Pocock: Interfaces vs Types — https://www.totaltypescript.com/tips/interfaces-vs-types