# JavaScript Comparison Operators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Comparison operators are symbols and expressions in JavaScript used to compare two operands and return a Boolean value (`true` or `false`) based on whether the comparison holds. They encompass equality operators (`==`, `===`, `!=`, `!==`) and relational operators (`>`, `<`, `>=`, `<=`).

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §7.2.11–7.2.13), comparison operations are abstract operations. The `IsLooselyEqual` operation (abstract equality, §7.2.13) performs type coercion before comparison when operand types differ. The `IsStrictlyEqual` operation (strict equality, §7.2.12) returns `false` immediately if types differ and requires identical Number, String, Boolean, Symbol, or Object values. The `IsLessThan` operation (abstract relational comparison, §7.2.11) converts operands to Numeric via `ToPrimitive` and returns `undefined` if either operand is `NaN`. Additionally, three distinct equality algorithms exist: **Abstract Equality** (`==`), **Strict Equality** (`===`), and **SameValue** (used by `Object.is()`), with **SameValueZero** being a fourth variant used internally by `Map`/`Set` operations.

**Beginner-Friendly Explanation:** Comparison operators are how you ask JavaScript "are these two things the same?" or "is this bigger than that?" The answer is always `true` or `false`. The trickiest part is understanding that `==` tries to convert types before comparing, `===` does not, and there are subtle differences in how `-0`, `+0`, and `NaN` are treated.

### Key Characteristics

- **Boolean result:** All comparison operators return `true` or `false`.
- **Type coercion in `==`:** The abstract equality operator converts operand types before comparing.
- **No coercion in `===`:** Strict equality requires identical types.
- **Three value-comparison operations:** JavaScript provides `===` (strict equality), `==` (loose equality), and `Object.is()` (same-value equality).
- **SameValueZero:** A fourth algorithm used internally by `Map`, `Set`, and `Array.prototype.includes()`.
- **Relational operators:** `>`, `<`, `>=`, `<=` use abstract relational comparison, returning `undefined` for `NaN` operands (which becomes `false` in Boolean context).
- **Object comparison by reference:** Objects and arrays are compared by reference, not by structural content.

### Prerequisites

- Basic JavaScript syntax (variables, expressions, data types).
- Understanding of JavaScript data types (Number, String, Boolean, Symbol, BigInt, Object, null, undefined).
- Familiarity with type coercion and the `ToPrimitive` / `ToNumber` conversion operations.
- Awareness of `NaN`, `+0`, and `-0` as special numeric values.

### Related Programming Areas

- **Conditional logic:** `if` statements, loops, ternary expressions.
- **Data validation:** Checking equality, ranges, and types.
- **Sorting algorithms:** Relational operators drive comparison functions.
- **Testing:** Assertion libraries use equality algorithms (`toBe`, `toEqual`).
- **Data structures:** `Map`/`Set` key equality relies on SameValueZero.

### Core Concepts / Features

---

## `==` (Abstract Equality / Loose Equality)

### Definitions

**Core Definition:** The `==` operator compares two operands for equality after performing type coercion when their types differ.

**Technical Definition:** Per ECMA-262 §7.2.13, the `IsLooselyEqual(x, y)` abstract operation: (1) If `SameType(x, y)`, return `IsStrictlyEqual(x, y)`. (2) If `x` is `null` and `y` is `undefined` (or vice versa), return `true`. (3) If `x` is Number and `y` is String, convert the String to Number and recurse. (4) If `x` is Boolean, convert to Number and recurse. (5) If `x` is String, Number, BigInt, or Symbol and `y` is Object, convert `y` to Primitive and recurse. (6) Otherwise, return `false`.

**Beginner-Friendly Explanation:** `==` checks if two values are "equal" after JavaScript tries to make them the same type. For example, `"5" == 5` is `true` because JavaScript converts the string `"5"` to the number `5`.

### Purposes

- To compare values of potentially different types with automatic coercion.
- To check for `null` or `undefined` in a single comparison (`x == null` is `true` for both).
- To perform loose comparisons in legacy or convenience-oriented code.

### Syntax Rules and Structure

```js
operand1 == operand2
```

**Component Breakdown:**
- `operand1`, `operand2`: Any JavaScript expressions.
- Returns: A Boolean.

