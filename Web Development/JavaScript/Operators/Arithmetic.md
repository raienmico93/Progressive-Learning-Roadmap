# JavaScript Arithmetic Operators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Arithmetic operators are the symbols and keywords in JavaScript that perform mathematical computations on numeric operands, including addition, subtraction, multiplication, division, remainder, exponentiation, increment, and decrement.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262), arithmetic operators are specified across several grammar productions. The multiplicative operators (`*`, `/`, `%`) are defined in §13.7, the additive operators (`+`, `-`) in §13.8, the exponentiation operator (`**`) in §13.6, and the unary operators (unary `+`, unary `-`, `++`, `--`) in §13.5. These operators perform operations according to IEEE 754-2019 binary double-precision arithmetic for Number operands, with special handling for BigInt operands and type coercion rules that convert non-numeric operands to Number or BigInt before the operation. The `+` operator is overloaded: it performs numeric addition when both operands are Numbers or BigInts, and string concatenation when either operand is a String.

**Beginner-Friendly Explanation:** Arithmetic operators are the math symbols you already know from school — plus, minus, times, divide — plus a few extras like remainder (`%`), power (`**`), and the increment/decrement shortcuts (`++` and `--`). JavaScript uses them to do calculations with numbers, but it also has some quirks: the `+` symbol can join strings together, and some operators automatically convert non-numbers into numbers.

### Key Characteristics

- **Overloaded `+`:** Performs both numeric addition and string concatenation.
- **Type coercion:** Most arithmetic operators convert operands to Number (or BigInt) before computing.
- **IEEE 754 semantics:** Division by zero yields `Infinity` or `-Infinity`; `0/0` yields `NaN`.
- **BigInt support:** All arithmetic operators except unary `+` work with BigInt operands.
- **Mutating operators:** `++` and `--` modify their operand in place and return a value.
- **Precedence and associativity:** Govern the order of evaluation in complex expressions.

### Prerequisites

- Basic JavaScript syntax (variables, expressions, statements).
- Understanding of JavaScript data types (`Number`, `BigInt`, `String`, `Boolean`, `null`, `undefined`).
- Familiarity with type coercion and the `Number()` conversion function.
- Basic awareness of operator precedence.

### Related Programming Areas

- **Mathematical computing:** Formulas, algorithms, simulations.
- **Data processing:** Aggregations, statistics, transformations.
- **Game development:** Physics, scoring, positioning.
- **Financial applications:** Interest calculations, currency conversions.
- **Graphics programming:** Coordinate transformations, matrix operations.

### Core Concepts / Features

The following sections cover each core concept and feature, each using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## `+` (Addition)

### Definitions

**Core Definition:** The `+` operator performs numeric addition when both operands are numbers, or string concatenation when at least one operand is a string.

**Technical Definition:** Per ECMA-262 §13.8.1, the addition operator evaluates `ApplyStringOrNumericBinaryOperator(lVal, "+", rVal)`. If either operand is a String after `ToPrimitive` conversion, both operands are converted to Strings and concatenated. Otherwise, both are converted to Numeric (Number or BigInt) and added. The result is a String in the concatenation case and a Number or BigInt in the addition case.

**Beginner-Friendly Explanation:** The `+` symbol does two jobs: it adds numbers together, and it joins strings together. If either side is a string, JavaScript treats it as "join" rather than "add."

### Purposes

- To compute the sum of two numbers.
- To concatenate strings.
- To convert values to numbers via unary plus.
- To build formatted output by mixing strings and numbers.

### Syntax Rules and Structure

**General Syntax:**
```js
operand1 + operand2
```

**Component Breakdown:**
- `operand1`, `operand2`: Any JavaScript expressions.
- Returns: A Number, BigInt, or String depending on operand types.

**Syntax Rules:**
- If either operand is a String (after `ToPrimitive`), both are converted to String and concatenated.
- If both are Numbers, numeric addition is performed.
- If both are BigInts, BigInt addition is performed.
- Mixing BigInt and Number throws `TypeError`.
- Objects are converted via `ToPrimitive` (usually `valueOf` then `toString`).

