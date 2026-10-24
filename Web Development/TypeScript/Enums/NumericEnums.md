# Numeric Enums in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **numeric enum** is a TypeScript enumeration whose members are assigned **numeric values**. It is one of the few TypeScript features that **emits runtime JavaScript code**, generating an object that supports both forward mappings (name → value) and, uniquely among enums, **reverse mappings** (value → name). Numeric enums are a **nominal** construct, though TypeScript’s type system deliberately relaxes this nominality in ways that have significant safety implications.

**Technical Definition.** A numeric enum declaration introduces a **type** (the union of its member types) and a **value** (a runtime object). Members without explicit initializers are **auto-incremented** from `0` (or from the previous member’s value plus one). The compiler generates a runtime object containing both `Name → value` entries and `value → Name` entries for numeric members. TypeScript **unsoundly** allows any `number` to be assigned to a numeric enum type, which supports bit-flag patterns but permits invalid values to enter typed APIs. Numeric enums are **non-erasable** and are **not tree-shakeable**, increasing bundle size.

**Beginner-Friendly Explanation.** A numeric enum is **an enum whose values are numbers**. If you do not give a member a value, TypeScript assigns the next number automatically, starting from `0`. Numeric enums also let you look up a member’s **name** from its **number**, which string enums cannot do. However, numeric enums have a surprising weakness: TypeScript lets you pass **any number** where a numeric enum is expected—even numbers that are not valid members—so the type system does not fully protect you.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Numeric values** | Members are assigned numbers; uninitialized members auto-increment from `0`. |
| **Reverse mapping** | Runtime object includes `value → Name` entries, enabling `Enum[value]` lookups. |
| **Non-erasable** | Emits a JavaScript object at runtime, unlike interfaces and type aliases. |
| **Nominal type** | Enum types are nominal, but any `number` is assignable to a numeric enum. |
| **Not tree-shakeable** | Enum objects are not removed by tree-shaking, increasing bundle size. |
| **Computed members** | Members may use constant expressions or runtime-computed values (with ordering rules). |
| **Six keys per three members** | A three-member numeric enum has **six** object keys (three forward, three reverse). |

---

### Prerequisites

- **TypeScript basics**: type annotations, type aliases, and union types.
- **JavaScript objects**: property access and runtime object behavior.
- **Compiler flags**: `isolatedModules`, `preserveConstEnums`.
- **Bitwise operations** for bit-flag use cases.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **String enums** — the safer, more readable alternative for many use cases.
- **Union types and `as const` objects** — modern enum alternatives.
- **Bit flags and permissions** — a legitimate numeric enum use case.
- **Runtime code generation** — enums are a non-erasable TypeScript feature.
- **Bundle size and tree-shaking** — enums resist tree-shaking.

---

## Core Concepts / Features

---

## 1. Numeric Enums

### Definitions

**Core Definition.** A **numeric enum** is an enum whose members are assigned numeric values. It provides named numeric constants, auto-incrementing behavior, and runtime reverse mappings.

**Technical Definition.** A numeric enum declaration creates a runtime object with forward and reverse mappings for all numeric members. Members may have explicit numeric initializers or be auto-incremented. The enum type is **nominal**, but TypeScript permits any `number` to be assigned to a numeric enum type—a deliberate unsoundness that supports bit-flag patterns but allows invalid values.

**Beginner-Friendly Explanation.** A numeric enum is **a set of named numbers**. It is useful for numeric codes, bit flags, and cases where you need to look up a name from a number at runtime.

### Purposes

- **To assign meaningful names to numeric codes** (HTTP status codes, error codes).
- **To enable reverse lookup** from numeric values to readable names for debugging and display.
- **To support bit-flag patterns** with computed bitwise members.
- **To provide a stable runtime object** of numeric constants.

### Syntax Rules and Structure

**General syntax:**

```typescript
enum NumericEnum {
  Member1,
  Member2 = value,
  Member3,
}
```

**Component breakdown:**

- `enum` — declaration keyword.
- `NumericEnum` — the enum identifier.
- `Member1` — a member without initializer (auto-incremented).
- `Member2 = value` — a member with an explicit numeric initializer.

**Syntax rules.**

