# Interface vs Type Alias in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** In TypeScript, **interfaces** and **type aliases** are the two primary constructs for naming types. Both can describe object shapes, be generic, and be used in type annotations, making them **largely interchangeable for simple object types**. They diverge, however, in **extensibility, mergeability, the range of type expressions they can represent, compiler performance characteristics, error message quality, and structural compatibility behavior** — differences that determine which construct is appropriate for a given scenario.

**Technical Definition.** An **interface declaration** (`interface Name { ... }`) introduces an **open, mergeable object type** into the type space. It supports declaration merging, `extends` heritage, and is the recommended form for object shapes in the TypeScript Handbook and Google Style Guide. A **type alias declaration** (`type Name = TypeExpression`) introduces a **closed, non-mergeable binding** for any type expression, including unions, intersections, primitives, tuples, mapped types, and conditional types. Type aliases have an **implicit index signature** making them assignable to `Record<string, unknown>`, while interfaces do not; interfaces produce **cleaner error messages** and are **cached by the compiler** more efficiently than type intersections. Both are erased at compile time and have no runtime representation.

**Beginner-Friendly Explanation.** Interfaces and type aliases are **two ways to name a type**. For simple objects, they often look identical and work the same way. The main differences are: interfaces can be **added to later** (declaration merging), while type aliases cannot; interfaces use `extends` to build on other interfaces, while type aliases use `&` to combine types; type aliases can name **any** type (including unions and primitives), while interfaces can only describe objects; and interfaces tend to give **better error messages** in your editor. For object shapes, the TypeScript team recommends interfaces; for everything else, use type aliases.

---

### Key Characteristics

| Characteristic | Interface | Type Alias |
|---|---|---|
| **Object shapes** | ✅ Yes | ✅ Yes |
| **Unions** | ❌ No | ✅ Yes |
| **Intersections** | ❌ No (uses `extends`) | ✅ Yes (`&`) |
| **Primitives / Tuples** | ❌ No | ✅ Yes |
| **Mapped / Conditional types** | ❌ No | ✅ Yes |
| **Declaration merging** | ✅ Yes | ❌ No (duplicate identifier error) |
| **`extends` keyword** | ✅ Yes | ❌ No (uses `&`) |
| **Implicit index signature** | ❌ No | ✅ Yes |
| **Error message clarity** | Generally better for objects | Can expand and become harder to read |
| **Compiler caching** | ✅ Cached by name | Intersections recomputed |
| **`implements` clause** | ✅ Preferred | ✅ Works (object types only) |
| **Erased at runtime** | ✅ Yes | ✅ Yes |

---

### Prerequisites

- **TypeScript basics**: type annotations, interfaces, and structural typing.
- **Object types**: properties, optional properties, readonly properties, and index signatures.
- **Generics fundamentals**: type parameters and constraints (`<T extends U>`).
- **Union and intersection types** (`|`, `&`).
- **A TypeScript environment** (`tsc` or an editor with TypeScript support) to observe compiler behavior.

---

### Related Programming Areas

- **Structural typing** — both constructs rely on TypeScript’s structural compatibility model.
- **Declaration merging** — a feature unique to interfaces, used for module augmentation.
- **Type-level programming** — unions, intersections, mapped types, and conditional types are type-alias territory.
- **Compiler performance** — interface `extends` is cached; type intersections are recomputed.
- **Design principles** — Interface Segregation, composition over inheritance, and API contract design.

---

## Core Concepts / Features

---

## 1. Interface vs Type Alias (Core Comparison)

### Definitions

**Core Definition.** **Interfaces** and **type aliases** are the two TypeScript constructs for naming types. Interfaces are **open** and restricted to **object shapes**; type aliases are **closed** and can name **any type expression**. For simple object types, they are interchangeable; for unions, intersections, primitives, tuples, and mapped types, only type aliases work.

**Technical Definition.** An interface declaration introduces a **mergeable named object type**; multiple declarations in the same scope combine. A type alias declaration introduces a **fixed name for a type expression**; redeclaration is an error. Interfaces support `extends` heritage and are cached by the compiler; type aliases use intersection (`&`) and are recomputed on each use. Type aliases have an implicit index signature; interfaces do not.

