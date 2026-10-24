# Null Safety in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Null safety** in TypeScript is the set of language features and compiler settings that make `null` and `undefined` **first-class, explicitly tracked types** rather than implicit members of every type. Under strict null checking, the compiler enforces that potentially nullish values are **narrowed or handled** before use, preventing the most common JavaScript runtime crash: reading a property of `undefined`.

**Technical Definition.** Null safety is enabled by the `strictNullChecks` compiler flag (part of `strict`). In strict null checking mode, `null` and `undefined` are **not in the domain of every type** and are assignable only to themselves (and `undefined` to `void`). The types `T`, `T | null`, and `T | undefined` become **distinct**, and the compiler uses **control-flow analysis** to narrow them via truthiness checks, equality checks against `null`/`undefined`, and other guards. TypeScript provides dedicated operators—optional chaining (`?.`), nullish coalescing (`??`), and the non-null assertion (`!`)—to handle nullish values concisely.

**Beginner-Friendly Explanation.** Null safety means **TypeScript makes you deal with `null` and `undefined` explicitly**. Without it, any variable can secretly be `null`, and your program can crash when you try to use it. With null safety turned on, TypeScript treats “a string” and “a string that might be null” as different things, and it will not let you use the second one until you have checked. Optional chaining (`?.`), nullish coalescing (`??`), and the non-null assertion (`!`) are the tools you use to handle nullish values cleanly.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **`strictNullChecks`** | Makes `null` and `undefined` distinct from other types; the foundation of null safety. |
| **Explicit nullable types** | A value that can be null must be declared as `T \| null` or `T \| undefined`. |
| **Control-flow narrowing** | Truthiness checks and `==`/`!=` against `null` narrow away nullish types. |
| **Optional chaining (`?.`)** | Short-circuits property access, element access, and calls when the receiver is `null` or `undefined`. |
| **Nullish coalescing (`??`)** | Supplies a default only when the left side is `null` or `undefined`, not other falsy values. |
| **Non-null assertion (`!`)** | Tells the compiler a value is non-null **without runtime proof**; unsafe if wrong. |
| **Erasure** | All null-safety annotations and operators are erased; the runtime checks remain as ordinary JavaScript. |

---

### Prerequisites

- **TypeScript basics**: type annotations, union types, and type aliases.
- **Control flow**: `if`/`else`, ternaries, and early returns.
- **Type narrowing and type guards**.
- **Compiler flags**: `strict`, `strictNullChecks`.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Type narrowing** and control-flow analysis.
- **Optional properties** (`?`) and **optional parameters**.
- **Truthiness vs. nullish checks**.
- **Assertion functions** (`asserts value is NonNullable<T>`) as safer alternatives to `!`.
- **Utility types**: `NonNullable<T>`.

---

## Core Concepts / Features

---

## 1. `null`

### Definitions

**Core Definition.** **`null`** is a JavaScript primitive value representing the **intentional absence of a value**. In TypeScript, `null` is a **type** whose only value is `null`.

**Technical Definition.** Under `strictNullChecks`, `null` is **not assignable to other types** and must be explicitly included in a union (`T | null`). The compiler requires null checks before accessing members of a possibly-null value. `null` is distinct from `undefined`: `null` typically means “explicitly empty,” while `undefined` means “not yet assigned” or “missing.”

**Beginner-Friendly Explanation.** `null` means **“there is intentionally no value here.”** When null safety is on, TypeScript will not let you use a possibly-null value until you have checked that it is not null.

### Purposes

- **To represent an intentional absence of a value** (e.g., a database field that is set to `NULL`).
- **To distinguish “no value” from “not yet assigned”** (`undefined`).
- **To force callers to handle the empty case** through explicit union types.

### Syntax Rules and Structure

**General syntax:**

```
let value: T | null = ...;
if (value !== null) { /* use value as T */ }
```

**Component breakdown:**

- `: T | null` — the explicit nullable type.
- `!== null` — the null check that narrows `value` to `T`.

**Syntax rules.**

