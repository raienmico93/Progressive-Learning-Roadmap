# Objects in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** In TypeScript, an **object type** is a type that describes the **shape** of a JavaScript object—the properties it has, the types of those properties, and whether they are optional or readonly. TypeScript provides **structural typing**: an object is compatible with an object type if it has at least the required members, regardless of how it was created or declared. Object types may be written **anonymously** (inline), as **interfaces**, or as **type aliases**.

**Technical Definition.** An object type is a type expression that specifies a set of **members**: property signatures (`name: Type`), method signatures (`method(): Type`), index signatures (`[key: string]: Type`), call signatures (`(args): Type`), and construct signatures (`new (args): Type`). Object literals undergo **excess property checks** when assigned directly to a typed target, a stricter form of structural compatibility. Type inference **widens** literal types in mutable contexts unless preserved by `const`, `as const`, or `satisfies`.

**Beginner-Friendly Explanation.** An object type is **a description of what an object should look like**. It lists the properties (data) and methods (actions) an object must have, which are optional, and which cannot be changed. TypeScript uses this description to check that you are using objects correctly. When you write an object literal like `{ name: "Ada", age: 36 }`, TypeScript can **infer** its type automatically, or you can **annotate** it explicitly with an interface or type alias.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Structural typing** | Objects are compatible by shape, not by name or declaration. |
| **Property modifiers** | `?` marks optional properties; `readonly` prevents reassignment. |
| **Excess property checks** | Direct object literal assignments reject extra properties; intermediate variables bypass the check. |
| **Type widening** | Mutable contexts widen literals to base types; `const`, `as const`, and `satisfies` preserve precision. |
| **Index signatures** | `[key: string]: T` allows dynamic keys with a consistent value type. |
| **Nested objects** | Object types may be nested arbitrarily, with each level checked recursively. |
| **Erasure** | All object type information is erased at compile time and emits no JavaScript. |

---

### Prerequisites

- **JavaScript objects**: object literals, property access, and `Object.keys`/`values`.
- **TypeScript basics**: type annotations, type aliases, and interfaces.
- **Primitive types**: `string`, `number`, `boolean`, `null`, `undefined`.
- **Literal types**: `"hello"`, `42`, `true`.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Interfaces and type aliases** — two primary ways to name object types.
- **Structural typing** and **excess property checks**.
- **Optional and readonly properties** — property modifiers.
- **Index signatures** — dictionary-style object typing.
- **Utility types** — `Partial<T>`, `Required<T>`, `Readonly<T>`, `Pick<T, K>`, `Omit<T, K>`.
- **`satisfies` and `as const`** — validation without widening.

---

## Core Concepts / Features

---

## 1. Object Types

### Definitions

**Core Definition.** An **object type** describes the shape of an object—the properties and methods it must have, and their types. It may be **anonymous** (inline), or named via an **interface** or **type alias**.

**Technical Definition.** Object types are type expressions containing **members**: property signatures, method signatures, index signatures, call signatures, and construct signatures. They work **structurally**: a value matches an object type if it has the required structure. Object types may be **nested**, **generic**, and composed via **intersection** (`&`) or **union** (`|`).

**Beginner-Friendly Explanation.** An object type is **a blueprint for objects**. It says what properties an object has and what types those properties are. You can write it inline, or give it a name with an interface or type alias.

### Purposes

- **To describe the shape of data** passed between functions and modules.
- **To enable compile-time checking** of property access and assignment.
- **To document APIs** and configuration objects.
- **To support structural polymorphism** where any conforming object is accepted.

### Syntax Rules and Structure

**Anonymous object type:**

```
function greet(person: { name: string; age: number }) { ... }
```

**Named interface:**

```
interface Person { name: string; age: number; }
```

**Named type alias:**

```
type Person = { name: string; age: number; };
```

**Component breakdown:**

- `{ ... }` — the object type literal.
- `name: string` — property signature.
- `age: number` — property signature.

