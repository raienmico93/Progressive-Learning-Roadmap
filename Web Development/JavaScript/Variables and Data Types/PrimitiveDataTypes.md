# JavaScript Primitive Data Types — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Primitive data types are the seven fundamental, immutable value types in JavaScript — String, Number, BigInt, Boolean, `undefined`, `null`, and Symbol — that are not objects and are represented directly at the lowest level of the language implementation.

**Technical Definition:** Per ECMA-262 (ECMAScript Language Specification), the ECMAScript language types are Undefined, Null, Boolean, String, Symbol, Number, BigInt, and Object. All types except Object define immutable values represented directly at the lowest level of the language; these are referred to as primitive values. Primitive values have no methods or properties of their own, but JavaScript automatically "boxes" them into corresponding wrapper objects when property access occurs.

**Beginner-Friendly Explanation:** JavaScript has seven basic "building blocks" for data: text (String), numbers (Number and BigInt), true/false (Boolean), "nothing" (`undefined` and `null`), and unique identifiers (Symbol). Everything else in JavaScript is an object. These primitive values are simple, unchangeable pieces of data — like individual Lego bricks rather than a built structure.

### Key Characteristics

- **Seven types, not objects:** All values that are not objects are primitives; there are exactly seven primitive types.
- **Immutability:** Once a primitive value is created, it cannot be changed. A variable holding a primitive can be reassigned to a new value, but the original value itself is immutable.
- **No methods or properties:** Primitives have no methods of their own, but they behave as if they do through auto-boxing (temporary wrapper objects).
- **`typeof` detection:** All primitive types except `null` can be tested with the `typeof` operator; `typeof null` returns `"object"` due to a historical bug.
- **Wrapper counterparts:** Except for `null` and `undefined`, every primitive has a corresponding wrapper object (`String`, `Number`, `Boolean`, `Symbol`, `BigInt`) that provides methods.

### Prerequisites

- Basic familiarity with JavaScript variables and the `typeof` operator.
- Understanding of the difference between values and objects.
- Awareness of type coercion (implicit conversion between types).

### Related Programming Areas

- **Type systems and type checking:** `typeof`, `instanceof`, and type guards.
- **Memory management:** Primitives are typically stored on the stack (implementation-dependent), while objects are heap-allocated.
- **Comparison and equality:** `===` vs `==` behaviour across primitive types.
- **Serialization:** JSON supports only a subset of primitives (String, Number, Boolean, `null`); BigInt and Symbol are not JSON-serializable.

### Core Concepts / Features

The following sections cover every requested primitive type and related concept using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## String

### Definitions

**Core Definition:** The String type represents textual data as a sequence of UTF-16 code units. Strings are immutable primitive values used to store and manipulate text.

**Technical Definition:** The String type is the set of all ordered sequences of zero or more 16-bit unsigned integer values (UTF-16 code units). Strings are primitives, not objects, but the `String` wrapper object provides methods like `includes()`, `slice()`, and `toUpperCase()`. When a property is accessed on a string primitive, JavaScript auto-boxes it into a temporary `String` object.

**Beginner-Friendly Explanation:** A string is any text inside quotes — `"hello"`, `'JavaScript'`, or `` `template literal` ``. Strings are used for names, messages, HTML content, and anything textual.

### Purposes

- To represent and manipulate textual data (user input, messages, file content).
- To store identifiers, labels, and configuration values.
- To enable text processing (searching, replacing, splitting, formatting).
- To serve as the primary data type for JSON and API communication.

### Syntax Rules and Structure

**String Literals:**
```js
' single quotes '
" double quotes "
` template literals (backticks) `
```

**Component Breakdown:**
- Single and double quotes are interchangeable for basic strings.
- Template literals (backticks) support interpolation (`${expression}`) and multi-line strings.

**Syntax Rules:**
- Strings are immutable; methods like `toUpperCase()` return a new string rather than modifying the original.
- `length` is a property (not a method) that returns the number of UTF-16 code units.
- String indices are zero-based and accessed via bracket notation (`str[0]`).

**Constraints and Limitations:**
- Strings use UTF-16 encoding; some characters (e.g., emoji) require surrogate pairs, making `length` potentially misleading.
- There is no "character" type; single characters are strings of length 1.
- Strings are immutable, so "mutating" operations always produce new strings.

### Annotated Code Examples

#### Example 1: String Creation and Basic Operations

