# JavaScript Numbers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The Number type in JavaScript represents all numeric values — integers and floating-point decimals alike — using a single 64-bit double-precision format, with special values for infinity and invalid operations.

**Technical Definition:** Per ECMA-262, the Number type is the set of all possible Number values, including the special values `NaN` (not-a-number), `+0`, `−0`, `+∞`, and `−∞`. Number values are represented using the IEEE 754-2019 binary64 (double-precision) format, which allocates 1 bit for the sign, 11 bits for the exponent, and 52 bits for the significand (mantissa). Integers can be represented exactly up to ±(2⁵³ − 1), beyond which precision is lost. The `Number` object provides static constants (`MAX_VALUE`, `MAX_SAFE_INTEGER`, etc.) and static methods (`isInteger()`, `parseInt()`, etc.) for working with Number values. The `Math` namespace object provides additional mathematical constants and functions.

**Beginner-Friendly Explanation:** JavaScript has only one number type: a floating-point number that can hold both whole numbers (like `42`) and decimals (like `3.14`). Because it uses a fixed number of bits, some very large integers and many decimal fractions cannot be stored perfectly — this is why `0.1 + 0.2` does not exactly equal `0.3`. JavaScript also has special values: `Infinity` (larger than any number), `-Infinity`, and `NaN` ("not a number") to represent invalid numeric results.

### Key Characteristics

- **Single numeric type:** There is no separate `int` or `float` type; all numbers are IEEE 754 double-precision floating-point values.
- **Safe integer range:** Integers between −(2⁵³ − 1) and 2⁵³ − 1 are exact; beyond this, precision is lost.
- **Special values:** `NaN`, `Infinity`, `-Infinity`, `+0`, and `-0` are part of the type.
- **Immutability:** Number primitives are immutable; operations return new numbers.
- **Literal syntaxes:** Numbers can be written in decimal, hexadecimal, octal, binary, exponent notation, and with numeric separators.
- **`Math` companion:** The `Math` object provides constants (`PI`, `E`) and functions (`round`, `floor`, `max`, `random`, etc.) for numeric computation.
- **Not JSON-friendly for special values:** `NaN` and `Infinity` serialise to `null` in `JSON.stringify()`.

### Prerequisites

- Understanding of JavaScript variables and primitive types.
- Familiarity with the `typeof` operator.
- Basic arithmetic operators (`+`, `-`, `*`, `/`, `%`, `**`).

### Related Programming Areas

- **Type coercion:** Numbers are frequently coerced from strings (see the Type Coercion cheat sheet).
- **BigInt:** For integers beyond the safe range, BigInt provides arbitrary precision.
- **Floating-point arithmetic:** Numerical stability and precision are critical in finance, science, and graphics.
- **Math object:** Geometry, statistics, simulations, and games.

---

## Number Literals

### Definitions

**Core Definition:** A number literal is a syntactic representation of a numeric value directly in source code.

**Technical Definition:** ECMAScript defines decimal, hexadecimal, octal, and binary numeric literals. Decimal literals may include a decimal point and an exponent part. Hexadecimal literals use `0x` or `0X`; binary uses `0b` or `0B`; octal uses `0o` or `0O`. Numeric separators (`_`) may be inserted between digits to improve readability.

**Beginner-Friendly Explanation:** A number literal is just a number you write directly in your code — `42`, `3.14`, `0xFF`, `1_000_000`. It is the raw value, not a variable.

### Purposes

- To represent fixed numeric values in code.
- To write numbers in the base most natural for the problem (hex for colours, binary for flags).
- To improve readability with separators and exponent notation.

### Syntax Rules and Structure

**General Syntax:**
```js
// Decimal
42
3.14
.5
1e6
1E-6

// Hexadecimal
0xFF
0XFF

// Binary
0b1010
0B1010

// Octal
0o755
0O755

// Numeric separators (ES2021)
1_000_000
0xFF_EC_DE_5E
```

**Component Breakdown:**
- `0x` / `0X`: Hexadecimal prefix; digits `0–9`, `A–F`.
- `0b` / `0B`: Binary prefix; digits `0` and `1`.
- `0o` / `0O`: Octal prefix; digits `0–7`.
- `e` / `E`: Exponent marker (`1e6` = 1 × 10⁶).
- `_`: Numeric separator (ignored by the parser; must be between digits).

**Constraints and Limitations:**
- Legacy octal syntax (leading `0` without `o`, e.g., `0644`) is forbidden in strict mode.
- Numeric separators cannot be leading, trailing, or adjacent to the decimal point (`_1`, `1_`, `1._0` are invalid).
- Hexadecimal literals cannot include a decimal point.

### Annotated Code Examples

#### Example 1: All Literal Forms

```js
// Step 1: Decimal (integer and floating-point).
console.log(42);        // 42
console.log(3.14);      // 3.14
console.log(.5);        // 0.5

// Step 2: Hexadecimal.
console.log(0xFF);      // 255
console.log(0x10);      // 16

// Step 3: Binary.
console.log(0b1010);    // 10
console.log(0B1111);    // 15

// Step 4: Octal.
console.log(0o755);     // 493
console.log(0O10);      // 8

// Step 5: Exponent notation.
console.log(1e3);       // 1000
console.log(2.5e-2);    // 0.025

// Step 6: Numeric separators.
console.log(1_000_000); // 1000000
console.log(0xFF_FF);   // 65535
```

**Expected Output:**
```
42
3.14
0.5
255
16
10
15
493
8
1000
0.025
1000000
65535
```

**Why This Output:** Each literal is parsed according to its prefix or format. The separators are ignored by the engine; the numeric value is the same as without them.

