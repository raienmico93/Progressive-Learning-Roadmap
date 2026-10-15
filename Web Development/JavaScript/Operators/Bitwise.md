# JavaScript Bitwise Operators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Bitwise operators are JavaScript operators that manipulate individual bits (binary digits) of integer values at the binary level, performing logical operations, shifts, and inversions on 32-bit signed integers (or BigInt values).

**Technical Definition:** Per ECMAScript 2025 (ECMA-262), bitwise operators are specified across several grammar productions. The unary bitwise NOT (`~`) is defined in §13.5.8, the binary bitwise operators (`&`, `|`, `^`) in §13.12, and the bitwise shift operators (`<<`, `>>`, `>>>`) in §13.9. For Number operands, bitwise operations use the `Number::bitwiseNOT`, `Number::bitwiseAND`, `Number::bitwiseOR`, `Number::bitwiseXOR`, and `Number::leftShift`/`Number::signedRightShift`/`Number::unsignedRightShift` abstract operations. Operands are first converted to 32-bit signed integers (via `ToInt32` or `ToUint32`) in two's complement format, the operation is performed bit-by-bit, and the result is converted back to a JavaScript Number. For BigInt operands, operations are performed on arbitrary-precision integers (except `>>>`, which is not supported for BigInt).

**Beginner-Friendly Explanation:** Computers store numbers as sequences of 0s and 1s called bits. Bitwise operators let you work directly with those individual bits — flipping them, shifting them left or right, or combining two numbers bit by bit. They are extremely fast and are commonly used for things like flags, permissions, colour manipulation, and performance-critical code.

### Key Characteristics

- **32-bit signed integers:** Number operands are converted to 32-bit signed integers in two's complement format before the operation.
- **BigInt support:** Most bitwise operators work with BigInt; `>>>` is the only exception.
- **Two's complement:** Negative numbers are represented using two's complement.
- **Bit positions:** Bits are numbered 0 (least significant) to 31 (most significant).
- **Overflow truncation:** Numbers exceeding 32 bits have their most significant bits discarded.
- **Results as Numbers:** For Number operands, the result is always a signed 32-bit integer converted back to a JavaScript Number.

### Prerequisites

- Basic JavaScript syntax (variables, expressions, data types).
- Understanding of binary number representation.
- Familiarity with two's complement for negative numbers.
- Awareness of operator precedence.

### Related Programming Areas

- **Systems programming:** Flags, permissions, hardware registers.
- **Graphics programming:** Colour manipulation (RGB/hex conversion).
- **Game development:** Collision layers, state flags.
- **Data compression:** Packing multiple values into a single integer.
- **Cryptography:** Low-level bit manipulation in hash functions.

### Core Concepts / Features

The following sections cover each core concept and feature, each using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## `&` (Bitwise AND)

### Definitions

**Core Definition:** The `&` operator returns a number whose binary representation has a `1` in each bit position where both operands have a `1`.

**Technical Definition:** Per ECMA-262 §13.12.1, the production `BitwiseANDExpression : BitwiseANDExpression & EqualityExpression` evaluates both operands to Numeric, applies `Number::bitwiseAND` (or `BigInt::bitwiseAND`), and returns the result. The operation uses the truth table: `0 & 0 = 0`, `0 & 1 = 0`, `1 & 0 = 0`, `1 & 1 = 1`.

**Beginner-Friendly Explanation:** `&` compares two numbers bit by bit. If both numbers have a `1` in the same position, the result has a `1` there; otherwise, it's `0`. It's like asking "are both bits on?"

### Purposes

- To test whether specific bits are set (bit masking).
- To clear specific bits (masking out).
- To check if a number is even or odd.
- To combine multiple boolean flags.

### Syntax Rules and Structure

```js
operand1 & operand2
```

**Component Breakdown:**
- `operand1`, `operand2`: Numbers or BigInts.
- Returns: A Number (32-bit signed) or BigInt.

