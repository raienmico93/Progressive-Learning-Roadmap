## Generic Constraints

**Topic Overview.** **Generic constraints** are TypeScript’s mechanism for restricting the set of types that can be supplied as **type arguments** to a generic declaration. Without a constraint, a type parameter `T` is implicitly constrained by `unknown` — the top type to which all types are assignable — which means the generic body can only use operations valid for every conceivable type. A constraint, written with the `extends` keyword in the type parameter list (`<T extends Constraint>`), tells the compiler that `T` must be assignable to `Constraint`, which in turn permits the generic body to safely access the members of that constraint. Constraints are the bridge between **reusability** and **type safety**: they let a single generic component work over a family of types while guaranteeing that the members it relies on are present. The `keyof` operator is the canonical companion to constraints, enabling type-safe property access by restricting keys to the actual property names of an object type. Constraints are erased at compile time like all type-level constructs; they exist solely to guide the checker.

---

## 1. Generic Constraints (General)

### Definitions

**Core Definition.** A **generic constraint** is a restriction placed on a type parameter that limits the set of types that may be supplied as type arguments, specified with the `extends` keyword in the type parameter list.

**Technical Definition.** In a type parameter declaration `<T extends Constraint>`, the type parameter `T` is constrained such that every type argument supplied for `T` must be assignable to `Constraint`. Within the generic declaration’s body, `T` is treated as a subtype of `Constraint`, so members of `Constraint` are accessible on values of type `T`. If no constraint is provided, `T` is implicitly constrained by `unknown`. Constraints participate in type argument inference: if an inferred argument does not satisfy the constraint, the compiler reports an error.

**Beginner-Friendly Explanation.** A generic constraint is **a rule about what types a generic will accept**. Without a constraint, `T` could be anything — a number, a string, a boolean — so the function cannot assume `T` has any particular property. With a constraint like `<T extends { length: number }>`, you are saying “`T` must be something that has a `length` property.” Now the function can safely use `.length`. Constraints turn a generic from “any type” into **“any type that fits”**.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Restrictive** | Limits the set of permissible type arguments to those assignable to the constraint. |
| **Enabling** | Grants access to the members of the constraint inside the generic body. |
| **Structural** | Compatibility is determined by members, not by nominal identity. |
| **Inference-participating** | Constraints guide type argument inference and produce errors when inferred arguments do not satisfy them. |
| **Composable** | Multiple constraints may be combined with intersection types (`&`). |
| **Erased** | Constraints have no runtime representation. |

### Prerequisites

- **Basic generics**: type parameters and type arguments.
- **Structural typing**: awareness that TypeScript compares object shapes by members.
- **Union and intersection types**: for combining constraints.
- **The `keyof` operator**: for property-key constraints.

### Related Programming Areas

- **Generic functions, interfaces, and classes**: constraints apply uniformly across all generic constructs.
- **Utility types**: `Pick<T, K extends keyof T>` and `Record<K extends keyof any, T>` rely on constraints.
- **Type-level programming**: conditional types and mapped types frequently constrain their inputs.
- **API design**: constraints document and enforce the minimal capabilities a generic component requires.

---

## 2. `extends` (The Constraint Keyword)

### Definitions

**Core Definition.** The **`extends` keyword**, when used in a type parameter list, introduces a **constraint** that restricts the type parameter to types assignable to the specified constraint type.

**Technical Definition.** In `<T extends Constraint>`, `extends` is not class inheritance; it expresses a **subtype (assignability) relationship**. A type argument `A` satisfies `T extends Constraint` if `A` is assignable to `Constraint` under TypeScript’s structural rules. Inside the generic body, `T` is narrowed to `Constraint`, so the members of `Constraint` are available. `extends` also appears in conditional types (`A extends B ? C : D`), where it tests assignability rather than declaring a constraint; the two uses are related but distinct.

**Beginner-Friendly Explanation.** `extends` in a generic constraint means **“must be compatible with”** or **“must be a subset of”**. When you write `<T extends HasLength>`, you are saying “`T` must be a type that fits the `HasLength` shape.” It does not mean `T` inherits from `HasLength` in the class sense; it means `T` has at least the members that `HasLength` requires.

