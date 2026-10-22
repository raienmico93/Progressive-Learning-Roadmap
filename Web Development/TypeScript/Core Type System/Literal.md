# Literal Types in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **literal type** is a type that denotes **exactly one specific value** of a primitive—a particular string, number, boolean, or bigint—rather than the general primitive type. Literal types are **subtypes** of their corresponding primitive types: `"hello"` is a `string`, but `string` is not `"hello"` inside the type system. They enable TypeScript to model **exact values** at the type level, supporting precise API contracts, configuration validation, and discriminated unions.

**Technical Definition.** TypeScript supports literal types for **strings**, **numbers**, **booleans**, and **bigints** (and, via `unique symbol`, for symbols). A literal type is written using the literal value itself in a type position: `type Easing = "ease-in" | "ease-out"`. Literal types are subject to **widening**: a `let` or `var` declaration widens a literal to its base primitive, while `const` preserves the literal type. The `as const` assertion (TS 3.4+) forces literal inference for objects, arrays, and primitives. Since TypeScript 4.1, **template literal types** build new string literal types from unions and patterns. Since TypeScript 5.0, **`const` type parameters** preserve literal types through generic function calls without requiring `as const` at every call site.

**Beginner-Friendly Explanation.** A literal type is **a type that means exactly one value**. Instead of saying “this is a string,” you say “this is the string `"ease-in"`.” It lets TypeScript spell-check your strings, restrict numbers to specific values, and model configuration options precisely. Think of a literal type as a **sealed label** that only matches one exact word or number—anything else is rejected.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Exact-value typing** | A literal type admits only one specific value, not the whole primitive category. |
| **Subtype of primitives** | `"hello"` is assignable to `string`; `string` is not assignable to `"hello"`. |
| **Widening with `let`/`var`** | Mutable bindings widen literals to their base types (`string`, `number`, etc.). |
| **Preservation with `const`** | `const` declarations preserve literal types automatically. |
| **`as const` assertion** | Forces literal inference for objects, arrays, and primitives (TS 3.4+). |
| **Template literal types** | Build new string literal types from unions and patterns (TS 4.1+). |
| **`const` type parameters** | Preserve literals through generic calls without `as const` (TS 5.0+). |
| **Zero runtime cost** | Literal types are erased at compile time and emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: type annotations, type aliases, and primitive types.
- **Union types** (`|`) and **intersection types** (`&`).
- **Control flow**: `switch`, `if`/`else`, and type narrowing.
- **Generics fundamentals**: type parameters and constraints.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Discriminated unions** — literal-typed discriminants enable automatic narrowing.
- **Configuration modeling** — literal unions model valid option sets.
- **Template literal types** — pattern-based string typing (CSS units, API routes, event names).
- **`as const` and `satisfies`** — preserving and validating literal precision.
- **Compiler performance** — large literal unions incur quadratic checking.

---

## Core Concepts / Features

---

## 1. String Literal Types

### Definitions

**Core Definition.** A **string literal type** is a type that permits **exactly one string value**, written as the string itself in a type position: `"hello"`. It is a subtype of `string`.

**Technical Definition.** String literal types combine with **union types**, **type guards**, and **type aliases** to provide enum-like behavior without runtime overhead. They are the foundation of **discriminated unions** and **template literal types**. String literals may use any valid JavaScript string syntax, including single quotes, double quotes, and escape sequences.

**Beginner-Friendly Explanation.** A string literal type says: **“This value must be exactly this string.”** `type Direction = "up" | "down"` means only `"up"` or `"down"` are allowed—nothing else.

### Purposes

- **To restrict a string to a fixed set of exact values**, catching typos at compile time.
- **To model enum-like closed sets** without runtime overhead.
- **To serve as discriminants** in discriminated unions.
- **To build template literal types** from string unions.
- **To document valid string options** in APIs and configuration.

### Syntax Rules and Structure

**General syntax:**

```
type LiteralName = "exact string";
type UnionName = "value1" | "value2" | "value3";
```

**Component breakdown:**

- `"exact string"` — the literal string value, used directly as a type.
- `|` — combines literals into a union.

**Syntax rules.**

- String literal types may use single or double quotes.
- They may include escape sequences and Unicode characters.
- They may be combined with `string` in a union to allow any string while preserving autocomplete: `"known" | (string & {})`.
- They are preserved by `const` and widened by `let`/`var`.

**Constraints and limitations.**

