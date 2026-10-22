# Extending Types in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Type extension** in TypeScript is the family of mechanisms by which a new type is derived from one or more existing types, incorporating their members and optionally adding or overriding members. The two primary mechanisms are **interface inheritance** via the `extends` keyword and **intersection types** via the `&` operator. A third, related mechanism—**declaration merging**—allows multiple declarations of the same interface name to combine into a single type. Together, these mechanisms enable code reuse, composition, and the construction of type hierarchies without duplication.

**Technical Definition.** Interface extension (`interface Derived extends Base1, Base2 { ... }`) creates a **new named object type** whose instance type is a **subtype** of each base type. The compiler verifies that the derived interface’s members satisfy the base contracts, caches the resulting type by name, and detects property conflicts at declaration. Intersection types (`type Combined = A & B`) create an **anonymous type** whose members are the **recursive merge** of the constituent types; conflicts are not detected at declaration and may silently produce `never`. Type aliases cannot use `extends`; they compose via intersection. Declaration merging applies **only to interfaces** (and namespaces), not to type aliases, and combines declarations with the same name in the same scope.

**Beginner-Friendly Explanation.** Extending a type means **building a new type from an existing one**. You can either use `interface Derived extends Base` to say “this new interface is a kind of the base interface,” or use `type Combined = A & B` to say “this type has everything from both A and B.” Interfaces are **open**—you can add to them later by declaring them again—while type aliases are **closed**. For building hierarchies and public APIs, interfaces are generally preferred; for combining types flexibly, intersection types are the tool.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Interface `extends`** | Creates a new named type that inherits members from one or more base interfaces or class types. |
| **Intersection `&`** | Combines multiple types into a single type containing all members from every constituent. |
| **Type aliases cannot `extends`** | Type aliases compose via `&`, not via the `extends` keyword. |
| **Declaration merging** | Interfaces with the same name in the same scope merge; type aliases cannot. |
| **Conflict handling** | Interface `extends` errors on incompatible members; intersections may silently produce `never`. |
| **Compiler caching** | Interface hierarchies are cached by name; intersections are recomputed on each use. |
| **Multiple inheritance (interfaces)** | An interface may extend multiple interfaces simultaneously. |
| **Erasure** | All extension constructs are erased during compilation. |

---

### Prerequisites

- **TypeScript basics**: interfaces, type aliases, and object types.
- **Generics fundamentals**: type parameters and constraints.
- **Union and intersection types** (`|`, `&`).
- **The distinction between the type space and the value space**.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support) to observe compiler behavior.

---

### Related Programming Areas

- **Object-oriented design**: inheritance, polymorphism, and the Liskov Substitution Principle.
- **Declaration merging** for module augmentation and global type extension.
- **Compiler performance** in large codebases.
- **Utility type composition**: `Omit<T, K>`, `Pick<T, K>`, and intersection types.
- **Design patterns**: mixins, composition, and interface segregation.

---

## Core Concepts / Features

---

## 1. Extending Interfaces

### Definitions

**Core Definition.** **Extending an interface** is the act of using the `extends` keyword to create a new interface that inherits the members of one or more base interfaces (or class types), optionally adding or overriding members. The derived interface is a **subtype** of each base and must satisfy all inherited contracts.

**Technical Definition.** An interface extension `interface Derived extends Base1, Base2 { ... }` creates a new named object type. The compiler verifies that `Derived` is assignable to each base, inherits all members not redeclared, and detects property conflicts at declaration (unlike intersections, which defer conflict resolution to use sites). Interface hierarchies are **cached by the compiler**, making `extends` more performant than recursive intersection flattening for deep hierarchies. An interface may extend multiple interfaces simultaneously.

**Beginner-Friendly Explanation.** Extending an interface means **creating a new interface that includes everything from an existing one, plus more**. `interface Dog extends Animal` means “a `Dog` is a kind of `Animal` and has all of `Animal`’s members.” You can extend multiple interfaces at once to compose contracts.

### Purposes

- **To reuse interface contracts** without duplicating member declarations.
- **To build clear type hierarchies** that mirror domain relationships.
- **To compose multiple contracts** into a single named interface.
- **To benefit from compiler caching** and declaration-time conflict detection.
- **To enable class implementation** of composed contracts via `implements`.