**Syntax Rules:**
- Operands are converted to 32-bit signed integers (Number) or kept as BigInt.
- Mixing BigInt and Number throws `TypeError`.
- Bitwise ANDing any number `x` with `-1` returns `x` converted to a 32-bit integer.

**Constraints and Limitations:**
- Numbers exceeding 32 bits lose their most significant bits.
- Not a logical AND (`&&`); does not short-circuit.

### Annotated Code Examples

#### Example 1: Basic Bitwise AND

```js
// Step 1: 9 in binary = 00000000000000000000000000001001
// Step 2: 14 in binary = 00000000000000000000000000001110
// Step 3: Apply AND bit by bit.
console.log(9 & 14); // 8 (binary 00001000)
```

**Expected Output:**
```
8
```

**Why This Output:** The only bit position where both numbers have a `1` is bit 3 (value 8). All other positions produce `0`.

#### Example 2: Even/Odd Check

```js
function isEven(n) {
  return (n & 1) === 0;
}

console.log(isEven(4)); // true
console.log(isEven(7)); // false
```

**Expected Output:**
```
true
false
```

**Why This Output:** The least significant bit (bit 0) determines parity. If it's `0`, the number is even. `4 & 1 = 0`, `7 & 1 = 1`.

#### Example 3: Bit Masking

```js
// Step 1: Check if bit 2 is set.
const flags = 0b1010; // 10
const BIT_2 = 0b0100; // 4

console.log((flags & BIT_2) !== 0); // true (bit 2 is set)

// Step 2: Clear bit 3.
const CLEAR_BIT_3 = ~(1 << 3);
console.log(flags & CLEAR_BIT_3); // 2 (bit 3 cleared)
```

**Expected Output:**
```
true
2
```

**Why This Output:** `flags & BIT_2` isolates bit 2, which is `1`. Clearing bit 3 uses a mask where bit 3 is `0`.

### Real-World Cases

- **Permission systems:** Checking if a user has a specific permission flag.
- **Colour extraction:** Masking out the red, green, or blue channel from an RGB value.
- **Feature flags:** Testing whether a feature is enabled.

---

## `|` (Bitwise OR)

### Definitions

**Core Definition:** The `|` operator returns a number whose binary representation has a `1` in each bit position where at least one operand has a `1`.

**Technical Definition:** Per ECMA-262 §13.12.2, `BitwiseORExpression : BitwiseORExpression | BitwiseXORExpression` evaluates both operands to Numeric and applies `Number::bitwiseOR` (or `BigInt::bitwiseOR`). Truth table: `0 | 0 = 0`, `0 | 1 = 1`, `1 | 0 = 1`, `1 | 1 = 1`.

**Beginner-Friendly Explanation:** `|` sets a bit to `1` if either of the two numbers has a `1` in that position. It's like asking "is at least one of these bits on?"

### Purposes

- To set specific bits (bit masking).
- To combine multiple flags into a single value.
- To initialise flags with defaults.

### Syntax Rules and Structure

```js
operand1 | operand2
```

**Syntax Rules:**
- Same conversion rules as `&`.
- Bitwise ORing any number `x` with `0` returns `x` converted to a 32-bit integer.

### Annotated Code Examples

#### Example 1: Basic Bitwise OR

```js
// 9  = 1001
// 14 = 1110
// OR = 1111 = 15
console.log(9 | 14); // 15
```

**Expected Output:**
```
15
```

**Why This Output:** Any position where either number has a `1` produces a `1`.

#### Example 2: Setting Flags

```js
const READ = 0b001;    // 1
const WRITE = 0b010;   // 2
const EXECUTE = 0b100; // 4

// Step 1: Combine permissions.
let permissions = READ | WRITE;
console.log(permissions); // 3 (binary 011)

// Step 2: Add execute permission.
permissions |= EXECUTE;
console.log(permissions); // 7 (binary 111)
```

**Expected Output:**
```
3
7
```

**Why This Output:** `|` combines bits, setting all specified flag positions to `1`.

### Real-World Cases

- **File permissions:** Combining read, write, and execute flags.
- **UI state:** Combining multiple boolean states into a single integer.
- **Graphics:** Setting individual colour channels.

