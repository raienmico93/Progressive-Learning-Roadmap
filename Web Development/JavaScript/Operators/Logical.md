# JavaScript Logical Operators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Logical operators are symbols and expressions in JavaScript used to combine or invert Boolean values, enabling conditional logic, short-circuit evaluation, and concise default-value assignment.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §13.13), logical operators are defined by the `LogicalANDExpression`, `LogicalORExpression`, and `LogicalNOTExpression` grammar productions. The `&&` and `||` operators are binary, left-associative, and perform short-circuit evaluation: they evaluate operands left-to-right and return the value of one of the operands (not necessarily a Boolean). The `!` operator is unary and returns a Boolean by applying `ToBoolean` and negating. ES2020 introduced the nullish coalescing operator (`??`, §13.13.3), which returns the right operand only when the left is `null` or `undefined`. ES2021 introduced logical assignment operators (`&&=`, `||=`, `??=`, §13.15.3), which combine logical short-circuiting with assignment. ES2020 also introduced optional chaining (`?.`, §13.3.9), which short-circuits property access or function calls when the base is `null` or `undefined`.

**Beginner-Friendly Explanation:** Logical operators are how you combine conditions in JavaScript. `&&` means "and," `||` means "or," and `!` means "not." They also have a superpower: they stop evaluating as soon as the answer is clear (called *short-circuiting*), and they return the actual value of the operand they stop at — not just `true` or `false`. Modern JavaScript adds `??` for "use this value unless it's null or undefined," `?.` for safely accessing nested properties, and `&&=`, `||=`, `??=` for conditional assignment.

### Key Characteristics

- **Non-Boolean return values:** `&&` and `||` return one of their operands, not a coerced Boolean.
- **Short-circuit evaluation:** The right operand may not be evaluated at all.
- **Left-associative:** `a && b && c` groups as `(a && b) && c`.
- **Truthy/falsy coercion:** Operands are evaluated for truthiness, not strictly as Booleans.
- **`!` always returns a Boolean:** It applies `ToBoolean` and negates.
- **`??` is nullish-specific:** Unlike `||`, it only triggers on `null` and `undefined`.
- **`?.` is nullish-specific:** It short-circuits property access when the base is `null` or `undefined`.
- **Logical assignment is conditional:** `&&=`, `||=`, `??=` only assign under specific conditions.

### Prerequisites

- Basic JavaScript syntax (variables, expressions, data types).
- Understanding of truthy and falsy values.
- Familiarity with Boolean logic (AND, OR, NOT).
- Awareness of `null` and `undefined`.
- Basic knowledge of operator precedence.

### Related Programming Areas

- **Conditional logic:** `if` statements, ternary expressions, guard clauses.
- **Default values:** Providing fallbacks for missing parameters or properties.
- **Configuration:** Merging options with defaults.
- **Data access:** Safely navigating nested object structures.
- **Functional programming:** Using logical operators for concise expressions.

### Core Concepts / Features

---

## `&&` (Logical AND)

### Definitions

**Core Definition:** The `&&` operator evaluates operands left-to-right and returns the first falsy operand, or the last operand if all are truthy.

**Technical Definition:** Per ECMA-262 §13.13.2, the production `LogicalANDExpression : LogicalANDExpression && BitwiseORExpression` is evaluated as follows: (1) Let `lref` be the result of evaluating `LogicalANDExpression`. (2) Let `lval` be `GetValue(lref)`. (3) If `ToBoolean(lval)` is `false`, return `lval`. (4) Otherwise, evaluate `BitwiseORExpression` and return its value. The operator is left-associative and performs short-circuit evaluation.

**Beginner-Friendly Explanation:** `&&` means "and." It returns the first value that is falsy (like `0`, `""`, `null`, `undefined`, `NaN`, or `false`). If all values are truthy, it returns the last one. It stops checking as soon as it finds a falsy value.

### Purposes

- To combine conditions that must all be true.
- To return a fallback value when a condition fails.
- To guard against accessing properties of `null` or `undefined`.
- To conditionally execute a function or expression.

### Syntax Rules and Structure

```js
operand1 && operand2
```

**Component Breakdown:**
- `operand1`, `operand2`: Any JavaScript expressions.
- Returns: The first falsy operand, or the last operand if all are truthy.

