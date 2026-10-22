# Intersection Types in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** An **intersection type** is a type formed by combining two or more types with the ampersand operator (`&`), representing a value that must satisfy **all constituent types simultaneously**. It is the type-level equivalent of a logical **AND**: a value of type `A & B` must be assignable to both `A` and `B`, possessing every member required by each constituent.

**Technical Definition.** In TypeScript's type grammar, an intersection type `T₁ & T₂ & … & Tₙ` denotes the **set-theoretic intersection** of its constituent types. A value is assignable to the intersection only if it is assignable to **every** member. For object types, properties are **recursively merged**; if the same property appears in multiple constituents with **incompatible types**, the resulting property type becomes `never`, because no value can satisfy both. Unlike interface `extends`, intersections do **not** detect property conflicts at declaration—conflicts surface only when the type is used. Since TypeScript 4.8, intersections with `{}` are simplified to the other object type. Intersection types are **erased** during compilation and have no runtime representation.

**Beginner-Friendly Explanation.** An intersection type says: **"This value must be everything from A AND everything from B."** If a union is an OR, an intersection is an AND. `A & B` requires the value to have all of A's members and all of B's members. It is TypeScript's primary mechanism for **type composition**—building a new type from existing pieces without duplication.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **AND semantics** | A value of `A & B` must be assignable to both `A` and `B`. |
| **Recursive merging** | Object properties are merged recursively on every use, not flattened once. |
| **Silent conflict to `never`** | Incompatible property types reduce to `never` without a declaration-time error. |
| **Composition tool** | Intersections are the primary way to compose types, especially when interfaces cannot be extended (unions, primitives, functions). |
| **Performance cost** | Intersections are **recomputed on every use**, causing measurable slowdowns in large codebases. |
| **Interface caching contrast** | Interfaces are cached by name; intersections are not. |
| **Mixin foundation** | Intersections model the ECMAScript mixin pattern by combining constructor types. |
| **Erasure** | Intersection types exist only at compile time and emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: type annotations, type aliases, and interfaces.
- **Union types** (`|`) — the dual of intersections.
- **Object types** and structural compatibility.
- **Generics fundamentals**: type parameters and constraints.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Type composition** and **mixin patterns** in TypeScript.
- **Interface inheritance** (`extends`) — the preferred alternative for object hierarchies.
- **Union types** (`|`) — the OR counterpart to intersection's AND.
- **Utility types** — `Omit<T, K>`, `Pick<T, K>`, and intersection composition.
- **Compiler performance** — interface caching vs. intersection recomputation.

---

## Core Concepts / Features

---

## 1. Intersection Types

### Definitions

**Core Definition.** An **intersection type** combines two or more types into a new type that has **every member of every constituent**. A value of the intersection must satisfy all constituents simultaneously.

**Technical Definition.** An intersection `A & B` is the **greatest lower bound** of `A` and `B`: it includes every member from both. For object types, members are merged recursively. Conflicting properties produce `never`. Intersections may combine interfaces, type aliases, object literals, primitives, function types, and tuples. They are the **only way** to extend type aliases that are not object types.

**Beginner-Friendly Explanation.** An intersection is **the "AND" of types**. `A & B` means "has all of A's stuff AND all of B's stuff." It is how you glue types together to make a richer type.

### Purposes

- **To compose multiple types** into one that satisfies all of them.
- **To extend type aliases** that cannot use `extends`.
- **To combine function types or primitives** that interfaces cannot extend.
- **To model values that participate in multiple contracts simultaneously**.
- **To implement mixins** by intersecting constructor and instance types.

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

- `&` has **higher precedence** than `|`: `A & B | C` is `(A & B) | C`.
- Intersections may combine any types: object types, interfaces, primitives, function types, tuples.
- Duplicate members are merged; conflicting members become `never`.
- Intersections may be used inline in parameters and return types.

**Constraints and limitations.**

- **Conflicts are silent**: incompatible properties produce `never` without a declaration-time error, unlike interface `extends`.
- **Performance cost**: intersections are **recomputed on every use**, unlike cached interfaces.
- Intersections cannot express unions or conditional types directly; use type aliases.