---

## `^` (Bitwise XOR)

### Definitions

**Core Definition:** The `^` operator returns a number whose binary representation has a `1` in each bit position where exactly one operand has a `1`.

**Technical Definition:** Per ECMA-262 §13.12.3, `BitwiseXORExpression : BitwiseXORExpression ^ BitwiseANDExpression` applies `Number::bitwiseXOR` (or `BigInt::bitwiseXOR`). Truth table: `0 ^ 0 = 0`, `0 ^ 1 = 1`, `1 ^ 0 = 1`, `1 ^ 1 = 0`.

**Beginner-Friendly Explanation:** `^` sets a bit to `1` if the two numbers have different bits in that position. If they're the same, the result is `0`. It's like asking "are these bits different?"

### Purposes

- To toggle specific bits.
- To swap two variables without a temporary variable.
- To implement simple encryption (XOR cipher).
- To check if two values are different.

### Syntax Rules and Structure

```js
operand1 ^ operand2
```

**Syntax Rules:**
- Same conversion rules as `&` and `|`.
- `x ^ 0` returns `x` converted to a 32-bit integer.
- `x ^ x` returns `0`.
- XOR is commutative and associative.

### Annotated Code Examples

#### Example 1: Basic XOR

```js
// 9  = 1001
// 14 = 1110
// XOR = 0111 = 7
console.log(9 ^ 14); // 7
```

**Expected Output:**
```
7
```

**Why This Output:** Positions where bits differ produce `1`; positions where they match produce `0`.

#### Example 2: Swapping Variables

```js
let a = 5, b = 9;

// Step 1: XOR swap.
a = a ^ b; // a = 5 ^ 9 = 12
b = a ^ b; // b = 12 ^ 9 = 5
a = a ^ b; // a = 12 ^ 5 = 9

console.log(a, b); // 9 5
```

**Expected Output:**
```
9 5
```

**Why This Output:** The XOR swap algorithm works because `x ^ y ^ y = x`. No temporary variable is needed.

#### Example 3: Toggling a Bit

```js
let flags = 0b1010; // 10
const TOGGLE = 0b0010; // 2

console.log(flags ^ TOGGLE); // 8 (bit 1 toggled off)
console.log(8 ^ TOGGLE);     // 10 (bit 1 toggled back on)
```

**Expected Output:**
```
8
10
```

**Why This Output:** XORing with a mask toggles the bits in the mask positions.

### Real-World Cases

- **Simple encryption:** XOR with a key for obfuscation.
- **Error detection:** XOR checksums.
- **Toggle switches:** UI state toggling.

---

## `~` (Bitwise NOT)

### Definitions

**Core Definition:** The `~` operator is a unary operator that inverts every bit of its operand.

**Technical Definition:** Per ECMA-262 §13.5.8, `UnaryExpression : ~ UnaryExpression` evaluates the operand to Numeric, applies `Number::bitwiseNOT` (or `BigInt::bitwiseNOT`), and returns the result. For 32-bit integers, `~x` is equivalent to `-(x + 1)`.

**Beginner-Friendly Explanation:** `~` flips every bit: `0` becomes `1` and `1` becomes `0`. The result of `~x` is always `-(x + 1)`.

### Purposes

- To create a bitmask that clears specific bits.
- To compute two's complement.
- To convert a value to its complement.

### Syntax Rules and Structure

```js
~operand
```

**Syntax Rules:**
- Unary operator; takes one operand.
- `~x` for 32-bit integers equals `-(x + 1)`.
- `~x` for BigInt equals `-(x + 1n)`.

### Annotated Code Examples

#### Example 1: Basic NOT

```js
console.log(~170); // -171
console.log(~0);   // -1
console.log(~-1);  // 0
console.log(~5n);  // -6n (BigInt)
```

**Expected Output:**
```
-171
-1
0
-6n
```

**Why This Output:** `~170` flips all bits of `170`, which in two's complement is `-171`. `~0` flips all zeros to ones, which is `-1`.