**Syntax Rules:**
- Left-associative: `a && b && c` is `(a && b) && c`.
- Higher precedence than `||`: `a || b && c` is `a || (b && c)`.
- Short-circuits: if `operand1` is falsy, `operand2` is not evaluated.
- Returns the original value of the operand, not a coerced Boolean.

**Constraints and Limitations:**
- Can produce surprising results when used with non-Boolean values.
- The returned value may not be a Boolean.

### Annotated Code Examples

#### Example 1: Basic AND Behavior

```js
// Step 1: All truthy → returns last operand.
console.log(true && true);        // true
console.log(1 && "hello");        // "hello"
console.log("a" && 42 && true);   // true

// Step 2: First falsy → returns that value.
console.log(false && true);       // false
console.log(0 && "hello");        // 0
console.log("" && 42);            // ""
console.log(null && "value");     // null
```

**Expected Output:**
```
true
hello
true
false
0

null
```

**Why This Output:** `&&` returns the first falsy operand it encounters, or the last operand if all are truthy. It preserves the original value and type.

#### Example 2: Short-Circuit Guard

```js
// Step 1: Guard against null access.
const user = { name: "Alice" };
const name = user && user.name;
console.log(name); // "Alice"

const noUser = null;
const safeName = noUser && noUser.name;
console.log(safeName); // null (no error)

// Step 2: Conditional function call.
function greet() { return "Hello!"; }
const shouldGreet = true;
console.log(shouldGreet && greet()); // "Hello!"
```

**Expected Output:**
```
Alice
null
Hello!
```

**Why This Output:** When `noUser` is `null` (falsy), `&&` short-circuits and returns `null` without attempting to access `.name`. This prevents a `TypeError`.

### Real-World Cases

- **Guard clauses:** `user && user.name`.
- **Conditional rendering:** `isVisible && renderComponent()`.
- **Input validation:** `email && password && submit()`.
- **Default parameters (pre-ES6):** `function greet(name) { name = name && name.trim(); }`.

---

## `||` (Logical OR)

### Definitions

**Core Definition:** The `||` operator evaluates operands left-to-right and returns the first truthy operand, or the last operand if all are falsy.

**Technical Definition:** Per ECMA-262 §13.13.3, `LogicalORExpression : LogicalORExpression || BitwiseORExpression` is evaluated as follows: (1) Let `lref` be the result of evaluating `LogicalORExpression`. (2) Let `lval` be `GetValue(lref)`. (3) If `ToBoolean(lval)` is `true`, return `lval`. (4) Otherwise, evaluate `BitwiseORExpression` and return its value. Left-associative and short-circuiting.

**Beginner-Friendly Explanation:** `||` means "or." It returns the first value that is truthy. If all values are falsy, it returns the last one. It stops checking as soon as it finds a truthy value.

### Purposes

- To combine conditions where at least one must be true.
- To provide default values for falsy variables.
- To select the first available value from a list.

### Syntax Rules and Structure

```js
operand1 || operand2
```

**Component Breakdown:**
- `operand1`, `operand2`: Any JavaScript expressions.
- Returns: The first truthy operand, or the last operand if all are falsy.

**Syntax Rules:**
- Left-associative: `a || b || c` is `(a || b) || c`.
- Lower precedence than `&&`: `a || b && c` is `a || (b && c)`.
- Short-circuits: if `operand1` is truthy, `operand2` is not evaluated.
- Returns the original value of the operand.

**Constraints and Limitations:**
- Treats all falsy values (`0`, `""`, `false`, `NaN`) as triggers for the default, which may not be desired. Use `??` for nullish-only defaults.

### Annotated Code Examples

#### Example 1: Basic OR Behavior

```js
console.log(false || true);       // true
console.log(0 || "default");      // "default"
console.log("" || 42);            // 42
console.log(null || undefined);   // undefined (last operand)
console.log(1 || 2);              // 1 (first truthy)
```

**Expected Output:**
```
true
default
42
undefined
1
```

**Why This Output:** `||` returns the first truthy operand, or the last operand if all are falsy.

#### Example 2: Default Value Pattern

```js
function greet(name) {
  name = name || "Anonymous";
  return `Hello, ${name}!`;
}

console.log(greet("Alice"));   // "Hello, Alice!"
console.log(greet(""));        // "Hello, Anonymous!"
console.log(greet(0));         // "Hello, Anonymous!"
```

**Expected Output:**
```
Hello, Alice!
Hello, Anonymous!
Hello, Anonymous!
```

