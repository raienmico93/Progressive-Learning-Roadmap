## Primitive Types

This section covers TypeScript’s primitive and special types. JavaScript defines 8 built-in types — `number`, `string`, `bigint`, `boolean`, `symbol`, `null`, `undefined`, and `object` — and TypeScript provides corresponding primitive types for each of them . TypeScript adds the special types `void`, `never`, `any`, and `unknown` on top of JavaScript’s primitive set .

---

## 1. `string`

### Definitions

**Core Definition.** The `string` type represents **immutable sequences of UTF-16 code units**, used for textual data.

**Technical Definition.** In TypeScript, `string` is a primitive type corresponding to JavaScript’s `String` primitive value. String literals are enclosed in single quotes, double quotes, or backticks. The type is **immutable**: methods such as `toUpperCase()` return new strings rather than modifying the original.

**Beginner-Friendly Explanation.** A `string` is **text** — anything you would put in quotes: names, sentences, messages, URLs. TypeScript checks that you only perform string operations on string values.

### Purposes

- **To represent textual data** such as names, messages, and identifiers.
- **To enforce that string operations are applied only to strings** (e.g., `.toUpperCase()`, `.length`).
- **To model string literal unions** for finite sets of allowed values.

### Syntax Rules and Structure

```
let variableName: string = "value";
```

- `variableName` — identifier.
- `: string` — type annotation.
- `= "value"` — optional initializer; any string literal or expression producing a string.

**Syntax rules.**

- Use lowercase `string`, **not** uppercase `String`. `String` refers to the boxed object wrapper and is almost never appropriate .
- String literals can use single quotes, double quotes, or template literals (backticks).
- Literal types such as `"hello"` are subtypes of `string` and are useful for discriminated unions and finite value sets.

**Constraints and limitations.**

- Strings are **immutable**; all string methods return new strings.
- `string` and `String` are **not interchangeable** in strict annotations; always use `string`.
- Template literal types can create precise string patterns but may become complex.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic string annotation and operations

```typescript
// example_string.ts
// Step 1: Annotate a string variable.
let greeting: string = "Hello, TypeScript";

// Step 2: Use string operations.
const upper = greeting.toUpperCase();
const length = greeting.length;

// Step 3: Log results.
console.log(upper);    // Expected: HELLO, TYPESCRIPT
console.log(length);   // Expected: 17
```

**Expected Output:**
```
HELLO, TYPESCRIPT
17
```

**Why this output occurs.** `greeting` is annotated as `string`, so the compiler knows that `.toUpperCase()` and `.length` are valid string members. The operations return new values; the original string is unchanged.

#### Example 2: String literal union

```typescript
// example_string_literal.ts
// Step 1: Define a union of allowed string literals.
type Direction = "up" | "down" | "left" | "right";

// Step 2: Declare a variable of that union type.
let move: Direction = "up";

// Step 3: Uncomment to see a compile-time error.
// move = "forward";
// Error: Type '"forward"' is not assignable to type 'Direction'.

console.log(move); // Expected: up
```

**Expected Output:**
```
up
```

**Why this output occurs.** The literal union restricts `move` to exactly four strings. Assigning `"forward"` is a compile-time error because it is not a member of `Direction`. This pattern is common for status fields and configuration options.

### Real-World Cases

- **API status fields**: `type Status = "pending" | "success" | "error"` prevents invalid status strings.
- **Configuration keys**: Literal unions document and enforce allowed option values.
- **UI text**: User-facing messages are typed as `string` and checked for accidental non-string assignment.

---

## 2. `number`

### Definitions

**Core Definition.** The `number` type represents **double-precision 64-bit IEEE 754 floating-point values**, covering both integers and floating-point numbers.

**Technical Definition.** JavaScript and TypeScript do not distinguish between integers and floats at the type level; every numeric value is a `number` . `number` supports decimal, hexadecimal, binary, octal, and exponential notation, as well as `NaN` and `Infinity`.

**Beginner-Friendly Explanation.** A `number` is **any numeric value** — `42`, `3.14`, `-7`, `1e6`. TypeScript treats all of these as the same type. You cannot annotate a variable as “integer only” with a primitive type.

### Purposes

- **To represent all numeric quantities** in JavaScript programs.
- **To enforce numeric operations** such as arithmetic, comparisons, and bitwise operations.
- **To model numeric literal unions** for finite numeric choices.

### Syntax Rules and Structure

```
let variableName: number = 42;
```

- `: number` — type annotation.
- `= 42` — numeric literal initializer.

**Syntax rules.**

- Use lowercase `number`, **not** uppercase `Number`. `Number` is the boxed wrapper type .
- Numeric literals include decimals (`3.14`), hex (`0xFF`), binary (`0b1010`), octal (`0o17`), and exponentials (`1e3`).
- `NaN` and `Infinity` are of type `number`.
- Numeric separators (`1_000_000`) are supported for readability.

**Constraints and limitations.**

- `number` cannot represent integers beyond `Number.MAX_SAFE_INTEGER` (2⁵³ − 1) precisely; use `bigint` for arbitrary-precision integers.
- Integer and floating-point distinctions are **not** enforceable at the type level with `number`.
- `NaN` is of type `number` and is not equal to itself.

### Annotated Complete Step by Step Code Examples

#### Example 1: Numeric annotation and arithmetic

```typescript
// example_number.ts
// Step 1: Annotate numeric variables.
let price: number = 19.99;
let quantity: number = 3;

// Step 2: Perform arithmetic.
const total = price * quantity;

// Step 3: Log with fixed precision.
console.log(total.toFixed(2)); // Expected: 59.97
```

**Expected Output:**
```
59.97
```

**Why this output occurs.** Both operands are `number`, so multiplication is statically valid. `toFixed(2)` formats the result to two decimal places. The compiler erases the annotations; the arithmetic is performed by JavaScript at runtime.

