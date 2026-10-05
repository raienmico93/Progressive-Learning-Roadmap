# TypeScript Template Literal Types: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Template literal types are a TypeScript type-level feature that mirrors JavaScript template literal syntax but operates on types instead of values. They enable the creation, transformation, and validation of string types at compile time, allowing developers to model complex string patterns, generate derived type unions, and enforce naming conventions without runtime checks.

**Technical Definition**
Template literal types, introduced in TypeScript 4.1, build on string literal types and use the same syntax as JavaScript template literals (backticks and `${}` interpolation) but in type positions. When concrete literal types are used in interpolation positions, the template literal produces a new string literal type by concatenating contents. When unions are used in interpolation positions, the type expands into the set of all possible string literals represented by each union member—the unions are cross-multiplied. The feature includes four intrinsic string manipulation utility types (`Uppercase`, `Lowercase`, `Capitalize`, `Uncapitalize`) that transform string literal types. Template literal types integrate with `infer` for pattern extraction and with mapped types for key remapping, enabling advanced type-level string processing.

**Beginner-Friendly Explanation**
Template literal types let you use template literal syntax (backticks and `${}`) to create and manipulate string types. Instead of saying "this is a string," you can say "this is a string that starts with `user-` and ends with a number." For example, `` `user-${number}` `` describes any string matching that pattern. When you use unions in the interpolation, TypeScript automatically generates all possible combinations: `` `${"a" | "b"}-${"x" | "y"}` `` produces `"a-x" | "a-y" | "b-x" | "b-y"`. You can also transform strings with built-in utilities like `Capitalize` and `Uppercase`, so `` `on${Capitalize<"click">}` `` gives you `"onClick"`. This is incredibly powerful for creating type-safe event names, API routes, CSS values, and any string pattern that follows a convention.

### Key Characteristics

- **Template literal syntax**: Uses backticks and `${}` interpolation in type positions, mirroring JavaScript template literals.
- **Union cross-multiplication**: Unions in interpolation positions expand into the Cartesian product of all possible strings.
- **Intrinsic string utilities**: Four built-in type-level string transformations (`Uppercase`, `Lowercase`, `Capitalize`, `Uncapitalize`).
- **Pattern extraction**: Combines with `infer` to extract substrings from patterns (e.g., route parameters).
- **Mapped type integration**: Enables key remapping via the `as` clause for generating derived property names.
- **Combinatorial explosion risk**: Cross-multiplication of large unions can produce exponentially large types, hitting compiler limits.
- **Compile-time only**: Template literal types are erased at runtime and exist only during type checking.

### Prerequisites

- Basic knowledge of TypeScript string literal types and union types
- Familiarity with generics and type parameters
- Understanding of conditional types and `infer` (for pattern extraction)
- Familiarity with mapped types (for key remapping)

### Related Programming Areas

- **Type-Level Programming**: Template literal types are a core tool for type-level string manipulation
- **String Literal Types**: Template literal types extend and build on string literal types
- **Conditional Types**: `infer` within conditional types enables pattern extraction from template literals
- **Mapped Types**: Key remapping with `as` clauses uses template literal types for property name transformation
- **Compiler Performance**: Cross-multiplication of unions can cause exponential type-checking time

### Core Concepts / Features

1. Template Literal Type Syntax and String Literal Interpolation
2. String Composition, Combinatorial Explosions, and Compiler Limits
3. Pattern Modeling and Validation (Hex Codes, Dates)
4. Event-Name Types (`on${Capitalize<Event>}`)
5. Route and API-Path Types (Parsing Parameters from Strings)
6. Intrinsic String Manipulation Utilities (`Uppercase`, `Lowercase`, `Capitalize`, `Uncapitalize`)


## 1. Template Literal Type Syntax and String Literal Interpolation

### Definitions

**Core Definition**
Template literal type syntax uses backticks and `${}` interpolation in type positions. When concrete literal types are used in interpolation positions, a new string literal type is produced by concatenating the contents. When unions are used, the type expands into the set of all possible strings.

**Technical Definition**
The template literal type syntax is `` `prefix${Interpolated}suffix` ``, where `Interpolated` is a type (string literal, union of string literals, `string`, `number`, `bigint`, `boolean`, or `null`/`undefined`). When all interpolated positions contain concrete literal types, the result is a single string literal type. When any position contains a union, the result is a union of all possible combinations—the cross product of the unions. TypeScript 4.1 introduced this feature as part of the "Template Literal Types" release. The feature supports `string`, `number`, `bigint`, `boolean`, `null`, and `undefined` as interpolated types. Template literal types can be nested and combined with conditional types.

**Beginner-Friendly Explanation**
Template literal type syntax looks exactly like JavaScript template literals, but you use it in type positions. Instead of `` `Hello, ${name}` `` (a value), you write `` type Greeting = `Hello, ${string}` `` (a type). The `${}` part is where you put a type. If you put a concrete string like `"world"`, you get the single literal type `` `Hello, world` ``. If you put a union like `"world" | "there"`, you get a union of both possibilities: `` `Hello, world` | `Hello, there` ``. This lets you generate precise string types from simpler building blocks, which is the foundation for all the advanced patterns in this cheat sheet.

### Purposes

- To create new string literal types by concatenating or interpolating existing types.
- To model string patterns that follow a specific format or convention.
- To generate unions of string literals from unions of constituent parts.
- To serve as the foundation for advanced patterns like event names and route paths.
- To enable type-level string manipulation without runtime validation.

### Syntax Rules and Structure

**General Syntax: Basic Template Literal Type**

```typescript
type Greeting = `Hello, ${string}`;
// Matches any string starting with "Hello, "
```

**Component Breakdown**
- `` `Hello, ${string}` ``: The template literal type.
- `Hello, `: Literal prefix text.
- `${string}`: The interpolated type (any string).

**General Syntax: Concrete Literal Interpolation**

```typescript
type World = "world";
type Greeting = `Hello, ${World}`;
// "Hello, world"
```

**Component Breakdown**
- `World` is a concrete literal type.
- The result is a single string literal type.

**General Syntax: Union Interpolation**

```typescript
type Color = "red" | "blue";
type Quantity = "one" | "two";
type SeussFish = `${Quantity | Color} fish`;
// "one fish" | "two fish" | "red fish" | "blue fish"
```

**Component Breakdown**
- Unions are cross-multiplied.
- Each combination produces a separate string literal.

**General Syntax: Multi-Position Interpolation**

```typescript
type Lang = "en" | "ja" | "pt";
type LocaleIDs = "welcome_email" | "email_heading";
type LocaleMessageIDs = `${Lang}_${LocaleIDs}`;
// "en_welcome_email" | "en_email_heading" | "ja_welcome_email" | ...
```

**Component Breakdown**
- Each interpolated position contributes to the cross product.
- The result is the Cartesian product of all positions.

**Syntax Rules**

- Template literal types use backticks and `${}` in type positions.
- Interpolated types can be `string`, `number`, `bigint`, `boolean`, `null`, `undefined`, or literal types.
- Concrete literal interpolation produces a single string literal type.
- Union interpolation produces a union of all possible combinations (cross product).
- Multiple interpolated positions multiply the combinations.
- Template literal types can be nested.
- Template literal types work with `infer` for pattern extraction.

**Constraints and Limitations**

- Template literal types are erased at runtime; they exist only at compile time.
- Large unions produce exponentially large types (see Section 2).
- Template literal types cannot contain runtime expressions.
- The interpolated type must be assignable to `string | number | bigint | boolean | null | undefined`.
- TypeScript 4.1 or later is required.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Template Literal Types