- A `let` variable widens the literal to `string` unless annotated or constrained.
- Very large literal unions (>12–50 members) cause quadratic type checking.
- String literals cannot be used as discriminants if the property is optional.

### Annotated Complete Step by Step Code Examples

#### Example 1: String literal with `const` vs. `let`

```typescript
// example_string_literal.ts
// Step 1: const preserves the literal type.
const helloWorld = "Hello World"; // type: "Hello World"

// Step 2: let widens to string.
let hiWorld = "Hi World"; // type: string

// Step 3: Explicit annotation preserves the literal.
let direction: "up" | "down" = "up";

console.log(helloWorld); // Expected: Hello World
console.log(hiWorld);    // Expected: Hi World
console.log(direction);  // Expected: up

// direction = "left"; // Error: "left" is not assignable to "up" | "down".
```

**Setup.** Compile with `npx tsc example_string_literal.ts --strict`.

**Expected Output:**
```
Hello World
Hi World
up
```

**Why this output occurs.** `const` preserves the literal type `"Hello World"` because the binding cannot change. `let` widens to `string` because reassignment is possible. The explicit union annotation restricts `direction` to two literals, rejecting `"left"`.

---

#### Example 2: String literal union for API options

```typescript
// example_string_union.ts
type Easing = "ease-in" | "ease-out" | "ease-in-out";

function animate(easing: Easing): void {
  console.log(`Animating with ${easing}`);
}

animate("ease-in"); // Expected: Animating with ease-in

// animate("linear"); // Error: "linear" is not assignable to Easing.
```

**Expected Output:**
```
Animating with ease-in
```

**Why this output occurs.** The `Easing` union restricts the parameter to three literal strings. `"linear"` is rejected at compile time. This is the canonical use case from the TypeScript Handbook.

---

### Real-World Cases

- **API options**: `type LogLevel = "debug" | "info" | "warn" | "error"`.
- **CSS values**: `type Alignment = "left" | "center" | "right"`.
- **Event names**: `type EventName = "click" | "focus" | "blur"`.
- **Status values**: `type Status = "idle" | "loading" | "success" | "error"`.

---

## 2. Number Literal Types

### Definitions

**Core Definition.** A **number literal type** is a type that permits **exactly one numeric value**, written as the number itself in a type position: `42`. It is a subtype of `number`.

**Technical Definition.** Number literal types may use decimal, hexadecimal, binary, and octal notation. They may be combined with unions and used as discriminants (e.g., `version: 0 | version: 1`). Like string literals, they are preserved by `const` and widened by `let`/`var`.

**Beginner-Friendly Explanation.** A number literal type says: **“This value must be exactly this number.”** `type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6` means only those six numbers are valid.

### Purposes

- **To restrict a number to a fixed set of exact values**.
- **To model numeric version discriminants** (`version: 0 | version: 1`).
- **To constrain numeric configuration** such as port numbers or status codes.
- **To serve as discriminants** in discriminated unions.

### Syntax Rules and Structure

**General syntax:**

```
type LiteralNumber = 42;
type UnionNumber = 1 | 2 | 3;
```

**Component breakdown:**

- `42` — the numeric literal, used directly as a type.
- `|` — combines literals.

**Syntax rules.**

- Number literal types support decimal, hex (`0x2A`), binary (`0b101010`), and octal (`0o52`) notation.
- They may be negative: `-1`.
- They may be combined with `number` to allow any number while preserving autocomplete.
- They are preserved by `const` and widened by `let`/`var`.

**Constraints and limitations.**

- Numeric literal types cannot represent `NaN`, `Infinity`, or `-0` distinctly.
- Large numeric unions still cause quadratic checking.

### Annotated Complete Step by Step Code Examples

#### Example 1: Number literal union

```typescript
// example_number_literal.ts
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;

function roll(result: DiceRoll): string {
  return `Rolled ${result}`;
}

console.log(roll(4)); // Expected: Rolled 4

// roll(7); // Error: 7 is not assignable to DiceRoll.
```

**Expected Output:**
```
Rolled 4
```

**Why this output occurs.** The `DiceRoll` union restricts the parameter to six numeric literals. `7` is rejected at compile time.

---

#### Example 2: Numeric version discriminant

```typescript
// example_number_discriminant.ts
type APIResponse =
  | { version: 0; msg: string }
  | { version: 1; message: string; status: number };

function handle(response: APIResponse) {
  if (response.version === 0) {
    console.log(response.msg); // narrowed to version 0
  } else {
    console.log(response.status, response.message); // narrowed to version 1
  }
}

handle({ version: 0, msg: "OK" }); // Expected: OK
```

