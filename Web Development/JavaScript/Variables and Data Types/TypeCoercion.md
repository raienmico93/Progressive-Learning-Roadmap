# JavaScript Type Coercion — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Type coercion is the automatic or deliberate conversion of a value from one data type to another (such as a string to a number) so that an operation can proceed. It works on both primitive values and objects, but always results in a primitive data type. Coercion occurs in two forms: implicit (performed automatically by the JavaScript engine) and explicit (performed intentionally by the developer using built-in functions).

**Technical Definition:** Per ECMA-262, the ECMAScript runtime system performs automatic type conversion as needed through a set of abstract operations defined in §7.1. The core operations are ToPrimitive (which, for objects, consults `Symbol.toPrimitive`, then `valueOf()` and `toString()` in a hint-dependent order), ToNumber, ToString, ToBoolean, and ToObject. Implicit coercion occurs when these operations are invoked by language constructs (operators, comparison algorithms, logical contexts) without developer intervention. Explicit coercion occurs when the developer invokes conversion functions (`Number()`, `String()`, `Boolean()`, etc.) or uses coercion-prone operators (such as the unary `+`) deliberately.

**Beginner-Friendly Explanation:** JavaScript is a "weakly typed" language, meaning it will happily mix different types together. If you try to add a string to a number, JavaScript automatically converts one of them so the operation can work — that's implicit coercion. You can also convert types yourself using functions like `Number("42")` — that's explicit coercion. Coercion is convenient but can also cause surprising bugs if you don't understand the rules.

### Key Characteristics

- **Two directions:** Coercion can be implicit (engine-driven) or explicit (developer-driven).
- **Always results in a primitive:** Regardless of whether the input is a primitive or an object, the output of a coercion is always a primitive value (string, number, boolean, bigint, symbol, `null`, or `undefined`).
- **Context-dependent:** The same value may coerce differently depending on the operation. For example, `+` prefers string concatenation when either operand is a string, while `-`, `*`, and `/` always force numeric conversion.
- **Abstract operations drive it:** All coercion follows the ECMAScript abstract operations ToPrimitive, ToNumber, ToString, and ToBoolean.
- **Objects have special rules:** When an object is coerced to a primitive, JavaScript first calls `Symbol.toPrimitive` (if present), then falls back to `valueOf()` and `toString()` in an order determined by the "hint" (default, number, or string).

### Prerequisites

- Familiarity with JavaScript's primitive data types (String, Number, Boolean, `null`, `undefined`, Symbol, BigInt) and reference types (Object, Array, etc.).
- Understanding of the `typeof` operator and equality operators (`==`, `===`).
- Awareness of the distinction between values and references.

### Related Programming Areas

- **Equality comparisons:** The `==` operator performs coercion; `===` does not.
- **Arithmetic operations:** `+` is overloaded for addition and concatenation; other arithmetic operators force numeric conversion.
- **Logical operations:** Truthiness and falsiness determine control flow.
- **API and input validation:** Explicit coercion is a best practice at program boundaries.
- **Performance optimisation:** Avoiding hidden coercion in hot paths improves speed.

---

## Implicit Coercion

### Definitions

**Core Definition:** Implicit coercion is the automatic conversion of a value from one data type to another, performed by the JavaScript engine without an explicit instruction from the developer.

**Technical Definition:** Implicit coercion occurs when a language construct (an operator, a comparison algorithm, a logical context, or a built-in method) requires a value of a specific type and invokes the appropriate abstract operation (ToPrimitive, ToNumber, ToString, or ToBoolean) on its operand. The developer does not call a conversion function; the engine silently applies the rules.

**Beginner-Friendly Explanation:** Implicit coercion is JavaScript "doing the conversion for you." For example, when you write `"5" * 2`, JavaScript automatically turns `"5"` into the number `5` so it can multiply. You didn't ask for it — it just happened.

### Purposes

- To allow operations to proceed when operands have mismatched types.
- To make the language more forgiving for beginners and rapid prototyping.
- To enable concise idioms (such as `!!value` for boolean conversion or `+value` for numeric conversion).
- To support dynamic, loosely typed programming patterns.

### Syntax Rules and Structure

Implicit coercion is not triggered by a dedicated syntax construct; it is a behaviour of operators and language contexts. The key contexts are:

| Context | Coercion Direction | Example |
|---|---|---|
| `+` operator (either operand is a string) | To String | `"5" + 2` → `"52"` |
| `-`, `*`, `/`, `%` operators | To Number | `"5" - 2` → `3` |
| Loose equality (`==`) | Varies (ToPrimitive/ToNumber) | `"5" == 5` → `true` |
| Relational operators (`<`, `>`, etc.) | To Primitive (number or string) | `"10" > 5` → `true` |
| Logical operators (`&&`, `\|\|`, `!`) | To Boolean | `!0` → `true` |
| `if`, `while`, ternary conditions | To Boolean | `if ("")` → falsy |
| Template literals | To String | `` `${42}` `` → `"42"` |

**Syntax Rules:**
- The `+` operator has a special rule: if either operand is a string (or an object that coerces to a string), the operation becomes string concatenation.
- Arithmetic operators other than `+` always force numeric conversion.
- Logical operators and conditional contexts coerce to boolean using the ToBoolean abstract operation.

**Constraints and Limitations:**
- Implicit coercion can produce unexpected results (e.g., `[] + {}` → `"[object Object]"`).
- The `+` operator's dual behaviour (addition vs. concatenation) is a frequent source of bugs.
- Coercion of objects depends on `valueOf()` and `toString()`, which can be overridden by the developer.