**Beginner-Friendly Explanation.** Think of an interface as a **contract that can be extended later** and a type alias as a **nickname for a specific type**. For simple objects, either works. For unions, primitives, and advanced types, you must use `type`. For public object APIs and class contracts, `interface` is the traditional choice.

### Purposes

- **To choose the correct construct** for each type-naming scenario.
- **To avoid unexpected behavior** from declaration merging or implicit index signatures.
- **To maximize error message clarity** and compiler performance.
- **To align with team and ecosystem conventions**.

### Syntax Rules and Structure

**Interface declaration syntax:**

```
interface InterfaceName<T> extends Base1, Base2 {
  property: Type;
  method(params: Type): ReturnType;
}
```

**Type alias declaration syntax:**

```
type AliasName<T> = TypeExpression;
```

**Component breakdown:**

- `interface` — introduces a mergeable object type; supports `extends`.
- `type` — introduces a fixed alias for any type expression; supports `&` intersection.
- `extends` — interface heritage; cached by the compiler.
- `&` — intersection; recursively merges properties and recomputes on use.

**Syntax rules.**

- Interfaces can only describe **object types** (and function/constructor types via call/construct signatures).
- Type aliases can describe **any type**, including unions, intersections, primitives, tuples, mapped types, and conditional types.
- Interfaces merge when declared multiple times in the same scope; type aliases error on duplicate names.
- Type aliases have an implicit index signature; interfaces require an explicit one to be assignable to `Record<string, unknown>`.

**Constraints and limitations.**

- Interfaces cannot express unions, intersections, or mapped types directly.
- Type aliases cannot be augmented via declaration merging.
- Interface `extends` detects property conflicts at declaration; type `&` can silently produce `never` on conflicts.
- Error messages for type aliases may show the expanded type rather than the alias name.

### Annotated Complete Step by Step Code Examples

#### Example 1: Both constructs for a simple object

```typescript
// example_both_objects.ts
// Interface form
interface UserInterface {
  id: string;
  name: string;
}

// Type alias form (same shape)
type UserType = {
  id: string;
  name: string;
};

const u1: UserInterface = { id: "u1", name: "Ada" };
const u2: UserType = { id: "u2", name: "Grace" };

console.log(u1.name); // Expected: Ada
console.log(u2.name); // Expected: Grace
```

**Expected Output:**
```
Ada
Grace
```

**Why this output occurs.** Both constructs describe the same object shape and are structurally compatible. For simple objects, they are interchangeable. The choice depends on context and convention, not on a functional difference.

---

#### Example 2: Type alias for a union (interfaces cannot)

```typescript
// example_type_union_only.ts
// ✅ Type alias for a union.
type Status = "active" | "inactive" | "pending";

const status: Status = "active";
console.log(status); // Expected: active

// ❌ An interface cannot express this union.
```

**Expected Output:**
```
active
```

**Why this output occurs.** Unions are **only** expressible via type aliases. Interfaces are restricted to object shapes. This is a fundamental reason to use `type` for unions, primitives, and other non-object types.

---

#### Example 3: Interface declaration merging (types cannot)

```typescript
// example_interface_merge.ts
interface Box {
  width: number;
}

// Second declaration merges with the first.
interface Box {
  height: number;
}

const box: Box = { width: 10, height: 20 };
console.log(box.width + box.height); // Expected: 30

// ❌ Type aliases cannot merge.
// type Box = { width: number };
// type Box = { height: number }; // Error: Duplicate identifier.
```

**Expected Output:**
```
30
```

**Why this output occurs.** Interfaces support **declaration merging**: two `interface Box` declarations combine into one type with both properties. Type aliases are **closed** and cannot be augmented. This is the key reason to choose interfaces when third-party type augmentation is needed.

---

### Real-World Cases

- **Public object APIs**: interfaces are preferred for clearer error messages and mergeability.
- **Union-based domain modeling**: `type OrderState = "pending" | "shipped" | "delivered"` requires a type alias.
- **Third-party type augmentation**: adding properties to `Window` or library types requires an interface.
- **Utility type definitions**: `Partial<T>`, `Readonly<T>`, and `Record<K, V>` are type aliases.

---

## 2. When to Use Interfaces

### Definitions