### Annotated Complete Step by Step Code Examples

#### Example 1: Composing two interfaces

```typescript
// example_intersection_basic.ts
interface ErrorHandling {
  success: boolean;
  error?: { message: string };
}

interface ArtworksData {
  artworks: { title: string }[];
}

// Compose via intersection.
type ArtworksResponse = ArtworksData & ErrorHandling;

const response: ArtworksResponse = {
  success: true,
  artworks: [{ title: "Mona Lisa" }],
};

console.log(response.artworks[0].title); // Expected: Mona Lisa
console.log(response.success);           // Expected: true
```

**Setup.** Compile with `npx tsc example_intersection_basic.ts --strict`.

**Expected Output:**
```
Mona Lisa
true
```

**Why this output occurs.** `ArtworksResponse` requires all members of both `ArtworksData` and `ErrorHandling`. The object literal satisfies both contracts. This is the canonical composition pattern from the TypeScript Playground.

---

#### Example 2: Intersection with a union (precise composition)

```typescript
// example_intersection_union.ts
interface CreateArtistBioBase {
  artistID: string;
  thirdParty?: boolean;
}

// Intersection with a union: must have base AND (html OR markdown).
type CreateArtistBioRequest = CreateArtistBioBase & ({ html: string } | { markdown: string });

const request: CreateArtistBioRequest = {
  artistID: "banksy",
  markdown: "Banksy is an anonymous graffiti artist...",
};

console.log(request.artistID); // Expected: banksy
```

**Expected Output:**
```
banksy
```

**Why this output occurs.** The intersection requires the base fields **and** exactly one of `html` or `markdown`. The object provides `artistID` and `markdown`, satisfying both the intersection and the union inside it.

---

### Real-World Cases

- **API response composition**: `type ArtworksResponse = ArtworksData & ErrorHandling`.
- **Request payloads**: combining base fields with one-of variants.
- **Function type composition**: combining call signatures with properties.

---

## 2. `&` Operator

### Definitions

**Core Definition.** The **`&` operator** is the syntactic token that combines two or more types into an intersection type. It is TypeScript's type-level analog of the logical AND.

**Technical Definition.** In TypeScript's grammar, `&` appears between type expressions in an intersection. It has **higher precedence** than `|` and is left-associative. Duplicate members are merged; incompatible members become `never`. The operator may combine any type expressions, including primitives, object types, function types, and tuples.

**Beginner-Friendly Explanation.** The `&` symbol is **the "AND" of the type world**. You put it between types to say "this value must be the first type AND the second type AND the third type."

### Purposes

- **To combine types into an intersection** concisely.
- **To compose object types** without creating a new interface.
- **To combine function signatures** and primitives that interfaces cannot extend.
- **To build mixin types** by intersecting constructor and instance types.

### Syntax Rules and Structure

**General syntax:**

```
type Combined = A & B & C;
```

**Component breakdown:**

- `A`, `B`, `C` — constituent types.
- `&` — the intersection operator.

**Syntax rules.**

- `&` has higher precedence than `|`.
- `&` is left-associative: `A & B & C` is `(A & B) & C`.
- Duplicate members are merged.
- Conflicting members produce `never`.
- `&` may combine primitives, objects, functions, and tuples.

**Constraints and limitations.**

- `&` cannot be used to combine **values**; it is type-level only.
- `&` cannot extend interfaces; use `extends` for interface hierarchies.
- Conflicts are silent and may produce `never` without warning.

### Annotated Complete Step by Step Code Examples

#### Example 1: Precedence of `&` vs. `|`

```typescript
// example_amp_precedence.ts
type A = { a: string };
type B = { b: number };
type C = { c: boolean };

// & binds tighter than |: (A & B) | C
type Combined = A & B | C;

const x: Combined = { a: "hello", b: 42 }; // ✅ Matches A & B
const y: Combined = { c: true };            // ✅ Matches C

console.log(x.a, x.b); // Expected: hello 42
console.log(y.c);      // Expected: true
```

**Expected Output:**
```
hello 42
true
```