### Purposes

- **To restrict the permissible type arguments** to those that satisfy a required structure.
- **To grant safe access to members** of the constraint inside the generic body.
- **To document the minimal capabilities** a generic component requires.
- **To improve error messages** by reporting constraint violations at the call site.

### Syntax Rules and Structure

**General syntax:**

```typescript
function fn<T extends Constraint>(value: T): void { ... }
interface Iface<T extends Constraint> { ... }
class Cls<T extends Constraint> { ... }
type Alias<T extends Constraint> = ...;
```

**Component breakdown:**

- `<T extends Constraint>` — the constrained type parameter declaration.
- `T` — the type parameter.
- `extends` — the constraint keyword (assignability, not inheritance).
- `Constraint` — any type expression: an interface, a primitive, a union, an object type literal, `keyof X`, etc.

**Syntax rules.**

- The constraint follows `extends` in the type parameter list.
- The constraint may reference other type parameters (e.g., `<T, K extends keyof T>`).
- A constraint may be an intersection (`A & B`) or a union (`A | B`).
- If an explicit type argument does not satisfy the constraint, it is a compile-time error.

**Constraints and limitations.**

- `extends` in a constraint does not permit access to members not declared in the constraint; over-constraining couples the generic to a specific implementation.
- The constraint is checked at compile time only; no runtime enforcement occurs.
- A constraint of `{}` accepts all non-nullish types but provides no member access beyond `Object` members.

### Annotated Complete Step by Step Code Examples

#### Example 1: Constraint enabling member access

```typescript
// example_extends_access.ts
// Step 1: Define the minimal structure needed.
interface HasLength {
  length: number;
}

// Step 2: Constrain T to HasLength.
function logLength<T extends HasLength>(value: T): number {
  return value.length;   // legal: T is known to have length
}

// Step 3: Call with compatible types.
console.log(logLength("hello"));    // Expected: 5
console.log(logLength([1, 2, 3]));  // Expected: 3
// console.log(logLength(42));       // Error: number does not satisfy HasLength
```

**Expected Output:**
```
5
3
```

**Why this output occurs.** The constraint `T extends HasLength` guarantees that every `T` has a `length: number` property, making `value.length` legal inside the function. Strings and arrays satisfy `HasLength`; numbers do not, so the call with `42` is rejected at compile time.

#### Example 2: Constraint without member access is still a restriction

```typescript
// example_extends_restriction.ts
function printObject<T extends object>(value: T): void {
  console.log(value);
}

printObject({ a: 1 });       // OK
printObject([1, 2, 3]);      // OK (arrays are objects)
// printObject("hello");     // Error: string is not assignable to object
```

**Expected Output:**
```
{ a: 1 }
[ 1, 2, 3 ]
```

**Why this output occurs.** `T extends object` restricts arguments to non-primitive values. Strings, numbers, and booleans are rejected; plain objects, arrays, and functions are accepted. The constraint does not grant access to specific properties, but it does restrict the domain of `T`.

### Real-World Cases

- **Type-safe property access**: `getProperty<T, K extends keyof T>` constrains the key to valid property names.
- **Component props**: `List<T extends HasId>` ensures every item has an `id` for use as a React key.
- **Repository APIs**: `Repository<T extends Entity>` ensures entities have the members the repository requires.

---

## 3. Constraint Types

### Definitions

**Core Definition.** **Constraint types** are the type expressions that appear on the right-hand side of `extends` in a type parameter declaration, specifying the structure, union, or condition that type arguments must satisfy.

**Technical Definition.** A constraint type may be any type expression: an interface or class type, a primitive (`string`, `number`), a union (`A | B`), an intersection (`A & B`), an object type literal (`{ id: string }`), `keyof X`, a conditional type, or a type parameter. The constraint determines both the set of permissible type arguments and the members accessible on the constrained type parameter inside the generic body.

**Beginner-Friendly Explanation.** A constraint type is **the shape or condition you require**. It can be a simple object shape (`{ length: number }`), a primitive (`string`), a union of allowed types, or `keyof T` to restrict keys. Whatever you put after `extends` is the requirement.