**Core Definition.** Interfaces are the preferred construct when defining **object shapes that may be extended or merged**, particularly for **public APIs, class contracts, and third-party type augmentation**. The TypeScript Handbook, Google Style Guide, and Microsoft Style Guide all recommend interfaces for object type definitions.

**Technical Definition.** Interfaces are appropriate when the type is an **object shape** (properties, methods, accessors), when **declaration merging** is required (module augmentation, global type extension), when the type will be used in a **class `implements` clause**, or when **cleaner error messages** and **compiler caching** are priorities. Interfaces use `extends` for composition, which detects property conflicts at declaration and caches the resulting type by name.

**Beginner-Friendly Explanation.** Use an interface when you are describing **what an object looks like**, especially if other code might need to **add to that description later** (like extending a library’s types) or if a class needs to **implement** it.

### Purposes

- **To define public object contracts** with clear, focused error messages.
- **To enable declaration merging** for augmenting third-party library types and global objects.
- **To provide class contracts** via `implements` that read naturally.
- **To leverage compiler caching** for faster type checking in large hierarchies.
- **To detect property conflicts at declaration** rather than silently producing `never`.

### Syntax Rules and Structure

**General syntax:**

```
interface InterfaceName<T> extends Base1, Base2 {
  property: Type;
  optional?: Type;
  readonly immutable: Type;
  method(params: Type): ReturnType;
  [index: string]: Type;
}
```

**Component breakdown:**

- `interface` — introduces a mergeable object type.
- `extends` — composition; cached and conflict-aware.
- Members — properties, methods, index signatures, call signatures, construct signatures.

**Syntax rules.**

- Interfaces can only describe object types (including function/constructor types via call/construct signatures).
- Interfaces merge when declared multiple times in the same scope.
- Interfaces use `extends` for inheritance, not `&`.
- Interfaces can be implemented by classes.
- Interfaces can be augmented via `declare module` or `declare global`.

**Constraints and limitations.**

- Interfaces cannot represent unions, intersections, primitives, tuples, or mapped/conditional types.
- Declaration merging can be surprising; use `type` if merging is undesirable.
- Interfaces lack an implicit index signature, so they are not assignable to `Record<string, unknown>` without an explicit index signature.

### Annotated Complete Step by Step Code Examples

#### Example 1: Interface for a class contract

```typescript
// example_interface_class.ts
interface UserContract {
  id: string;
  name: string;
  getDisplayName(): string;
}

class Employee implements UserContract {
  constructor(public id: string, public name: string) {}

  getDisplayName(): string {
    return `${this.name} (${this.id})`;
  }
}

const emp = new Employee("e1", "Ada");
console.log(emp.getDisplayName()); // Expected: Ada (e1)
```

**Expected Output:**
```
Ada (e1)
```

**Why this output occurs.** The interface defines a contract that `Employee` implements. The `implements` clause reads naturally with an interface. The compiler verifies that all required members are present.

---

#### Example 2: Interface for declaration merging (library augmentation)

```typescript
// example_interface_merge_library.ts
// Simulate a library interface.
interface LibraryConfig {
  apiUrl: string;
}

// Augment it with an additional property.
interface LibraryConfig {
  timeout?: number;
}

const config: LibraryConfig = { apiUrl: "https://api.example.com", timeout: 5000 };
console.log(config.timeout); // Expected: 5000
```

**Expected Output:**
```
5000
```

**Why this output occurs.** The two `LibraryConfig` declarations merge, adding `timeout` to the original `apiUrl`. This is the primary mechanism for augmenting third-party types without modifying source code.

---

### Real-World Cases

- **React props interfaces**: `interface ButtonProps { label: string; onClick: () => void; }` — public contracts with clear errors.
- **Global augmentation**: `declare global { interface Window { analytics: Analytics; } }`.
- **Class contracts**: `class UserService implements IUserService { ... }`.
- **Library APIs**: exported interfaces that consumers may extend or merge.

---

## 3. When to Use Type Aliases

### Definitions

**Core Definition.** Type aliases are the **only** construct for naming **unions, intersections, primitives, tuples, mapped types, conditional types, and recursive types**. They are also the **safer default** for generic utility-style objects because they avoid the surprises of declaration merging and have an implicit index signature.