**Why This Output:** `""` and `0` are falsy, so the default `"Anonymous"` is used. To preserve `0` and `""`, use `??`.

### Real-World Cases

- **Default parameters:** `function f(x) { x = x || 10; }`.
- **Fallback values:** `const port = process.env.PORT || 3000;`.
- **First available:** `const value = a || b || c;`.

---

## `!` (Logical NOT)

### Definitions

**Core Definition:** The `!` operator converts its operand to a Boolean and returns the opposite value.

**Technical Definition:** Per ECMA-262 §13.5.1, `UnaryExpression : ! UnaryExpression` evaluates by calling `ToBoolean` on the operand and returning `true` if the result is `false`, and `false` if the result is `true`. The result is always a Boolean primitive.

**Beginner-Friendly Explanation:** `!` flips a value: `!true` is `false`, `!false` is `true`. When used on non-Boolean values, it first converts them to a Boolean and then flips. Double negation (`!!`) converts any value to its Boolean equivalent.

### Purposes

- To invert a Boolean condition.
- To convert any value to its Boolean equivalent (via `!!`).
- To check for absence or invalidity.

### Syntax Rules and Structure

```js
!operand
```

**Component Breakdown:**
- `operand`: Any JavaScript expression.
- Returns: A Boolean primitive.

**Syntax Rules:**
- Right-associative: `!!x` is `!(!x)`.
- Always returns a Boolean.
- `!` has high precedence, higher than `&&` and `||`.
- Can be applied to any value, not just Booleans.

### Annotated Code Examples

#### Example 1: Basic NOT and Double Negation

```js
// Step 1: Basic negation.
console.log(!true);           // false
console.log(!false);          // true

// Step 2: Double negation for Boolean conversion.
console.log(!!"hello");       // true
console.log(!!0);             // false
console.log(!!null);          // false
console.log(!!undefined);     // false
console.log(!![]);            // true (empty array is truthy)
console.log(!!"");            // false
```

**Expected Output:**
```
false
true
true
false
false
false
true
false
```

**Why This Output:** `!` applies `ToBoolean` and negates. `!!` applies it twice, effectively converting any value to its Boolean equivalent.

#### Example 2: Checking for Absence

```js
const value = "";
if (!value) {
  console.log("Value is empty or falsy");
}

const items = [];
if (!items.length) {
  console.log("No items");
}
```

**Expected Output:**
```
Value is empty or falsy
No items
```

**Why This Output:** `!value` is `true` when `value` is falsy. `!items.length` is `true` when the array is empty.

### Real-World Cases

- **Guard clauses:** `if (!user) return;`.
- **Boolean conversion:** `const isEnabled = !!options.enabled;`.
- **Toggle logic:** `isOpen = !isOpen;`.
- **Feature detection:** `if (!window.fetch) { ... }`.

---

## Short-Circuit Evaluation

### Definitions

**Core Definition:** Short-circuit evaluation is the behaviour where `&&` and `||` stop evaluating operands as soon as the result is determined, returning the value of the operand they stop at.

**Technical Definition:** Per ECMA-262 §13.13, both `&&` and `||` evaluate their left operand first. If the result is sufficient to determine the outcome (`false` for `&&`, `true` for `||`), the right operand is not evaluated. The expression returns the value of the operand at which evaluation stopped. `??` also short-circuits: if the left operand is not `null` or `undefined`, the right operand is not evaluated.

**Beginner-Friendly Explanation:** Short-circuiting means JavaScript doesn't waste time evaluating the rest of a logical expression once the answer is clear. It also means you can use `&&` and `||` to conditionally execute code without `if` statements.

### Purposes

- To avoid unnecessary computation.
- To prevent errors (e.g., accessing properties of `null`).
- To conditionally execute expressions.
- To provide default values efficiently.

### Syntax Rules and Structure

```js
// AND: if left is falsy, return left; otherwise evaluate right.
falsy && expr     // expr not evaluated
truthy && expr    // expr evaluated

// OR: if left is truthy, return left; otherwise evaluate right.
truthy || expr    // expr not evaluated
falsy || expr     // expr evaluated

// Nullish: if left is not nullish, return left; otherwise evaluate right.
notNullish ?? expr // expr not evaluated
nullish ?? expr    // expr evaluated
```

**Constraints and Limitations:**
- Side effects in the right operand may not occur.
- Can be confusing when used for conditional execution instead of `if`.
- Return values are not always Booleans.

### Annotated Code Examples

