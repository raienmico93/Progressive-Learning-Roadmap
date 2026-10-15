# JavaScript Assignment Operators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Assignment operators are the symbols and expressions in JavaScript used to store values into variables, object properties, or destructuring patterns, with the simple assignment (`=`) forming the foundation and compound assignments (`+=`, `-=`, etc.) providing shorthand for read-modify-write operations.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §13.15), assignment operators are defined by the `AssignmentExpression` grammar production. The simple assignment `LeftHandSideExpression = AssignmentExpression` evaluates the left-hand side to a Reference, evaluates the right-hand side, applies `PutValue` to store the result, and returns the assigned value. Compound assignments of the form `LeftHandSideExpression @= AssignmentExpression` desugar to `LeftHandSideExpression = LeftHandSideExpression @ AssignmentExpression` with the key difference that the left-hand side is evaluated only once. Logical assignment operators (`&&=`, `||=`, `??=`) short-circuit based on the truthiness or nullishness of the left operand. Destructuring assignment extends the grammar to allow array and object patterns on the left-hand side, unpacking iterable or object values into multiple bindings or assignments.

**Beginner-Friendly Explanation:** Assignment operators are how you put values into variables. The basic one is `=` — you write `x = 5` to store the number 5 in `x`. The other assignment operators are shortcuts: `x += 3` means "add 3 to whatever `x` is and store the result back in `x`." JavaScript also has special assignment operators that only assign under certain conditions (like when a value is missing or falsy), and a powerful feature called *destructuring* that lets you unpack values from arrays and objects into multiple variables at once.

### Key Characteristics

- **Assignment is an expression:** Every assignment returns the assigned value, enabling chained assignments (`a = b = c = 5`).
- **Right-associative:** Chained assignments evaluate from right to left.
- **Compound shorthand:** `x += y` is equivalent to `x = x + y`, but `x` is evaluated only once.
- **Short-circuiting logical assignments:** `&&=`, `||=`, and `??=` may not evaluate the right-hand side at all.
- **Destructuring is both declaration and assignment:** Patterns can be used with `let`, `const`, `var`, or as plain assignment expressions.
- **Strict mode differences:** Assignment to undeclared variables throws `ReferenceError` in strict mode.

### Prerequisites

- Basic JavaScript syntax (variables, expressions, statements).
- Understanding of references and values (primitives vs. objects).
- Familiarity with truthiness and falsiness.
- Knowledge of arrays, objects, and iterables.
- Awareness of strict mode semantics.

### Related Programming Areas

- **Variable management:** Declaring, initialising, and updating variables.
- **Functional programming:** Destructuring in function parameters and return values.
- **State management:** Updating application state with compound assignments.
- **Configuration:** Applying defaults with `??=` and `||=`.
- **Data transformation:** Extracting and restructuring data via destructuring.

### Core Concepts / Features

The following sections cover each core concept and feature, each using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## `=` (Simple Assignment)

### Definitions

**Core Definition:** The `=` operator assigns the value of its right operand to the variable, property, or destructuring pattern on its left.

**Technical Definition:** Per ECMA-262 §13.15.2, the production `LeftHandSideExpression = AssignmentExpression` evaluates the left-hand side to a Reference, evaluates the right-hand side, calls `GetValue` on the result, calls `PutValue` on the Reference with the value, and returns the value. The left-hand side must be a valid assignment target (identifier, property accessor, or destructuring pattern). The assignment expression itself evaluates to the assigned value, enabling chaining.

**Beginner-Friendly Explanation:** `=` stores a value. Write `x = 5` to put 5 into `x`. You can also store values into object properties (`obj.name = "Alice"`) or array elements (`arr[0] = 10`).

### Purposes

- To store a value in a variable.
- To update an object property or array element.
- To initialise destructuring patterns.
- To chain multiple assignments in one expression.

### Syntax Rules and Structure

**General Syntax:**
```js
target = value
```

**Component Breakdown:**
- `target`: A valid assignment target: identifier, property accessor (`obj.prop`, `obj["prop"]`), or destructuring pattern.
- `value`: Any JavaScript expression.
- Returns: The assigned value.