**Technical Definition.** Type aliases are required when the type expression is **not a plain object shape**: union types (`A | B`), intersection types (`A & B`), primitive aliases (`type ID = string`), tuples (`[string, number]`), template literal types, mapped types, conditional types, and recursive type references. They are also preferred for **one-off combinations** and **utility-style objects** passed to generic functions expecting `Record<string, unknown>`.

**Beginner-Friendly Explanation.** Use a type alias when you need to name something that **is not a simple object**—like a union of strings, a tuple, or a function type—or when you want to avoid the surprising behavior of interfaces merging and not having an implicit index signature.

### Purposes

- **To name unions, intersections, primitives, tuples, and advanced type expressions** that interfaces cannot represent.
- **To provide a safer default** for generic utility-style objects with implicit index signatures.
- **To avoid unexpected declaration merging** that can silently modify types.
- **To compose types** via intersection (`&`) for one-off combinations.
- **To model recursive data structures** such as JSON and trees.

### Syntax Rules and Structure

**General syntax:**

```
type AliasName<T extends Constraint = Default> = TypeExpression;
```

**Component breakdown:**

- `type` — keyword.
- `AliasName` — identifier.
- `<T extends Constraint = Default>` — optional generics.
- `=` — assignment.
- `TypeExpression` — any valid TypeScript type.

**Syntax rules.**

- Type aliases can name any type expression.
- They are **closed**: redeclaration in the same scope is an error.
- They have an **implicit index signature** (assignable to `Record<string, unknown>`).
- They compose via intersection (`&`), not `extends`.
- They are hoisted and can be referenced before declaration.

**Constraints and limitations.**

- Type aliases cannot be augmented via declaration merging.
- Error messages may expand the alias, reducing readability.
- Intersection (`&`) can silently produce `never` on conflicting properties.
- Intersections are recomputed on each use, which can affect compiler performance in deep hierarchies.

### Annotated Complete Step by Step Code Examples

#### Example 1: Type alias for a union of literals

```typescript
// example_type_union.ts
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

const method: HttpMethod = "GET";
console.log(method); // Expected: GET

// const bad: HttpMethod = "PATCH"; // Error: not assignable.
```

**Expected Output:**
```
GET
```

**Why this output occurs.** The union of string literals is expressible **only** via a type alias. The compiler restricts `method` to the four permitted literals.

---

#### Example 2: Type alias for a tuple

```typescript
// example_type_tuple.ts
type Coordinate = [number, number];

const point: Coordinate = [10, 20];
console.log(point[0], point[1]); // Expected: 10 20
```

**Expected Output:**
```
10 20
```

**Why this output occurs.** Tuples are type-alias territory. The compiler checks that the array has exactly two numbers and provides typed access to the elements.

---

#### Example 3: Type alias with implicit index signature

```typescript
// example_type_index_signature.ts
type UserType = { id: number; name: string };
interface UserInterface { id: number; name: string }

function processData(data: Record<string, unknown>) {
  console.log(data);
}

const fromType: UserType = { id: 1, name: "Ada" };
const fromInterface: UserInterface = { id: 1, name: "Ada" };

processData(fromType); // ✅ Works: type has implicit index signature.
// processData(fromInterface); // ❌ Error: index signature missing.
```

**Expected Output:**
```
{ id: 1, name: 'Ada' }
```

**Why this output occurs.** Type aliases have an **implicit index signature** of `Record<PropertyKey, unknown>`, so `UserType` is assignable to `Record<string, unknown>`. Interfaces do **not** have this implicit signature, so `UserInterface` is rejected. This is a practical reason to use type aliases for utility-style objects passed to generic functions.

---

### Real-World Cases

- **API unions**: `type ApiResult<T> = { ok: true; data: T } | { ok: false; error: string }`.
- **Domain primitives**: `type UserID = string;`, `type Seconds = number;`.
- **Recursive JSON modeling**: `type Json = string | number | boolean | null | Json[] | { [key: string]: Json };`.
- **Utility types**: `Partial<T>`, `Readonly<T>`, `Record<K, V>`, and `Pick<T, K>` are all type aliases.
- **Generic utility objects**: type aliases passed to functions expecting `Record<string, unknown>`.

---

## 4. Comparison Table and Detailed Differences

### Definitions

**Core Definition.** A structured comparison of interfaces and type aliases across **syntax, capabilities, performance, error messages, declaration merging, index signatures, and class implementation**.