**Constraints and Limitations:**
- Cannot mix BigInt and Number: `1n + 1` throws `TypeError`.
- String concatenation has higher precedence than numeric addition when a string is involved.
- `Symbol` operands throw `TypeError`.
- Floating-point precision limits: `0.1 + 0.2 !== 0.3`.

### Annotated Code Examples

#### Example 1: Numeric Addition vs String Concatenation

```js
// Step 1: Both operands are numbers.
console.log(2 + 3);           // 5

// Step 2: One operand is a string.
console.log("2" + 3);         // "23"

// Step 3: Both operands are strings.
console.log("2" + "3");       // "23"

// Step 4: Boolean coerced to number in numeric context.
console.log(true + 1);        // 2

// Step 5: Boolean coerced to string in string context.
console.log(true + "");       // "true"
```

**Expected Output:**
```
5
23
23
2
true
```

**Why This Output:** When both operands are numbers, numeric addition is performed. When a string is involved, string concatenation takes precedence. Booleans are converted to `1` or `0` in numeric contexts, and to `"true"` or `"false"` in string contexts.

#### Example 2: BigInt Addition

```js
// Step 1: Both operands are BigInt.
console.log(1n + 2n);         // 3n

// Step 2: Mixing BigInt and Number throws.
try {
  console.log(1n + 1);
} catch (err) {
  console.log(err.name);      // "TypeError"
  console.log(err.message);   // "Cannot mix BigInt and other types, use explicit conversions"
}
```

**Expected Output:**
```
3n
TypeError
Cannot mix BigInt and other types, use explicit conversions
```

**Why This Output:** BigInt addition requires both operands to be BigInts. Mixing types is an error because the result could lose precision.

#### Example 3: Floating-Point Precision

```js
console.log(0.1 + 0.2);              // 0.30000000000000004
console.log(0.1 + 0.2 === 0.3);      // false
console.log((0.1 + 0.2).toFixed(2)); // "0.30"
```

**Expected Output:**
```
0.30000000000000004
false
true
```

**Why This Output:** IEEE 754 double-precision cannot represent all decimal fractions exactly. `toFixed()` rounds for display.

### Real-World Cases

- **Cart totals:** `itemPrice + tax`.
- **String formatting:** `"Hello, " + name + "!"`.
- **Counter increments:** `count = count + 1`.
- **Coordinate math:** `x + velocity`.
- **Template building:** `"Total: $" + total.toFixed(2)`.

---

## `-` (Subtraction)

### Definitions

**Core Definition:** The `-` operator subtracts the right operand from the left operand, performing numeric subtraction.

**Technical Definition:** Per ECMA-262 §13.8.2, the subtraction operator evaluates `ApplyStringOrNumericBinaryOperator(lVal, "-", rVal)`. Both operands are converted to Numeric (Number or BigInt), and the right is subtracted from the left. The result is a Number or BigInt.

**Beginner-Friendly Explanation:** `-` subtracts numbers. Unlike `+`, it always works numerically — it never concatenates strings.

### Purposes

- To compute the difference between two numbers.
- To negate a value (as unary minus).
- To calculate deltas, remainders, or offsets.

### Syntax Rules and Structure

**General Syntax:**
```js
operand1 - operand2
```

**Syntax Rules:**
- Both operands are converted to Number (or BigInt) before subtraction.
- Strings are coerced to numbers: `"5" - 2` yields `3`.
- Non-numeric strings become `NaN`: `"abc" - 2` yields `NaN`.
- Mixing BigInt and Number throws `TypeError`.

**Constraints and Limitations:**
- `NaN` propagates: any operation with `NaN` yields `NaN`.
- `Infinity - Infinity` yields `NaN`.
- Cannot mix BigInt and Number.

### Annotated Code Examples

#### Example 1: Numeric Subtraction and Coercion

```js
// Step 1: Both numbers.
console.log(10 - 4);          // 6

// Step 2: String coerced to number.
console.log("10" - 4);        // 6

// Step 3: Non-numeric string.
console.log("abc" - 4);       // NaN

// Step 4: Boolean coerced.
console.log(true - 1);        // 0
```

**Expected Output:**
```
6
6
NaN
0
```