#### Example 2: Clearing Bits with NOT

```js
const flags = 0b1111; // 15

// Step 1: Create a mask with bit 1 cleared.
const CLEAR_BIT_1 = ~(1 << 1); // ~0b0010 = 0b...1101

// Step 2: Apply the mask.
console.log(flags & CLEAR_BIT_1); // 13 (binary 1101)
```

**Expected Output:**
```
13
```

**Why This Output:** `~(1 << 1)` creates a mask with all bits set except bit 1, clearing that bit when ANDed.

### Real-World Cases

- **Bit clearing:** Creating masks to remove specific bits.
- **Two's complement:** Understanding negative number representation.
- **Double NOT for truncation:** `~~x` truncates `x` toward zero (equivalent to `Math.trunc` for numbers in range).

---

## `<<` (Left Shift)

### Definitions

**Core Definition:** The `<<` operator shifts the bits of the left operand to the left by the number of positions specified by the right operand, filling vacated positions with zeros.

**Technical Definition:** Per ECMA-262 §13.9.1, `ShiftExpression : ShiftExpression << AdditiveExpression` evaluates the left operand to Numeric, evaluates the right operand to a shift count (masked to 5 bits for 32-bit operations, i.e., `shiftCount & 0x1F`), and performs `Number::leftShift` (or `BigInt::leftShift`). For 32-bit integers, the result is `(leftOperand << (shiftCount & 31))` with bits shifted off the left discarded.

**Beginner-Friendly Explanation:** `<<` moves all the bits to the left. New bits on the right are `0`. Shifting left by `n` is like multiplying by `2^n`.

### Purposes

- To multiply by powers of two quickly.
- To create bit masks by positioning a `1` at a specific bit.
- To pack values into a larger integer.

### Syntax Rules and Structure

```js
operand1 << operand2
```

**Component Breakdown:**
- `operand1`: The value to shift.
- `operand2`: The number of positions to shift (masked to 5 bits for Numbers).
- Returns: A 32-bit signed integer (or BigInt).

**Syntax Rules:**
- Shift count is masked to 5 bits: only the lowest 5 bits are used (0–31).
- Shifting by 0 returns the original value converted to 32-bit.
- For BigInt, there is no 5-bit masking; the shift amount is the full BigInt value.

### Annotated Code Examples

#### Example 1: Multiplication by Powers of Two

```js
console.log(5 << 1);  // 10 (5 * 2)
console.log(5 << 2);  // 20 (5 * 4)
console.log(5 << 3);  // 40 (5 * 8)
console.log(1 << 4);  // 16 (bit 4 set)
```

**Expected Output:**
```
10
20
40
16
```

**Why This Output:** Left shifting by `n` multiplies by `2^n`. `1 << 4` creates a mask with only bit 4 set.

#### Example 2: Creating Bit Masks

```js
const BIT_0 = 1 << 0; // 1
const BIT_1 = 1 << 1; // 2
const BIT_2 = 1 << 2; // 4
const BIT_3 = 1 << 3; // 8

console.log(BIT_0, BIT_1, BIT_2, BIT_3); // 1 2 4 8

// Combine flags.
const ALL = (1 << 4) - 1; // 15 (bits 0-3)
console.log(ALL); // 15
```

**Expected Output:**
```
1 2 4 8
15
```

**Why This Output:** `1 << n` creates a mask with only bit `n` set. `(1 << 4) - 1` creates a mask with bits 0–3 set.

#### Example 3: RGB to Hex Conversion

```js
function rgbToHex(r, g, b) {
  return "#" + ((1 << 24) | (r << 16) | (g << 8) | b).toString(16).slice(1).toUpperCase();
}

console.log(rgbToHex(255, 128, 0)); // "#FF8000"
```

**Expected Output:**
```
"#FF8000"
```

**Why This Output:** Each colour channel is shifted to its correct byte position (red at bits 16–23, green at 8–15, blue at 0–7) and combined with OR.

### Real-World Cases