**Technical Definition.** The key differences are: **declaration merging** (interfaces merge; type aliases error), **type expression coverage** (type aliases cover all types; interfaces only objects), **composition mechanism** (`extends` vs `&`), **conflict handling** (interfaces error; intersections produce `never`), **implicit index signatures** (type aliases have them; interfaces do not), **compiler caching** (interfaces cached; intersections recomputed), and **error message clarity** (interfaces clearer for objects).

**Beginner-Friendly Explanation.** The comparison table below summarizes the practical differences. The most important ones for daily use are: interfaces merge and type aliases do not; type aliases can name unions and primitives while interfaces cannot; and type aliases work better with `Record<string, unknown>`.

### Purposes

- **To provide a quick reference** for deciding between the two constructs.
- **To highlight behavioral differences** that cause real bugs.
- **To document performance and error-message trade-offs**.

### Syntax Rules and Structure

| Aspect | Interface | Type Alias |
|---|---|---|
| **Object shape** | ✅ Yes | ✅ Yes |
| **Union** | ❌ No | ✅ Yes |
| **Intersection** | ❌ No (uses `extends`) | ✅ Yes (`&`) |
| **Primitive** | ❌ No | ✅ Yes |
| **Tuple** | ❌ No | ✅ Yes |
| **Mapped type** | ❌ No | ✅ Yes |
| **Conditional type** | ❌ No | ✅ Yes |
| **Declaration merging** | ✅ Yes | ❌ No |
| **`extends` keyword** | ✅ Yes | ❌ No |
| **`&` intersection** | ❌ No | ✅ Yes |
| **Implicit index signature** | ❌ No | ✅ Yes |
| **Compiler caching** | ✅ Cached by name | Intersections recomputed |
| **Conflict detection** | ✅ At declaration | ❌ Silently `never` |
| **Error message clarity** | ✅ Better for objects | May expand |
| **Class `implements`** | ✅ Preferred | ✅ Works (objects) |
| **Recursive types** | ❌ No (directly) | ✅ Yes |

**Syntax rules.**

- Use `extends` for interface inheritance; use `&` for type composition.
- Interfaces merge; type aliases do not.
- Type aliases cover all type expressions; interfaces cover objects only.

**Constraints and limitations.**

- Interfaces cannot express unions, intersections, primitives, tuples, or mapped/conditional types.
- Type aliases cannot merge and lack interface-style conflict detection.
- Intersections can silently produce `never` on conflicting properties, masking bugs.

### Annotated Complete Step by Step Code Examples

#### Example 1: Conflict handling — interface errors, type produces `never`

```typescript
// example_conflict.ts
interface A { x: string }
interface B { x: number }

// ❌ Interface extends detects the conflict.
// interface C extends A, B {}
// Error: Interface 'C' cannot simultaneously extend types 'A' and 'B'.
//        Named property 'x' of types 'A' and 'B' are not identical.

// ✅ Type intersection silently produces never.
type CA = { x: string };
type CB = { x: number };
type CC = CA & CB; // No error — x becomes never.

console.log("Interfaces detect conflicts; intersections produce never");
```

**Expected Output:**
```
Interfaces detect conflicts; intersections produce never
```

**Why this output occurs.** Interface `extends` checks for incompatible properties and reports a clear error. Type intersection (`&`) recursively merges properties; conflicting types reduce to `never` without an error. This can mask bugs, which is why interfaces are preferred for inheritance hierarchies.

---

#### Example 2: Performance — interface caching vs intersection recomputation

```typescript
// example_performance.ts
// Interface extends: cached by name.
interface Base { id: string; name: string; }
interface User extends Base { age: number; }

// Type intersection: recomputed on each use.
type BaseType = { id: string; name: string };
type UserType = BaseType & { age: number };

// Both are structurally equivalent, but the compiler handles them differently.
const user: User = { id: "u1", name: "Ada", age: 36 };
const user2: UserType = { id: "u2", name: "Grace", age: 85 };

console.log(user.name);  // Expected: Ada
console.log(user2.name); // Expected: Grace
```

**Expected Output:**
```
Ada
Grace
```

**Why this output occurs.** `User` is cached by the compiler under its name, so repeated checks are fast. `UserType` is an intersection that must be recursively flattened on each use. For large, deeply nested hierarchies, this difference can affect compile times.