```typescript
// Step 1: Define a simple template literal type.
type Greeting = `Hello, ${string}`;

const greeting1: Greeting = "Hello, world";  // ✅
const greeting2: Greeting = "Hello, TypeScript";  // ✅
// const greeting3: Greeting = "Hi, world";  // ❌ Error: Does not match pattern.

console.log(greeting1);  // "Hello, world"

// Step 2: Interpolate a concrete literal.
type World = "world";
type SpecificGreeting = `Hello, ${World}`;  // "Hello, world"

const specific: SpecificGreeting = "Hello, world";  // ✅
// const wrong: SpecificGreeting = "Hello, there";  // ❌ Error.

console.log(specific);  // "Hello, world"

// Step 3: Interpolate a union.
type Direction = "top" | "bottom";
type Alignment = "left" | "right";
type Position = `${Direction}-${Alignment}`;
// "top-left" | "top-right" | "bottom-left" | "bottom-right"

const pos: Position = "top-left";  // ✅
console.log(pos);  // "top-left"

// Step 4: Use in a function.
function setPosition(position: Position): void {
  console.log(`Setting position to ${position}`);
}

setPosition("bottom-right");  // "Setting position to bottom-right"
// setPosition("middle-center");  // ❌ Error: Not a valid position.

console.log("Template literal types complete.");
```

**Expected Output:**
```
Hello, world
Hello, world
top-left
Setting position to bottom-right
Template literal types complete.
```

**Why This Output Occurs:** The `Greeting` type matches any string starting with `"Hello, "`. `SpecificGreeting` is exactly `"Hello, world"`. The `Position` type is a union of four possible strings generated by cross-multiplying the two unions. The function enforces that only valid positions are passed.

#### Example 2: Cross-Multiplication with Multiple Positions

```typescript
// Step 1: Define unions for each position.
type Size = "small" | "medium" | "large";
type Color = "red" | "blue" | "green";
type Material = "cotton" | "wool";

// Step 2: Create a product type with cross-multiplied combinations.
type ProductSKU = `${Size}-${Color}-${Material}`;
// 3 × 3 × 2 = 18 possible combinations

const sku1: ProductSKU = "small-red-cotton";  // ✅
const sku2: ProductSKU = "large-blue-wool";   // ✅
// const sku3: ProductSKU = "small-purple-cotton";  // ❌ Error: "purple" is not a valid color.

console.log(sku1);  // "small-red-cotton"
console.log(sku2);  // "large-blue-wool"

// Step 3: Extract parts using infer.
type ExtractSize<T> = T extends `${infer S}-${string}-${string}` ? S : never;
type ExtractColor<T> = T extends `${string}-${infer C}-${string}` ? C : never;

type Size1 = ExtractSize<"small-red-cotton">;  // "small"
type Color1 = ExtractColor<"small-red-cotton">;  // "red"

const size: Size1 = "small";
const color: Color1 = "red";
console.log(size);   // "small"
console.log(color);  // "red"

// Step 4: Use in a function with validation.
function createSKU(size: Size, color: Color, material: Material): ProductSKU {
  return `${size}-${color}-${material}`;
}

const newSKU = createSKU("medium", "green", "wool");
console.log(newSKU);  // "medium-green-wool"

console.log("Cross-multiplication complete.");
```

**Expected Output:**
```
small-red-cotton
large-blue-wool
small
red
medium-green-wool
Cross-multiplication complete.
```

**Why This Output Occurs:** The `ProductSKU` type cross-multiplies three unions, producing 18 possible string literals. The `infer` keyword extracts parts of the template literal pattern. The `createSKU` function uses template literal types in its return type, ensuring the result matches the expected pattern.

### Real-World Cases

**Case 1: CSS-in-JS**
CSS-in-JS libraries use template literal types to type CSS values like `` `${number}px` ``, `` `${number}rem` ``, and color codes.

**Case 2: Design Systems**
Design systems use template literal types to generate size, color, and variant combinations for component props.

**Case 3: API Versioning**
API clients use template literal types to type versioned endpoints like `` `/api/${"v1" | "v2"}/users` ``.

**Case 4: Internationalization**
i18n systems use template literal types to generate locale keys like `` `${Lang}_${MessageID}` ``.

**Case 5: Feature Flags**
Feature flag systems use template literal types to generate flag names like `` `${Feature}-branch` ``.

**Case 6: Test Identifiers**
Testing frameworks use template literal types to generate test IDs from component and variant names.

### References

- TypeScript Handbook: Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- TypeScript 4.1 Release Notes: Template Literal Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#template-literal-types
- Playground Example: Intro to Template Literals — https://www.typescriptlang.org/play/4-1/template-literals/intro-to-template-literals.ts.html
- MDN: Template Literals — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals


## 2. String Composition, Combinatorial Explosions, and Compiler Limits

### Definitions

**Core Definition**
String composition is the process of building complex string types from simpler constituent types using template literal types. Combinatorial explosion is the exponential growth in the number of type combinations when multiple unions are cross-multiplied, which can cause severe compiler performance degradation or complete failure.

**Technical Definition**
When multiple unions are used in interpolation positions, the resulting type is the Cartesian product of all union members. For `n` unions with `k₁, k₂, ..., kₙ` members, the result contains `k₁ × k₂ × ... × kₙ` string literals. TypeScript's type checker must materialize every possible combination, which can cause exponential growth in memory and type-checking time. Union types in TypeScript are limited to fewer than 100,000 constituents. Exceeding this limit causes a `RangeError: Map maximum size exceeded` or exponentially slow type checking. The TypeScript team recommends using ahead-of-time generation for large string unions instead of relying on template literal types. GitHub issue #63342 documents that type-checking time doubles with every constituent of dynamic routes with template literals.

**Beginner-Friendly Explanation**
When you cross-multiply unions in template literal types, the number of possible strings grows exponentially. If you have three unions with 10 members each, that's 10 × 10 × 10 = 1,000 combinations. With 20 members each, it's 8,000. With 30 members each, it's 27,000. TypeScript has to create all these combinations in memory, and when the number gets too large, the compiler slows down dramatically or crashes. TypeScript has a hard limit of fewer than 100,000 union constituents. The rule of thumb is: small unions are fine, but large unions can cause serious problems. For large-scale string generation, use ahead-of-time code generation instead.

### Purposes

- To understand the performance implications of cross-multiplying unions.
- To avoid compiler crashes and exponential type-checking time.
- To recognize when template literal types are appropriate versus when ahead-of-time generation is needed.
- To design type-level string patterns that stay within compiler limits.
- To debug performance issues caused by template literal type explosions.

### Syntax Rules and Structure

**General Syntax: Cross-Multiplication Growth**

```typescript
type A = "a1" | "a2" | "a3";  // 3 members
type B = "b1" | "b2" | "b3";  // 3 members
type C = "c1" | "c2" | "c3";  // 3 members

type Combined = `${A}-${B}-${C}`;
// 3 × 3 × 3 = 27 combinations
```

**Component Breakdown**
- Each union contributes its member count.
- The total is the product of all member counts.

**General Syntax: Exponential Growth Example**

```typescript
type Filters = "active" | "inactive" | "pending" | "deleted" | "archived" | "draft";
// 6 members
type Operators = "eq" | "neq" | "gt" | "lt" | "gte" | "lte" | "in" | "nin" | "like";
// 9 members
type QueryFilter = `${Filters}:${Operators}`;
// 6 × 9 = 54 combinations (manageable)
```