- **Colour conversion:** RGB to hex, hex to RGB.
- **Flag creation:** `1 << n` for bit positions.
- **Fast multiplication:** `x << 1` instead of `x * 2`.

---

## `>>` (Sign-Propagating Right Shift)

### Definitions

**Core Definition:** The `>>` operator shifts the bits of the left operand to the right, filling vacated positions with copies of the sign bit (the leftmost bit).

**Technical Definition:** Per ECMA-262 §13.9.2, `ShiftExpression : ShiftExpression >> AdditiveExpression` performs `Number::signedRightShift` (or `BigInt::signedRightShift`). The sign bit is replicated to preserve the sign of the number. For 32-bit integers, the shift count is masked to 5 bits.

**Beginner-Friendly Explanation:** `>>` moves bits to the right. If the number is negative (sign bit is `1`), `1`s are filled in from the left. If positive, `0`s are filled. It's like dividing by `2^n` and rounding toward negative infinity.

### Purposes

- To divide by powers of two (with sign preservation).
- To extract specific bits from a larger value.
- To preserve sign during right shifts.

### Syntax Rules and Structure

```js
operand1 >> operand2
```

**Syntax Rules:**
- Shift count masked to 5 bits.
- Sign bit is propagated: `-8 >> 1` = `-4`.
- For BigInt, no masking.

### Annotated Code Examples

#### Example 1: Division by Powers of Two

```js
console.log(16 >> 1);  // 8
console.log(16 >> 2);  // 4
console.log(-16 >> 1); // -8
console.log(-16 >> 2); // -4
```

**Expected Output:**
```
8
4
-8
-4
```

**Why This Output:** Right shifting by `n` divides by `2^n`. The sign is preserved for negative numbers.

#### Example 2: Extracting Colour Channels

```js
const color = 0xFF8000; // RGB colour

// Step 1: Extract red (bits 16-23).
const red = (color >> 16) & 0xFF;
console.log(red); // 255

// Step 2: Extract green (bits 8-15).
const green = (color >> 8) & 0xFF;
console.log(green); // 128

// Step 3: Extract blue (bits 0-7).
const blue = color & 0xFF;
console.log(blue); // 0
```

**Expected Output:**
```
255
128
0
```

**Why This Output:** Shifting right moves the desired byte to the low-order position, and ANDing with `0xFF` isolates it.

### Real-World Cases

- **Colour extraction:** Extracting RGB channels.
- **Fast division:** `x >> 1` instead of `Math.floor(x / 2)`.
- **Sign-preserving bit extraction.**

---

## `>>>` (Zero-Fill Right Shift)

### Definitions

**Core Definition:** The `>>>` operator shifts the bits of the left operand to the right, filling vacated positions with zeros regardless of the sign bit.

**Technical Definition:** Per ECMA-262 §13.9.3, `ShiftExpression : ShiftExpression >>> AdditiveExpression` performs `Number::unsignedRightShift`. The left operand is converted to an unsigned 32-bit integer (`ToUint32`), shifted right, and the result is a non-negative 32-bit integer. This operator is **not** supported for BigInt.

**Beginner-Friendly Explanation:** `>>>` is like `>>` but always fills with `0`s from the left, so the result is always positive. It treats the number as unsigned.

### Purposes

- To convert negative numbers to their unsigned 32-bit representation.
- To perform unsigned division.
- To handle binary data that should be treated as unsigned.

### Syntax Rules and Structure

```js
operand1 >>> operand2
```

**Syntax Rules:**
- Result is always a non-negative 32-bit integer.
- Shift count masked to 5 bits.
- **Not supported for BigInt** — throws `TypeError`.
- `x >>> 0` converts `x` to an unsigned 32-bit integer.

### Annotated Code Examples

#### Example 1: Basic Unsigned Shift

```js
console.log(16 >>> 1);   // 8
console.log(-16 >>> 1);  // 2147483640 (unsigned representation)
console.log(-1 >>> 0);   // 4294967295 (2^32 - 1)
```

**Expected Output:**
```
8
2147483640
4294967295
```