**Why This Output:** The `-` operator always converts operands to numbers. `"10"` becomes `10`, `"abc"` becomes `NaN`, and `true` becomes `1`.

#### Example 2: Date Difference

```js
const start = new Date("2025-01-01");
const end = new Date("2025-01-10");

// Dates are converted to timestamps (milliseconds).
const diffMs = end - start;
console.log(diffMs);                           // 777600000
console.log(diffMs / (1000 * 60 * 60 * 24));   // 9 days
```

**Expected Output:**
```
777600000
9
```

**Why This Output:** The `-` operator coerces Date objects to their numeric timestamp values, allowing date arithmetic.

### Real-World Cases

- **Countdown timers:** `targetTime - currentTime`.
- **Inventory:** `stock - sold`.
- **Coordinates:** `targetX - currentX`.
- **Score differences:** `scoreA - scoreB`.

---

## `*` (Multiplication)

### Definitions

**Core Definition:** The `*` operator multiplies two operands, producing their product.

**Technical Definition:** Per ECMA-262 §13.7.1, the multiplication operator evaluates `ApplyStringOrNumericBinaryOperator(lVal, "*", rVal)`. Both operands are converted to Numeric, and the result is computed using IEEE 754 multiplication (or BigInt multiplication). Multiplication is commutative but not associative in floating-point arithmetic due to finite precision.

**Beginner-Friendly Explanation:** `*` multiplies numbers. It always works numerically and never concatenates.

### Purposes

- To compute products (area, total cost, scaling).
- To repeat values (scaling factors).
- To perform matrix and vector operations.

### Syntax Rules and Structure

```js
operand1 * operand2
```

**Syntax Rules:**
- Both operands converted to Number or BigInt.
- `NaN` propagates.
- `Infinity * 0` yields `NaN`.
- Mixing BigInt and Number throws `TypeError`.

### Annotated Code Examples

#### Example 1: Basic Multiplication

```js
console.log(6 * 7);           // 42
console.log("6" * 7);         // 42 (string coerced)
console.log(2.5 * 4);         // 10
console.log(-3 * 4);          // -12
console.log(Infinity * 0);    // NaN
```

**Expected Output:**
```
42
42
10
-12
NaN
```

**Why This Output:** The `*` operator coerces strings to numbers. `Infinity * 0` is mathematically indeterminate, so it yields `NaN`.

### Real-World Cases

- **Area calculation:** `width * height`.
- **Total price:** `unitPrice * quantity`.
- **Scaling:** `originalSize * scaleFactor`.
- **Physics:** `mass * acceleration`.

---

## `/` (Division)

### Definitions

**Core Definition:** The `/` operator divides the left operand by the right operand.

**Technical Definition:** Per ECMA-262 §13.7.2, the division operator evaluates `ApplyStringOrNumericBinaryOperator(lVal, "/", rVal)`. Both operands are converted to Numeric, and IEEE 754 division is performed. Division by zero yields `Infinity` or `-Infinity` (unless the dividend is zero, yielding `NaN`).

**Beginner-Friendly Explanation:** `/` divides numbers. Unlike some languages, JavaScript does not throw an error when dividing by zero — it returns `Infinity` instead.

### Purposes

- To compute ratios, averages, and rates.
- To split quantities evenly.
- To normalise values.

### Syntax Rules and Structure

```js
operand1 / operand2
```

**Syntax Rules:**
- Both operands converted to Number or BigInt.
- `x / 0` yields `Infinity` (or `-Infinity`).
- `0 / 0` yields `NaN`.
- BigInt division truncates toward zero.
- Mixing BigInt and Number throws `TypeError`.

### Annotated Code Examples

#### Example 1: Division Semantics

```js
console.log(10 / 2);          // 5
console.log(10 / 3);          // 3.3333333333333335
console.log(10 / 0);          // Infinity
console.log(-10 / 0);         // -Infinity
console.log(0 / 0);           // NaN
console.log("10" / 2);        // 5
```

**Expected Output:**
```
5
3.3333333333333335
Infinity
-Infinity
NaN
0 / 0 is NaN
NaN
5
```

**Why This Output:** Division by zero yields `Infinity` or `-Infinity`. Zero divided by zero is `NaN`. String operands are coerced to numbers.