### Annotated Code Examples

#### Example 1: Implicit Coercion with the `+` Operator

```js
// Step 1: String + Number → String concatenation.
console.log("20" + 18); // "2018" — number 18 coerced to string

// Step 2: Number + Boolean → Boolean coerced to number.
console.log(20 + true); // 21 — true coerced to 1

// Step 3: String * Number → String coerced to number.
console.log("20" * 18); // 360 — string "20" coerced to 20

// Step 4: Loose equality → coercion occurs.
console.log("20" == 20); // true — string "20" coerced to 20

// Step 5: Strict equality → no coercion.
console.log("20" === 20); // false — types differ
```

**Expected Output:**
```
2018
21
360
true
false
```

**Why This Output:** The `+` operator prefers string concatenation when either operand is a string. `true` becomes `1` in numeric context. The `*` operator forces numeric conversion. `==` coerces types, while `===` does not.

#### Example 2: Implicit Boolean Coercion in Conditions

```js
// Step 1: A non-empty string is truthy.
if ("hello") {
  console.log("String is truthy"); // executes
}

// Step 2: The number 0 is falsy.
if (0) {
  console.log("This will not run");
} else {
  console.log("0 is falsy"); // executes
}

// Step 3: An empty array is truthy (objects are always truthy).
if ([]) {
  console.log("[] is truthy"); // executes
}
```

**Expected Output:**
```
String is truthy
0 is falsy
[] is truthy
```

**Why This Output:** `if` converts its condition to boolean using ToBoolean. `"hello"` is truthy; `0` is falsy; `[]` is an object and objects are truthy regardless of content.

### Real-World Cases

- **Form input handling:** Form values are always strings; arithmetic with them triggers implicit numeric coercion.
- **URL query parameters:** Query strings are strings; comparing them to numbers with `==` can produce surprising results.
- **Legacy codebases:** Implicit coercion idioms (`!!x`, `+x`, `"" + x`) are common in older code.

---

## Explicit Coercion

### Definitions

**Core Definition:** Explicit coercion (also called type casting or type conversion) is the intentional conversion of a value from one data type to another, performed by the developer using built-in functions or operators.

**Technical Definition:** Explicit coercion occurs when the developer directly invokes a conversion function (`Number()`, `String()`, `Boolean()`, `parseInt()`, `parseFloat()`, `BigInt()`, `Symbol()`) or a coercion-prone operator (unary `+`, `!!`) to convert a value. Unlike implicit coercion, the developer controls when and how the conversion happens.

**Beginner-Friendly Explanation:** Explicit coercion is when you deliberately say "convert this value to a number" (or string, or boolean). You use functions like `Number()` or operators like `!!` to make your intent clear. This makes code easier to read and less bug-prone.

### Purposes

- To make type conversions explicit and readable.
- To avoid the surprises of implicit coercion.
- To validate and normalise input at program boundaries.
- To signal intent clearly to other developers.

### Syntax Rules and Structure

**Explicit Conversion Functions:**
```js
Number(value)      // convert to number
String(value)      // convert to string
Boolean(value)     // convert to boolean
parseInt(string, radix) // parse integer from string
parseFloat(string)      // parse floating-point from string
```

**Explicit Coercion Operators (idiomatic but less clear):**
```js
+value    // unary plus: convert to number
!!value   // double NOT: convert to boolean
value + "" // concatenate with empty string: convert to string
```

**Syntax Rules:**
- Conversion functions can be called without `new`; they return primitives, not wrapper objects.
- `parseInt()` stops at the first non-digit character and returns `NaN` if no digits are found.
- `parseFloat()` stops at the first non-numeric character (including a second decimal point).
- `Number()` attempts to convert the entire value; if any part is non-numeric (after trimming), the result is `NaN`.

**Constraints and Limitations:**
- `Number("")` returns `0` (empty string is treated as zero).
- `Number(null)` returns `0`; `Number(undefined)` returns `NaN`.
- `parseInt("10px")` returns `10`, but `Number("10px")` returns `NaN`. This is a crucial distinction.
- Always specify the radix (10) with `parseInt()` to avoid legacy octal interpretation.

### Annotated Code Examples

#### Example 1: Explicit Conversion Functions

```js
// Step 1: Number conversion.
console.log(Number("42"));    // 42
console.log(Number("7km"));   // NaN (whole string must be numeric)
console.log(Number(true));    // 1
console.log(Number(null));    // 0
console.log(Number(undefined)); // NaN

// Step 2: String conversion.
console.log(String(42));      // "42"
console.log(String(true));    // "true"
console.log(String(null));    // "null"

// Step 3: Boolean conversion.
console.log(Boolean(1));      // true
console.log(Boolean(0));      // false
console.log(Boolean(""));     // false
console.log(Boolean("0"));    // true (non-empty string)
```

**Expected Output:**
```
42
NaN
1
0
NaN
42
true
null
true
false
false
true
```

**Why This Output:** `Number()` converts the whole value; non-numeric strings yield `NaN`. `null` becomes `0`; `undefined` becomes `NaN`. `Boolean()` applies the falsy set: only `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN` become `false`.

#### Example 2: `parseInt()` vs. `Number()`

```js
// Step 1: parseInt stops at non-digits.
console.log(parseInt("42px"));     // 42
console.log(parseInt("3.14"));     // 3 (integer only)
console.log(parseInt("  10  "));   // 10 (leading whitespace ignored)

// Step 2: Number requires the whole value to be numeric.
console.log(Number("42px"));       // NaN
console.log(Number("3.14"));       // 3.14

// Step 3: parseFloat handles decimals but stops at invalid characters.
console.log(parseFloat("3.14abc")); // 3.14
```