**Component Breakdown**
- Growth is manageable for small unions.
- With 20+ members each, the count explodes.

**General Syntax: Compiler Limit**

```typescript
// TypeScript union types are limited to less than 100,000 constituents.
// Exceeding this limit causes:
// RangeError: Map maximum size exceeded
```

**Component Breakdown**
- The hard limit is fewer than 100,000 union members.
- Exceeding it causes compiler failure.

**Syntax Rules**

- Cross-multiplication grows exponentially with the number of unions and their sizes.
- TypeScript has a hard limit of fewer than 100,000 union constituents.
- Type-checking time doubles with each additional constituent in dynamic routes (GitHub issue #63342).
- The `RangeError: Map maximum size exceeded` error occurs when the limit is exceeded.
- The TypeScript team recommends ahead-of-time generation for large string unions.
- Template literal types are best for small to moderate unions.

**Constraints and Limitations**

- Exponential growth is inherent to cross-multiplication.
- The compiler limit is a hard constraint, not a soft warning.
- Type-checking time can become prohibitively slow before hitting the limit.
- Recursive template literal types can cause `Maximum call stack size exceeded`.
- The memory footprint of large unions can cause out-of-memory errors.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing Small vs. Large Unions

```typescript
// Step 1: Small union — manageable.
type Size = "small" | "medium" | "large";  // 3 members
type Color = "red" | "blue";               // 2 members
type SmallProduct = `${Size}-${Color}`;
// 3 × 2 = 6 combinations

const small1: SmallProduct = "small-red";
const small2: SmallProduct = "large-blue";
console.log(small1);  // "small-red"
console.log(small2);  // "large-blue"

// Step 2: Medium union — still manageable but growing.
type Filters = "active" | "inactive" | "pending" | "deleted" | "archived" | "draft";
type Operators = "eq" | "neq" | "gt" | "lt" | "gte" | "lte" | "in" | "nin" | "like";
type QueryFilter = `${Filters}:${Operators}`;
// 6 × 9 = 54 combinations

const filter: QueryFilter = "active:eq";
console.log(filter);  // "active:eq"

// Step 3: Large union — dangerous.
// type LargeA = ... (20 members)
// type LargeB = ... (20 members)
// type LargeC = ... (20 members)
// type LargeCombined = `${LargeA}-${LargeB}-${LargeC}`;
// 20 × 20 × 20 = 8,000 combinations — may slow down the compiler.

// Step 4: Very large union — compiler failure.
// 30 × 30 × 30 = 27,000 combinations
// 40 × 40 × 40 = 64,000 combinations
// 50 × 50 × 50 = 125,000 combinations — EXCEEDS THE 100,000 LIMIT!

console.log("Combinatorial explosion demonstration complete.");
```

**Expected Output:**
```
small-red
large-blue
active:eq
Combinatorial explosion demonstration complete.
```

**Why This Output Occurs:** Small and medium unions produce manageable numbers of combinations. Large unions (8,000+ combinations) may slow the compiler. Very large unions (125,000+ combinations) exceed TypeScript's hard limit and cause compiler failure.

#### Example 2: Performance Impact of Dynamic Routes

```typescript
// Step 1: Define dynamic routes (based on GitHub issue #63342).
type SearchOrHash = `?${string}` | `#${string}`;
type WithProtocol = `${string}:${string}`;
type Suffix = "" | SearchOrHash;

type SafeSlug<S extends string> = S extends `${string}/${string}`
  ? never
  : S extends `${string}${SearchOrHash}`
  ? never
  : S extends ""
  ? never
  : S;

type StaticRoutes = "/home" | "/about" | "/contact";
type DynamicRoutes<T extends string = string> =
  | `/${SafeSlug<T>}/${SafeSlug<T>}/${SafeSlug<T>}`
  | `/${SafeSlug<T>}/${SafeSlug<T>}/integrations/${SafeSlug<T>}/${SafeSlug<T>}/resources/${SafeSlug<T>}/${SafeSlug<T>}/billing`;

type RouteImpl<T> =
  | StaticRoutes
  | SearchOrHash
  | `${StaticRoutes}${SearchOrHash}`
  | (T extends `${DynamicRoutes<infer _>}${Suffix}` ? T : never);

function Link<RouteType>(href: RouteImpl<RouteType>): void {
  console.log(`Navigating to ${href}`);
}

// Step 2: Type checking time doubles with each additional dynamic route constituent.
Link("/home");  // Fast
Link("/api/ai-playground/sandbox");  // Slower
Link("/new/~/integrations/vercel/front/billing");  // Slowest

// Step 3: The performance impact is exponential.
// 6 routes → 45s
// 5 routes → 24s
// 4 routes → 12s
// 3 routes → 6s

console.log("Dynamic route type checking complete.");
```

**Expected Output:**
```
Navigating to /home
Navigating to /api/ai-playground/sandbox
Navigating to /new/~/integrations/vercel/front/billing
Dynamic route type checking complete.
```

**Why This Output Occurs:** The `RouteImpl<T>` type cross-multiplies multiple dynamic route patterns, causing type-checking time to double with each additional constituent. This demonstrates the exponential performance impact documented in GitHub issue #63342.

### Real-World Cases

**Case 1: Next.js Link Components**
Next.js's `<Link>` component uses template literal types for route validation, but the complexity of dynamic routes can cause type-checking performance issues (GitHub issue #63342).

**Case 2: Large Design Systems**
Design systems with many size, color, and variant combinations must carefully limit the number of unions to avoid combinatorial explosion.

**Case 3: API Route Generators**
API route generators with many path segments and parameter combinations can exceed compiler limits.

**Case 4: Query Builders**
Query builders that generate filter types from field and operator unions must keep union sizes small to avoid exponential growth.

### References

- GitHub Issue #63342: type checking complexity with multiple template literals in unions — https://github.com/microsoft/TypeScript/issues/63342
- TypeScript Handbook: Template Literal Types (Ahead-of-time generation recommendation) — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- Stack Overflow: Template Literal Types Typescript repeat (100,000 constituent limit) — https://stackoverflow.com/questions/65357726
- GitHub Issue #62933: Maximum call stack size exceeded with recursive template literal types — https://github.com/microsoft/TypeScript/issues/62933


## 3. Pattern Modeling and Validation (Hex Codes, Dates)

### Definitions

**Core Definition**
Pattern modeling and validation uses template literal types to define types that match specific string formats, such as hexadecimal color codes, ISO date strings, or numeric patterns. These types enforce format constraints at compile time without requiring runtime validation.

**Technical Definition**
Template literal types can model string patterns by combining literal characters with interpolated types (`string`, `number`, or specific unions). For hex codes, patterns like `` `#${string}` `` provide basic validation, while more precise patterns like `` `#${HexDigit}${HexDigit}${HexDigit}${HexDigit}${HexDigit}${HexDigit}` `` (where `HexDigit` is a union of `0-9` and `a-f`) provide stricter validation. For dates, patterns like `` `${YYYY}-${MM}-${DD}` `` where each component is a union of valid values can enforce format and range constraints. TypeScript 4.1+ supports these patterns, but they are not full regular expressions—complex patterns may require workarounds or generic constraints.

**Beginner-Friendly Explanation**
Template literal types let you validate string formats at compile time. For example, you can define a type `` `#${string}` `` that matches any string starting with `#` (a basic hex code check). For stricter validation, you can define the exact pattern: `` `#${"0" | "1" | "2" | ... | "f"}...` `` to ensure the string is a valid 6-digit hex code. For dates, you can define `` `${"19" | "20"}${number}${"-"}${"01" | "02" | ... | "12"}...` `` to match date formats. These types catch format errors at compile time, preventing runtime bugs. However, they're not as powerful as regular expressions—you can't enforce arbitrary constraints like "valid calendar dates" (e.g., February 30th would still pass a simple pattern check).

### Purposes

- To enforce string format constraints at compile time (hex codes, dates, UUIDs, etc.).
- To prevent format-related runtime errors before the code runs.
- To provide precise type definitions for APIs and libraries that expect specific string formats.
- To enable type-safe validation without runtime checks.
- To document the expected format of string values in function signatures and interfaces.

### Syntax Rules and Structure

**General Syntax: Basic Pattern**

```typescript
type HexColor = `#${string}`;
// Matches any string starting with #
```

**Component Breakdown**
- `#`: Literal prefix.
- `${string}`: Any string.

**General Syntax: Strict Hex Code Pattern**

```typescript
type HexDigit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
  | "a" | "b" | "c" | "d" | "e" | "f";

type HexColor = `#${HexDigit}${HexDigit}${HexDigit}${HexDigit}${HexDigit}${HexDigit}`;
// Matches exactly 6 hex digits after #
```

**Component Breakdown**
- Each position is constrained to a valid hex digit.
- The total length is exactly 7 characters.

**General Syntax: Date Pattern**

```typescript
type Year = `${"19" | "20"}${number}`;
type Month = "01" | "02" | "03" | "04" | "05" | "06" | "07" | "08" | "09" | "10" | "11" | "12";
type Day = `${"0" | "1" | "2" | "3"}${number}` | "30" | "31";

type DateString = `${Year}-${Month}-${Day}`;
// Matches "YYYY-MM-DD" format
```

**Component Breakdown**
- Each component is constrained to valid values.
- The hyphens are literal characters.

**General Syntax: Numeric Pattern**

```typescript
type CSSUnit = "px" | "em" | "rem" | "%";
type CSSValue = `${number}${CSSUnit}`;
// Matches "100px", "2.5rem", "50%"
```

**Component Breakdown**
- `${number}` accepts any numeric value.
- `CSSUnit` constrains the suffix.

**Syntax Rules**

- Template literal types can model string patterns with literal characters and interpolated types.
- `${string}` accepts any string; `${number}` accepts any number; `${bigint}` accepts bigints.
- Unions constrain interpolated positions to specific values.
- Hex codes use a union of hex digits.
- Dates use unions of valid year, month, and day components.
- CSS values use `${number}` with a unit union.
- Patterns are not full regex—complex constraints (like calendar date validity) are not supported.

**Constraints and Limitations**

- Template literal types are not regular expressions—complex patterns are not supported.
- Range constraints (e.g., "day must be 01-31") require explicit unions.
- Calendar validity (e.g., February 30th) cannot be enforced.
- Large digit unions (e.g., 0-99) can cause combinatorial explosion.
- Format validation is compile-time only; runtime data still needs validation.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Hex Code Pattern

```typescript
// Step 1: Define a basic hex color type.
type BasicHex = `#${string}`;

const basic1: BasicHex = "#FF0000";  // ✅
const basic2: BasicHex = "#abc";     // ✅
// const basic3: BasicHex = "FF0000";  // ❌ Error: Missing #.

console.log(basic1);  // "#FF0000"

// Step 2: Define a strict hex color type.
type HexDigit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
  | "a" | "b" | "c" | "d" | "e" | "f";

type StrictHex = `#${HexDigit}${HexDigit}${HexDigit}${HexDigit}${HexDigit}${HexDigit}`;
// Exactly 6 hex digits after #

const strict1: StrictHex = "#ff0000";  // ✅
const strict2: StrictHex = "#a1b2c3";  // ✅
// const strict3: StrictHex = "#abc";   // ❌ Error: Too short (3 digits).
// const strict4: StrictHex = "#gggggg"; // ❌ Error: 'g' is not a valid hex digit.

console.log(strict1);  // "#ff0000"
console.log(strict2);  // "#a1b2c3"

// Step 3: Use in a function.
function setColor(color: StrictHex): void {
  console.log(`Setting color to ${color}`);
}

setColor("#ff0000");  // "Setting color to #ff0000"
// setColor("#00ff00"); // ✅
// setColor("red");     // ❌ Error: Does not match hex pattern.

console.log("Hex code validation complete.");
```

**Expected Output:**
```
#FF0000
#ff0000
#a1b2c3
Setting color to #ff0000
Hex code validation complete.
```

**Why This Output Occurs:** `BasicHex` accepts any string starting with `#`. `StrictHex` requires exactly 6 valid hex digits. The function accepts only strings matching the strict pattern. Invalid formats produce compile errors.

#### Example 2: Date and CSS Pattern Validation

```typescript
// Step 1: Define a date pattern.
type Year = `${"19" | "20"}${number}`;
type Month = "01" | "02" | "03" | "04" | "05" | "06" | "07" | "08" | "09" | "10" | "11" | "12";
type Day = `${"0" | "1" | "2" | "3"}${number}` | "30" | "31";

type DateString = `${Year}-${Month}-${Day}`;

const date1: DateString = "2024-01-15";  // ✅
const date2: DateString = "1999-12-31";  // ✅
// const date3: DateString = "2024-13-01";  // ❌ Error: "13" is not a valid month.
// const date4: DateString = "2024-01-32";  // ❌ Error: "32" is not a valid day.

console.log(date1);  // "2024-01-15"
console.log(date2);  // "1999-12-31"

// Step 2: Define a CSS value pattern.
type CSSUnit = "px" | "em" | "rem" | "%";
type CSSValue = `${number}${CSSUnit}`;

const css1: CSSValue = "100px";   // ✅
const css2: CSSValue = "2.5rem";  // ✅
const css3: CSSValue = "50%";     // ✅
// const css4: CSSValue = "100pt"; // ❌ Error: "pt" is not a valid unit.

console.log(css1);  // "100px"
console.log(css2);  // "2.5rem"
console.log(css3);  // "50%"

// Step 3: Use in a function.
function applyStyle(value: CSSValue): void {
  console.log(`Applying ${value}`);
}

applyStyle("100px");   // "Applying 100px"
applyStyle("2.5rem");  // "Applying 2.5rem"
// applyStyle("100pt"); // ❌ Error.

console.log("Pattern validation complete.");
```

**Expected Output:**
```
2024-01-15
1999-12-31
100px
2.5rem
50%
Applying 100px
Applying 2.5rem
Pattern validation complete.
```

**Why This Output Occurs:** The `DateString` type enforces the `YYYY-MM-DD` format with valid month and day ranges. The `CSSValue` type enforces a number followed by a valid CSS unit. Invalid formats produce compile errors. Note that date validity (e.g., February 30th) is not enforced—only format and range.

### Real-World Cases

**Case 1: Design System Colors**
Design systems use strict hex code types to validate color tokens, preventing invalid color values in component props.

**Case 2: CSS-in-JS Libraries**
CSS-in-JS libraries use `${number}${unit}` patterns to type CSS values, ensuring only valid units are used.

**Case 3: Date Format Validation**
APIs that accept date strings use template literal types to enforce ISO date format, catching format errors at compile time.

**Case 4: UUID Validation**
Libraries that work with UUIDs use template literal types to enforce the UUID format pattern.

**Case 5: Version Strings**
Package managers use template literal types to validate semantic version strings like `` `${number}.${number}.${number}` ``.

**Case 6: File Extensions**
File processing libraries use template literal types to validate file extensions like `` `.${"png" | "jpg" | "gif"}` ``.

### References

- TypeScript Handbook: Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- Stack Overflow: A way to mark arbitrary strings in Typescript Template Literals (Hex color codes) — https://stackoverflow.com/questions/66826206
- GeeksforGeeks: How to Define a Regex-Matched String Type in TypeScript — https://www.geeksforgeeks.org/how-to-define-a-regex-matched-string-type-in-typescript/
- Stack Overflow: Is it possible to specify string format with TypeScript? — https://stackoverflow.com/questions/47162098


## 4. Event-Name Types (`on${Capitalize<Event>}`)

### Definitions

**Core Definition**
Event-name types use template literal types to derive event handler names from event names. The pattern `` `on${Capitalize<Event>}` `` transforms an event name like `"click"` into a handler name like `"onClick"`. This is one of the most common real-world applications of template literal types, enabling type-safe event systems.

**Technical Definition**
Event-name types combine template literal types with the `Capitalize` intrinsic string utility to transform event names into handler names. The pattern `` `on${Capitalize<EventName>}` `` cross-multiplies with the union of event names, producing a union of handler names. Combined with `infer` and mapped types, this enables type-safe event emitters where the payload type is derived from the event name. The pattern is used in React (event handler props), DOM APIs (`addEventListener`), and custom event systems.

**Beginner-Friendly Explanation**
Event-name types let you automatically generate handler names from event names. If you have events `"click"`, `"focus"`, and `"blur"`, then `` `on${Capitalize<"click" | "focus" | "blur">}` `` gives you `"onClick" | "onFocus" | "onBlur"`. This is exactly how React names its event handler props. The `Capitalize` utility capitalizes the first letter, and the `on` prefix follows the convention. This pattern is used everywhere in event systems—DOM events, React props, custom event emitters—to ensure type-safe event handling. When you combine it with `infer` and mapped types, you can create fully type-safe event systems where both the event name and the payload type are validated.

### Purposes

- To derive event handler names from event names automatically.
- To enforce naming conventions for event handlers at compile time.
- To enable type-safe event emitters with correct payload types.
- To generate React-style event handler prop types.
- To ensure consistency between event names and their handlers.

### Syntax Rules and Structure

**General Syntax: Basic Event Handler Type**

```typescript
type EventName = "click" | "focus" | "blur";
type EventHandler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"
```

**Component Breakdown**
- `on`: Literal prefix.
- `Capitalize<EventName>`: Capitalizes each event name.
- The result is a union of handler names.

**General Syntax: Typed Event Emitter**

```typescript
interface EventMap {
  "user:login": { userId: number; sessionId: string };
  "user:logout": { userId: number };
  "cart:add": { productId: string; quantity: number };
}

type EventName = keyof EventMap & string;
type PayloadFor<E extends EventName> = EventMap[E];

class TypedEventBus {
  on<E extends EventName>(event: E, handler: (payload: PayloadFor<E>) => void): void { }
  emit<E extends EventName>(event: E, payload: PayloadFor<E>): void { }
}
```

**Component Breakdown**
- `EventMap`: Defines event names and their payload types.
- `EventName`: Extracts event names from the map.
- `PayloadFor<E>`: Derives the payload type from the event name.

**General Syntax: DOM Event Handler Types**

```typescript
type DOMEvent = "click" | "focus" | "blur" | "input" | "change";
type DOMEventHandler = `on${Capitalize<DOMEvent>}`;
// "onClick" | "onFocus" | "onBlur" | "onInput" | "onChange"
```

**Component Breakdown**
- The pattern matches React's event handler prop naming convention.

**Syntax Rules**

- The `on` prefix is conventional for event handlers.
- `Capitalize<EventName>` capitalizes the first letter of each event name.
- The pattern works with any union of event names.
- Combined with `infer` and mapped types, it enables full event system typing.
- The `& string` intersection is needed when using `keyof` with template literals.
- The pattern works with both camelCase and colon-separated event names.

**Constraints and Limitations**

- Large event unions can cause combinatorial explosion (see Section 2).
- The pattern assumes the `on` + `Capitalize` convention.
- Event names with special characters (e.g., `:`) cannot be directly capitalized.
- Template literal types are compile-time only; runtime event names are still strings.
- The `& string` intersection is needed for `keyof` in some cases.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Event Handler Types

```typescript
// Step 1: Define event names.
type EventName = "click" | "focus" | "blur" | "input";

// Step 2: Derive handler names.
type EventHandler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur" | "onInput"

const handler1: EventHandler = "onClick";  // ✅
const handler2: EventHandler = "onFocus";  // ✅
// const handler3: EventHandler = "onClickk"; // ❌ Error: Not a valid handler name.

console.log(handler1);  // "onClick"

// Step 3: Use in a function.
function registerHandler(name: EventHandler): void {
  console.log(`Registering handler: ${name}`);
}

registerHandler("onClick");   // "Registering handler: onClick"
registerHandler("onInput");   // "Registering handler: onInput"
// registerHandler("onHover"); // ❌ Error: "onHover" is not a valid handler name.

console.log("Event handler types complete.");
```

**Expected Output:**
```
onClick
Registering handler: onClick
Registering handler: onInput
Event handler types complete.
```

**Why This Output Occurs:** The `EventHandler` type derives handler names from event names using the `on${Capitalize<...>}` pattern. The function accepts only valid handler names. Invalid names produce compile errors.

#### Example 2: Type-Safe Event Emitter

```typescript
// Step 1: Define an event map with payload types.
interface EventMap {
  "user:login": { userId: number; sessionId: string };
  "user:logout": { userId: number };
  "cart:add": { productId: string; quantity: number };
  "cart:remove": { productId: string };
}

// Step 2: Extract event names and payload types.
type EventName = keyof EventMap & string;
type PayloadFor<E extends EventName> = EventMap[E];

// Step 3: Create a typed event bus.
class TypedEventBus {
  private listeners = new Map<string, Function[]>();

  on<E extends EventName>(event: E, handler: (payload: PayloadFor<E>) => void): void {
    const entries = this.listeners.get(event) ?? [];
    entries.push(handler);
    this.listeners.set(event, entries);
  }

  emit<E extends EventName>(event: E, payload: PayloadFor<E>): void {
    this.listeners.get(event)?.forEach((fn) => fn(payload));
  }
}

// Step 4: Use the event bus with type safety.
const bus = new TypedEventBus();

bus.on("user:login", (payload) => {
  // payload.userId is number
  // payload.sessionId is string
  console.log(`User ${payload.userId} logged in with session ${payload.sessionId}`);
});

bus.on("cart:add", (payload) => {
  // payload.productId is string
  // payload.quantity is number
  console.log(`Added ${payload.quantity} of ${payload.productId} to cart`);
});

bus.emit("user:login", { userId: 1, sessionId: "abc123" });
// "User 1 logged in with session abc123"

bus.emit("cart:add", { productId: "prod-1", quantity: 2 });
// "Added 2 of prod-1 to cart"

// bus.emit("user:login", { userId: 1 }); // ❌ Error: Missing sessionId.
// bus.emit("nonexistent:event", {});     // ❌ Error: Not a valid event.

console.log("Typed event emitter complete.");
```

**Expected Output:**
```
User 1 logged in with session abc123
Added 2 of prod-1 to cart
Typed event emitter complete.
```

**Why This Output Occurs:** The `EventMap` defines event names and their payload types. The `PayloadFor<E>` conditional type derives the payload type from the event name. The `on` and `emit` methods use generics to enforce that the handler's parameter type matches the event's payload type. Invalid events and mismatched payloads produce compile errors.

### Real-World Cases

**Case 1: React Event Props**
React components use `on${Capitalize<Event>}` to type event handler props like `onClick`, `onChange`, and `onFocus`.

**Case 2: DOM Event Listeners**
DOM event listener libraries use event-name types to type `addEventListener` calls with specific event names and handler signatures.

**Case 3: Custom Event Systems**
Custom event emitters (Node.js EventEmitter, mitt, etc.) use event-name types to provide type-safe event handling.

**Case 4: UI Component Libraries**
Component libraries use event-name types to generate handler prop types from event maps, ensuring consistency.

**Case 5: State Machine Events**
State machine libraries use event-name types to type transitions and event handlers.

**Case 6: WebSocket Events**
WebSocket libraries use event-name types to type message events and connection events.

### References

- TypeScript Template Literal Types: String Manipulation at the Type Level — https://dev.to/kaithorne/typescript-template-literal-types-string-manipulation-at-the-type-level-2b47
- String-pattern typing (route params, event names) — TypeScript Academy — https://www.coddykit.com
- TypeScript Template Literal Types: Type-Level String Manipulation — https://rune.codes
- Template Literal Types — Frontend Academy — https://www.coddykit.com


## 5. Route and API-Path Types (Parsing Parameters from Strings)

### Definitions

**Core Definition**
Route and API-path types use template literal types with `infer` to extract parameter names from route patterns like `/users/:userId/posts/:postId`. This enables type-safe routing where route parameters are known at compile time, and renaming a route parameter causes compile errors at every usage site.

**Technical Definition**
Route parameter inference uses conditional types with template literal patterns to parse `:paramName` segments from route strings. The pattern `` `${string}:${infer Param}/${infer Rest}` `` extracts a parameter name and the remaining path. Recursive conditional types accumulate parameters into an object type. This technique is used by libraries like Hono, TanStack Router, and React Router to provide compile-time route safety. The extraction requires the route path to be a string literal type (via `as const` or explicit literal types), not a general `string`. Wildcard and regex segments (`*`, `:id(\d+)`) require extended parsers.

**Beginner-Friendly Explanation**
Route types let you parse parameter names from route strings at compile time. If you have a route `/users/:userId/posts/:postId`, TypeScript can extract `userId` and `postId` as parameter names. Then, when you write a handler, TypeScript knows that `params.userId` and `params.postId` are strings, and it will flag typos like `params.userID`. This is how modern routing libraries like Hono and TanStack Router provide type safety. The key is using `infer` in a recursive conditional type to parse the route string. This prevents the common bug where renaming a route parameter leaves stale references scattered across the codebase.

### Purposes

- To extract parameter names from route patterns at compile time.
- To enable type-safe route handlers where parameters are known and validated.
- To prevent parameter name drift between route declarations and handlers.
- To provide autocompletion and error checking for route parameters.
- To support type-safe route builders and navigation functions.

### Syntax Rules and Structure

**General Syntax: Basic Route Parameter Extraction**

```typescript
type ExtractParams<Path extends string> =
  Path extends `${string}:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof ExtractParams<`/${Rest}`>]: string }
    : Path extends `${string}:${infer Param}`
    ? { [K in Param]: string }
    : Record<string, never>;