### Syntax Rules and Structure

**General syntax:**

```
interface Derived extends Base1, Base2 {
  // additional or overridden members
}
```

**Component breakdown:**

- `interface` — declaration keyword.
- `Derived` — the derived interface name.
- `extends` — heritage keyword.
- `Base1, Base2` — comma-separated base interfaces or class types.
- `{ ... }` — additional members; inherited members may be redeclared with compatible types.

**Syntax rules.**

- An interface may extend **zero or more** base interfaces or class types.
- Inherited members are available on the derived interface.
- Members with the same name and **identical** types in multiple bases are merged without error.
- Members with the same name but **different** types must be redeclared with a subtype-compatible type, or the compiler reports an error.
- An interface may not extend itself directly or indirectly.
- Interfaces may extend **class types** with public members, but not classes with `private` or `protected` members.

**Constraints and limitations.**

- Interfaces cannot extend **type aliases of unions or primitives**; they can extend object-type aliases (TypeScript 2.2+).
- Inherited members from multiple bases must be compatible.
- Declaration merging may introduce unexpected members; use `type` if merging is undesirable.
- Interface `extends` is restricted to object shapes; it cannot express unions or mapped types.

### Annotated Complete Step by Step Code Examples

#### Example 1: Single interface extension

```typescript
// example_extend_single.ts
// Step 1: Define a base interface.
interface Animal {
  name: string;
}

// Step 2: Extend it.
interface Dog extends Animal {
  breed: string;
}

// Step 3: Use the derived interface.
const dog: Dog = { name: "Rex", breed: "Labrador" };
console.log(dog.name);  // Expected: Rex
console.log(dog.breed); // Expected: Labrador
```

**Setup.** Compile with `npx tsc example_extend_single.ts --strict`.

**Expected Output:**
```
Rex
Labrador
```

**Why this output occurs.** `Dog` extends `Animal`, inheriting `name` and adding `breed`. The object literal must satisfy both properties. The compiler caches the `Dog` type by name, making repeated checks efficient.

---

#### Example 2: Multiple interface extension (composition)

```typescript
// example_extend_multiple.ts
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

// Extend both interfaces simultaneously.
interface ColorfulCircle extends Colorful, Circle {}

const cc: ColorfulCircle = { color: "red", radius: 42 };
console.log(cc.color);  // Expected: red
console.log(cc.radius); // Expected: 42
```

**Expected Output:**
```
red
42
```

**Why this output occurs.** `ColorfulCircle` extends both `Colorful` and `Circle`, composing their contracts. The object literal must satisfy all required members from both bases. This is the interface equivalent of multiple inheritance for contracts.

---

#### Example 3: Conflict detection in interface extension

```typescript
// example_extend_conflict.ts
interface A {
  value: string;
}

interface B {
  value: number;
}

// ❌ The compiler detects the conflict.
// interface C extends A, B {}
// Error: Interface 'C' cannot simultaneously extend types 'A' and 'B'.
//        Named property 'value' of types 'A' and 'B' are not identical.

console.log("Interface extends detects conflicts at declaration");
```

**Expected Output:**
```
Interface extends detects conflicts at declaration
```

**Why this output occurs.** Interface `extends` checks for incompatible members **at declaration time** and reports a clear error. This is a key advantage over intersection types, which can silently produce `never` on conflicts.

---

### Real-World Cases

- **Domain hierarchies**: `interface Admin extends User { permissions: string[]; }`.
- **Composed contracts**: `interface BlogPost extends Identifiable, Timestamped { title: string; }`.
- **Framework props**: extending base component props with additional fields.
- **Library APIs**: exported interfaces that consumers can extend safely.

---

## 2. Extending Type Aliases

### Definitions

**Core Definition.** **Extending a type alias** is achieved by creating a **new intersection type** (`type Derived = Base & { extra: Type }`), because the `extends` keyword is not available for type aliases. The resulting alias contains all members of the base and the additional members.

**Technical Definition.** Type aliases cannot use `extends`; they compose via the intersection operator `&`. The resulting type is a **recursive merge** of the constituent types. Conflicts are not detected at declaration and may produce `never` when used. Since TypeScript 2.7, type aliases can be extended by creating a new intersection type, e.g., `type Cat = Animal & { purrs: true }`. Type aliases are **closed**: they cannot be augmented via declaration merging.