#### Example 2: Numeric literal union

```typescript
// example_number_literal.ts
// Step 1: Define a union of allowed numeric literals.
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;

// Step 2: Declare and assign.
let roll: DiceRoll = 4;

// Step 3: Uncomment to see an error.
// roll = 7;
// Error: Type '7' is not assignable to type 'DiceRoll'.

console.log(roll); // Expected: 4
```

**Expected Output:**
```
4
```

**Why this output occurs.** The literal union restricts `roll` to the six valid dice values. Assigning `7` is rejected at compile time, even though `7` is a valid `number`. Literal unions provide finer-grained numeric constraints than the general `number` type.

### Real-World Cases

- **Financial calculations**: Monetary amounts typed as `number` (though for exact decimal arithmetic, consider branded types or integer cents).
- **Game mechanics**: Dice rolls, card values, and level ranges modeled as numeric literal unions.
- **Charting and visualization**: Coordinates, dimensions, and scale factors typed as `number`.

---

## 3. `boolean`

### Definitions

**Core Definition.** The `boolean` type represents **logical values** — exactly `true` and `false`.

**Technical Definition.** In TypeScript, `boolean` is a primitive type with two possible values. Boolean expressions result from comparison operators (`===`, `!==`, `<`, `>`), logical operators (`&&`, `||`, `!`), and predicates. Type guards often return `boolean` or a type predicate.

**Beginner-Friendly Explanation.** A `boolean` is **yes or no** — `true` or `false`. It is used for conditions, flags, and any situation where the answer is binary.

### Purposes

- **To represent truth values** in conditions and flags.
- **To enforce binary logic** in control flow.
- **To return the results of predicates and type guards**.

### Syntax Rules and Structure

```
let variableName: boolean = true;
```

- `: boolean` — type annotation.
- `= true` — boolean literal initializer.

**Syntax rules.**

- Use lowercase `boolean`, **not** uppercase `Boolean` .
- Boolean literals are exactly `true` and `false`.
- Type predicates (`x is T`) are assignable to positions expecting `boolean`.

**Constraints and limitations.**

- JavaScript truthiness (e.g., `0`, `""`, `null` being falsy) is a **runtime** concept; TypeScript’s `boolean` type only covers actual `true` and `false`.
- A boolean variable cannot hold truthy/falsy non-boolean values without a type error.

### Annotated Complete Step by Step Code Examples

#### Example 1: Boolean flag and condition

```typescript
// example_boolean.ts
// Step 1: Annotate a boolean flag.
let isLoggedIn: boolean = false;

// Step 2: Use it in a condition.
if (isLoggedIn) {
  console.log("Welcome back");
} else {
  console.log("Please log in");
}
// Expected: Please log in
```

**Expected Output:**
```
Please log in
```

**Why this output occurs.** `isLoggedIn` is `false`, so the `else` branch executes. The boolean type ensures that only `true` or `false` can be assigned; assigning `1` or `"yes"` would be a compile-time error.

#### Example 2: Boolean-returning predicate

```typescript
// example_boolean_predicate.ts
// Step 1: Define a predicate returning boolean.
function isEven(n: number): boolean {
  return n % 2 === 0;
}

// Step 2: Use it.
console.log(isEven(4)); // Expected: true
console.log(isEven(7)); // Expected: false
```

**Expected Output:**
```
true
false
```

**Why this output occurs.** `n % 2 === 0` produces a `boolean`, satisfying the return type annotation. The compiler verifies that the function returns `boolean` on all code paths.

### Real-World Cases

- **Feature flags**: `const isFeatureEnabled: boolean` controls whether a feature is active.
- **Form validation**: `isValid: boolean` tracks whether a form passes validation.
- **Type guards**: Predicate functions return `boolean` (often as `x is T`) to narrow types.

---

## 4. `bigint`

### Definitions

**Core Definition.** The `bigint` type represents **arbitrary-precision integers**, capable of representing integers beyond `Number.MAX_SAFE_INTEGER`.

**Technical Definition.** `bigint` was introduced in ECMAScript 2020 and is supported by TypeScript when the compilation target is `ES2020` or later. BigInt literals are written with an `n` suffix (e.g., `123n`). `bigint` and `number` are **not** interchangeable in arithmetic without explicit conversion.

**Beginner-Friendly Explanation.** A `bigint` is a **really big whole number** that `number` cannot represent precisely. If you need integers larger than about 9 quadrillion, use `bigint`. You write them with an `n` at the end: `9007199254740993n`.

### Purposes

- **To represent integers beyond the safe range of `number`** without precision loss.
- **To perform exact integer arithmetic** in domains such as cryptography, finance, and distributed systems.
- **To model identifiers and hashes** that exceed 53-bit precision.

### Syntax Rules and Structure

```
let variableName: bigint = 123n;
```

- `: bigint` — type annotation.
- `= 123n` — bigint literal with the `n` suffix.

**Syntax rules.**

- BigInt literals **must** use the `n` suffix: `100n`, not `100`.
- The target must be **ES2020 or later**; older targets do not support `bigint` .
- Mixing `bigint` and `number` in arithmetic operations is a **compile-time error**; explicit conversion is required.
- `BigInt()` can be used to convert strings or numbers to `bigint`.

**Constraints and limitations.**

- `bigint` cannot be used with `Math` methods or bitwise operators that expect `number` without conversion.
- JSON does not support `bigint`; serialization requires custom handling.
- Performance is generally slower than `number` arithmetic.
- `bigint` is **not** available in environments that do not support ES2020.

### Annotated Complete Step by Step Code Examples

#### Example 1: Large integer arithmetic