- Under `strictNullChecks`, `null` is assignable only to `null`, `any`, and `unknown`.
- Null checks must precede member access on a possibly-null value.
- `== null` (loose equality) checks for **both** `null` and `undefined`; `=== null` checks only `null`.

**Constraints and limitations.**

- Without `strictNullChecks`, `null` is assignable to every type and no checking occurs.
- `typeof null` returns `"object"`, which is a historical JavaScript quirk.

### Annotated Complete Step by Step Code Examples

#### Example 1: Nullable type with explicit check

```typescript
// example_null.ts
let username: string | null = "Ada";

if (username !== null) {
  console.log(username.toUpperCase()); // Expected: ADA
}

username = null;
if (username === null) {
  console.log("No username"); // Expected: No username
}
```

**Setup.** Compile with `npx tsc example_null.ts --strict`.

**Expected Output:**
```
ADA
No username
```

**Why this output occurs.** `username` is typed `string | null`. The `!== null` check narrows it to `string` in the first branch, enabling `.toUpperCase()`. The `=== null` check handles the null case explicitly.

---

### Real-World Cases

- **Database fields**: nullable columns mapped to `T | null`.
- **API responses**: fields that are explicitly `null` rather than missing.
- **Optional configuration**: settings that can be deliberately cleared.

---

## 2. `undefined`

### Definitions

**Core Definition.** **`undefined`** is a JavaScript primitive value representing a **missing value**—a variable that has not been assigned, a missing property, or a function that returns nothing. In TypeScript, `undefined` is a **type** whose only value is `undefined`.

**Technical Definition.** Under `strictNullChecks`, `undefined` is **not assignable to other types** and must be explicitly included (`T | undefined`). Optional parameters and properties automatically include `undefined` in their types. The compiler performs **assigned-before-use checking** for local variables whose types do not include `undefined`. `undefined` is assignable to `void`.

**Beginner-Friendly Explanation.** `undefined` means **“this value is missing”**—it was never given a value. Optional properties and parameters are `undefined` when omitted. Null safety forces you to check before using them.

### Purposes

- **To represent unassigned or missing values**.
- **To type optional parameters and properties** naturally.
- **To distinguish “missing” from “explicitly null”**.

### Syntax Rules and Structure

**General syntax:**

```
let value: T | undefined = ...;
if (value !== undefined) { /* use value as T */ }
```

**Component breakdown:**

- `: T | undefined` — the explicit undefined-able type.
- `!== undefined` — the check that narrows to `T`.

**Syntax rules.**

- Optional parameters and properties automatically include `undefined`.
- `== null` checks for both `null` and `undefined`.
- `undefined` is assignable to `void`.

**Constraints and limitations.**

- `undefined` is not the same as “not declared”; the variable exists but holds the value `undefined`.
- Accessing properties of `undefined` throws a runtime error.

### Annotated Complete Step by Step Code Examples

#### Example 1: Optional property and undefined check

```typescript
// example_undefined.ts
interface User {
  name: string;
  email?: string; // type is string | undefined
}

const user: User = { name: "Ada" };

if (user.email !== undefined) {
  console.log(user.email.toLowerCase()); // narrowed to string
} else {
  console.log("No email provided"); // Expected: No email provided
}
```

**Expected Output:**
```
No email provided
```

**Why this output occurs.** `email` is optional, so its type is `string | undefined`. The `!== undefined` check narrows it to `string` in the `if` branch; the `else` branch handles the missing case.

---

### Real-World Cases

- **Optional configuration**: missing config keys.
- **Function return values**: functions that may not return a value.
- **Array access**: out-of-bounds indices yield `undefined` (with `noUncheckedIndexedAccess`).

---

## 3. `strictNullChecks`

### Definitions

**Core Definition.** **`strictNullChecks`** is a TypeScript compiler flag that enables **strict null checking mode**, making `null` and `undefined` distinct types that are not assignable to every other type.