**Beginner-Friendly Explanation.** Since you cannot write `type B extends A`, you write `type B = A & { extra: string }`. This says “B has everything from A, plus an extra field.” It is the type-alias way of extending. Unlike interface extension, conflicts are not caught early—so be careful.

### Purposes

- **To extend type aliases** that cannot use `extends`.
- **To compose multiple types** flexibly, including unions, tuples, and primitives.
- **To create one-off extended types** without introducing a new interface name.
- **To leverage utility types** such as `Omit<T, K>` alongside intersections for fine-grained composition.

### Syntax Rules and Structure

**General syntax:**

```
type Derived = Base & {
  extraProperty: Type;
};
```

**Component breakdown:**

- `type` — alias keyword.
- `Derived` — the new alias name.
- `=` — assignment.
- `Base` — the existing type being extended.
- `&` — intersection operator.
- `{ extraProperty: Type }` — the additional members.

**Syntax rules.**

- Type aliases extend via `&`, not `extends`.
- The resulting alias includes all members from every constituent.
- Type aliases remain **closed**; they cannot be declaration-merged.
- Intersections may combine interfaces, type aliases, object literals, primitives, and function types.

**Constraints and limitations.**

- Intersections do **not** detect property conflicts at declaration; incompatible properties may silently become `never`.
- Intersections are **recomputed on every use**, which can affect compiler performance in deep hierarchies.
- Type aliases cannot be augmented via declaration merging.
- Error messages may show the expanded intersection rather than the alias name.

### Annotated Complete Step by Step Code Examples

#### Example 1: Extending a type alias with intersection

```typescript
// example_type_extend.ts
// Step 1: Define a base type alias.
type Animal = {
  name: string;
};

// Step 2: Extend via intersection.
type Dog = Animal & {
  breed: string;
};

const dog: Dog = { name: "Rex", breed: "Labrador" };
console.log(dog.name);  // Expected: Rex
console.log(dog.breed); // Expected: Labrador
```

**Expected Output:**
```
Rex
Labrador
```

**Why this output occurs.** `Dog` is an intersection of `Animal` and `{ breed: string }`. The compiler recursively merges the properties. The resulting type has both `name` and `breed`.

---

#### Example 2: Extending with inline properties

```typescript
// example_type_extend_inline.ts
type Person = { name: string; age: number };

// Extend inline — no separate type declaration needed.
type PremiumCustomer = Person & {
  id: string;
  tier: "gold" | "platinum";
};

const customer: PremiumCustomer = {
  name: "Ada",
  age: 36,
  id: "c1",
  tier: "gold",
};

console.log(customer.tier); // Expected: gold
```

**Expected Output:**
```
gold
```

**Why this output occurs.** `PremiumCustomer` intersects `Person` with the inline object type, adding `id` and `tier`. The compiler merges the properties recursively. This pattern is useful for one-off extended types.

---

#### Example 3: Intersection conflict producing `never`

```typescript
// example_type_conflict.ts
interface Person1 {
  name: string;
}

interface Person2 {
  name: number;
}

// ✅ Compiles, but name becomes never.
type Staff = Person1 & Person2;

declare const staffer: Staff;
// staffer.name; // Type is never — cannot be used.

console.log("Intersection conflict silently produces never");
```

**Expected Output:**
```
Intersection conflict silently produces never
```

**Why this output occurs.** `Person1.name` is `string` and `Person2.name` is `number`. The intersection requires `name` to be **both** types simultaneously, which reduces to `never`. The compiler does **not** report an error at declaration, unlike interface `extends`. This is a critical difference and a reason to prefer `extends` in hierarchies.

---

### Real-World Cases

- **Extending non-interface types**: `type StringMap = Record<string, string> & { metadata: object };`.
- **Utility type composition**: `type ReadonlyUser = Readonly<User> & { version: number };`.
- **One-off domain variants**: `type AdminUser = User & { role: "admin" };`.
- **Combining unrelated types**: `type Customer = Person & Address;`.

---

## 3. Intersection Types

### Definitions