```typescript
// example_bigint.ts
// Step 1: Annotate bigint values.
let largeNumber: bigint = 9007199254740993n;

// Step 2: Perform exact arithmetic.
const doubled = largeNumber * 2n;

// Step 3: Log.
console.log(doubled.toString());
// Expected: 18014398509481986
```

**Expected Output:**
```
18014398509481986
```

**Why this output occurs.** The `n` suffix creates bigint literals. Multiplication uses `2n` (a bigint), avoiding the mixed-type error. The result is exact, unlike `number` arithmetic which would lose precision beyond 2⁵³ − 1.

#### Example 2: Mixing `number` and `bigint` is an error

```typescript
// example_bigint_mix.ts
let big: bigint = 10n;
let num: number = 5;

// Uncommenting the next line causes a compile-time error.
// const sum = big + num;
// Error: Operator '+' cannot be applied to types 'bigint' and 'number'.

console.log(big + 5n); // Expected: 15n (as bigint)
```

**Expected Output:**
```
15
```

**Why this output occurs.** TypeScript forbids implicit mixing of `bigint` and `number` to prevent precision loss. Adding `5n` (a bigint) to `big` is valid. The `console.log` prints the bigint value, which may appear as `15n` in some environments or `15` in others.

### Real-World Cases

- **Financial systems**: Exact integer arithmetic for monetary amounts in minor units (cents, satoshis).
- **Cryptography**: Large prime numbers and modular exponentiation.
- **Database IDs**: 64-bit identifiers that exceed JavaScript’s safe integer range.

---

## 5. `symbol`

### Definitions

**Core Definition.** The `symbol` type represents **unique, immutable primitive values** often used as object property keys to avoid naming collisions.

**Technical Definition.** Symbols are created with `Symbol()` and are guaranteed to be unique. `Symbol.for()` creates or retrieves a symbol from a global registry. In TypeScript, `symbol` is a primitive type, and `unique symbol` is a subtype used for `const` declarations and `readonly static` properties.

**Beginner-Friendly Explanation.** A `symbol` is a **guaranteed-unique identifier**. Even if two symbols have the same description, they are different values. Symbols are often used as “hidden” property keys that cannot accidentally clash with string keys.

### Purposes

- **To create unique property keys** that avoid collisions with other properties.
- **To define well-known symbols** such as `Symbol.iterator` that customize language behavior.
- **To model unique tokens** in APIs where nominal distinction is needed.

### Syntax Rules and Structure

```
let variableName: symbol = Symbol("description");
```

- `: symbol` — type annotation.
- `= Symbol("description")` — symbol creation with an optional description.

**Syntax rules.**

- `Symbol()` creates a new unique symbol every time it is called.
- `Symbol.for("key")` uses a global registry; repeated calls with the same key return the same symbol.
- `unique symbol` is only allowed on `const` declarations and `readonly static` properties.
- Symbols can be used as computed property names in object literals.

**Constraints and limitations.**

- Symbols are **not** enumerable in `for...in` loops or `Object.keys()`; they require `Object.getOwnPropertySymbols()`.
- JSON serialization ignores symbol-keyed properties.
- `unique symbol` is a narrow subtype; a general `symbol` annotation does not preserve uniqueness.

### Annotated Complete Step by Step Code Examples

#### Example 1: Symbol as a unique property key

```typescript
// example_symbol.ts
// Step 1: Create a unique symbol key.
const idKey: unique symbol = Symbol("id");

// Step 2: Use it as a computed property key.
const user = {
  name: "Ada",
  [idKey]: 123,
};

// Step 3: Access via the symbol.
console.log(user.name);        // Expected: Ada
console.log(user[idKey]);      // Expected: 123
```

**Expected Output:**
```
Ada
123
```

**Why this output occurs.** `idKey` is a `unique symbol`, so it is guaranteed not to clash with string keys. The property is accessible only via the symbol reference, providing a form of encapsulation. `user.name` accesses the string-keyed property.

#### Example 2: Well-known symbol for iteration

```typescript
// example_symbol_iterator.ts
// Step 1: Define an iterable object.
const iterable = {
  *[Symbol.iterator]() {
    yield 1;
    yield 2;
    yield 3;
  },
};

// Step 2: Iterate with for...of.
for (const value of iterable) {
  console.log(value); // Expected: 1, 2, 3
}
```

**Expected Output:**
```
1
2
3
```

**Why this output occurs.** `Symbol.iterator` is a well-known symbol that the `for...of` loop looks up on the object. Defining a generator method at that symbol key makes the object iterable. TypeScript types `Symbol.iterator` as a symbol, and the generator method is structurally compatible.

### Real-World Cases

- **Library metadata**: Libraries attach non-enumerable symbol-keyed metadata to objects.
- **Dependency injection**: Symbols serve as unique tokens for service registration.
- **Custom iteration**: Objects implement `Symbol.iterator` to support `for...of`.

---

## 6. `null`

### Definitions

**Core Definition.** The `null` type represents the **intentional absence of a value**.

**Technical Definition.** `null` is a primitive value in JavaScript and a corresponding type in TypeScript. Under `strictNullChecks`, `null` is **not** assignable to every type; it is only assignable to itself, `any`, and `unknown`. Without `strictNullChecks`, `null` is assignable to all types, which defeats null-safety .

**Beginner-Friendly Explanation.** `null` means **“no value on purpose”**. When a function looks up a record and finds nothing, it might return `null`. TypeScript’s strict mode forces you to handle the possibility of `null` before using a value.

### Purposes

- **To represent an explicitly absent value** (as opposed to `undefined`, which often means “not yet assigned”).
- **To force null checks** under `strictNullChecks`, preventing null-pointer-style errors.
- **To model optional or missing data** in APIs and databases.

### Syntax Rules and Structure

```
let variableName: null = null;
let nullable: string | null = null;
```