**Expected Output:**
```
OK
```

**Why this output occurs.** The numeric literal discriminant `version` narrows the union in each branch, enabling access to variant-specific properties.

---

### Real-World Cases

- **HTTP status codes**: `type SuccessCode = 200 | 201 | 204`.
- **Numeric versions**: `{ version: 0 } | { version: 1 }`.
- **Dice/enum-like sets**: `type Suit = 0 | 1 | 2 | 3`.

---

## 3. Boolean Literal Types

### Definitions

**Core Definition.** A **boolean literal type** is a type that permits **exactly `true` or `false`** (or a union of the two). It is a subtype of `boolean`.

**Technical Definition.** Boolean literal types are written as `true` or `false` in type positions. A union `true | false` is equivalent to `boolean`, but keeping them separate allows **discriminated unions** on boolean discriminants. Boolean literals are preserved by `const` and widened by `let`/`var`.

**Beginner-Friendly Explanation.** A boolean literal type says: **“This value must be exactly `true` (or exactly `false`).”** It is useful for discriminated unions where the discriminant is a boolean flag.

### Purposes

- **To model a boolean discriminant** in a discriminated union.
- **To restrict a value to a specific boolean** in generic constraints.
- **To document intent** when a flag must be a specific value.

### Syntax Rules and Structure

**General syntax:**

```
type LiteralTrue = true;
type LiteralFalse = false;
type Both = true | false; // same as boolean
```

**Component breakdown:**

- `true` — the boolean literal type for `true`.
- `false` — the boolean literal type for `false`.
- `true | false` — equivalent to `boolean`.

**Syntax rules.**

- Boolean literal types are written as `true` and `false`.
- They are preserved by `const` and widened by `let`/`var`.
- They may be used as discriminants in unions.

**Constraints and limitations.**

- A union `true | false` collapses to `boolean`, losing discrimination.
- Boolean literals are less commonly used than string and number literals.

### Annotated Complete Step by Step Code Examples

#### Example 1: Boolean literal with `as const`

```typescript
// example_boolean_literal.ts
const isEnabled = true; // type: boolean (widened for const? Actually const preserves literal)
// Actually: const isEnabled = true; // type: true

let flag = true; // type: boolean

const literalFlag = true as const; // type: true

console.log(isEnabled);  // Expected: true
console.log(flag);       // Expected: true
console.log(literalFlag); // Expected: true
```

**Expected Output:**
```
true
true
true
```

**Why this output occurs.** `const isEnabled = true` infers the literal type `true` because `const` preserves literals. `let flag = true` widens to `boolean`. The `as const` assertion forces the literal type explicitly.

---

#### Example 2: Boolean discriminant

```typescript
// example_boolean_discriminant.ts
type Success = { success: true; data: string };
type Failure = { success: false; error: string };

type Result = Success | Failure;

function handle(result: Result) {
  if (result.success) {
    console.log(result.data); // narrowed to Success
  } else {
    console.log(result.error); // narrowed to Failure
  }
}

handle({ success: true, data: "OK" }); // Expected: OK
```

**Expected Output:**
```
OK
```

**Why this output occurs.** The boolean literal discriminant `success` narrows the union in each branch, enabling access to `data` or `error` respectively.

---

### Real-World Cases

- **Result types**: `{ success: true; data: T } | { success: false; error: E }`.
- **Feature flags** with literal `true` constraints in generic APIs.
- **Conditional configuration** where a flag must be `true`.

---

## 4. BigInt Literal Types

### Definitions

**Core Definition.** A **bigint literal type** is a type that permits **exactly one bigint value**, written as the value with an `n` suffix in a type position: `100n`. It is a subtype of `bigint`.

**Technical Definition.** BigInt literal types are supported for arbitrary-precision integers. They may be combined with unions and used in the same ways as number literals. BigInts require an ES2020+ target for runtime support.

**Beginner-Friendly Explanation.** A bigint literal type says: **“This value must be exactly this big integer.”** It is useful for very large numeric constants that exceed `number` precision.

### Purposes

- **To restrict a bigint to exact values**.
- **To model large numeric constants** in configuration.
- **To combine with unions** for precise bigint options.

### Syntax Rules and Structure

**General syntax:**