**Syntax rules.**

- Members may be separated by `;` or `,`.
- Property keys may be identifiers, string literals, or numeric literals.
- Object types may be **generic**: `type Box<T> = { value: T };`.
- Object types are **erased** at runtime.

**Constraints and limitations.**

- Structural typing means extra properties are allowed unless excess property checks apply.
- Object types cannot be used with `instanceof`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Anonymous, interface, and type alias

```typescript
// example_object_types.ts
// Anonymous object type.
function greet(person: { name: string; age: number }) {
  return "Hello " + person.name;
}

// Interface.
interface Person {
  name: string;
  age: number;
}

// Type alias.
type PersonAlias = {
  name: string;
  age: number;
};

console.log(greet({ name: "Ada", age: 36 })); // Expected: Hello Ada
const p: PersonAlias = { name: "Grace", age: 85 };
console.log(p.name); // Expected: Grace
```

**Expected Output:**
```
Hello Ada
Grace
```

**Why this output occurs.** All three forms describe the same shape. The function accepts any object with `name` and `age`. At runtime, the type information is erased.

---

### Real-World Cases

- **API response models**: `interface User { id: string; name: string }`.
- **Function parameters**: `function save(config: { path: string; overwrite: boolean })`.
- **Configuration objects**: `type AppConfig = { port: number; host: string }`.

---

## 2. Object Literals

### Definitions

**Core Definition.** An **object literal** is a JavaScript expression that creates an object with a fixed set of properties, written as `{ key: value, ... }`. In TypeScript, object literals are **contextually typed** by their assignment target and undergo **excess property checks** when assigned directly.

**Technical Definition.** Object literals are subject to **fresh literal** treatment: when assigned directly to a typed variable or passed as an argument, TypeScript checks that they do not contain properties absent from the target type (excess property checks). Passing through an intermediate variable **bypasses** the check due to structural typing. The `satisfies` operator (TS 4.9) restores excess property checking for intermediate variables.

**Beginner-Friendly Explanation.** An object literal is **the `{ ... }` syntax you write to create an object**. TypeScript checks it more strictly than variables: if you add an extra property that the target type does not know about, TypeScript warns you—but only if you write the literal directly in the assignment.

### Purposes

- **To create objects inline** with type checking.
- **To trigger excess property checks** that catch typos and extra fields.
- **To leverage contextual typing** for properties.

### Syntax Rules and Structure

**General syntax:**

```
const obj: Type = { prop1: value1, prop2: value2 };
```

**Component breakdown:**

- `{ prop1: value1 }` — the object literal.
- `: Type` — the contextual type.

**Syntax rules.**

- Object literals are checked against the contextual type.
- Excess properties are rejected when the literal is assigned **directly**.
- Intermediate variables bypass excess property checks.
- `satisfies` enforces excess property checks even through intermediate variables.

**Constraints and limitations.**

- Excess property checks only apply to **fresh** object literals, not variables.
- Object literals cannot have duplicate keys.

### Annotated Complete Step by Step Code Examples

#### Example 1: Excess property checks

```typescript
// example_object_literal_epc.ts
interface CreateUserInput {
  name: string;
  email: string;
}

// ✅ Direct literal: excess properties are caught.
// createUser({ name: "Alice", email: "a@example.com", role: "admin" });
// Error: 'role' does not exist in type 'CreateUserInput'.

// ❌ Intermediate variable: excess properties bypass the check.
const input = { name: "Alice", email: "a@example.com", role: "admin" };
function createUser(input: CreateUserInput) { /* ... */ }
createUser(input); // Compiles — "role" silently ignored.

// ✅ `satisfies` restores the check.
const safeInput = {
  name: "Alice",
  email: "a@example.com",
  role: "admin",
} satisfies CreateUserInput;
// Error: 'role' does not exist in type 'CreateUserInput'.
```

**Expected Output:**
```
(compile-time errors as annotated; no runtime output)
```