- `: null` — annotates a variable that can only be `null`.
- `string | null` — a union indicating the value may be a string or null.

**Syntax rules.**

- Under `strictNullChecks`, `null` must be explicitly included in the type when a value may be null.
- `x === null` and `x !== null` are the recommended comparisons.
- `x == null` checks for **both** `null` and `undefined` (loose equality).

**Constraints and limitations.**

- Without `strictNullChecks`, `null` is assignable to every type, and TypeScript cannot catch null-related errors .
- `null` is a primitive value, but `typeof null` returns `"object"` — a historical JavaScript quirk.
- Optional properties and parameters automatically include `undefined`, not `null` .

### Annotated Complete Step by Step Code Examples

#### Example 1: Nullable type and null check

```typescript
// example_null.ts
// Step 1: Annotate a nullable string.
let middleName: string | null = null;

// Step 2: Narrow with a null check.
if (middleName !== null) {
  console.log(middleName.toUpperCase());
} else {
  console.log("No middle name");
}
// Expected: No middle name
```

**Expected Output:**
```
No middle name
```

**Why this output occurs.** `middleName` is `null`, so the `else` branch executes. The null check narrows the type: inside the `if` block, `middleName` is `string`. Without the check, calling `.toUpperCase()` on a possibly-null value would be a compile-time error under `strictNullChecks`.

#### Example 2: `== null` checks both null and undefined

```typescript
// example_null_loose.ts
let value: string | null | undefined = undefined;

if (value == null) {
  console.log("Value is null or undefined");
  // Expected: Value is null or undefined
}
```

**Expected Output:**
```
Value is null or undefined
```

**Why this output occurs.** The loose equality check `value == null` is `true` for both `null` and `undefined`. This is a common idiom for guarding against both nullish values. TypeScript understands this narrowing and treats `value` as `string` in the `else` branch.

### Real-World Cases

- **Database queries**: A query that finds no record returns `null`, and the type `Record | null` forces handling of the missing case.
- **DOM APIs**: `document.querySelector()` returns `Element | null`; strict null checks force a null guard before calling element methods.
- **Optional configuration**: `timeout: number | null` distinguishes “use default” (null) from “no timeout” (0).

---

## 7. `undefined`

### Definitions

**Core Definition.** The `undefined` type represents a value that has **not been assigned** or a property that **does not exist**.

**Technical Definition.** `undefined` is a primitive value in JavaScript. Under `strictNullChecks`, `undefined` is not assignable to every type; it is only assignable to itself, `any`, `unknown`, and `void` . Optional parameters and properties automatically include `undefined` in their types even when not explicitly annotated .

**Beginner-Friendly Explanation.** `undefined` means **“no value yet”** or **“this doesn’t exist”**. It is the default value of uninitialized variables and the value returned when accessing a missing property. TypeScript’s strict mode forces you to check for `undefined` before using potentially missing values.

### Purposes

- **To represent uninitialized or missing values**.
- **To model optional parameters and properties** through automatic `| undefined` inclusion.
- **To force undefined checks**, preventing “cannot read property of undefined” runtime errors.

### Syntax Rules and Structure

```
let variableName: undefined = undefined;
let optional: string | undefined = undefined;
```

- `: undefined` — annotates a variable that can only be `undefined`.
- `string | undefined` — a union indicating the value may be a string or undefined.

**Syntax rules.**

- Under `strictNullChecks`, `undefined` must be explicitly included in the type when a value may be undefined, except for optional parameters and properties, which include it automatically .
- `x === undefined` and `x !== undefined` are explicit checks.
- `x == null` checks for both `undefined` and `null`.
- `x ?? fallback` (nullish coalescing) provides a fallback for `null` or `undefined`.

**Constraints and limitations.**

- Without `strictNullChecks`, `undefined` is assignable to every type, hiding potential errors .
- Optional chaining (`?.`) and nullish coalescing (`??`) help handle `undefined` but do not substitute for explicit validation when semantics require it.
- `undefined` is a valid value in JSON only as `null`; it is not serialized as a key-value pair.

### Annotated Complete Step by Step Code Examples

#### Example 1: Optional property and undefined check

```typescript
// example_undefined.ts
// Step 1: Define an interface with an optional property.
interface Config {
  timeout?: number;  // type is number | undefined
}

// Step 2: Create a config without timeout.
const config: Config = {};

// Step 3: Check for undefined before use.
if (config.timeout !== undefined) {
  console.log(config.timeout.toFixed(1));
} else {
  console.log("No timeout set");
}
// Expected: No timeout set
```

**Expected Output:**
```
No timeout set
```

**Why this output occurs.** The optional property `timeout?: number` has type `number | undefined`. Accessing `config.timeout` yields `undefined`, so the `else` branch runs. Without the check, calling `.toFixed()` would be a compile-time error under `strictNullChecks`.

#### Example 2: Nullish coalescing for default values

```typescript
// example_nullish_coalescing.ts
function greet(name?: string): string {
  // name is string | undefined
  const displayName = name ?? "Guest";
  return `Hello, ${displayName}`;
}

console.log(greet());          // Expected: Hello, Guest
console.log(greet("Ada"));     // Expected: Hello, Ada
```

**Expected Output:**
```
Hello, Guest
Hello, Ada
```

**Why this output occurs.** The `??` operator returns the right-hand side when the left-hand side is `null` or `undefined`. When `name` is `undefined` (no argument passed), `displayName` becomes `"Guest"`. When `name` is `"Ada"`, it remains `"Ada"`.

### Real-World Cases

- **Optional configuration**: Settings that may not be provided are typed as `T | undefined` and defaulted with `??`.
- **Array access**: `array[index]` may be `undefined` under `noUncheckedIndexedAccess`; guards are required before use.
- **Function arguments**: Optional parameters are automatically `T | undefined`, encouraging explicit handling.