**Technical Definition.** When `strictNullChecks` is enabled, `null` and `undefined` are **removed from the domain of every type** and are assignable only to themselves (and `undefined` to `void`). The types `T`, `T | null`, and `T | undefined` become **distinct**, and the compiler enforces null checks through control-flow analysis. `strictNullChecks` is included in `strict: true` and is described as “the highest-value flag in any migration” because it prevents the most common JavaScript runtime crash.

**Beginner-Friendly Explanation.** `strictNullChecks` is **the switch that turns on null safety**. When it is off, TypeScript assumes every value exists and lets `null` and `undefined` slip into any variable. When it is on, TypeScript forces you to handle nullish values explicitly.

### Purposes

- **To prevent null pointer crashes** at compile time.
- **To make nullable types explicit** in function signatures and interfaces.
- **To enable control-flow narrowing** of nullish values.
- **To serve as the foundation** for optional chaining and nullish coalescing.

### Syntax Rules and Structure

**Configuration:**

```json
{
  "compilerOptions": {
    "strict": true,
    "strictNullChecks": true
  }
}
```

**Component breakdown:**

- `strict: true` — enables the full family of strict checks, including `strictNullChecks`.
- `strictNullChecks: true` — explicitly enables strict null checking.

**Syntax rules.**

- When enabled, `null` and `undefined` are assignable only to themselves (and `any`, `unknown`, and `void` for `undefined`).
- A value that can be null must be declared as `T | null` or `T | undefined`.
- The compiler requires **definite assignment** before use for non-undefined types.
- Type guards (`if (x)`, `x != null`, `x !== undefined`) narrow nullish types.

**Constraints and limitations.**

- Enabling `strictNullChecks` on an existing codebase can produce many errors; migration should be incremental.
- Some third-party libraries may lack accurate nullability annotations.

### Annotated Complete Step by Step Code Examples

#### Example 1: `strictNullChecks` prevents unsafe access

```typescript
// example_strict_null.ts
// Without strictNullChecks: compiles but crashes at runtime.
// function getEmail(users: Map<string, User>, id: string): string {
//   const user = users.get(id); // typed User, but actually User | undefined
//   return user.email; // throws at runtime when id is absent
// }

// With strictNullChecks: the gap is a compile error.
function getEmailSafe(users: Map<string, { email: string }>, id: string): string {
  const user = users.get(id); // typed User | undefined
  if (!user) throw new Error(`No user for id ${id}`);
  return user.email; // narrowed to User, safe
}

console.log(getEmailSafe(new Map([["u1", { email: "ada@example.com" }]]), "u1"));
// Expected: ada@example.com
```

**Expected Output:**
```
ada@example.com
```

**Why this output occurs.** With `strictNullChecks`, `users.get(id)` is typed `User | undefined`. The `if (!user)` guard narrows it to `User` before accessing `email`. Without the flag, the same code would compile but crash when the ID is absent.

---

### Real-World Cases

- **New projects**: enabling `strict: true` from day one.
- **Migrations**: adding `strictNullChecks` immediately after `noImplicitAny` for the highest bug yield.
- **Library authoring**: shipping accurate nullability annotations to consumers.

---

## 4. Optional Chaining (`?.`)

### Definitions

**Core Definition.** **Optional chaining (`?.`)** is an operator that **short-circuits** a property access, element access, or function call when the left-hand receiver is `null` or `undefined`, returning `undefined` instead of throwing.

**Technical Definition.** Introduced in TypeScript 3.7 (and ES2020), `?.` checks only the **immediate left-hand value** for `null` or `undefined`—not subsequent properties. It supports **optional property access** (`obj?.prop`), **optional element access** (`arr?.[0]`), and **optional call** (`fn?.()`). It **does not** short-circuit on falsy values like `0`, `""`, or `false`; it only short-circuits on `null` and `undefined`. Under `strictNullChecks`, the result type includes `undefined` when the receiver is nullable.

**Beginner-Friendly Explanation.** `?.` means **“if the thing before me is null or undefined, stop and give back `undefined`; otherwise, keep going.”** It replaces long `foo && foo.bar && foo.bar.baz` chains with `foo?.bar?.baz`.