**Syntax Rules:**
- The left-hand side must be a valid `LeftHandSideExpression`; literals and function calls are invalid.
- In strict mode, assigning to an undeclared identifier throws `ReferenceError`.
- Assigning to a non-writable property throws `TypeError` in strict mode.
- The expression evaluates to the assigned value, enabling chaining.

**Constraints and Limitations:**
- Cannot assign to `const` bindings after initialisation.
- Cannot assign to `this`.
- Assignment has lower precedence than most operators, so `x = a + b` groups as `x = (a + b)`.

### Annotated Code Examples

#### Example 1: Basic Assignment

```js
// Step 1: Assign a number to a variable.
let x = 5;
console.log(x); // 5

// Step 2: Reassign the variable.
x = 10;
console.log(x); // 10

// Step 3: Assign to an object property.
const obj = {};
obj.name = "Alice";
console.log(obj.name); // "Alice"

// Step 4: Assign to an array element.
const arr = [1, 2, 3];
arr[0] = 99;
console.log(arr); // [99, 2, 3]
```

**Expected Output:**
```
5
10
Alice
[ 99, 2, 3 ]
```

**Why This Output:** The simple assignment stores each value into the target. Object properties and array elements are mutable, so reassignment works.

#### Example 2: Assignment as an Expression

```js
// Step 1: Assignment returns the assigned value.
const result = (x = 42);
console.log(x);      // 42
console.log(result); // 42

// Step 2: Use in a conditional.
let y;
if ((y = computeValue()) > 0) {
  console.log("Positive:", y);
}
function computeValue() { return 7; }
```

**Expected Output:**
```
42
42
Positive: 7
```

**Why This Output:** The assignment expression evaluates to the assigned value, which can be used in other expressions.

### Real-World Cases

- **Variable initialisation:** `let count = 0;`.
- **State updates:** `state.user = newUser;`.
- **DOM manipulation:** `element.textContent = "Hello";`.
- **Configuration:** `config.timeout = 5000;`.

---

## `+=` (Addition Assignment)

### Definitions

**Core Definition:** The `+=` operator adds the right operand to the left operand and assigns the result back to the left operand.

**Technical Definition:** Per ECMA-262 §13.15.3, `x += y` is evaluated as `x = x + y`, with the key optimisation that the left-hand side is evaluated only once. If either operand is a String, string concatenation is performed; otherwise, numeric addition is performed.

**Beginner-Friendly Explanation:** `x += 3` means "add 3 to x and store the result in x." It also works for joining strings: `message += " world"` appends text.

### Purposes

- To increment a numeric value by a given amount.
- To concatenate strings.
- To accumulate results in loops.

### Syntax Rules and Structure

```js
target += value
```

**Component Breakdown:**
- `target`: A valid assignment target.
- `value`: Any expression.
- Returns: The new value of `target`.

