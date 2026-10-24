# Enum Alternatives in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Enum alternatives** are TypeScript patterns that provide the same benefits as enums—named, type-safe constants for a closed set of values—without the runtime artifacts, nominal typing quirks, and tooling incompatibilities of the `enum` keyword. The two primary alternatives are **union types of literal values** (e.g., `type Status = "loading" | "success"`) and **`as const` objects with derived union types** (e.g., `const Status = { Loading: "loading" } as const; type Status = (typeof Status)[keyof typeof Status]`).

**Technical Definition.** Union literal types exist **only at compile time** and are fully **erasable**: they emit no JavaScript, produce zero runtime cost, and work with all modern type-stripping toolchains. `as const` objects preserve **literal types** (preventing widening to `string`) while emitting a plain JavaScript object that is **tree-shakeable** and structurally typed. Both alternatives are **structural**, unlike enums, which are **nominal**. The decisive modern problem with `enum` is no longer bundle size—it is that enums emit a runtime lookup object, making them **non-erasable** and incompatible with `erasableSyntaxOnly` (TS 5.8) and Node.js native type-stripping.

**Beginner-Friendly Explanation.** Enum alternatives are **other ways to define a set of named constants** that avoid the problems with enums. The **simplest** is a **union of string literals**: `"loading" | "success"`. It adds **zero runtime code**—it disappears completely when your code is compiled. When you also need a **runtime object** (for iteration or lookup), use an **`as const` object**: it gives you both a plain JavaScript object and precise literal types. Both alternatives are **structural** (any matching value works) and **tree-shakeable**, unlike enums.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Erasability** | Union types emit **no runtime code**; `as const` objects emit plain, tree-shakeable JavaScript. |
| **Structural typing** | Alternatives are **structural**—values with the right shape are compatible; enums are **nominal**. |
| **Literal preservation** | `as const` prevents widening, preserving exact literal types for narrowing and exhaustiveness checks. |
| **Tree-shaking** | Union types and `as const` objects are tree-shakeable; enum objects are **not**. |
| **Tooling compatibility** | Alternatives work with `isolatedModules`, Babel, esbuild, SWC, and Node.js type-stripping. |
| **Iteration support** | `as const` arrays/objects support runtime iteration; union types require a separate runtime array. |
| **No reverse mapping** | Alternatives do not provide enum-style reverse mappings; build a lookup if needed. |

---

### Prerequisites

- **TypeScript basics**: type aliases, interfaces, and union types.
- **Literal types**: `"hello"`, `42`, `true`.
- **`as const` assertions**: how they preserve literal types and make properties readonly.
- **`typeof` and `keyof`**: for deriving union types from objects and arrays.
- **Control flow**: `switch`, `if`/`else`, and exhaustiveness checking with `never`.

---

### Related Programming Areas

- **Enum trade-offs**: non-erasable code, nominal typing, and tree-shaking resistance.
- **Erasable syntax**: TypeScript 5.8's `erasableSyntaxOnly` and Node.js native type-stripping.
- **Structural vs. nominal typing**: alternatives are structural; enums are nominal.
- **Exhaustiveness checking**: union types enable `never`-based checks in `switch` statements.
- **Bundle optimization**: eliminating runtime enum objects reduces bundle size.

---

## Core Concepts / Features

---

## 1. Union Types

### Definitions

**Core Definition.** A **union literal type** is a type composed entirely of literal values (strings, numbers, or booleans), representing a **closed set** of permitted values. It is the **lightest-weight** enum alternative: fully erasable, structural, and tree-shakeable.

**Technical Definition.** A union literal type `type T = "a" | "b" | "c"` restricts a value to one of the specified literals. It exists **only at compile time**—nothing is emitted to JavaScript. It supports **exhaustive narrowing** via `switch` and `never`, and can be **derived from arrays or objects** for runtime iteration. Union types are **structural**: any value equal to one of the literals is assignable, unlike nominal enums.

**Beginner-Friendly Explanation.** A union type is **a list of allowed values**. `type Status = "loading" | "success" | "error"` means only those three strings are valid. It is the simplest enum alternative: it adds **no runtime code** and works everywhere.