```
type LiteralBigInt = 100n;
type UnionBigInt = 100n | 200n;
```

**Component breakdown:**

- `100n` — the bigint literal, used directly as a type.
- `|` — combines bigint literals.

**Syntax rules.**

- The `n` suffix is required for bigint literals.
- BigInt literal types are preserved by `const` and widened by `let`/`var`.
- They require an ES2020+ target.

**Constraints and limitations.**

- BigInts cannot be mixed with `number` in arithmetic without explicit conversion.
- BigInt literals are less commonly used than string and number literals.

### Annotated Complete Step by Step Code Examples

#### Example 1: BigInt literal type

```typescript
// example_bigint_literal.ts
type LargeID = 9007199254740993n; // > Number.MAX_SAFE_INTEGER

const id: LargeID = 9007199254740993n;
console.log(id); // Expected: 9007199254740993n

// const bad: LargeID = 9007199254740994n;
// Error: 9007199254740994n is not assignable to 9007199254740993n.
```

**Expected Output:**
```
9007199254740993n
```

**Why this output occurs.** The bigint literal type permits exactly the specified bigint value. A different bigint is rejected at compile time.

---

### Real-World Cases

- **Large numeric IDs**: Twitter snowflake IDs, database bigints.
- **Cryptographic constants** that exceed `number` precision.
- **Precise large-value configuration**.

---

## 5. Template Literal Types

### Definitions

**Core Definition.** A **template literal type** is a type-level construct with the same syntax as JavaScript template literals, used in **type positions** to build new string literal types from **unions and patterns**. It can expand into many strings via unions.

**Technical Definition.** Template literal types were introduced in TypeScript 4.1. They interpolate types—including literal types and unions—into a string pattern. When a union is used in an interpolated position, the type expands to **every possible string** that could be represented by each union member. TypeScript provides four intrinsic string manipulation types: `Uppercase`, `Lowercase`, `Capitalize`, and `Uncapitalize`.

**Beginner-Friendly Explanation.** A template literal type is **a string pattern at the type level**. `` `hello ${World}` `` produces `"hello world"`. When used with unions, it generates every combination. It is how you type things like CSS units (`` `${number}px` ``), event names (`` `on${Capitalize<T>}` ``), and API routes (`` `/api/${string}` ``).

### Purposes

- **To build new string literal types** from existing literals and unions.
- **To model string patterns** such as CSS units, URLs, and email addresses.
- **To generate event-handler names** with capitalization transformations.
- **To type API routes and path parameters**.
- **To cross-multiply unions** into precise string sets.

### Syntax Rules and Structure

**General syntax:**

```
type TemplateLiteral = `prefix${Type}suffix`;
```

**Component breakdown:**

- `` ` `` — backtick delimiters (same as JavaScript template literals).
- `${Type}` — an interpolated type position.
- Surrounding text — literal strings.

**Syntax rules.**

- Template literal types use **backticks**, not quotes.
- Interpolated types may be `string`, `number`, `bigint`, `boolean`, `null`, `undefined`, or literal/union types.
- Unions in interpolated positions are **cross-multiplied**.
- Intrinsic types (`Uppercase`, `Lowercase`, `Capitalize`, `Uncapitalize`) transform string literals.
- Template literal types may be used as **index signatures** since TS 4.4.

**Constraints and limitations.**

- Very large cross-multiplications can cause compiler performance issues.
- Template literal types cannot produce infinitely many strings; use `string` for unbounded patterns.
- Intrinsic string manipulation types only work on literal types.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic template literal type

```typescript
// example_template_basic.ts
type World = "world";
type Greeting = `hello ${World}`; // type: "hello world"

const greeting: Greeting = "hello world";
console.log(greeting); // Expected: hello world

// const bad: Greeting = "hello there"; // Error
```

**Expected Output:**
```
hello world
```

**Why this output occurs.** The template literal type concatenates `"hello "` with the literal type `"world"`, producing exactly `"hello world"`.

---

#### Example 2: Union cross-multiplication

```typescript
// example_template_union.ts
type Color = "red" | "blue";
type Quantity = "one" | "two";

type SeussFish = `${Quantity | Color} fish`;
// Expands to: "one fish" | "two fish" | "red fish" | "blue fish"

const fish: SeussFish = "red fish";
console.log(fish); // Expected: red fish
```

**Expected Output:**
```
red fish
```

**Why this output occurs.** The union in the interpolated position expands to **every possible combination** of `Quantity | Color` with `" fish"`.

---

#### Example 3: Intrinsic string manipulation

```typescript
// example_template_intrinsic.ts
type EventName<T extends string> = `on${Capitalize<T>}`;