```

**Component Breakdown**
- `` `${string}:${infer Param}/${infer Rest}` ``: Matches a parameter followed by `/` and more path.
- `{ [K in Param | keyof ExtractParams<`/${Rest}`>]: string }`: Accumulates parameters recursively.
- The base case returns `Record<string, never>` for paths without parameters.

**General Syntax: Type-Safe Route Function**

```typescript
function route<P extends string>(
  path: P,
  handler: (params: ExtractParams<P>) => Response
): void { }
```

**Component Breakdown**
- `P extends string`: The route path literal type.
- `handler: (params: ExtractParams<P>) => Response`: The handler receives typed params.

**General Syntax: Route Builder**

```typescript
type BuildPath<Path extends string, Params extends Record<string, string>> =
  Path extends `${infer Prefix}:${infer Param}/${infer Rest}`
    ? Param extends keyof Params
    ? `${Prefix}${Params[Param]}/${BuildPath<Rest, Params>}`
    : never
    : Path extends `${infer Prefix}:${infer Param}`
    ? Param extends keyof Params
    ? `${Prefix}${Params[Param]}`
    : never
    : Path;
```

**Component Breakdown**
- Builds a concrete path string from a route pattern and parameter values.

**Syntax Rules**

- Route parameters are identified by the `:paramName` syntax.
- `infer` extracts parameter names from the route string.
- Recursive conditional types accumulate parameters into an object type.
- The route path must be a string literal type (not `string`).
- `as const` is used to preserve route literal types.
- Wildcard and regex segments require extended parsers.
- Routes built at runtime from user input cannot be inferred.

**Constraints and Limitations**

- Route parameter inference requires the path to be a literal type.
- Wildcard (`*`) and regex (`:id(\d+)`) segments are not supported by simple parsers.
- Large numbers of route parameters can cause performance issues.
- The inference does not validate that parameter values are non-empty.
- Routes built dynamically at runtime cannot benefit from inference.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Route Parameter Extraction

```typescript
// Step 1: Define the parameter extraction type.
type ExtractParams<Path extends string> =
  Path extends `${string}:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof ExtractParams<`/${Rest}`>]: string }
    : Path extends `${string}:${infer Param}`
    ? { [K in Param]: string }
    : Record<string, never>;

// Step 2: Test with different routes.
type Params1 = ExtractParams<"/users/:userId">;
// { userId: string }

type Params2 = ExtractParams<"/users/:userId/posts/:postId">;
// { userId: string; postId: string }

type Params3 = ExtractParams<"/home">;
// Record<string, never>

// Step 3: Create a type-safe route function.
function route<P extends string>(
  path: P,
  handler: (params: ExtractParams<P>) => string
): (params: ExtractParams<P>) => string {
  return handler;
}

// Step 4: Use the route function.
const userHandler = route("/users/:userId", (params) => {
  // params.userId is string
  return `User: ${params.userId}`;
});

console.log(userHandler({ userId: "123" }));  // "User: 123"

const postHandler = route("/users/:userId/posts/:postId", (params) => {
  // params.userId is string, params.postId is string
  return `User ${params.userId}, Post ${params.postId}`;
});

console.log(postHandler({ userId: "1", postId: "42" }));
// "User 1, Post 42"

// Step 5: Typos are compile errors.
// postHandler({ userId: "1", postID: "42" });
// ❌ Error: Property 'postID' does not exist. Did you mean 'postId'?

console.log("Route parameter extraction complete.");
```