#### Example 2: Average Calculation

```js
const scores = [85, 92, 78, 95, 88];
const total = scores.reduce((sum, s) => sum + s, 0);
const average = total / scores.length;
console.log(average); // 87.6
```

**Expected Output:**
```
87.6
```

**Why This Output:** The total is divided by the count to produce the average.

### Real-World Cases

- **Averages:** `sum / count`.
- **Unit rates:** `distance / time`.
- **Percentages:** `(part / whole) * 100`.
- **Normalisation:** `value / maxValue`.

---

## `%` (Remainder)

### Definitions

**Core Definition:** The `%` operator returns the remainder of dividing the left operand by the right operand.

**Technical Definition:** Per ECMA-262 §13.7.3, the remainder operator evaluates `ApplyStringOrNumericBinaryOperator(lVal, "%", rVal)`. The result has the same sign as the dividend (left operand), following IEEE 754 remainder semantics. It is **not** a true modulo operator: `-7 % 3` yields `-1`, whereas mathematical modulo would yield `2`.

**Beginner-Friendly Explanation:** `%` gives you what's left over after division. For example, `7 % 3` is `1` because 3 goes into 7 twice with 1 left over. With negative numbers, the sign follows the left operand.

### Purposes

- To check divisibility (even/odd).
- To wrap values within a range (with adjustment for negatives).
- To extract digits or cycle through indices.

### Syntax Rules and Structure

```js
operand1 % operand2
```

**Syntax Rules:**
- Result sign matches the dividend (left operand).
- `x % 0` yields `NaN`.
- `Infinity % x` yields `NaN`.
- Works with floating-point numbers.
- BigInt `%` is supported.

**Constraints and Limitations:**
- Not a true mathematical modulo for negative operands.
- To get a positive modulo, use `((n % m) + m) % m`.

### Annotated Code Examples

#### Example 1: Basic Remainder

```js
console.log(7 % 3);           // 1
console.log(-7 % 3);          // -1
console.log(7 % -3);          // 1
console.log(7.5 % 2);         // 1.5
console.log(7 % 0);           // NaN
```

**Expected Output:**
```
1
-1
1
1.5
NaN
```

**Why This Output:** The sign of the result matches the dividend. Division by zero yields `NaN`.

#### Example 2: Even/Odd Check and Positive Modulo

```js
// Step 1: Even/odd check.
function isEven(n) {
  return n % 2 === 0;
}
console.log(isEven(4));       // true
console.log(isEven(7));       // false

// Step 2: True modulo for negative numbers.
function mod(n, m) {
  return ((n % m) + m) % m;
}
console.log(mod(-7, 3));      // 2
console.log(mod(7, 3));       // 1
```

**Expected Output:**
```
true
false
2
1
```

**Why This Output:** `n % 2 === 0` is true for even numbers. The `mod` function adjusts negative results to be positive.

#### Example 3: Cycling Through an Array

```js
const colors = ["red", "green", "blue"];
for (let i = 0; i < 7; i++) {
  console.log(colors[i % colors.length]);
}
```

**Expected Output:**
```
red
green
blue
red
green
blue
red
```

**Why This Output:** The remainder operator wraps the index within the array bounds.

### Real-World Cases

- **Pagination:** `pageNumber % totalPages`.
- **Hash tables:** `hash % tableSize`.
- **Game loops:** Frame-based cycling.
- **Checksums:** Digit extraction.
- **Time formatting:** Seconds to minutes/seconds.

---

## `**` (Exponentiation)

### Definitions

**Core Definition:** The `**` operator raises the left operand to the power of the right operand.

**Technical Definition:** Per ECMA-262 §13.6, the exponentiation operator evaluates `Number::exponentiate(base, exponent)` or `BigInt::exponentiate(base, exponent)`. It is right-associative: `a ** b ** c` is evaluated as `a ** (b ** c)`. It has higher precedence than multiplicative operators but lower than unary operators (except that unary operators cannot directly precede `**` without parentheses).

**Beginner-Friendly Explanation:** `**` is the power operator. `2 ** 3` means "2 to the power of 3," which is 8. It's a shorthand for `Math.pow(2, 3)`.

### Purposes