### Purposes

- **To specify structural requirements** such as required properties or methods.
- **To restrict type arguments to primitives, objects, or unions**.
- **To bind keys to valid property names** using `keyof`.
- **To compose multiple requirements** through intersection or union types.

### Syntax Rules and Structure

**Common constraint types:**

```typescript
<T extends HasLength>                 // interface / object shape
<T extends object>                    // non-primitive
<T extends string | number>           // union of primitives
<T extends A & B>                     // intersection (multiple requirements)
<T extends keyof U>                   // key constraint
<T extends Record<string, unknown>>   // record-like object
```

**Component breakdown:**

- `HasLength` — an interface or type alias specifying required members.
- `object` — the non-primitive type.
- `string | number` — a union restricting to primitives.
- `A & B` — an intersection requiring both `A` and `B`.
- `keyof U` — the union of property names of `U`.

**Syntax rules.**

- The constraint type may reference other type parameters in scope.
- Intersections are the standard way to require multiple structures.
- Unions allow any of the listed types.
- `{}` accepts all non-nullish types but provides almost no member access.

**Constraints and limitations.**

- `object` excludes primitives but does not guarantee any specific property.
- `{}` is often too permissive; prefer a specific structural constraint.
- Deeply nested intersections reduce readability; use a named type alias.

### Annotated Complete Step by Step Code Examples

#### Example 1: Structural constraint

```typescript
// example_constraint_structural.ts
interface Identifiable {
  id: string;
}

function findById<T extends Identifiable>(items: T[], id: string): T | undefined {
  return items.find((item) => item.id === id);
}

const users = [{ id: "1", name: "Ada" }, { id: "2", name: "Grace" }];
console.log(findById(users, "2")?.name);   // Expected: Grace
```

**Expected Output:**
```
Grace
```

**Why this output occurs.** The constraint `T extends Identifiable` guarantees an `id: string` property, making `item.id` legal. The inferred type argument is `{ id: string; name: string }`, which satisfies the constraint.

#### Example 2: Union constraint

```typescript
// example_constraint_union.ts
function formatId<T extends string | number>(id: T): string {
  return typeof id === "string" ? id.toUpperCase() : id.toFixed(0);
}

console.log(formatId("abc"));   // Expected: ABC
console.log(formatId(42));      // Expected: 42
// console.log(formatId(true)); // Error: boolean is not assignable to string | number
```

**Expected Output:**
```
ABC
42
```

**Why this output occurs.** The union constraint restricts `T` to `string` or `number`. Inside the function, narrowing via `typeof` handles both possibilities. Booleans and other types are rejected.

### Real-World Cases

- **Entity constraints**: `T extends { id: string }` for repositories and lists.
- **Primitive restrictions**: `T extends string | number` for formatting functions.
- **Record constraints**: `T extends Record<string, unknown>` for table-like components.

---

## 4. Multiple Constraints

### Definitions

**Core Definition.** **Multiple constraints** combine two or more constraint types on a single type parameter, requiring the type argument to satisfy **all** of them simultaneously.

**Technical Definition.** TypeScript does not support the syntax `<T extends A, B>` for a single type parameter. Multiple constraints are expressed with the **intersection operator** `&`: `<T extends A & B>`. A type argument satisfies the constraint if it is assignable to both `A` and `B`. Alternatively, multiple type parameters may each have their own constraint (`<T extends A, U extends B>`), which is distinct from a single parameter with multiple constraints.

**Beginner-Friendly Explanation.** Multiple constraints mean **“the type must satisfy all of these requirements at once.”** You write `<T extends A & B>` to say “`T` must have everything `A` requires **and** everything `B` requires.” You cannot write `<T extends A, B>`; the `&` intersection is the correct syntax.

### Purposes

- **To require a type argument to satisfy several structural requirements** simultaneously.
- **To compose constraints from reusable pieces** rather than creating a monolithic constraint interface.
- **To combine a structural requirement with a primitive or union restriction**.

### Syntax Rules and Structure

```typescript
<T extends A & B>                     // intersection of two constraints
<T extends HasId & { active: boolean }> // structural + inline shape
<T extends A, U extends B>            // two parameters, each constrained
```

