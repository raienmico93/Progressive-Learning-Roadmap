# JavaScript Hoisting — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Hoisting is JavaScript's default behaviour of moving declarations to the top of their containing scope during the compilation phase, before any code is executed.

**Technical Definition:** Hoisting is **not** a term normatively defined in the ECMAScript specification. The spec does define a group of declarations as `HoistableDeclaration`, but this only includes `function`, `function*`, `async function`, and `async function*` declarations. Hoisting is often considered a feature of `var` declarations as well, although in a different way. In colloquial terms, any of the following behaviours may be regarded as hoisting: being able to use a variable's value in its scope before the line it is declared ("Value hoisting"); being able to reference a variable in its scope before the line it is declared, without throwing a `ReferenceError`, but the value is always `undefined` ("Declaration hoisting"); the declaration of the variable causes behaviour changes in its scope before the line in which it is declared; the side effects of a declaration are produced before evaluating the rest of the code that contains it.

**Beginner-Friendly Explanation:** Hoisting is JavaScript's way of "lifting up" declarations to the top of their scope before running your code. It's like the JavaScript engine reads through your code first, notes all the variables and functions you're going to create, and prepares space for them — before actually running anything. This is why you can sometimes use a function before you've written it, or why a variable might be `undefined` instead of causing an error.

### Key Characteristics

- **Not a formal spec term:** Hoisting emerges from declaration processing rules, not a defined feature.
- **Four types of hoisting behaviour** (per MDN): value hoisting, declaration hoisting, scope-tainting hoisting, and side-effect hoisting.
- **Function declarations are hoisted first:** Functions are given higher priority than `var` statements; later `var` statements cannot overwrite a previously-handled function declaration.
- **`var` is hoisted and initialised to `undefined`**.
- **`let`, `const`, and `class` are hoisted but not initialised** — they exist in the Temporal Dead Zone (TDZ).
- **Function expressions are not hoisted** (only their variable name is, if assigned to `var`).
- **Class declarations are hoisted but not initialised** — same TDZ rules as `let`/`const`.

### Prerequisites

- Basic JavaScript syntax (variables, functions, blocks).
- Understanding of scope (global, function, block).
- Familiarity with the `var`, `let`, and `const` keywords.
- Awareness of the execution context and compilation phase.

### Related Programming Areas

- **Scope:** Hoisting interacts directly with scope rules.
- **Closures:** Hoisting affects how closures capture variables.
- **Execution context:** The compilation phase processes declarations.
- **Temporal Dead Zone:** The period before `let`/`const` initialisation.
- **Best practices:** Avoiding hoisting-related bugs.

### Core Concepts / Features

---

## Hoisting

### Definitions

**Core Definition:** Hoisting is the apparent movement of declarations (variables, functions, classes, imports) to the top of their containing scope before code execution.

**Technical Definition:** The ECMAScript specification does not normatively define hoisting. Instead, the behaviour emerges from how execution contexts are initialised. At the start of a function call (or script/module evaluation), the Lexical Environment is initialised with all identifiers contained immediately within it. Function declarations are instantiated and fully initialised; `var` declarations are instantiated and initialised to `undefined`; lexical declarations (`let`, `const`, `class`) are instantiated but remain uninitialised until their declaration is evaluated.

**Beginner-Friendly Explanation:** Hoisting is like the JavaScript engine reading through your code and making a "list" of everything you're going to declare. It then processes that list before running any actual statements. Functions are ready to use immediately; `var` variables start as `undefined`; `let`/`const` variables are "locked" until their declaration line.

### Purposes (Why Hoisting Exists)

- To allow function declarations to be called before their textual definition (supporting mutual recursion and code organisation).
- To enable variables to be referenced throughout their scope, even if declared later.
- To simplify the language's execution model by processing declarations in a predictable order.

### Syntax Rules and Structure

Hoisting is not written explicitly; it is an implicit behaviour. The order of processing is:

1. **Function declarations** (`function`, `function*`, `async function`, `async function*`): Hoisted first and fully initialised.
2. **`var` declarations:** Hoisted second and initialised to `undefined`.
3. **Lexical declarations** (`let`, `const`, `class`): Hoisted third but **not initialised** (TDZ begins).
4. **Import declarations:** Hoisted with value-hoisting behaviour.

### Annotated Code Examples

#### Example 1: Function Declaration Hoisting