- To compute powers and roots.
- To implement exponential growth/decay formulas.
- To perform compound interest and scientific calculations.

### Syntax Rules and Structure

```js
base ** exponent
```

**Syntax Rules:**
- Right-associative: `2 ** 3 ** 2` is `2 ** (3 ** 2)` = `2 ** 9` = `512`.
- Unary minus cannot directly precede `**`: `-2 ** 2` is a syntax error; use `(-2) ** 2` or `-(2 ** 2)`.
- BigInt exponentiation requires a non-negative BigInt exponent.
- `0 ** 0` yields `1`.
- Negative base with non-integer exponent yields `NaN`.

**Constraints and Limitations:**
- BigInt exponent must be non-negative.
- `(-8) ** (1/3)` yields `NaN` (use `Math.cbrt` for cube roots).
- No `^` operator for exponentiation in JavaScript (`^` is bitwise XOR).

### Annotated Code Examples

#### Example 1: Basic Exponentiation

```js
console.log(2 ** 3);          // 8
console.log(2 ** 0);          // 1
console.log(0 ** 0);          // 1
console.log((-2) ** 2);       // 4
console.log(2 ** -1);         // 0.5
console.log(2 ** 3 ** 2);     // 512 (right-associative)
```

**Expected Output:**
```
8
1
1
4
0.5
512
```

**Why This Output:** Exponentiation follows mathematical rules. Right-associativity means `2 ** 3 ** 2` is `2 ** 9`, not `(2 ** 3) ** 2`.

#### Example 2: Compound Interest

```js
const principal = 1000;
const rate = 0.05;
const years = 10;

const amount = principal * (1 + rate) ** years;
console.log(amount.toFixed(2)); // "1628.89"
```

**Expected Output:**
```
1628.89
```

**Why This Output:** The compound interest formula is `P * (1 + r) ** t`.

#### Example 3: Syntax Error with Unary Minus

```js
try {
  eval("-2 ** 2");
} catch (err) {
  console.log(err.name);    // "SyntaxError"
  console.log(err.message); // "Unary operator used immediately before exponentiation expression. Parenthesis must be used to disambiguate operator precedence"
}
```

**Expected Output:**
```
SyntaxError
Unary operator used immediately before exponentiation expression. Parenthesis must be used to disambiguate operator precedence
```

**Why This Output:** JavaScript requires parentheses to disambiguate `-2 ** 2`. Write `(-2) ** 2` or `-(2 ** 2)`.

### Real-World Cases

- **Compound interest:** `P * (1 + r) ** n`.
- **Scientific notation:** `mass * c ** 2`.
- **Area/volume:** `side ** 2`, `side ** 3`.
- **Growth models:** `initial * growthRate ** periods`.

---

## `++` (Increment)

### Definitions

**Core Definition:** The `++` operator adds 1 to its operand and returns either the original value (postfix) or the incremented value (prefix).

**Technical Definition:** Per ECMA-262 §13.5.4 and §13.5.5, the increment operator converts its operand to Numeric, adds 1, and assigns the result back to the operand. Prefix (`++x`) returns the new value; postfix (`x++`) returns the old value.

**Beginner-Friendly Explanation:** `++` is a shortcut for "add 1." If you write `x++`, you get the old value and then `x` is increased. If you write `++x`, `x` is increased first and you get the new value.

### Purposes

- To increment counters and loop variables.
- To implement compact iteration.
- To adjust values in place.

### Syntax Rules and Structure

```js
x++   // postfix: returns old value, then increments
++x   // prefix: increments, then returns new value
```

**Syntax Rules:**
- Operand must be a valid assignment target (variable, property, etc.).
- Operand is coerced to Number (or BigInt).
- Postfix and prefix produce different return values but the same final state.
- Cannot be used on literals: `5++` is a syntax error.

**Constraints and Limitations:**
- Mixing prefix and postfix in complex expressions can be confusing; prefer explicit statements.
- BigInt is supported.
- `++` on a `const` variable throws `TypeError`.

### Annotated Code Examples

#### Example 1: Prefix vs Postfix