- Uninitialized members auto-increment from `0` (or from the previous member).
- Explicit numeric initializers set the value directly.
- Reverse mappings are generated for all numeric members.
- Computed members may appear after constant members.

**Constraints and limitations.**

- **Unsoundness**: any `number` is assignable to a numeric enum type, allowing invalid values.
- **Opaque runtime values**: numeric codes are often meaningless without a reverse lookup.
- **Non-erasable**: emits a runtime object, increasing bundle size.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic numeric enum

```typescript
// example_numeric_basic.ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

console.log(Direction.Up);    // Expected: 0
console.log(Direction.Right); // Expected: 3
```

**Setup.** Compile with `npx tsc example_numeric_basic.ts --strict`.

**Expected Output:**
```
0
3
```

**Why this output occurs.** Members auto-increment from `0`. `Up` is `0`, and `Right` is `3`. The enum is a numeric enum with forward mappings and reverse mappings.

---

### Real-World Cases

- **HTTP status codes**: `enum HttpStatus { OK = 200, NotFound = 404 }`.
- **Error codes**: `enum ErrorCode { BadRequest = 400, Unauthorized = 401 }`.
- **Bit flags**: `enum Permission { Read = 1 << 0, Write = 1 << 1 }`.

---

## 2. Auto-Incrementing

### Definitions

**Core Definition.** **Auto-incrementing** is the behavior whereby numeric enum members without explicit initializers are automatically assigned values: the first such member is `0`, and each subsequent one is the previous member’s value plus `1`.

**Technical Definition.** The TypeScript Handbook states: “If we wanted, we could leave off the initializers entirely … Up would have the value 0, Down would have 1, etc.” This auto-incrementing behavior is useful when the member values themselves do not matter, only that each value is distinct. Auto-incrementing applies to members without initializers; explicit initializers reset the counter.

**Beginner-Friendly Explanation.** Auto-incrementing means **TypeScript numbers the members for you**. If you do not give a member a value, it gets the next number automatically. This is convenient when you just need distinct values.

### Purposes

- **To avoid manually numbering members** when values do not matter.
- **To ensure distinct values** for each member.
- **To simplify enum definitions** for simple ordered sets.

### Syntax Rules and Structure

```typescript
enum AutoEnum {
  A,       // 0
  B,       // 1
  C,       // 2
  D = 10,  // 10
  E,       // 11
}
```

**Syntax rules.**

- The first uninitialized member is `0`.
- Each subsequent uninitialized member is previous + 1.
- An explicit initializer resets the auto-increment counter.
- Auto-incrementing works only for numeric members.

**Constraints and limitations.**

- Auto-incrementing is not available for string enums.
- Sparse or negative values can make reverse mapping unreliable.
- Members without initializers must appear after constant numeric members if computed members are present.

### Annotated Complete Step by Step Code Examples

#### Example 1: Auto-increment with explicit reset

```typescript
// example_auto_increment.ts
enum Status {
  Active,   // 0
  Inactive, // 1
  Pending,  // 2
  Archived = 10,
  Deleted,  // 11
}

console.log(Status.Active);   // Expected: 0
console.log(Status.Archived); // Expected: 10
console.log(Status.Deleted);  // Expected: 11
```

**Expected Output:**
```
0
10
11
```

**Why this output occurs.** The first three members auto-increment from `0`. The explicit initializer `Archived = 10` resets the counter, and `Deleted` auto-increments to `11`.

---

### Real-World Cases

- **Ordered lists**: `enum Priority { Low, Medium, High }`.
- **Simple flags**: `enum Feature { DarkMode, Beta, Experimental }`.

---

## 3. Custom Values

### Definitions

**Core Definition.** **Custom values** in numeric enums are **explicitly assigned numeric initializers** that override the default auto-incrementing behavior.

**Technical Definition.** Numeric enum members may be initialized with **numeric literals**, **constant expressions** (e.g., `1 << 2`), or **computed values** (e.g., function calls). Constant members are evaluated at compile time; computed members are evaluated at runtime. When computed members are used, uninitialized members must come first or after other constant members.

**Beginner-Friendly Explanation.** Custom values let you **choose the exact numbers** for enum members. You can use them for specific codes (like HTTP statuses) or for bit flags.