### Purposes

- **To restrict a value to a fixed set of literals** without any runtime cost.
- **To enable exhaustive checking** in `switch` statements.
- **To replace enums in compile-time-only contexts** (the vast majority of cases).
- **To remain compatible with type-stripping tools** and modern build pipelines.
- **To serve as the foundation for discriminated unions** and literal-based patterns.

### Syntax Rules and Structure

```typescript
type Status = "loading" | "success" | "error";
```

**Syntax rules.**

- Union members must be **literal types** (`"a"`, `42`, `true`).
- The union is **fully erasable**—no JavaScript is emitted.
- Supports exhaustive `switch` with `never` in the `default` branch.
- May be **derived from arrays** (`type T = typeof arr[number]`) for runtime iteration.
- Union types are **structural**—any matching literal is assignable.

**Constraints and limitations.**

- **No runtime object**: iteration and validation require a separate array or object.
- **No reverse mapping**: you cannot convert a value back to a name without a lookup.
- **Large unions** (>12–50 members) cause quadratic checking; group into discriminated categories.

### Annotated Complete Step by Step Code Examples

#### Example 1: Union type with exhaustive switch

```typescript
// example_union.ts
type Status = "loading" | "success" | "error";

function assertNever(value: never): never {
  throw new Error(`Unhandled: ${value}`);
}

function handle(status: Status): string {
  switch (status) {
    case "loading": return "Loading...";
    case "success": return "Done";
    case "error": return "Failed";
    default: return assertNever(status);
  }
}

console.log(handle("loading")); // Expected: Loading...
```

**Setup.** Compile with `npx tsc example_union.ts --strict`.

**Expected Output:**
```
Loading...
```

**Why this output occurs.** The union type restricts `status` to three literals. The `switch` handles all three; the `default` branch receives `never`, ensuring exhaustiveness. No runtime code is emitted for the type.

---

### Real-World Cases

- **API statuses**: `type Status = "ok" | "error"`.
- **Sort orders**: `type Order = "asc" | "desc"`.
- **Log levels**: `type Level = "debug" | "info" | "warn" | "error"`.

---

## 2. Object Literals (`as const`)

### Definitions

**Core Definition.** An **`as const` object** is a plain JavaScript object literal annotated with `as const`, which **preserves exact literal types** (preventing widening) and makes properties **readonly**. A **derived union type** extracted from the object provides the same closed-set type safety as an enum, with a runtime object that is tree-shakeable and structurally typed.

**Technical Definition.** The `as const` assertion causes the compiler to infer the **narrowest possible literal types** for the object's properties. The type `(typeof Object)[keyof typeof Object]` extracts the union of all property values. This pattern **replaces enums idiomatically**: it provides a runtime object (for iteration, lookup, and mapping) without the non-erasable IIFE and nominal typing of enums. The emitted JavaScript is a plain object literal, fully compatible with tree-shaking and type-stripping.

**Beginner-Friendly Explanation.** An `as const` object is **a plain JavaScript object whose values keep their exact types**. Instead of TypeScript widening `"loading"` to `string`, it stays `"loading"`. You then derive a union type from it. This gives you **both** a runtime object **and** precise types—without the enum baggage.

### Purposes

- **To provide a runtime object** (for iteration, mapping, or lookup) with **zero enum overhead**.
- **To preserve literal types** for exhaustive narrowing and type safety.
- **To replace enums idiomatically** with a structural, tree-shakeable pattern.
- **To serve as a single source of truth** for both values and types.
- **To enable runtime validation** through `Object.values()` and type guards.

### Syntax Rules and Structure

```typescript
const Status = {
  Loading: "loading",
  Success: "success",
  Error: "error",
} as const;

type Status = (typeof Status)[keyof typeof Status];
```

**Component breakdown:**

- `as const` — preserves literal types and makes properties readonly.
- `typeof Status` — the type of the object.
- `keyof typeof Status` — the union of keys.
- `(typeof Status)[keyof typeof Status]` — the union of values.

**Syntax rules.**