```js
// Step 1: Create strings using different literal forms.
const single = 'Hello';
const double = "World";
const template = `${single}, ${double}!`;

// Step 2: Access the length and individual characters.
console.log(template);       // "Hello, World!"
console.log(template.length); // 13
console.log(template[0]);     // "H"

// Step 3: Use string methods (they return new strings).
console.log(template.toUpperCase()); // "HELLO, WORLD!"
console.log(template.includes("World")); // true

// Step 4: Demonstrate immutability.
const original = "abc";
const upper = original.toUpperCase();
console.log(original); // "abc" (unchanged)
console.log(upper);    // "ABC"
```

**Expected Output:**
```
Hello, World!
13
H
HELLO, WORLD!
true
abc
ABC
```

**Why This Output:** Template literals interpolate the values. `length` counts UTF-16 code units. String methods return new strings; the original remains unchanged because strings are immutable.

#### Example 2: Template Literals and Multi-line Strings

```js
// Step 1: Multi-line string with template literal.
const message = `Dear user,
Your order has shipped.
Thank you.`;

console.log(message);
// Output:
// Dear user,
// Your order has shipped.
// Thank you.

// Step 2: Interpolation with expressions.
const price = 19.99;
const quantity = 3;
console.log(`Total: $${(price * quantity).toFixed(2)}`);
// "Total: $59.97"
```

**Expected Output:**
```
Dear user,
Your order has shipped.
Thank you.
Total: $59.97
```

**Why This Output:** Backticks preserve line breaks. `${}` evaluates expressions and inserts their string representations.

### Real-World Cases

- **Form data:** Storing user names, email addresses, and messages.
- **API communication:** JSON keys and values are strings; URLs and query parameters are strings.
- **Template rendering:** Building HTML or text templates with interpolation.
- **Internationalisation:** Strings with Unicode and locale-specific formatting.

---

## Number

### Definitions

**Core Definition:** The Number type represents both integer and floating-point values using the IEEE 754 64-bit double-precision floating-point format.

**Technical Definition:** A Number value is a double-precision 64-bit binary format IEEE 754 value, similar to `double` in Java or C#. It can safely represent integers between \(-(2^{53} - 1)\) and \(2^{53} - 1\) without loss of precision, and floating-point values up to approximately \(1.79 \times 10^{308}\). JavaScript does not distinguish between integer and floating-point numbers at the type level.

**Beginner-Friendly Explanation:** The Number type handles all ordinary numbers — whole numbers like `42`, decimals like `3.14`, negative numbers, and special values like `NaN` and `Infinity`. It is a floating-point type, so some decimal arithmetic may be slightly imprecise (e.g., `0.1 + 0.2`).

### Purposes

- To perform arithmetic and mathematical calculations.
- To represent quantities, measurements, and scores.
- To handle numeric input and output.
- To serve as the default numeric type for most applications.

### Syntax Rules and Structure

**Number Literals:**
```js
42          // integer
3.14        // floating-point
0xFF        // hexadecimal (255)
0b1010      // binary (10)
0o755       // octal (493)
1e6         // scientific notation (1,000,000)
NaN         // "Not-a-Number"
Infinity    // positive infinity
```

**Key Constants:**
- `Number.MAX_SAFE_INTEGER` (2⁵³ − 1 = 9,007,199,254,740,991)
- `Number.MIN_SAFE_INTEGER` (−(2⁵³ − 1))
- `Number.MAX_VALUE`, `Number.MIN_VALUE`
- `Number.EPSILON` (smallest difference between 1 and the next representable number)

**Syntax Rules:**
- Integers outside the safe range lose precision.
- `0.1 + 0.2 !== 0.3` due to IEEE 754 floating-point representation.
- `NaN` is not equal to anything, including itself (`NaN === NaN` is `false`); use `Number.isNaN()`.
- `-0` exists and is distinct from `0` in some operations.

**Constraints and Limitations:**
- Precision loss for integers beyond \(2^{53} - 1\) (use BigInt for exact large integers).
- Floating-point arithmetic is not exact for many decimal fractions.
- No distinction between integer and float at the type level.

### Annotated Code Examples

#### Example 1: Safe Integer Boundary

```js
// Step 1: Largest safe integer.
console.log(Number.MAX_SAFE_INTEGER); // 9007199254740991

// Step 2: Exceeding the safe range loses precision.
console.log(9007199254740992 === 9007199254740993); // true (both are the same value)

// Step 3: Check safety.
console.log(Number.isSafeInteger(9007199254740991)); // true
console.log(Number.isSafeInteger(9007199254740992)); // false
```

**Expected Output:**
```
9007199254740991
true
true
false
```

**Why This Output:** Beyond \(2^{53} - 1\), integers cannot be represented exactly; two distinct mathematical values become indistinguishable.

#### Example 2: Floating-Point Precision