**Core Definition.** An **intersection type** is a type formed by combining two or more types with the `&` operator, producing a new type that has **all members of every constituent type**. A value of the intersection type must satisfy every constituent simultaneously.

**Technical Definition.** The intersection `A & B` is the **greatest lower bound** of `A` and `B`: it includes every member from both types. For object types, members are merged recursively. For conflicting properties, the resulting property type is `never` (since the value must be both types at once). Intersection types are the **only way** to extend type aliases and are also used to combine function types, primitives, and object types. They are **erased** at runtime.

**Beginner-Friendly Explanation.** An intersection type says **“this value must be everything from A AND everything from B.”** It is the type-level equivalent of “and.” If `A` has `name` and `B` has `age`, then `A & B` requires both `name` and `age`. If both define `name` differently, the value can never satisfy both—so the property becomes impossible to use.

### Purposes

- **To combine multiple types** into a single type containing all members.
- **To extend type aliases** (which cannot use `extends`).
- **To compose object types, function types, and primitives** flexibly.
- **To model values that satisfy multiple contracts simultaneously**.
- **To work with utility types** like `Omit<T, K>` and `Pick<T, K>` for granular composition.

### Syntax Rules and Structure

**General syntax:**

```
type Combined = TypeA & TypeB & TypeC;
```

**Component breakdown:**

- `type` — alias keyword (or inline in a type position).
- `Combined` — the resulting type name.
- `&` — intersection operator.
- `TypeA, TypeB, TypeC` — constituent types.

**Syntax rules.**

- Intersections may combine any types: object types, interfaces, type aliases, primitives, function types, and tuples.
- The resulting type has all members from every constituent.
- Identical members are merged.
- Conflicting members become `never`.
- Intersections may be used inline in function parameters and return types.

**Constraints and limitations.**

- Conflicts are **not** detected at declaration; they produce `never` silently.
- Intersections are **recomputed on every use**, affecting compiler performance in deep hierarchies.
- Error messages may show expanded intersections rather than alias names.
- Intersections cannot express unions or primitives in a single constituent (use type aliases for unions).

### Annotated Complete Step by Step Code Examples

#### Example 1: Combining two interfaces

```typescript
// example_intersection_basic.ts
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

// Combine via intersection.
type ColorfulCircle = Colorful & Circle;

const cc: ColorfulCircle = { color: "red", radius: 42 };
console.log(cc.color);  // Expected: red
console.log(cc.radius); // Expected: 42
```

**Expected Output:**
```
red
42
```

**Why this output occurs.** `ColorfulCircle` is the intersection of `Colorful` and `Circle`. It requires both `color` and `radius`. The compiler recursively merges the members.

---

#### Example 2: Intersection in function parameters

```typescript
// example_intersection_function.ts
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

function draw(circle: Colorful & Circle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}

draw({ color: "blue", radius: 42 });
// Expected:
// Color was blue
// Radius was 42
```

**Expected Output:**
```
Color was blue
Radius was 42
```

**Why this output occurs.** The function parameter is an intersection type. The argument must have both `color` and `radius`. The compiler checks that the object literal satisfies both constituents.

---

#### Example 3: Intersection with utility types

```typescript
// example_intersection_utility.ts
interface User {
  id: string;
  name: string;
  email: string;
}

// Combine Omit with an inline extension.
type PublicUser = Omit<User, "email"> & {
  displayName: string;
};

const publicUser: PublicUser = {
  id: "u1",
  name: "Ada",
  displayName: "Ada Lovelace",
};

console.log(publicUser.displayName); // Expected: Ada Lovelace
```

**Expected Output:**
```
Ada Lovelace
```

**Why this output occurs.** `Omit<User, "email">` removes `email`, and the intersection adds `displayName`. This pattern is common for composing precise types from utility types and inline extensions.

---

### Real-World Cases

- **Composing React props**: `type ButtonProps = BaseProps & { variant: "primary" | "secondary" };`.
- **Extending non-interface types**: combining `Record<K, V>` with additional metadata.
- **Utility type composition**: `type ReadonlyUser = Readonly<User> & { version: number };`.
- **Function type intersection**: combining call signatures with properties.

---

## 4. Merging Declarations

### Definitions