#### Example 1: Demonstrating Short-Circuit

```js
// Step 1: Function with side effect.
function logAndReturn(value) {
  console.log("Evaluated:", value);
  return value;
}

// Step 2: AND short-circuits on falsy left.
console.log("--- AND ---");
const andResult = false && logAndReturn("right");
console.log("Result:", andResult);
// "Evaluated:" is NOT logged.

// Step 3: OR short-circuits on truthy left.
console.log("--- OR ---");
const orResult = true || logAndReturn("right");
console.log("Result:", orResult);
// "Evaluated:" is NOT logged.

// Step 4: Nullish short-circuits on non-nullish left.
console.log("--- NULLISH ---");
const nullishResult = "value" ?? logAndReturn("right");
console.log("Result:", nullishResult);
// "Evaluated:" is NOT logged.
```

**Expected Output:**
```
--- AND ---
Result: false
--- OR ---
Result: true
--- NULLISH ---
Result: value
```

**Why This Output:** In each case, the right operand's function is never called because the left operand determines the result.

#### Example 2: Conditional Execution

```js
// Step 1: Using && to conditionally call a function.
const user = { name: "Alice", isAdmin: true };
user.isAdmin && console.log("Admin access granted"); // Logged

// Step 2: Using || to provide a default.
const username = null;
const displayName = username || "Guest";
console.log(displayName); // "Guest"

// Step 3: Using ?? to preserve 0.
const count = 0;
const safeCount = count ?? 10;
console.log(safeCount); // 0
```

**Expected Output:**
```
Admin access granted
Guest
0
```

**Why This Output:** `&&` only calls `console.log` if `isAdmin` is truthy. `||` provides a default for `null`. `??` preserves `0` because it is not nullish.

### Real-World Cases

- **Conditional rendering:** `isVisible && <Component />` in React.
- **Safe property access:** `user && user.address && user.address.city`.
- **Default values:** `config.timeout || 5000`.
- **Lazy evaluation:** Avoiding expensive function calls.

---

## Logical Assignment

### Definitions

**Core Definition:** Logical assignment operators combine a logical operation with assignment, conditionally assigning the right operand to the left based on the left operand's truthiness or nullishness.

**Technical Definition:** Per ECMA-262 §13.15.3, the logical assignment operators are defined by the `AssignmentOperator` grammar: `&&=`, `||=`, and `??=`. The production `LeftHandSideExpression &&= AssignmentExpression` is evaluated as: (1) Let `lref` be the result of evaluating `LeftHandSideExpression`. (2) Let `lval` be `GetValue(lref)`. (3) If `ToBoolean(lval)` is `false`, return `lval`. (4) Otherwise, evaluate `AssignmentExpression`, call `PutValue(lref, rval)`, and return `rval`. The `||=` and `??=` operators follow analogous logic with `ToBoolean(lval)` being `true` and `IsNullish(lval)` respectively.

**Beginner-Friendly Explanation:** Logical assignment is a shortcut. `x &&= y` means "if x is truthy, set x to y." `x ||= y` means "if x is falsy, set x to y." `x ??= y` means "if x is null or undefined, set x to y." They save you from writing `if` statements.

### Purposes

- To conditionally update variables without `if` statements.
- To apply defaults only when needed.
- To avoid unnecessary assignments and side effects.
- To write concise, readable configuration code.

### Syntax Rules and Structure

```js
x &&= y    // x = x && (x = y)
x ||= y    // x = x || (x = y)
x ??= y    // x = x ?? (x = y)
```

**Component Breakdown:**
- `x`: A valid assignment target.
- `y`: Any expression.
- Returns: The new value of `x` (or the original if no assignment).

**Syntax Rules:**
- Short-circuiting: the right operand is only evaluated if the condition is met.
- Left-hand side is evaluated only once.
- `&&=` assigns when left is truthy.
- `||=` assigns when left is falsy.
- `??=` assigns when left is nullish.

**Constraints and Limitations:**
- ES2021 feature; requires modern runtime.
- Can be confusing when overused.

### Annotated Code Examples

#### Example 1: `&&=` (Logical AND Assignment)

```js
// Step 1: Assign only if truthy.
let user = { name: "Alice" };
user.name &&= user.name.toUpperCase();
console.log(user.name); // "ALICE"

let empty = "";
empty &&= "default";
console.log(empty); // "" (unchanged, empty is falsy)

// Step 2: Short-circuit avoids side effect.
let x = 0;
x &&= console.log("Never runs");
console.log(x); // 0
```