---

## 8. `void`

### Definitions

**Core Definition.** The `void` type represents the **absence of a meaningful return value** — typically used as the return type of functions that perform side effects and do not return a value.

**Technical Definition.** `void` is a special TypeScript type that is the return type of functions with no `return` statement or with a bare `return;`. Under `strictNullChecks`, `undefined` is assignable to `void`, but `void` is not assignable to other types . In callback positions, `void` is the recommended return type for functions whose return value is ignored .

**Beginner-Friendly Explanation.** `void` means **“this function doesn’t give back anything useful”**. It is the type of a function that logs something, modifies an object, or throws an error — but does not return a value you are supposed to use.

### Purposes

- **To document that a function’s return value is not intended to be used**.
- **To make callback types safer** by preventing accidental use of return values .
- **To model side-effect-only functions** with a clear type-level signal.

### Syntax Rules and Structure

```
function functionName(): void {
  // side effects only
}
```

- `): void` — return type annotation indicating no meaningful return value.

**Syntax rules.**

- Use `void` for callbacks whose return value is ignored; this is safer than `any` .
- A function annotated `void` may use a bare `return;` but may not return a value (unless the value is `undefined`).
- `undefined` is assignable to `void` under `strictNullChecks`.

**Constraints and limitations.**

- `void` is **not** the same as `never`; a `void` function returns normally (with no value), while a `never` function never returns at all .
- Using a `void` return value in an expression is a compile-time error, which is the intended safety benefit.
- `void` is often confused with `undefined`; `undefined` is a value, while `void` is a return-type signal.

### Annotated Complete Step by Step Code Examples

#### Example 1: Void callback prevents accidental return-value use

```typescript
// example_void.ts
// Step 1: Define a function accepting a void callback.
function runCallback(callback: () => void): void {
  callback();
}

// Step 2: Pass a callback that returns a number.
const result = runCallback(() => 42);

// Step 3: The return value of the callback is ignored and inaccessible.
// Uncommenting the next line is a compile-time error.
// console.log(result);

console.log("Callback executed");
// Expected: Callback executed
```

**Expected Output:**
```
Callback executed
```

**Why this output occurs.** The callback `() => 42` is assignable to `() => void` because TypeScript allows functions returning values to be used where `void` is expected — the return value is simply discarded. However, `runCallback` itself returns `void`, so `result` has type `void` and cannot be used in an expression. This prevents accidental reliance on the callback’s return value .

#### Example 2: Void function with side effects

```typescript
// example_void_side_effect.ts
function logMessage(message: string): void {
  console.log(`[LOG] ${message}`);
}

const logResult = logMessage("Hello");
// logResult has type void; it cannot be used.
console.log("Done"); // Expected: Done
```

**Expected Output:**
```
[LOG] Hello
Done
```

**Why this output occurs.** `logMessage` performs a side effect but returns nothing meaningful. Its return type is `void`. The variable `logResult` receives `undefined` at runtime, but its static type is `void`, preventing misuse.

### Real-World Cases

- **Event handlers**: `onClick: () => void` callbacks are not expected to return values.
- **Logging and telemetry**: Functions that record metrics or emit logs return `void`.
- **Array iteration**: `forEach` takes a `void` callback; the return value of the callback is ignored.

---

## 9. `never`

### Definitions

**Core Definition.** The `never` type represents the **bottom type** — the type of values that **never occur**. It is the return type of functions that never return normally (they always throw or loop forever).

**Technical Definition.** `never` is assignable to every type, but **no type is assignable to `never`** (except `never` itself) . It is used for **exhaustiveness checking** in discriminated unions: if a `default` case receives a value of type `never`, the compiler confirms that all union members have been handled .

**Beginner-Friendly Explanation.** `never` means **“this cannot happen”**. A function that always throws an error never returns, so its return type is `never`. `never` is also a trick for making the compiler check that you have handled every possible case in a union.

### Purposes

- **To model functions that never return** (always throw or loop infinitely).
- **To enforce exhaustiveness checks** in `switch` statements and conditional logic .
- **To represent impossible states** in type-level computations.

### Syntax Rules and Structure

```
function fail(message: string): never {
  throw new Error(message);
}
```

- `): never` — return type for a function that never returns normally.

**Exhaustiveness check pattern:**

```
default:
  const _exhaustive: never = value;
  return _exhaustive;
```

- `_exhaustive: never` — if `value` is not `never`, the assignment is a compile-time error.

**Syntax rules.**

- A function returning `never` must not have any reachable `return` statement; it must throw or loop forever.
- `never` is only assignable to itself; assigning any other type to `never` is an error .
- For function **expressions**, TypeScript infers `never` as the return type when the body always throws; for **declarations**, it infers `void` for backward compatibility .

**Constraints and limitations.**

- `never` is often confused with `void`; `void` returns normally with no value, `never` never returns .
- Inference for function declarations defaults to `void` even when the body always throws; explicit annotation is required .
- `never` cannot be used as a value type in most practical scenarios; it is primarily a type-level tool.

### Annotated Complete Step by Step Code Examples

#### Example 1: Function that always throws

```typescript
// example_never.ts
// Step 1: Define a function returning never.
function fail(message: string): never {
  throw new Error(message);
}

// Step 2: Call it in a context where a value is expected.
function process(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  } else if (typeof value === "number") {
    return value.toFixed(2);
  }
  return fail("Unexpected type");
}

console.log(process("hello")); // Expected: HELLO
console.log(process(3.14159)); // Expected: 3.14
```

**Expected Output:**
```
HELLO
3.14
```

**Why this output occurs.** `fail` returns `never`, so TypeScript knows it never returns normally. The `return fail(...)` call satisfies the compiler’s requirement that all code paths return a value, even though the `never` path never actually produces a value. Without `never`, the compiler might complain about a missing return .