### Real-World Cases

- **Colour codes:** `0xFF0000` for red in graphics.
- **File permissions (Unix):** `0o755` for executable files.
- **Bitmasks and flags:** `0b1010` for permission bits.
- **Readable large constants:** `1_000_000` for one million.

---

## Integer Literals

### Definitions

**Core Definition:** An integer literal is a number literal without a fractional part or exponent.

**Technical Definition:** Integer literals are decimal, hexadecimal, octal, or binary sequences of digits without a decimal point. They are stored as Number values and are exact up to the safe integer limit (2⁵³ − 1).

**Beginner-Friendly Explanation:** An integer literal is a whole number you write in code — `42`, `0xFF`, `0b1010`.

### Purposes

- To represent whole numbers (counts, indices, IDs).
- To use base-specific representations for clarity.
- To define constants and configuration values.

### Syntax Rules and Structure

```js
42          // decimal integer
0xFF        // hexadecimal integer
0b1010      // binary integer
0o755       // octal integer
```

**Constraints and Limitations:**
- Integers beyond ±(2⁵³ − 1) may lose precision.
- Legacy octal (leading `0`) is forbidden in strict mode.

### Annotated Code Examples

#### Example 1: Integer Literals in Different Bases

```js
const decimal = 255;
const hex = 0xFF;
const binary = 0b11111111;
const octal = 0o377;

console.log(decimal, hex, binary, octal); // all print 255
```

**Expected Output:**
```
255 255 255 255
```

**Why This Output:** All four literals represent the same numeric value, written in different bases.

### Real-World Cases

- **Array indices:** `arr[0]`, `arr[1]`.
- **Loop counters:** `for (let i = 0; i < 10; i++)`.
- **Configuration constants:** `MAX_CONNECTIONS = 100`.

---

## Floating-Point Literals

### Definitions

**Core Definition:** A floating-point literal is a number literal that includes a decimal point and/or an exponent, representing a fractional or very large/small value.

**Technical Definition:** Floating-point literals follow the syntax `DecimalDigits . DecimalDigits? ExponentPart?` or `. DecimalDigits ExponentPart?`, optionally with an exponent (`e`/`E` followed by an optionally signed integer). They are stored as IEEE 754 double-precision values.

**Beginner-Friendly Explanation:** A floating-point literal is a number with a decimal point or an exponent — `3.14`, `0.5`, `1e6`.

### Purposes

- To represent measurements, prices, and scientific values.
- To express very large or very small numbers compactly with exponent notation.
- To perform fractional arithmetic.

### Syntax Rules and Structure

```js
3.14
.5
1.5e3
2.5E-2
```

**Constraints and Limitations:**
- Most decimal fractions cannot be represented exactly in binary; precision is limited.
- `0.1 + 0.2 !== 0.3` due to IEEE 754 representation.

### Annotated Code Examples

#### Example 1: Floating-Point Precision

```js
console.log(0.1 + 0.2);       // 0.30000000000000004
console.log(0.1 + 0.2 === 0.3); // false

// Use EPSILON for comparison
console.log(Math.abs((0.1 + 0.2) - 0.3) < Number.EPSILON); // true
```

**Expected Output:**
```
0.30000000000000004
false
true
```

**Why This Output:** `0.1` and `0.2` cannot be represented exactly in binary floating-point, so their sum is slightly off. `Number.EPSILON` provides a tolerance for approximate equality.

### Real-World Cases

- **Financial calculations:** Use integer cents or a decimal library to avoid floating-point errors.
- **Graphics coordinates:** Positions and transformations use floating-point values.
- **Scientific computing:** Measurements and simulations.

---

## Hexadecimal

### Definitions

**Core Definition:** Hexadecimal is a base-16 numeric representation using digits `0–9` and letters `A–F`, prefixed with `0x` or `0X` in JavaScript.

**Technical Definition:** A hexadecimal literal is `0x` followed by one or more hexadecimal digits. The value is computed as the sum of each digit multiplied by 16 raised to its position.

**Beginner-Friendly Explanation:** Hexadecimal is base 16. It is useful because one hex digit represents exactly four binary bits, making it a compact way to write binary data.

### Purposes

- To represent binary data compactly (four bits per digit).
- To write colour codes, memory addresses, and bitmasks readably.
- To interface with APIs that use hexadecimal notation.

### Syntax Rules and Structure

```js
0xFF        // 255
0x10        // 16
0xdeadbeef  // 3735928559
```

**Constraints and Limitations:**
- Digits must be `0–9` or `A–F` (case-insensitive).
- No decimal point allowed.

### Annotated Code Examples

#### Example 1: Hexadecimal to Decimal

```js
console.log(0xFF);      // 255
console.log(0x10);      // 16
console.log(0x1A);      // 26
console.log(0xdeadbeef); // 3735928559
```

**Expected Output:**
```
255
16
26
3735928559
```

**Why This Output:** Each hexadecimal digit contributes its value times 16 raised to the position from the right.

### Real-World Cases

- **CSS colours:** `#FF0000` (red).
- **Bitwise operations:** Masks like `0xFF`.
- **Cryptography:** Hex digests.

---

## Octal

### Definitions

**Core Definition:** Octal is a base-8 numeric representation using digits `0–7`, prefixed with `0o` or `0O` in modern JavaScript.

**Technical Definition:** The `0o` prefix introduces an octal literal. Legacy octal syntax (leading `0` without `o`) is deprecated and forbidden in strict mode.

**Beginner-Friendly Explanation:** Octal is base 8. It is used in some Unix permissions (like `755`) and older systems.