```js
// Step 1: Call the function before its declaration.
console.log(greet("Alice")); // "Hello, Alice!"

// Step 2: Function declaration (hoisted).
function greet(name) {
  return `Hello, ${name}!`;
}
```

**Expected Output:**
```
Hello, Alice!
```

**Why This Output:** Function declarations are fully hoisted, so `greet` is available before its textual position.

#### Example 2: `var` Hoisting

```js
// Step 1: Reference before declaration.
console.log(x); // undefined

// Step 2: Declaration (hoisted, initialised to undefined).
var x = 5;
console.log(x); // 5
```

**Expected Output:**
```
undefined
5
```

**Why This Output:** The `var x` declaration is hoisted and initialised to `undefined`. The assignment `x = 5` happens at its original position.

#### Example 3: `let`/`const` Hoisting (TDZ)

```js
// Step 1: Attempt to access before declaration.
try {
  console.log(y); // ReferenceError
} catch (err) {
  console.log(err.name); // "ReferenceError"
}

// Step 2: Declaration (hoisted, but TDZ blocks access until here).
let y = 10;
console.log(y); // 10
```

**Expected Output:**
```
ReferenceError
10
```

**Why This Output:** `let y` is hoisted but not initialised. Accessing it before its declaration throws a `ReferenceError` due to the Temporal Dead Zone.

### Real-World Cases

- **Mutual recursion:** Two functions calling each other, with one defined before the other.
- **Utility functions:** Calling helper functions defined later in the file.
- **Legacy code:** Understanding why old code works despite "backwards" references.

---

## Variable Hoisting

### Definitions

**Core Definition:** Variable hoisting is the behaviour where variable declarations are processed before code execution, with different initialisation behaviour depending on the declaration keyword.

**Technical Definition:** Per the ECMAScript specification, when a Lexical Environment is created (for a script, module, or function), its Environment Record is populated with bindings for all declarations within that environment. `var` declarations create bindings initialised to `undefined`. `let` and `const` declarations create bindings that are uninitialised (in the TDZ) until their declaration is evaluated.

**Beginner-Friendly Explanation:** When JavaScript prepares to run your code, it creates "slots" for all your variables. For `var`, the slot starts with the value `undefined`. For `let` and `const`, the slot exists but is "locked" until the declaration line runs.

### Purposes

- To allow variables to be declared anywhere in their scope without causing "not defined" errors.
- To provide a consistent initialisation model for `var` (undefined) versus lexical declarations (TDZ).

### Syntax Rules and Structure

```js
// var: hoisted and initialised to undefined
console.log(a); // undefined
var a = 1;

// let/const: hoisted but not initialised (TDZ)
// console.log(b); // ReferenceError
let b = 2;

// Function declaration: fully hoisted
// function foo() {} is available before its line
```

**Constraints and Limitations:**
- `var` hoisting can mask typos (using an undeclared variable returns `undefined` instead of throwing).
- `let`/`const` TDZ prevents access before declaration, which is safer but can surprise developers.

### Annotated Code Examples

#### Example 1: `var` Hoisting with Function Scope

```js
function example() {
  console.log(hoisted); // undefined
  if (true) {
    var hoisted = "value";
  }
  console.log(hoisted); // "value"
}
example();
```

**Expected Output:**
```
undefined
value
```

**Why This Output:** `var hoisted` is hoisted to the top of the function scope (not the block), so it exists throughout the function.

#### Example 2: `let` Hoisting with Block Scope

```js
{
  // console.log(blockVar); // ReferenceError
  let blockVar = "block-scoped";
  console.log(blockVar); // "block-scoped"
}
// console.log(blockVar); // ReferenceError
```

**Expected Output:**
```
block-scoped
```

**Why This Output:** `let` is hoisted to the top of the block but remains in the TDZ until its declaration line.

### Real-World Cases

- **Loop variables:** `var` leaks out of loops; `let` is scoped to the loop.
- **Conditional declarations:** Understanding why `var` inside `if` is function-scoped.
- **Debugging:** Recognising `undefined` as a hoisting symptom.

---

## Function Hoisting

### Definitions

**Core Definition:** Function hoisting is the behaviour where function declarations are fully hoisted (name and body) to the top of their scope, making them callable before their textual definition.

**Technical Definition:** Per ECMA-262, `FunctionDeclaration` is a `HoistableDeclaration`. During environment initialisation, function declarations are instantiated by creating a function object and binding the name to it. The entire function definition (not just the name) is hoisted. Function expressions, however, are not hoisted — only the variable name they are assigned to is hoisted (if declared with `var`).