**Component breakdown:**

- `A & B` — an intersection type; `T` must be assignable to both `A` and `B`.
- `HasId & { active: boolean }` — an intersection of a named interface and an inline object type.
- `<T extends A, U extends B>` — two independent type parameters with separate constraints.

**Syntax rules.**

- Multiple constraints on one parameter use `&`, not `,`.
- Intersections may combine interfaces, object type literals, and type aliases.
- Multiple type parameters are separated by commas, each with its own optional constraint.
- Deeply nested intersections (`A & B & C & D`) should be extracted into a named type alias for readability.

**Constraints and limitations.**

- Intersections are not the same as union constraints: `A & B` requires both; `A | B` requires one.
- Contradictory intersections (e.g., `string & number`) produce `never` and make the generic unusable.
- Multiple constraints can make inference harder; prefer the minimal necessary set.

### Annotated Complete Step by Step Code Examples

#### Example 1: Intersection constraint

```typescript
// example_multiple_constraints.ts
interface HasId {
  id: string;
}

interface HasName {
  name: string;
}

function describe<T extends HasId & HasName>(item: T): string {
  return `${item.name} (${item.id})`;
}

console.log(describe({ id: "1", name: "Ada" }));   // Expected: Ada (1)
// console.log(describe({ id: "1" }));             // Error: missing name
```

**Expected Output:**
```
Ada (1)
```

**Why this output occurs.** The constraint `T extends HasId & HasName` requires both `id: string` and `name: string`. The argument satisfies both; an argument missing `name` is rejected.

#### Example 2: Multiple type parameters, each constrained

```typescript
// example_multiple_params_constraints.ts
function merge<T extends object, U extends object>(base: T, updates: U): T & U {
  return { ...base, ...updates };
}

const merged = merge({ name: "Ada" }, { age: 36 });
console.log(merged);   // Expected: { name: 'Ada', age: 36 }
```

**Expected Output:**
```
{ name: 'Ada', age: 36 }
```

**Why this output occurs.** `T` and `U` each have an independent `object` constraint. The return type `T & U` preserves both shapes. This is distinct from a single parameter with an intersection constraint.

### Real-World Cases

- **Entity + status constraints**: `T extends Entity & { status: Status }`.
- **Sortable lists**: `T extends HasId & { createdAt: Date }`.
- **Composed API contracts**: `T extends Serializable & Identifiable`.

---

## 5. `keyof` (Key Constraints)

### Definitions

**Core Definition.** The **`keyof` operator** takes an object type and produces a **union of its property names** (string or numeric literal types). Used as a constraint (`K extends keyof T`), it restricts a key type parameter to the actual keys of an object type.

**Technical Definition.** For a type `T`, `keyof T` is the union of all known, public property names of `T`. If `T` has a string or number index signature, `keyof T` returns those types instead. The constraint `<K extends keyof T>` ensures that `K` is a valid property name of `T`, enabling **type-safe indexed access**: the expression `obj[key]` has type `T[K]`, the precise type of the property named by `K`.

**Beginner-Friendly Explanation.** `keyof` means **“the names of the properties.”** `keyof User` is `"id" | "name" | "email"`. When you constrain `K extends keyof T`, you are saying “`K` must be one of the property names of `T`.” This lets a function look up a property safely: pass `"name"`, get a `string`; pass `"age"`, get a `number` — and a typo like `"nmae"` is caught at compile time.

### Purposes

- **To restrict keys to valid property names** of an object type.
- **To preserve the precise return type** of a property access (`T[K]`).
- **To catch typos in property names at compile time** rather than at runtime.
- **To enable generic utilities** like `Pick`, `Omit`, and property plucking.

### Syntax Rules and Structure

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

**Component breakdown:**

- `T` — the object type parameter.
- `K extends keyof T` — the key type parameter, constrained to `T`’s property names.
- `obj: T` — the object argument.
- `key: K` — the key argument, restricted to valid names.
- `): T[K]` — the **indexed access type**; the return type is the type of the property `K` on `T`.

**Syntax rules.**