---

### Real-World Cases

- **Deep type hierarchies**: interface `extends` for performance and conflict detection.
- **One-off utility types**: type intersections for quick composition.
- **Public APIs**: interfaces for cleaner error messages and mergeability.
- **Generic utility functions**: type aliases for implicit index signatures.

---

## 5. Interface Best Practices

### Definitions

**Core Definition.** **Interface best practices** are guidelines for using interfaces in ways that maximize **error message clarity, compiler performance, extensibility, and API stability** — primarily by reserving interfaces for **object shapes, class contracts, public APIs, and declaration-merging scenarios**.

**Technical Definition.** Best practices include: **preferring interfaces for object shapes and public APIs** (TypeScript Handbook, Google Style Guide, Microsoft Style Guide); **using interfaces for class `implements` contracts**; **using `extends` rather than `&` for inheritance hierarchies** to benefit from caching and conflict detection; **avoiding interfaces for unions, primitives, and utility types**; and **documenting interfaces that are intended for augmentation**.

**Beginner-Friendly Explanation.** Use interfaces when you are defining **objects that other code will implement or extend**, especially public APIs. Use `extends` rather than `&` when building hierarchies. Do not use interfaces for unions or primitives—that is what type aliases are for.

### Purposes

- **To produce clearer compiler errors** for object shapes.
- **To enable safe extension and augmentation** of public APIs.
- **To leverage compiler caching** in large hierarchies.
- **To align with ecosystem conventions** (Handbook, Google, Microsoft).

### Syntax Rules and Structure

**General syntax:**

```
interface InterfaceName extends Base {
  // object members
}
```

**Component breakdown:**

- `interface` — object-only, mergeable.
- `extends` — cached, conflict-aware inheritance.

**Syntax rules.**

- Use interfaces for object shapes, class contracts, and public APIs.
- Use `extends` for interface inheritance.
- Document interfaces intended for declaration merging.
- Do not use interfaces for unions, primitives, tuples, or mapped types.

**Constraints and limitations.**

- Interfaces cannot express non-object types.
- Declaration merging can be surprising; use ESLint `no-redeclare` if needed.

### Annotated Complete Step by Step Code Examples

#### Example 1: Interface `extends` for a clean hierarchy

```typescript
// example_interface_extends.ts
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

const dog: Dog = { name: "Rex", breed: "Labrador" };
console.log(dog.name);  // Expected: Rex
console.log(dog.breed); // Expected: Labrador
```

**Expected Output:**
```
Rex
Labrador
```

**Why this output occurs.** `Dog` extends `Animal`, inheriting `name` and adding `breed`. The compiler caches `Dog` by name and detects conflicts at declaration. This is the recommended pattern for object inheritance.

---

#### Example 2: Interface for a public API contract

```typescript
// example_interface_public_api.ts
export interface UserProfile {
  id: string;
  name: string;
  email?: string;
}

export class UserService {
  getUser(id: string): UserProfile {
    return { id, name: "Ada" };
  }
}
```

**Expected Output:**
```
(no output; type-level example)
```

**Why this output occurs.** The exported interface provides a stable public contract. Consumers can rely on the shape and receive clear error messages if they misuse it. Interfaces are preferred for public APIs because they are mergeable and produce better tooling feedback.

---

### Real-World Cases

- **React component props**: exported interfaces for public component APIs.
- **Service contracts**: interfaces implemented by service classes.
- **Global augmentation**: `declare global { interface Window { ... } }`.
- **Library type definitions**: exported interfaces that consumers may extend.

---

## 6. Type Alias Best Practices

### Definitions

**Core Definition.** **Type alias best practices** are guidelines for using type aliases as the **safer default** for non-object types, utility-style objects, unions, intersections, and advanced type expressions — while avoiding their pitfalls (lack of merging, `never` conflicts, expanded error messages).

**Technical Definition.** Best practices include: **using type aliases by default until an interface-specific feature is needed** (Total TypeScript recommendation); **using type aliases for unions, intersections, primitives, tuples, mapped types, conditional types, and recursive types**; **using type aliases for generic utility objects** because of their implicit index signature; **using `&` only for one-off combinations** rather than deep hierarchies; and **being aware that intersections silently produce `never` on conflicts**.