**Beginner-Friendly Explanation:** Function declarations are like "ready-to-use" tools — you can use them before you've written them down. Function expressions (where you assign a function to a variable) are not hoisted in the same way.

### Purposes

- To allow function declarations to be called before their textual definition.
- To support mutual recursion (two functions calling each other).
- To enable code organisation where helper functions are defined at the bottom.

### Syntax Rules and Structure

```js
// Function declaration: fully hoisted
console.log(add(2, 3)); // 5
function add(a, b) { return a + b; }

// Function expression: NOT fully hoisted
// console.log(subtract(5, 3)); // TypeError: subtract is not a function
var subtract = function(a, b) { return a - b; };
```

**Syntax Rules:**
- Function declarations are hoisted first, before `var` declarations.
- Function expressions are not hoisted (only the variable name is, if using `var`).
- Arrow functions assigned to variables follow the same rules as function expressions.
- Functions declared inside blocks (in strict mode) are block-scoped and hoisted to the top of that block.

**Constraints and Limitations:**
- Function declarations inside conditional blocks have inconsistent behaviour across environments.
- Function expressions are not callable before their assignment.

### Annotated Code Examples

#### Example 1: Function Declaration Hoisting

```js
// Step 1: Call before definition.
console.log(multiply(3, 4)); // 12

// Step 2: Definition.
function multiply(a, b) {
  return a * b;
}
```

**Expected Output:**
```
12
```

**Why This Output:** The function declaration is fully hoisted, so `multiply` is available before its line.

#### Example 2: Function Expression Not Hoisted

```js
// Step 1: Attempt to call before assignment.
try {
  console.log(divide(10, 2));
} catch (err) {
  console.log(err.name);    // "TypeError"
  console.log(err.message); // "divide is not a function"
}

// Step 2: Function expression.
var divide = function(a, b) {
  return a / b;
};

// Step 3: Now it works.
console.log(divide(10, 2)); // 5
```

**Expected Output:**
```
TypeError
divide is not a function
5
```

**Why This Output:** `var divide` is hoisted as `undefined`. Attempting to call `undefined` throws `TypeError`. After assignment, the function works.

#### Example 3: Functions Hoisted Before `var`

```js
console.log(typeof foo); // "function"
console.log(typeof bar); // "undefined"

var bar = "variable";
function foo() { return "function"; }
```

**Expected Output:**
```
function
undefined
```

**Why This Output:** Function declarations are hoisted before `var` declarations. `foo` is fully available; `bar` is `undefined`.

### Real-World Cases

- **Utility modules:** Functions defined at the bottom, called at the top.
- **Event handlers:** Named functions used before definition.
- **Recursion:** Functions calling themselves or each other.

---

## `var` Hoisting

### Definitions

**Core Definition:** `var` hoisting is the behaviour where `var` declarations are hoisted to the top of their enclosing function (or global) scope and initialised to `undefined`.

**Technical Definition:** Per ECMA-262, `var` declarations are instantiated during environment initialisation with the value `undefined`. The assignment part of the declaration is **not** hoisted; only the declaration itself is.

**Beginner-Friendly Explanation:** When you write `var x = 5`, JavaScript sees `var x` and moves that to the top of the function, initialising it to `undefined`. The `= 5` part stays where it is.

### Purposes

- To provide a predictable initial state for all `var`-declared variables.
- To allow `var` variables to be referenced throughout their function scope.

### Syntax Rules and Structure

```js
// What you write:
console.log(x); // undefined
var x = 5;

// What JavaScript "sees":
var x;          // hoisted to top, initialised to undefined
console.log(x); // undefined
x = 5;          // assignment stays in place
```

**Constraints and Limitations:**
- `var` ignores block scope; it leaks out of `if`, `for`, and `while` blocks.
- `var` can be re-declared in the same scope without error.
- `var` hoisting masks typos and undeclared variables.

### Annotated Code Examples

#### Example 1: `var` Hoisting in Action

```js
console.log(a); // undefined
var a = 10;
console.log(a); // 10
```

**Expected Output:**
```
undefined
10
```

**Why This Output:** `var a` is hoisted and initialised to `undefined`. The assignment happens later.

#### Example 2: `var` Leaks from Blocks

```js
if (true) {
  var leaked = "I escape!";
}
console.log(leaked); // "I escape!"
```

**Expected Output:**
```
I escape!
```

**Why This Output:** `var` ignores block scope and binds to the enclosing function or global scope.