### Purposes

- **To safely access nested properties** without repetitive null checks.
- **To safely call optional methods** (`log?.("message")`).
- **To safely access array elements** of possibly-missing arrays (`arr?.[0]`).
- **To replace verbose `&&` chains** with concise, intent-revealing syntax.

### Syntax Rules and Structure

**General syntax:**

```
obj?.property
obj?.[index]
fn?.()
```

**Component breakdown:**

- `?.` — the optional chaining operator.
- `obj`, `arr`, `fn` — the receiver.
- `property`, `[index]`, `()` — the access or call performed if the receiver is non-nullish.

**Syntax rules.**

- `?.` checks only the **immediate** left-hand value, not subsequent properties.
- For deep chains, repeat the operator: `foo?.bar?.baz`.
- The result type is `T | undefined` when the receiver is nullable.
- `?.` **does not** short-circuit on `0`, `""`, or `false`.
- **Optional call** (`fn?.()`) only skips the call; arguments are not evaluated if the function is nullish.

**Constraints and limitations.**

- `?.` does not protect against errors in subsequent properties: `foo?.bar.baz()` still throws if `bar` is nullish.
- In arithmetic or other expressions, the result may be `undefined`, causing errors under `strictNullChecks`.
- Optional chaining cannot be used on the left side of an assignment.

### Annotated Complete Step by Step Code Examples

#### Example 1: Optional property and element access

```typescript
// example_optional_chaining.ts
interface User {
  name: string;
  address?: {
    city: string;
    zip?: string;
  };
}

const user: User = { name: "Ada" };

// Safe property access.
console.log(user.address?.city); // Expected: undefined

// Safe element access.
const first: number | undefined = [1, 2, 3]?.[0];
console.log(first); // Expected: 1
```

**Expected Output:**
```
undefined
1
```

**Why this output occurs.** `user.address` is `undefined`, so `user.address?.city` short-circuits to `undefined`. The array is non-nullish, so `?.[0]` returns `1`. The `?.` checks only the immediate left side.

---

#### Example 2: Optional call

```typescript
// example_optional_call.ts
function makeRequest(log?: (msg: string) => void) {
  log?.("Request started"); // only calls if log is not nullish
  console.log("Done");
}

makeRequest(); // Expected: Done
makeRequest((msg) => console.log(msg)); // Expected: Request started, then Done
```

**Expected Output:**
```
Done
Request started
Done
```

**Why this output occurs.** When `log` is omitted, `log?.("Request started")` short-circuits and no call occurs. When provided, the function is called. The optional call only guards the call itself.

---

### Real-World Cases

- **API responses**: `response.data?.items?.[0]?.name`.
- **Optional callbacks**: `onSuccess?.(result)`.
- **DOM queries**: `document.querySelector?.("#id")?.textContent`.

---

## 5. Nullish Coalescing (`??`)

### Definitions

**Core Definition.** **Nullish coalescing (`??`)** is a logical operator that returns its **right-hand operand** when the left-hand operand is `null` or `undefined`, and otherwise returns the left-hand operand.

**Technical Definition.** Introduced in TypeScript 3.7 (and ES2020), `??` is a safer alternative to `||` for default values because it **only** treats `null` and `undefined` as “missing.” Other falsy values—`0`, `""`, `false`, `NaN`—are preserved. The operator short-circuits: the right-hand side is evaluated only when the left-hand side is nullish.

**Beginner-Friendly Explanation.** `??` means **“use the left side unless it is null or undefined; then use the right side.”** Unlike `||`, it does not discard `0` or empty strings. It is the correct way to provide defaults for potentially nullish values.

### Purposes

- **To provide defaults for nullish values** without losing valid falsy data.
- **To replace `||`** when `0`, `""`, or `false` are legitimate values.
- **To combine with optional chaining** for safe defaults: `obj?.prop ?? defaultValue`.
- **To handle missing configuration keys** precisely.

### Syntax Rules and Structure

**General syntax:**

```
value ?? defaultValue
```

**Component breakdown:**