**Expected Output:**
```
42
3
10
NaN
3.14
3.14
```

**Why This Output:** `parseInt` parses character by character and stops at the first invalid character; `Number` converts the entire string and returns `NaN` if any part is non-numeric. `parseFloat` behaves similarly to `parseInt` but accepts decimal points.

### Real-World Cases

- **Input validation:** `Number(input.value)` to ensure a form field contains a number.
- **API parameter parsing:** `parseInt(query.id, 10)` to extract an integer from a URL.
- **Boolean flags:** `Boolean(config.enabled)` to normalise truthy/falsy values.

---

## `String()`

### Definitions

**Core Definition:** `String()` is a built-in function that explicitly converts a value to its string representation.

**Technical Definition:** When called as a function (without `new`), `String(value)` invokes the ToString abstract operation on its argument. For objects, ToString calls ToPrimitive with the "string" hint, which invokes `toString()` then `valueOf()`. The result is always a primitive string.

**Beginner-Friendly Explanation:** `String()` turns anything into text. `String(42)` gives `"42"`; `String(true)` gives `"true"`; `String(null)` gives `"null"`.

### Purposes

- To convert numbers, booleans, and other values to strings explicitly.
- To make string conversion intent clear in code.
- To normalise values for concatenation or display.
- To avoid the ambiguity of implicit string coercion with `+`.

### Syntax Rules and Structure

```js
String(value)
```

**Component Breakdown:**
- `value`: Any JavaScript value.

**Syntax Rules:**
- `String()` without `new` returns a primitive string. `new String()` creates a wrapper object (avoid).
- `String(null)` returns `"null"`; `String(undefined)` returns `"undefined"`.
- `String(Symbol())` works; implicit concatenation with a Symbol throws.

### Annotated Code Examples

#### Example 1: Converting Various Types to Strings

```js
// Step 1: Numbers.
console.log(String(42));       // "42"
console.log(String(3.14));     // "3.14"

// Step 2: Booleans.
console.log(String(true));     // "true"
console.log(String(false));    // "false"

// Step 3: Null and undefined.
console.log(String(null));     // "null"
console.log(String(undefined)); // "undefined"

// Step 4: Objects (calls toString()).
console.log(String([1, 2, 3])); // "1,2,3"
console.log(String({}));        // "[object Object]"
```

**Expected Output:**
```
42
3.14
true
false
null
undefined
1,2,3
[object Object]
```

**Why This Output:** `String()` applies ToString. Arrays join their elements with commas; plain objects use the default `Object.prototype.toString()`.

### Real-World Cases

- **Display formatting:** Converting numbers to strings for display.
- **Building messages:** `String(error)` to get the error message.
- **Key generation:** `String(key)` before using a value as an object property key.

---

## `Number()`

### Definitions

**Core Definition:** `Number()` is a built-in function that explicitly converts a value to a number.

**Technical Definition:** `Number(value)` invokes the ToNumber abstract operation. For strings, it trims whitespace and attempts to parse the entire remaining string as a numeric literal (decimal, hexadecimal with `0x`, binary with `0b`, octal with `0o`, `Infinity`, `NaN`). An empty string becomes `0`. Booleans become `1` or `0`. `null` becomes `0`; `undefined` becomes `NaN`. Objects are first converted to primitives.

**Beginner-Friendly Explanation:** `Number()` tries to turn a value into a number. `Number("42")` gives `42`; `Number("hello")` gives `NaN` (Not a Number). Unlike `parseInt()`, it requires the entire string to be numeric.

### Purposes

- To convert strings to numbers explicitly.
- To convert booleans to `1`/`0`.
- To normalise values for arithmetic.
- To validate numeric input (checking for `NaN` after conversion).

### Syntax Rules and Structure

```js
Number(value)
```

**Constraints and Limitations:**
- `Number("")` returns `0` (often surprising).
- `Number(" 42 ")` returns `42` (whitespace is trimmed).
- `Number("42px")` returns `NaN` (unlike `parseInt`).
- `Number(Symbol())` throws a `TypeError`.
- `Number(10n)` returns `10` (BigInt to Number, potentially lossy).

### Annotated Code Examples

#### Example 1: `Number()` Conversion Behaviour

```js
// Step 1: Numeric strings.
console.log(Number("42"));      // 42
console.log(Number("3.14"));    // 3.14
console.log(Number("  42  "));  // 42 (whitespace trimmed)

// Step 2: Non-numeric strings.
console.log(Number("42px"));    // NaN
console.log(Number("hello"));   // NaN

// Step 3: Special values.
console.log(Number(""));        // 0
console.log(Number(" "));       // 0
console.log(Number(true));      // 1
console.log(Number(false));     // 0
console.log(Number(null));      // 0
console.log(Number(undefined)); // NaN
```

**Expected Output:**
```
42
3.14
42
NaN
NaN
0
0
1
0
0
NaN
```

**Why This Output:** `Number()` invokes ToNumber, which trims whitespace, treats the empty string as zero, and returns `NaN` for any unparseable content.

### Real-World Cases

- **Form validation:** `Number(input.value)` and checking `!Number.isNaN(...)`.
- **Configuration parsing:** Converting environment variables (always strings) to numbers.
- **API payloads:** Ensuring numeric fields are numbers before sending.

---

## `Boolean()`

### Definitions