**Why this output occurs.** `&` has higher precedence than `|`, so `A & B | C` is parsed as `(A & B) | C`. The union accepts either the intersection or `C`.

---

#### Example 2: Intersection with primitives (interfaces cannot do this)

```typescript
// example_amp_primitives.ts
// Interfaces cannot extend primitives; intersections can.
type BrandedString = string & { __brand: "UserId" };

function createUserId(id: string): BrandedString {
  return id as BrandedString;
}

const userId = createUserId("u1");
console.log(typeof userId); // Expected: string
```

**Expected Output:**
```
string
```

**Why this output occurs.** The intersection combines `string` with a phantom brand property. At runtime, the value is a plain string; the brand exists only in the type system. This is a key use case where intersections are **mandatory**—interfaces cannot extend primitives.

---

### Real-World Cases

- **Branded types**: `type UserId = string & { __brand: "UserId" }`.
- **Function type composition**: `type Handler = ((e: Event) => void) & { priority: number }`.
- **One-off type combinations**: `type Config = BaseConfig & { override: string }`.

---

## 3. Intersection Type Declaration

### Definitions

**Core Definition.** An **intersection type declaration** is a type alias or inline annotation that defines an intersection by listing its constituent types separated by `&`.

**Technical Definition.** Intersection types may be declared as **named type aliases** (`type Combined = A & B`) or used **inline** in annotations, parameters, return types, and generic arguments. Interfaces **cannot** declare intersections directly. Intersections may be **generic** (`type Tagged<T> = T & { __tag: string }`) and may reference other intersections (which are not flattened but recursively merged).

**Beginner-Friendly Explanation.** An intersection type declaration is **the statement that defines an intersection**. It gives the intersection a name or uses it directly, listing the types to be combined with `&`.

### Purposes

- **To name a composed type** for reuse.
- **To define precise contracts** that require multiple constituent types.
- **To parameterize intersections** with generics.
- **To document the composition** of independent type pieces.

### Syntax Rules and Structure

**General syntax (named intersection):**

```
type Combined = TypeA & TypeB;
```

**General syntax (generic intersection):**

```
type Tagged<T> = T & { __tag: string };
```

**General syntax (inline intersection):**

```
function f(param: TypeA & TypeB): ReturnType { ... }
```

**Component breakdown:**

- `type Combined` — the named intersection alias.
- `= TypeA & TypeB` — the constituent types.

**Syntax rules.**

- Intersection aliases may be **generic** with constraints.
- Intersections may include **primitives**, **object types**, **interfaces**, and **other aliases**.
- Intersections may be **recursive** through type aliases.
- Intersections may combine with unions: `(A | B) & C`.

**Constraints and limitations.**

- **Interfaces cannot extend intersections**; use type aliases.
- Intersection aliases are **closed** (cannot be declaration-merged).
- **Recursive intersections** force the compiler to recompute the merged type on every reference.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generic intersection alias

```typescript
// example_intersection_generic.ts
type Timestamped<T> = T & { createdAt: Date; updatedAt: Date };

interface User {
  id: string;
  name: string;
}

type TimestampedUser = Timestamped<User>;

const user: TimestampedUser = {
  id: "u1",
  name: "Ada",
  createdAt: new Date(),
  updatedAt: new Date(),
};

console.log(user.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `Timestamped<T>` intersects any type `T` with timestamp fields. When instantiated with `User`, the result requires `id`, `name`, `createdAt`, and `updatedAt`. This is a reusable generic composition pattern.

---

#### Example 2: Intersection of interfaces

```typescript
// example_intersection_interfaces.ts
interface User {
  id: string;
  name: string;
}

interface Permissions {
  canRead: boolean;
  canWrite: boolean;
}

type UserWithPermissions = User & Permissions;

const admin: UserWithPermissions = {
  id: "u1",
  name: "Ada",
  canRead: true,
  canWrite: true,
};

console.log(admin.canWrite); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** The intersection requires all members from both `User` and `Permissions`. The object literal satisfies the combined contract.

---

### Real-World Cases