#### Example 2: Exhaustiveness checking with discriminated unions

```typescript
// example_never_exhaustive.ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    default:
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}

console.log(area({ kind: "square", side: 2 })); // Expected: 4
```

**Expected Output:**
```
4
```

**Why this output occurs.** After handling `"circle"` and `"square"`, the `default` case narrows `shape` to `never`. The assignment `const _exhaustive: never = shape` succeeds because `shape` is `never`. If a new member were added to `Shape` (e.g., `"triangle"`), `shape` would no longer be `never` in the `default` case, and the assignment would fail — catching the incomplete handling at compile time .

### Real-World Cases

- **Redux reducers**: Exhaustiveness checks ensure every action type is handled.
- **State machines**: `never` confirms that all states and transitions are covered.
- **Error handling**: Functions that always throw (e.g., `assertNever`) use `never` to satisfy return-type requirements.

---

## 10. `any`

### Definitions

**Core Definition.** The `any` type is a **wildcard type** that **disables type checking** for the value it annotates. Any operation is permitted, and `any` is assignable to and from every type.

**Technical Definition.** `any` opts out of the type system entirely for the affected value. The compiler treats `any` as “please turn off type checking for this thing” . It is assignable to and from every type, and property accesses, calls, and constructions on `any` are unchecked. `any` propagates: expressions involving `any` typically produce `any`.

**Beginner-Friendly Explanation.** `any` means **“TypeScript, stop checking this — I’ll handle it”**. It is an escape hatch. Using `any` is like putting a “ignore type errors here” sign on a value. It is useful during JavaScript migration but dangerous in production TypeScript because it can hide bugs.

### Purposes

- **To facilitate gradual migration** from JavaScript to TypeScript by temporarily disabling checking on unported code .
- **To represent values that are genuinely dynamic** in rare cases where no better type exists.
- **To interoperate with untyped third-party code** when type definitions are unavailable.

### Syntax Rules and Structure

```
let variableName: any = anything;
```

- `: any` — disables type checking for `variableName`.

**Syntax rules.**

- `any` is assignable to and from every type.
- Any property access, call, or construction on an `any` value is permitted.
- Under `--noImplicitAny`, unannotated positions that would otherwise become `any` produce errors.
- `any` propagates through expressions: `any + number` is `any`.

**Constraints and limitations.**

- **`any` defeats the purpose of TypeScript**; it should be avoided in production code .
- `any` is **not type-safe**; runtime errors are not caught at compile time .
- `any` is **deprecated as a default choice**; prefer `unknown` when the type is genuinely unknown .
- Overuse of `any` is considered an anti-pattern; new code should not introduce `any` .

### Annotated Complete Step by Step Code Examples

#### Example 1: `any` disables checking

```typescript
// example_any.ts
// Step 1: Annotate as any.
let value: any = 42;

// Step 2: Reassign to a different type — allowed.
value = "hello";

// Step 3: Access a property that may not exist — allowed.
console.log(value.toUpperCase()); // Expected: HELLO

// Step 4: Assign to a typed variable — allowed because source is any.
let s: string = value;
console.log(s); // Expected: hello
```

**Expected Output:**
```
HELLO
hello
```

**Why this output occurs.** `any` disables static checking, so reassignment, property access, and assignment to `string` are all permitted. At runtime, `value` is a string, so `toUpperCase()` works. However, if `value` had been a number, the call would fail at runtime — TypeScript would not catch it .

#### Example 2: `any` hides a bug

```typescript
// example_any_bug.ts
let data: any = { name: "Ada" };

// This compiles but crashes at runtime.
console.log(data.age.toFixed(0));
// Runtime error: Cannot read properties of undefined (reading 'toFixed')
```

**Expected Output (runtime error):**
```
TypeError: Cannot read properties of undefined (reading 'toFixed')
```

**Why this output occurs.** `data` is `any`, so `data.age` is unchecked and has type `any`. Calling `.toFixed()` on `undefined` is not a compile-time error because `any` suppresses checking. At runtime, `data.age` is `undefined`, and the call throws. This demonstrates why `any` is dangerous .

### Real-World Cases

- **JavaScript migration**: Temporarily annotating unported modules as `any` while incrementally adding types .
- **Untyped third-party libraries**: Using `any` as a bridge until `@types` packages are available.
- **Legacy code**: Isolating unsafe areas while planning refactoring.

---

## 11. `unknown`

### Definitions

**Core Definition.** The `unknown` type is the **type-safe counterpart of `any`**. It can hold any value, but **no operations are permitted** on an `unknown` value until it is **narrowed** to a more specific type.

**Technical Definition.** `unknown` is the **top type** in TypeScript’s type system . Every type is assignable to `unknown`, but `unknown` is assignable only to `unknown` (and `any`). Before using an `unknown` value, the programmer must perform a type check (`typeof`, `instanceof`, `in`, custom type guard, or assertion) to narrow it.

**Beginner-Friendly Explanation.** `unknown` means **“I don’t know what this is, so I’ll be careful”**. Unlike `any`, which lets you do anything (and crash at runtime), `unknown` forces you to **prove** what the value is before using it. It is the safe choice for data from external sources.

### Purposes

- **To safely represent values whose type is genuinely unknown**, such as parsed JSON or third-party API responses .
- **To force runtime validation** before a value is used .
- **To replace `any`** in most situations while preserving type safety .

### Syntax Rules and Structure

```
let variableName: unknown = value;
```

- `: unknown` — accepts any value but requires narrowing before use.

**Narrowing pattern:**

```
if (typeof variableName === "string") {
  variableName.toUpperCase();  // narrowed to string
}
```

- `typeof` check — a runtime guard that narrows `unknown` to `string` within the branch.