```js
let a = 5;
let b = a++;   // b gets old value (5), a becomes 6
console.log(a, b); // 6 5

let c = 5;
let d = ++c;   // c becomes 6, d gets new value (6)
console.log(c, d); // 6 6
```

**Expected Output:**
```
6 5
6 6
```

**Why This Output:** Postfix returns the value before incrementing; prefix returns after.

#### Example 2: Loop Counter

```js
for (let i = 0; i < 3; i++) {
  console.log(i);
}
```

**Expected Output:**
```
0
1
2
```

**Why This Output:** The postfix `i++` returns the old value for the comparison but increments `i` for the next iteration.

### Real-World Cases

- **Loop counters:** `for (let i = 0; i < n; i++)`.
- **Click counters:** `clickCount++`.
- **Pagination:** `currentPage++`.
- **Game scores:** `playerScore++`.

---

## `--` (Decrement)

### Definitions

**Core Definition:** The `--` operator subtracts 1 from its operand and returns either the original value (postfix) or the decremented value (prefix).

**Technical Definition:** Per ECMA-262 §13.5.6 and §13.5.7, the decrement operator converts its operand to Numeric, subtracts 1, and assigns the result back. Prefix (`--x`) returns the new value; postfix (`x--`) returns the old value.

**Beginner-Friendly Explanation:** `--` is a shortcut for "subtract 1." Like `++`, postfix returns the old value and prefix returns the new value.

### Purposes

- To decrement counters and loop variables.
- To implement countdowns.
- To adjust values in place.

### Syntax Rules and Structure

```js
x--   // postfix: returns old value, then decrements
--x   // prefix: decrements, then returns new value
```

**Constraints and Limitations:**
- Same as `++`.
- Cannot be used on literals.
- `--` on a `const` variable throws `TypeError`.

### Annotated Code Examples

#### Example 1: Countdown

```js
let count = 3;
while (count > 0) {
  console.log(count--);
}
console.log("Liftoff!");
```

**Expected Output:**
```
3
2
1
Liftoff!
```

**Why This Output:** `count--` logs the current value and then decrements.

#### Example 2: Prefix vs Postfix

```js
let x = 10;
const y = x--;
console.log(x, y); // 9 10

let a = 10;
const b = --a;
console.log(a, b); // 9 9
```

**Expected Output:**
```
9 10
9 9
```

**Why This Output:** Postfix returns the old value; prefix returns the new value.

### Real-World Cases

- **Countdowns:** `timer--`.
- **Inventory:** `stock--`.
- **Lives in games:** `lives--`.
- **Retry logic:** `retries--`.

---

## Unary Plus (`+`)

### Definitions

**Core Definition:** The unary plus operator converts its operand to a Number.

**Technical Definition:** Per ECMA-262 §13.5.2, the unary plus operator performs `ToNumber(operand)`. It is the fastest and preferred way to convert a value to a number because it performs no other operations.

**Beginner-Friendly Explanation:** Putting `+` in front of a value converts it to a number. For example, `+"5"` gives the number `5`.

### Purposes

- To convert strings and booleans to numbers.
- To perform numeric coercion without side effects.
- To convert `Date` objects to timestamps.

### Syntax Rules and Structure

```js
+operand
```

**Syntax Rules:**
- Operand can be any expression.
- Strings are parsed as numbers if valid; otherwise `NaN`.
- `null` becomes `0`; `undefined` becomes `NaN`.
- `true` becomes `1`; `false` becomes `0`.
- Empty string becomes `0`; whitespace-only strings become `0`.
- Objects are converted via `ToPrimitive` then `ToNumber`.

**Constraints and Limitations:**
- `+` on `BigInt` throws `TypeError` (unary `+` does not support BigInt).
- Does not parse strings with trailing characters: `+"5px"` yields `NaN`.

### Annotated Code Examples

#### Example 1: String to Number Conversion

```js
console.log(+"42");         // 42
console.log(+"3.14");       // 3.14
console.log(+"  10  ");     // 10
console.log(+"abc");        // NaN
console.log(+"");           // 0
console.log(+"5px");        // NaN
```

**Expected Output:**
```
42
3.14
10
NaN
0
NaN
```

**Why This Output:** The unary plus parses the string as a number. Trailing non-numeric characters cause `NaN`.