**Expected Output:**
```
User: 123
User 1, Post 42
Route parameter extraction complete.
```

**Why This Output Occurs:** The `ExtractParams<Path>` conditional type recursively parses `:param` segments from the route string. The `route` function uses this type to ensure the handler receives the correct parameter object. Typos in parameter names produce compile errors.

#### Example 2: Route Builder and as const

```typescript
// Step 1: Define a route builder type.
type BuildPath<Path extends string, Params extends Record<string, string>> =
  Path extends `${infer Prefix}:${infer Param}/${infer Rest}`
    ? Param extends keyof Params
    ? `${Prefix}${Params[Param]}/${BuildPath<Rest, Params>}`
    : never
    : Path extends `${infer Prefix}:${infer Param}`
    ? Param extends keyof Params
    ? `${Prefix}${Params[Param]}`
    : never
    : Path;

// Step 2: Define routes with as const to preserve literal types.
const routes = {
  user: "/users/:userId",
  post: "/users/:userId/posts/:postId",
} as const;

// Step 3: Build concrete paths.
type UserPath = BuildPath<typeof routes.user, { userId: string }>;
// "/users/123"

const userPath: UserPath = "/users/123";
console.log(userPath);  // "/users/123"

type PostPath = BuildPath<typeof routes.post, { userId: string; postId: string }>;
// "/users/1/posts/42"

const postPath: PostPath = "/users/1/posts/42";
console.log(postPath);  // "/users/1/posts/42"

// Step 4: Missing parameters are compile errors.
// type InvalidPath = BuildPath<typeof routes.post, { userId: string }>;
// ❌ Error: postId is not in the params object.

// Step 5: Combine with route parameter extraction.
type ExtractParams<Path extends string> =
  Path extends `${string}:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof ExtractParams<`/${Rest}`>]: string }
    : Path extends `${string}:${infer Param}`
    ? { [K in Param]: string }
    : Record<string, never>;