**Core Definition.** **Declaration merging** is a TypeScript feature in which **multiple declarations with the same name in the same scope are combined into a single definition** containing the members of all declarations. It applies to **interfaces** (and namespaces), but **not** to type aliases.

**Technical Definition.** When two interface declarations with the same name appear in the same scope, the compiler **mechanically joins** their members into a single interface. Non-function members must have **identical** types across declarations; function members of the same name are treated as **overloads**. Later declarations have **higher precedence** in overload resolution, with string-literal-parameter signatures bubbled to the top. Merging applies **per scope**: interfaces in different modules do not merge unless explicitly augmented via `declare module`.

**Beginner-Friendly Explanation.** Declaration merging means **declaring the same interface twice adds to it**. If you declare `interface Box { width: number }` and later `interface Box { height: number }`, TypeScript combines them into one `Box` with both `width` and `height`. This is how libraries let you add properties to their types without modifying the source. Type aliases cannot do this—declaring the same alias twice is an error.

### Purposes

- **To augment existing interfaces** (library types, global types) without editing the original source.
- **To split large interface definitions** across multiple files or modules.
- **To extend global types** such as `Window` or `ProcessEnv`.
- **To enable plugin architectures** where each plugin contributes members to a shared interface.

### Syntax Rules and Structure

**General syntax (interface merging):**

```
interface Box {
  width: number;
}

interface Box {
  height: number;
}

// Box now has width AND height.
```

**Component breakdown:**

- Both declarations use the same interface name in the same scope.
- Members are combined mechanically.
- Non-function members must have identical types.
- Function members become overloads.

**Syntax rules.**

- Merging applies to interfaces and namespaces; **not** to type aliases.
- Non-function members must be **unique** or have **identical** types across declarations.
- Function members with the same name are treated as **overloads**; later declarations have higher precedence.
- Merging is **per scope**; interfaces in different modules do not merge unless augmented via `declare module`.

**Constraints and limitations.**

- Type aliases **cannot** be merged; redeclaration is a compile-time error.
- Conflicting non-function member types produce a compile error.
- Declaration merging can be surprising; use `type` when merging is undesirable.
- Classes do not merge with interfaces in the same way; a class and an interface with the same name merge only in specific declaration patterns.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic interface merging

```typescript
// example_merge_basic.ts
interface Box {
  width: number;
}

// Second declaration merges with the first.
interface Box {
  height: number;
}

const box: Box = { width: 10, height: 20 };
console.log(box.width + box.height); // Expected: 30
```

**Expected Output:**
```
30
```

**Why this output occurs.** The two `Box` declarations merge into a single interface with both `width` and `height`. The object literal must satisfy both properties.

---

#### Example 2: Conflicting member types in merged interfaces

```typescript
// example_merge_conflict.ts
interface Person {
  name: string;
}

// ❌ Conflicting member type.
// interface Person {
//   name: number;
// }
// Error: Subsequent property declarations must have the same type.

console.log("Conflicting members are rejected");
```

**Expected Output:**
```
Conflicting members are rejected
```

**Why this output occurs.** Non-function members must have **identical** types across merged declarations. A `string` and `number` conflict produces a compile-time error.

---

#### Example 3: Merging as overloads (function members)

```typescript
// example_merge_overload.ts
interface Cloner {
  clone(animal: string): string;
}

interface Cloner {
  clone(animal: number): number;
}

// The merged interface has two overloads.
declare const cloner: Cloner;

console.log(typeof cloner.clone); // Expected: function
```

**Expected Output:**
```
function
```

**Why this output occurs.** Function members with the same name are treated as **overloads** rather than conflicts. The merged `Cloner` interface has two `clone` signatures—one for strings and one for numbers.

---

#### Example 4: Module augmentation via declaration merging