type ClickEvent = EventName<"click">; // "onClick"
type ChangeEvent = EventName<"change">; // "onChange"

const handler: ClickEvent = "onClick";
console.log(handler); // Expected: onClick
```

**Expected Output:**
```
onClick
```

**Why this output occurs.** `Capitalize<"click">` produces `"Click"`, and the template literal prepends `"on"`, yielding `"onClick"`.

---

#### Example 4: CSS unit pattern

```typescript
// example_template_css.ts
type CSSUnit = `${number}px` | `${number}%` | `${number}em`;

const width: CSSUnit = "100px";
const height: CSSUnit = "50%";

console.log(width);  // Expected: 100px
console.log(height); // Expected: 50%
```

**Expected Output:**
```
100px
50%
```

**Why this output occurs.** The template literal types match strings that start with a number and end with `px`, `%`, or `em`.

---

### Real-World Cases

- **CSS-in-JS**: `` type CSSUnit = `${number}px` | `${number}rem` ``.
- **Event names**: `` type HandlerName<T> = `on${Capitalize<T>}` ``.
- **API routes**: `` type ApiRoute = `/api/${string}` ``.
- **Email validation**: `` type Email = `${string}@${string}.${string}` ``.
- **Index signatures**: `` interface DataAttributes { [key: `data-${string}`]: string } ``.

---

## 6. Literal Unions

### Definitions

**Core Definition.** A **literal union** is a union type composed entirely of **literal types**, representing a **closed set** of permitted exact values. It is the primary way to model enum-like choices in TypeScript.

**Technical Definition.** Literal unions combine string, number, boolean, or bigint literal types with `|`. They are **erased** at runtime and have **zero runtime cost**, unlike numeric `enum`s which emit runtime objects. They support **exhaustiveness checking** with `never` and serve as **discriminants** in discriminated unions. Very large literal unions (>12–50 members) cause **quadratic type checking**.

**Beginner-Friendly Explanation.** A literal union is **a list of allowed exact values**. `type Status = "idle" | "loading" | "success" | "error"` means only those four strings are valid. It is like an enum, but simpler and with no runtime overhead.

### Purposes

- **To model a closed set of named values** without runtime overhead.
- **To enable exhaustive checking** of all variants.
- **To serve as discriminants** in discriminated unions.
- **To replace `enum`** for simple string-based sets.
- **To restrict configuration options** to valid literals.

### Syntax Rules and Structure

**General syntax:**

```
type UnionName = "value1" | "value2" | "value3";
```

**Component breakdown:**

- `"value1"`, `"value2"`, `"value3"` — literal types.
- `|` — union operator.

**Syntax rules.**

- Literal unions may mix string, number, and boolean literals (though mixing types is unusual).
- They may be **exhaustively checked** with `switch` and `never`.
- They may be combined with `string` to allow any string while preserving autocomplete: `"known" | (string & {})`.
- They are preferred over `enum` for **closed sets of strings** because they have zero runtime cost.

**Constraints and limitations.**

- **Large unions** (>12–50 members) cause quadratic checking; group them into discriminated categories.
- Literal unions cannot be iterated at runtime without a separate runtime array.
- They cannot be used with `instanceof` or runtime reflection.

### Annotated Complete Step by Step Code Examples

#### Example 1: Literal union with exhaustive switch

```typescript
// example_literal_union.ts
type Status = "idle" | "loading" | "success" | "error";

function assertNever(value: never): never {
  throw new Error(`Unhandled: ${value}`);
}

function handle(status: Status): string {
  switch (status) {
    case "idle": return "Ready";
    case "loading": return "Loading...";
    case "success": return "Done";
    case "error": return "Failed";
    default: return assertNever(status);
  }
}

console.log(handle("idle")); // Expected: Ready
```

**Expected Output:**
```
Ready
```

**Why this output occurs.** The literal union restricts `status` to four values. The `switch` handles all four, and the `default` branch receives `never`, ensuring exhaustiveness. Adding a new status without handling it causes a compile-time error.

---

#### Example 2: Literal union with `satisfies` for configuration

```typescript
// example_literal_union_satisfies.ts
type Config = Record<string, string | number>;

const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
} satisfies Config;