### Purposes

- To represent file permissions compactly.
- To interface with systems that use octal notation.
- To write binary data in groups of three bits.

### Syntax Rules and Structure

```js
0o755   // 493
0O10    // 8
```

**Constraints and Limitations:**
- Legacy octal (`0644`) causes a SyntaxError in strict mode.
- Digits must be `0–7`.

### Annotated Code Examples

#### Example 1: Octal Literals

```js
console.log(0o755); // 493
console.log(0o10);  // 8
console.log(0o644); // 420
```

**Expected Output:**
```
493
8
420
```

**Why This Output:** Each digit is multiplied by 8 raised to its position.

### Real-World Cases

- **Unix file permissions:** `0o755` (rwxr-xr-x).
- **Legacy systems:** Some embedded and mainframe systems use octal.

---

## Binary

### Definitions

**Core Definition:** Binary is a base-2 numeric representation using digits `0` and `1`, prefixed with `0b` or `0B` in JavaScript.

**Technical Definition:** The `0b` prefix introduces a binary literal. Binary literals are useful for bitwise operations and flags.

**Beginner-Friendly Explanation:** Binary is base 2 — the language of computers. `0b1010` is the binary representation of the decimal number `10`.

### Purposes

- To represent bitmasks and flags clearly.
- To work with binary protocols and hardware.
- To teach and demonstrate binary arithmetic.

### Syntax Rules and Structure

```js
0b1010  // 10
0B1111  // 15
```

**Constraints and Limitations:**
- Only digits `0` and `1` are allowed.
- No decimal point.

### Annotated Code Examples

#### Example 1: Binary to Decimal

```js
console.log(0b1010); // 10
console.log(0B1111); // 15
console.log(0b11111111); // 255
```

**Expected Output:**
```
10
15
255
```

**Why This Output:** Each binary digit represents a power of 2.

### Real-World Cases

- **Permission flags:** `0b100` (read), `0b010` (write), `0b001` (execute).
- **Hardware registers:** Bit-level configuration.
- **Network protocols:** Bit fields in headers.

---

## Exponential Notation

### Definitions

**Core Definition:** Exponential notation expresses a number as a mantissa multiplied by 10 raised to an exponent, written with `e` or `E`.

**Technical Definition:** The exponent part is `e` or `E` followed by an optionally signed decimal integer. `1e3` means 1 × 10³; `2.5e-2` means 2.5 × 10⁻².

**Beginner-Friendly Explanation:** Exponential notation is a shorthand for very large or very small numbers. `1e6` is one million; `1e-6` is one-millionth.

### Purposes

- To write very large or very small numbers compactly.
- To represent scientific measurements.
- To avoid counting zeros.

### Syntax Rules and Structure

```js
1e3      // 1000
1.5e3    // 1500
2.5e-2   // 0.025
1E6      // 1000000
```

**Constraints and Limitations:**
- The exponent must be an integer (no decimal point).
- `e` must be followed by a digit or sign+digit.

### Annotated Code Examples

#### Example 1: Exponential Notation

```js
console.log(1e3);    // 1000
console.log(1.5e3);  // 1500
console.log(2.5e-2); // 0.025
console.log(1E6);    // 1000000
```

**Expected Output:**
```
1000
1500
0.025
1000000
```

**Why This Output:** The mantissa is multiplied by 10 raised to the exponent.

### Real-World Cases

- **Scientific data:** Avogadro's number `6.022e23`.
- **File sizes:** `1e6` bytes (one megabyte).
- **Time intervals:** `1e-3` seconds (one millisecond).

---

## Numeric Separators

### Definitions

**Core Definition:** Numeric separators (`_`) are underscores inserted between digits in numeric literals to improve readability; they are ignored by the parser.

**Technical Definition:** Introduced in ES2021, numeric separators may appear between digits in decimal, hexadecimal, binary, octal, and BigInt literals. They cannot appear at the start or end of a literal, adjacent to a decimal point, or in exponent parts.

**Beginner-Friendly Explanation:** Numeric separators are just underscores you can put inside numbers to make them easier to read, like `1_000_000` instead of `1000000`. JavaScript ignores them.

### Purposes

- To improve readability of large numbers.
- To group digits logically (thousands, bytes, etc.).

### Syntax Rules and Structure

```js
1_000_000
0xFF_FF
0b1010_1010
1_000.000_1
```

**Constraints and Limitations:**
- Cannot be leading (`_1`), trailing (`1_`), or adjacent to a decimal point (`1._0`, `1_.0`).
- Cannot be used in the exponent part (`1e_3` is invalid).

### Annotated Code Examples

#### Example 1: Readable Numbers

```js
console.log(1_000_000);    // 1000000
console.log(0xFF_FF);      // 65535
console.log(0b1010_1010);  // 170
console.log(1_000.000_1);  // 1000.0001
```

**Expected Output:**
```
1000000
65535
170
1000.0001
```

**Why This Output:** The separators are purely cosmetic; the parser ignores them.

### Real-World Cases

- **Large constants:** `1_000_000` for one million.
- **Byte values:** `0xFF_FF` for 65535.
- **Binary masks:** `0b1111_0000`.

---

## `NaN`

### Definitions

**Core Definition:** `NaN` (Not-a-Number) is a special Number value representing the result of an invalid or undefined mathematical operation.

**Technical Definition:** `NaN` is a value of the Number type that is not equal to any value, including itself. It is produced by operations such as `0 / 0`, `Infinity - Infinity`, `Math.sqrt(-1)`, and `parseInt("abc")`. The global `isNaN()` function coerces its argument to a number before testing; `Number.isNaN()` does not.