- **Generic utility types**: `type Tagged<T> = T & { __tag: string }`.
- **API response composition**: combining domain data with error handling and pagination metadata.
- **React prop composition**: `type ButtonProps = BaseProps & { variant: "primary" | "secondary" }`.

---

## 4. Intersection Type Usage

### Definitions

**Core Definition.** **Intersection type usage** refers to applying intersection types in type annotations, generic constraints, parameters, return types, and other type positions to require values that satisfy multiple constituents simultaneously.

**Technical Definition.** Intersections are used wherever a value must meet multiple contracts. Common usage patterns include **composition of interfaces**, **generic constraints** (`<T extends A & B>`), **parameter typing** (`function f(x: A & B)`), **mixins** (intersecting constructor types), and **branded types** (intersecting a base type with a phantom marker). Usage sites inherit the **conflict-to-`never`** and **recomputation** behavior of intersections.

**Beginner-Friendly Explanation.** Intersection type usage is simply **using `&` wherever you need a value to satisfy more than one type**. You write `A & B` in a parameter, a return type, or a type alias. It tells TypeScript: "whatever goes here must have everything from A and everything from B."

### Purposes

- **To require multiple contracts** in function parameters and return types.
- **To constrain generics** to types that satisfy several conditions.
- **To compose reusable type pieces** without creating new interfaces.
- **To implement mixins** by intersecting constructor types with base types.

### Syntax Rules and Structure

**Usage positions:**

- **Parameters**: `function f(x: A & B): void`
- **Return types**: `function f(): A & B`
- **Generic constraints**: `<T extends A & B>`
- **Type aliases**: `type C = A & B`
- **Variable annotations**: `const x: A & B = ...`

**Syntax rules.**

- The constituents must be types, not values.
- The resulting type requires **all** members from **all** constituents.
- Conflicting members become `never`.
- Intersections may be combined with unions: `(A | B) & C`.

**Constraints and limitations.**

- **Recomputation cost**: every use of an intersection forces recursive merging, slowing compilation in large codebases.
- **Silent `never`**: conflicts are not caught at declaration.
- **Error message complexity**: intersections can produce hard-to-read error messages.

### Annotated Complete Step by Step Code Examples

#### Example 1: Intersection in a function parameter

```typescript
// example_intersection_param.ts
interface Serializable {
  serialize(): string;
}

interface Loggable {
  log(): void;
}

function process(entity: Serializable & Loggable) {
  entity.log();
  console.log(entity.serialize());
}

class User implements Serializable, Loggable {
  serialize() { return "User data"; }
  log() { console.log("Logging user"); }
}

process(new User());
// Expected:
// Logging user
// User data
```

**Expected Output:**
```
Logging user
User data
```

**Why this output occurs.** The parameter requires both `serialize` and `log`. The `User` class provides both. The function can safely call either method because the intersection guarantees their presence.

---

#### Example 2: Intersection in a generic constraint

```typescript
// example_intersection_constraint.ts
interface Identifiable {
  id: string;
}

interface Timestamped {
  createdAt: Date;
}

function sortById<T extends Identifiable & Timestamped>(items: T[]): T[] {
  return items.sort((a, b) => a.id.localeCompare(b.id));
}

const items = [
  { id: "b", createdAt: new Date() },
  { id: "a", createdAt: new Date() },
];

console.log(sortById(items).map((i) => i.id));
// Expected: [ 'a', 'b' ]
```

**Expected Output:**
```
[ 'a', 'b' ]
```

**Why this output occurs.** The generic constraint `T extends Identifiable & Timestamped` ensures that every item has both `id` and `createdAt`. The function can safely access `.id` and `.createdAt` on any `T`.

---

#### Example 3: Mixin pattern with constructor intersections

```typescript
// example_intersection_mixin.ts
type Constructor = new (...args: any[]) => {};

// Mixin returns a class expression; the type is an intersection.
function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActive = false;
    activate() { this.isActive = true; }
    deactivate() { this.isActive = false; }
  };
}

class User {
  constructor(public name: string) {}
}

const ActiveUser = Activatable(User);
const user = new ActiveUser("Ada");

user.activate();
console.log(user.isActive); // Expected: true
console.log(user.name);     // Expected: Ada
```