**Why this output occurs.** Direct object literals undergo excess property checks. Intermediate variables are structurally typed, so extra properties are allowed. `satisfies` forces the excess property check on the intermediate literal.

---

### Real-World Cases

- **Function arguments**: `createUser({ name: "Ada", email: "ada@example.com" })`.
- **Configuration objects**: `const config = { port: 3000, host: "localhost" } satisfies Config`.
- **Test fixtures**: object literals with known shapes.

---

## 3. Object Type Annotations

### Definitions

**Core Definition.** An **object type annotation** is an explicit type declaration attached to a variable, parameter, return value, or property that specifies the object's shape using an object type expression, interface, or type alias.

**Technical Definition.** Annotations are written after the variable or parameter name using `: Type`. They **suppress widening** and enforce the declared shape, including optional and readonly modifiers. Annotations may reference named interfaces or type aliases, or use inline object type literals.

**Beginner-Friendly Explanation.** An object type annotation is **you telling TypeScript exactly what shape an object should have**. Instead of letting TypeScript infer it, you write `: Person` or `: { name: string }` to enforce a specific contract.

### Purposes

- **To enforce a specific object shape** regardless of the assigned literal.
- **To prevent type widening** by providing a contextual type.
- **To document the intended contract** of an object.
- **To enable excess property checks** on object literals.

### Syntax Rules and Structure

**General syntax:**

```
const obj: ObjectType = { ... };
function f(param: ObjectType): ReturnType { ... }
```

**Component breakdown:**

- `: ObjectType` — the annotation.
- `ObjectType` — an interface, type alias, or inline object type.

**Syntax rules.**

- Annotations may be inline (`{ name: string }`) or named (`Person`).
- Annotations enforce the shape but do not prevent excess property checks (which apply to literals).
- Annotations widen literal types unless `as const` or `satisfies` is used.

**Constraints and limitations.**

- Annotations widen literal types; use `satisfies` to validate without widening.
- Annotations do not perform runtime validation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Annotation vs. inference

```typescript
// example_annotation.ts
// Inference: property types are widened.
const inferred = { name: "Ada", age: 36 };
// type: { name: string; age: number }

// Annotation: enforces a specific shape.
interface Person {
  name: string;
  age: number;
}
const annotated: Person = { name: "Ada", age: 36 };

console.log(annotated.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The inferred type widens `"Ada"` to `string`. The annotation enforces the `Person` contract and checks the object literal against it.

---

### Real-World Cases

- **Function parameters**: `function save(user: User): void`.
- **Variable declarations**: `const config: AppConfig = { ... }`.
- **Return types**: `function create(): User { ... }`.

---

## 4. Object Type Inference

### Definitions

**Core Definition.** **Object type inference** is the process by which TypeScript automatically determines the type of an object literal based on its properties and values, without an explicit annotation.

**Technical Definition.** Type inference for objects is **context-sensitive** and subject to **widening**: in mutable contexts, literal property types widen to their base primitives (`string`, `number`, `boolean`). `const` declarations, `as const` assertions, and `const` type parameters preserve literal types. The `satisfies` operator validates without widening.

**Beginner-Friendly Explanation.** Type inference is **TypeScript figuring out the type of an object for you**. When you write `const user = { name: "Ada", age: 36 }`, TypeScript infers `{ name: string; age: number }`—it widens `"Ada"` to `string` because the property is mutable. Use `as const` or `satisfies` to keep exact values.

### Purposes

- **To reduce annotation verbosity** while retaining type safety.
- **To enable contextual typing** of object properties.
- **To preserve literal precision** when needed with `as const` or `satisfies`.

### Syntax Rules and Structure

**Inference from object literal:**

```
const obj = { name: "Ada", age: 36 };
// type: { name: string; age: number }
```

**Widening:**

- `let`/`var` and mutable properties widen literals to base types.
- `const` declarations preserve literal types for the binding.
- `as const` makes properties `readonly` and preserves literals.
- `satisfies` validates without widening.

**Syntax rules.**

- Inference is **syntactic**: it uses the values in the literal.
- Contextual typing applies when the literal is assigned to a typed target.
- `const` type parameters (TS 5.0+) preserve literals through generic calls.

**Constraints and limitations.**

- Inference may produce `any` in non-strict configurations.
- Inference does not perform whole-program analysis.

### Annotated Complete Step by Step Code Examples

#### Example 1: Widening and preservation

```typescript
// example_inference_widening.ts
// Mutable context widens literals.
const mutable = { status: "success" };
// type: { status: string }