**Expected Output:**
```
ALICE

0
```

**Why This Output:** `&&=` only assigns when the left operand is truthy. `empty` is falsy, so no assignment. `x` is `0` (falsy), so the right-hand side is not evaluated.

#### Example 2: `||=` (Logical OR Assignment)

```js
// Step 1: Assign only if falsy.
let title = "";
title ||= "Untitled";
console.log(title); // "Untitled"

let duration = 50;
duration ||= 10;
console.log(duration); // 50 (truthy, unchanged)

// Step 2: Note: 0 is falsy, so it gets overwritten.
let count = 0;
count ||= 10;
console.log(count); // 10
```

**Expected Output:**
```
Untitled
50
10
```

**Why This Output:** `||=` assigns when the left operand is falsy. `""` and `0` are falsy, so they are overwritten. `50` is truthy, so it is preserved.

#### Example 3: `??=` (Nullish Coalescing Assignment)

```js
// Step 1: Assign only if nullish.
let name = null;
name ??= "Anonymous";
console.log(name); // "Anonymous"

let zero = 0;
zero ??= 10;
console.log(zero); // 0 (preserved, 0 is not nullish)

// Step 2: Configuration defaults.
function config(options) {
  options.duration ??= 100;
  options.speed ??= 25;
  return options;
}
console.log(config({ duration: 125 })); // { duration: 125, speed: 25 }
console.log(config({ duration: 0 }));   // { duration: 0, speed: 25 }
```

**Expected Output:**
```
Anonymous
0
{ duration: 125, speed: 25 }
{ duration: 0, speed: 25 }
```

**Why This Output:** `??=` only assigns when the left operand is `null` or `undefined`. `0` is preserved.

### Real-World Cases

- **Configuration merging:** `options.timeout ??= 5000`.
- **Caching:** `cache[key] ??= compute(key)`.
- **Conditional updates:** `user.name &&= sanitize(user.name)`.
- **DOM manipulation:** `element.innerHTML ||= "Default content"`.

---

## Nullish Coalescing (`??`)

### Definitions

**Core Definition:** The `??` operator returns the right operand only when the left operand is `null` or `undefined`; otherwise, it returns the left operand.

**Technical Definition:** Per ECMA-262 §13.13.3, `ShortCircuitExpression : CoalesceExpression` evaluates as: (1) Let `lref` be the result of evaluating `CoalesceExpressionHead`. (2) Let `lval` be `GetValue(lref)`. (3) If `IsNullish(lval)` is `true`, evaluate `BitwiseORExpression` and return its value. (4) Otherwise, return `lval`. Cannot be mixed with `&&` or `||` without parentheses (SyntaxError).

**Beginner-Friendly Explanation:** `??` is like `||` but smarter: it only falls back to the right side if the left side is `null` or `undefined`. If the left side is `0`, `""`, or `false`, it keeps those values.

### Purposes

- To provide defaults only when a value is truly missing (`null` or `undefined`).
- To preserve falsy values like `0` and `""`.
- To safely handle optional configuration values.

### Syntax Rules and Structure

```js
leftExpr ?? rightExpr
```

**Component Breakdown:**
- `leftExpr`: The value to check.
- `rightExpr`: The fallback value.
- Returns: `leftExpr` if not nullish, else `rightExpr`.

**Syntax Rules:**
- Short-circuits: if `leftExpr` is not nullish, `rightExpr` is not evaluated.
- Cannot be mixed with `&&` or `||` without parentheses.
- Has lower precedence than `||` but higher than the conditional operator.

**Constraints and Limitations:**
- Only triggers on `null` and `undefined`.
- Syntax error when combined with `&&` or `||` without parentheses.

### Annotated Code Examples

#### Example 1: Nullish vs Falsy

```js
// Step 1: nullish triggers default.
console.log(null ?? "default");      // "default"
console.log(undefined ?? "default"); // "default"

// Step 2: Other falsy values are preserved.
console.log(0 ?? "default");         // 0
console.log("" ?? "default");        // ""
console.log(false ?? "default");     // false
console.log(NaN ?? "default");       // NaN
```

**Expected Output:**
```
default
default
0

false
NaN
```

**Why This Output:** `??` only triggers on `null` and `undefined`. Other falsy values are returned as-is.