type PostParams = ExtractParams<typeof routes.post>;
// { userId: string; postId: string }

const params: PostParams = { userId: "1", postId: "42" };
console.log(params);  // { userId: '1', postId: '42' }

console.log("Route builder complete.");
```

**Expected Output:**
```
/users/123
/users/1/posts/42
{ userId: '1', postId: '42' }
Route builder complete.
```

**Why This Output Occurs:** The `BuildPath` type constructs a concrete path string from a route pattern and parameter values. The `as const` assertion preserves the route literal types. The `ExtractParams` type extracts parameter names from the route pattern. Missing parameters produce compile errors.

### Real-World Cases

**Case 1: Hono Router**
Hono uses template literal types to infer route parameters from path patterns, providing type-safe route handlers.

**Case 2: TanStack Router**
TanStack Router uses route parameter inference to provide fully typed route handlers and navigation functions.

**Case 3: React Router**
React Router's type definitions use template literal types for route parameter extraction and validation.

**Case 4: Express with TypeScript**
Express route handlers can use template literal types to type `req.params` based on the route pattern.

**Case 5: Next.js App Router**
Next.js uses template literal types for type-safe routing, though complex dynamic routes can cause performance issues (GitHub issue #63342).

**Case 6: Fastify**
Fastify's TypeScript types use template literal types for route parameter inference.

### References

- Infer Route Parameters from Path Patterns — https://raw.githubusercontent.com/pproenca/dot-skills/refs/heads/master/skills/.experimental/typescript-advanced-patterns/references/dsl-route-param-inference.md
- TypeScript Handbook: Inference in Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types
- ts-routes: Strongly typed parameterized routing paths — https://github.com/leancodepl/ts-routes
- @alevettih/typed-route-template — https://www.npmjs.com/package/@alevettih/typed-route-template


## 6. Intrinsic String Manipulation Utilities (`Uppercase`, `Lowercase`, `Capitalize`, `Uncapitalize`)

### Definitions

**Core Definition**
Intrinsic string manipulation utilities are four built-in TypeScript type functions—`Uppercase`, `Lowercase`, `Capitalize`, and `Uncapitalize`—that transform string literal types at compile time. They are intrinsic to the TypeScript compiler and are implemented natively rather than in user code.

**Technical Definition**
The four intrinsic string types were introduced in TypeScript 4.1 alongside template literal types. They are declared in `lib.es5.d.ts` as: `type Uppercase<S extends string> = intrinsic;`, `type Lowercase<S extends string> = intrinsic;`, `type Capitalize<S extends string> = intrinsic;`, and `type Uncapitalize<S extends string> = intrinsic;`. The `intrinsic` keyword indicates that the compiler provides the implementation. These types take a single string type parameter and return the transformed string literal type. They work exclusively at the type level and have no runtime equivalent. They replace the earlier `uppercase`, `lowercase`, `capitalize`, and `uncapitalize` modifiers that were proposed in PR #40336. The intrinsic types are only applicable to string literal types; applying them to the general `string` type returns `string`.

**Beginner-Friendly Explanation**
These four utilities let you transform strings at the type level. `Uppercase<"hello">` gives `"HELLO"`. `Lowercase<"HELLO">` gives `"hello"`. `Capitalize<"hello">` gives `"Hello"` (first letter uppercase). `Uncapitalize<"Hello">` gives `"hello"` (first letter lowercase). They work on string literal types, not on the general `string` type. You can use them inside template literal types to transform interpolated values. For example, `` `on${Capitalize<"click">}` `` gives `"onClick"`. These are built into TypeScript 4.1+, so you don't need to define them—they're always available.

### Purposes

- To transform string literal types to uppercase or lowercase.
- To capitalize or uncapitalize the first letter of string literal types.
- To enable naming convention conversions (camelCase → PascalCase, etc.).
- To generate derived type names from existing ones.
- To serve as building blocks for template literal type transformations.

### Syntax Rules and Structure

**General Syntax: `Uppercase`**

```typescript
type Upper = Uppercase<"hello">;  // "HELLO"
```

**Component Breakdown**
- `Uppercase<S extends string>`: Converts all characters to uppercase.

**General Syntax: `Lowercase`**

```typescript
type Lower = Lowercase<"HELLO">;  // "hello"
```

**Component Breakdown**
- `Lowercase<S extends string>`: Converts all characters to lowercase.

**General Syntax: `Capitalize`**

```typescript
type Cap = Capitalize<"hello">;  // "Hello"
```

**Component Breakdown**
- `Capitalize<S extends string>`: Capitalizes the first character.

**General Syntax: `Uncapitalize`**

```typescript
type Uncap = Uncapitalize<"Hello">;  // "hello"
```

**Component Breakdown**
- `Uncapitalize<S extends string>`: Lowercases the first character.

**General Syntax: In Template Literal Types**

```typescript
type EventHandler = `on${Capitalize<"click" | "focus">}`;
// "onClick" | "onFocus"
```

**Component Breakdown**
- Intrinsic utilities can be used inside template literal interpolations.

**Syntax Rules**

- All four utilities take a single `string` type parameter.
- They transform string literal types; applying them to `string` returns `string`.
- They are intrinsic to the compiler and are always available (TypeScript 4.1+).
- They can be used inside template literal types.
- They can be combined with `infer` for pattern extraction and transformation.
- They work on unions by distributing over each member.

**Constraints and Limitations**

- They only work on string literal types, not on the general `string` type.
- They cannot transform runtime values—they are compile-time only.
- They do not handle locale-specific casing rules.
- `Uppercase` and `Lowercase` transform all characters, not just the first.
- They cannot be used with `number` or `symbol` types.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Intrinsic String Transformations

```typescript
// Step 1: Apply each intrinsic type.
type Upper = Uppercase<"hello world">;      // "HELLO WORLD"
type Lower = Lowercase<"HELLO WORLD">;      // "hello world"
type Cap = Capitalize<"hello world">;       // "Hello world"
type Uncap = Uncapitalize<"Hello world">;   // "hello world"