**Syntax rules.**

- Every type is assignable to `unknown`.
- `unknown` is assignable only to `unknown` and `any`.
- Before property access, calls, or operations, `unknown` must be narrowed by a type guard, assertion, or equality check .

**Constraints and limitations.**

- `unknown` requires **more code** than `any` because narrowing is mandatory.
- Type assertions (`as T`) can bypass narrowing but do **not** perform runtime checks; they should be used only when you have runtime evidence.
- `unknown` is **not** a substitute for proper runtime validation (e.g., schema validation with Zod or io-ts).

### Annotated Complete Step by Step Code Examples

#### Example 1: `unknown` requires narrowing

```typescript
// example_unknown.ts
// Step 1: Annotate as unknown.
let value: unknown = "hello";

// Step 2: Uncommenting the next line is a compile-time error.
// console.log(value.toUpperCase());
// Error: Object is of type 'unknown'.

// Step 3: Narrow with typeof.
if (typeof value === "string") {
  console.log(value.toUpperCase()); // Expected: HELLO
}
```

**Expected Output:**
```
HELLO
```

**Why this output occurs.** `unknown` does not permit property access or method calls until narrowed. The `typeof value === "string"` guard narrows `value` to `string` inside the `if` block, making `.toUpperCase()` valid. This ensures that the operation is safe at runtime .

#### Example 2: `unknown` with custom type guard

```typescript
// example_unknown_guard.ts
interface User {
  name: string;
  age: number;
}

function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "name" in value &&
    "age" in value &&
    typeof (value as User).name === "string" &&
    typeof (value as User).age === "number"
  );
}

function process(input: unknown): string {
  if (isUser(input)) {
    return `${input.name} (${input.age})`;
  }
  return "Not a user";
}

console.log(process({ name: "Ada", age: 36 }));
// Expected: Ada (36)
console.log(process({ name: "Ada" }));
// Expected: Not a user
```

**Expected Output:**
```
Ada (36)
Not a user
```

**Why this output occurs.** `isUser` is a user-defined type guard that checks the runtime shape of the value. If it returns `true`, TypeScript narrows `input` to `User` inside the `if` block, allowing safe access to `name` and `age`. The second call fails the guard because `age` is missing, so the fallback string is returned.

### Real-World Cases

- **JSON parsing**: `JSON.parse` returns `any`; annotating the result as `unknown` forces validation before use.
- **API responses**: External data typed as `unknown` and narrowed with schema validators (Zod, io-ts).
- **Error handling**: `catch` clause variables are `unknown` under `useUnknownInCatchVariables`; narrowing is required before accessing error properties.

---

## 12. `object`

### Definitions

**Core Definition.** The `object` type represents **any non-primitive value** — that is, any value that is not `number`, `string`, `boolean`, `bigint`, `symbol`, `null`, or `undefined`.

**Technical Definition.** `object` (lowercase) was introduced in TypeScript 2.2 to represent non-primitive types . It is **distinct from `Object` (uppercase) and `{}`**:

- `object` — any non-primitive value (arrays, functions, class instances, plain objects).
- `Object` (uppercase) — instances of class `Object`; accepts all non-nullish values, including primitives .
- `{}` (empty object type) — accepts all non-nullish values, including primitives .

**Beginner-Friendly Explanation.** `object` means **“a real object, not a primitive”**. Numbers, strings, and booleans are **not** `object`. Arrays, functions, and `{}` literals **are** `object`. Use `object` when you want to accept any non-primitive value without specifying its properties.

### Purposes

- **To accept any non-primitive value** without committing to a specific shape.
- **To exclude primitives** at the type level, which is useful for APIs that operate on object-like data .
- **To avoid the pitfalls of `Object` and `{}`**, which include primitives in their domains .

### Syntax Rules and Structure

```
function acceptObject(value: object): void { ... }
```

- `value: object` — accepts any non-primitive value.

**Syntax rules.**

- `object` is assignable from arrays, functions, class instances, and plain objects.
- `object` is **not** assignable from `number`, `string`, `boolean`, `bigint`, `symbol`, `null`, or `undefined`.
- Use lowercase `object`, **not** uppercase `Object`, unless you specifically need the boxed-object type .

**Constraints and limitations.**

- `object` does **not** describe any properties; you cannot access properties without narrowing to a more specific type.
- `{}` and `Object` accept primitives (except nullish), so they are often **too permissive** .
- `object` is **not** the same as “plain object”; arrays and functions are `object` too.

### Annotated Complete Step by Step Code Examples

#### Example 1: `object` accepts non-primitives only

```typescript
// example_object.ts
function logObject(value: object): void {
  console.log("Received an object");
}

logObject({ a: 1 });      // OK
logObject([1, 2, 3]);     // OK — arrays are objects
logObject(() => {});      // OK — functions are objects

// Uncommenting any of these is a compile-time error.
// logObject(42);
// logObject("hello");
// logObject(null);
```

**Expected Output:**
```
Received an object
Received an object
Received an object
```

**Why this output occurs.** `object` accepts any non-primitive value. Plain objects, arrays, and functions are all non-primitive, so they satisfy `object`. Numbers, strings, and `null` are primitives or nullish and are rejected at compile time .

#### Example 2: `object` vs `Object` vs `{}`

```typescript
// example_object_comparison.ts
let obj: object = { a: 1 };       // OK
// let obj2: object = 42;         // Error: number is a primitive
let obj3: Object = 42;            // OK — Object accepts primitives (non-nullish)
let obj4: {} = "hello";           // OK — {} accepts non-nullish values

console.log(typeof obj3);         // Expected: number
console.log(typeof obj4);         // Expected: string
```

**Expected Output:**
```
number
string
```