#### Example 2: Syntax Error with `&&` or `||`

```js
try {
  eval("null || undefined ?? 'foo'");
} catch (err) {
  console.log(err.name);    // "SyntaxError"
  console.log(err.message); // "Unexpected token '??'"
}
```

**Expected Output:**
```
SyntaxError
Unexpected token '??'
```

**Why This Output:** JavaScript requires parentheses to disambiguate precedence between `??` and `||`/`&&`.

#### Example 3: Configuration Defaults

```js
function createConfig(options) {
  return {
    timeout: options.timeout ?? 5000,
    retries: options.retries ?? 3,
    verbose: options.verbose ?? false,
  };
}

console.log(createConfig({ timeout: 0, verbose: true }));
// { timeout: 0, retries: 3, verbose: true }
```

**Expected Output:**
```
{ timeout: 0, retries: 3, verbose: true }
```

**Why This Output:** `timeout: 0` is preserved because `0` is not nullish. `retries` is missing, so the default `3` is used.

### Real-World Cases

- **API response handling:** `data.items ?? []`.
- **Configuration:** `config.timeout ?? 5000`.
- **React props:** `const title = props.title ?? "Untitled"`.
- **Environment variables:** `process.env.PORT ?? 3000`.

---

## Optional Chaining (`?.`)

### Definitions

**Core Definition:** The `?.` operator accesses an object's property or calls a function, short-circuiting to `undefined` if the base is `null` or `undefined`.

**Technical Definition:** Per ECMA-262 §13.3.9, the `OptionalExpression` grammar covers `MemberExpression OptionalChain`, `CallExpression OptionalChain`, and `OptionalExpression OptionalChain`. The `OptionalChain` production includes `?. Arguments`, `?. [ Expression ]`, and `?. IdentifierName`. When the base value is `null` or `undefined`, the chain short-circuits and evaluates to `undefined` without throwing a `TypeError`.

**Beginner-Friendly Explanation:** `?.` lets you safely access deeply nested properties without checking each level. If any part of the chain is `null` or `undefined`, the whole expression returns `undefined` instead of crashing.

### Purposes

- To safely access nested object properties.
- To safely call potentially undefined methods.
- To simplify code that would otherwise require multiple `&&` checks.
- To work with optional API responses.

### Syntax Rules and Structure

```js
obj?.prop           // property access
obj?.[expr]         // computed property access
func?.(args)        // function call
obj?.prop?.nested   // chained optional access
```

**Component Breakdown:**
- `obj`: The base value.
- `?.`: The optional chaining operator.
- `prop`: The property or expression.

**Syntax Rules:**
- Short-circuits: if the base is nullish, the rest of the chain is not evaluated.
- Can be chained: `obj?.a?.b?.c`.
- Cannot be used on undeclared variables.
- Can be combined with `??` for default values.

**Constraints and Limitations:**
- ES2020 feature; requires modern runtime.
- Does not validate that the property exists — only that the base is not nullish.
- Cannot be used on the left-hand side of an assignment.

### Annotated Code Examples

#### Example 1: Safe Property Access

```js
// Step 1: Without optional chaining.
const user = { name: "Alice" };
// const city = user.address.city; // TypeError: Cannot read properties of undefined

// Step 2: With optional chaining.
const city = user.address?.city;
console.log(city); // undefined (no error)

// Step 3: Chained access.
const data = { user: { profile: { name: "Bob" } } };
console.log(data?.user?.profile?.name); // "Bob"
console.log(data?.admin?.role);         // undefined
```

**Expected Output:**
```
undefined
Bob
undefined
```

**Why This Output:** `user.address` is `undefined`, so `?.city` short-circuits to `undefined`. The chained access safely navigates the structure.

#### Example 2: Optional Function Calls

```js
// Step 1: Safe method call.
const obj = {
  greet() { return "Hello!"; },
};
console.log(obj.greet?.());      // "Hello!"
console.log(obj.missing?.());    // undefined (no error)

// Step 2: Optional callback invocation.
function process(callback) {
  return callback?.(42);
}
console.log(process((x) => x * 2)); // 84
console.log(process());             // undefined
```

**Expected Output:**
```
Hello!
undefined
84
undefined
```

**Why This Output:** `?.()` safely calls the method if it exists. If the method is missing or the callback is undefined, it returns `undefined`.

#### Example 3: Combining with `??`