**Why This Output:** `-16` in two's complement has all high bits set. Shifting right with zero-fill produces a large positive number. `-1 >>> 0` produces `4294967295` (all 32 bits set).

#### Example 2: Converting to Unsigned

```js
const signed = -1;
const unsigned = signed >>> 0;
console.log(unsigned); // 4294967295

// Compare with >>.
console.log(-1 >> 0);  // -1 (sign preserved)
```

**Expected Output:**
```
4294967295
-1
```

**Why This Output:** `>>> 0` reinterprets the 32-bit pattern as unsigned. `>> 0` preserves the sign.

### Real-World Cases

- **Hash functions:** Converting hashes to unsigned integers.
- **Binary protocols:** Reading unsigned integers from buffers.
- **Cryptography:** Working with 32-bit unsigned values.

---

## Bitwise Operations

### Definitions

**Core Definition:** Bitwise operations are the individual computations performed by bitwise operators on binary representations of integers.

**Technical Definition:** JavaScript performs bitwise operations in two modes: (1) **Number mode**, where operands are converted to 32-bit signed integers (or unsigned for `>>>`) using `ToInt32`/`ToUint32`, the operation is applied bit-by-bit, and the result is converted back to a JavaScript Number; (2) **BigInt mode**, where operands are arbitrary-precision integers and no truncation occurs (except `>>>` is unsupported).

**Beginner-Friendly Explanation:** Bitwise operations work on the binary form of numbers. JavaScript temporarily converts your number into 32 bits, does the operation, and converts it back.

### Purposes

- To manipulate individual bits for flags, masks, and packed data.
- To perform fast arithmetic (multiplication, division by powers of two).
- To implement low-level algorithms.

### Syntax Rules and Structure

All bitwise operations follow the same pattern:

```js
// Binary operators
a & b    // AND
a | b    // OR
a ^ b    // XOR
a << b   // Left shift
a >> b   // Signed right shift
a >>> b  // Unsigned right shift

// Unary operator
~a       // NOT
```

**Constraints and Limitations:**
- 32-bit truncation for Number operands.
- `NaN`, `Infinity`, and `-Infinity` convert to `0`.
- Non-integer Numbers are truncated toward zero.
- Mixing BigInt and Number throws `TypeError`.

### Annotated Code Examples

#### Example 1: 32-bit Truncation

```js
// Numbers larger than 32 bits lose high bits.
const large = 0x1FFFFFFFF; // 8589934591
console.log(large | 0);    // -1 (truncated to 32-bit)
console.log(large >>> 0);  // 4294967295 (unsigned 32-bit)
```

**Expected Output:**
```
-1
4294967295
```

**Why This Output:** `0x1FFFFFFFF` has 33 bits. The highest bit is discarded, leaving `0xFFFFFFFF`, which is `-1` as a signed 32-bit integer.

#### Example 2: Non-integer Truncation

```js
console.log(5.9 & 0xFF); // 5 (5.9 truncated to 5)
console.log(-5.9 & 0xFF); // 251 (-5.9 truncated to -5, then 32-bit)
```

**Expected Output:**
```
5
251
```

**Why This Output:** Bitwise operators truncate fractional parts before operating.

### Real-World Cases

- **Data packing:** Packing multiple small integers into one 32-bit value.
- **Cryptography:** Bitwise operations in hash and cipher algorithms.
- **Performance optimisation:** Replacing `Math.floor` with `| 0` or `~~`.

---

## Bit Masks

### Definitions

**Core Definition:** A bit mask is a binary number used to selectively manipulate specific bits of another number through bitwise operations.

**Technical Definition:** A bit mask is an integer where specific bit positions are set to `1` (to select or enable those bits) or `0` (to ignore or disable them). Masks are applied via `&` (to test/clear bits), `|` (to set bits), `^` (to toggle bits), and `~` (to invert a mask).

**Beginner-Friendly Explanation:** A bit mask is like a stencil: you place it over a number to keep, change, or check only the bits you care about.

### Purposes