**Core Definition:** `Boolean()` is a built-in function that explicitly converts a value to `true` or `false` according to JavaScript's truthiness rules.

**Technical Definition:** `Boolean(value)` invokes the ToBoolean abstract operation. The result is `false` for exactly eight values: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN`. All other values — including non-empty strings, non-zero numbers, objects, arrays, and functions — become `true`.

**Beginner-Friendly Explanation:** `Boolean()` asks: "Is this value truthy or falsy?" `Boolean(1)` is `true`; `Boolean(0)` is `false`; `Boolean("")` is `false`; `Boolean("0")` is `true`.

### Purposes

- To convert any value to a boolean explicitly.
- To normalise truthy/falsy values for storage or comparison.
- To make boolean intent clear (instead of relying on implicit coercion in conditions).
- To implement type guards.

### Syntax Rules and Structure

```js
Boolean(value)
```

**The Falsy Set (the only values that return `false`):**
- `false`
- `0`
- `-0`
- `0n` (BigInt zero)
- `""` (empty string)
- `null`
- `undefined`
- `NaN`

All other values are truthy.

### Annotated Code Examples

#### Example 1: Falsy and Truthy Values

```js
// Step 1: All falsy values return false.
console.log(Boolean(false));    // false
console.log(Boolean(0));        // false
console.log(Boolean(-0));       // false
console.log(Boolean(0n));       // false
console.log(Boolean(""));       // false
console.log(Boolean(null));     // false
console.log(Boolean(undefined));// false
console.log(Boolean(NaN));      // false

// Step 2: Surprising truthy values.
console.log(Boolean("0"));      // true (non-empty string)
console.log(Boolean("false"));  // true (non-empty string)
console.log(Boolean([]));       // true (object)
console.log(Boolean({}));       // true (object)
```

**Expected Output:**
```
false
false
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

**Why This Output:** Only the eight falsy values convert to `false`. Non-empty strings — even `"0"` and `"false"` — are truthy because they have length greater than zero. Objects and arrays are always truthy.

### Real-World Cases

- **Storing flags:** `Boolean(input.checked)` to get a clean boolean.
- **Configuration:** `Boolean(process.env.DEBUG)` to normalise an environment variable.
- **Validation:** Checking whether a value is truthy before proceeding.

---

## `parseInt()`

### Definitions

**Core Definition:** `parseInt()` is a built-in function that parses a string argument and returns an integer of the specified radix (base).

**Technical Definition:** `parseInt(string, radix)` converts its first argument to a string, trims leading whitespace, and reads characters from left to right as long as they form a valid integer in the given radix. It stops at the first invalid character and returns the parsed integer. If no valid digit is found, it returns `NaN`. The radix is an integer between 2 and 36.

**Beginner-Friendly Explanation:** `parseInt()` extracts a whole number from the beginning of a string. `parseInt("42px")` gives `42`; `parseInt("3.14")` gives `3`; `parseInt("hello")` gives `NaN`.

### Purposes

- To extract an integer from a string that may contain non-numeric characters.
- To parse user input or URL parameters into integers.
- To convert strings in a specific base (binary, octal, hexadecimal) to decimal integers.

### Syntax Rules and Structure

```js
parseInt(string)
parseInt(string, radix)
```

**Component Breakdown:**
- `string`: The value to parse (converted to string if not already a string).
- `radix` (optional but strongly recommended): An integer between 2 and 36 representing the base. If omitted or `0`, JavaScript guesses the base (often decimal, but legacy octal for leading `0`).

**Constraints and Limitations:**
- Always specify the radix (`10` for decimal) to avoid legacy octal behaviour and ensure predictable results.
- `parseInt("0x10")` returns `16` (hexadecimal detected).
- `parseInt("10", 2)` returns `2` (binary interpretation).
- `parseInt("08")` may return `8` in modern engines, but `parseInt("08", 10)` is explicit and safe.

### Annotated Code Examples

#### Example 1: Parsing Integers from Strings

```js
// Step 1: Standard decimal parsing.
console.log(parseInt("42"));      // 42
console.log(parseInt("42px"));    // 42 (stops at 'p')
console.log(parseInt("3.14"));    // 3 (stops at '.')

// Step 2: With radix.
console.log(parseInt("1010", 2)); // 10 (binary)
console.log(parseInt("FF", 16));  // 255 (hexadecimal)

// Step 3: Invalid input.
console.log(parseInt("hello"));   // NaN
console.log(parseInt(""));        // NaN
```

**Expected Output:**
```
42
42
3
10
255
NaN
NaN
```

**Why This Output:** `parseInt` reads valid digits from left to right and stops at the first invalid character. If no valid digits exist, it returns `NaN`. The radix determines the interpretation of digits.

### Real-World Cases

- **Parsing CSS values:** `parseInt("16px", 10)` → `16`.
- **URL parameters:** `parseInt(pageParam, 10)` to get the page number.
- **Binary/hex input:** Converting user-supplied hex colour codes.

---

## `parseFloat()`

### Definitions

**Core Definition:** `parseFloat()` is a built-in function that parses a string argument and returns a floating-point number.

**Technical Definition:** `parseFloat(string)` converts its argument to a string, trims leading whitespace, and reads characters as long as they form a valid floating-point number (digits, one decimal point, exponent notation). It stops at the first invalid character and returns the parsed number. If no valid number is found, it returns `NaN`.

**Beginner-Friendly Explanation:** `parseFloat()` extracts a decimal number from the beginning of a string. `parseFloat("3.14abc")` gives `3.14`; `parseFloat("42")` gives `42`.

### Purposes