// `as const` preserves literals and makes properties readonly.
const immutable = { status: "success" } as const;
// type: { readonly status: "success" }

// `satisfies` validates without widening.
const validated = { status: "success" } satisfies { status: string };
// type: { status: "success" }

console.log(mutable.status);   // Expected: success
console.log(immutable.status); // Expected: success
console.log(validated.status); // Expected: success
```

**Expected Output:**
```
success
success
success
```

**Why this output occurs.** `mutable` widens `"success"` to `string`. `immutable` preserves the literal type and makes the property readonly. `validated` preserves the literal type while validating against the target shape.

---

### Real-World Cases

- **Configuration objects**: using `satisfies` to validate without widening.
- **Redux action creators**: `as const` to preserve literal action types.
- **Generic factories**: `const T` to preserve literal configuration values.

---

## 5. Nested Objects

### Definitions

**Core Definition.** **Nested objects** are objects whose properties are themselves objects, forming a hierarchy. TypeScript checks nested object types **recursively**, ensuring that each level conforms to its declared shape.

**Technical Definition.** Nested object types are declared by nesting object type literals or referencing named interfaces inside other object types. The compiler checks nested properties against their declared types, including optionality and readonly modifiers. Excess property checks apply **at each level** for direct object literals.

**Beginner-Friendly Explanation.** Nested objects are **objects inside objects**. TypeScript checks the shape of every level, so if you have a `user` with an `address` object, TypeScript ensures the address has the right properties too.

### Purposes

- **To model hierarchical data** such as users with addresses, orders with items, and configurations with sub-configs.
- **To enforce recursive shape checking** at every level.
- **To document complex data structures** clearly.

### Syntax Rules and Structure

**General syntax:**

```
interface Outer {
  inner: {
    prop: Type;
  };
}
```

**Component breakdown:**

- `inner: { ... }` — a nested object type.

**Syntax rules.**

- Nested types may be inline or named.
- Optional nested properties: `inner?: { ... }`.
- Readonly nested properties: `readonly inner: { ... }`.
- Excess property checks apply recursively for direct literals.

**Constraints and limitations.**

- `readonly` is **shallow**; it does not deep-freeze nested objects.
- Nested excess property checks may be bypassed by intermediate variables at any level.

### Annotated Complete Step by Step Code Examples

#### Example 1: Nested object with excess property checks

```typescript
// example_nested_objects.ts
interface Address {
  city: string;
  zip: string;
}

interface User {
  name: string;
  address: Address;
}

const user: User = {
  name: "Ada",
  address: { city: "London", zip: "SW1A" },
};

console.log(user.address.city); // Expected: London