- To store multiple boolean flags in a single integer.
- To extract specific bit fields from a packed value.
- To set, clear, or toggle individual bits.
- To implement permission systems.

### Syntax Rules and Structure

**Creating masks:**
```js
const MASK = 1 << n;         // Single bit at position n
const MASK = (1 << 4) - 1;   // Bits 0-3 set
const MASK = 0b11110000;     // Explicit binary literal
```

**Applying masks:**
```js
value & MASK     // Test/clear bits
value | MASK     // Set bits
value ^ MASK     // Toggle bits
value & ~MASK    // Clear bits
```

**Constraints and Limitations:**
- 32-bit limit for Number masks.
- Bit numbering: bit 0 is the least significant (rightmost).

### Annotated Code Examples

#### Example 1: Permission System

```js
// Step 1: Define permission flags as powers of two.
const READ    = 1 << 0; // 1 (0001)
const WRITE   = 1 << 1; // 2 (0010)
const EXECUTE = 1 << 2; // 4 (0100)
const DELETE  = 1 << 3; // 8 (1000)

// Step 2: Grant permissions.
let permissions = READ | WRITE | EXECUTE; // 7 (0111)

// Step 3: Check permissions.
console.log((permissions & READ) !== 0);    // true
console.log((permissions & DELETE) !== 0);  // false

// Step 4: Revoke a permission.
permissions &= ~WRITE; // Remove WRITE
console.log((permissions & WRITE) !== 0);   // false
console.log(permissions); // 5 (0101)
```

**Expected Output:**
```
true
false
false
5
```

**Why This Output:** Each permission is a unique bit. OR combines them; AND tests them; `& ~MASK` clears them.

#### Example 2: Storing Multiple Flags

```js
// Step 1: Define feature flags.
const FEATURE_A = 1 << 0; // 1
const FEATURE_B = 1 << 1; // 2
const FEATURE_C = 1 << 2; // 4

// Step 2: Store enabled features in one integer.
let flags = FEATURE_A | FEATURE_C; // 5

// Step 3: Check and toggle.
console.log((flags & FEATURE_A) !== 0); // true
console.log((flags & FEATURE_B) !== 0); // false

flags ^= FEATURE_B; // Toggle FEATURE_B on
console.log((flags & FEATURE_B) !== 0); // true
```

**Expected Output:**
```
true
false
true
```

**Why This Output:** Each flag occupies a unique bit position, enabling compact storage and fast manipulation.

### Real-World Cases

- **Operating system permissions:** Unix file permissions (rwx).
- **Game state:** Collision layers, entity states.
- **Feature toggles:** Application feature flags.
- **Network protocols:** TCP flags (SYN, ACK, FIN).

---

## Bitwise Use Cases

### Definitions

**Core Definition:** Bitwise use cases are practical scenarios where bitwise operators provide efficient or idiomatic solutions.

**Technical Definition:** Bitwise operations are applicable whenever data can be represented as sets of boolean flags, packed fields, or requires low-level manipulation. They exploit the 32-bit integer representation for compact storage and fast operations.

**Beginner-Friendly Explanation:** Bitwise operators are useful whenever you need to work with flags, colours, permissions, or any situation where individual bits matter.

### Common Use Cases

1. **Colour Manipulation (RGB/Hex)**
   ```js
   const rgbToHex = (r, g, b) => "#" + ((1 << 24) | (r << 16) | (g << 8) | b).toString(16).slice(1);
   const hexToRgb = (hex) => {
     const n = parseInt(hex.slice(1), 16);
     return { r: (n >> 16) & 0xFF, g: (n >> 8) & 0xFF, b: n & 0xFF };
   };
   ```

2. **Permission Systems**
   ```js
   const READ = 1, WRITE = 2, EXECUTE = 4;
   let perm = READ | WRITE;
   if (perm & READ) { /* has read */ }
   ```

3. **Fast Arithmetic**
   ```js
   x << 1;          // x * 2
   x >> 1;          // Math.floor(x / 2)
   ~~x;             // Math.trunc(x)
   (a + b) | 0;     // Integer addition within 32-bit range
   ```