- To extract a decimal number from a string with trailing non-numeric characters.
- To parse user input that may contain units (e.g., `"3.5kg"`).
- To handle floating-point data from external sources.

### Syntax Rules and Structure

```js
parseFloat(string)
```

**Constraints and Limitations:**
- `parseFloat` does not accept a radix; it always parses decimal (and recognises `Infinity`, `NaN`, and exponent notation).
- `parseFloat("3.14.15")` returns `3.14` (stops at the second decimal point).
- Leading zeros are ignored; `parseFloat("0.5")` returns `0.5`.

### Annotated Code Examples

#### Example 1: Parsing Floating-Point Values

```js
// Step 1: Basic decimal parsing.
console.log(parseFloat("3.14"));     // 3.14
console.log(parseFloat("3.14abc"));  // 3.14 (stops at 'a')
console.log(parseFloat("42"));       // 42

// Step 2: Scientific notation.
console.log(parseFloat("1e3"));      // 1000

// Step 3: Invalid input.
console.log(parseFloat("abc"));      // NaN
console.log(parseFloat(""));         // NaN
```

**Expected Output:**
```
3.14
3.14
42
1000
NaN
NaN
```

**Why This Output:** `parseFloat` reads valid floating-point syntax and stops at the first invalid character. If no valid number exists, it returns `NaN`.

### Real-World Cases

- **Measurements:** `parseFloat("3.5kg")` → `3.5`.
- **Financial input:** Parsing decimal currency amounts from strings.
- **Scientific data:** Parsing exponent-notation values from CSVs.

---

## `+` Operator

### Definitions

**Core Definition:** The `+` operator is overloaded in JavaScript: it performs numeric addition when both operands are numbers (or coerce to numbers), and string concatenation when either operand is a string.

**Technical Definition:** The `+` operator evaluates its operands and applies the ToPrimitive abstract operation. If either resulting primitive is a string, both operands are converted to strings (ToString) and concatenated. Otherwise, both operands are converted to numbers (ToNumber) and added. This dual behaviour is unique to `+`; `-`, `*`, `/`, and `%` always force numeric conversion.

**Beginner-Friendly Explanation:** `+` is "smart" but sometimes too clever. If you add a string to anything, it turns everything into text and glues it together. If you add numbers, it does maths. `"5" + 2` gives `"52"`, but `"5" - 2` gives `3`.

### Purposes

- To perform addition of numbers.
- To concatenate strings.
- To convert values to numbers (via unary `+`).
- To convert values to strings (by adding `""`).

### Syntax Rules and Structure

```js
operand1 + operand2
+operand    // unary plus: ToNumber
```

**Syntax Rules:**
- Binary `+` with a string operand → string concatenation.
- Binary `+` without a string operand → numeric addition.
- Unary `+` always applies ToNumber.
- `+` is left-associative: `1 + 2 + "3"` → `"33"` (1+2=3, then `3+"3"="33"`).

**Constraints and Limitations:**
- The dual behaviour makes `+` a frequent source of bugs.
- Objects coerce to primitives via ToPrimitive; `[] + []` → `""`, `[] + {}` → `"[object Object]"`.
- Using `+` for string conversion is discouraged in favour of `String()`.

### Annotated Code Examples

#### Example 1: Binary `+` Behaviour

```js
// Step 1: Numeric addition.
console.log(1 + 2);        // 3

// Step 2: String concatenation.
console.log("1" + 2);      // "12"
console.log(1 + "2");      // "12"

// Step 3: Left associativity.
console.log(1 + 2 + "3");  // "33" (1+2=3, then "3"+"3"? No: 3+"3" = "33")

// Step 4: Unary plus converts to number.
console.log(+"42");        // 42
console.log(+"3.14");      // 3.14
console.log(+true);        // 1
console.log(+"");          // 0
```

**Expected Output:**
```
3
12
12
33
42
3.14
1
0
```

**Why This Output:** Binary `+` with any string operand concatenates. Unary `+` applies ToNumber. Left associativity means `1 + 2` is evaluated first, then `3 + "3"` becomes `"33"`.

### Real-World Cases

- **Building strings:** `"Hello, " + name`.
- **Numeric conversion (legacy):** `+input.value` to convert a string to a number (prefer `Number()` for clarity).
- **Template literals:** `+` is rarely needed for string building now that template literals exist.

---

## `==` vs `===`

### Definitions

**Core Definition:** `==` (loose equality) compares two values after performing type coercion if their types differ; `===` (strict equality) compares two values without coercion, requiring both the type and the value to be identical.

**Technical Definition:** `==` invokes the IsLooselyEqual abstract operation, which applies a complex set of rules involving ToPrimitive and ToNumber when operand types differ. `===` invokes IsStrictlyEqual, which returns `false` if the types differ and otherwise compares the values (with special handling for `NaN`, `+0`, and `-0`).

**Beginner-Friendly Explanation:** `==` asks "are these roughly equal?" and will convert types to try to make them match. `===` asks "are these exactly equal?" — same type and same value. `===` is almost always the safer choice.

### Purposes

- `==` : To compare values of different types when you intentionally want coercion (rare).
- `===` : To compare values safely and predictably without hidden conversions.
- `!==` : The strict inequality counterpart of `===`.

### Syntax Rules and Structure

```js
a == b   // loose equality (coerces)
a === b  // strict equality (no coercion)
a != b   // loose inequality
a !== b  // strict inequality
```

**Key Difference:**
- `"5" == 5` → `true` (string coerced to number).
- `"5" === 5` → `false` (types differ).