**Beginner-Friendly Explanation:** `NaN` means "this is not a valid number." It appears when you try to do maths that doesn't make sense, like dividing zero by zero or parsing `"hello"` as a number.

### Purposes

- To signal invalid numeric operations.
- To propagate errors through numeric computations.
- To serve as the return value for failed numeric parsing.

### Syntax Rules and Structure

```js
NaN
Number.NaN
```

**Constraints and Limitations:**
- `NaN !== NaN` is `true`; use `Number.isNaN()` for reliable detection.
- Global `isNaN("hello")` returns `true` because the string is coerced to `NaN`; `Number.isNaN("hello")` returns `false` because the argument is not of type Number.

### Annotated Code Examples

#### Example 1: Producing and Detecting `NaN`

```js
console.log(0 / 0);              // NaN
console.log(parseInt("abc"));    // NaN
console.log(Math.sqrt(-1));      // NaN

console.log(NaN === NaN);        // false
console.log(Number.isNaN(NaN));  // true
console.log(Number.isNaN("abc")); // false
```

**Expected Output:**
```
NaN
NaN
NaN
false
true
false
```

**Why This Output:** Invalid operations produce `NaN`. `NaN` is not equal to itself. `Number.isNaN()` only returns `true` for the actual `NaN` value, not for strings.

### Real-World Cases

- **Input validation:** Checking `Number.isNaN(Number(input))` to reject non-numeric input.
- **Mathematical computations:** Propagating invalid results.
- **Data cleaning:** Filtering out `NaN` values.

---

## `Infinity` and `-Infinity`

### Definitions

**Core Definition:** `Infinity` and `-Infinity` are special Number values representing positive and negative infinity, respectively.

**Technical Definition:** `Infinity` is a property of the global object and `Number.POSITIVE_INFINITY`. `-Infinity` is `Number.NEGATIVE_INFINITY`. They result from overflow (a number exceeding `Number.MAX_VALUE`) or division by zero (with a non-zero numerator). `Infinity` is greater than any finite number; `-Infinity` is less than any finite number.

**Beginner-Friendly Explanation:** `Infinity` is "larger than any number" and `-Infinity` is "smaller than any number." They appear when calculations overflow or when you divide by zero.

### Purposes

- To represent unbounded values in computations.
- To serve as initial values for `Math.min()` and `Math.max()`.
- To signal overflow in numeric operations.

### Syntax Rules and Structure

```js
Infinity
-Infinity
Number.POSITIVE_INFINITY
Number.NEGATIVE_INFINITY
```

**Constraints and Limitations:**
- `Infinity - Infinity` yields `NaN`.
- `0 * Infinity` yields `NaN`.
- `Infinity` is not JSON-serializable (becomes `null`).

### Annotated Code Examples

#### Example 1: Infinity in Practice

```js
console.log(1 / 0);             // Infinity
console.log(-1 / 0);            // -Infinity
console.log(Number.MAX_VALUE * 2); // Infinity

console.log(Infinity > 1e308);  // true
console.log(Infinity - Infinity); // NaN
console.log(Infinity === Infinity); // true
```

**Expected Output:**
```
Infinity
-Infinity
Infinity
true
NaN
true
```

**Why This Output:** Division by zero and overflow produce `Infinity`. `Infinity` is greater than any finite number. Subtracting `Infinity` from itself is undefined (`NaN`).

### Real-World Cases

- **Finding minimum/maximum:** Initialising with `Infinity` / `-Infinity`.
- **Numerical algorithms:** Handling unbounded results.
- **Data validation:** Detecting overflow.

---

## `Number.MAX_VALUE`

### Definitions

**Core Definition:** `Number.MAX_VALUE` is the largest positive finite Number value representable in JavaScript, approximately 1.7976931348623157 × 10³⁰⁸.

**Technical Definition:** `Number.MAX_VALUE` is a static data property of the `Number` object with value approximately `1.7976931348623157e+308`. It is the maximum positive value that can be represented without becoming `Infinity`.

**Beginner-Friendly Explanation:** `Number.MAX_VALUE` is the biggest number JavaScript can handle before it turns into `Infinity`.

### Purposes

- To check for overflow in numeric computations.
- To set upper bounds in algorithms.
- To understand the limits of the Number type.

### Syntax Rules and Structure

```js
Number.MAX_VALUE // 1.7976931348623157e+308
```

**Constraints and Limitations:**
- Values greater than `Number.MAX_VALUE` become `Infinity`.
- Not writable, enumerable, or configurable.

### Annotated Code Examples

#### Example 1: Overflow Detection

```js
console.log(Number.MAX_VALUE);          // 1.7976931348623157e+308
console.log(Number.MAX_VALUE * 2);      // Infinity
console.log(Number.MAX_VALUE + 1);      // 1.7976931348623157e+308 (no change)
```

**Expected Output:**
```
1.7976931348623157e+308
Infinity
1.7976931348623157e+308
```

**Why This Output:** Doubling `MAX_VALUE` overflows to `Infinity`. Adding 1 has no effect because 1 is below the precision threshold at this magnitude.

### Real-World Cases

- **Physics simulations:** Checking whether values exceed representable limits.
- **Financial models:** Guarding against overflow.

---

## `Number.MIN_VALUE`

### Definitions

**Core Definition:** `Number.MIN_VALUE` is the smallest positive Number value greater than zero, approximately 5 × 10⁻³²⁴.

**Technical Definition:** `Number.MIN_VALUE` is the smallest positive value representable in the Number type, approximately `5e-324`. It is not the most negative number; it is the closest positive number to zero.