#### Example 2: Boolean and Null Conversion

```js
console.log(+true);         // 1
console.log(+false);        // 0
console.log(+null);         // 0
console.log(+undefined);    // NaN
```

**Expected Output:**
```
1
0
0
NaN
```

**Why This Output:** `true` → `1`, `false` → `0`, `null` → `0`, `undefined` → `NaN`.

#### Example 3: Date to Timestamp

```js
const now = new Date();
console.log(typeof +now);   // "number"
console.log(+now > 0);      // true
```

**Expected Output:**
```
number
true
```

**Why This Output:** `ToPrimitive` on a `Date` returns its numeric timestamp, which unary plus preserves.

### Real-World Cases

- **Form input conversion:** `+input.value`.
- **Date comparison:** `+date1 > +date2`.
- **Numeric parsing:** `+"3.14"`.
- **Boolean to number:** `+isEnabled`.

---

## Unary Minus (`-`)

### Definitions

**Core Definition:** The unary minus operator converts its operand to a Number and negates it.

**Technical Definition:** Per ECMA-262 §13.5.3, the unary minus operator performs `ToNumeric(operand)` and then negates the result. It supports both Number and BigInt.

**Beginner-Friendly Explanation:** Putting `-` in front of a value converts it to a number and flips its sign.

### Purposes

- To negate numeric values.
- To convert and negate in one step.
- To create negative numbers from positive literals.

### Syntax Rules and Structure

```js
-operand
```

**Syntax Rules:**
- Converts operand to Numeric, then negates.
- Supports BigInt: `-1n` yields `-1n`.
- `-NaN` yields `NaN`.
- `-Infinity` yields `-Infinity`.

### Annotated Code Examples

#### Example 1: Basic Negation and Coercion

```js
console.log(-5);            // -5
console.log(-"5");          // -5
console.log(-true);         // -1
console.log(-false);        // -0
console.log(-null);         // -0
console.log(-undefined);    // NaN
console.log(-(-3));         // 3
```

**Expected Output:**
```
-5
-5
-1
-0
-0
NaN
3
```

**Why This Output:** Unary minus coerces to number and negates. Double negation restores the original value.

#### Example 2: BigInt Negation

```js
console.log(-10n);          // -10n
console.log(typeof -10n);   // "bigint"
```

**Expected Output:**
```
-10n
bigint
```

**Why This Output:** Unlike unary plus, unary minus supports BigInt.

### Real-World Cases

- **Coordinate systems:** `-y` for screen coordinates.
- **Vector math:** Negating directions.
- **Temperature:** `-temperature`.
- **Financial:** `-debit`.

---

## Operator Precedence

### Definitions

**Core Definition:** Operator precedence determines which operators are evaluated first in an expression containing multiple operators.

**Technical Definition:** Per ECMA-262, precedence levels are defined by the grammar hierarchy. Higher-precedence operators bind tighter. The arithmetic precedence order (highest to lowest) is: grouping `()` → unary `+`/`-`/`++`/`--` → exponentiation `**` → multiplicative `*`/`/`/`%` → additive `+`/`-`.

**Beginner-Friendly Explanation:** Precedence is like the order of operations you learned in math (PEMDAS). Multiplication happens before addition unless parentheses say otherwise.

### Purposes

- To control the order of evaluation in complex expressions.
- To avoid bugs from incorrect assumptions about evaluation order.
- To write clear, unambiguous code.

### Precedence Table (Arithmetic-Related)

| Precedence | Operator | Description | Associativity |
|---|---|---|---|
| 1 (highest) | `()` | Grouping | — |
| 2 | `++`, `--` (postfix) | Postfix increment/decrement | — |
| 3 | `!`, `~`, `+` (unary), `-` (unary), `++`, `--` (prefix) | Unary operators | Right-to-left |
| 4 | `**` | Exponentiation | Right-to-left |
| 5 | `*`, `/`, `%` | Multiplicative | Left-to-right |
| 6 | `+`, `-` | Additive | Left-to-right |

### Annotated Code Examples

#### Example 1: Precedence in Action