**Constraints and Limitations:**
- `==` has non-transitive behaviour (`"" == 0` is `true`, `0 == "0"` is `true`, but `"" == "0"` is `false`).
- `null == undefined` is `true`, but `null === undefined` is `false`.
- `NaN == NaN` is `false`; `NaN === NaN` is also `false`.
- Using `==` with `true`/`false` is particularly error-prone (`false == 0` is `true`).

### Annotated Code Examples

#### Example 1: Loose vs. Strict Equality

```js
// Step 1: Same type, same value.
console.log(5 == 5);    // true
console.log(5 === 5);   // true

// Step 2: Different types.
console.log("5" == 5);  // true (coercion)
console.log("5" === 5); // false (no coercion)

// Step 3: null and undefined.
console.log(null == undefined);  // true
console.log(null === undefined); // false

// Step 4: NaN.
console.log(NaN == NaN);  // false
console.log(NaN === NaN); // false
```

**Expected Output:**
```
true
true
true
false
true
false
false
false
```

**Why This Output:** `==` coerces `"5"` to `5`, making the comparison `true`. `===` requires identical types. `null` and `undefined` are loosely equal but not strictly equal. `NaN` is never equal to itself under either operator.

### Real-World Cases

- **General comparisons:** Always prefer `===` unless you have a specific reason to use `==`.
- **Null checks:** `value == null` is a deliberate idiom to check for both `null` and `undefined` in one comparison.
- **Code reviews:** Linters like ESLint enforce `===` by default (`eqeqeq` rule).

---

## `!=` vs `!==`

### Definitions

**Core Definition:** `!=` is the loose inequality operator (the negation of `==`); `!==` is the strict inequality operator (the negation of `===`).

**Technical Definition:** `a != b` is equivalent to `!(a == b)`; `a !== b` is equivalent to `!(a === b)`. Therefore, `!=` performs coercion and `!==` does not.

**Beginner-Friendly Explanation:** `!=` means "not loosely equal"; `!==` means "not strictly equal." Just as `===` is safer than `==`, `!==` is safer than `!=`.

### Purposes

- `!==` : To check that two values are not identical in type and value.
- `!=` : Rarely used; the same caveats as `==` apply.

### Syntax Rules and Structure

```js
a != b   // loose inequality
a !== b  // strict inequality
```

**Constraints and Limitations:** All the coercion pitfalls of `==` apply to `!=`.

### Annotated Code Examples

#### Example 1: Loose vs. Strict Inequality

```js
console.log("5" != 5);   // false (because "5" == 5 is true)
console.log("5" !== 5);  // true (because "5" === 5 is false)

console.log(null != undefined);  // false
console.log(null !== undefined); // true
```

**Expected Output:**
```
false
true
false
true
```

**Why This Output:** `!=` negates the coerced result of `==`; `!==` negates the non-coerced result of `===`.

### Real-World Cases

- **Guard clauses:** `if (value !== null && value !== undefined)`.
- **Avoiding coercion bugs:** Use `!==` everywhere `!=` might be tempting.

---

## Truthy and Falsy Values

### Definitions

**Core Definition:** Truthy and falsy values describe how non-boolean values behave when converted to a boolean in a logical context (such as an `if` condition or a logical operator).

**Technical Definition:** JavaScript's ToBoolean abstract operation maps exactly eight values to `false` (the falsy set) and all other values to `true` (the truthy set).

**Beginner-Friendly Explanation:** In a condition, JavaScript treats some values as "false-like" (falsy) and everything else as "true-like" (truthy). Knowing which values are falsy helps you avoid bugs.

### Purposes

- To control program flow with any value, not just booleans.
- To write concise conditionals and default-value patterns.
- To understand why certain values behave unexpectedly in conditions.

### Syntax Rules and Structure

The falsy set is fixed and consists of exactly these eight values:

| Falsy Value | Type |
|---|---|
| `false` | Boolean |
| `0` | Number |
| `-0` | Number |
| `0n` | BigInt |
| `""` | String |
| `null` | Null |
| `undefined` | Undefined |
| `NaN` | Number |

Every other value — including `"0"`, `"false"`, `[]`, `{}`, and functions — is truthy.

### Annotated Code Examples

#### Example 1: Testing the Falsy Set

```js
// Step 1: All eight falsy values.
const falsyValues = [false, 0, -0, 0n, "", null, undefined, NaN];
falsyValues.forEach((v) => {
  console.log(`${String(v)} → ${Boolean(v)}`);
});

// Step 2: Surprising truthy values.
console.log(Boolean("0"));    // true
console.log(Boolean(" "));    // true (space is non-empty)
console.log(Boolean([]));     // true
console.log(Boolean({}));     // true
```

**Expected Output:**
```
false → false
0 → false
0 → false
0 → false
  → false
null → false
undefined → false
NaN → false
true
true
true
true
```

**Why This Output:** Only the eight falsy values convert to `false`. Empty arrays and objects are truthy because they are objects.

### Real-World Cases

- **Default values:** `const name = input || "Guest"` (but beware: `0` and `""` are valid values that would be replaced).
- **Conditional rendering:** `{items.length && <List items={items} />}` (renders `0` when empty — a classic bug).
- **Nullish coalescing:** Use `??` instead of `||` when `0` and `""` are valid values.

---

## Falsy Values (Detailed)

### `false`

**Definitions:**
- **Core:** The boolean primitive `false`.
- **Technical:** The Boolean value `false` is the canonical falsy value; ToBoolean(`false`) returns `false`.
- **Beginner:** The literal `false`.