**Beginner-Friendly Explanation:** `Number.MIN_VALUE` is the tiniest positive number JavaScript can represent — much smaller than `Number.EPSILON`.

### Purposes

- To understand the lower limit of positive representable numbers.
- To detect underflow in numeric computations.
- To set lower bounds for positive quantities.

### Syntax Rules and Structure

```js
Number.MIN_VALUE // 5e-324
```

**Constraints and Limitations:**
- Values smaller than `MIN_VALUE` underflow to zero.
- Not the same as `-Number.MAX_VALUE` (the most negative finite value).

### Annotated Code Examples

#### Example 1: Underflow

```js
console.log(Number.MIN_VALUE);      // 5e-324
console.log(Number.MIN_VALUE / 2);  // 0 (underflow)
```

**Expected Output:**
```
5e-324
0
```

**Why This Output:** Dividing the smallest positive number by 2 underflows to zero.

### Real-World Cases

- **Numerical analysis:** Detecting underflow in iterative algorithms.
- **Probability calculations:** Very small probabilities.

---

## `Number.MAX_SAFE_INTEGER`

### Definitions

**Core Definition:** `Number.MAX_SAFE_INTEGER` is the largest integer that can be represented exactly and compared safely in JavaScript, equal to 2⁵³ − 1 (9,007,199,254,740,991).

**Technical Definition:** `Number.MAX_SAFE_INTEGER` is a static data property with value `9007199254740991` (2⁵³ − 1). Integers up to this value can be represented exactly; beyond this, adjacent integers may become indistinguishable.

**Beginner-Friendly Explanation:** This is the largest whole number JavaScript can handle without losing precision. Beyond it, `9007199254740992 === 9007199254740993` is `true` because both are stored as the same value.

### Purposes

- To check whether an integer is within the safe range.
- To guard against precision loss in ID handling.
- To decide when to use BigInt instead.

### Syntax Rules and Structure

```js
Number.MAX_SAFE_INTEGER // 9007199254740991
```

### Annotated Code Examples

#### Example 1: Safe Integer Boundary

```js
console.log(Number.MAX_SAFE_INTEGER);               // 9007199254740991
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER)); // true
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1)); // false
```

**Expected Output:**
```
9007199254740991
true
false
```

**Why This Output:** `MAX_SAFE_INTEGER` itself is safe; adding 1 produces an integer outside the safe range.

### Real-World Cases

- **Database IDs:** Twitter/X snowflake IDs exceed this limit; use BigInt or strings.
- **Financial calculations:** Ensuring exact integer arithmetic.
- **Hash codes:** Checking whether a computed hash is safe.

---

## `Number.MIN_SAFE_INTEGER`

### Definitions

**Core Definition:** `Number.MIN_SAFE_INTEGER` is the smallest integer that can be represented exactly and compared safely in JavaScript, equal to −(2⁵³ − 1).

**Technical Definition:** `Number.MIN_SAFE_INTEGER` is `-9007199254740991`. It is the negative counterpart of `MAX_SAFE_INTEGER`.

**Beginner-Friendly Explanation:** This is the most negative whole number JavaScript can handle without losing precision.

### Purposes

- To check whether negative integers are within the safe range.
- To guard against precision loss in negative values.

### Syntax Rules and Structure

```js
Number.MIN_SAFE_INTEGER // -9007199254740991
```

### Annotated Code Examples

#### Example 1: Negative Safe Integer Boundary

```js
console.log(Number.MIN_SAFE_INTEGER);                    // -9007199254740991
console.log(Number.isSafeInteger(Number.MIN_SAFE_INTEGER)); // true
console.log(Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1)); // false
```

**Expected Output:**
```
-9007199254740991
true
false
```

**Why This Output:** The boundary itself is safe; going one step further leaves the safe range.

### Real-World Cases

- **Negative account balances:** Checking precision limits.
- **Coordinate systems:** Ensuring exact negative integers.

---

## Floating-Point Precision

### Definitions

**Core Definition:** Floating-point precision refers to the limited accuracy with which decimal fractions and very large integers can be represented in binary floating-point format, leading to rounding errors.

**Technical Definition:** The IEEE 754 binary64 format allocates 53 bits of precision (52 stored bits plus an implicit leading 1). Many decimal fractions (such as `0.1`, `0.2`, `0.3`) cannot be represented exactly in binary, so arithmetic results may differ from the mathematically exact value. `Number.EPSILON` (2⁻⁵²) is the difference between 1 and the next representable number and serves as a common tolerance for approximate comparison.

**Beginner-Friendly Explanation:** Computers store numbers in binary, and some decimals — like `0.1` — cannot be stored perfectly. This is why `0.1 + 0.2` gives `0.30000000000000004` instead of `0.3`. It is not a JavaScript bug; it is a fundamental limitation of binary floating-point.

### Purposes

- To understand why decimal arithmetic may produce unexpected results.
- To implement approximate comparison with `Number.EPSILON`.
- To decide when to use integer arithmetic or BigInt.

### Syntax Rules and Structure

There is no dedicated syntax; the behaviour emerges from IEEE 754 representation. The standard workaround is:

```js
Math.abs(a - b) < Number.EPSILON
```

**Constraints and Limitations:**
- No finite binary format can represent all real numbers exactly.
- Precision decreases for very large numbers (the exponent grows while the mantissa stays fixed).
- `Number.EPSILON` is a reasonable tolerance for values near 1 but may not be appropriate for very large or very small numbers.

### Annotated Code Examples