// config.apiUrl is "https://api.example.com" (literal preserved)
console.log(config.apiUrl); // Expected: https://api.example.com
```

**Expected Output:**
```
https://api.example.com
```

**Why this output occurs.** `satisfies` validates the object against `Config` without widening the literal types. `config.apiUrl` remains the literal type `"https://api.example.com"` rather than `string`.

---

### Real-World Cases

- **Status enums**: `type Status = "idle" | "loading" | "success" | "error"`.
- **Log levels**: `type Level = "debug" | "info" | "warn" | "error"`.
- **Sort orders**: `type Order = "asc" | "desc"`.
- **Theme modes**: `type Theme = "light" | "dark" | "system"`.

---

## 7. Configuration Modeling

### Definitions

**Core Definition.** **Configuration modeling with literal types** is the practice of using literal unions, `as const`, and `satisfies` to type configuration objects so that **valid values are enforced** while **literal precision is preserved** for downstream inference.

**Technical Definition.** Configuration objects are typically declared with `as const` to preserve literal types and `readonly` properties, or with `satisfies` to validate against a type without widening. Literal unions model valid option values. The `const` type parameter (TS 5.0+) preserves literals through generic configuration factories without requiring `as const` at every call site.

**Beginner-Friendly Explanation.** Configuration modeling means **typing your config objects so that only valid values are allowed and the exact values are remembered**. You use literal unions for the allowed values, `as const` to keep the exact values instead of widening to `string`, and `satisfies` to check the shape without losing precision.

### Purposes

- **To enforce valid configuration values** at compile time.
- **To preserve literal types** for downstream inference and autocomplete.
- **To validate configuration shape** without widening types.
- **To model environment-specific configuration** with literal discriminants.
- **To avoid runtime configuration errors** by catching invalid values before execution.

### Syntax Rules and Structure

**Pattern 1 — `as const` for preservation:**

```
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
} as const;
```

**Pattern 2 — `satisfies` for validation without widening:**

```
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
} satisfies Config;
```

**Pattern 3 — `const` type parameter for generic factories:**

```
function createConfig<const T extends Config>(config: T): T {
  return config;
}
```

**Component breakdown:**

- `as const` — infers literal types and makes properties `readonly`.
- `satisfies` — validates against a type while preserving the inferred literal types.
- `const T` — preserves literals through generic function calls (TS 5.0+).

**Syntax rules.**

- `as const` applies to objects, arrays, and primitives.
- `satisfies` was introduced in TypeScript 4.9.
- `const` type parameters were introduced in TypeScript 5.0.
- `satisfies` is preferred over `as const` when you need **validation** without **widening**.

**Constraints and limitations.**

- `as const` makes all properties `readonly`, which may be undesirable for mutable configuration.
- `satisfies` does not make properties `readonly`.
- `const` type parameters require TypeScript 5.0+.

### Annotated Complete Step by Step Code Examples

#### Example 1: `as const` for literal preservation

```typescript
// example_config_as_const.ts
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
} as const;

// config.apiUrl: "https://api.example.com" (literal)
// config.timeout: 5000 (literal)
// config.retries: 3 (literal)

console.log(config.apiUrl); // Expected: https://api.example.com

// config.apiUrl = "https://other.com"; // Error: readonly
```

**Expected Output:**
```
https://api.example.com
```

**Why this output occurs.** `as const` preserves the literal types and makes properties `readonly`, preventing reassignment and enabling precise downstream inference.

---

#### Example 2: `satisfies` for validation without widening

```typescript
// example_config_satisfies.ts
interface Config {
  apiUrl: string;
  timeout: number;
  debug?: boolean;
}

const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
} satisfies Config;

// config.apiUrl remains the literal type "https://api.example.com"
console.log(config.apiUrl); // Expected: https://api.example.com

// const bad = { apiUrl: 123, timeout: 5000 } satisfies Config;
// Error: number not assignable to string
```

**Expected Output:**
```
https://api.example.com
```

**Why this output occurs.** `satisfies Config` validates the shape without widening `apiUrl` to `string`. The literal type is preserved, which is valuable for downstream generic inference.

---

#### Example 3: `const` type parameter for configuration factory

```typescript
// example_config_const_param.ts
function createConfig<const T extends Record<string, unknown>>(config: T): T {
  return config;
}

const config = createConfig({
  apiUrl: "https://api.example.com",
  timeout: 5000,
});