- `value` — the potentially nullish expression.
- `??` — the nullish coalescing operator.
- `defaultValue` — the fallback used only when `value` is `null` or `undefined`.

**Syntax rules.**

- Only `null` and `undefined` trigger the fallback.
- `??` cannot be mixed with `||` or `&&` without parentheses.
- The right-hand side is evaluated **lazily** (only when needed).

**Constraints and limitations.**

- Requires ES2020 or a downleveling transform.
- Mixing `??` with `||` without parentheses is a syntax error.

### Annotated Complete Step by Step Code Examples

#### Example 1: `??` preserves falsy values

```typescript
// example_nullish_coalescing.ts
interface Config {
  name?: string;
  items?: number;
  active?: boolean;
}

function updateApp(config: Config) {
  config.name = config.name ?? "(no name)";
  config.items = config.items ?? -1;
  config.active = config.active ?? true;
}

const config: Config = { name: "", items: 0, active: false };
updateApp(config);
console.log(config); // Expected: { name: '', items: 0, active: false }
```

**Expected Output:**
```
{ name: '', items: 0, active: false }
```

**Why this output occurs.** `??` treats only `null` and `undefined` as missing. The empty string, `0`, and `false` are preserved. Using `||` instead would have replaced them with the defaults, corrupting valid data.

---

#### Example 2: Combining `?.` and `??`

```typescript
// example_optional_nullish.ts
interface User {
  settings?: { theme?: string };
}

const user: User = {};

const theme = user.settings?.theme ?? "light";
console.log(theme); // Expected: light
```

**Expected Output:**
```
light
```

**Why this output occurs.** `user.settings?.theme` is `undefined`, so `??` supplies `"light"`. This pattern is the idiomatic way to access a deeply optional value with a fallback.

---

### Real-World Cases

- **Configuration defaults**: `config.timeout ?? 5000`.
- **Optional props**: `user.nickname ?? user.name`.
- **API payloads**: `response.data ?? emptyData`.

---

## 6. Non-Null Assertion (`!`)

### Definitions

**Core Definition.** The **non-null assertion operator (`!`)** is a postfix operator that tells the compiler a value is **not `null` or `undefined`**, overriding `strictNullChecks` without any runtime proof.

**Technical Definition.** The `!` operator may be used to assert that its operand is non-null and non-undefined in contexts where the type checker is unable to conclude that fact. It is **purely a compile-time assertion**: the operator is **removed in the emitted JavaScript**, and no runtime check occurs. If the value is actually nullish, the program will crash at runtime—reintroducing exactly the errors `strictNullChecks` exists to prevent.

**Beginner-Friendly Explanation.** `!` means **“trust me, this is not null.”** TypeScript removes the null check from the type, but it does **not** verify anything at runtime. If you are wrong, your program crashes. It is a dangerous shortcut that should be used only when you have proof the compiler cannot see.

### Purposes

- **To silence null-check errors** when the developer has external proof of non-nullness (e.g., a preceding check the compiler cannot follow).
- **To access members of a value** that is typed nullable but is known to be present at a specific point.
- **As a last resort** during migration, with the intention of replacing it with proper narrowing.

### Syntax Rules and Structure

**General syntax:**

```
value!
value!.property
value!()
```

**Component breakdown:**

- `value!` — the non-null assertion.
- `!` — the postfix operator.

**Syntax rules.**

- `!` is removed during compilation; it emits **no JavaScript**.
- It does **not** perform a runtime check.
- It can be applied to expressions, not just variables.

**Constraints and limitations.**

- **Unsafe**: if the value is nullish, the program crashes.
- **Trades a compile error for a runtime error**, which is strictly worse for reliability.
- **ESLint `no-non-null-assertion`** and many style guides discourage or ban it.
- It should be replaced by a guard or an **assertion function** (`assertExists(value)`) that actually checks at runtime.

### Annotated Complete Step by Step Code Examples

#### Example 1: `!` bypasses null checking