// ❌ Nested excess property check.
// const bad: User = {
//   name: "Ada",
//   address: { city: "London", zip: "SW1A", country: "UK" },
// };
// Error: 'country' does not exist in type 'Address'.
```

**Expected Output:**
```
London
```

**Why this output occurs.** The `address` property is checked against `Address`. A direct nested literal with an extra `country` property triggers an excess property error at the nested level.

---

### Real-World Cases

- **User profiles**: `{ name: string; address: { city: string; zip: string } }`.
- **API responses**: `{ data: { items: [{ id: string }] } }`.
- **App configuration**: `{ server: { port: number; host: string } }`.

---

## 6. Optional Properties

### Definitions

**Core Definition.** An **optional property** is an object property marked with `?` after its name, indicating that the property **may be omitted**. When present, it must satisfy its declared type.

**Technical Definition.** The syntax `propertyName?: Type` makes the property optional. Reading an optional property yields `Type | undefined`. Under `exactOptionalPropertyTypes`, the property may be **omitted** or be `Type`, but **not** explicitly `undefined`.

**Beginner-Friendly Explanation.** An optional property is **a property that might not be there**. `email?: string` means the object can have an email string, or it can omit it entirely.

### Purposes

- **To model properties that are not always present**.
- **To support option bags** where only some fields are supplied.
- **To reduce the burden on object creators**.

### Syntax Rules and Structure

**General syntax:**

```
interface InterfaceName {
  propertyName?: Type;
}
```

**Component breakdown:**

- `propertyName?` — the optional property.
- `: Type` — the property’s type when present.

**Syntax rules.**

- `?` follows the property name immediately.
- Optional properties may be combined with `readonly`.
- Reading yields `Type | undefined`.
- Under `exactOptionalPropertyTypes`, explicit `undefined` is rejected unless `undefined` is in the type.

**Constraints and limitations.**

- Optional properties do not enforce that at least one of a group is present.
- Consumers must check for `undefined`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Optional property

```typescript
// example_optional_property.ts
interface User {
  name: string;
  age?: number;
}

const user1: User = { name: "Ada" };
const user2: User = { name: "Grace", age: 85 };

console.log(user1.age); // Expected: undefined
console.log(user2.age); // Expected: 85

// user1.age = "old"; // Error: string not assignable to number | undefined.
```

**Expected Output:**
```
undefined
85
```

**Why this output occurs.** `age` is optional, so `user1` may omit it; reading yields `undefined`. `user2` provides `85`, so reading yields `85`.

---

### Real-World Cases

- **React props**: `onClick?: () => void`.
- **API parameters**: `page?: number`.
- **Configuration**: `timeout?: number`.

---

## 7. Readonly Properties

### Definitions

**Core Definition.** A **readonly property** is an object property marked with `readonly` that **cannot be reassigned** after initialization.

**Technical Definition.** The `readonly` modifier prevents assignment to the property outside its declaration or constructor (for class fields). It is **compile-time only** and **shallow**: it prevents reassignment of the property binding, not mutation of the referenced value. `readonly` is ignored during structural compatibility checks.

**Beginner-Friendly Explanation.** A readonly property is **data that can be set once but not changed afterward**. `readonly id: string` means the object must provide an `id`, but you cannot assign a new value to it.

### Purposes

- **To enforce immutability of identity** (IDs, creation timestamps).
- **To document intent** that a property should not change.
- **To prevent accidental reassignment** of configuration values.

### Syntax Rules and Structure

**General syntax:**

```
interface InterfaceName {
  readonly propertyName: Type;
}
```

**Component breakdown:**

- `readonly` — the modifier.
- `propertyName` — the property name.
- `: Type` — the property’s type.

**Syntax rules.**

- `readonly` may be combined with `?`: `readonly prop?: Type`.
- `readonly` is **shallow**: `readonly arr: number[]` still allows `arr.push(1)`.
- `readonly` does not participate in structural compatibility checks.

**Constraints and limitations.**

- `readonly` is erased at runtime.
- `readonly` is shallow; nested objects remain mutable.

### Annotated Complete Step by Step Code Examples

#### Example 1: Readonly property

```typescript
// example_readonly_property.ts
interface Point {
  readonly x: number;
  readonly y: number;
}

const p: Point = { x: 1, y: 2 };
console.log(p.x); // Expected: 1