```js
// Step 1: Classic floating-point issue.
console.log(0.1 + 0.2); // 0.30000000000000004
console.log(0.1 + 0.2 === 0.3); // false

// Step 2: Use EPSILON for approximate comparison.
const isApproximatelyEqual = Math.abs((0.1 + 0.2) - 0.3) < Number.EPSILON;
console.log(isApproximatelyEqual); // true
```

**Expected Output:**
```
0.30000000000000004
false
true
```

**Why This Output:** IEEE 754 cannot represent `0.1` and `0.2` exactly, so their sum is slightly off. `Number.EPSILON` provides a tolerance for comparison.

### Real-World Cases

- **Financial calculations:** Currency and pricing (often using integer cents to avoid floating-point issues).
- **Graphics and games:** Coordinates, physics, and animations.
- **Data analysis:** Statistical computations and measurements.
- **Timers and scheduling:** `Date.now()` returns a Number (milliseconds since epoch).

---

## BigInt

### Definitions

**Core Definition:** BigInt is a numeric primitive type that represents arbitrarily large integers, beyond the safe integer range of the Number type.

**Technical Definition:** A BigInt value is a `bigint` primitive created by appending `n` to an integer literal or by calling the `BigInt()` function without `new`. BigInts are exact integers with no upper or lower bound (subject to memory), but they cannot represent fractional values. Mixing BigInt and Number in arithmetic operations throws a `TypeError`.

**Beginner-Friendly Explanation:** BigInt is for numbers too large for ordinary Number — like huge IDs from databases, precise timestamps in nanoseconds, or cryptographic values. You write it with an `n` at the end, like `9007199254740993n`.

### Purposes

- To represent integers beyond \(2^{53} - 1\) exactly.
- To perform exact large-integer arithmetic.
- To handle database IDs, cryptographic values, and high-precision timestamps.
- To avoid precision loss in financial or scientific calculations requiring large integers.

### Syntax Rules and Structure

**BigInt Literals:**
```js
42n
9007199254740993n
0x1FFFFFFFFFFFFFn
```

**Constructor:**
```js
BigInt(42)        // 42n
BigInt("9007199254740993") // 9007199254740993n
```

**Syntax Rules:**
- BigInt literals end with `n`.
- `BigInt()` is the only case where a number can be converted to BigInt without throwing (and only integers are allowed).
- Arithmetic between BigInt and Number throws a `TypeError`; both operands must be BigInt.
- BigInt division truncates toward zero (no fractional results).
- BigInt cannot be used with `Math` methods; use BigInt-specific operations.

**Constraints and Limitations:**
- Not JSON-serializable by default (`JSON.stringify` throws on BigInt).
- Cannot represent fractional values.
- Cannot be mixed with Number in arithmetic without explicit conversion.
- `typeof` returns `"bigint"`.

### Annotated Code Examples

#### Example 1: Creating and Operating on BigInts

```js
// Step 1: Create BigInt values.
const big1 = 9007199254740993n;
const big2 = BigInt("9007199254740995");

// Step 2: Arithmetic with BigInts.
console.log(big1 + big2); // 18014398509481988n
console.log(big2 - big1); // 2n

// Step 3: Division truncates.
console.log(10n / 3n); // 3n

// Step 4: Mixing Number and BigInt throws.
try {
  console.log(10n + 5);
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
18014398509481988n
2n
3n
TypeError
```

**Why This Output:** BigInt arithmetic is exact and unbounded. Division truncates. Mixing types without conversion is forbidden.

#### Example 2: BigInt with `typeof` and Comparison

```js
// Step 1: typeof BigInt.
console.log(typeof 42n); // "bigint"

// Step 2: Comparison across types works.
console.log(42n === 42);  // false (strict equality checks type)
console.log(42n == 42);   // true (loose equality coerces)

// Step 3: Relational comparison.
console.log(42n > 41);    // true
```

**Expected Output:**
```
bigint
false
true
true
```

**Why This Output:** `typeof` returns `"bigint"`. Strict equality requires matching types; loose equality coerces. Relational comparison allows mixed types.

### Real-World Cases

- **Database IDs:** Twitter/X snowflake IDs exceed `Number.MAX_SAFE_INTEGER`.
- **Cryptography:** Large prime numbers and modular arithmetic.
- **High-precision timestamps:** Nanosecond timestamps.
- **Financial systems:** Exact large-integer calculations.

---

## Boolean

### Definitions

**Core Definition:** The Boolean type represents a logical entity having two values: `true` and `false`.

**Technical Definition:** Boolean values are typically produced by relational operators, equality operators, and the logical NOT (`!`) operator. The `Boolean` wrapper object can convert any value to a boolean: falsy values (`false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`) become `false`; all other values become `true`.