**Why this output occurs.** `Object` and `{}` accept non-nullish primitives, while `object` does not . This example illustrates why lowercase `object` is the correct choice when you want to **exclude primitives** and accept only genuine object-like values .

### Real-World Cases

- **Utility functions**: Functions like `Object.keys` conceptually accept `object` and reject primitives.
- **Reflection and metadata**: APIs that inspect non-primitive values use `object` to exclude primitives.
- **Avoiding `Object` and `{}`**: Replacing permissive `Object`/`{}` annotations with `object` when primitives should not be accepted .

---

## Primitive Type Best Practices

### Definitions

**Core Definition.** **Primitive type best practices** are conventions and rules for choosing, annotating, and using TypeScript’s primitive types in ways that maximize safety, clarity, and tooling effectiveness.

**Technical Definition.** Best practices include preferring **lowercase primitive types** over boxed types, using **`unknown` instead of `any`**, enabling **`strictNullChecks`**, using **`void` for callbacks**, leveraging **literal types and unions** for finite value sets, and avoiding **primitive obsession** through branded types.

**Beginner-Friendly Explanation.** Best practices are **the habits that prevent bugs**. They include using `string` instead of `String`, `unknown` instead of `any`, and strict null checks. They also mean using literal unions for fixed sets of values and branded types for domain-specific primitives.

### Purposes

- **To prevent entire classes of bugs**, especially null/undefined errors and `any`-hidden errors.
- **To keep types precise and expressive**, avoiding overly wide or permissive types.
- **To improve tooling**, since accurate primitive types power autocompletion and refactoring.
- **To align with official TypeScript guidance** and community standards.

### Syntax Rules and Structure

**Key best-practice rules:**

1. **Use lowercase primitives**: `string`, `number`, `boolean`, `symbol`, `bigint`, `object` — **not** `String`, `Number`, `Boolean`, `Symbol`, `Object` .
2. **Prefer `unknown` over `any`** when the type is genuinely unknown; narrow before use .
3. **Enable `strict: true`**, including `strictNullChecks`, to distinguish `null`/`undefined` from other types .
4. **Use `void` for callbacks** whose return value is ignored .
5. **Use literal unions** for finite sets of allowed values .
6. **Avoid primitive obsession**: brand primitives when domain semantics require nominal distinction (e.g., `UserId` vs `OrderId`) .
7. **Never use `any` in new production code**; restrict it to migration scenarios .

**Component breakdown (`tsconfig.json` strictness):**

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

- `strict: true` — enables the full strictness family.
- `noImplicitAny: true` — errors on implicit `any`.
- `strictNullChecks: true` — separates `null` and `undefined` from other types.

**Constraints and limitations.**

- Strict settings may surface many errors in existing codebases; migrate incrementally.
- `unknown` requires more code than `any`; the safety benefit justifies the verbosity.
- Branded types require assertions at creation points and are a user-land pattern.

### Annotated Complete Step by Step Code Examples

#### Example 1: `unknown` with runtime validation (best practice)

```typescript
// example_best_unknown.ts
function parseName(input: unknown): string {
  if (typeof input !== "string") {
    throw new Error("Expected a string");
  }
  return input.trim();
}

console.log(parseName("  Ada  ")); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `unknown` forces a runtime check before the value is treated as a string. This prevents the unsafe behavior that `any` would allow, where `input.trim()` would compile but fail at runtime .

#### Example 2: Literal union instead of raw `string` (best practice)

```typescript
// example_best_literal_union.ts
type OrderStatus = "pending" | "shipped" | "delivered";

function renderStatus(status: OrderStatus): string {
  switch (status) {
    case "pending": return "Pending";
    case "shipped": return "Shipped";
    case "delivered": return "Delivered";
  }
}

console.log(renderStatus("shipped")); // Expected: Shipped
```

**Expected Output:**
```
Shipped
```

**Why this output occurs.** The literal union restricts `status` to three valid strings. A typo like `"shiped"` would be a compile-time error. This is far safer than annotating `status: string`, which would accept any string .

#### Example 3: `void` callback (best practice)

```typescript
// example_best_void.ts
function forEachItem<T>(items: T[], callback: (item: T) => void): void {
  for (const item of items) {
    callback(item);
  }
}

forEachItem([1, 2, 3], (n) => console.log(n * 2));
// Expected: 2, 4, 6
```

**Expected Output:**
```
2
4
6
```

**Why this output occurs.** `void` communicates that the callback’s return value is intentionally unused. Using `any` instead would allow accidental use of the return value, defeating the safety benefit .

### Real-World Cases

- **Strict `tsconfig.json`** in CI pipelines prevents `any` leakage and null errors .
- **API response handling** uses `unknown` plus schema validation to safely process external data.
- **Domain modeling** uses literal unions for statuses and branded types for identifiers to prevent primitive obsession .
- **Library callbacks** are typed with `void` returns to signal that the return value is ignored.

---

## References

- TypeScript Handbook — Everyday Types - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html 
- TypeScript Handbook — Do’s and Don’ts - https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html 
- TypeScript Handbook — TypeScript for Functional Programmers (Built-in types) - https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html 
- TypeScript Release Notes 2.0 — `strictNullChecks` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html 
- TypeScript Documentation — `any` vs `unknown` - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#any 
- TypeScript Wiki — FAQ (Type Erasure) - https://github.com/microsoft/TypeScript/wiki/FAQ
- TypeScript Book — `never` type - https://github.com/mbiesiad/typescript-book/blob/master/docs/types/never.md 
- Exploring TypeScript — `object` vs `Object` vs `{}` - https://exploringjs.com/ts/ 
- W3Schools — TypeScript Special Types - https://www.w3schools.com/typescript/typescript_special_types.php 
- TypeScript Documentation — `object` type (TS 2.2 release notes) - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-2.html#object-type 