**Expected Output:**
```
true
Ada
```

**Why this output occurs.** The mixin pattern in TypeScript 2.2+ represents a mixin class as an intersection between the mixin constructor type and the base class constructor type. The resulting `ActiveUser` type combines `User`'s members with the mixin's members.

---

### Real-World Cases

- **Dependency injection**: requiring services that satisfy multiple interfaces.
- **Mixin classes**: intersecting constructor types with base classes.
- **Generic constraints**: `<T extends A & B>` for functions that need multiple capabilities.
- **Branded types**: `string & { __brand: "UserId" }` for nominal-like safety.

---

## 5. Intersection Type Best Practices

### Definitions

**Core Definition.** **Intersection type best practices** are guidelines for using intersections in ways that maximize **type safety, compiler performance, and maintainability**, while avoiding the pitfalls of silent `never` conflicts, excessive intersection hierarchies, and performance degradation from recursive merging.

**Technical Definition.** Best practices center on the principle emphasized by the TypeScript Performance Wiki and community experts: **prefer `interface extends` over intersections for object inheritance**. Interfaces create a **single flat type**, detect property conflicts **at declaration**, and are **cached by the compiler**. Intersections **recursively merge on every use**, force the compiler to recompute the combined type repeatedly, and may silently produce `never` on conflicts. Intersections should be reserved for cases where interfaces cannot be used: **combining function types or primitives**, **creating mapped or conditional types**, and **one-off type combinations not reused elsewhere**.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep intersections safe and fast. The big rule is: **if you are building an object hierarchy, use `interface extends`, not `&`**. Intersections are for one-off combinations, primitives, and functions. They are slower and hide conflicts, so use them sparingly.

### Purposes

- **To maximize compiler performance** by preferring cached interfaces over recomputed intersections.
- **To catch property conflicts early** at declaration time.
- **To avoid silent `never` types** that mask bugs.
- **To choose the right tool** between `extends` and `&`.
- **To keep type definitions clear and maintainable**.

### Syntax Rules and Structure

Best practices are conventions that interact with intersection syntax. The core rules are:

1. **Prefer `interface extends` for object inheritance.** Interfaces create a single flat type, detect conflicts at declaration, and are cached by the compiler—resulting in **2–5× faster type resolution**.
2. **Use intersections only when interfaces cannot work.** Appropriate cases: combining **function types**, combining **primitives**, creating **mapped or conditional types**, and **one-off combinations not reused elsewhere**.
3. **Avoid building deep intersection hierarchies.** Each intersection layer adds another level of computation on every reference.
4. **Watch for silent `never`.** Intersections with incompatible properties produce `never` without a declaration-time error; use `extends` when conflict detection matters.
5. **Prefer `interface extends` in large codebases.** The performance difference is measurable and grows with codebase size.
6. **Use intersections for branded types and mixins.** These are legitimate use cases where intersections are the only or best tool.
7. **Consider merging object-only intersections into a single type literal.** ESLint plugins can enforce this: `type A = { x: 1 } & { y: 2 }` becomes `type A = { x: 1; y: 2 }`.

**Component breakdown (intersection vs. interface decision tree):**

```
Are you building an object type hierarchy used repeatedly?
  ├── Yes → Use interface extends.
  │        ├── Does it need to combine function types or primitives? → Use &.
  │        └── Is it a one-off combination? → & is acceptable.
  └── No → Are you combining function types, primitives, or mapped types?
           ├── Yes → Use intersection (&).
           └── No → Re-evaluate; interface extends is likely better for objects.
```

**Constraints and limitations.**

- Interface `extends` cannot combine unions, primitives, or function types; intersections are required for those.
- Intersections silently produce `never` on conflicts, which can hide bugs.
- Intersections are recomputed on every use, causing performance degradation in large codebases.
- ESLint plugins exist to enforce the `interface extends` preference.

### Annotated Complete Step by Step Code Examples

#### Example 1: Prefer interface `extends` over intersection for object hierarchies