**Beginner-Friendly Explanation:** Booleans are simple true/false values — like a light switch. They are used in conditions (`if`), loops, and logical expressions.

### Purposes

- To represent truth values in conditional logic.
- To control program flow (`if`, `while`, `for`).
- To store flags and state (e.g., `isLoading`, `hasError`).
- To serve as the result type for comparisons and logical operations.

### Syntax Rules and Structure

**Boolean Literals:**
```js
true
false
```

**Conversion:**
```js
Boolean(value)   // explicit conversion
!!value          // double negation (idiomatic)
```

**Falsy Values (convert to `false`):** `false`, `0`, `-0`, `0n`, `""` (empty string), `null`, `undefined`, `NaN`.

**Truthy Values (convert to `true`):** All other values, including `"0"`, `"false"`, `[]`, `{}`, and functions.

**Syntax Rules:**
- `Boolean` is a primitive type, not an object.
- `new Boolean(false)` creates a wrapper object that is truthy — avoid this pattern.
- Logical operators (`&&`, `||`, `!`) work with booleans and other types via truthiness.

**Constraints and Limitations:**
- The wrapper object `new Boolean(false)` is truthy, which is a common source of bugs.
- Implicit coercion in conditions can lead to unexpected results for empty arrays and objects (both truthy).

### Annotated Code Examples

#### Example 1: Falsy and Truthy Values

```js
// Step 1: Falsy values.
console.log(Boolean(false));    // false
console.log(Boolean(0));        // false
console.log(Boolean(""));       // false
console.log(Boolean(null));     // false
console.log(Boolean(undefined));// false
console.log(Boolean(NaN));      // false

// Step 2: Truthy values.
console.log(Boolean("0"));      // true
console.log(Boolean([]));       // true
console.log(Boolean({}));       // true
console.log(Boolean(-1));       // true
```

**Expected Output:**
```
false
false
false
false
false
false
true
true
true
true
```

**Why This Output:** The falsy set is fixed; everything else (including empty arrays and objects) is truthy.

#### Example 2: Conditional Logic with Booleans

```js
// Step 1: Use a boolean flag.
let isLoading = true;

if (isLoading) {
  console.log("Loading..."); // "Loading..."
}

// Step 2: Toggle the flag.
isLoading = !isLoading;
console.log(isLoading); // false

// Step 3: Comparison produces a boolean.
const age = 25;
const isAdult = age >= 18;
console.log(isAdult); // true
console.log(typeof isAdult); // "boolean"
```

**Expected Output:**
```
Loading...
false
true
boolean
```

**Why This Output:** `if` evaluates truthiness; `!` toggles the boolean; comparisons return booleans.

### Real-World Cases

- **Form validation:** `isValid`, `hasErrors`.
- **UI state:** `isLoading`, `isDarkMode`.
- **Feature flags:** `isFeatureEnabled`.
- **Access control:** `canEdit`, `hasPermission`.

---

## `undefined`

### Definitions

**Core Definition:** `undefined` is a primitive value automatically assigned to variables that have been declared but not initialised, to missing function return values, and to non-existent object properties.

**Technical Definition:** The `undefined` global property represents the primitive value `undefined`. It is one of JavaScript's primitive types. In modern environments (ECMAScript 5+), `undefined` is a non-configurable, non-writable property of the global object. It is the value used when a variable or property does not currently have another value.

**Beginner-Friendly Explanation:** `undefined` means "no value has been assigned yet." If you declare `let x;` and don't give it a value, `x` is `undefined`. If a function doesn't return anything, it returns `undefined`.

### Purposes

- To represent the absence of an assigned value.
- To serve as the default return value for functions without a `return` statement.
- To indicate missing object properties.
- To signal that a variable exists but has not been initialised.

### Syntax Rules and Structure

```js
let x;
console.log(x); // undefined

function noReturn() {}
console.log(noReturn()); // undefined

const obj = {};
console.log(obj.missing); // undefined
```

**Syntax Rules:**
- `undefined` is a primitive value and a global property (not a keyword).
- `typeof undefined` returns `"undefined"`.
- `undefined` is falsy.
- In strict mode, assigning to an undeclared variable throws; `undefined` can be shadowed in non-global scopes (avoid this).

**Constraints and Limitations:**
- `undefined` can be shadowed by a local variable named `undefined` (though this is bad practice).
- `undefined` is not a reserved word, but it should be treated as a constant.
- `void 0` is a safe alternative to obtain `undefined` (useful in old code).

### Annotated Code Examples

#### Example 1: Sources of `undefined`