- `as const` must be applied to the **object literal**.
- Properties become **readonly**.
- Values are **literal types**, not widened.
- The value and type may **share the same name** (declaration merging of value and type spaces).

**Constraints and limitations.**

- Properties are **readonly**—mutation is prohibited.
- No **reverse mapping**; build a separate lookup if needed.
- The object is a **runtime artifact** (though tree-shakeable and plain JavaScript).

### Annotated Complete Step by Step Code Examples

#### Example 1: `as const` object with derived union

```typescript
// example_as_const.ts
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
console.log(Status.Loading);         // Expected: loading
```

**Expected Output:**
```
Loading...
loading
```

**Why this output occurs.** `as const` preserves the literal values `"loading"`, `"success"`, and `"error"`. The derived type `Status` is the union of those values. The object is a plain JavaScript object—no enum IIFE, no reverse mappings, fully tree-shakeable.

---

#### Example 2: `as const` array for iteration

```typescript
// example_as_const_array.ts
const HTTP_STATUS_CODES = ["200", "400", "404", "500"] as const;

type HttpStatusCode = (typeof HTTP_STATUS_CODES)[number];

for (const code of HTTP_STATUS_CODES) {
  console.log(code); // code is typed as HttpStatusCode
}
// Expected: 200, 400, 404, 500
```

**Expected Output:**
```
200
400
404
500
```

**Why this output occurs.** `as const` on the array preserves the literal types. `(typeof arr)[number]` extracts the union of elements. The array is a plain JavaScript array—iterable, tree-shakeable, and type-safe.

---

### Real-World Cases

- **Configuration maps**: `const Config = { Dev: "development", Prod: "production" } as const`.
- **Icon/asset maps**: `const Icons = { Sun: "sun.png", Moon: "moon.png" } as const`.
- **HTTP status maps**: `const Codes = ["200", "404"] as const` for iteration.

---

## 3. Enum Alternative Best Practices

### Definitions

**Core Definition.** **Enum alternative best practices** are guidelines for replacing enums with **union types** or **`as const` objects**, choosing the right pattern for each use case, and avoiding the runtime, tooling, and type-safety pitfalls of enums.

**Technical Definition.** Best practices center on the **modern consensus** that TypeScript enums are often an anti-pattern: they are **non-erasable**, **not tree-shakeable**, and **incompatible with type-stripping toolchains**. Anders Hejlsberg, TypeScript's creator, has stated he would not have included enums in the language in retrospect, and the recommendation is to **use union types instead**. The decision framework is: use **union types** when no runtime object is needed; use **`as const` objects** when runtime values (iteration, lookup, mapping) are required.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to replace enums cleanly. The big rule is: **use union types for compile-time-only sets** (which is almost everything), and **use `as const` objects when you need the values at runtime** (iteration, mapping). Avoid enums entirely in new code.

### Purposes

- **To eliminate non-erasable runtime code** and improve tree-shaking.
- **To remain compatible with `isolatedModules`, Babel, esbuild, SWC, and Node.js type-stripping**.
- **To preserve precise literal types** for exhaustive narrowing.
- **To choose the lightest-weight pattern** for each use case.
- **To migrate existing enums** to modern alternatives incrementally.

### Syntax Rules and Structure

Best practices are conventions that interact with union and `as const` syntax. The core rules are:

1. **Use union types for compile-time-only sets.** `type Status = "loading" | "success"` is erasable, tree-shakeable, and supports exhaustive checks. This is the default choice.
2. **Use `as const` objects when a runtime object is needed.** For iteration, mapping, or lookup, `as const` + derived union provides a plain JavaScript object without enum baggage.
3. **Never use `enum` in new code.** Enums emit runtime objects, resist tree-shaking, and conflict with type-stripping. Hejlsberg himself has expressed regret over adding them.
4. **Derive types from arrays for iteration.** `const arr = ["a", "b"] as const; type T = typeof arr[number]` gives type-safe iteration without duplication.
5. **Use `as const` objects for key-value maps.** `const Icons = { Sun: "sun.png" } as const; type Icon = keyof typeof Icons` provides both keys and values as types.
6. **Use `satisfies` for validation without widening.** When you need to validate an object against a type while preserving literals, `satisfies` is preferable to `as`.
7. **Migrate enums incrementally.** Replace frozen-object enums with `as const` + derived unions; replace simple enums with union types.