```typescript
// example_best_extends.ts
// ❌ Intersection hierarchy: recomputed on every use, no conflict detection.
// type UserWithPermissions = User & Permissions & AuditInfo;

// ✅ Interface extends: cached, conflict-detecting, 2–5× faster.
interface User {
  id: string;
  name: string;
}

interface Permissions {
  canRead: boolean;
  canWrite: boolean;
}

interface AuditInfo {
  createdAt: Date;
}

interface UserWithPermissions extends User, Permissions, AuditInfo {}

const admin: UserWithPermissions = {
  id: "u1",
  name: "Ada",
  canRead: true,
  canWrite: true,
  createdAt: new Date(),
};

console.log(admin.canWrite); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** The interface hierarchy uses `extends`, which creates a single flat type, detects conflicts at declaration, and is cached by the compiler. The intersection version would recursively merge on every use, causing slower type checking.

---

#### Example 2: Intersection for one-off function type composition

```typescript
// example_best_function.ts
// ✅ Intersection is appropriate for combining function types.
type Handler = ((event: Event) => void) & { priority: number };

const handler: Handler = Object.assign(
  (event: Event) => console.log(event.type),
  { priority: 1 }
);

console.log(handler.priority); // Expected: 1
```

**Expected Output:**
```
1
```

**Why this output occurs.** Interfaces cannot extend function types. The intersection combines a callable signature with a property. This is a legitimate intersection use case where `extends` is not an option.

---

#### Example 3: Avoiding silent `never` with `extends`

```typescript
// example_best_never.ts
// ❌ Intersection silently produces never.
// type Broken = { x: string } & { x: number };
// const b: Broken = { x: "hello" }; // No error at declaration.

// ✅ Interface extends detects the conflict at declaration.
// interface A { x: string }
// interface B { x: number }
// interface C extends A, B {}
// Error: Interface 'C' cannot simultaneously extend types 'A' and 'B'.

console.log("Interface extends detects conflicts early");
```

**Expected Output:**
```
Interface extends detects conflicts early
```

**Why this output occurs.** The intersection `{ x: string } & { x: number }` compiles without error, but `x` becomes `never` and cannot be assigned. Interface `extends` reports the conflict at declaration, preventing the bug from propagating.

---

### Real-World Cases

- **Large codebases**: preferring `interface extends` for object hierarchies to benefit from caching and conflict detection.
- **Utility types**: using intersections for one-off combinations like `Omit<T, K> & { extra: string }`.
- **Function types**: combining callable signatures with properties via intersection.
- **Branded primitives**: `string & { __brand: "UserId" }` where interfaces cannot extend primitives.

---

## References

- TypeScript Playground — Union and Intersection Types (Official) - https://www.typescriptlang.org/play/typescript/primitives/union-and-intersection-types.ts.html
- TypeScript Handbook — Unions and Intersection Types (Deprecated) - https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html
- TypeScript 4.8 Release Notes — Improved Intersection Reduction, Union Compatibility, and Narrowing - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-8.html
- TypeScript Performance Wiki — Preferring Interfaces Over Intersections - https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections
- Total TypeScript — Comparing Intersection and Interface Extends - https://www.totaltypescript.com/workshops/typescript-pro-essentials/objects/comparing-intersection-and-interface-extends-in-typescript
- Prefer Interfaces Over Type Intersections — pproenca/dot-skills - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.curated/typescript/references/type-interfaces-over-intersections.md
- TypeScript 2.2 Release Notes — Mixin Class Pattern (Intersection Types) - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-2.html
- TypeScript ESLint — `prefer-interface-extends-over-type-intersection` (eslint-plugin-ts-type-preferences) - https://www.npmjs.com/package/eslint-plugin-ts-type-preferences
- ESLint Plugin — `prefer-merged-type-literal-over-intersection` - https://www.npmjs.com/package/eslint-plugin-ts-type-preferences
- Stack Overflow — Intersection Types vs Interfaces in TypeScript - https://stackoverflow.com/questions/52681316/intersection-types-vs-interfaces-in-typescript
- Stack Overflow — Type safety difference between type and interface when extending - https://stackoverflow.com/questions/71221037/type-safety-difference-between-type-and-interface-when-extending