```js
// Step 1: Uninitialised variable.
let uninitialised;
console.log(uninitialised); // undefined

// Step 2: Missing property.
const user = { name: "Alice" };
console.log(user.age); // undefined

// Step 3: Function without return.
function greet() {
  console.log("Hello");
}
const result = greet();
console.log(result); // undefined

// Step 4: Missing function argument.
function add(a, b) {
  return a + b;
}
console.log(add(1)); // NaN (undefined + 1 = NaN)
```

**Expected Output:**
```
undefined
undefined
Hello
undefined
NaN
```

**Why This Output:** Declared but uninitialised variables, missing properties, and missing return values all yield `undefined`. Arithmetic with `undefined` produces `NaN`.

#### Example 2: Checking for `undefined`

```js
// Step 1: Use typeof for safe checks (works even if variable is undeclared).
if (typeof someVar === "undefined") {
  console.log("someVar is undefined or undeclared");
}

// Step 2: Strict equality check.
let value = undefined;
console.log(value === undefined); // true
```

**Expected Output:**
```
someVar is undefined or undeclared
true
```

**Why This Output:** `typeof` does not throw for undeclared variables; strict equality checks the value.

### Real-World Cases

- **Optional parameters:** Detecting whether a function argument was provided.
- **Configuration objects:** Checking whether an option was set.
- **DOM queries:** `document.getElementById()` returns `null`, not `undefined`, for missing elements.
- **API responses:** Missing fields may be `undefined`.

---

## `null`

### Definitions

**Core Definition:** `null` is a primitive value that represents the intentional absence of any object value.

**Technical Definition:** The `null` value is a primitive value that represents the intentional absence of any object value. Unlike `undefined`, `null` is a literal and is not a property of the global object. It is often used in APIs where an object is expected but no suitable object exists.

**Beginner-Friendly Explanation:** `null` means "deliberately empty." It is used when you want to explicitly say "there is no value here" — for example, when a search returns no result.

### Purposes

- To explicitly indicate "no object" or "no value."
- To reset a variable or property to an empty state.
- To serve as a return value when an expected object cannot be found.
- To distinguish intentional emptiness from uninitialised state (`undefined`).

### Syntax Rules and Structure

```js
const empty = null;
console.log(empty); // null
```

**Syntax Rules:**
- `null` is a literal, not a global property (unlike `undefined`).
- `typeof null` returns `"object"` (a historical bug).
- `null` is falsy.
- `null` is not an identifier for a global property; it cannot be shadowed.

**Constraints and Limitations:**
- `typeof null` is `"object"`, so `typeof` is unreliable for detecting `null`; use `=== null`.
- `null` and `undefined` are loosely equal (`null == undefined` is `true`) but strictly unequal (`null === undefined` is `false`).

### Annotated Code Examples

#### Example 1: Intentional Absence

```js
// Step 1: Function returns null when no result is found.
function findUser(id) {
  if (id === 1) {
    return { id: 1, name: "Alice" };
  }
  return null; // deliberately no user
}

console.log(findUser(1)); // { id: 1, name: 'Alice' }
console.log(findUser(2)); // null

// Step 2: Check for null explicitly.
const result = findUser(2);
if (result === null) {
  console.log("No user found");
}
```

**Expected Output:**
```
{ id: 1, name: 'Alice' }
null
No user found
```

**Why This Output:** `null` signals the intentional absence of an object. Strict equality distinguishes it from other falsy values.

#### Example 2: `typeof null` Quirk

```js
// Step 1: typeof null returns "object" (historical bug).
console.log(typeof null); // "object"

// Step 2: Use strict equality for reliable detection.
console.log(null === null); // true
console.log(null === undefined); // false
```

**Expected Output:**
```
object
true
false
```

**Why This Output:** The `typeof` result for `null` is a legacy artefact; `=== null` is the correct check.

### Real-World Cases

- **DOM APIs:** `document.querySelector()` returns `null` when no element matches.
- **JSON:** `null` is a valid JSON value representing emptiness.
- **Database records:** A missing optional field may be `null`.
- **State management:** Resetting a piece of state to "empty."

---

## Symbol

### Definitions

**Core Definition:** Symbol is a primitive type whose values are guaranteed to be unique and immutable, commonly used as unique property keys that cannot collide with other keys.

**Technical Definition:** A Symbol is a built-in object whose constructor returns a `symbol` primitive — also called a Symbol value — that is guaranteed to be unique. Every call to `Symbol()` returns a unique symbol; `Symbol("foo") !== Symbol("foo")`. Symbols are often used to add unique property keys to objects that are hidden from normal property-access mechanisms, providing a weak form of encapsulation.