```typescript
// example_merge_module.ts
// Simulate a library interface.
interface LibraryConfig {
  apiUrl: string;
}

// Augment it in the same scope.
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

**Why this output occurs.** The second `LibraryConfig` declaration merges with the first, adding `timeout`. This is the standard pattern for augmenting third-party library types without editing the library.

---

### Real-World Cases

- **Global augmentation**: `declare global { interface Window { analytics: Analytics; } }`.
- **Library augmentation**: adding properties to `Express.Request` or `NodeJS.ProcessEnv`.
- **Plugin systems**: each plugin contributes members to a shared interface via merging.
- **Splitting large interfaces** across multiple files in the same module.

---

## 5. Extending Best Practices

### Definitions

**Core Definition.** **Extending best practices** are guidelines for choosing the correct extension mechanism—interface `extends`, intersection `&`, or declaration merging—in ways that maximize **type safety, compiler performance, error clarity, and maintainability**, while avoiding the pitfalls of silent `never` conflicts, unbounded declaration merging, and intersection performance costs.

**Technical Definition.** Best practices center on the principle articulated by the TypeScript performance wiki and community experts: **prefer interface `extends` over intersection types for object inheritance**. Interfaces create a **single flat object type**, detect property conflicts at declaration, and are **cached by the compiler**. Intersections recursively merge properties on every use, may silently produce `never`, and impose a performance cost. Type aliases are the **safer default** for non-object types and one-off utility compositions. Declaration merging should be used **deliberately** for augmentation, not accidentally.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep type extension safe and fast. The big rules are: **use `interface extends` for object hierarchies** (it catches errors early and is faster); **use intersections for one-off combinations and non-object types**; **use declaration merging only when you intentionally want to augment a type**; and **avoid building deep intersection hierarchies** because they slow down the compiler and can hide conflicts.

### Purposes

- **To maximize compiler performance** by preferring cached interface hierarchies over recursive intersections.
- **To catch property conflicts early** at declaration time.
- **To keep error messages clear** and actionable.
- **To avoid accidental declaration merging** that silently modifies types.
- **To choose the right tool** for each extension scenario.

### Syntax Rules and Structure

Best practices are conventions that interact with extension syntax. The core rules are:

1. **Prefer interface `extends` for object inheritance.** Interfaces create a flat type, detect conflicts at declaration, and are cached by the compiler.
2. **Use intersections for one-off combinations and non-object types.** Intersections are appropriate when combining function types, primitives, or inline object extensions where an interface would be overkill.
3. **Use declaration merging deliberately.** Merging is powerful for augmentation but can be surprising; document interfaces intended for merging.
4. **Avoid deep intersection hierarchies.** Intersections are recomputed on every use, causing performance degradation in large codebases.
5. **Prefer `extends` over `&` when the result is an object type used as a contract.** This ensures conflict detection and better tooling.
6. **Use `type` as the safer default for everything else.** Type aliases avoid declaration-merging pitfalls and handle unions, primitives, and advanced types.
7. **Be aware of `never` from intersection conflicts.** If an intersection produces `never`, the compiler will not warn you at declaration; use `extends` for critical hierarchies.

**Component breakdown (extension mechanism decision tree):**

```
Are you extending an object type used as a contract or hierarchy?
  ├── Yes → Use interface extends.
  │        ├── Does it need declaration merging? → interface (mergeable).
  │        └── Is it a deep hierarchy? → interface extends (cached).
  └── No → Are you combining types for a one-off utility?
           ├── Yes → Use intersection (&).
           │        ├── Combining function types or primitives? → &.
           │        └── Inline object extension? → &.
           └── No → Use type alias as the safer default.
```

**Constraints and limitations.**

- Interface `extends` cannot combine unions or primitives.
- Intersections silently produce `never` on conflicts; interface `extends` errors early.
- Declaration merging applies only to interfaces and namespaces, not type aliases.
- Intersections have a performance cost in large hierarchies; interfaces are cached.

### Annotated Complete Step by Step Code Examples

#### Example 1: Prefer interface `extends` over intersection for object hierarchy

```typescript
// example_best_extends.ts
// ❌ Intersection hierarchy: slower, no conflict detection.
// type User = Base & { name: string };
// type Admin = User & { role: string };

// ✅ Interface extends: cached, conflict-detecting.
interface Base {
  id: string;
}

interface User extends Base {
  name: string;
}

interface Admin extends User {
  role: string;
}