```typescript
// example_non_null.ts
function greet(users: Map<string, { name: string }>, id: string): string {
  // ❌ Non-null assertion: crashes when id is absent.
  return `Hi ${users.get(id)!.name}`;
}

// ✅ Safe alternative: guard.
function greetSafe(users: Map<string, { name: string }>, id: string): string {
  const user = users.get(id);
  if (!user) return "Hi there";
  return `Hi ${user.name}`;
}

console.log(greetSafe(new Map([["u1", { name: "Ada" }]]), "u1"));
// Expected: Hi Ada
```

**Expected Output:**
```
Hi Ada
```

**Why this output occurs.** `greetSafe` narrows `user` with a guard before accessing `name`. The unsafe version would compile but throw when `id` is missing, because `!` does not check at runtime.

---

### Real-World Cases

- **Migration shortcuts**: temporarily silencing errors while porting JavaScript, with a plan to remove `!`.
- **Post-guard contexts**: after a check the compiler cannot follow across a helper boundary; prefer `assertExists` instead.
- **Test code**: asserting fixture data is present (still risky).

---

## 7. Null Safety Best Practices

### Definitions

**Core Definition.** **Null safety best practices** are guidelines for enabling and using TypeScript’s null-safety features in ways that **eliminate null-related runtime crashes** and avoid the false safety of the non-null assertion.

**Technical Definition.** Best practices center on **enabling `strictNullChecks` (and `strict`) as early as possible**, **preferring narrowing over assertions**, **using `??` instead of `||` for nullish defaults**, **using `?.` for optional access**, **keeping non-null assertions to an absolute minimum**, and **using assertion functions when a runtime check is needed**. `strictNullChecks` is described as the **highest-value flag** in a migration because it prevents the single most common JavaScript crash.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to make null safety actually work. The big rules are: turn on `strict`; never use `!` without proof; prefer `??` over `||`; and use `?.` instead of long `&&` chains.

### Purposes

- **To eliminate null pointer crashes** at compile time.
- **To avoid the false safety** of the non-null assertion.
- **To preserve valid falsy data** with `??` instead of `||`.
- **To make nullability explicit** in public APIs.
- **To migrate incrementally** without compromising safety.

### Syntax Rules and Structure

Best practices are conventions that interact with null-safety syntax. The core rules are:

1. **Enable `strict: true` (including `strictNullChecks`) early.** It is the highest-value flag for preventing runtime crashes. Sequence it immediately after `noImplicitAny` in migrations.
2. **Prefer narrowing over `!`.** Use guards (`if (!value) return`), optional chaining, and nullish coalescing. `!` should be rare and justified.
3. **Use `??` instead of `||` for nullish defaults.** `||` discards `0`, `""`, `false`, and `NaN`, which are often valid.
4. **Use `?.` for optional access.** It replaces verbose `&&` chains and only short-circuits on `null`/`undefined`.
5. **Use assertion functions instead of `!` when a runtime check is needed.** `assertExists(value)` both narrows the type and verifies at runtime.
6. **Make nullability explicit in APIs.** Return `T | null` or `T | undefined` rather than hiding nullable values behind `any`.
7. **Handle `undefined` from array access.** With `noUncheckedIndexedAccess`, indexed access returns `T | undefined`; check before use.

**Component breakdown (null-safety decision tree):**

```
Is the value possibly null or undefined?
  ├── Yes → Can you check it with a guard?
  │        ├── Yes → Narrow with if/early return.
  │        └── No → Can you use ?. and ?? safely?
  │                 ├── Yes → Use optional chaining and nullish coalescing.
  │                 └── No → Use an assertion function (assertExists).
  └── No → Access normally.
```

**Constraints and limitations.**

- Enabling `strictNullChecks` on legacy codebases can produce many errors; migrate incrementally.
- `??` and `?.` require ES2020 or downleveling.
- `!` remains available but should be treated as unsafe.

### Annotated Complete Step by Step Code Examples

#### Example 1: Narrow instead of `!`

