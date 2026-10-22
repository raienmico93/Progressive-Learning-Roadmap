# Property Modifiers in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Property modifiers** are TypeScript type-level operators—`?` for optionality and `readonly` for immutability—that refine the **behavior** of individual properties within object types, interfaces, type aliases, and classes. They specify **whether a property must be present** and **whether it may be reassigned after initialization**, without altering the property's underlying type.

**Technical Definition.** In TypeScript’s type grammar, a property signature may carry two independent modifiers: `propertyName?: Type` marks the property as **optional** (the containing object may omit it), and `readonly propertyName: Type` marks the property as **non-assignable** after initialization. These modifiers are **erased** during compilation and have **no runtime effect**; they exist exclusively for compile-time checking and tooling. The `?` modifier interacts with the `exactOptionalPropertyTypes` compiler flag to control whether an optional property may be explicitly assigned `undefined`. The `readonly` modifier is **shallow**: it prevents reassignment of the property binding, not mutation of the referenced value.

**Beginner-Friendly Explanation.** Property modifiers are **small labels you attach to properties** to control how they can be used. The `?` means “this property is optional—it might not be there.” The `readonly` means “this property can be set once but not changed afterward.” They help you describe the **real-world behavior** of your data more accurately and let TypeScript catch mistakes when you forget to check for missing properties or accidentally try to change something that should stay fixed.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Compile-time only** | Both `?` and `readonly` are erased at runtime; they emit no JavaScript. |
| **`?` marks optionality** | An optional property may be omitted entirely; reading it yields `Type \| undefined`. |
| **`readonly` prevents reassignment** | A readonly property may be assigned only at declaration or in the constructor (for classes). |
| **`readonly` is shallow** | It protects the property binding, not the internal contents of an object or array. |
| **`?` interacts with `exactOptionalPropertyTypes`** | When enabled, `prop?: T` means the property may be **omitted** or be `T`, but **not** explicitly `undefined`. |
| **Modifiers are independent** | A property may be both optional and readonly: `readonly prop?: Type`. |
| **Structural compatibility ignores `readonly`** | TypeScript does not consider `readonly` when checking whether two object types are compatible. |

---

### Prerequisites

- **TypeScript basics**: interfaces, type aliases, and object types.
- **JavaScript objects**: object literals, properties, and property access.
- **The distinction between the type space and the value space**.
- **Compiler flags**: `strict`, `exactOptionalPropertyTypes`.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Object type design** in interfaces and type aliases.
- **Class field declarations** and constructor parameter properties.
- **Utility types**: `Partial<T>`, `Required<T>`, and `Readonly<T>` are mapped-type implementations of `?` and `readonly`.
- **Immutable data modeling** and functional programming.
- **API contract design** where optionality and immutability carry semantic meaning.

---

## Core Concepts / Features

---

## 1. Optional Properties (`?`)

### Definitions

**Core Definition.** An **optional property** is a property marked with `?` after its name, indicating that the containing object **may omit** the property. When present, the property must satisfy its declared type.

**Technical Definition.** The syntax `propertyName?: Type` declares an optional property. The property’s type when accessed is `Type | undefined` (by default). Objects are not required to provide the property. The compiler permits omission but rejects assignment of an incompatible value when the property is present. Under `exactOptionalPropertyTypes`, the semantics are stricter: the property may be **omitted** or be of type `Type`, but it may **not** be explicitly assigned `undefined` unless `undefined` is part of `Type`.

**Beginner-Friendly Explanation.** An optional property is **a piece of data that an object may or may not have**. `email?: string` means an object can have an `email` string, or it can leave it out entirely. It is TypeScript’s way of saying “this property is not always there.”

### Purposes

- **To model properties that are not always present** in conforming objects.
- **To support “option bag” patterns** where only some configuration fields are supplied.
- **To reduce the burden on object creators** by not requiring every field.
- **To represent partial or conditional data** in API and configuration models.

### Syntax Rules and Structure

**General syntax:**

```
interface InterfaceName {
  propertyName?: Type;
}
```