**Example:** `Boolean(false)` → `false`.

---

### `0`

**Definitions:**
- **Core:** The number zero.
- **Technical:** ToBoolean(`+0`) returns `false`. `0` is the integer zero in the Number type.
- **Beginner:** The number `0` is falsy.

**Example:** `Boolean(0)` → `false`.

---

### `-0`

**Definitions:**
- **Core:** Negative zero.
- **Technical:** IEEE 754 distinguishes `+0` and `-0`, but ToBoolean treats both as `false`. `-0 === 0` is `true`.
- **Beginner:** `-0` is effectively zero and is falsy.

**Example:** `Boolean(-0)` → `false`.

---

### `0n`

**Definitions:**
- **Core:** BigInt zero.
- **Technical:** The BigInt value `0n` is falsy. ToBoolean(`0n`) returns `false`.
- **Beginner:** Zero as a BigInt is falsy.

**Example:** `Boolean(0n)` → `false`.

---

### `""` (empty string)

**Definitions:**
- **Core:** A string of length zero.
- **Technical:** ToBoolean of the empty string returns `false`. Any non-empty string, including `"0"`, `"false"`, and whitespace, is truthy.
- **Beginner:** An empty string is falsy.

**Example:** `Boolean("")` → `false`.

---

### `null`

**Definitions:**
- **Core:** The null primitive, representing intentional absence.
- **Technical:** ToBoolean(`null`) returns `false`.
- **Beginner:** `null` means "no value" and is falsy.

**Example:** `Boolean(null)` → `false`.

---

### `undefined`

**Definitions:**
- **Core:** The undefined primitive, representing an uninitialised or missing value.
- **Technical:** ToBoolean(`undefined`) returns `false`.
- **Beginner:** `undefined` means "not set" and is falsy.

**Example:** `Boolean(undefined)` → `false`.

---

### `NaN`

**Definitions:**
- **Core:** "Not-a-Number," a special Number value representing an invalid numeric result.
- **Technical:** ToBoolean(`NaN`) returns `false`. `NaN` is also unique in that it is not equal to itself under `===`.
- **Beginner:** `NaN` means "not a valid number" and is falsy.

**Example:** `Boolean(NaN)` → `false`.

---

## Truthy Values

### Definitions

**Core Definition:** Truthy values are all values that are not in the falsy set; they convert to `true` in boolean contexts.

**Technical Definition:** The complement of the falsy set. Any value that is not `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, or `NaN` is truthy.

**Beginner-Friendly Explanation:** Almost everything is truthy! Even `"0"`, `"false"`, `[]`, and `{}` are truthy because they are not in the falsy list.

### Purposes

- To leverage concise conditionals.
- To understand why objects and non-empty strings always pass truthiness checks.
- To avoid bugs when checking for "empty" values.

### Annotated Code Examples

#### Example 1: Surprising Truthy Values

```js
console.log(Boolean("0"));      // true
console.log(Boolean("false"));  // true
console.log(Boolean([]));       // true
console.log(Boolean({}));       // true
console.log(Boolean(-1));       // true
console.log(Boolean(Infinity)); // true
```

**Expected Output:**
```
true
true
true
true
true
true
```

**Why This Output:** None of these values are in the falsy set, so they are truthy.

### Real-World Cases

- **Checking for empty arrays:** `if (arr.length)` (correct) vs. `if (arr)` (always truthy).
- **Default parameters:** `function f(x = 10)` uses `undefined` as the trigger, not falsiness.
- **Optional chaining:** `obj?.prop` handles `null`/`undefined` without truthiness checks.

---

## Coercion Pitfalls

### Definitions

**Core Definition:** Coercion pitfalls are surprising or error-prone behaviours that arise from JavaScript's implicit type conversion rules.

**Technical Definition:** Pitfalls occur when the abstract operations ToPrimitive, ToNumber, ToString, or ToBoolean produce results that differ from developer expectations — often due to the `+` operator's dual behaviour, the loose equality algorithm's complexity, truthiness of objects, or the special values `NaN`, `null`, and `undefined`.

**Beginner-Friendly Explanation:** Coercion pitfalls are the "gotchas" of JavaScript — the weird results that make you say "wait, what?" Examples: `[] + []` is `""`, `[] == false` is `true`, `"0"` is truthy.

### Common Pitfalls

**Pitfall 1: `+` with Mixed Types**
```js
1 + "2"       // "12" (not 3)
"3" - 1       // 2 (numeric coercion)
[] + {}       // "[object Object]"
```

**Pitfall 2: Loose Equality Chains**
```js
"" == 0       // true
0 == "0"      // true
"" == "0"     // false (non-transitive)
```

**Pitfall 3: Truthiness of Objects**
```js
Boolean([])   // true
Boolean({})   // true
if ([]) { }   // always executes
```

**Pitfall 4: `NaN` Comparisons**
```js
NaN === NaN   // false
Number.isNaN(NaN) // true (correct check)
```

**Pitfall 5: `null` and `undefined`**
```js
null == undefined  // true
null === undefined // false
Number(null)       // 0
Number(undefined)  // NaN
```

**Pitfall 6: `parseInt` Without Radix**
```js
parseInt("08")  // may be 8 or legacy octal (implementation-dependent)
parseInt("08", 10) // 8 (safe)
```

### Annotated Code Examples

#### Example 1: Classic Coercion Pitfall Chain

```js
// Step 1: Empty array + empty object.
console.log([] + {}); // "[object Object]"

// Step 2: Empty array + empty array.
console.log([] + []); // ""