```typescript
// example_best_narrow.ts
function greet(users: Map<string, { name: string }>, id: string): string {
  const user = users.get(id);
  if (!user) return "Hi there"; // guard narrows user to the object
  return `Hi ${user.name}`;
}

console.log(greet(new Map([["u1", { name: "Ada" }]]), "u1")); // Expected: Hi Ada
console.log(greet(new Map(), "missing"));                     // Expected: Hi there
```

**Expected Output:**
```
Hi Ada
Hi there
```

**Why this output occurs.** The guard proves non-nullness to both the compiler and the runtime. Unlike `!`, it does not crash when the user is missing.

---

#### Example 2: `??` over `||`

```typescript
// example_best_nullish.ts
interface Config {
  retries?: number;
}

function getRetries(config: Config): number {
  return config.retries ?? 3; // preserves 0
}

console.log(getRetries({ retries: 0 })); // Expected: 0
console.log(getRetries({}));             // Expected: 3
```

**Expected Output:**
```
0
3
```

**Why this output occurs.** `??` preserves `0` because it is not nullish. `||` would have returned `3`, discarding the valid zero value.

---

#### Example 3: Assertion function instead of `!`

```typescript
// example_best_assert.ts
function assertExists<T>(value: T | null | undefined): asserts value is T {
  if (value == null) throw new Error("Expected value to exist");
}

function process(value: string | null) {
  assertExists(value); // runtime check + narrowing
  console.log(value.toUpperCase()); // narrowed to string
}

process("hello"); // Expected: HELLO
```

**Expected Output:**
```
HELLO
```

**Why this output occurs.** The assertion function both checks at runtime and narrows the type. It is safer than `!` because it throws when the value is actually nullish.

---

### Real-World Cases

- **New projects**: `strict: true` in `tsconfig.json` from day one.
- **Migrations**: enabling `strictNullChecks` after `noImplicitAny` for the highest bug yield.
- **API clients**: returning `T | undefined` and handling it with `??` or guards.
- **Code review**: rejecting bare `!` in favor of guards or assertion functions.

---

## References

- TypeScript 2.0 Release Notes — Null- and undefined-aware types, `strictNullChecks` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0
- TypeScript 3.7 Release Notes — Optional Chaining - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#optional-chaining
- TypeScript 3.7 Release Notes — Nullish Coalescing - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#nullish-coalescing
- TypeScript 2.0 Release Notes — Non-null Assertion Operator - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html#non-null-assertion-operator
- TSConfig Reference — `strictNullChecks` - https://www.typescriptlang.org/tsconfig#strictNullChecks
- TSConfig Reference — `strict` - https://www.typescriptlang.org/tsconfig#strict
- TypeScript Playground — Nullish Coalescing Example - https://www.typescriptlang.org/play/3-7/syntax-and-messaging/nullish-coalescing.ts.html
- pproenca/dot-skills — Prioritize `strictNullChecks` for the Highest Bug Yield - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/migrate-js-to-modern-typescript/references/strict-prioritize-null-checks.md
- pproenca/dot-skills — Narrow Values Instead of Using the Non-Null Assertion - https://raw.githubusercontent.com/pproenca/dot-skills/refs/heads/master/skills/.experimental/migrate-js-to-modern-typescript/references/unsafe-narrow-instead-of-nonnull.md
- pproenca/dot-skills — Null Coalescing and Optional Chaining - https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.experimental/chrome-extension/references/err-null-coalescing.md
- TypeScript Handbook — Narrowing (`==`/`!=` null checks) - https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- TypeScript Handbook — More on Functions (Optional Parameters) - https://www.typescriptlang.org/docs/handbook/2/functions.html
- MDN — Optional Chaining (`?.`) - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining
- MDN — Nullish Coalescing Operator (`??`) - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing
- TypeScript ESLint — `no-non-null-assertion` Rule - https://typescript-eslint.io/rules/no-non-null-assertion/
- TypeScript 4.6 Release Notes — Control Flow Analysis for Destructured Discriminated Unions - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-6.html
- TypeScript 4.4 Release Notes — Control Flow Analysis of Aliased Conditions - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html