**Component breakdown:**

- `propertyName` — the property name (identifier, string literal, or numeric literal).
- `?` — the optionality modifier, placed **immediately after** the property name.
- `: Type` — the property’s type when present.

**Combined with `readonly`:**

```
readonly propertyName?: Type;
```

**Syntax rules.**

- The `?` must follow the property name directly: `name?: string`, not `name ?: string`.
- Optional properties may be combined with `readonly`.
- Optional properties may appear in interfaces, type aliases, and class field declarations.
- Reading an optional property yields `Type | undefined` under default settings.
- Under `exactOptionalPropertyTypes`, explicit `undefined` assignment is rejected unless `undefined` is included in the type.

**Constraints and limitations.**

- Optional properties **do not enforce that at least one** of a group is present; for that, use discriminated unions.
- Optional properties can hide errors if consumers forget to check for `undefined`.
- Under `exactOptionalPropertyTypes`, assigning `undefined` to an optional property is a compile-time error unless the type includes `undefined`.
- The `?` modifier does **not** affect runtime behavior; it is purely a compile-time construct.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic optional property

```typescript
// example_optional_basic.ts
// Step 1: Declare an interface with an optional property.
interface User {
  name: string;
  age?: number; // age may be omitted
}

// Step 2: Create objects with and without the optional property.
const user1: User = { name: "Ada" };
const user2: User = { name: "Grace", age: 85 };

// Step 3: Access the optional property.
console.log(user1.age); // Expected: undefined
console.log(user2.age); // Expected: 85

// user1.age = "old"; // Error: string not assignable to number.
```

**Setup.** Compile with `npx tsc example_optional_basic.ts --strict`.

**Expected Output:**
```
undefined
85
```

**Why this output occurs.** `age` is optional, so `user1` may omit it; reading `user1.age` yields `undefined`. `user2` provides `85`, so reading it yields `85`. The compiler rejects assigning a `string` to `age` because its type is `number | undefined`.

---

#### Example 2: Optional property with `exactOptionalPropertyTypes`

```typescript
// example_optional_exact.ts
// Compile with: npx tsc example_optional_exact.ts --strict --exactOptionalPropertyTypes
interface Settings {
  theme?: "dark" | "light";
}

const s1: Settings = {}; // ✅ Omitted — allowed.
const s2: Settings = { theme: "dark" }; // ✅ Present with valid value.

// ❌ Explicit undefined is rejected under exactOptionalPropertyTypes.
// const s3: Settings = { theme: undefined };
// Error: Type 'undefined' is not assignable to type '"dark" | "light"'.
```

**Expected Output:**
```
(no runtime output; compile-time behavior demonstrated)
```

**Why this output occurs.** With `exactOptionalPropertyTypes` enabled, `theme?: "dark" | "light"` means the property may be **absent** or be `"dark"` or `"light"`, but **not** explicitly `undefined`. This separates the concepts of “missing” and “present but undefined,” which have different runtime behaviors (e.g., `"theme" in settings`).

---

#### Example 3: Optional method property

```typescript
// example_optional_method.ts
interface Logger {
  log?: (message: string) => void;
}

function safeLog(logger: Logger, message: string) {
  // Must check for optionality before calling.
  logger.log?.(message);
}

safeLog({ log: (m) => console.log(m) }, "Hello"); // Expected: Hello
safeLog({}, "Ignored"); // No output — log is undefined.
```

**Expected Output:**
```
Hello
```

**Why this output occurs.** `log` is an optional property whose type is a function. The optional chaining operator `?.` safely invokes `log` only if it is present. The second call produces no output because `log` is `undefined`.

---

### Real-World Cases

- **React props**: optional callbacks, styles, and configuration fields (`onClick?: () => void`).
- **API request parameters**: optional query parameters and headers (`page?: number`).
- **Configuration objects**: optional overrides with sensible defaults (`timeout?: number`).
- **Option bags**: functions that accept only the options the caller needs.

---

## 2. Readonly Properties (`readonly`)

### Definitions

**Core Definition.** A **readonly property** is a property marked with the `readonly` modifier, indicating that the property **cannot be reassigned** after its initial assignment. Reading is permitted; writing is rejected at compile time.

**Technical Definition.** The `readonly` modifier prevents assignment to the property outside of its declaration or the constructor (for class fields). It is **compile-time only** and **shallow**: it prevents reassignment of the property binding but does not prevent mutation of the referenced object or array. `readonly` is ignored during structural compatibility checks, meaning a readonly object is assignable to a mutable type and vice versa. It can be combined with `?` for optional readonly properties.

**Beginner-Friendly Explanation.** A readonly property is **data that can be set once but not changed afterward**. `readonly id: string` means the object must provide an `id`, but once the object exists, you cannot assign a new value to it. It is TypeScript’s way of saying “this should not be modified.”

### Purposes

- **To enforce immutability of identity** (IDs, creation timestamps).
- **To document intent** that a property should not change after creation.
- **To prevent accidental reassignment** of configuration or reference values.
- **To support immutable data modeling** in functional and reactive architectures.
- **To signal safe-to-share data** that will not be mutated by consumers.

### Syntax Rules and Structure

**General syntax:**

```
interface InterfaceName {
  readonly propertyName: Type;
}
```

**Component breakdown:**

- `readonly` — the immutability modifier.
- `propertyName` — the property name.
- `: Type` — the property’s type.

**Class field syntax:**

```
class ClassName {
  readonly propertyName: Type = initializer;
}
```

**Syntax rules.**

- `readonly` may be combined with `?`: `readonly prop?: Type`.
- `readonly` may be applied to class fields, interface properties, and index signatures.
- In classes, a readonly field may be assigned at declaration or in the constructor.
- `readonly` is **shallow**: `readonly arr: number[]` still allows `arr.push(1)`.
- For read-only arrays, use `readonly T[]` or `ReadonlyArray<T>`.
- `readonly` does **not** participate in structural compatibility checks.

**Constraints and limitations.**

- `readonly` is erased at runtime; it does not enforce immutability in JavaScript.
- `readonly` is **shallow**; nested objects remain mutable.
- `readonly` cannot be applied to methods.
- Google’s TypeScript Style Guide recommends marking properties that are never reassigned outside the constructor as `readonly`, but notes they “need not be deeply immutable”.

### Annotated Complete Step by Step Code Examples

#### Example 1: Readonly property prevents reassignment

```typescript
// example_readonly_basic.ts
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

#### Example 2: Readonly is shallow — array mutation is still allowed

```typescript
// example_readonly_shallow.ts
interface Container {
  readonly items: string[];
}

const c: Container = { items: [] };
c.items.push("a"); // Allowed: mutating the array, not reassigning the property.
console.log(c.items); // Expected: [ 'a' ]

// c.items = []; // Error: Cannot assign to 'items' because it is a read-only property.
```

**Expected Output:**
```
[ 'a' ]
```

**Why this output occurs.** `readonly` prevents reassignment of `items`, but the array itself remains mutable. For true read-only arrays, the type should be `readonly string[]` or `ReadonlyArray<string>`.

---

#### Example 3: Readonly class field

```typescript
// example_readonly_class.ts
class User {
  readonly id: string;
  name: string;

  constructor(id: string, name: string) {
    this.id = id; // ✅ Allowed: assignment in constructor.
    this.name = name;
  }
}

const user = new User("u1", "Ada");
console.log(user.id); // Expected: u1

// user.id = "u2"; // Error: Cannot assign to 'id' because it is a read-only property.
```

**Expected Output:**
```
u1
```

**Why this output occurs.** `id` is readonly and assigned once in the constructor. Subsequent assignment is rejected. `name` is mutable and can be changed.

---

### Real-World Cases

- **Entity IDs**: `readonly id: string` assigned once.
- **Configuration values**: `readonly apiUrl: string`.
- **Immutable value objects**: `readonly x`, `readonly y` in `Point`.
- **React props and hook returns**: marking data as `readonly` documents intent and prevents accidental mutation.

---

## 3. Property Modifier Best Practices

### Definitions

**Core Definition.** **Property modifier best practices** are guidelines for applying `?` and `readonly` in ways that maximize **type safety, clarity, and maintainability**, while avoiding common pitfalls such as overusing `readonly` on every property, using `?` when `| undefined` is intended, and misunderstanding the shallow nature of `readonly`.

**Technical Definition.** Best practices center on **intentional use**: `?` should model genuine optionality (the property may be absent), not nullable data (use `| null` or `| undefined` for that). `readonly` should be applied to properties that are **never reassigned outside the constructor**, as recommended by the Google TypeScript Style Guide. `readonly` should be used primarily for **arrays, maps, and matching external interfaces**, and only when mutation is a realistic footgun—stop prefixing every property with `readonly`. `exactOptionalPropertyTypes` should be enabled to distinguish missing from explicitly `undefined`.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep property modifiers from becoming noise. The big rules are: use `?` only when the property can genuinely be absent; use `readonly` for things that should never change after creation, especially arrays; and remember that `readonly` does not make the contents immutable—it only stops reassignment of the property itself.

### Purposes

- **To model data accurately** with optionality and immutability that reflect real-world constraints.
- **To prevent accidental mutation** of values that should not change.
- **To avoid the noise** of `readonly` on every property when it adds little value.
- **To distinguish missing from `undefined`** with `exactOptionalPropertyTypes`.
- **To improve code clarity and reviewability** by making intent explicit.

### Syntax Rules and Structure

Best practices are conventions that interact with modifier syntax. The core rules are:

1. **Use `?` for genuinely optional data.** If a property can be absent, mark it `?`. If it can be present but `undefined`, include `| undefined` explicitly (or enable `exactOptionalPropertyTypes` to enforce the distinction).
2. **Enable `exactOptionalPropertyTypes`.** This flag makes `prop?: T` mean “absent or `T`” rather than “absent, `T`, or `undefined`,” preventing accidental explicit `undefined` assignments.
3. **Use `readonly` for properties never reassigned outside the constructor.** Google’s Style Guide recommends this and notes deep immutability is not required.
4. **Use `readonly` primarily for arrays and maps.** `readonly` is most valuable when mutation is a realistic footgun; do not prefix every property with `readonly`.
5. **Remember `readonly` is shallow.** For deep immutability, use `Readonly<T>` with a recursive `DeepReadonly` type or immutable data structures.
6. **Do not use `readonly` to signal deep immutability.** It only prevents reassignment of the binding, not mutation of the contents.
7. **Combine `?` and `readonly` when both apply.** `readonly id?: string` is valid for an optional, immutable ID.
8. **Use mapped types for bulk application.** `Partial<T>` makes all properties optional; `Readonly<T>` makes all properties readonly. Apply these deliberately, not as a default.

**Component breakdown (modifier decision tree):**

```
Should this property always be present?
  ├── No → Use ? (optional property).
  └── Yes → Is it reassigned outside the constructor?
            ├── No → Consider readonly (especially for arrays/maps).
            └── Yes → Leave it mutable.
```

**Constraints and limitations.**

- `exactOptionalPropertyTypes` may require code changes if existing code assigns `undefined` to optional properties.
- `readonly` on every property is considered noise by some style guides and adds little safety for primitives.
- `readonly` does not deep-freeze objects; nested mutation remains possible.
- Optional properties shift `undefined` handling to consumers; ensure this is intentional.

### Annotated Complete Step by Step Code Examples

#### Example 1: `?` for genuine optionality, `| undefined` for explicit undefined

```typescript
// example_best_optionality.ts
// ✅ Optional: the property may be omitted.
interface User {
  name: string;
  nickname?: string;
}

// ✅ Explicit undefined: the property may be present but undefined.
interface ExplicitUser {
  name: string;
  nickname: string | undefined;
}

const u1: User = { name: "Ada" }; // ✅ Omitted.
const u2: ExplicitUser = { name: "Ada", nickname: undefined }; // ✅ Present, undefined.

console.log(u1.nickname); // Expected: undefined
console.log(u2.nickname); // Expected: undefined
```

**Expected Output:**
```
undefined
undefined
```

**Why this output occurs.** `User.nickname` is optional and omitted. `ExplicitUser.nickname` is required but allows `undefined`. The distinction matters for runtime checks like `"nickname" in obj`, which behaves differently for missing vs. `undefined` values.

---

#### Example 2: `readonly` for arrays (high value)

```typescript
// example_best_readonly_array.ts
// ✅ readonly on an array property prevents reassignment.
interface Config {
  readonly hosts: string[];
}

const config: Config = { hosts: ["a", "b"] };
config.hosts.push("c"); // Allowed: mutating the array.
console.log(config.hosts); // Expected: [ 'a', 'b', 'c' ]

// config.hosts = []; // Error: Cannot assign to 'hosts'.
```

**Expected Output:**
```
[ 'a', 'b', 'c' ]
```

**Why this output occurs.** `readonly` prevents reassigning `hosts`, but the array remains mutable. For true read-only arrays, use `readonly string[]` or `ReadonlyArray<string>`. This is where `readonly` provides the most value—preventing the “swap the whole array” footgun.

---

#### Example 3: Avoiding `readonly` noise on primitives

```typescript
// example_best_readonly_noise.ts
// ❌ Readonly on every primitive adds little safety.
interface Noisy {
  readonly id: number;
  readonly name: string;
  readonly active: boolean;
}

// ✅ Use readonly selectively — especially on arrays and objects.
interface Focused {
  readonly id: number; // Identity — should not change.
  name: string;        // Mutable display name.
  tags: readonly string[]; // Read-only array — mutation is a footgun.
}

const focused: Focused = { id: 1, name: "Ada", tags: ["pioneer"] };
console.log(focused.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `readonly` on primitives like `id` is useful for identity, but prefixing every primitive with `readonly` adds visual noise without meaningful safety. `readonly` on the `tags` array is high-value because it prevents reassignment of the collection.

---

### Real-World Cases

- **Entity models**: `readonly id` for identity; mutable fields for editable data.
- **Configuration objects**: `readonly` on arrays of hosts, allowed origins, or feature flags.
- **API responses**: `readonly` on fields that should not be mutated by consumers.
- **Strict optionality**: enabling `exactOptionalPropertyTypes` in libraries that need to distinguish absent from `undefined`.

---

## References

- TypeScript Handbook — Object Types: Property Modifiers (Optional and Readonly) - https://www.typescriptlang.org/docs/handbook/2/objects.html#property-modifiers
- TSConfig Reference — `exactOptionalPropertyTypes` - https://www.typescriptlang.org/tsconfig/exactOptionalPropertyTypes.html
- Google TypeScript Style Guide — Use `readonly` - https://google.github.io/styleguide/tsguide.html#use-readonly
- Google TypeScript Style Guide — Visibility and Parameter Properties - https://google.github.io/styleguide/tsguide.html
- TypeScript Documentation — Classes: `readonly` Modifier - https://www.typescriptlang.org/docs/handbook/2/classes.html#readonly
- Effective TypeScript — Item 14: Use `readonly` to Avoid Errors Associated with Mutation - https://github.com/NeverSight/skills_feed/blob/main/data/skills-md/marius-townhouse/effective-typescript-skills/readonly-shallow/SKILL.md
- “`readonly` Is Mostly Noise (Except for Arrays)” — Practical TypeScript Guidance - https://github.com/ts-essentials/ts-essentials/blob/main/docs/readonly-is-mostly-noise.md
- Convex — Readonly in TypeScript: Best Practices - https://www.convex.dev/typescript/core-concepts/object-oriented-programming/typescript-readonly
- SonarSource — Common TypeScript Issues Nº 5: Optional Property Declarations - https://www.sonarsource.com/blog/typescript-optional-property-declarations/
- TypeScript 3.7 Release Notes — Optional Chaining (`?.`) - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html