// p.x = 10; // Error: Cannot assign to 'x' because it is a read-only property.
```

**Expected Output:**
```
1
```

**Why this output occurs.** `x` and `y` are readonly. The object literal initializes them; subsequent assignment is rejected at compile time.

---

### Real-World Cases

- **Entity IDs**: `readonly id: string`.
- **Configuration values**: `readonly apiUrl: string`.
- **Immutable value objects**: `readonly x`, `readonly y`.

---

## 8. Index Signatures

### Definitions

**Core Definition.** An **index signature** declares that an object type may be indexed with keys of a specific type (`string`, `number`, or `symbol`), returning values of a declared type. It allows **dynamic property names** while retaining type safety.

**Technical Definition.** The syntax `[key: string]: T` permits any string key with value type `T`. Explicit properties must be assignable to `T`. Since TypeScript 4.4, template literal types may be used as key types. Under `noUncheckedIndexedAccess`, indexed access returns `T | undefined`.

**Beginner-Friendly Explanation.** An index signature is **a rule that says “this object is a dictionary.”** You can use any string key, and every key gives you a value of the declared type.

### Purposes

- **To type dynamically keyed objects** without falling back to `any`.
- **To model dictionaries, caches, and configuration maps**.
- **To combine known properties with arbitrary additional properties**.

### Syntax Rules and Structure

**General syntax:**

```
interface TypeName {
  [key: KeyType]: ValueType;
}
```

**Component breakdown:**

- `[key: KeyType]` — the index signature.
- `: ValueType` — the value type for all keys.

**Syntax rules.**

- Key types may be `string`, `number`, `symbol`, or template literals.
- Explicit properties must be assignable to `ValueType`.
- Only one index signature per key type is allowed.
- `readonly` may be applied.

**Constraints and limitations.**

- Same value type for all keys.
- No required keys.
- No typo protection.

### Annotated Complete Step by Step Code Examples

#### Example 1: String index signature

```typescript
// example_index_signature.ts
interface StringMap {
  [key: string]: string;
}

const map: StringMap = { hello: "world", foo: "bar" };
console.log(map["hello"]); // Expected: world

// map["count"] = 42; // Error: number not assignable to string.
```

**Expected Output:**
```
world
```

**Why this output occurs.** The index signature permits any string key with a string value. Assigning a number is rejected.

---

### Real-World Cases

- **Environment variables**: `{ [key: string]: string | undefined }`.
- **Feature flags**: `{ [flag: string]: boolean }`.
- **CSS custom properties**: `` { [key: `--${string}`]: string } ``.

---

## 9. Object Type Best Practices

### Definitions

**Core Definition.** **Object type best practices** are guidelines for designing and using object types in TypeScript in ways that maximize **type safety, clarity, and compiler performance**, while avoiding pitfalls such as over-widening, excess property bypasses, and unnecessary index signatures.

**Technical Definition.** Best practices center on **using `interface` for object shapes** (better error messages, declaration merging, caching), **using `type` for unions and utility types**, **using `satisfies` for validation without widening**, **preferring `as const` for literal preservation**, **avoiding index signatures when keys are known**, **using `readonly` for immutable identity**, and **enabling `exactOptionalPropertyTypes`** to distinguish missing from `undefined`.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep object types clean and safe. The big rules are: use interfaces for object shapes; use `satisfies` to validate without losing precision; do not use index signatures when you know the property names; and use `readonly` for things that should not change.

### Purposes

- **To maximize type safety** and error message clarity.
- **To preserve literal precision** where it matters.
- **To avoid performance pitfalls** from large unions and unnecessary intersections.
- **To keep object types maintainable and consistent**.

### Syntax Rules and Structure

Best practices are conventions that interact with object type syntax. The core rules are:

1. **Prefer `interface` for object shapes.** Interfaces produce clearer errors, support declaration merging, and are cached by the compiler. Use `type` for unions, intersections, and utility types.
2. **Use `satisfies` for validation without widening.** `satisfies` validates against a type while preserving the inferred literal types.
3. **Use `as const` for literal preservation.** `as const` makes properties readonly and preserves literal types.
4. **Avoid index signatures when keys are known.** Use interfaces or `Record` with literal unions instead.
5. **Use `readonly` for immutable identity.** Mark properties that should not change after creation.
6. **Enable `exactOptionalPropertyTypes`.** Distinguish missing from explicitly `undefined`.
7. **Beware of excess property checks.** Direct object literals are checked; intermediate variables bypass the check. Use `satisfies` to restore it.

**Component breakdown (object type decision tree):**

```
Are you defining an object shape used as a public contract?
  ├── Yes → Use interface.
  └── No → Are you combining types or defining a union?
           ├── Yes → Use type.
           └── No → Is the object used for validation without widening?
                    ├── Yes → Use satisfies.
                    └── No → Use interface or type as appropriate.