**Syntax Rules:**
- The left-hand side is evaluated once (important when it's a property access with side effects).
- String concatenation applies when either operand is a String.
- Mixing BigInt and Number throws `TypeError`.

### Annotated Code Examples

#### Example 1: Numeric and String Accumulation

```js
// Step 1: Numeric accumulation.
let total = 0;
total += 10;
total += 20;
console.log(total); // 30

// Step 2: String concatenation.
let message = "Hello";
message += ", World";
console.log(message); // "Hello, World"
```

**Expected Output:**
```
30
Hello, World
```

**Why This Output:** `+=` performs numeric addition for numbers and string concatenation for strings.

#### Example 2: Left-Hand Side Evaluated Once

```js
const obj = { count: 0 };
function getObj() {
  console.log("getObj called");
  return obj;
}

getObj().count += 5;
console.log(obj.count); // 5
```

**Expected Output:**
```
getObj called
5
```

**Why This Output:** `getObj()` is called only once, even though `+=` conceptually reads and writes the property.

### Real-World Cases

- **Counters:** `clickCount += 1`.
- **Totals:** `cartTotal += itemPrice`.
- **String building:** `html += "<li>...</li>"`.
- **Score tracking:** `score += points`.

---

## `-=` (Subtraction Assignment)

### Definitions

**Core Definition:** The `-=` operator subtracts the right operand from the left operand and assigns the result back.

**Technical Definition:** Per ECMA-262 §13.15.3, `x -= y` desugars to `x = x - y` with single evaluation of `x`. Both operands are converted to Numeric.

**Beginner-Friendly Explanation:** `x -= 3` means "subtract 3 from x and store the result in x."

### Purposes

- To decrement a numeric value by a given amount.
- To reduce counters, balances, or quantities.

### Syntax Rules and Structure

```js
target -= value
```

### Annotated Code Examples

```js
let balance = 100;
balance -= 30;
console.log(balance); // 70

let countdown = 10;
countdown -= 3;
console.log(countdown); // 7
```

**Expected Output:**
```
70
7
```

**Why This Output:** `-=` subtracts the right operand from the left and assigns back.

### Real-World Cases

- **Account balances:** `balance -= withdrawal`.
- **Inventory:** `stock -= sold`.
- **Timers:** `timeLeft -= elapsed`.
- **Health points:** `hp -= damage`.

---

## `*=` (Multiplication Assignment)

### Definitions

**Core Definition:** The `*=` operator multiplies the left operand by the right operand and assigns the result back.

**Technical Definition:** Per ECMA-262 §13.15.3, `x *= y` desugars to `x = x * y` with single evaluation of `x`.

**Beginner-Friendly Explanation:** `x *= 2` means "double x and store the result in x."

### Purposes

- To scale a value multiplicatively.
- To compound growth or interest.

### Annotated Code Examples

```js
let value = 5;
value *= 3;
console.log(value); // 15

let growth = 1.05;
let investment = 1000;
investment *= growth;
console.log(investment); // 1050
```

**Expected Output:**
```
15
1050
```

### Real-World Cases

- **Scaling:** `size *= scaleFactor`.
- **Compound interest:** `amount *= (1 + rate)`.
- **Damage multipliers:** `damage *= criticalMultiplier`.

---

## `/=` (Division Assignment)

### Definitions

**Core Definition:** The `/=` operator divides the left operand by the right operand and assigns the result back.

**Technical Definition:** Per ECMA-262 §13.15.3, `x /= y` desugars to `x = x / y` with single evaluation of `x`. Division by zero yields `Infinity` or `NaN`.

**Beginner-Friendly Explanation:** `x /= 2` means "halve x and store the result in x."

### Annotated Code Examples

```js
let value = 20;
value /= 4;
console.log(value); // 5

let distance = 100;
distance /= 0;
console.log(distance); // Infinity
```

**Expected Output:**
```
5
Infinity
```

### Real-World Cases

- **Averages:** `sum /= count`.
- **Unit conversion:** `meters /= 1000`.
- **Normalisation:** `value /= maxValue`.

---

## `%=` (Remainder Assignment)

### Definitions

**Core Definition:** The `%=` operator computes the remainder of dividing the left operand by the right operand and assigns the result back.

**Technical Definition:** Per ECMA-262 §13.15.3, `x %= y` desugars to `x = x % y` with single evaluation of `x`. The result sign matches the dividend.

**Beginner-Friendly Explanation:** `x %= 3` means "replace x with the remainder of x divided by 3."

### Annotated Code Examples

```js
let value = 17;
value %= 5;
console.log(value); // 2

let index = 0;
index = (index + 1) % 3; // cycling
console.log(index); // 1
```

**Expected Output:**
```
2
1
```

### Real-World Cases

- **Cycling:** `index %= arrayLength`.
- **Digit extraction:** `num %= 10`.
- **Wrapping coordinates:** `x %= canvasWidth`.

---

## `**=` (Exponentiation Assignment)

### Definitions

**Core Definition:** The `**=` operator raises the left operand to the power of the right operand and assigns the result back.

**Technical Definition:** Per ECMA-262 §13.6, `x **= y` desugars to `x = x ** y` with single evaluation of `x`. Supports Number and BigInt (with non-negative BigInt exponent).

**Beginner-Friendly Explanation:** `x **= 2` means "square x and store the result in x."

### Annotated Code Examples

```js
let value = 3;
value **= 2;
console.log(value); // 9

let compound = 2;
compound **= 3;
console.log(compound); // 8
```

**Expected Output:**
```
9
8
```

### Real-World Cases

- **Squaring:** `area **= 2`.
- **Cubing:** `volume **= 3`.
- **Exponential growth:** `population **= growthFactor`.

---

## `&&=` (Logical AND Assignment)

### Definitions

**Core Definition:** The `&&=` operator assigns the right operand to the left only if the left operand is truthy.

**Technical Definition:** Per ECMA-262 §13.15.3, `x &&= y` is equivalent to `x && (x = y)`, except that `x` is evaluated only once. If `x` is falsy, the right-hand side is not evaluated at all (short-circuit).

**Beginner-Friendly Explanation:** `x &&= y` means "if x is truthy, set it to y; otherwise, leave it alone."

### Purposes

- To conditionally update a value only when the current value is truthy.
- To guard assignments behind existence or validity checks.

### Syntax Rules and Structure

```js
target &&= value
```

**Syntax Rules:**
- Short-circuits: if `target` is falsy, `value` is not evaluated.
- No assignment occurs when `target` is falsy.
- Works even when `target` is `const` if the short-circuit prevents assignment.

### Annotated Code Examples

#### Example 1: Conditional Update

```js
let user = { name: "Alice" };
user.name &&= user.name.toUpperCase();
console.log(user.name); // "ALICE"

let empty = "";
empty &&= "default";
console.log(empty); // "" (unchanged)
```

**Expected Output:**
```
ALICE
```

**Why This Output:** `user.name` is truthy, so it is updated. `empty` is falsy, so no assignment occurs.

#### Example 2: Short-Circuit Avoids Side Effects

```js
let x = 0;
x &&= console.log("This never runs");
console.log(x); // 0
```

**Expected Output:**
```
0
```

**Why This Output:** `x` is falsy, so the right-hand side (the `console.log`) is never evaluated.

### Real-World Cases

- **Conditional formatting:** `label &&= label.trim()`.
- **Guarded updates:** `config.value &&= sanitize(config.value)`.
- **Feature flags:** `features.analytics &&= enableAnalytics()`.

---

## `||=` (Logical OR Assignment)

### Definitions

**Core Definition:** The `||=` operator assigns the right operand to the left only if the left operand is falsy.

**Technical Definition:** Per ECMA-262 §13.15.3, `x ||= y` is equivalent to `x || (x = y)`, except that `x` is evaluated only once. If `x` is truthy, the right-hand side is not evaluated.

**Beginner-Friendly Explanation:** `x ||= y` means "if x is falsy, set it to y; otherwise, leave it alone." It's a shortcut for providing a default value.

### Purposes

- To provide default values for falsy variables.
- To initialise options or configuration.
- To replace `if (!x) x = y` patterns.

### Syntax Rules and Structure

```js
target ||= value
```

**Syntax Rules:**
- Short-circuits: if `target` is truthy, `value` is not evaluated.
- Assigns when `target` is falsy (`0`, `""`, `false`, `null`, `undefined`, `NaN`).
- If `target` is truthy, no assignment occurs.

### Annotated Code Examples

#### Example 1: Default Values

```js
let title = "";
title ||= "Untitled";
console.log(title); // "Untitled"

let duration = 50;
duration ||= 10;
console.log(duration); // 50 (unchanged, 50 is truthy)
```

**Expected Output:**
```
Untitled
50
```

**Why This Output:** `""` is falsy, so the default is assigned. `50` is truthy, so no change.

#### Example 2: Short-Circuit Avoids Evaluation

```js
let x = "value";
x ||= console.log("This never runs");
console.log(x); // "value"
```

**Expected Output:**
```
value
```

**Why This Output:** `x` is truthy, so the right-hand side is not evaluated.

#### Example 3: Note on Falsy Values

```js
let count = 0;
count ||= 10;
console.log(count); // 10 (0 is falsy, so 10 is assigned)
```

**Expected Output:**
```
10
```

**Why This Output:** `0` is falsy, so `||=` assigns the default. If you want to preserve `0`, use `??=` instead.

### Real-World Cases

- **Default configuration:** `config.timeout ||= 5000`.
- **Placeholder text:** `element.textContent ||= "No content"`.
- **User preferences:** `preferences.theme ||= "light"`.

---

## `??=` (Nullish Coalescing Assignment)

### Definitions

**Core Definition:** The `??=` operator assigns the right operand to the left only if the left operand is `null` or `undefined`.

**Technical Definition:** Per ECMA-262 §13.15.3, `x ??= y` is equivalent to `x ?? (x = y)`, except that `x` is evaluated only once. If `x` is neither `null` nor `undefined`, the right-hand side is not evaluated.

**Beginner-Friendly Explanation:** `x ??= y` means "if x is null or undefined, set it to y; otherwise, leave it alone." Unlike `||=`, it preserves falsy values like `0` and `""`.

### Purposes

- To apply defaults only when a value is missing.
- To preserve falsy values like `0`, `""`, and `false`.
- To initialise optional properties without overwriting valid values.

### Syntax Rules and Structure

```js
target ??= value
```

**Syntax Rules:**
- Short-circuits: if `target` is not nullish, `value` is not evaluated.
- Assigns when `target` is exactly `null` or `undefined`.
- Preserves `0`, `""`, `false`, and `NaN`.

### Annotated Code Examples

#### Example 1: Preserving Falsy Values

```js
let count = 0;
count ??= 10;
console.log(count); // 0 (preserved, 0 is not nullish)

let name = null;
name ??= "Anonymous";
console.log(name); // "Anonymous"
```

**Expected Output:**
```
0
Anonymous
```

**Why This Output:** `0` is not nullish, so it is preserved. `null` is nullish, so the default is assigned.

#### Example 2: Configuration Defaults

```js
function config(options) {
  options.duration ??= 100;
  options.speed ??= 25;
  return options;
}

console.log(config({ duration: 125 })); // { duration: 125, speed: 25 }
console.log(config({}));                // { duration: 100, speed: 25 }
console.log(config({ duration: 0 }));   // { duration: 0, speed: 25 }
```

**Expected Output:**
```
{ duration: 125, speed: 25 }
{ duration: 100, speed: 25 }
{ duration: 0, speed: 25 }
```

**Why This Output:** `??=` only assigns when the property is `null` or `undefined`. `0` is preserved.

### Real-World Cases

- **Optional parameters:** `options.timeout ??= 5000`.
- **API response handling:** `data.items ??= []`.
- **Caching:** `cache[key] ??= compute(key)`.

---

## Chained Assignment

### Definitions

**Core Definition:** Chained assignment is the practice of using multiple assignment operators in a single expression, where each assignment evaluates to the assigned value.

**Technical Definition:** Because assignment is right-associative and evaluates to the assigned value, `a = b = c = 5` parses as `a = (b = (c = 5))`. The innermost assignment (`c = 5`) evaluates to `5`, which becomes the value assigned to `b`, and so on. All three variables end up with the value `5`.

**Beginner-Friendly Explanation:** You can assign the same value to several variables at once: `a = b = c = 5` gives all three the value 5.

### Purposes

- To initialise multiple variables with the same value.
- To compactly express shared initialisation.
- To chain assignments in a single expression.

### Syntax Rules and Structure

```js
a = b = c = value
```

**Syntax Rules:**
- Right-associative: evaluates right to left.
- Each assignment expression returns the assigned value.
- In non-strict mode, undeclared variables in a chain become globals; in strict mode, they throw `ReferenceError`.
- Destructuring can be combined with chaining.

**Constraints and Limitations:**
- Chained assignment of object values creates a shared reference (aliasing hazard).
- Chained assignment of arrays creates a shared reference.
- Prefer separate assignments for clarity when initialising independent objects.

### Annotated Code Examples

#### Example 1: Basic Chaining

```js
let a, b, c;
a = b = c = 5;
console.log(a, b, c); // 5 5 5

// Equivalent to:
// c = 5;
// b = c;
// a = b;
```

**Expected Output:**
```
5 5 5
```

**Why This Output:** The rightmost assignment happens first, and its value flows leftward.

#### Example 2: Aliasing Hazard with Objects

```js
let x, y;
x = y = {};
x.name = "Alice";
console.log(y.name); // "Alice" (same object!)
console.log(x === y); // true

// Correct: separate assignments.
let p = {};
let q = {};
p.name = "Bob";
console.log(q.name); // undefined (different objects)
```

**Expected Output:**
```
Alice
true
undefined
```

**Why This Output:** Chained assignment of `{}` creates one object referenced by both `x` and `y`. To create separate objects, assign separately.

#### Example 3: Chaining with Destructuring

```js
let a, b, c;
[a, b] = [c] = [1, 2, 3];
console.log(a, b, c); // 1 2 1
```

**Expected Output:**
```
1 2 1
```

**Why This Output:** `[c] = [1, 2, 3]` assigns `1` to `c` and evaluates to `[1, 2, 3]`. Then `[a, b] = [1, 2, 3]` assigns `1` to `a` and `2` to `b`.

### Real-World Cases

- **Initialising counters:** `let x = y = z = 0;`.
- **Shared defaults:** `a = b = defaultConfig;` (with caution about aliasing).
- **Avoiding:** Prefer explicit separate assignments for objects and arrays.

---

## Destructuring Assignment

### Definitions

**Core Definition:** Destructuring assignment is a JavaScript expression that unpacks values from arrays or properties from objects into distinct variables using a syntax that mirrors array and object literals.

**Technical Definition:** Per ECMA-262 §13.15.5, destructuring assignment uses `AssignmentPattern` grammar productions. Array patterns use `[ ]` and iterate the source using the iterator protocol; object patterns use `{ }` and perform property access via `GetV`. Both support default values (`= default`), rest elements (`...rest`), nested patterns, and computed property names (`[key]: value`). Destructuring works with `let`, `const`, `var`, or as a plain assignment expression.

**Beginner-Friendly Explanation:** Destructuring lets you pull values out of arrays and objects and put them into variables in one step. Instead of writing `const a = arr[0]; const b = arr[1];`, you write `const [a, b] = arr;`.

### Purposes

- To extract multiple values from arrays and objects concisely.
- To swap variables without a temporary variable.
- To provide default values for missing properties.
- To collect remaining elements into a rest variable.
- To destructure function parameters and return values.

### Syntax Rules and Structure

**Array Destructuring:**
```js
const [a, b] = array;
const [a, , b] = array;            // skip element
const [a = 1, b = 2] = array;      // defaults
const [a, b, ...rest] = array;     // rest
```

**Object Destructuring:**
```js
const { a, b } = obj;
const { a: x, b: y } = obj;        // rename
const { a = 1, b = 2 } = obj;      // defaults
const { a, ...rest } = obj;        // rest
const { [key]: value } = obj;      // computed key
```

**Component Breakdown:**
- `[ ]`: Array pattern (uses iterator protocol).
- `{ }`: Object pattern (uses property access).
- `= default`: Default value if the extracted value is `undefined`.
- `...rest`: Collects remaining elements/properties.
- `a: x`: Renames the property to a different variable name.

**Syntax Rules:**
- In assignment (non-declaration) context, object patterns must be wrapped in parentheses: `({ a, b } = obj);`.
- Default values only apply when the value is `undefined` (not `null`).
- Rest elements must be last in the pattern.
- Nested patterns are supported: `const { a: { b } } = obj;`.
- Array patterns work on any iterable (arrays, strings, Sets, Maps, generators).

**Constraints and Limitations:**
- Destructuring `null` or `undefined` throws `TypeError`.
- Default values do not trigger on `null` (only `undefined`).
- Object rest (`...rest`) creates a new object with own enumerable properties.

### Annotated Code Examples

#### Example 1: Array Destructuring

```js
// Step 1: Basic array destructuring.
const [first, second] = [10, 20];
console.log(first, second); // 10 20

// Step 2: Skip elements.
const [, , third] = [1, 2, 3];
console.log(third); // 3

// Step 3: Default values.
const [a = 1, b = 2] = [10];
console.log(a, b); // 10 2

// Step 4: Rest elements.
const [head, ...tail] = [1, 2, 3, 4];
console.log(head); // 1
console.log(tail); // [2, 3, 4]
```

**Expected Output:**
```
10 20
3
10 2
1
[ 2, 3, 4 ]
```

**Why This Output:** Array patterns extract by position. Defaults apply for missing values. Rest collects remaining elements.

#### Example 2: Object Destructuring

```js
// Step 1: Basic object destructuring.
const { name, age } = { name: "Alice", age: 30 };
console.log(name, age); // Alice 30

// Step 2: Renaming.
const { name: userName, age: userAge } = { name: "Bob", age: 25 };
console.log(userName, userAge); // Bob 25

// Step 3: Default values.
const { city = "Unknown" } = { name: "Carol" };
console.log(city); // Unknown

// Step 4: Rest properties.
const { a, ...rest } = { a: 1, b: 2, c: 3 };
console.log(a);    // 1
console.log(rest); // { b: 2, c: 3 }
```

**Expected Output:**
```
Alice 30
Bob 25
Unknown
1
{ b: 2, c: 3 }
```

**Why This Output:** Object patterns match by property name. Renaming creates a new variable name. Defaults apply for missing properties. Rest collects remaining own enumerable properties.

#### Example 3: Swapping Variables

```js
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2 1
```

**Expected Output:**
```
2 1
```

**Why This Output:** The right-hand side creates `[2, 1]`, which is destructured into `x` and `y`, swapping their values without a temporary variable.

#### Example 4: Destructuring in Function Parameters

```js
function greet({ name, greeting = "Hello" }) {
  return `${greeting}, ${name}!`;
}

console.log(greet({ name: "Alice" }));                    // "Hello, Alice!"
console.log(greet({ name: "Bob", greeting: "Hi" }));      // "Hi, Bob!"
```

**Expected Output:**
```
Hello, Alice!
Hi, Bob!
```

**Why This Output:** The parameter is destructured, allowing direct access to `name` and `greeting` with a default.

#### Example 5: Nested Destructuring

```js
const user = {
  id: 1,
  profile: {
    name: "Alice",
    address: { city: "Paris", country: "France" },
  },
};

const { profile: { name, address: { city } } } = user;
console.log(name); // "Alice"
console.log(city); // "Paris"
```

**Expected Output:**
```
Alice
Paris
```

**Why This Output:** Nested patterns traverse the object structure in one expression.

#### Example 6: Assignment Without Declaration

```js
let a, b;
({ a, b } = { a: 1, b: 2 });
console.log(a, b); // 1 2

// Array assignment without declaration.
let x, y;
[x, y] = [10, 20];
console.log(x, y); // 10 20
```

**Expected Output:**
```
1 2
10 20
```

**Why This Output:** Object destructuring in assignment context requires parentheses to avoid being parsed as a block statement.

### Real-World Cases

- **Function parameters:** `function render({ width, height }) { ... }`.
- **API responses:** `const { data, status } = await fetch(url);`.
- **React props:** `function Button({ label, onClick }) { ... }`.
- **Import statements:** `import { useState, useEffect } from "react";`.
- **Configuration:** `const { timeout = 5000, retries = 3 } = options;`.
- **Array swapping:** `[arr[i], arr[j]] = [arr[j], arr[i]];`.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Assignment Operators — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-assignment-operators
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Destructuring Assignment — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-destructuring-assignment
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Exponentiation Operator — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-exp-operator
- MDN Web Docs – Assignment (=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Assignment
- MDN Web Docs – Addition assignment (+=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Addition_assignment
- MDN Web Docs – Subtraction assignment (-=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Subtraction_assignment
- MDN Web Docs – Multiplication assignment (*=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Multiplication_assignment
- MDN Web Docs – Division assignment (/=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Division_assignment
- MDN Web Docs – Remainder assignment (%=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Remainder_assignment
- MDN Web Docs – Exponentiation assignment (**=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Exponentiation_assignment
- MDN Web Docs – Logical AND assignment (&&=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND_assignment
- MDN Web Docs – Logical OR assignment (||=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_OR_assignment
- MDN Web Docs – Nullish coalescing assignment (??=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_assignment
- MDN Web Docs – Destructuring assignment — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
- MDN Web Docs – Expressions and operators — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_operators
- TC39 Proposal – Logical Assignment Operators — https://github.com/tc39/proposal-logical-assignment
- TC39 Proposal – Nullish Coalescing — https://github.com/tc39/proposal-nullish-coalescing