**Beginner-Friendly Explanation:** A Symbol is a completely unique identifier — like a serial number that no other value can duplicate. Symbols are used to create "hidden" or "special" property names that won't clash with anything else.

### Purposes

- To create unique property keys that cannot collide with other keys.
- To define well-known symbols (`Symbol.iterator`, `Symbol.toPrimitive`) that customise language behaviour.
- To implement a weak form of information hiding/encapsulation.
- To avoid name clashes in libraries and mixins.

### Syntax Rules and Structure

**Creating Symbols:**
```js
const sym1 = Symbol();
const sym2 = Symbol("description");
const shared = Symbol.for("key"); // global registry
```

**Key Points:**
- `Symbol()` returns a new unique primitive each call.
- The optional string is a description for debugging, not an identifier.
- `Symbol.for("key")` uses the global symbol registry; repeated calls with the same key return the same symbol.
- `new Symbol()` throws a `TypeError` to prevent wrapper object creation.

**Well-Known Symbols:**
- `Symbol.iterator` — defines the default iterator.
- `Symbol.toPrimitive` — customises primitive conversion.
- `Symbol.toStringTag` — customises `Object.prototype.toString()` output.

**Constraints and Limitations:**
- Symbols are not JSON-serializable (they are omitted).
- Symbols are not enumerable in `for...in`, `Object.keys()`, or `JSON.stringify()`.
- `typeof` returns `"symbol"`.
- Symbols cannot be converted to strings implicitly (e.g., `"" + sym` throws a `TypeError`).

### Annotated Code Examples

#### Example 1: Unique Property Keys

```js
// Step 1: Create two symbols with the same description.
const key1 = Symbol("id");
const key2 = Symbol("id");

// Step 2: They are different.
console.log(key1 === key2); // false

// Step 3: Use symbols as object keys.
const user = {
  name: "Alice",
  [key1]: 123,
};

console.log(user[key1]); // 123
console.log(user[key2]); // undefined
console.log(Object.keys(user)); // ["name"] (symbol key is hidden)
```

**Expected Output:**
```
false
123
undefined
["name"]
```

**Why This Output:** Each `Symbol()` call creates a unique value. Symbol-keyed properties are not included in `Object.keys()`.

#### Example 2: Well-Known Symbol — `Symbol.iterator`

```js
// Step 1: Define an iterable object.
const range = {
  from: 1,
  to: 3,
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;
    return {
      next() {
        return current <= last
          ? { value: current++, done: false }
          : { value: undefined, done: true };
      },
    };
  },
};

// Step 2: Use for...of (which uses Symbol.iterator).
for (const num of range) {
  console.log(num);
}
```

**Expected Output:**
```
1
2
3
```

**Why This Output:** `for...of` looks up `Symbol.iterator` on the object; the custom iterator yields 1, 2, 3.

### Real-World Cases

- **Library internals:** Hidden metadata keys.
- **Iteration protocols:** Custom iterables (`Symbol.iterator`).
- **Type coercion:** Custom `Symbol.toPrimitive` behaviour.
- **Framework internals:** React uses symbols for internal markers.

---

## Primitive Immutability

### Definitions

**Core Definition:** Primitive immutability means that primitive values cannot be changed once created; operations that appear to modify a primitive actually create a new value.

**Technical Definition:** All primitive values are immutable — they cannot be altered. A variable holding a primitive can be reassigned to a new value, but the existing value itself cannot be changed. There are no utilities to mutate primitive values. Objects, arrays, and functions are mutable; primitives are not.

**Beginner-Friendly Explanation:** If you have a string `"hello"` and call `toUpperCase()`, you get `"HELLO"` — but the original `"hello"` is still unchanged. You can't edit a primitive in place; you can only create a new one.

### Purposes

- To ensure predictable values that cannot be accidentally mutated.
- To enable safe sharing of values across functions and scopes.
- To simplify reasoning about program state (primitives are value types).
- To support efficient implementation (primitives can be stored directly).

### Syntax Rules and Structure

Immutability is a property of the type, not a syntax construct. Attempting to mutate a primitive fails silently (or throws in strict mode for some operations):

```js
const str = "abc";
str[0] = "x"; // silently ignored (non-strict) or TypeError (strict, for some cases)
console.log(str); // "abc"
```

### Annotated Code Examples

#### Example 1: String Immutability

```js
// Step 1: Create a string.
let str = "hello";

// Step 2: Attempt to "mutate" it.
str.toUpperCase(); // returns "HELLO" but doesn't change str
console.log(str); // "hello"

// Step 3: Reassign the variable to a new string.
str = str.toUpperCase();
console.log(str); // "HELLO"
```