// config.apiUrl: "https://api.example.com" (literal preserved automatically)
console.log(config.apiUrl); // Expected: https://api.example.com
```

**Expected Output:**
```
https://api.example.com
```

**Why this output occurs.** The `const T` type parameter tells TypeScript to infer the **narrowest** type for `T`, preserving literal types without requiring `as const` at the call site.

---

### Real-World Cases

- **Environment configuration**: `const env = { NODE_ENV: "production" } as const`.
- **Feature flags**: `const flags = { darkMode: true, newUI: false } satisfies FeatureFlags`.
- **API client configuration**: `const client = createClient({ baseUrl: "/api", retries: 3 })` with `const T` preserving literals.
- **Theme configuration**: `const theme = { mode: "dark" } as const` for precise downstream typing.

---

## 8. Literal Type Best Practices

### Definitions

**Core Definition.** **Literal type best practices** are guidelines for using literal types, literal unions, template literal types, `as const`, `satisfies`, and `const` type parameters in ways that maximize **type safety, compiler performance, and maintainability**, while avoiding widening losses, excessive union sizes, and over-annotation.

**Technical Definition.** Best practices center on **preserving literal types with `as const` or `const` type parameters**, **using `satisfies` for validation without widening**, **preferring literal unions over `enum` for closed string sets**, **avoiding large flat literal unions** (>12–50 members cause quadratic checking), **using template literal types for patterned strings**, and **not overusing literal types where base primitives suffice**. The `const` type parameter (TS 5.0+) should be used when a function's return type depends on the literal value; otherwise, plain `T extends string` is sufficient.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to get the most out of literal types. The big rules are: use `as const` or `const T` to keep exact values; use `satisfies` to validate without losing precision; prefer literal unions over enums for simple closed sets; do not create enormous literal unions; and do not add literal precision where it does not help.

### Purposes

- **To preserve literal types** through inference and generic calls.
- **To validate configuration** without widening exact values.
- **To maximize compiler performance** by avoiding large unions and unnecessary literal types.
- **To choose the right tool** between `enum`, literal union, `as const`, and `satisfies`.
- **To keep code readable and maintainable** with clear literal contracts.

### Syntax Rules and Structure

Best practices are conventions that interact with literal type syntax. The core rules are:

1. **Preserve literals with `as const` or `const T`.** Use `as const` for object literals; use `const` type parameters for generic functions (TS 5.0+). Use `satisfies` when you need validation without widening.
2. **Prefer literal unions over `enum` for closed string sets.** Literal unions have zero runtime cost, serialize as themselves, and tree-shake better than numeric enums.
3. **Avoid large flat literal unions.** Unions with >12 elements cause quadratic checking; >50 members cause measurable slowdowns. Group into discriminated categories or use branded types.
4. **Use template literal types for patterned strings.** CSS units, event names, API routes, and email formats are ideal candidates.
5. **Do not overuse literal types.** If a function's return type does not depend on the literal value, `T extends string` is sufficient; `const T` adds noise.
6. **Use `satisfies` instead of `as` for configuration.** `satisfies` validates without asserting, preserving safety and literal precision.
7. **Use literal unions as discriminants** in discriminated unions for exhaustive narrowing.
8. **Avoid widening with type annotations** when literal precision matters. Annotating `const config: Record<string, string>` widens literals; use `satisfies` instead.

**Component breakdown (literal type decision tree):**

```
Do you need a closed set of exact values?
  ├── Yes → Use a literal union.
  │        ├── Are the values strings? → Prefer literal union over enum.
  │        ├── Do you need patterns? → Use template literal types.
  │        └── Is the union >12 members? → Group into categories.
  └── No → Do you need to preserve literals through inference?
           ├── Yes → Use as const (objects) or const T (generics).
           └── No → Use base primitives (string, number, boolean).
```

**Constraints and limitations.**

- `as const` makes all properties `readonly`, which may not be desired.
- `const` type parameters require TypeScript 5.0+.
- Template literal types with large unions can explode combinatorially and slow compilation.
- Literal unions are erased at runtime; if you need runtime iteration, use an array `as const`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Preserving literals with `satisfies`

```typescript
// example_best_satisfies.ts
// ❌ Annotation widens literal types.
const wideConfig: Record<string, string | number> = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
};
// wideConfig.apiUrl is string | number (too wide)

// ✅ satisfies validates without widening.
const narrowConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
} satisfies Record<string, string | number>;