4. **Toggling Flags**
   ```js
   state ^= FLAG;   // Toggle a flag
   ```

5. **Packing Data**
   ```js
   // Pack two 16-bit values into one 32-bit integer.
   const packed = (high << 16) | low;
   const high = packed >> 16;
   const low = packed & 0xFFFF;
   ```

6. **Swapping Variables**
   ```js
   a ^= b; b ^= a; a ^= b;  // Swap without temp
   ```

7. **Even/Odd Detection**
   ```js
   if (n & 1) { /* odd */ } else { /* even */ }
   ```

8. **Hash Functions**
   ```js
   // Simple string hash using bitwise operations.
   function hash(str) {
     let h = 0;
     for (let i = 0; i < str.length; i++) {
       h = (h << 5) - h + str.charCodeAt(i);
       h |= 0; // Convert to 32-bit
     }
     return h;
   }
   ```

### Annotated Code Examples

#### Example 1: Colour Conversion

```js
// Step 1: RGB to hex.
function rgbToHex(r, g, b) {
  return "#" + ((1 << 24) | (r << 16) | (g << 8) | b).toString(16).slice(1);
}
console.log(rgbToHex(255, 128, 0)); // "#ff8000"

// Step 2: Hex to RGB.
function hexToRgb(hex) {
  const n = parseInt(hex.slice(1), 16);
  return {
    r: (n >> 16) & 0xFF,
    g: (n >> 8) & 0xFF,
    b: n & 0xFF,
  };
}
console.log(hexToRgb("#ff8000")); // { r: 255, g: 128, b: 0 }
```

**Expected Output:**
```
"#ff8000"
{ r: 255, g: 128, b: 0 }
```

**Why This Output:** The `<<` operator positions each colour byte, and `>>`/`&` extract them.

#### Example 2: Fast Integer Operations

```js
// Step 1: Floor division by 2.
console.log(17 >> 1);   // 8 (floor(17/2))

// Step 2: Multiply by 4.
console.log(17 << 2);   // 68 (17 * 4)

// Step 3: Truncate to integer.
console.log(~~3.7);     // 3
console.log(~~-3.7);    // -3

// Step 4: Modulo 2 for even/odd.
console.log(7 & 1);     // 1 (odd)
console.log(8 & 1);     // 0 (even)
```

**Expected Output:**
```
8
68
3
-3
1
0
```

**Why This Output:** Bitwise shifts implement multiplication/division by powers of two; `~~` truncates; `& 1` tests the least significant bit.

### Real-World Cases

- **Canvas pixel manipulation:** Extracting RGBA values from `ImageData`.
- **WebGL/Graphics:** Packing vertex data.
- **Cryptography:** SHA, MD5, and other hash functions.
- **Compression algorithms:** Bit-level data packing.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Bitwise NOT Operator — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-bitwise-not-operator
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Binary Bitwise Operators — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-binary-bitwise-operators
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Bitwise Shift Operators — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-bitwise-shift-operators
- MDN Web Docs – Bitwise AND (&) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_AND
- MDN Web Docs – Bitwise OR (|) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_OR
- MDN Web Docs – Bitwise XOR (^) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_XOR
- MDN Web Docs – Bitwise NOT (~) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_NOT
- MDN Web Docs – Left shift (<<) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Left_shift
- MDN Web Docs – Right shift (>>) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Right_shift
- MDN Web Docs – Unsigned right shift (>>>) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Unsigned_right_shift
- MDN Web Docs – Bitwise operators — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_operators#bitwise_operators
- MDN Web Docs – BigInt — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt
- LogRocket Blog – A guide to JavaScript bitwise operators — https://blog.logrocket.com/guide-javascript-bitwise-operators/
- W3Schools – JavaScript Bitwise — https://www.w3schools.com/js/js_bitwise.asp
- Lullabot – Bitmasks in JavaScript: A Computer Science Crash Course — https://www.lullabot.com/articles/bitmasks-javascript-computer-science-crash-course