```

**Constraints and limitations.**

- `as const` makes properties readonly, which may not be desired.
- `satisfies` does not make properties readonly.
- Excess property checks only apply to fresh object literals.

### Annotated Complete Step by Step Code Examples

#### Example 1: `satisfies` for validation without widening

```typescript
// example_best_satisfies.ts
interface Config {
  apiUrl: string;
  timeout: number;
}

// ❌ Annotation widens literals.
const wide: Config = { apiUrl: "/api", timeout: 5000 };
// wide.apiUrl: string

// ✅ `satisfies` preserves literals.
const narrow = { apiUrl: "/api", timeout: 5000 } satisfies Config;
// narrow.apiUrl: "/api"

console.log(narrow.apiUrl); // Expected: /api
```

**Expected Output:**
```
/api
```

**Why this output occurs.** The annotation widens `"/api"` to `string`. `satisfies` validates the shape while preserving the literal type `"/api"`.

---

#### Example 2: Avoiding index signatures when keys are known

```typescript
// example_best_no_index.ts
// ❌ Index signature: no typo protection.
// interface Config { [key: string]: string }

// ✅ Interface with known keys.
interface Config {
  apiUrl: string;
  timeout: string;
}

const config: Config = { apiUrl: "/api", timeout: "5000" };
console.log(config.apiUrl); // Expected: /api

// config.apiUl; // Error: Property 'apiUl' does not exist.
```

**Expected Output:**
```
/api
```

**Why this output occurs.** The interface with known keys catches typos like `apiUl`, while an index signature would allow any string key.

---

### Real-World Cases

- **Configuration objects**: `satisfies` for validation without widening.
- **API contracts**: interfaces for public object shapes.
- **Immutable entities**: `readonly` for IDs and timestamps.
- **Dynamic maps**: index signatures only when keys are genuinely unknown.

---

## References

- TypeScript Handbook — Object Types - https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Handbook — Mapped Types (Index Signatures and Modifiers) - https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- TypeScript 4.9 Release Notes — `satisfies` Operator - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html
- TypeScript 3.4 Release Notes — `const` Assertions - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html
- Exploring JS — Object Types (Structural Typing, Members, Excess Property Checks) - https://exploringjs.com/ts/downloads/exploring-ts-screen-preview.pdf
- TSTyche — Type Inference and Widening - https://tstyche.org/explanations/type-inference
- v0 — Understand Excess Property Checks on Object Literals - https://github.com/shipshitdev/v0/blob/master/.agents/skills/typescript-refactor/references/quirk-excess-property-checks.md
- pproenca/dot-skills — Excess Property Checks - https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/typescript-refactor/references/quirk-excess-property-checks.md
- GeeksforGeeks — TypeScript Object Type Property Modifiers - https://origin.geeksforgeeks.org/typescript-object-type-property-modifiers/
- Keploy — TypeScript Interface: Complete Guide - https://keploy.io/blog/community/typescript-interface-complete-guide-for-modern-developers
- Microsoft/TypeScript Issue #63515 — Excess Property Check on Nested Object Literals - https://github.com/microsoft/TypeScript/issues/63515