#### Example 1: The Classic `0.1 + 0.2` Problem

```js
console.log(0.1 + 0.2);          // 0.30000000000000004
console.log(0.1 + 0.2 === 0.3);  // false

// Approximate comparison with EPSILON
const a = 0.1 + 0.2;
const b = 0.3;
console.log(Math.abs(a - b) < Number.EPSILON); // true
```

**Expected Output:**
```
0.30000000000000004
false
true
```

**Why This Output:** `0.1` and `0.2` have non-terminating binary expansions, so their sum is not exactly `0.3`. `Number.EPSILON` provides a tolerance within which the values are considered equal.

### Real-World Cases

- **Financial software:** Using integer cents or decimal libraries to avoid rounding errors.
- **Graphics:** Accepting small errors in floating-point coordinates.
- **Testing:** Using `toBeCloseTo()` in Jest for floating-point assertions.

---

## `Number.isNaN()`

### Definitions

**Core Definition:** `Number.isNaN()` determines whether a value is the special `NaN` value, without coercing the argument.

**Technical Definition:** `Number.isNaN(value)` returns `true` only if `value` is of type Number and is `NaN`. Unlike the global `isNaN()`, it does not convert strings or other values to numbers.

**Beginner-Friendly Explanation:** `Number.isNaN()` is the safe way to check for `NaN`. It returns `true` only for the actual `NaN` value, not for things like the string `"hello"`.

### Purposes

- To reliably detect `NaN` without coercion surprises.
- To validate that a numeric operation produced a valid result.
- To replace the global `isNaN()` in modern code.

### Syntax Rules and Structure

```js
Number.isNaN(value)
```

**Constraints and Limitations:**
- Returns `false` for non-Number arguments (including `"NaN"`, `undefined`, and objects).
- Does not coerce.

### Annotated Code Examples

#### Example 1: `Number.isNaN()` vs. Global `isNaN()`

```js
console.log(Number.isNaN(NaN));      // true
console.log(Number.isNaN("hello"));  // false
console.log(Number.isNaN(undefined));// false

console.log(isNaN("hello"));         // true (global coerces)
console.log(Number.isNaN(0 / 0));    // true
```

**Expected Output:**
```
true
false
false
true
true
```

**Why This Output:** `Number.isNaN()` only returns `true` for the `NaN` value itself; the global `isNaN()` coerces its argument first.

### Real-World Cases

- **Input validation:** `Number.isNaN(Number(input))` to reject non-numeric input.
- **Calculation guards:** Checking results before using them.

---

## `Number.isFinite()`

### Definitions

**Core Definition:** `Number.isFinite()` determines whether a value is a finite number (not `Infinity`, `-Infinity`, or `NaN`), without coercing the argument.

**Technical Definition:** `Number.isFinite(value)` returns `true` only if `value` is of type Number and is a finite number. Non-number arguments return `false`.

**Beginner-Friendly Explanation:** `Number.isFinite()` checks whether a value is a "normal" number — not infinite and not `NaN`. It does not try to convert strings.

### Purposes

- To validate that a value is a finite number.
- To guard against `Infinity` and `NaN` in calculations.
- To replace the coercing global `isFinite()`.

### Syntax Rules and Structure

```js
Number.isFinite(value)
```

**Constraints and Limitations:**
- Returns `false` for non-number arguments.
- `Number.isFinite(Infinity)` is `false`.

### Annotated Code Examples

#### Example 1: Finite Number Check

```js
console.log(Number.isFinite(42));        // true
console.log(Number.isFinite(Infinity));  // false
console.log(Number.isFinite(NaN));       // false
console.log(Number.isFinite("42"));      // false (no coercion)
```

**Expected Output:**
```
true
false
false
false
```

**Why This Output:** Only finite Number values return `true`; strings and special values return `false`.

### Real-World Cases

- **API validation:** Ensuring numeric fields are finite.
- **Financial calculations:** Rejecting infinite or `NaN` results.

---

## `Number.isInteger()`

### Definitions

**Core Definition:** `Number.isInteger()` determines whether a value is an integer (a whole number without a fractional part).

**Technical Definition:** `Number.isInteger(value)` returns `true` if `value` is of type Number and its value is an integer (including `0`, negative integers, and integers beyond the safe range, provided they have no fractional part).

**Beginner-Friendly Explanation:** `Number.isInteger()` checks whether a number is whole — like `42` or `-7`, but not `3.14`.

### Purposes

- To validate integer input.
- To distinguish whole numbers from decimals.
- To guard array indexing and loop counters.

### Syntax Rules and Structure

```js
Number.isInteger(value)
```

**Constraints and Limitations:**
- Returns `false` for non-number arguments.
- Returns `true` for integers beyond the safe range (even if imprecise), as long as the value is integral.

### Annotated Code Examples

#### Example 1: Integer Check

```js
console.log(Number.isInteger(42));    // true
console.log(Number.isInteger(3.14));  // false
console.log(Number.isInteger(-7));    // true
console.log(Number.isInteger("42"));  // false
```

**Expected Output:**
```
true
false
true
false
```

**Why This Output:** Only whole Number values return `true`; decimals and strings return `false`.

### Real-World Cases

- **Pagination:** Validating page numbers.
- **Array indices:** Ensuring integer indices.
- **Form validation:** Accepting only whole-number input.

---

## `Number.isSafeInteger()`

### Definitions

**Core Definition:** `Number.isSafeInteger()` determines whether a value is an integer within the safe range (±(2⁵³ − 1)).

**Technical Definition:** `Number.isSafeInteger(value)` returns `true` if `value` is an integer and its absolute value is less than or equal to `Number.MAX_SAFE_INTEGER`.