### Real-World Cases

- **Legacy code:** Pre-ES6 code relied heavily on `var`.
- **Global variables:** `var` at the top level creates properties on `window`.
- **Loop counters:** `var i` in loops leaks to the surrounding function.

---

## `let` and `const` Hoisting

### Definitions

**Core Definition:** `let` and `const` declarations are hoisted but **not initialised**; they exist in the Temporal Dead Zone from the start of their block until their declaration is evaluated.

**Technical Definition:** Per ECMA-262, `let` and `const` declarations create bindings in the Lexical Environment that are initially uninitialised. Accessing an uninitialised binding throws a `ReferenceError`. The binding is initialised when the declaration statement is evaluated.

**Beginner-Friendly Explanation:** `let` and `const` variables exist from the start of their block, but you can't touch them until the declaration line runs. It's like a "Do Not Touch" sign that gets removed at the right moment.

### Purposes

- To prevent access to variables before they are declared (safer than `var`).
- To enforce block scoping (variables don't leak out of blocks).
- To support `const` immutability from the point of declaration.

### Syntax Rules and Structure

```js
{
  // TDZ begins here
  // console.log(x); // ReferenceError
  let x = 5;         // TDZ ends here
  console.log(x);    // 5
}

// console.log(y); // ReferenceError
const y = 10;
```

**Syntax Rules:**
- Both `let` and `const` are block-scoped.
- Both are hoisted but remain uninitialised until their declaration.
- `const` must be initialised at declaration (cannot be declared without a value).
- Re-declaration in the same scope is a `SyntaxError`.

**Constraints and Limitations:**
- TDZ errors can be surprising to developers coming from `var`.
- `typeof` on a TDZ variable throws `ReferenceError` (unlike `var`, where `typeof` on an undeclared variable is safe).

### Annotated Code Examples

#### Example 1: TDZ with `let`

```js
{
  try {
    console.log(blockLet);
  } catch (err) {
    console.log(err.name); // "ReferenceError"
  }
  let blockLet = "value";
  console.log(blockLet); // "value"
}
```

**Expected Output:**
```
ReferenceError
value
```

**Why This Output:** `blockLet` is in the TDZ until its declaration line.

#### Example 2: TDZ with `const`

```js
{
  try {
    console.log(PI);
  } catch (err) {
    console.log(err.name); // "ReferenceError"
  }
  const PI = 3.14159;
  console.log(PI); // 3.14159
}
```

**Expected Output:**
```
ReferenceError
3.14159
```

**Why This Output:** `const PI` is in the TDZ until its declaration.

#### Example 3: `typeof` in TDZ

```js
{
  try {
    console.log(typeof x); // ReferenceError (unlike var)
  } catch (err) {
    console.log(err.name); // "ReferenceError"
  }
  let x = 5;
}
```

**Expected Output:**
```
ReferenceError
```

**Why This Output:** Unlike `var`, `typeof` on a `let` variable in the TDZ throws `ReferenceError`.

### Real-World Cases

- **Modern JavaScript:** `let` and `const` are the default choices.
- **Block scoping:** Preventing variable leakage in loops and conditionals.
- **Immutability:** Using `const` for values that shouldn't change.

---

## Temporal Dead Zone (TDZ)

### Definitions

**Core Definition:** The Temporal Dead Zone (TDZ) is the period of time during which a `let` or `const` variable cannot be accessed because it has been hoisted but not yet initialised.

**Technical Definition:** Per ECMA-262, `let` and `const` bindings are created during environment initialisation but remain uninitialised. The TDZ begins when the code execution enters the block containing the declaration and continues until the declaration has been evaluated. Accessing a binding in the TDZ throws a `ReferenceError`. The TDZ is temporal, not spatial — it's about when you access the variable, not where in the code you are.

**Beginner-Friendly Explanation:** The TDZ is like a "waiting period." When JavaScript enters a block, it knows a `let` or `const` variable is coming, but it's not ready yet. If you try to use it before its declaration line, you get an error. Once the declaration runs, the variable is "unlocked."

### Purposes

- To prevent use-before-declaration bugs that `var` allows.
- To enforce a clear declaration-before-use pattern.
- To make code more predictable and safer.

### Syntax Rules and Structure

```js
{
  // TDZ for x begins here
  // console.log(x); // ReferenceError
  // let x = 5;      // TDZ for x ends here
  console.log(x);    // 5
}
```

**Constraints and Limitations:**
- The TDZ applies to `let`, `const`, and `class` declarations.
- `typeof` on a TDZ variable throws `ReferenceError` (unlike `var`).
- The TDZ can surprise developers familiar with `var`.

### Annotated Code Examples

#### Example 1: TDZ in Action

```js
{
  console.log("Start");
  // console.log(x); // ReferenceError: Cannot access 'x' before initialization
  let x = 10;
  console.log("x =", x); // "x = 10"
}
```

**Expected Output:**
```
Start
x = 10
```

**Why This Output:** The `console.log(x)` before the `let` declaration would throw; after the declaration, `x` is accessible.

#### Example 2: TDZ with Block Shadowing

```js
const x = 1;
{
  try {
    console.log(x); // ReferenceError (TDZ, not outer x)
  } catch (err) {
    console.log(err.name); // "ReferenceError"
  }
  const x = 2;
  console.log(x); // 2
}
```

**Expected Output:**
```
ReferenceError
2
```

**Why This Output:** The inner `const x` creates a TDZ for the entire block, shadowing the outer `x` even before the declaration line.

### Real-World Cases

- **Debugging:** Understanding why a `ReferenceError` occurs despite an outer variable existing.
- **Code review:** Ensuring declarations precede usage.
- **Migration:** Moving from `var` to `let`/`const` reveals TDZ issues.

---

## Hoisting Pitfalls

### Definitions

**Core Definition:** Hoisting pitfalls are the common bugs, surprises, and confusions caused by JavaScript's hoisting behaviour.

**Technical Definition:** Pitfalls arise from the interaction between hoisting and scope, particularly when `var` leaks out of blocks, when function expressions are mistaken for function declarations, and when the TDZ causes unexpected `ReferenceError`s.

**Beginner-Friendly Explanation:** Hoisting can trick you. Code that looks correct might print `undefined` instead of a value, or a function might not be available when you expect it.

### Common Pitfalls

#### 1. `var` Leaking from Blocks

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// Output: 3, 3, 3 (not 0, 1, 2)
```

**Why:** `var i` is function-scoped, so all closures share the same `i`, which is `3` after the loop.

#### 2. Function Expression vs Declaration

```js
// Function declaration: hoisted
console.log(declared()); // "declared"
function declared() { return "declared"; }

// Function expression: not hoisted
try {
  console.log(expressed()); // TypeError
} catch (err) {
  console.log(err.name); // "TypeError"
}
var expressed = function() { return "expressed"; };
```

#### 3. Function Overwriting

```js
function greet() { return "first"; }
function greet() { return "second"; }
console.log(greet()); // "second" (last declaration wins)
```

#### 4. TDZ Surprises

```js
let x = 1;
{
  try {
    console.log(x); // ReferenceError (inner x in TDZ)
  } catch (err) {
    console.log(err.name); // "ReferenceError"
  }
  let x = 2;
}
```

### Annotated Code Examples

#### Example 1: `var` Loop Pitfall

```js
// BAD: var leaks
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log("var:", i), 0);
}
// Output: var: 3, var: 3, var: 3