**Expected Output:**
```
hello
HELLO
```

**Why This Output:** `toUpperCase()` returns a new string. The original binding still points to `"hello"` until reassigned.

#### Example 2: Number Immutability

```js
let num = 10;
num + 5; // returns 15, but num unchanged
console.log(num); // 10

num = num + 5;
console.log(num); // 15
```

**Expected Output:**
```
10
15
```

**Why This Output:** Arithmetic creates new numbers; the original binding is unchanged until assignment.

### Real-World Cases

- **React state:** Immutable updates (`setState` with new objects/values) are central to predictable rendering.
- **Redux:** State is immutable; reducers return new state.
- **Functional programming:** Pure functions rely on immutable data.
- **Caching and memoisation:** Immutable values can be safely cached without defensive copying.

---

## Primitive Wrappers

### Definitions

**Core Definition:** Primitive wrapper objects are object types (`String`, `Number`, `Boolean`, `Symbol`, `BigInt`) that correspond to primitive types and provide methods for working with primitive values.

**Technical Definition:** With the exception of `null` and `undefined`, every primitive has a corresponding wrapper object: `String` for string primitives, `Number` for number primitives, `Boolean` for boolean primitives, and `Symbol` for symbol primitives. When a property is accessed on a primitive, JavaScript auto-boxes the value into a wrapper object and accesses the property on that object instead. The wrapper's `valueOf()` method returns the primitive value.

**Beginner-Friendly Explanation:** Primitives like `"hello"` don't have methods, but you can still call `"hello".toUpperCase()`. JavaScript temporarily wraps the primitive in a `String` object, calls the method, and then discards the wrapper. This is called auto-boxing.

### Purposes

- To provide methods and properties for working with primitive values (`toFixed()`, `includes()`, `charAt()`).
- To enable conversion between primitives and objects.
- To support explicit object creation when needed (though rarely recommended).
- To explain the behaviour of property access on primitives.

### Syntax Rules and Structure

**Wrapper Constructors:**
```js
new String("hello")  // String object (wrapper)
new Number(42)       // Number object (wrapper)
new Boolean(true)    // Boolean object (wrapper)
Object(Symbol())     // Symbol wrapper (new Symbol() throws)
Object(10n)          // BigInt wrapper (new BigInt() throws)
```

**Auto-boxing:**
```js
"hello".toUpperCase(); // auto-boxes to String object, calls method, discards object
```

**Syntax Rules:**
- `new String()`, `new Number()`, and `new Boolean()` create wrapper objects (which are truthy regardless of value).
- `Symbol()` and `BigInt()` cannot be called with `new`; use `Object()` for wrapper objects.
- Wrapper objects are objects (`typeof` returns `"object"`), not primitives.
- `valueOf()` returns the underlying primitive.

**Constraints and Limitations:**
- Wrapper objects are generally discouraged; they introduce type confusion (`typeof new String("x")` is `"object"`).
- `new Boolean(false)` is truthy — a classic source of bugs.
- Auto-boxing creates a new wrapper each time; repeated property access can be slightly slower than object property access.

### Annotated Code Examples

#### Example 1: Auto-boxing in Action

```js
// Step 1: Access a method on a string primitive.
const primitive = "hello";
console.log(primitive.toUpperCase()); // "HELLO"

// Step 2: The primitive itself is unchanged.
console.log(primitive); // "hello"

// Step 3: Demonstrate that the wrapper is temporary.
primitive.customProperty = "test";
console.log(primitive.customProperty); // undefined (wrapper discarded)
```

**Expected Output:**
```
HELLO
hello
undefined
```

**Why This Output:** The string primitive is auto-boxed to a `String` object for the method call. The temporary wrapper is discarded, so custom properties do not persist.

#### Example 2: Explicit Wrapper Objects

```js
// Step 1: Create wrapper objects.
const strObj = new String("hello");
const numObj = new Number(42);

// Step 2: typeof shows "object".
console.log(typeof strObj); // "object"
console.log(typeof numObj); // "object"

// Step 3: valueOf() returns the primitive.
console.log(strObj.valueOf()); // "hello"
console.log(numObj.valueOf()); // 42

// Step 4: Wrapper objects are truthy.
console.log(Boolean(new Boolean(false))); // true (gotcha!)
```

**Expected Output:**
```
object
object
hello
42
true
```

**Why This Output:** Wrapper objects are real objects; `typeof` reports `"object"`. `Boolean(new Boolean(false))` is `true` because the object exists (objects are truthy).

### Real-World Cases