const admin: Admin = { id: "a1", name: "Ada", role: "admin" };
console.log(admin.role); // Expected: admin
```

**Expected Output:**
```
admin
```

**Why this output occurs.** The interface hierarchy uses `extends`, which creates a flat cached type and detects conflicts at declaration. The intersection version would recursively flatten on every use and could silently produce `never` if `Base` and `User` had conflicting members.

---

#### Example 2: Use intersection for one-off utility composition

```typescript
// example_best_intersection_utility.ts
interface User {
  id: string;
  name: string;
  email: string;
}

// ✅ Intersection is appropriate for one-off utility composition.
type PublicUser = Omit<User, "email"> & {
  displayName: string;
};

const publicUser: PublicUser = {
  id: "u1",
  name: "Ada",
  displayName: "Ada Lovelace",
};

console.log(publicUser.displayName); // Expected: Ada Lovelace
```

**Expected Output:**
```
Ada Lovelace
```

**Why this output occurs.** The intersection combines `Omit<User, "email">` with an inline extension. This is a **one-off utility composition**, not a deep hierarchy, so the intersection is appropriate. Converting this to an interface would require a named base type and would be more ceremony than necessary.

---

#### Example 3: Deliberate declaration merging for augmentation

```typescript
// example_best_merging.ts
// Base interface from a library.
interface LibraryConfig {
  apiUrl: string;
}

// ✅ Deliberate augmentation via declaration merging.
interface LibraryConfig {
  timeout?: number;
  retries?: number;
}

const config: LibraryConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
};

console.log(config.retries); // Expected: 3
```

**Expected Output:**
```
3
```

**Why this output occurs.** The second `LibraryConfig` declaration merges with the first, adding `timeout` and `retries`. This is a **deliberate augmentation** of a library type—the intended use of declaration merging. The interface is documented as mergeable, and the added members are optional to avoid breaking existing code.

---

### Real-World Cases

- **Large codebases**: interface `extends` for domain hierarchies to benefit from caching and conflict detection.
- **Utility type composition**: intersections with `Omit<T, K>` and `Pick<T, K>` for one-off precise types.
- **Library augmentation**: deliberate declaration merging to add application-specific fields to third-party types.
- **Code review checklists**: enforcing “interface `extends` for object hierarchies; intersections for one-off utilities” as a team convention.

---

## References

- TypeScript Playground — Types vs Interfaces (Official Comparison) - https://www.typescriptlang.org/play/typescript/language-extensions/types-vs-interfaces.ts.html
- TypeScript Handbook — Object Types (Extending Types, Intersection Types) - https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Handbook — Declaration Merging - https://www.typescriptlang.org/docs/handbook/declaration-merging.html
- TypeScript Performance Wiki — Preferring Interfaces Over Intersections - https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections
- Convex — TypeScript Extend: Interfaces, Intersections, and Best Practices - https://www.convex.dev/typescript/resources/typescript-extend-type
- Convex — Understanding Interfaces in TypeScript (Extending Interfaces, Multiple Interfaces) - https://www.convex.dev/typescript/core-concepts/object-oriented-programming/typescript-interface
- Microsoft Learn — Extend an Interface in TypeScript - https://learn.microsoft.com/en-us/training/modules/typescript-implement-interfaces/4-extend-interface
- Stack Overflow — The `extends` keyword can be used for interfaces and classes only - https://stackoverflow.com/questions/41385015/typescript-extending-types-with-intersection
- TypeScript 2.2 Release Notes — Extending Type Aliases (Intersection Types) - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-2.html
- TypeScript Handbook (Legacy) — Interfaces vs. Type Aliases (Intersection Extension) - https://raw.githubusercontent.com/microsoft/TypeScript-Handbook/357cdb6b7ec6838a1e35f4bba78d5a0faf6ab91e/pages/Advanced%20Types.md
- ESLint Plugin — `prefer-interface-extends-over-type-intersection` - https://www.npmjs.com/package/eslint-plugin-ts-type-preferences
- TypeScript FAQ — Interface vs. Type (Extends vs Intersection) - https://github.com/Zamiell/typescript-faq/blob/main/questions/interface-vs-type.md
- Cory House — Speed Up TypeScript Builds: Replace Intersections with Interfaces - https://coryhouse.dev/speed-up-typescript-builds
- Stack Overflow — Type safety difference between type and interface when extending - https://stackoverflow.com/questions/71221037/type-safety-difference-between-type-and-interface-when-extending