- `keyof T` produces a union of literal types (e.g., `"id" | "name"`).
- `K extends keyof T` constrains `K` to that union.
- `T[K]` is the indexed access type, preserving the property’s precise type.
- `keyof` works with interfaces, type aliases, and object type literals.

**Constraints and limitations.**

- `keyof` returns only **public** property names; private and protected members are excluded.
- If `T` has an index signature, `keyof T` may be `string | number`, reducing key precision.
- Without the `K extends keyof T` constraint, the return type would be the union of all property types (`T[keyof T]`), losing precision.

### Annotated Complete Step by Step Code Examples

#### Example 1: Type-safe property access with `keyof`

```typescript
// example_keyof_access.ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: "Ada", active: true };

const name = getProperty(user, "name");   // string
const id = getProperty(user, "id");       // number
const active = getProperty(user, "active"); // boolean

console.log(name, id, active);   // Expected: Ada 1 true
// getProperty(user, "age");     // Error: "age" is not keyof typeof user
```

**Expected Output:**
```
Ada 1 true
```

**Why this output occurs.** `K extends keyof T` restricts the key to `"id" | "name" | "active"`. `T[K]` resolves to the precise property type for each key: `string` for `"name"`, `number` for `"id"`, `boolean` for `"active"`. The invalid key `"age"` is rejected at compile time.

#### Example 2: `keyof` with an index signature

```typescript
// example_keyof_index.ts
type Dictionary = { [key: string]: number };

function getValue<T extends Dictionary, K extends keyof T>(dict: T, key: K): T[K] {
  return dict[key];
}

const scores: Dictionary = { alice: 10, bob: 20 };
console.log(getValue(scores, "alice"));   // Expected: 10
```

**Expected Output:**
```
10
```

**Why this output occurs.** `keyof Dictionary` is `string` (from the index signature). `K extends string` is satisfied by `"alice"`. The return type `T[K]` is `number` because every property value in `Dictionary` is a `number`.

### Real-World Cases

- **Property plucking**: `pluck<T, K extends keyof T>(items: T[], key: K): T[K][]`.
- **Table components**: `columns: { key: keyof T; header: string }[]` catches typos in column keys.
- **Configuration updates**: `update<T, K extends keyof T>(obj: T, key: K, value: T[K])`.

---

## 6. Generic Constraint Best Practices

### Definitions

**Core Definition.** **Generic constraint best practices** are guidelines for choosing, composing, and documenting constraints in ways that maximize **reusability, type safety, inference success, and readability**, while avoiding over-constraining, under-constraining, and misleading constraint types.

**Technical Definition.** Best practices include: constraining to the **minimal structure** the generic body actually uses; preferring **`keyof` constraints** for property access; using **intersections** for multiple requirements and extracting deeply nested intersections into named type aliases; avoiding `{}` and `object` when a more specific constraint is possible; and documenting constraints so callers understand the required capabilities. The guiding principle is: **constrain only what you access, and access only what you constrain**.

**Beginner-Friendly Explanation.** Best practices help you write constraints that are **useful but not restrictive**. Constrain only the members you actually use. Use `keyof` when you need valid property names. Combine constraints with `&` for multiple requirements, but don’t overdo it. Avoid constraints so broad they don’t help (`{}`) or so narrow they exclude reasonable types.

### Purposes

- **To maximize reusability** by requiring only the members the generic actually needs.
- **To maximize type safety** by requiring enough structure to access members safely.
- **To improve inference** by keeping constraints aligned with parameter types.
- **To keep signatures readable** by using named constraints and avoiding deep intersections.
- **To document intent** so callers know what capabilities are required.

### Syntax Rules and Structure

**Key best-practice rules:**

1. **Constrain to the minimal structure used.** If the generic only accesses `.length`, constrain to `{ length: number }`, not a full `Array` or `String` type.
2. **Prefer `keyof` for property access.** Use `<K extends keyof T>` rather than `<K extends string>` when the key must be a valid property name.
3. **Use `&` for multiple constraints, not commas.** `<T extends A & B>` is the correct syntax; `<T extends A, B>` is invalid.
4. **Extract deep intersections into named aliases.** `type EntityWithStatus = Entity & { status: Status }` improves readability.
5. **Avoid `{}` when a specific constraint is possible.** `{}` accepts almost everything but provides almost no member access.
6. **Prefer `object` over `{}` when excluding primitives** is the goal.
7. **Document constraints** when their purpose is not self-evident, using TSDoc.