**Beginner-Friendly Explanation:** `Number.isSafeInteger()` checks whether a whole number is small enough to be represented exactly without precision loss.

### Purposes

- To validate that integer arithmetic will be exact.
- To decide whether BigInt is needed.
- To guard IDs and large counters.

### Syntax Rules and Structure

```js
Number.isSafeInteger(value)
```

### Annotated Code Examples

#### Example 1: Safe Integer Check

```js
console.log(Number.isSafeInteger(42));                    // true
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER)); // true
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1)); // false
console.log(Number.isSafeInteger(3.14));                  // false
```

**Expected Output:**
```
true
true
false
false
```

**Why This Output:** The boundary itself is safe; exceeding it or providing a non-integer returns `false`.

### Real-World Cases

- **Large IDs:** Checking whether an ID fits in a Number.
- **Cryptographic counters:** Ensuring exact arithmetic.
- **Financial ledgers:** Validating that integer amounts are safe.

---

## `Number.parseInt()`

### Definitions

**Core Definition:** `Number.parseInt()` is the same function as the global `parseInt()`, attached to the `Number` object for consistency.

**Technical Definition:** `Number.parseInt(string, radix)` parses a string argument and returns an integer of the specified radix (base). It stops at the first invalid character and returns `NaN` if no valid digit is found.

**Beginner-Friendly Explanation:** `Number.parseInt()` extracts a whole number from the start of a string. It is identical to the global `parseInt()`.

### Purposes

- To parse integers from strings.
- To convert strings in a specific base to decimal.
- To maintain consistency by using the `Number` namespace.

### Syntax Rules and Structure

```js
Number.parseInt(string, radix)
```

**Constraints and Limitations:**
- Always specify the radix (`10`) for predictable behaviour.
- Stops at the first non-digit character.

### Annotated Code Examples

#### Example 1: Parsing Integers

```js
console.log(Number.parseInt("42", 10));    // 42
console.log(Number.parseInt("42px", 10));  // 42
console.log(Number.parseInt("FF", 16));    // 255
console.log(Number.parseInt("hello", 10)); // NaN
```

**Expected Output:**
```
42
42
255
NaN
```

**Why This Output:** `parseInt` reads valid digits and stops at invalid characters; the radix determines the base.

### Real-World Cases

- **CSS value parsing:** `Number.parseInt("16px", 10)`.
- **URL parameters:** Extracting integer IDs.
- **Hexadecimal input:** Converting colour codes.

---

## `Number.parseFloat()`

### Definitions

**Core Definition:** `Number.parseFloat()` is the same function as the global `parseFloat()`, attached to the `Number` object for consistency.

**Technical Definition:** `Number.parseFloat(string)` parses a string argument and returns a floating-point number. It stops at the first invalid character.

**Beginner-Friendly Explanation:** `Number.parseFloat()` extracts a decimal number from the start of a string. It is identical to the global `parseFloat()`.

### Purposes

- To parse floating-point numbers from strings.
- To handle numeric input with trailing units.
- To maintain consistency by using the `Number` namespace.

### Syntax Rules and Structure

```js
Number.parseFloat(string)
```

**Constraints and Limitations:**
- Does not accept a radix.
- Stops at the first invalid character.

### Annotated Code Examples

#### Example 1: Parsing Floats

```js
console.log(Number.parseFloat("3.14"));      // 3.14
console.log(Number.parseFloat("3.14abc"));   // 3.14
console.log(Number.parseFloat("1e3"));       // 1000
console.log(Number.parseFloat("abc"));       // NaN
```

**Expected Output:**
```
3.14
3.14
1000
NaN
```

**Why This Output:** `parseFloat` reads valid floating-point syntax and stops at invalid characters.

### Real-World Cases

- **Measurements:** `Number.parseFloat("3.5kg")` → `3.5`.
- **Financial input:** Parsing decimal amounts.
- **Scientific data:** Parsing exponent notation.

---

## Math Object

### Definitions

**Core Definition:** The `Math` object is a built-in namespace containing static properties and methods for mathematical constants and functions.

**Technical Definition:** `Math` is not a constructor; all its properties and methods are static. It works with the Number type but not BigInt. Many `Math` functions are implementation-dependent in precision.

**Beginner-Friendly Explanation:** `Math` is a toolbox of mathematical helpers — rounding, powers, square roots, random numbers, and constants like π and e.

### Purposes

- To perform common mathematical operations without writing algorithms manually.
- To access mathematical constants.
- To generate random numbers.
- To round numbers in various ways.

### Syntax Rules and Structure

```js
Math.method(value)
Math.CONSTANT
```

**Constraints and Limitations:**
- Cannot be instantiated with `new`.
- Does not work with BigInt.
- Precision is implementation-dependent for many functions.

### Sub-features

#### `Math.round()`

**Definitions:**
- **Core:** Rounds a number to the nearest integer.
- **Technical:** Returns the integer closest to the argument; if the fractional part is exactly 0.5, rounds toward +∞.
- **Beginner:** Rounds to the nearest whole number (0.5 goes up).

**Example:**
```js
console.log(Math.round(4.4)); // 4
console.log(Math.round(4.5)); // 5
console.log(Math.round(-4.5)); // -4
```

**Expected Output:**
```
4
5
-4
```

---

#### `Math.floor()`

**Definitions:**
- **Core:** Rounds a number down to the nearest integer.
- **Technical:** Returns the largest integer less than or equal to the argument.
- **Beginner:** Always rounds down (toward −∞).