// Step 3: Loose equality with false.
console.log([] == false); // true ([] → "" → 0, false → 0)
```

**Expected Output:**
```
[object Object]

true
```

**Why This Output:** `[]` coerces to `""`, then to `0` in numeric context. `{}` coerces to `"[object Object]"`. Loose equality triggers a multi-step conversion that results in `0 == 0`.

### Real-World Cases

- **Form validation:** `if (input.value)` fails for `"0"`? No — `"0"` is truthy, but `0` (number) is falsy.
- **API responses:** `data.count || "N/A"` replaces `0` with `"N/A"` (a bug; use `??`).
- **Sorting:** Default `sort()` converts elements to strings, causing `[10, 2].sort()` → `[10, 2]`.

---

## Coercion Best Practices

### Definitions

**Core Definition:** Coercion best practices are the guidelines for using type conversion safely, predictably, and readably.

**Technical Definition:** Best practices centre on preferring explicit conversion over implicit coercion, using strict equality (`===`) instead of loose equality (`==`), converting values once at program boundaries, avoiding coercion in hot paths, and using the nullish coalescing operator (`??`) when `0` and `""` are valid values.

**Beginner-Friendly Explanation:** The golden rule is: be explicit. If you need a number, use `Number()`. If you need to compare, use `===`. Don't rely on JavaScript to guess what you mean.

### Key Best Practices

1. **Prefer explicit conversion functions:** Use `Number()`, `String()`, and `Boolean()` instead of coercion-prone operators like `+`, `!!`, or `"" +`. This improves readability.

2. **Use `===` and `!==`:** Strict equality avoids the complex and error-prone coercion rules of `==`. The only common exception is `value == null` to check for both `null` and `undefined`.

3. **Convert at boundaries:** Convert inputs (API responses, form data, URL parameters) to the correct type once, at the point of entry. Keep data in the correct types throughout the rest of the program.

4. **Avoid coercion in hot paths:** Repeated implicit coercion inside loops or sort comparators can cause performance bottlenecks. Convert once outside the loop.

5. **Use `??` instead of `||` for defaults:** The `||` operator treats `0`, `""`, and `NaN` as falsy and replaces them. The nullish coalescing operator (`??`) only falls back for `null` and `undefined`, preserving valid falsy values.

6. **Be explicit about boolean intent:** Use `Boolean(value)` rather than `!!value` when the goal is readability.

7. **Always specify the radix in `parseInt()`:** `parseInt(value, 10)` prevents legacy octal surprises.

### Annotated Code Examples

#### Example 1: Applying Best Practices

```js
// BAD: implicit coercion and loose equality.
function badExample(input) {
  if (input == 0) { /* ... */ }
  const num = +input;
  return num;
}

// GOOD: explicit conversion and strict equality.
function goodExample(input) {
  const num = Number(input);
  if (num === 0) { /* ... */ }
  return num;
}
```

**Expected Output:** No runtime difference for valid numeric input; the good version is clearer and avoids coercion surprises.

#### Example 2: Nullish Coalescing vs. Logical OR

```js
// BAD: || replaces 0 and "" with the default.
const count = data.count || 10; // if data.count is 0, result is 10

// GOOD: ?? only replaces null/undefined.
const count2 = data.count ?? 10; // if data.count is 0, result is 0
```

**Expected Output:**
```
10  (if data.count is 0, using ||)
0   (if data.count is 0, using ??)
```

**Why This Output:** `||` treats `0` as falsy and substitutes the default; `??` only triggers on `null` or `undefined`, preserving `0` as a valid value.

### Real-World Cases

- **API clients:** Parsing JSON and immediately converting fields to their expected types.
- **Form handling:** Converting `input.value` (always a string) to the appropriate type once.
- **Configuration:** Using `??` to provide defaults without overriding valid `0` or `""` values.

---

## References

- ECMAScript Language Specification (ECMA-262, 2023) – Type Conversion and Testing (Abstract Operations §7.1) — https://tc39.es/ecma262/2023/multipage/abstract-operations.html
- ECMAScript Language Specification – ToPrimitive — https://tc39.es/ecma262/multipage/abstract-operations.html#sec-toprimitive
- ECMAScript Language Specification – ToNumber — https://tc39.es/ecma262/multipage/abstract-operations.html#sec-tonumber
- ECMAScript Language Specification – ToBoolean — https://tc39.es/ecma262/multipage/abstract-operations.html#sec-toboolean
- MDN Web Docs – Type coercion — https://developer.mozilla.org/en-US/docs/Glossary/Type_coercion
- MDN Web Docs – Equality comparisons and sameness — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Equality_comparisons_and_sameness
- MDN Web Docs – Symbol.toPrimitive — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toPrimitive
- MDN Web Docs – Number() constructor — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/Number
- MDN Web Docs – parseInt() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt
- MDN Web Docs – parseFloat() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseFloat
- MDN Web Docs – Truthy — https://developer.mozilla.org/en-US/docs/Glossary/Truthy
- MDN Web Docs – Falsy — https://developer.mozilla.org/en-US/docs/Glossary/Falsy
- JavaScript in Depth (Manning) – Primitives, Coercion, and Equality — https://www-qa.manning.com/preview/javascript-in-depth/chapter-4
- DeepSource – Found shorthand type coercions (JS-0066) — https://app.deepsource.com/gh/atcoder-for-chinese-developers/atcoder-for-chinese/issue/JS-0066/occurrences?listindex=0
- Codecademy – JavaScript Type Coercion — https://portal-app.production-eks.codecademy.com/resources/docs/javascript/type-coercion