```js
const config = { server: { port: 8080 } };

// Step 1: Optional chaining + nullish coalescing.
const timeout = config?.server?.timeout ?? 5000;
console.log(timeout); // 5000

const port = config?.server?.port ?? 3000;
console.log(port);    // 8080
```

**Expected Output:**
```
5000
8080
```

**Why This Output:** `?.` safely accesses nested properties, and `??` provides a default only when the result is nullish.

### Real-World Cases

- **API responses:** `data?.user?.name`.
- **Event handlers:** `element?.addEventListener?.("click", handler)`.
- **Configuration:** `config?.database?.host ?? "localhost"`.
- **React props:** `props?.user?.avatar`.

---

## Logical Patterns

### Definitions

**Core Definition:** Logical patterns are idiomatic uses of logical operators to solve common programming problems concisely.

**Technical Definition:** Logical patterns leverage the non-Boolean return values, short-circuit evaluation, and conditional assignment capabilities of `&&`, `||`, `??`, `?.`, and logical assignment operators.

**Beginner-Friendly Explanation:** Logical patterns are "tricks" or "recipes" that experienced developers use to write cleaner code with logical operators.

### Common Patterns

#### 1. Default Value Pattern

```js
// Using || (falsy-based)
function greet(name) {
  name = name || "Anonymous";
  return `Hello, ${name}!`;
}

// Using ?? (nullish-based, preserves 0 and "")
function greet(name) {
  name = name ?? "Anonymous";
  return `Hello, ${name}!`;
}
```

#### 2. Guard Pattern

```js
// Only proceed if user exists.
user && user.doSomething();

// Only proceed if value is not nullish.
value ?? doSomething();
```

#### 3. Conditional Assignment Pattern

```js
// Assign only if currently falsy.
cache[key] ||= computeValue(key);

// Assign only if currently nullish.
config.timeout ??= 5000;
```

#### 4. Safe Navigation Pattern

```js
// Deep property access with defaults.
const city = user?.address?.city ?? "Unknown";
```

#### 5. Toggle Pattern

```js
let isOpen = false;
isOpen = !isOpen; // true
isOpen = !isOpen; // false
```

#### 6. Fallback Chain Pattern

```js
// First truthy value.
const value = a || b || c || "default";

// First non-nullish value.
const value = a ?? b ?? c ?? "default";
```

### Annotated Code Examples

#### Example 1: Configuration Merging

```js
function createServer(options = {}) {
  return {
    host: options.host ?? "localhost",
    port: options.port ?? 3000,
    timeout: options.timeout ?? 5000,
    retries: options.retries ?? 3,
    verbose: options.verbose ?? false,
  };
}

console.log(createServer({ port: 8080, timeout: 0 }));
// { host: "localhost", port: 8080, timeout: 0, retries: 3, verbose: false }
```

**Expected Output:**
```
{ host: 'localhost', port: 8080, timeout: 0, retries: 3, verbose: false }
```

**Why This Output:** `??=` or `??` preserves `0` for `timeout`. All other missing values get their defaults.

#### Example 2: Safe Nested Access with Defaults

```js
const apiResponse = {
  data: {
    user: {
      profile: { name: "Alice" },
    },
  },
};

const name = apiResponse?.data?.user?.profile?.name ?? "Anonymous";
console.log(name); // "Alice"

const email = apiResponse?.data?.user?.email ?? "no-email@example.com";
console.log(email); // "no-email@example.com"
```

**Expected Output:**
```
Alice
no-email@example.com
```

**Why This Output:** `?.` safely navigates the chain. `??` provides a default when the property is missing.

#### Example 3: Conditional Caching

```js
const cache = new Map();

function getCached(key, compute) {
  if (!cache.has(key)) {
    cache.set(key, compute());
  }
  return cache.get(key);
}

// Using ??=
function getCached(key, compute) {
  cache[key] ??= compute();
  return cache[key];
}
```

### Real-World Cases

- **React props:** `const title = props.title ?? "Default"`.
- **API clients:** `const baseUrl = config.baseUrl ?? "https://api.example.com"`.
- **Form handling:** `const name = formData.get("name")?.trim() ?? ""`.
- **State management:** `state.user ||= fetchUser()`.

---

## Default Values

### Definitions

**Core Definition:** Default values are fallback values used when a variable, parameter, or property is missing or falsy/nullish.

**Technical Definition:** JavaScript provides several mechanisms for default values: `||` (falsy-based), `??` (nullish-based), default function parameters (`function f(x = 10)`), and destructuring defaults (`const { x = 10 } = obj`). Each has different trigger conditions.