// GOOD: let is block-scoped
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log("let:", j), 0);
}
// Output: let: 0, let: 1, let: 2
```

**Expected Output:**
```
var: 3
var: 3
var: 3
let: 0
let: 1
let: 2
```

**Why This Output:** `var` is function-scoped, so all timeouts capture the same final `i`. `let` creates a new binding per iteration.

#### Example 2: Function Declaration in Block

```js
// Behaviour varies across environments in non-strict mode.
if (true) {
  function blockFunction() { return "block"; }
}
// console.log(blockFunction()); // Works in some browsers, not others
```

**Note:** In strict mode, function declarations inside blocks are block-scoped.

### Real-World Cases

- **Event handlers in loops:** The classic `var` loop bug.
- **Conditional function definitions:** Function declarations inside `if` blocks.
- **Hoisting confusion:** Debugging `undefined` values.

---

## Hoisting Best Practices

### Definitions

**Core Definition:** Hoisting best practices are the coding conventions and patterns that prevent hoisting-related bugs and make code more predictable.

**Technical Definition:** Best practices include declaring variables at the top of their scope, using `let`/`const` instead of `var`, using strict mode, and preferring function declarations for hoisted functions and function expressions for conditionally assigned functions.

**Beginner-Friendly Explanation:** Follow these rules to avoid hoisting surprises: declare variables at the top, use `let` and `const`, and be careful with function expressions.

### Best Practices

1. **Declare variables at the top of their scope**

```js
// GOOD
function process() {
  var result;
  var count;
  // ... logic ...
}