**Syntax Rules:**
- If types match, behaves identically to `===`.
- `null == undefined` is `true`; `null == 0` is `false`.
- `NaN == NaN` is `false`.
- Objects are converted via `ToPrimitive` (usually `valueOf` then `toString`).
- `[] == false` is `true` (array → empty string → 0; false → 0).

**Constraints and Limitations:**
- Type coercion can produce surprising results (`"" == false` is `true`).
- Cannot be used for structural object comparison.
- `Object.is()` and `===` avoid coercion pitfalls.

### Annotated Code Examples

#### Example 1: Basic Type Coercion

```js
// Step 1: Same type — behaves like ===
console.log(1 == 1);              // true
console.log("hello" == "hello");  // true

// Step 2: Different types — coercion
console.log("5" == 5);            // true (string → number)
console.log(0 == false);          // true (boolean → number)
console.log("" == false);         // true (both → 0)
console.log(null == undefined);   // true (special rule)
console.log(null == 0);           // false (null only equals undefined)
console.log(NaN == NaN);          // false (NaN is never equal to anything)
```

**Expected Output:**
```
true
true
true
true
true
true
false
false
```

**Why This Output:** `"5"` is converted to `5`; `false` to `0`; `""` to `0`; `null` and `undefined` are equal to each other but not to `0`. `NaN` is not equal to itself.

#### Example 2: Object to Primitive Coercion

```js
// Step 1: Array to primitive.
console.log([] == false);         // true ([] → "" → 0; false → 0)
console.log([1] == 1);            // true ([1] → "1" → 1)
console.log([1, 2] == "1,2");     // true ([1,2] → "1,2")

// Step 2: Object to primitive.
const obj = { valueOf: () => 42 };
console.log(obj == 42);           // true (obj → 42)
```

**Expected Output:**
```
true
true
true
true
```

**Why This Output:** Arrays and objects are converted to primitives via `ToPrimitive`, which calls `valueOf` and `toString`.

### Real-World Cases

- **Null/undefined check:** `if (value == null)` catches both `null` and `undefined`.
- **Form input comparison:** `if (input.value == 0)` when the input could be a string.
- **Legacy code:** Maintaining older codebases where `==` was idiomatic.

---

## `===` (Strict Equality)

### Definitions

**Core Definition:** The `===` operator compares two operands for equality without performing type coercion.

**Technical Definition:** Per ECMA-262 §7.2.12, the `IsStrictlyEqual(x, y)` abstract operation: (1) If `Type(x)` ≠ `Type(y)`, return `false`. (2) If `Type(x)` is Undefined or Null, return `true`. (3) If `Type(x)` is Number: `NaN` is not equal to anything; `+0` and `-0` are equal; otherwise, compare numeric values. (4) If `Type(x)` is String, compare UTF-16 code unit sequences. (5) If `Type(x)` is Boolean, compare values. (6) If `Type(x)` is Symbol, compare identity. (7) If `Type(x)` is Object, compare reference identity.

**Beginner-Friendly Explanation:** `===` checks if two values are exactly the same — same type and same value. No conversion happens. `"5" === 5` is `false` because one is a string and the other is a number.

### Purposes

- To compare values without type coercion surprises.
- To check for exact equality of primitives and object references.
- To serve as the default equality operator in modern JavaScript.

### Syntax Rules and Structure

```js
operand1 === operand2
```

**Syntax Rules:**
- Different types always yield `false`.
- `NaN === NaN` is `false`.
- `+0 === -0` is `true`.
- Objects and arrays compare by reference only.

**Constraints and Limitations:**
- Does not perform structural comparison of objects or arrays.
- `NaN` requires `Number.isNaN()` or `Object.is()` for detection.

### Annotated Code Examples

#### Example 1: Strict Equality Basics

```js
console.log(1 === 1);              // true
console.log("5" === 5);            // false (different types)
console.log(0 === false);          // false
console.log(null === undefined);   // false (different types)
console.log(NaN === NaN);          // false
console.log(+0 === -0);            // true
```

**Expected Output:**
```
true
false
false
false
false
true
```

**Why This Output:** Strict equality requires same type. `NaN` is never equal to itself. `+0` and `-0` are considered equal.