**Beginner-Friendly Explanation.** Use type aliases when you need to name **anything that is not a plain object**—unions, tuples, primitives, functions—or when you want the safer default for generic utility objects. Avoid building deep hierarchies with `&`; use interfaces for those.

### Purposes

- **To name unions, tuples, primitives, and advanced types** that interfaces cannot represent.
- **To provide a safer default** with no declaration merging surprises.
- **To enable assignment to `Record<string, unknown>`** through implicit index signatures.
- **To compose types quickly** for one-off utility shapes.

### Syntax Rules and Structure

**General syntax:**

```
type AliasName<T> = TypeExpression;
```

**Component breakdown:**

- `type` — keyword.
- `=` — assignment.
- `TypeExpression` — unions, intersections, primitives, tuples, mapped types, conditional types, recursive types.

**Syntax rules.**

- Use type aliases for unions (`|`), intersections (`&`), primitives, tuples, and advanced types.
- Use type aliases for generic utility objects passed to `Record<string, unknown>`.
- Use `&` for one-off composition; avoid deep hierarchies.
- Document aliases that intentionally rely on implicit index signatures.

**Constraints and limitations.**

- Type aliases cannot merge.
- Intersections can silently produce `never` on conflicts.
- Error messages may expand aliases.

### Annotated Complete Step by Step Code Examples

#### Example 1: Type alias for a discriminated union

```typescript
// example_type_discriminated_union.ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
  }
}

console.log(area({ kind: "circle", radius: 1 }).toFixed(2)); // Expected: 3.14
console.log(area({ kind: "square", side: 2 }));             // Expected: 4
```

**Expected Output:**
```
3.14
4
```

**Why this output occurs.** Discriminated unions are type-alias territory. The `kind` discriminant allows the compiler to narrow the shape in each `switch` branch. Interfaces cannot express this union.

---

#### Example 2: Type alias with implicit index signature for utility functions

```typescript
// example_type_utility_index.ts
type User = { id: number; name: string };

function logObject(obj: Record<string, unknown>) {
  console.log(obj);
}

const user: User = { id: 1, name: "Ada" };
logObject(user); // ✅ Works: type alias has implicit index signature.
```

**Expected Output:**
```
{ id: 1, name: 'Ada' }
```

**Why this output occurs.** `User` is a type alias, so it has an implicit index signature and is assignable to `Record<string, unknown>`. If `User` were an interface, this call would fail without an explicit index signature.

---

### Real-World Cases

- **API results**: `type ApiResult<T> = { ok: true; data: T } | { ok: false; error: string }`.
- **Utility objects**: type aliases passed to generic functions expecting `Record<string, unknown>`.
- **Domain primitives**: `type UserID = string;`, `type OrderID = string;`.
- **JSON modeling**: `type Json = string | number | boolean | null | Json[] | { [key: string]: Json };`.

---

## 7. Best Practices (Combined)

### Definitions

**Core Definition.** **Combined best practices** synthesize the recommendations from the TypeScript Handbook, Google Style Guide, and community experts into a **decision framework** for choosing between interfaces and type aliases.

**Technical Definition.** The consensus framework is: **prefer interfaces for object shapes, public APIs, and class contracts** (Handbook, Google, Microsoft); **prefer type aliases for unions, intersections, primitives, tuples, mapped types, conditional types, and utility-style objects**; **use `extends` rather than `&` for deep hierarchies** to benefit from caching and conflict detection; **use type aliases as the safer default** when no interface-specific feature is needed (Total TypeScript); and **follow a consistent team convention** when both work.

**Beginner-Friendly Explanation.** The simple rule is: **use `interface` for object shapes and class contracts; use `type` for everything else**. When both work (simple objects), pick one and be consistent. If you need declaration merging, use `interface`. If you need a union or implicit index signature, use `type`.

### Purposes

- **To provide a clear decision rule** that covers all scenarios.
- **To avoid the pitfalls** of declaration merging, `never` conflicts, and missing index signatures.
- **To align with official and community guidance**.
- **To maintain consistency** across a codebase.

### Syntax Rules and Structure

**Decision tree:**