- **String methods:** `slice()`, `replace()`, `split()`.
- **Number formatting:** `toFixed()`, `toPrecision()`.
- **Boolean conversion:** `Boolean()` function (without `new`).
- **Understanding legacy code:** Wrapper objects appear in old code and interview questions.

---

## `typeof` Operator

### Definitions

**Core Definition:** The `typeof` operator returns a string indicating the type of its operand's value.

**Technical Definition:** The `typeof` operator returns a string indicating the type of the operand's value. It works with undeclared identifiers, returning `"undefined"` instead of throwing an error. The possible return values are `"undefined"`, `"object"` (including `null`), `"boolean"`, `"number"`, `"bigint"`, `"string"`, `"symbol"`, `"function"`, and `"object"` for any other object.

**Beginner-Friendly Explanation:** `typeof` tells you what kind of value you have — `"string"`, `"number"`, `"boolean"`, and so on. It's useful for checking types before using a value.

### Purposes

- To determine the type of a value at runtime.
- To perform safe type checks without throwing errors (especially for undeclared variables).
- To implement type guards and feature detection.
- To debug and inspect values.

### Syntax Rules and Structure

```js
typeof operand;
typeof (expression);
```

**Return Values:**

| Type | Result |
|---|---|
| Undefined | `"undefined"` |
| Null | `"object"` (historical bug) |
| Boolean | `"boolean"` |
| Number | `"number"` |
| BigInt | `"bigint"` |
| String | `"string"` |
| Symbol | `"symbol"` |
| Function | `"function"` |
| Any other object | `"object"` |

**Syntax Rules:**
- `typeof` is a unary operator; parentheses are optional.
- `typeof` has higher precedence than binary operators like `+`.
- `typeof undeclaredVariable` returns `"undefined"` without throwing.

**Constraints and Limitations:**
- `typeof null` returns `"object"`; use `=== null` for null checks.
- `typeof` cannot distinguish between different object types (arrays, dates, regexes all return `"object"`); use `Array.isArray()` or `instanceof`.
- `typeof` on a `let`/`const` variable in the TDZ throws a `ReferenceError` (unlike undeclared variables).

### Annotated Code Examples

#### Example 1: `typeof` Across Primitive Types

```js
// Step 1: Check each primitive type.
console.log(typeof "hello");    // "string"
console.log(typeof 42);         // "number"
console.log(typeof 42n);        // "bigint"
console.log(typeof true);       // "boolean"
console.log(typeof undefined);  // "undefined"
console.log(typeof Symbol());   // "symbol"

// Step 2: The null quirk.
console.log(typeof null);       // "object"

// Step 3: Functions and objects.
console.log(typeof function(){}); // "function"
console.log(typeof []);           // "object"
console.log(typeof {});           // "object"
```

**Expected Output:**
```
string
number
bigint
boolean
undefined
symbol
object
function
object
object
```

**Why This Output:** Each primitive has a distinct `typeof` result except `null`, which returns `"object"` due to a historical artefact. Functions are callable objects and return `"function"`.

#### Example 2: Safe Type Checking

```js
// Step 1: Safe check for an undeclared variable.
if (typeof maybeUndefined === "undefined") {
  console.log("Variable is undefined or undeclared");
}

// Step 2: Check for a function before calling.
const fn = 42;
if (typeof fn === "function") {
  fn();
} else {
  console.log("Not a function");
}
```

**Expected Output:**
```
Variable is undefined or undeclared
Not a function
```

**Why This Output:** `typeof` on an undeclared variable does not throw. Type checks prevent calling non-functions.

### Real-World Cases

- **Feature detection:** `typeof window !== "undefined"` to check for browser environment.
- **Polyfills:** Checking whether a method exists before using it.
- **Type guards in TypeScript:** `typeof` is a built-in type guard.
- **Debugging:** Inspecting values during development.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – ECMAScript Data Types and Values, §6 — https://tc39.es/ecma262/
- MDN Web Docs – JavaScript data types and data structures — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Data_structures
- MDN Web Docs – Primitive — https://developer.mozilla.org/en-US/docs/Glossary/Primitive
- MDN Web Docs – String — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String
- MDN Web Docs – Number — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number
- MDN Web Docs – BigInt — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt
- MDN Web Docs – Boolean — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean
- MDN Web Docs – `undefined` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/undefined
- MDN Web Docs – `null` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/null
- MDN Web Docs – Symbol — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol
- MDN Web Docs – `typeof` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof
- MDN Web Docs – JavaScript language overview — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Language_overview
- MDN Web Docs – Immutable — https://developer.mozilla.org/en-US/docs/Glossary/Immutable