#### Example 2: Object Reference Comparison

```js
const a = { id: 1 };
const b = { id: 1 };
const c = a;

console.log(a === b);  // false (different references)
console.log(a === c);  // true (same reference)
console.log([] === []); // false (different array references)
```

**Expected Output:**
```
false
true
false
```

**Why This Output:** Objects and arrays are compared by reference, not by content.

### Real-World Cases

- **Default equality:** Use `===` unless there is a specific reason for coercion.
- **Object identity:** Checking if two variables point to the same object.
- **Linting:** ESLint's `eqeqeq` rule enforces `===` over `==`.

---

## `!=` (Abstract Inequality)

### Definitions

**Core Definition:** The `!=` operator returns `true` if its operands are not equal after type coercion.

**Technical Definition:** Per ECMA-262, `x != y` is evaluated as `!(x == y)`, using the `IsLooselyEqual` abstract operation and negating the result.

**Beginner-Friendly Explanation:** `!=` is the opposite of `==`. It returns `true` when `==` would return `false`.

### Purposes

- To check for inequality with type coercion.
- To complement `==` in legacy code.

### Syntax Rules and Structure

```js
operand1 != operand2
```

**Returns:** Boolean (negation of `==`).

### Annotated Code Examples

```js
console.log(1 != 2);          // true
console.log("5" != 5);        // false (== is true)
console.log(null != undefined); // false (== is true)
console.log(NaN != NaN);      // true (== is false)
```

**Expected Output:**
```
true
false
false
true
```

**Why This Output:** `!=` negates `==`. Since `NaN == NaN` is `false`, `NaN != NaN` is `true`.

### Real-World Cases

- **Guard clauses:** `if (value != null) { ... }` skips both `null` and `undefined`.
- **Legacy code:** Maintaining older codebases.

---

## `!==` (Strict Inequality)

### Definitions

**Core Definition:** The `!==` operator returns `true` if its operands are not equal in type or value, without coercion.

**Technical Definition:** Per ECMA-262, `x !== y` is evaluated as `!(x === y)`, using `IsStrictlyEqual` and negating.

**Beginner-Friendly Explanation:** `!==` is the opposite of `===`. It returns `true` when `===` would return `false`.

### Purposes

- To check for inequality without coercion.
- To complement `===` as the default inequality operator.

### Syntax Rules and Structure

```js
operand1 !== operand2
```

**Returns:** Boolean (negation of `===`).

### Annotated Code Examples

```js
console.log(1 !== 2);           // true
console.log("5" !== 5);         // true (different types)
console.log(null !== undefined); // true
console.log(NaN !== NaN);       // true
console.log(0 !== false);       // true
```

**Expected Output:**
```
true
true
true
true
true
```

**Why This Output:** `!==` returns `true` whenever `===` returns `false`.

### Real-World Cases

- **Type-safe inequality:** Preferred over `!=` in modern code.
- **ESLint `eqeqeq`:** Enforces `!==` over `!=`.

---

## `>` (Greater Than)

### Definitions

**Core Definition:** The `>` operator returns `true` if the left operand is greater than the right operand.

**Technical Definition:** Per ECMA-262 §7.2.11, the `IsLessThan(x, y, true)` operation is used to evaluate `x > y`. Operands are converted to Numeric via `ToPrimitive` (hint Number). If either is `NaN`, the result is `undefined` (which evaluates to `false` in Boolean context). Strings are compared lexicographically by UTF-16 code units.

**Beginner-Friendly Explanation:** `>` checks if the left value is bigger than the right. It works for numbers and strings (alphabetical order).

### Purposes

- To compare numeric magnitudes.
- To compare strings alphabetically.
- To implement sorting and range checks.

### Syntax Rules and Structure

```js
operand1 > operand2
```

**Syntax Rules:**
- Both operands converted to Numeric via `ToPrimitive`.
- Strings compared lexicographically by UTF-16 code units.
- `NaN` comparisons yield `undefined` (falsy).
- BigInt and Number can be compared.

### Annotated Code Examples