```js
console.log(2 + 3 * 4);        // 14 (3*4 first)
console.log((2 + 3) * 4);      // 20 (parentheses override)
console.log(2 * 3 ** 2);       // 18 (3**2 first)
console.log(-2 ** 2);          // SyntaxError (must parenthesise)
console.log((-2) ** 2);        // 4
console.log(-(2 ** 2));        // -4
```

**Expected Output:**
```
14
20
18
SyntaxError
4
-4
```

**Why This Output:** `**` has higher precedence than `*`, which has higher precedence than `+`. Parentheses override. Unary minus cannot directly precede `**`.

#### Example 2: Common Precedence Bugs

```js
// BUG: Intended (a + b) / 2, but got a + (b / 2)
const a = 10, b = 20;
console.log(a + b / 2);        // 20, not 15
console.log((a + b) / 2);      // 15
```

**Expected Output:**
```
20
15
```

**Why This Output:** Division binds tighter than addition. Always parenthesise to clarify intent.

### Real-World Cases

- **Mathematical formulas:** Ensuring correct evaluation order.
- **Financial calculations:** Avoiding rounding errors from wrong order.
- **Game physics:** `x + velocity * dt`.
- **Data transformations:** Compound expressions.

---

## Associativity

### Definitions

**Core Definition:** Associativity determines the order of evaluation when operators of the same precedence appear consecutively.

**Technical Definition:** Left-associative operators group from left to right: `a - b - c` is `(a - b) - c`. Right-associative operators group from right to left: `a ** b ** c` is `a ** (b ** c)`. Most arithmetic operators are left-associative; exponentiation is right-associative.

**Beginner-Friendly Explanation:** Associativity tells you which side to evaluate first when operators have equal precedence. Subtraction is left-associative, so `10 - 5 - 2` is `(10 - 5) - 2 = 3`, not `10 - (5 - 2) = 7`.

### Purposes

- To understand how chains of same-precedence operators evaluate.
- To avoid off-by-one errors in calculations.
- To correctly predict the result of `**` chains.

### Associativity Table

| Operators | Associativity |
|---|---|
| `*`, `/`, `%` | Left-to-right |
| `+`, `-` | Left-to-right |
| `**` | Right-to-left |
| Unary `+`, `-`, `++`, `--` | Right-to-left |

### Annotated Code Examples

#### Example 1: Left-Associativity of Subtraction and Division

```js
console.log(10 - 5 - 2);      // 3 = (10 - 5) - 2
console.log(10 - (5 - 2));    // 7
console.log(100 / 10 / 2);    // 5 = (100 / 10) / 2
console.log(100 / (10 / 2));  // 20
```

**Expected Output:**
```
3
7
5
20
```

**Why This Output:** Subtraction and division are left-associative, so they evaluate left-to-right.

#### Example 2: Right-Associativity of Exponentiation

```js
console.log(2 ** 3 ** 2);     // 512 = 2 ** (3 ** 2)
console.log((2 ** 3) ** 2);   // 64
```

**Expected Output:**
```
512
64
```

**Why This Output:** Exponentiation is right-associative, so `2 ** 3 ** 2` is `2 ** 9`, not `(2 ** 3) ** 2`.

### Real-World Cases

- **Chained calculations:** `total - discount1 - discount2`.
- **Ratio calculations:** `a / b / c`.
- **Power towers:** Scientific notation with nested exponents.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Multiplicative Operators — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-multiplicative-operators
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Additive Operators — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-additive-operators
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Exponentiation Operator — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-exp-operator
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Unary Operators — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-unary-operators
- MDN Web Docs – Addition (+) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Addition
- MDN Web Docs – Subtraction (-) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Subtraction
- MDN Web Docs – Multiplication (*) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Multiplication
- MDN Web Docs – Division (/) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Division
- MDN Web Docs – Remainder (%) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Remainder
- MDN Web Docs – Exponentiation (**) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Exponentiation
- MDN Web Docs – Increment (++) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Increment
- MDN Web Docs – Decrement (--) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Decrement
- MDN Web Docs – Unary plus (+) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Unary_plus
- MDN Web Docs – Unary negation (-) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Unary_negation
- MDN Web Docs – Operator precedence — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_precedence
- MDN Web Docs – Expressions and operators — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_operators