```
Is the type an object shape used as a public contract or class implementation?
  ├── Yes → Use interface.
  │        ├── Does it need declaration merging? → interface (required).
  │        └── Is it a deep hierarchy? → interface extends (cached).
  └── No → Is it a union, intersection, primitive, tuple, mapped type,
           conditional type, or utility object?
           ├── Yes → Use type.
           └── No → Re-evaluate; type is usually the safer default.
```

**Syntax rules.**

- Interfaces for objects, public APIs, class contracts, and merging.
- Type aliases for unions, intersections, primitives, tuples, and utility objects.
- `extends` for deep hierarchies; `&` for one-off combinations.
- Be consistent within a codebase.

**Constraints and limitations.**

- Declaration merging can cause unexpected behavior; document interfaces intended for augmentation.
- Type intersections can silently produce `never`; prefer `extends` in hierarchies.
- Type aliases lack interface-style conflict detection.

### Annotated Complete Step by Step Code Examples

#### Example 1: Interface for object, type for union

```typescript
// example_best_combined.ts
// ✅ Interface for an object shape.
interface User {
  id: string;
  name: string;
}

// ✅ Type for a union.
type UserRole = "admin" | "editor" | "viewer";

// ✅ Type for an intersection.
type AdminUser = User & { role: "admin" };

const user: User = { id: "u1", name: "Ada" };
const role: UserRole = "admin";

console.log(user.name); // Expected: Ada
console.log(role);      // Expected: admin
```

**Expected Output:**
```
Ada
admin
```

**Why this output occurs.** The object shape uses an interface; the union and intersection use type aliases. This follows the Handbook and Google Style Guide recommendation: interfaces for objects, types for everything else.

---

#### Example 2: Consistent convention when both work

```typescript
// example_best_consistency.ts
// ✅ Consistent choice: interface for objects throughout.
interface Product {
  id: string;
  price: number;
}

interface CartItem {
  product: Product;
  quantity: number;
}

// ✅ Type for unions.
type CartStatus = "empty" | "active" | "checked-out";

const item: CartItem = { product: { id: "p1", price: 10 }, quantity: 2 };
console.log(item.product.price); // Expected: 10
```

**Expected Output:**
```
10
```

**Why this output occurs.** The object shapes use interfaces consistently; the union uses a type alias. Consistency makes the codebase predictable and aligns with the dominant convention.

---

### Real-World Cases

- **Enterprise codebases**: interfaces for domain objects and public APIs; type aliases for unions and utility types.
- **Library authoring**: exported interfaces for object contracts; internal type aliases for unions and utility objects.
- **Code review checklists**: enforcing “interface for objects, type for everything else” as a team convention.

---

## References

- TypeScript Playground — Types vs Interfaces (Official Comparison) - https://www.typescriptlang.org/play/typescript/language-extensions/types-vs-interfaces.ts.html
- TypeScript Handbook — Everyday Types: Differences Between Type Aliases and Interfaces - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces
- TypeScript Handbook — Object Types - https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Performance Wiki — Preferring Interfaces Over Intersections - https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections
- Convex — Interface vs. Type (Performance, Implicit Index Signatures, Declaration Merging) - https://www.convex.dev/typescript/core-concepts/object-oriented-programming/typescript-interface-vs-type
- Google TypeScript Style Guide — Prefer Interfaces Over Type Literal Aliases - https://google.github.io/styleguide/tsguide.html
- Total TypeScript — Type vs Interface: Which Should You Use? - https://www.totaltypescript.com/type-vs-interface-which-should-you-use
- opencode TypeScript Best Practices — Interface vs Type Decision Matrix - https://github.com/Thomashighbaugh/opencode/blob/main/skills/typescript-interface-vs-type/SKILL.md
- TypeScript ESLint — `no-redeclare` Rule - https://typescript-eslint.io/rules/no-redeclare/
- W3Schools — TypeScript Type Aliases and Interfaces - https://www.w3schools.com/typescript/typescript_aliases_and_interfaces.php
- Stack Overflow — TypeScript Interfaces vs Types (Exhaustive Comparison) - https://stackoverflow.com/questions/37233735/typescript-interfaces-vs-types/52682220#52682220
- Microsoft/TypeScript Issue #15300 — Implicit Index Signatures for Type Aliases - https://github.com/microsoft/TypeScript/issues/15300