```js
console.log(5 > 3);            // true
console.log("b" > "a");        // true (lexicographic)
console.log("10" > "9");       // false (string comparison: "1" < "9")
console.log(NaN > 5);          // false (undefined → falsy)
console.log(5n > 3);           // true (BigInt vs Number)
```

**Expected Output:**
```
true
true
false
false
true
```

**Why This Output:** String comparison is lexicographic, not numeric. `"10" > "9"` compares character by character: `"1"` is less than `"9"`.

### Real-World Cases

- **Sorting:** `arr.sort((a, b) => a - b)` uses numeric comparison.
- **Range checks:** `if (score > 90)`.
- **Date comparison:** `if (date1 > date2)`.

---

## `<` (Less Than)

### Definitions

**Core Definition:** The `<` operator returns `true` if the left operand is less than the right operand.

**Technical Definition:** Per ECMA-262 §7.2.11, `IsLessThan(x, y, true)`. Operands are converted to Numeric via `ToPrimitive`. Strings compare lexicographically.

**Beginner-Friendly Explanation:** `<` checks if the left value is smaller than the right.

### Annotated Code Examples

```js
console.log(3 < 5);            // true
console.log("a" < "b");        // true
console.log("9" < "10");       // false (string comparison)
console.log(NaN < 5);          // false
```

**Expected Output:**
```
true
true
false
false
```

### Real-World Cases

- **Sorting:** Comparing items.
- **Range checks:** `if (age < 18)`.
- **Time comparison:** `if (now < deadline)`.

---

## `>=` (Greater Than or Equal)

### Definitions

**Core Definition:** The `>=` operator returns `true` if the left operand is greater than or equal to the right operand.

**Technical Definition:** Per ECMA-262 §7.2.11, `x >= y` is evaluated as `IsLessThan(y, x, false)` and negated, or equivalently `!(y > x)`. The left-first evaluation flag is adjusted to preserve left-to-right evaluation order.

**Beginner-Friendly Explanation:** `>=` checks if the left value is bigger than or equal to the right.

### Annotated Code Examples

```js
console.log(5 >= 5);           // true
console.log(5 >= 3);           // true
console.log(3 >= 5);           // false
console.log(NaN >= 5);         // false
```

**Expected Output:**
```
true
true
false
false
```

### Real-World Cases

- **Passing thresholds:** `if (score >= 60)`.
- **Inclusive range checks:** `if (value >= min && value <= max)`.

---

## `<=` (Less Than or Equal)

### Definitions

**Core Definition:** The `<=` operator returns `true` if the left operand is less than or equal to the right operand.

**Technical Definition:** Per ECMA-262 §7.2.11, `x <= y` is evaluated as `IsLessThan(x, y, false)` and negated, or equivalently `!(x > y)`.

**Beginner-Friendly Explanation:** `<=` checks if the left value is smaller than or equal to the right.

### Annotated Code Examples

```js
console.log(3 <= 5);           // true
console.log(5 <= 5);           // true
console.log(5 <= 3);           // false
console.log(NaN <= 5);         // false
```

**Expected Output:**
```
true
true
false
false
```

### Real-World Cases

- **Maximum limits:** `if (usage <= quota)`.
- **Inclusive ranges:** `if (age <= 65)`.

---

## Abstract Equality

### Definitions

**Core Definition:** Abstract equality is the algorithm used by the `==` operator, performing type coercion when operand types differ.

**Technical Definition:** Per ECMA-262 §7.2.13, `IsLooselyEqual(x, y)` follows a multi-step algorithm: same type → strict equality; `null`/`undefined` pair → `true`; Number/String → convert String to Number; Boolean → convert to Number; Object with primitive → convert Object to Primitive; otherwise → `false`.

**Beginner-Friendly Explanation:** Abstract equality is the "convert types then compare" algorithm that `==` uses.

### Purposes

- To understand why `==` produces surprising results.
- To document the coercion rules for developers.

### Annotated Code Examples

```js
// Coercion chain examples
console.log(0 == "");         // true ("" → 0)
console.log(0 == "0");        // true
console.log(false == "0");    // true (false → 0; "0" → 0)
console.log(null == false);   // false (null only equals undefined)
console.log([] == 0);         // true ([] → "" → 0)
```

**Expected Output:**
```
true
true
true
false
true
```

### Real-World Cases