const upper: Upper = "HELLO WORLD";
const lower: Lower = "hello world";
const cap: Cap = "Hello world";
const uncap: Uncap = "hello world";

console.log(upper);  // "HELLO WORLD"
console.log(lower);  // "hello world"
console.log(cap);    // "Hello world"
console.log(uncap);  // "hello world"

// Step 2: Apply to unions (distributes over each member).
type Event = "click" | "focus" | "blur";
type PascalEvent = Capitalize<Event>;
// "Click" | "Focus" | "Blur"

const pascal: PascalEvent = "Click";
console.log(pascal);  // "Click"

// Step 3: Combine with template literal types.
type Handler = `on${Capitalize<Event>}`;
// "onClick" | "onFocus" | "onBlur"

const handler: Handler = "onClick";
console.log(handler);  // "onClick"

// Step 4: Apply to general string (returns string).
type General = Uppercase<string>;  // string
const general: General = "anything";
console.log(general);  // "anything"

console.log("Intrinsic string types complete.");
```

**Expected Output:**
```
HELLO WORLD
hello world
Hello world
hello world
Click
onClick
anything
Intrinsic string types complete.
```

**Why This Output Occurs:** The four intrinsic types transform string literal types at compile time. `Uppercase` converts all characters to uppercase. `Capitalize` capitalizes only the first character. When applied to unions, they distribute over each member. When applied to the general `string` type, they return `string`.

#### Example 2: Naming Convention Conversion

```typescript
// Step 1: Define camelCase event names.
type CamelEvent = "userLogin" | "userLogout" | "cartAdd" | "cartRemove";