### Purposes

- **To assign specific numeric codes** that must match external systems.
- **To create bit-flag values** with shift expressions.
- **To enforce distinct, meaningful numbers** for each member.

### Syntax Rules and Structure

```typescript
enum CustomEnum {
  A = 1,
  B = 5,
  C = 10,
  D = 1 << 4, // 16
}
```

**Syntax rules.**

- Custom values may be numeric literals or constant expressions.
- Computed members (e.g., function calls) are evaluated at runtime.
- Uninitialized members after computed members are not allowed unless the computed members are constant.
- Reverse mapping is generated for all numeric members, including custom values.

**Constraints and limitations.**

- Computed members require careful ordering.
- Sparse values can make reverse mapping unreliable.
- Mixing computed and uninitialized members requires the uninitialized members to come first or after constant members.

### Annotated Complete Step by Step Code Examples

#### Example 1: Custom values and bit flags

```typescript
// example_custom_values.ts
enum Permissions {
  None = 0,
  Read = 1 << 0,  // 1
  Write = 1 << 1, // 2
  Execute = 1 << 2, // 4
  All = Read | Write | Execute, // 7
}

console.log(Permissions.Read);  // Expected: 1
console.log(Permissions.All);   // Expected: 7
console.log(Permissions[7]);    // Expected: All (reverse mapping)
```

**Expected Output:**
```
1
7
All
```

**Why this output occurs.** The bit-shift expressions assign specific numeric values. `All` combines the flags with bitwise OR, producing `7`. Reverse mapping returns `"All"` for `7`.

---

### Real-World Cases

- **HTTP status codes**: `enum HttpStatus { OK = 200, Created = 201 }`.
- **Bit flags**: `enum FileMode { Read = 4, Write = 2, Execute = 1 }`.
- **External API codes**: matching third-party numeric identifiers.

---

## 4. Reverse Mapping

### Definitions

**Core Definition.** **Reverse mapping** is the runtime behavior of numeric enums whereby the compiled object contains **value → name** entries, allowing `Enum[value]` to return the member name.

**Technical Definition.** TypeScript generates both forward (`Name → value`) and reverse (`value → Name`) mappings for numeric enums. String enums do **not** have reverse mappings; `StringEnum[StringEnum.Member]` returns `undefined`. Reverse mappings are available only for **constant numeric values**; computed values may not produce reliable reverse mappings. Sparse or negative values can make reverse mapping unreliable.

**Beginner-Friendly Explanation.** Reverse mapping is **looking up a number’s name in a numeric enum**. If `Status.Active` is `0`, then `Status[0]` gives you `"Active"`. String enums do not have this—there is no reverse lookup for strings.

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
- **Computed values** may not produce reliable reverse mappings.
- **`const` enums** lose runtime objects, so reverse mapping is unavailable.

### Annotated Complete Step by Step Code Examples

#### Example 1: Reverse mapping with numeric and string enums

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

## 5. Numeric Enum Best Practices

### Definitions

**Core Definition.** **Numeric enum best practices** are guidelines for using numeric enums safely and sparingly, given their **unsoundness**, **bundle-size cost**, and the availability of safer alternatives like **string enums**, **union types**, and **`as const` objects**.

**Technical Definition.** Best practices center on the **modern consensus** that numeric enums should be used only when their specific features (reverse mapping, bit flags, runtime lookup object) are genuinely needed. The TypeScript community increasingly recommends **string enums or `as const` objects** for most use cases because numeric enums are **unsound** (any `number` is assignable), increase **bundle size**, and are **not tree-shakeable**. Bitwarden’s architecture decision record **deprecates enums entirely** in favor of `as const` objects, citing invalid value transmission and output size.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to avoid numeric enum pitfalls. The big rules are: do not use numeric enums for API boundaries because any number is accepted; prefer string enums or `as const` objects for readability and safety; use numeric enums only for bit flags or when you genuinely need reverse mapping.

### Purposes

- **To avoid the unsoundness** of numeric enums at API boundaries.
- **To reduce bundle size** by preferring erasable alternatives.
- **To maximize type safety** with string enums or `as const` objects.
- **To use numeric enums only where their specific features are needed** (bit flags, reverse mapping).