- **Legacy code review:** Identifying coercion bugs.
- **Code migration:** Replacing `==` with `===` and explicit conversions.

---

## Strict Equality

### Definitions

**Core Definition:** Strict equality is the algorithm used by `===`, comparing type and value without coercion.

**Technical Definition:** Per ECMA-262 §7.2.12, `IsStrictlyEqual(x, y)`: type mismatch → `false`; `undefined`/`null` → `true` (for same type); Number: `NaN` → `false`, `+0`/`-0` → `true`; String → code unit comparison; Boolean → value comparison; Symbol → identity; Object → reference identity.

**Beginner-Friendly Explanation:** Strict equality means "same type and same value."

### Purposes

- To compare values reliably without coercion.
- To serve as the default comparison operator.

### Annotated Code Examples

```js
console.log(1 === 1);         // true
console.log(1 === "1");       // false
console.log(NaN === NaN);     // false
console.log(+0 === -0);       // true
console.log({} === {});       // false
```

**Expected Output:**
```
true
false
false
true
false
```

### Real-World Cases

- **All modern JavaScript:** Default equality operator.

---

## `Object.is()`

### Definitions

**Core Definition:** `Object.is()` is a static method that determines whether two values are the same value, using the SameValue algorithm.

**Technical Definition:** Per ECMA-262 §20.1.2.10, `Object.is(value1, value2)` returns `true` if: both `undefined`, both `null`, both `true`/`false`, same-length strings with same code units, same object reference, same BigInt value, same Symbol identity, or both Numbers that are `+0`/`+0`, `-0`/`-0`, `NaN`/`NaN`, or same nonzero non-NaN value. The only differences from `===` are: `Object.is(NaN, NaN)` is `true`, and `Object.is(0, -0)` is `false`.

**Beginner-Friendly Explanation:** `Object.is()` is like `===` but with two fixes: it correctly identifies `NaN` as equal to `NaN`, and it treats `+0` and `-0` as different.

### Purposes

- To check if two values are the same, including `NaN` and signed zero distinctions.
- To detect `-0` and `NaN` precisely.
- To serve as the comparison used by `Object.is` in testing frameworks.

### Syntax Rules and Structure

```js
Object.is(value1, value2)
```

**Syntax Rules:**
- Does not coerce types.
- `Object.is(NaN, NaN)` → `true`.
- `Object.is(0, -0)` → `false`.
- All other behavior matches `===`.

**Constraints and Limitations:**
- Not supported in very old environments (ES2015+).
- Not a constructor; cannot be used with `new`.

### Annotated Code Examples

#### Example 1: Object.is vs ===

```js
// Same behavior as ===
console.log(Object.is(25, 25));        // true
console.log(Object.is("foo", "foo"));  // true
console.log(Object.is(null, null));    // true
console.log(Object.is([], []));        // false

// Differences from ===
console.log(Object.is(NaN, NaN));      // true (=== gives false)
console.log(Object.is(0, -0));         // false (=== gives true)
console.log(Object.is(-0, -0));        // true
```

**Expected Output:**
```
true
true
true
false
true
false
true
```

**Why This Output:** `Object.is` uses the SameValue algorithm, which treats `NaN` as equal to `NaN` and distinguishes `+0` from `-0`.

#### Example 2: Detecting -0

```js
function isNegativeZero(value) {
  return Object.is(value, -0);
}

console.log(isNegativeZero(-0));       // true
console.log(isNegativeZero(0));        // false
console.log(isNegativeZero(-1 * 0));   // true
console.log(isNegativeZero(0 * -1));   // true
```

**Expected Output:**
```
true
false
true
true
```

**Why This Output:** `Object.is` is the only reliable way to distinguish `-0` from `+0`.

### Real-World Cases

- **Testing frameworks:** Vitest's `toBe` uses `Object.is` for exact comparisons.
- **Precise numeric checks:** Detecting `-0` in scientific or financial code.
- **React dependency arrays:** Uses `Object.is` for comparing props.

---

## SameValueZero

### Definitions

**Core Definition:** SameValueZero is an internal equality algorithm used by `Map`, `Set`, `Array.prototype.includes()`, and `TypedArray` constructors. It is identical to SameValue except that `+0` and `-0` are considered equal.