**Component breakdown (enum alternative decision tree):**

```
Do you need a runtime object (iteration, lookup, mapping)?
  ├── Yes → Use an `as const` object + derived union type.
  └── No → Use a union literal type (the default).
```

**Constraints and limitations.**

- Union types cannot be iterated at runtime without a separate array.
- `as const` objects do not provide reverse mappings; build a lookup if needed.
- Large union types (>12–50 members) cause quadratic checking; group into categories.

### Annotated Complete Step by Step Code Examples

#### Example 1: Replacing an enum with a union type

```typescript
// example_best_union.ts
// ❌ Enum: non-erasable, nominal, not tree-shakeable.
// enum Status { Loading, Success, Error }

// ✅ Union type: erasable, structural, tree-shakeable.
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

**Why this output occurs.** The union type emits no runtime code. The `switch` is exhaustive and fully erasable, working with every build tool.

---

#### Example 2: Replacing an enum with `as const` + derived union

```typescript
// example_best_as_const.ts
// ❌ Enum: runtime object, no tree-shaking.
// enum OrderStatus { Pending = "pending", Paid = "paid" }

// ✅ `as const` object + derived union: runtime object, tree-shakeable.
const OrderStatus = {
  Pending: "pending",
  Paid: "paid",
} as const;

type OrderStatus = (typeof OrderStatus)[keyof typeof OrderStatus];

function isComplete(status: OrderStatus): boolean {
  return status === OrderStatus.Paid;
}

console.log(isComplete(OrderStatus.Paid)); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** The `as const` object preserves the literal values. The derived union type enables exhaustive narrowing. The emitted JavaScript is a plain, tree-shakeable object—no enum IIFE, no reverse mappings.

---

#### Example 3: `as const` array for type-safe iteration

```typescript
// example_best_array.ts
const FRUITS = ["Apple", "Banana", "Cherry"] as const;
type Fruit = (typeof FRUITS)[number];

for (const fruit of FRUITS) {
  console.log(fruit); // fruit is typed as Fruit
}
// Expected: Apple, Banana, Cherry
```

**Expected Output:**
```
Apple
Banana
Cherry
```

**Why this output occurs.** `as const` preserves the literal types. `(typeof FRUITS)[number]` extracts the union. The array is iterable, tree-shakeable, and type-safe—unlike an enum, which would emit a reverse-mapped object.

---

### Real-World Cases

- **API statuses**: replacing `enum Status` with `type Status = "ok" | "error"`.
- **Configuration maps**: `const Config = { Dev: "development", Prod: "production" } as const`.
- **Icon/asset registries**: `const Icons = { Sun: "sun.png", Moon: "moon.png" } as const`.
- **Migration**: converting legacy enums to union types or `as const` objects.

---

## References

- TypeScript Coding Standard for Mission-Critical Systems — `as const` Objects vs. Enums - https://github.com/MichaelLeeHobbs/stderr/blob/main/docs/TypeScript Coding Standard for Mission-Critical Systems.md
- Convert Frozen-Object Enums to `const` Objects or Unions — pproenca/dot-skills - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/migrate-js-to-modern-typescript/references/idiom-enum-to-union-or-const.md
- Do You Know Why to Use Const Assertions Instead of TypeScript Enums? — SSW.Rules - https://www.ssw.com.au/rules/typescript-enums
- Use Union Literals Instead of Enums — pproenca/dot-skills - https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/typescript-refactor/references/perf-union-literals-over-enums.md
- Union Types and Enums — KodeKloud - https://notes.kodekloud.com/docs/CDK-for-Terraform-with-TypeScript/Introduction-to-TypeScript/Union-Types-and-Enums/page
- `as const` Objects and Derived Union Types — TypeScript Handbook (Const Assertions) - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions
- Better Enums — Why Union Types Are Preferred - https://www.npmjs.com/package/better-enums
- TypeScript 5.8 — `erasableSyntaxOnly` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-8.html