**Beginner-Friendly Explanation:** Default values are what you use when something might be missing. JavaScript gives you several ways to specify them.

### Purposes

- To provide fallbacks for missing function arguments.
- To initialise configuration options.
- To handle optional API response fields.
- To avoid `undefined` errors.

### Comparison of Default Value Mechanisms

| Mechanism | Triggers On | Preserves 0/""/false | ES Version |
|---|---|---|---|
| `\|\|` | Any falsy value | No | ES1 |
| `??` | `null` / `undefined` only | Yes | ES2020 |
| Default parameter `=` | `undefined` only | Yes | ES2015 |
| Destructuring default `=` | `undefined` only | Yes | ES2015 |

### Annotated Code Examples

#### Example 1: Function Default Parameters

```js
// Step 1: ES2015 default parameter.
function greet(name = "Anonymous") {
  return `Hello, ${name}!`;
}

console.log(greet("Alice")); // "Hello, Alice!"
console.log(greet());        // "Hello, Anonymous!"
console.log(greet(undefined)); // "Hello, Anonymous!"
console.log(greet(null));    // "Hello, null!" (null does NOT trigger default)
```

**Expected Output:**
```
Hello, Alice!
Hello, Anonymous!
Hello, Anonymous!
Hello, null!
```

**Why This Output:** Default parameters only trigger on `undefined`. `null` is a valid value.

#### Example 2: Destructuring Defaults

```js
// Step 1: Object destructuring defaults.
const { name = "Anonymous", age = 0 } = { name: "Alice" };
console.log(name); // "Alice"
console.log(age);  // 0

// Step 2: Array destructuring defaults.
const [first = 1, second = 2] = [10];
console.log(first);  // 10
console.log(second); // 2
```

**Expected Output:**
```
Alice
0
10
2
```

**Why This Output:** Destructuring defaults apply when the extracted value is `undefined`.

#### Example 3: Choosing the Right Default Mechanism

```js
// Step 1: Use || when any falsy value should trigger default.
const a = "" || "default";    // "default"
const b = 0 || 10;            // 10

// Step 2: Use ?? when only nullish should trigger default.
const c = "" ?? "default";    // ""
const d = 0 ?? 10;            // 0

// Step 3: Use default parameters for functions.
function f(x = 5) { return x; }
console.log(f(0));     // 0 (0 is a valid argument)
console.log(f());      // 5

// Step 4: Use ??= for conditional assignment.
let config = { timeout: 0 };
config.timeout ??= 5000;
console.log(config.timeout); // 0 (preserved)
```

**Expected Output:**
```
default
10

0

0
5
0
```

**Why This Output:** Each mechanism triggers under different conditions. Choose based on whether you want to preserve falsy values.

### Real-World Cases

- **Function parameters:** `function render(options = {}) { ... }`.
- **Configuration:** `const port = process.env.PORT ?? 3000`.
- **API responses:** `const items = response.data?.items ?? []`.
- **Destructuring:** `const { timeout = 5000 } = config`.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Binary Logical Operators — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-binary-logical-operators
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Logical NOT Operator — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-logical-not-operator
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Nullish Coalescing Operator — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-optional-chaining
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Optional Chaining — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-optional-chaining
- MDN Web Docs – Logical AND (&&) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND
- MDN Web Docs – Logical OR (||) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_OR
- MDN Web Docs – Logical NOT (!) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_NOT
- MDN Web Docs – Nullish coalescing operator (??) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing
- MDN Web Docs – Optional chaining (?.) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining
- MDN Web Docs – Logical AND assignment (&&=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND_assignment
- MDN Web Docs – Logical OR assignment (||=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_OR_assignment
- MDN Web Docs – Nullish coalescing assignment (??=) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_assignment
- MDN Web Docs – Default parameters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters
- MDN Web Docs – Destructuring assignment — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
- MDN Web Docs – Operator precedence — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_precedence
- V8 Blog – Logical assignment — https://v8.dev/features/logical-assignment
- TC39 Proposal – Optional Chaining — https://github.com/tc39/proposal-optional-chaining
- TC39 Proposal – Nullish Coalescing — https://github.com/tc39/proposal-nullish-coalescing
- TC39 Proposal – Logical Assignment — https://github.com/tc39/proposal-logical-assignment