**Technical Definition:** Per ECMA-262 §7.2.10, `SameValueZero(x, y)`: type mismatch → `false`; `undefined`/`null` → `true`; Number: `NaN` → `true`, `+0`/`-0` → `true` (unlike SameValue), otherwise numeric comparison; String/Boolean/Symbol → same as SameValue; Object → reference identity.

**Beginner-Friendly Explanation:** SameValueZero is like `Object.is()` but treats `+0` and `-0` as equal. It's what `Map`, `Set`, and `includes()` use internally to check if two keys or values are the same.

### Purposes

- To provide consistent equality semantics for collection operations.
- To ensure `NaN` can be used as a `Map` key or `Set` value.
- To treat `+0` and `-0` as the same key in collections.

### Syntax Rules and Structure

SameValueZero is not directly exposed as a function; it is used internally:

```js
// Map uses SameValueZero for keys
const map = new Map();
map.set(NaN, "value");
console.log(map.get(NaN)); // "value" (SameValueZero: NaN === NaN)

map.set(+0, "positive");
console.log(map.get(-0)); // "positive" (SameValueZero: +0 === -0)

// Set uses SameValueZero for values
const set = new Set([NaN, NaN, +0, -0]);
console.log(set.size); // 2 (NaN added once; +0/-0 treated as same)
```

**Expected Output:**
```
value
positive
2
```

**Why This Output:** `Map` and `Set` use SameValueZero, so `NaN` is a valid unique key, and `+0`/`-0` are treated as the same key.

### Real-World Cases

- **Map keys:** Using `NaN` as a key.
- **Set values:** Deduplicating `NaN`.
- **Array.includes:** `[NaN].includes(NaN)` returns `true` (SameValueZero).
- **IndexedDB:** Uses SameValueZero for key comparison.

---

## Comparison of Objects

### Definitions

**Core Definition:** Objects in JavaScript are compared by reference, not by structural content. Two objects are equal only if they reference the same memory location.

**Technical Definition:** In `IsStrictlyEqual`, when `Type(x)` is Object, step 7 returns `true` only if `x` and `y` refer to the same Object value. This applies to all object types: plain objects, arrays, functions, dates, regexes, and class instances.

**Beginner-Friendly Explanation:** When you compare two objects with `===` or `==`, JavaScript checks if they are literally the same object in memory — not whether they have the same properties.

### Purposes

- To check object identity (same reference).
- To understand why `{} === {}` is `false`.
- To know when structural comparison is needed (deep equality).

### Syntax Rules and Structure

```js
const a = { id: 1 };
const b = { id: 1 };
const c = a;

a === b;  // false (different references)
a === c;  // true (same reference)
```

**Constraints and Limitations:**
- No built-in structural comparison.
- `JSON.stringify()` comparison is unreliable (key order, circular references, `undefined` values).
- Deep equality requires custom functions or libraries (Lodash `isEqual`, Node.js `assert.deepStrictEqual`).

### Annotated Code Examples

#### Example 1: Reference vs Structural

```js
const obj1 = { name: "Alice" };
const obj2 = { name: "Alice" };
const obj3 = obj1;

console.log(obj1 === obj2);  // false (different references)
console.log(obj1 === obj3);  // true (same reference)
```

**Expected Output:**
```
false
true
```

**Why This Output:** `obj1` and `obj2` are distinct objects with identical content. `obj3` points to the same object as `obj1`.

#### Example 2: Shallow vs Deep Equality

```js
// Shallow: compares references.
const arr1 = [1, 2, 3];
const arr2 = [1, 2, 3];
console.log(arr1 === arr2); // false

// Structural comparison using length and every().
function arraysEqual(a, b) {
  return a.length === b.length && a.every((val, i) => val === b[i]);
}
console.log(arraysEqual(arr1, arr2)); // true
```

**Expected Output:**
```
false
true
```

**Why This Output:** `===` compares array references. A manual structural comparison checks each element.

### Real-World Cases

- **React state updates:** Knowing when a new object triggers re-render.
- **Caching:** Using object references as cache keys.
- **Testing:** `toEqual` for structural comparison vs `toBe` for identity.