**Example:**
```js
console.log(Math.floor(4.9)); // 4
console.log(Math.floor(-4.1)); // -5
```

**Expected Output:**
```
4
-5
```

---

#### `Math.ceil()`

**Definitions:**
- **Core:** Rounds a number up to the nearest integer.
- **Technical:** Returns the smallest integer greater than or equal to the argument.
- **Beginner:** Always rounds up (toward +∞).

**Example:**
```js
console.log(Math.ceil(4.1)); // 5
console.log(Math.ceil(-4.9)); // -4
```

**Expected Output:**
```
5
-4
```

---

#### `Math.trunc()`

**Definitions:**
- **Core:** Removes the fractional part of a number, returning only the integer part.
- **Technical:** Returns the integer part by discarding the fractional digits (no rounding).
- **Beginner:** Chops off the decimal part.

**Example:**
```js
console.log(Math.trunc(4.9)); // 4
console.log(Math.trunc(-4.9)); // -4
```

**Expected Output:**
```
4
-4
```

---

#### `Math.abs()`

**Definitions:**
- **Core:** Returns the absolute value of a number.
- **Technical:** Returns the non-negative magnitude of the argument.
- **Beginner:** Removes the minus sign.

**Example:**
```js
console.log(Math.abs(-5)); // 5
console.log(Math.abs(5)); // 5
```

**Expected Output:**
```
5
5
```

---

#### `Math.max()` and `Math.min()`

**Definitions:**
- **Core:** Return the largest or smallest of zero or more numbers.
- **Technical:** `Math.max(...values)` returns the maximum; `Math.min(...values)` returns the minimum. With no arguments, `max` returns `-Infinity` and `min` returns `Infinity`.
- **Beginner:** Find the biggest or smallest number in a list.

**Example:**
```js
console.log(Math.max(1, 5, 3)); // 5
console.log(Math.min(1, 5, 3)); // 1
console.log(Math.max()); // -Infinity
console.log(Math.min()); // Infinity
```

**Expected Output:**
```
5
1
-Infinity
Infinity
```

---

#### `Math.random()`

**Definitions:**
- **Core:** Returns a pseudo-random floating-point number in the range [0, 1).
- **Technical:** Returns a Number with a positive sign, greater than or equal to 0 but less than 1, chosen randomly or pseudo-randomly.
- **Beginner:** Generates a random decimal between 0 (inclusive) and 1 (exclusive).

**Example:**
```js
const random = Math.random(); // e.g., 0.7234...
console.log(random >= 0 && random < 1); // true
```

**Expected Output:**
```
true
```

---

#### `Math.pow()`

**Definitions:**
- **Core:** Returns the base raised to the power of the exponent.
- **Technical:** `Math.pow(base, exponent)` computes base^exponent. Equivalent to the `**` operator.
- **Beginner:** Calculates powers (e.g., 2³ = 8).

**Example:**
```js
console.log(Math.pow(2, 3)); // 8
console.log(Math.pow(9, 0.5)); // 3 (square root)
```

**Expected Output:**
```
8
3
```

---

#### `Math.sqrt()`

**Definitions:**
- **Core:** Returns the square root of a number.
- **Technical:** Computes √x. Returns `NaN` for negative arguments.
- **Beginner:** Calculates the square root.

**Example:**
```js
console.log(Math.sqrt(9)); // 3
console.log(Math.sqrt(2)); // 1.4142135623730951
console.log(Math.sqrt(-1)); // NaN
```

**Expected Output:**
```
3
1.4142135623730951
NaN
```

---

#### `Math.PI`

**Definitions:**
- **Core:** The ratio of a circle's circumference to its diameter, approximately 3.14159.
- **Technical:** `Math.PI` is a static property with value approximately 3.141592653589793.
- **Beginner:** The number π, used in circle calculations.

**Example:**
```js
console.log(Math.PI); // 3.141592653589793
console.log(2 * Math.PI * 5); // circumference of radius 5
```

**Expected Output:**
```
3.141592653589793
31.41592653589793
```

---

#### `Math.E`

**Definitions:**
- **Core:** Euler's number, the base of natural logarithms, approximately 2.71828.
- **Technical:** `Math.E` is a static property with value approximately 2.718281828459045.
- **Beginner:** The number e, used in exponential growth and logarithms.

**Example:**
```js
console.log(Math.E); // 2.718281828459045
```

**Expected Output:**
```
2.718281828459045
```

---

### Real-World Cases (Math Object)

- **Game development:** Random enemy spawns (`Math.random()`), distance calculations (`Math.sqrt()`), rounding positions (`Math.floor()`).
- **Data visualisation:** Scaling axes (`Math.max()`/`Math.min()`), trigonometric functions (not covered here but part of `Math`).
- **Finance:** Rounding currency (`Math.round()`, `Math.floor()`).
- **Geometry:** Area and circumference calculations with `Math.PI`.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Numbers — https://tc39.es/ecma262/
- MDN Web Docs – Numbers and strings — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Numbers_and_strings
- MDN Web Docs – Number — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number
- MDN Web Docs – Number.EPSILON — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/EPSILON
- MDN Web Docs – Math — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math
- MDN Web Docs – NaN — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/NaN
- MDN Web Docs – Infinity — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Infinity
- MDN Web Docs – Lexical grammar (numeric literals) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#numeric_literals
- MDN Web Docs – Numeric separators — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#numeric_separators
- W3Schools – JavaScript Number Properties — https://www.w3schools.com/js/js_number_properties.asp
- You Don't Know JS: Types & Grammar – Chapter 2: Values (Numbers) — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/types-grammar/ch2.md