// narrowConfig.apiUrl is "https://api.example.com"
console.log(narrowConfig.apiUrl); // Expected: https://api.example.com
```

**Expected Output:**
```
https://api.example.com
```

**Why this output occurs.** `satisfies` checks the object against the type but preserves the inferred literal types. The annotated version widens `apiUrl` to `string | number`, losing precision.

---

#### Example 2: Prefer literal union over enum for string sets

```typescript
// example_best_literal_over_enum.ts
// ❌ Enum: runtime object, less tree-shakeable.
// enum Status { Idle = "idle", Loading = "loading", Success = "success" }

// ✅ Literal union: zero runtime cost, better tree-shaking.
type Status = "idle" | "loading" | "success";

function handle(status: Status): void {
  console.log(status);
}

handle("loading"); // Expected: loading
```

**Expected Output:**
```
loading
```

**Why this output occurs.** The literal union has **zero runtime cost** and is preferred for closed sets of strings. The `enum` version would emit a runtime object.

---

#### Example 3: Avoid large flat literal unions

```typescript
// example_best_large_union.ts
// ❌ 200+ flat literal members cause quadratic checking.
// type Event = "user_login" | "user_logout" | ... (200+ members);

// ✅ Group into discriminated categories.
type UserEvent = { category: "user"; action: "login" | "logout" | "signup" };
type PageEvent = { category: "page"; action: "view" | "scroll" | "leave" };

type AppEvent = UserEvent | PageEvent;

function handle(event: AppEvent): void {
  switch (event.category) {
    case "user": console.log(event.action); break;
    case "page": console.log(event.action); break;
  }
}

handle({ category: "user", action: "login" }); // Expected: login
```

**Expected Output:**
```
login
```

**Why this output occurs.** The outer union has only **two members**, avoiding quadratic checking. Each category narrows to its specific action union. This pattern preserves type safety while keeping compiler performance reasonable.

---

#### Example 4: `const T` only when return type depends on the literal

```typescript
// example_best_const_param.ts
// ✅ const T is appropriate: return type depends on the literal.
function route<const T extends string>(path: T): { path: T } {
  return { path };
}

const r = route("/users"); // type: { path: "/users" }
console.log(r.path); // Expected: /users

// ❌ const T is unnecessary when return type does not depend on the literal.
function log<T extends string>(message: T): void {
  console.log(message);
}
log("hello"); // Expected: hello
```

**Expected Output:**
```
/users
hello
```

**Why this output occurs.** `const T` preserves the literal `"/users"` in the return type. For `log`, the return type is `void`, so `const T` would add noise without benefit.

---

### Real-World Cases

- **Configuration validation**: `satisfies` for config objects that must match a schema but preserve literal precision.
- **Closed-set APIs**: literal unions for log levels, status codes, and sort orders.
- **Patterned strings**: template literal types for CSS units, routes, and event names.
- **Generic factories**: `const T` for route builders, query builders, and schema definitions.

---

## References

- TypeScript Handbook — Literal Types (Deprecated, replaced by Everyday Types) - https://www.typescriptlang.org/docs/handbook/literal-types.html
- TypeScript Handbook — Everyday Types: Literal Types - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types
- TypeScript Handbook — Template Literal Types - https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- TypeScript 4.1 Release Notes — Template Literal Types - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#template-literal-types
- TypeScript 4.9 Release Notes — `satisfies` Operator - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html
- TypeScript 5.0 Release Notes — `const` Type Parameters - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html#const-type-parameters
- TypeScript 3.4 Release Notes — `const` Assertions - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions
- TypeScript Handbook — Basic Types (BigInt) - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#bigint
- Total TypeScript — Const Type Parameters - https://www.totaltypescript.com/const-type-parameters
- OneUptime — How to Create Const Assertions in TypeScript - https://github.com/OneUptime/blog/blob/master/posts/2026-01-30-typescript-const-assertions/README.md
- Steve Kinney — Template Literal Types - https://stevekinney.com/courses/react-typescript/template-literal-types
- Ashby — Flexible Types That Support Autocomplete with Template Literals - https://www.ashbyhq.com/blog/engineering/flexible-types-with-template-literals
- pproenca/dot-skills — Use `const T` to Preserve Literals Through Overloaded APIs - https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/typescript-advanced-patterns/references/mod-const-type-params-overloads.md
- pproenca/dot-skills — Avoid Large Union Types - https://github.com/pproenca/dot-skills/blob/master/skills/.curated/typescript/references/type-avoid-large-unions.md