---

## Comparison of Arrays

### Definitions

**Core Definition:** Arrays are objects in JavaScript, so they are compared by reference with `===` and `==`. Structural comparison requires custom logic.

**Technical Definition:** Since arrays are Objects, `IsStrictlyEqual` returns `true` only if both operands reference the same Array exotic object. `Array.prototype.includes()` uses SameValueZero for element comparison, but comparing two arrays requires manual iteration.

**Beginner-Friendly Explanation:** Two arrays are equal with `===` only if they're the same array. To check if they have the same elements, you need to compare them element by element.

### Purposes

- To understand why `[1,2] === [1,2]` is `false`.
- To implement structural array comparison.
- To know when to use `JSON.stringify()` (with caveats).

### Syntax Rules and Structure

```js
// Reference comparison
[1, 2, 3] === [1, 2, 3]; // false

// Structural comparison (manual)
function arraysEqual(a, b) {
  return a.length === b.length && a.every((v, i) => v === b[i]);
}
```

**Constraints and Limitations:**
- `JSON.stringify()` fails with key order, `undefined`, functions, circular references.
- Nested arrays require recursive comparison.
- `NaN` in arrays: `[NaN].includes(NaN)` is `true` (SameValueZero), but `[NaN].indexOf(NaN)` is `-1` (uses `===`).

### Annotated Code Examples

#### Example 1: Array Reference vs Content

```js
const a = [1, 2, 3];
const b = [1, 2, 3];
const c = a;

console.log(a === b);  // false
console.log(a === c);  // true
console.log(a == b);   // false (also reference comparison)
```

**Expected Output:**
```
false
true
false
```

**Why This Output:** `a` and `b` are different array objects. `c` references `a`.

#### Example 2: Structural Array Comparison

```js
function arraysEqual(a, b) {
  if (a === b) return true;
  if (a.length !== b.length) return false;
  for (let i = 0; i < a.length; i++) {
    if (a[i] !== b[i]) return false;
  }
  return true;
}

console.log(arraysEqual([1, 2, 3], [1, 2, 3])); // true
console.log(arraysEqual([1, 2], [1, 2, 3]));    // false
console.log(arraysEqual([1, 2, 3], [1, 2, 4])); // false
```

**Expected Output:**
```
true
false
false
```

**Why This Output:** The function compares length and each element.

#### Example 3: NaN in Arrays

```js
console.log([NaN].includes(NaN));   // true (SameValueZero)
console.log([NaN].indexOf(NaN));    // -1 (strict equality)
console.log([NaN].findIndex(Number.isNaN)); // 0
```

**Expected Output:**
```
true
-1
0
```

**Why This Output:** `includes()` uses SameValueZero, which treats `NaN` as equal to `NaN`. `indexOf()` uses strict equality, which does not.

### Real-World Cases

- **Form validation:** Comparing arrays of selected options.
- **State management:** Detecting array changes in Redux.
- **Testing:** `expect(arr1).toEqual(arr2)` for structural comparison.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Abstract Equality Comparison — https://tc39.es/ecma262/multipage/abstract-operations.html#sec-islooselyequal
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Strict Equality Comparison — https://tc39.es/ecma262/multipage/abstract-operations.html#sec-isstrictlyequal
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – SameValueZero — https://tc39.es/ecma262/multipage/abstract-operations.html#sec-samevaluezero
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Abstract Relational Comparison — https://tc39.es/ecma262/multipage/abstract-operations.html#sec-islessthan
- MDN Web Docs – Equality comparisons and sameness — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Equality_comparisons_and_sameness
- MDN Web Docs – Comparison operators — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators#comparison_operators
- MDN Web Docs – Equality (==) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Equality
- MDN Web Docs – Strict equality (===) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Strict_equality
- MDN Web Docs – Object.is() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
- MDN Web Docs – Greater than (>) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Greater_than
- MDN Web Docs – Less than (<) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Less_than
- MDN Web Docs – Operator precedence — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_precedence
- MDN Web Docs – Deep equality — https://developer.mozilla.org/en-US/docs/Glossary/Deep_equality
- Exploring JS – Equality, ordering, and sameness — https://exploringjs.com/impatient-js/ch_equality.html