**Constraints and limitations.**

- Over-constraining reduces reusability; a constraint that requires members the body never uses is a red flag.
- Under-constraining forces casts to `any` inside the body, defeating type safety.
- Contradictory intersections (`string & number`) produce `never` and make the generic unusable.
- Deeply nested intersections harm readability and error messages.

### Annotated Complete Step by Step Code Examples

#### Example 1: Minimal constraint

```typescript
// example_best_minimal.ts
// GOOD: constraint includes only what the body uses.
function printLength<T extends { length: number }>(value: T): void {
  console.log(value.length);
}

// Over-constrained: requires unrelated members.
// function bad<T extends { length: number; name: string }>(value: T) { ... }

printLength("hello");      // Expected: 5
printLength([1, 2, 3]);    // Expected: 3
```

**Expected Output:**
```
5
3
```

**Why this output occurs.** The constraint requires only `length`, so strings and arrays both qualify. Requiring `name` would exclude both, reducing reusability without benefit.

#### Example 2: `keyof` instead of broad string

```typescript
// example_best_keyof.ts
// GOOD: K is restricted to actual property names.
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// BAD: key: string loses type safety and requires a cast.
function getPropertyBad<T>(obj: T, key: string): unknown {
  return (obj as any)[key];
}

const user = { id: 1, name: "Ada" };
console.log(getProperty(user, "name"));   // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `K extends keyof T` restricts the key to `"id" | "name"` and returns the precise property type `string` for `"name"`. The unconstrained version accepts any string and returns `unknown`, forcing a cast and losing safety.

#### Example 3: Named alias for multiple constraints

```typescript
// example_best_alias.ts
interface Entity {
  id: string;
}

interface Timestamped {
  createdAt: Date;
}

// Extract the intersection into a named alias.
type EntityWithTimestamp = Entity & Timestamped;

function sortByCreation<T extends EntityWithTimestamp>(items: T[]): T[] {
  return [...items].sort((a, b) => a.createdAt.getTime() - b.createdAt.getTime());
}

const rows = [
  { id: "1", createdAt: new Date("2024-01-02") },
  { id: "2", createdAt: new Date("2024-01-01") },
];

console.log(sortByCreation(rows).map((r) => r.id));   // Expected: [ '2', '1' ]
```

**Expected Output:**
```
[ '2', '1' ]
```

**Why this output occurs.** The named alias `EntityWithTimestamp` makes the constraint readable. The function accesses `createdAt` and `id`, both guaranteed by the constraint. The intersection is documented in one place.

### Real-World Cases

- **Repository design**: Minimal `T extends { id: string }` constraints keep repositories reusable.
- **UI tables**: `keyof T` constraints for column keys catch typos at compile time.
- **Composed domain constraints**: Named aliases for `Entity & Timestamped & SoftDeletable` improve clarity.

---

## References

- TypeScript Handbook — Generics (Generic Constraints) - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook — Keyof Type Operator - https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- TypeScript Handbook — Creating Types from Types - https://www.typescriptlang.org/docs/handbook/2/types-from-types.html
- TypeScript Documentation — Indexed Access Types - https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
- TypeScript Documentation — Conditional Types - https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Generic Constraints — React Patterns (keyof and extends) - https://github.com/voku/agent-skills/blob/a51823f68844c282d1a32f60fd575840c8b3465f/skills/typescript-react-patterns/AGENTS.md
- TypeScript 泛型约束完整指南 (extends、keyof、T[K]) - https://www.w3cschool.cn/article/36032363.html
- Can You Specify Multiple Type Constraints For TypeScript Generics — Stack Overflow - https://stackoverflow.com/questions/17125764/can-you-specify-multiple-type-constraints-for-typescript-generics
- TypeScript 两种泛型函数定义方式对比 (K extends keyof T) - https://www.volcengine.com/article/179177