// BAD
function process() {
  // ... logic ...
  var result = compute();
}
```

2. **Use `let` and `const` instead of `var`**

```js
// GOOD
const PI = 3.14159;
let count = 0;

// BAD
var PI = 3.14159;
var count = 0;
```

3. **Use `const` by default, `let` when reassignment is needed**

```js
const MAX_SIZE = 100;      // Never reassigned
let currentSize = 0;       // Reassigned later
```

4. **Enable strict mode**

```js
"use strict";
// Undeclared assignments now throw ReferenceError.
```

5. **Be explicit with function expressions**

```js
// GOOD: Function expression declared before use.
const greet = function(name) { return `Hello, ${name}`; };
console.log(greet("Alice"));

// AVOID: Relying on function declaration hoisting when the function is
// conditionally defined.
```

6. **Use function declarations for top-level utilities**

```js
// Function declarations are hoisted and can be called anywhere.
function formatDate(date) { /* ... */ }
```

7. **Avoid function declarations inside blocks**

```js
// GOOD: Use function expressions inside blocks.
if (condition) {
  const handler = () => { /* ... */ };
}

// AVOID: Function declarations inside blocks (inconsistent behaviour).
```

8. **Use ESLint rules to enforce hoisting best practices**

- `vars-on-top`: Requires `var` declarations at the top of their scope.
- `no-use-before-define`: Disallows use before definition.
- `no-var`: Requires `let` or `const` instead of `var`.
- `no-shadow`: Prevents variable shadowing.

### Annotated Code Examples

#### Example 1: Best Practice — Declare at Top

```js
function calculateTotal(items) {
  // Declarations at the top.
  let subtotal = 0;
  let tax = 0;
  let total = 0;

  for (const item of items) {
    subtotal += item.price;
  }
  tax = subtotal * 0.1;
  total = subtotal + tax;
  return total;
}

console.log(calculateTotal([{ price: 10 }, { price: 20 }])); // 33
```

**Expected Output:**
```
33
```

**Why This Output:** Declarations are at the top, making the function's scope clear and avoiding hoisting surprises.

#### Example 2: Best Practice — `let`/`const` Over `var`

```js
// GOOD: let/const with block scope.
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// Output: 0, 1, 2

// BAD: var leaks and causes shared state.
for (var j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}
// Output: 3, 3, 3
```

**Expected Output:**
```
0
1
2
3
3
3
```

**Why This Output:** `let` creates a new binding per iteration; `var` shares one binding across all closures.

### Real-World Cases

- **Modern JavaScript projects:** `let`/`const` are the standard.
- **Code reviews:** Hoisting best practices are common review criteria.
- **Linting:** ESLint enforces hoisting-related rules.

---

## References

- MDN Web Docs – Hoisting — https://developer.mozilla.org/en-US/docs/Glossary/Hoisting
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – HoistableDeclaration — https://tc39.es/ecma262/multipage/ecmascript-language-statements-and-declarations.html#prod-HoistableDeclaration
- MDN Web Docs – var — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var
- MDN Web Docs – let — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let
- MDN Web Docs – const — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const
- MDN Web Docs – function — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function
- MDN Web Docs – Temporal Dead Zone — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz
- MDN Web Docs – Scope — https://developer.mozilla.org/en-US/docs/Glossary/Scope
- MDN Web Docs – Closures — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures
- Exploring JS – Hoisting — https://exploringjs.com/impatient-js/ch_variables.html#hoisting
- ESLint – vars-on-top — https://eslint.org/docs/latest/rules/vars-on-top
- ESLint – no-use-before-define — https://eslint.org/docs/latest/rules/no-use-before-define
- ESLint – no-var — https://eslint.org/docs/latest/rules/no-var
- ESLint – no-shadow — https://eslint.org/docs/latest/rules/no-shadow
- You Don't Know JS – Scope & Closures — https://github.com/getify/You-Dont-Know-JS
- W3Schools – JavaScript Hoisting — https://www.w3schools.com/js/js_hoisting.asp
- Stack Overflow – Order of hoisting in JavaScript — https://stackoverflow.com/questions/28246589/order-of-hoisting-in-javascript
- ECMAScript 2015 Temporal Dead Zone — https://stackoverflow.com/questions/36444913/ecmascript-2015-temporal-dead-zone