// Step 2: Convert to PascalCase for handler names.
type PascalHandler = `on${Capitalize<CamelEvent>}`;
// "onUserLogin" | "onUserLogout" | "onCartAdd" | "onCartRemove"

const handler: PascalHandler = "onUserLogin";
console.log(handler);  // "onUserLogin"

// Step 3: Convert to kebab-case for CSS class names.
type KebabCase<S extends string> =
  S extends `${infer First}${infer Rest}`
    ? First extends Uppercase<First>
      ? `-${Lowercase<First>}${KebabCase<Rest>}`
      : `${First}${KebabCase<Rest>}`
    : S;

type KebabEvent = KebabCase<CamelEvent>;
// "user-login" | "user-logout" | "cart-add" | "cart-remove"

const kebab: KebabEvent = "user-login";
console.log(kebab);  // "user-login"

// Step 4: Convert to SCREAMING_SNAKE_CASE for constants.
type ScreamingSnake<S extends string> =
  S extends `${infer First}${infer Rest}`
    ? First extends Uppercase<First>
      ? `_${Uppercase<First>}${ScreamingSnake<Rest>}`
      : `${Uppercase<First>}${ScreamingSnake<Rest>}`
    : S;

type ConstantEvent = ScreamingSnake<CamelEvent>;
// "USER_LOGIN" | "USER_LOGOUT" | "CART_ADD" | "CART_REMOVE"

const constant: ConstantEvent = "USER_LOGIN";
console.log(constant);  // "USER_LOGIN"

console.log("Naming convention conversion complete.");
```

**Expected Output:**
```
onUserLogin
user-login
USER_LOGIN
Naming convention conversion complete.
```

**Why This Output Occurs:** The intrinsic types enable naming convention conversions. `Capitalize` converts camelCase to PascalCase. Recursive conditional types with `Uppercase` and `Lowercase` convert camelCase to kebab-case and SCREAMING_SNAKE_CASE. These transformations happen entirely at the type level.

### Real-World Cases

**Case 1: Event Handler Naming**
React and DOM event systems use `Capitalize` to generate handler names from event names.

**Case 2: CSS Class Name Generation**
CSS-in-JS libraries use `KebabCase` transformations (built on `Lowercase` and `Uppercase`) to generate class names from component names.

**Case 3: Constant Generation**
Build tools use `ScreamingSnakeCase` transformations to generate constant names from identifiers.

**Case 4: API Client Methods**
API clients use `Capitalize` to generate method names from endpoint names (e.g., `getUser` from `user`).

**Case 5: Type-Safe Enum Alternatives**
String literal unions combined with intrinsic types provide type-safe alternatives to enums with automatic case transformations.

**Case 6: Internationalization Keys**
i18n systems use `Uppercase` and `Lowercase` to normalize locale keys for case-insensitive lookups.

### References

- TypeScript 4.1 Release Notes: Template Literal Types (Intrinsic String Manipulation Types) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#template-literal-types
- Playground Example: Intro to Template Literals — https://www.typescriptlang.org/play/4-1/template-literals/intro-to-template-literals.ts.html
- GitHub PR #40580: Intrinsic string types — https://github.com/microsoft/TypeScript/pull/40580
- Stack Overflow: What's intrinsic in TypeScript? — https://stackoverflow.com/questions/66209873
- TypeScript Handbook: Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html


## Summary: Template Literal Type Feature Comparison

| Feature | Syntax | Purpose | TypeScript Version |
|---------|--------|---------|-------------------|
| Template literal type | `` `prefix${Type}suffix` `` | Create string literal types by interpolation | 4.1 |
| Union cross-multiplication | `` `${A | B}-${C | D}` `` | Generate all combinations | 4.1 |
| `Uppercase` | `Uppercase<S>` | Convert to uppercase | 4.1 |
| `Lowercase` | `Lowercase<S>` | Convert to lowercase | 4.1 |
| `Capitalize` | `Capitalize<S>` | Capitalize first character | 4.1 |
| `Uncapitalize` | `Uncapitalize<S>` | Uncapitalize first character | 4.1 |
| `infer` extraction | `` T extends `${infer P}` `` | Extract substrings from patterns | 4.1 |
| Key remapping | `as \`get${Capitalize<K>}\`` | Transform property names in mapped types | 4.1 |
| Pattern validation | `` `#${HexDigit}${HexDigit}...` `` | Enforce string formats | 4.1 |
| Route parameters | `` `${string}:${infer Param}/${infer Rest}` `` | Parse route params | 4.1 |
| Event names | `` `on${Capitalize<Event>}` `` | Derive handler names | 4.1 |


## References

- TypeScript Handbook: Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- TypeScript 4.1 Release Notes: Template Literal Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#template-literal-types
- Playground Example: Intro to Template Literals — https://www.typescriptlang.org/play/4-1/template-literals/intro-to-template-literals.ts.html
- Playground Example: String Manipulation with Template Literals — https://www.typescriptlang.org/play/typescript/meta-types/string-manipulation-with-template-literals.ts.html
- GitHub Issue #63342: type checking complexity with multiple template literals in unions — https://github.com/microsoft/TypeScript/issues/63342
- GitHub Issue #62933: Maximum call stack size exceeded with recursive template literal types — https://github.com/microsoft/TypeScript/issues/62933
- GitHub PR #40580: Intrinsic string types — https://github.com/microsoft/TypeScript/pull/40580
- GitHub PR #40336: Key Remapping in Mapped Types — https://github.com/microsoft/TypeScript/pull/40336
- TypeScript Template Literal Types: String Manipulation at the Type Level — https://dev.to/kaithorne/typescript-template-literal-types-string-manipulation-at-the-type-level-2b47
- Infer Route Parameters from Path Patterns — https://raw.githubusercontent.com/pproenca/dot-skills/refs/heads/master/skills/.experimental/typescript-advanced-patterns/references/dsl-route-param-inference.md
- TypeScript Handbook: Inference in Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types
- Stack Overflow: A way to mark arbitrary strings in Typescript Template Literals — https://stackoverflow.com/questions/66826206
- Stack Overflow: Is it possible to specify string format with TypeScript? — https://stackoverflow.com/questions/47162098
- GeeksforGeeks: How to Define a Regex-Matched String Type in TypeScript — https://www.geeksforgeeks.org/how-to-define-a-regex-matched-string-type-in-typescript/
- MDN: Template Literals — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals