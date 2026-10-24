# Enums in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A TypeScript **enum** is a named set of constant values that allows developers to define a collection of related members under a single identifier, documenting intent and enforcing a finite set of values. Unlike most TypeScript features, enums are one of the few constructs that **emit actual JavaScript objects at runtime**, making them a non-erasable addition to the language.

**Technical Definition.** An enum declaration introduces both a **type** and a **value** into the program. The type is the union of the enum's member types; the value is a runtime object containing forward mappings (name → value) and, for numeric members, **reverse mappings** (value → name). TypeScript supports numeric, string, heterogeneous, and const enums. Numeric enums auto-increment; string enums require explicit initialization. `const` enums are fully inlined at compile time and emit no runtime object, but they conflict with `isolatedModules` and single-file transpilers.

**Beginner-Friendly Explanation.** An enum is **a way to give names to a fixed set of related values**. Instead of scattering magic strings or numbers throughout your code, you define them once in an enum. For example, `enum Direction { Up, Down, Left, Right }` lets you write `Direction.Up` instead of `0`. Enums make your code more readable and help TypeScript catch invalid values—but they also generate JavaScript code at runtime, which is unusual for TypeScript features and has trade-offs you should understand.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Named constants** | Enums group related constants under a single identifier. |
| **Runtime object** | Non-const enums emit a JavaScript object at runtime—unlike most TypeScript features. |
| **Numeric auto-increment** | Numeric members without initializers auto-increment from `0`. |
| **Reverse mappings** | Numeric enums support value→name lookup at runtime; string enums do not. |
| **Const enums** | `const enum` inlines values and emits no runtime object, but conflicts with `isolatedModules`. |
| **Heterogeneous support** | Enums may mix string and numeric members, though this is discouraged. |
| **Nominal typing** | Enums are nominal—`number` is implicitly assignable to numeric enums, causing unsoundness. |
| **Non-erasable** | Enums generate runtime code, unlike interfaces and type aliases. |

---

### Prerequisites

- **TypeScript basics**: type annotations, type aliases, and union types.
- **JavaScript objects**: property access and runtime objects.
- **Compiler flags**: `isolatedModules`, `preserveConstEnums`.
- **Module systems**: CommonJS/ES modules and single-file transpilers (Babel, esbuild, SWC).
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Union types and literal types** — the modern alternative to enums.
- **`as const` objects** — runtime-preserving alternatives with literal narrowing.
- **Runtime code generation** — enums emit JavaScript, unlike most TypeScript features.
- **Tree-shaking and bundle size** — enums are not tree-shakeable.
- **Nominal vs. structural typing** — enums are nominal, while most TypeScript types are structural.

---

## Core Concepts / Features

---

## 1. Enums

### Definitions

**Core Definition.** An **enum** (enumeration) is a named collection of constant values that allows developers to define a set of related members under a single identifier, making code more readable and documenting intent.

**Technical Definition.** An enum declaration introduces a **type** (the union of its members' types) and a **value** (a runtime object with forward and, for numeric members, reverse mappings). Enums are **nominal**: a numeric enum is not interchangeable with a plain `number` at the type level, though TypeScript unsoundly allows `number` to be assigned to a numeric enum type. Enums are **non-erasable**—they emit JavaScript objects, making them unique among TypeScript features.

**Beginner-Friendly Explanation.** An enum is **a set of named values grouped together**. Instead of using raw numbers or strings, you give each value a descriptive name. For example, `enum Status { Active, Inactive }` lets you write `Status.Active` instead of `0`. It makes code self-documenting and helps prevent typos.

### Purposes

- **To document intent** by giving descriptive names to a finite set of values.
- **To enforce a finite set of values** at the type level.
- **To organize related constants** under a single identifier.
- **To provide runtime lookup** through forward mappings (name → value) and reverse mappings (value → name, numeric only).
- **To serve as a stable identifier** across a codebase for values that need runtime representation.

### Syntax Rules and Structure

**General syntax:**

```typescript
enum EnumName {
  Member1,
  Member2 = value,
  Member3,
}
```

**Component breakdown:**

- `enum` — declaration keyword.
- `EnumName` — the enum identifier (PascalCase convention).
- `Member1`, `Member2` — enum members (PascalCase convention).
- `= value` — optional initializer for each member.

**Syntax rules.**

- Enum members are accessed as `EnumName.Member`.
- Members may have explicit initializers or be auto-incremented (numeric only).
- Enums are both a **type** and a **value**.
- Enums may be `const`, `declare`, or `export`ed.
- Enums may be merged across declarations (declaration merging).

**Constraints and limitations.**

- **Non-erasable**: enums emit runtime code, unlike most TypeScript features.
- **Not tree-shakeable**: enum objects are not removed by tree-shaking.
- **Nominal typing**: numeric enums unsoundly accept any `number`.
- **Const enum issues**: const enums conflict with `isolatedModules` and single-file transpilers.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic enum

```typescript
// example_enum_basic.ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

console.log(Direction.Up);    // Expected: 0
console.log(Direction.Right); // Expected: 3

// Reverse mapping works for numeric enums.
console.log(Direction[0]);    // Expected: Up
```

**Setup.** Compile with `npx tsc example_enum_basic.ts --strict`.

**Expected Output:**
```
0
3
Up
```

**Why this output occurs.** The enum members auto-increment from `0`. `Direction.Up` is `0`, `Direction.Right` is `3`. The reverse mapping `Direction[0]` returns `"Up"` because numeric enums generate value→name mappings at runtime.

---

### Real-World Cases

- **Status codes**: `enum HttpStatus { OK = 200, NotFound = 404 }`.
- **User roles**: `enum Role { Admin, Editor, Viewer }`.
- **Configuration flags**: `enum LogLevel { Debug, Info, Warn, Error }`.

---

## 2. `enum` Keyword

### Definitions

**Core Definition.** The **`enum` keyword** is the syntactic token that introduces an enumeration declaration. It creates a named set of constants that exists both as a type and as a runtime value.

**Technical Definition.** The `enum` keyword appears at the start of an enum declaration. It is a **value-level and type-level** construct: the enum name is added to both the value space (as a runtime object) and the type space (as a union type). Enums may be `const`, `declare`, or `export`ed.

**Beginner-Friendly Explanation.** `enum` is **the word you type to start an enum**. Everything after it—the name and the curly braces—defines the set of named constants.

### Purposes

- **To introduce a named enum** into the program.
- **To create both a type and a runtime value** in a single declaration.
- **To group related constants** under a single identifier.

### Syntax Rules and Structure

```typescript
enum EnumName {
  Member = value,
}
```

**Syntax rules.**

- The enum name must be a valid identifier.
- Members may be separated by commas.
- Enums may be exported: `export enum E { ... }`.
- Enums may be `const`: `const enum E { ... }`.
- Enums may be merged: multiple declarations with the same name combine.

**Constraints and limitations.**

- Enum names cannot be reserved words.
- Const enums are inlined and cannot be used with `isolatedModules` when ambient.

### Annotated Complete Step by Step Code Examples

#### Example 1: `enum` keyword with explicit values

```typescript
// example_enum_keyword.ts
enum Status {
  Active = 1,
  Inactive = 0,
}

console.log(Status.Active);   // Expected: 1
console.log(Status.Inactive); // Expected: 0
```

**Expected Output:**
```
1
0
```

**Why this output occurs.** The `enum` keyword introduces `Status` with explicit numeric values. The values are used as-is.

---

### Real-World Cases

- **API error codes**: `enum ErrorCode { BadRequest = 400, Unauthorized = 401 }`.
- **Feature flags**: `enum Feature { DarkMode = 1, Beta = 2 }`.

---

## 3. Numeric Enums

### Definitions

**Core Definition.** A **numeric enum** is an enum whose members are assigned **numeric values**. Members without explicit initializers auto-increment from `0` (or from the previous member's value plus one).

**Technical Definition.** Numeric enums generate **reverse mappings** at runtime: the compiled JavaScript object contains both `Name → value` and `value → Name` entries. Numeric enums are **nominal** in the type system, but TypeScript unsoundly allows `number` to be assigned to a numeric enum type, permitting invalid values to enter typed APIs. Numeric members may be **constant** (fully evaluated at compile time) or **computed** (evaluated at runtime).

**Beginner-Friendly Explanation.** A numeric enum is **an enum where the values are numbers**. If you do not give a member a value, TypeScript assigns the next number automatically. Numeric enums also let you look up a member's name from its number.

### Purposes

- **To assign numeric codes** to named constants (HTTP status codes, error codes).
- **To enable reverse lookup** from value to name for debugging and display.
- **To support bit-flag patterns** with computed bitwise members.

### Syntax Rules and Structure

```typescript
enum NumericEnum {
  A,        // 0
  B,        // 1
  C = 10,   // 10
  D,        // 11
}
```

**Syntax rules.**

- The first member without an initializer is `0`.
- Subsequent members without initializers are previous value + 1.
- Explicit numeric initializers set the value.
- Computed members may use constant expressions.
- Reverse mapping is generated only for numeric members.

**Constraints and limitations.**

- **Unsoundness**: `number` is assignable to numeric enum types, so invalid values can enter typed APIs.
- **Opaque runtime values**: numeric enum values are often meaningless on their own (e.g., `0` instead of `"Active"`).
- **Reverse mapping only for numeric members**; string enums do not have reverse mappings.

### Annotated Complete Step by Step Code Examples

#### Example 1: Auto-increment and reverse mapping

```typescript
// example_numeric_enum.ts
enum Status {
  Active,   // 0
  Inactive, // 1
  Pending,  // 2
}

console.log(Status.Active);    // Expected: 0
console.log(Status[0]);        // Expected: Active (reverse mapping)
console.log(Status.Pending);   // Expected: 2
```

**Expected Output:**
```
0
Active
2
```

**Why this output occurs.** Members auto-increment from `0`. The reverse mapping `Status[0]` returns `"Active"` because numeric enums generate value→name entries.

---

#### Example 2: Unsound `number` assignment

```typescript
// example_numeric_unsound.ts
enum Status {
  Active = 0,
  Inactive = 1,
}

function setStatus(status: Status) {
  console.log(status);
}

setStatus(999); // No error — unsound!
// Expected: 999 (invalid value accepted)
```

**Expected Output:**
```
999
```

**Why this output occurs.** TypeScript allows any `number` to be assigned to a numeric enum type, so `999` is accepted despite not being a valid `Status` member. This is a known unsoundness.

---

### Real-World Cases

- **HTTP status codes**: `enum HttpStatus { OK = 200, NotFound = 404 }`.
- **Bit flags**: `enum Permissions { Read = 1 << 0, Write = 1 << 1 }`.
- **Legacy APIs**: numeric codes required by external systems.

---

## 4. String Enums

### Definitions

**Core Definition.** A **string enum** is an enum whose members are assigned **string values**. Every member must be explicitly initialized with a string literal or another string enum member.

**Technical Definition.** String enums **serialize well**—the runtime value is a readable string that conveys meaning independent of the member name. String enums do **not** generate reverse mappings; `EnumName[EnumName.Member]` returns `undefined` for string enums. String enums are nominal, but unlike numeric enums, a plain `string` is not assignable to a string enum type (though the reverse is true).

**Beginner-Friendly Explanation.** A string enum is **an enum where the values are strings**. String enums are more readable at runtime because the value itself is meaningful. They do not support reverse lookup—you just use the value directly.

### Purposes

- **To provide meaningful, self-describing runtime values**.
- **To serialize cleanly** to JSON or APIs without numeric codes.
- **To avoid opaque numeric values** in debugging and logging.
- **To enforce a closed set of string values** at the type level.

### Syntax Rules and Structure

```typescript
enum StringEnum {
  A = "VALUE_A",
  B = "VALUE_B",
}
```

**Syntax rules.**

- Every member must be explicitly initialized with a string literal or another string enum member.
- String enums do not auto-increment.
- String enums do not generate reverse mappings.
- String members may reference other string enum members from the same or another enum.

**Constraints and limitations.**

- **No reverse mapping**—`StringEnum[StringEnum.A]` is `undefined`.
- **No auto-increment**—each member must be initialized.
- **Nominal typing**—plain strings are not assignable to string enum types.

### Annotated Complete Step by Step Code Examples

#### Example 1: String enum with reverse mapping limitation

```typescript
// example_string_enum.ts
enum Direction {
  Up = "UP",
  Down = "DOWN",
}

console.log(Direction.Up);          // Expected: UP
console.log(Direction["UP"]);       // Expected: undefined (no reverse mapping)
console.log(Direction[Direction.Up]); // Expected: undefined
```

**Expected Output:**
```
UP
undefined
undefined
```

**Why this output occurs.** String enums do not generate reverse mappings. `Direction["UP"]` attempts to look up the key `"UP"` on the enum object, which does not exist. The value is used directly.

---

### Real-World Cases

- **API statuses**: `enum ApiStatus { Success = "SUCCESS", Error = "ERROR" }`.
- **Event names**: `enum EventType { Click = "click", Focus = "focus" }`.
- **Configuration modes**: `enum Mode { Development = "development", Production = "production" }`.

---

## 5. Heterogeneous Enums

### Definitions

**Core Definition.** A **heterogeneous enum** is an enum that mixes **string and numeric members** in the same declaration. TypeScript technically permits this, though it is **strongly discouraged**.

**Technical Definition.** Heterogeneous enums allow string and numeric members to coexist. The TypeScript Handbook advises against them unless you are exploiting JavaScript's runtime behavior in a clever way. Reverse mappings are generated only for numeric members; string members do not get reverse mappings.

**Beginner-Friendly Explanation.** A heterogeneous enum **mixes strings and numbers**. It is allowed, but it is confusing and rarely useful. Most style guides recommend avoiding it.

### Purposes

- **To exploit JavaScript runtime coercion** in rare, advanced cases (the only legitimate use).
- **To understand legacy code** that uses mixed enums.

### Syntax Rules and Structure

```typescript
enum Mixed {
  No = 0,
  Yes = "YES",
}
```

**Syntax rules.**

- String members require explicit string initializers.
- Numeric members may auto-increment.
- Reverse mappings are generated only for numeric members.

**Constraints and limitations.**

- **Discouraged by the TypeScript Handbook**.
- Confusing semantics and inconsistent runtime behavior.
- Reverse mapping behavior is inconsistent across members.

### Annotated Complete Step by Step Code Examples

#### Example 1: Heterogeneous enum

```typescript
// example_heterogeneous.ts
enum BooleanLike {
  No = 0,
  Yes = "YES",
}

console.log(BooleanLike.No);    // Expected: 0
console.log(BooleanLike.Yes);   // Expected: YES
console.log(BooleanLike[0]);    // Expected: No (reverse mapping for numeric)
console.log(BooleanLike["YES"]); // Expected: undefined
```

**Expected Output:**
```
0
YES
No
undefined
```

**Why this output occurs.** Numeric members generate reverse mappings; string members do not. The mixed behavior is exactly why heterogeneous enums are discouraged.

---

### Real-World Cases

- **Legacy code migration**: understanding existing heterogeneous enums.
- **Interop with external systems**: rarely, when an external API requires mixed values.

---

## 6. Const Enums

### Definitions

**Core Definition.** A **`const` enum** is an enum declared with the `const` modifier. Its members are **fully inlined at compile time**, and the enum object is **removed entirely** from the emitted JavaScript unless `preserveConstEnums` is enabled.

**Technical Definition.** `const` enums use the `const` modifier to **remove the enum object entirely at compile time**, inlining values directly and reducing bundle overhead. However, `const` enums conflict with **`isolatedModules`** and single-file transpilers (Babel, esbuild, SWC) because those tools cannot see across files to inline values. When `isolatedModules` is enabled, referencing an ambient `const` enum member is an error. `preserveConstEnums` forces `const` enums to be emitted as regular enums, restoring runtime objects at the cost of bundle size.

**Beginner-Friendly Explanation.** A `const` enum is **an enum that disappears completely when the code is compiled**. The values are copied directly into the places where they are used. This makes the output smaller, but it can break when using modern build tools that compile files one at a time.

### Purposes

- **To reduce bundle size** by inlining values and eliminating the runtime object.
- **To avoid runtime overhead** for enums used only as compile-time constants.
- **To enable constant folding** by the compiler.

### Syntax Rules and Structure

```typescript
const enum ConstEnum {
  A = 1,
  B = 2,
}
```

**Syntax rules.**

- `const` precedes `enum`.
- Members must be constant expressions (no computed runtime values).
- `preserveConstEnums` can force emission of the runtime object.
- `isolatedModules` prohibits ambient `const` enums.

**Constraints and limitations.**

- **Incompatible with `isolatedModules` and single-file transpilers**.
- **Cannot be used with `declare`** in ambient contexts when `isolatedModules` is on.
- **No runtime object** unless `preserveConstEnums` is enabled.
- **Values are inlined**, so changing an enum value requires recompiling all consumers.

### Annotated Complete Step by Step Code Examples

#### Example 1: `const` enum inlining

```typescript
// example_const_enum.ts
const enum Direction {
  Up = 1,
  Down = 2,
}

const dir = Direction.Up;
console.log(dir); // Expected: 1
// Emitted JavaScript: const dir = 1; (no enum object)
```

**Expected Output:**
```
1
```

**Why this output occurs.** The `const` enum values are inlined at compile time. The emitted JavaScript contains `const dir = 1;` with no enum object.

---

#### Example 2: `const` enum with `preserveConstEnums`

```typescript
// example_const_preserve.ts
// Compiled with: tsc --preserveConstEnums
const enum Status {
  Active = 0,
  Inactive = 1,
}

console.log(Status.Active); // Expected: 0
```

**Expected Output:**
```
0
```

**Why this output occurs.** With `preserveConstEnums`, the enum object is emitted as a regular enum, so runtime access works. Without the flag, the value would be inlined and the object removed.

---

### Real-World Cases

- **Compile-time constants**: numeric flags that never need runtime iteration.
- **Performance-sensitive code**: eliminating enum objects from bundles.
- **Libraries**: avoiding runtime enum objects for values that consumers only use at compile time.

---

## 7. Reverse Mappings

### Definitions

**Core Definition.** **Reverse mappings** are runtime entries in a numeric enum object that map **values back to member names** (`enum[value] → "Name"`). They are generated automatically for numeric enum members only.

**Technical Definition.** When TypeScript compiles a numeric enum, it generates an object with both forward (`Name → value`) and reverse (`value → Name`) entries. String enum members do **not** receive reverse mappings—`StringEnum[StringEnum.Member]` returns `undefined`. Heterogeneous enums receive reverse mappings only for their numeric members. Sparse or negative numeric values can make reverse mapping unreliable.

**Beginner-Friendly Explanation.** Reverse mapping is **looking up a number's name in a numeric enum**. If `Status.Active` is `0`, then `Status[0]` gives you `"Active"`. String enums do not have this—there is no reverse lookup for strings.

### Purposes

- **To convert numeric enum values back to readable names** for debugging, logging, and UI display.
- **To build display labels** from enum values without a separate mapping object.
- **To introspect numeric enums at runtime**.

### Syntax Rules and Structure

```typescript
enum Status { Active, Inactive }
const name = Status[Status.Active]; // "Active"
```

**Syntax rules.**

- Reverse mapping works only for **numeric** members.
- String and heterogeneous string members return `undefined`.
- Sparse or negative values may produce unreliable results.
- `const` enums inlined at compile time cannot be reverse-mapped at runtime.

**Constraints and limitations.**

- **Numeric only**—string enums have no reverse mapping.
- **Sparse/negative values** can make mapping unreliable.
- **`const` enums** lose runtime objects, so reverse mapping is unavailable.

### Annotated Complete Step by Step Code Examples

#### Example 1: Numeric vs. string reverse mapping

```typescript
// example_reverse_mapping.ts
enum NumericStatus { Active, Inactive }
enum StringStatus { Active = "ACTIVE", Inactive = "INACTIVE" }

console.log(NumericStatus[0]);                // Expected: Active
console.log(StringStatus[StringStatus.Active]); // Expected: undefined
```

**Expected Output:**
```
Active
undefined
```

**Why this output occurs.** Numeric enums generate reverse mappings; string enums do not. The numeric lookup returns the member name, while the string lookup returns `undefined`.

---

### Real-World Cases

- **Debugging**: logging `Status[status]` to see the name instead of a number.
- **UI display**: converting a stored numeric status to a label.
- **Legacy APIs**: numeric codes that need human-readable representations.

---

## 8. Enum Alternatives

### Definitions

**Core Definition.** **Enum alternatives** are patterns that provide the type safety and named-constant benefits of enums without their runtime and tooling drawbacks. The two primary alternatives are **union types of string literals** and **`as const` objects with derived union types**.

**Technical Definition.** Union types (`type Status = "loading" | "success"`) are lightweight, erased at compile time, and support exhaustive narrowing, but they have **no runtime representation**—iteration and validation require a separate array. `as const` objects (`const Status = { Loading: "loading", Success: "success" } as const; type Status = (typeof Status)[keyof typeof Status]`) preserve **literal types** and provide a **runtime lookup object** while remaining structurally typed and tree-shakeable. Both alternatives avoid the non-erasable code, nominal unsoundness, and tooling conflicts of enums.

**Beginner-Friendly Explanation.** Enum alternatives are **other ways to define a set of named values** that avoid the problems with enums. A **union of strings** is the simplest: `"loading" | "success"`. An **`as const` object** gives you both a runtime object and precise types. Both are erased or tree-shakeable and work with all build tools.

### Purposes

- **To eliminate runtime enum objects** while preserving type safety.
- **To avoid const enum conflicts** with `isolatedModules` and single-file transpilers.
- **To enable tree-shaking and smaller bundles**.
- **To preserve literal types** for exhaustive narrowing and discriminated unions.
- **To provide a runtime lookup object** when iteration or validation is needed.

### Syntax Rules and Structure

**Union type alternative:**

```typescript
type Status = "loading" | "success" | "error";
```

**`as const` object alternative:**

```typescript
const Status = {
  Loading: "loading",
  Success: "success",
} as const;

type Status = (typeof Status)[keyof typeof Status];
```

**Component breakdown:**

- `as const` — preserves literal types and makes properties readonly.
- `typeof Status` — the type of the object.
- `keyof typeof Status` — the union of keys.
- `(typeof Status)[keyof typeof Status]` — the union of values.

**Syntax rules.**

- Union types are erased and have no runtime representation.
- `as const` objects provide both a value and a derived type.
- The type and value may share the same name.
- `as const` objects are structural, not nominal.

**Constraints and limitations.**

- Union types cannot be iterated at runtime without a separate array.
- `as const` objects do not provide reverse mappings; a separate lookup is needed.
- `as const` object properties are readonly; mutation is prohibited.

### Annotated Complete Step by Step Code Examples

#### Example 1: Union type alternative

```typescript
// example_union_alternative.ts
type Status = "loading" | "success" | "error";

function handle(status: Status): string {
  switch (status) {
    case "loading": return "Loading...";
    case "success": return "Done";
    case "error": return "Failed";
  }
}

console.log(handle("loading")); // Expected: Loading...
```

**Expected Output:**
```
Loading...
```

**Why this output occurs.** The union type is erased at compile time. The `switch` exhaustively handles all three literals. No runtime enum object is generated.

---

#### Example 2: `as const` object alternative

```typescript
// example_as_const_alternative.ts
const Status = {
  Loading: "loading",
  Success: "success",
  Error: "error",
} as const;

type Status = (typeof Status)[keyof typeof Status];

function handle(status: Status): string {
  switch (status) {
    case Status.Loading: return "Loading...";
    case Status.Success: return "Done";
    case Status.Error: return "Failed";
  }
}

console.log(handle(Status.Loading)); // Expected: Loading...
console.log(Status.Loading);          // Expected: loading
```

**Expected Output:**
```
Loading...
loading
```

**Why this output occurs.** The `as const` object preserves the literal types `"loading"`, `"success"`, and `"error"`. The derived type `Status` is the union of those values. The object is a plain JavaScript object with no enum runtime machinery.

---

### Real-World Cases

- **API statuses**: `type Status = "ok" | "error"` or `as const` object.
- **Configuration modes**: `type Mode = "development" | "production"`.
- **Exhaustive switches**: union types enable `assertNever` checks.

---

## 9. Trade-offs of Enums

### Definitions

**Core Definition.** **Enum trade-offs** are the costs and benefits of using TypeScript enums compared with alternatives, including **bundle size, runtime behavior, type safety, tree-shaking, and tooling compatibility**.

**Technical Definition.** Enums are **non-erasable** and emit runtime objects, increasing bundle size; they are **not tree-shakeable**; numeric enums are **unsound** (any `number` is assignable); string enums have **no reverse mappings**; `const` enums conflict with `isolatedModules`; and heterogeneous enums are discouraged. However, enums provide **runtime lookup objects**, **reverse mappings** (numeric only), and **stable identifiers** that can be useful for APIs that need runtime values.

**Beginner-Friendly Explanation.** Enums have **good points and bad points**. They give you a runtime object and readable names, but they make your bundle bigger, do not tree-shake, and can cause type-safety holes. Modern TypeScript development often prefers union types or `as const` objects instead.

### Purposes

- **To make an informed decision** about whether to use enums or alternatives.
- **To understand the costs** of enums in bundle size and tooling.
- **To identify cases where enums are still appropriate** (runtime lookup objects, reverse mapping).
- **To align with team and project conventions**.

### Syntax Rules and Structure

Trade-offs are not syntax; they are **decision criteria**:

| Factor | Enum | Union Type | `as const` Object |
|---|---|---|---|
| **Runtime object** | ✅ Yes (non-const) | ❌ No | ✅ Yes |
| **Reverse mapping** | ✅ Numeric only | ❌ No | ❌ No (unless built) |
| **Bundle size** | Larger | Smallest | Medium |
| **Tree-shaking** | ❌ No | N/A | ✅ Yes |
| **Nominal vs. structural** | Nominal | Structural | Structural |
| **`isolatedModules`** | ✅ (except const enums) | ✅ | ✅ |
| **Unsoundness** | Numeric enums accept any `number` | None | None |
| **Exhaustive narrowing** | ✅ | ✅ | ✅ |

**Constraints and limitations.**

- Enums are **not tree-shakeable** and increase bundle size.
- `const` enums are **incompatible with `isolatedModules`** when ambient.
- Numeric enums are **unsound**.
- String enums lack reverse mappings.

### Annotated Complete Step by Step Code Examples

#### Example 1: Bundle size comparison

```typescript
// example_tradeoff_size.ts
// Enum: emits a runtime object with forward and reverse mappings.
enum StatusEnum { Active, Inactive }

// Union type: emits no runtime code.
type StatusUnion = "active" | "inactive";

// `as const` object: emits a small runtime object, no reverse mappings.
const StatusObject = { Active: "active", Inactive: "inactive" } as const;

console.log(StatusEnum.Active); // Expected: 0
console.log(StatusObject.Active); // Expected: active
```

**Expected Output:**
```
0
active
```

**Why this output occurs.** The enum emits a full runtime object with reverse mappings. The union type emits nothing. The `as const` object emits only the forward mapping. The bundle size differences are measurable.

---

### Real-World Cases

- **Bundle-sensitive frontends**: preferring union types or `as const` to reduce size.
- **Library authoring**: avoiding `const` enums to remain compatible with `isolatedModules`.
- **Runtime-heavy APIs**: using enums when a runtime lookup object is genuinely needed.

---

## 10. Enum Best Practices

### Definitions

**Core Definition.** **Enum best practices** are guidelines for using enums safely and sparingly, preferring **union types** or **`as const` objects** for new code, avoiding **heterogeneous enums**, **const enums** (in isolated-module projects), and **numeric enums** where unsoundness matters.

**Technical Definition.** Best practices center on the **modern consensus** that TypeScript enums are often an anti-pattern: they generate non-erasable runtime code, are not tree-shakeable, and have tooling conflicts. The recommendation is to **use union types instead** when no runtime object is needed, and **`as const` objects** when a runtime object is required. If enums must be used, prefer **string enums** for readability and safety, avoid **numeric enums** for API boundaries, and **avoid `const` enums** in projects using `isolatedModules`.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to avoid enum pitfalls. The big rules are: use union types or `as const` objects instead of enums for new code; if you must use enums, use string enums; never use heterogeneous enums; and do not use `const` enums with modern build tools.

### Purposes

- **To avoid the runtime and tooling pitfalls of enums**.
- **To maximize tree-shaking and minimize bundle size**.
- **To preserve type safety** by avoiding numeric enum unsoundness.
- **To remain compatible with `isolatedModules` and single-file transpilers**.
- **To align with modern TypeScript community consensus**.

### Syntax Rules and Structure

Best practices are conventions that interact with enum syntax. The core rules are:

1. **Prefer union types for compile-time-only sets.** `type Status = "loading" | "success"` is erased, tree-shakeable, and supports exhaustive narrowing.
2. **Use `as const` objects when a runtime object is needed.** They preserve literal types and provide a runtime lookup without enum baggage.
3. **Avoid heterogeneous enums.** The TypeScript Handbook explicitly discourages them.
4. **Prefer string enums over numeric enums.** String enums serialize well and do not suffer from numeric unsoundness.
5. **Avoid `const` enums in isolated-module projects.** `const` enums conflict with `isolatedModules` and single-file transpilers.
6. **Do not use enums for dynamic values.** Enums are for finite, static sets.
7. **Enable lint rules** (`no-enums` or similar) to enforce team conventions.

**Component breakdown (enum vs. alternative decision tree):**

```
Do you need a runtime object?
  ├── No → Use a union type.
  └── Yes → Do you need reverse mapping?
           ├── Yes → Use a numeric enum (carefully).
           └── No → Use an `as const` object + derived union.
```

**Constraints and limitations.**

- Enums are still appropriate for **autogenerated code** or **legacy APIs** that require them.
- Some teams allow enums for **runtime-required constants** while banning them elsewhere.
- Lint rules can enforce the chosen convention.

### Annotated Complete Step by Step Code Examples

#### Example 1: Union type over enum

```typescript
// example_best_union.ts
// ❌ Enum: emits runtime object, not tree-shakeable.
// enum Status { Loading, Success, Error }

// ✅ Union type: erased, tree-shakeable, exhaustive.
type Status = "loading" | "success" | "error";

function handle(status: Status): string {
  switch (status) {
    case "loading": return "Loading...";
    case "success": return "Done";
    case "error": return "Failed";
  }
}

console.log(handle("success")); // Expected: Done
```

**Expected Output:**
```
Done
```

**Why this output occurs.** The union type is fully erased. The `switch` is exhaustive and no runtime enum object is emitted.

---

#### Example 2: `as const` object over enum when runtime object is needed

```typescript
// example_best_as_const.ts
// ❌ String enum: emits runtime object with no reverse mapping.
// enum Status { Loading = "loading", Success = "success" }

// ✅ `as const` object: runtime object + precise type.
const Status = {
  Loading: "loading",
  Success: "success",
} as const;

type Status = (typeof Status)[keyof typeof Status];

function handle(status: Status): string {
  switch (status) {
    case Status.Loading: return "Loading...";
    case Status.Success: return "Done";
  }
}

console.log(handle(Status.Loading)); // Expected: Loading...
```

**Expected Output:**
```
Loading...
```

**Why this output occurs.** The `as const` object provides a runtime object and a derived union type. It is structurally typed, tree-shakeable, and compatible with all build tools.

---

#### Example 3: Avoiding `const` enums with `isolatedModules`

```typescript
// example_best_no_const_enum.ts
// ❌ `const` enum with isolatedModules can fail.
// const enum Status { Active = 0, Inactive = 1 }

// ✅ Regular enum (or union type) works with isolatedModules.
enum Status {
  Active = 0,
  Inactive = 1,
}

console.log(Status.Active); // Expected: 0
```

**Expected Output:**
```
0
```

**Why this output occurs.** `const` enums are inlined and may not work with single-file transpilers. A regular enum (or union type) is compatible with `isolatedModules`.

---

### Real-World Cases

- **Modern codebases**: union types and `as const` objects as the default.
- **Library authoring**: avoiding `const` enums to support all build tools.
- **Legacy migration**: replacing enums with `as const` objects incrementally.
- **Autogenerated code**: allowing enums when generated by external tools.

---

## References

- TypeScript Handbook — Enums - https://www.typescriptlang.org/docs/handbook/enums.html
- TypeScript Handbook — Enums (Legacy) - https://www.typescriptlang.org/docs/handbook/2/enums.html
- TypeScript 2.0 Release Notes — `const` enums - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html
- TSConfig Reference — `isolatedModules` - https://www.typescriptlang.org/tsconfig/isolatedModules.html
- TSConfig Reference — `preserveConstEnums` - https://www.typescriptlang.org/tsconfig/preserveConstEnums.html
- TypeScript Handbook — Enums: Reverse Mappings - https://www.typescriptlang.org/docs/handbook/enums.html#reverse-mappings
- Bitwarden Contributing Docs — ADR-0025: Deprecate TypeScript Enum Types - https://contributing.bitwarden.com/architecture/adr/ts-deprecate-enums/
- Beraliv — With or Without Enums in TypeScript (Bundle Size Impact) - https://github.com/Beraliv/with-or-without-enums-bundle-size-impact
- pproenca/dot-skills — Convert Frozen-Object Enums to `const` Objects or Unions - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/migrate-js-to-modern-typescript/references/idiom-enum-to-union-or-const.md
- SSW Rules — Do You Know Why to Use `const` Assertions Instead of TypeScript Enums? - https://www.ssw.com.au/rules/do-you-know-why-to-use-const-assertions-instead-of-typescript-enums
- Convex — Converting TypeScript Enums to Strings (Reverse Mappings) - https://www.convex.dev/typescript/core-concepts/typescript-enum-to-string
- Refine — A Detailed Guide on TypeScript Enum with Examples - https://raw.githubusercontent.com/refinedev/refine/724f795367cd8c5bd4fdb7c77e9fed45bbda000d/documentation/blog/2025-01-06-typescript-enums.md
- Educative — TypeScript Enum Guide: Get Started in 5 Minutes - https://www.educative.io/blog/typescript-enum-guide
- Cloudpack — TypeScript Coding Standard (Avoid `const` Enums, Prefer Unions) - https://microsoft.github.io/
- TypeScript ESLint — `no-duplicate-enum-values` Rule - https://typescript-eslint.io/rules/no-duplicate-enum-values/