### Syntax Rules and Structure

Best practices are conventions that interact with numeric enum syntax. The core rules are:

1. **Do not use numeric enums at API boundaries.** Any `number` is assignable to a numeric enum, so invalid values can enter typed functions. Use string enums or union types instead.
2. **Prefer string enums or `as const` objects for readability.** String enums serialize well and do not suffer from numeric unsoundness.
3. **Use numeric enums only for bit flags** or when **reverse mapping** is genuinely needed.
4. **Avoid sparse or negative numeric values** because they make reverse mapping unreliable.
5. **Prefer `as const` objects with derived union types** for most named-constant use cases.
6. **Enable lint rules** (e.g., `no-enums` or `no-mixed-enums`) to enforce team conventions.

**Component breakdown (numeric enum decision tree):**

```
Do you need bitwise flags or reverse mapping?
  ├── Yes → Use a numeric enum (carefully).
  └── No → Do you need a runtime object?
           ├── Yes → Use an `as const` object + derived union.
           └── No → Use a union type.
```

**Constraints and limitations.**

- Numeric enums remain appropriate for **autogenerated code** or **legacy bit-flag APIs**.
- Some teams ban numeric enums entirely via lint rules.
- `const` enums can reduce bundle size but conflict with `isolatedModules`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Unsound numeric enum at an API boundary

```typescript
// example_best_unsound.ts
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

**Why this output occurs.** TypeScript allows any `number` to be assigned to a numeric enum type, so `999` is accepted despite not being a valid `Status` member. This is the primary reason to avoid numeric enums at API boundaries.

---

#### Example 2: String enum or `as const` alternative

```typescript
// example_best_alternative.ts
// ❌ Numeric enum: unsound at boundaries.
// enum Status { Active = 0, Inactive = 1 }

// ✅ String enum: rejects invalid strings.
enum StringStatus {
  Active = "ACTIVE",
  Inactive = "INACTIVE",
}

function setStatus(status: StringStatus) {
  console.log(status);
}

setStatus(StringStatus.Active); // Expected: ACTIVE
// setStatus("ACTIVE"); // Error: string not assignable to StringStatus.
```

**Expected Output:**
```
ACTIVE
```

**Why this output occurs.** String enums reject arbitrary strings at the type level, preventing the unsoundness that numeric enums allow. For many use cases, an `as const` object with a derived union type is even safer and more flexible.

---

### Real-World Cases

- **API boundaries**: avoiding numeric enums in favor of string enums or union types.
- **Bit flags**: using numeric enums only for permission masks and similar patterns.
- **Bundle-sensitive projects**: preferring `as const` objects to avoid runtime enum objects.
- **Code review**: flagging numeric enums at API boundaries and suggesting alternatives.

---

## References

- TypeScript Handbook — Enums (Numeric Enums, Auto-incrementing, Reverse Mappings) - https://www.typescriptlang.org/docs/handbook/enums.html
- TypeScript Handbook — Enums: Reverse Mappings - https://www.typescriptlang.org/docs/handbook/enums.html#reverse-mappings
- TypeScript Handbook — Enums: Computed and Constant Members - https://www.typescriptlang.org/docs/handbook/enums.html#computed-and-constant-members
- Total TypeScript — Why I Don’t Like Enums - https://www.totaltypescript.com/why-i-dont-like-typescript-enums
- Bitwarden Contributing Docs — ADR-0025: Deprecate TypeScript Enum Types - https://contributing.bitwarden.com/architecture/adr/ts-deprecate-enums/
- Educative — TypeScript Enum Guide: Get Started in 5 Minutes - https://www.educative.io/blog/typescript-enum-guide
- Microsoft/TypeScript Issue #53562 — Numeric Enums May Not Be Type Safe - https://github.com/microsoft/TypeScript/issues/53562
- TypeScript ESLint — `no-unsafe-enum-assignment` Rule - https://typescript-eslint.io/rules/no-unsafe-enum-assignment/
- Oxlint — `typescript/no-mixed-enums` Rule - https://oxc.rs/docs/guide/usage/linter/rules/typescript/no-mixed-enums