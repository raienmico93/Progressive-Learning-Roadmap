# JavaScript Scope — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Scope is the current context of execution in which values and expressions are "visible" or can be referenced. It determines which variables and functions are accessible from any given point in a program.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §8.1), scope is modelled through **Lexical Environments**, which are specification types used to define the association of Identifiers to specific variables and functions based upon the lexical nesting structure of ECMAScript code. A Lexical Environment consists of an **Environment Record** that maps identifiers to values and an optional **outer environment reference** that links to the enclosing lexical environment. Identifier resolution is the process of determining the binding of an Identifier by searching the scope chain from the innermost environment outward.

**Beginner-Friendly Explanation:** Scope is like the "visibility rule" of your code. It answers the question: "Given a variable name, which value does it refer to?" In JavaScript, where you write your code determines what variables you can see. A variable declared inside a function is only visible inside that function. A variable declared at the top of your file is visible everywhere in that file. JavaScript uses **lexical (static) scoping**, meaning the scope is determined by where code is written, not by where it is called from.

### Key Characteristics

- **Lexical (static):** Scope is determined at author time by the physical structure of the code.
- **Hierarchical:** Scopes nest, forming a chain from innermost to outermost (global).
- **Not dynamic:** JavaScript does **not** use dynamic scoping; a function's scope does not depend on its caller.
- **Four main kinds:** Global, Function, Block, and Module scope.
- **Chain resolution:** Name resolution walks up the scope chain until a match is found or the global scope is reached.
- **Determined by declaration:** `var` is function-scoped; `let`, `const`, `class`, and `function` (in strict mode) are block-scoped.
- **Hoisting:** Declarations are processed before execution, affecting when variables become accessible.

### Prerequisites

- Basic JavaScript syntax (variables, functions, blocks).
- Understanding of data types and references.
- Familiarity with functions and function calls.
- Awareness of the global object (`window` in browsers, `globalThis`).

### Related Programming Areas

- **Closures:** Functions that capture their lexical environment.
- **Modules:** Encapsulation through file-level scope.
- **Hoisting:** Declaration processing before execution.
- **Memory management:** Scope determines lifetime of variables.
- **Security:** Scope isolation prevents unintended access.

### Core Concepts / Features

---

## Global Scope

### Definitions

**Core Definition:** Global scope is the outermost scope in a JavaScript program, where variables declared are accessible from anywhere in the program.

**Technical Definition:** Per ECMA-262 §9.1.1.4, the global environment is a unique Lexical Environment created before any ECMAScript code is executed. In script mode, top-level `var` declarations become properties of the global object (`window` in browsers, `globalThis` in all environments), while top-level `let` and `const` declarations create bindings in the global environment record but do not become properties of the global object.

**Beginner-Friendly Explanation:** Global scope is the "outermost layer" of your program. Anything declared at the top level of a script is global and can be accessed from any other part of that script. In a browser, global `var` variables become properties of the `window` object.

### Purposes

- To provide a shared namespace for values needed across the entire program.
- To allow built-in objects (`Math`, `JSON`, `console`) to be accessible everywhere.
- To enable configuration constants and application-wide state.

### Syntax Rules and Structure

```js
// Top-level var (becomes window property in browsers)
var globalVar = "I am global";

// Top-level let/const (global binding, not a window property)
let globalLet = "I am a global binding";
const globalConst = "I am also global";

// Function declarations at top level
function globalFunction() { /* ... */ }
```

**Syntax Rules:**
- In script mode, top-level `var` becomes a property of the global object.
- Top-level `let` and `const` are global bindings but **not** properties of the global object.
- In module mode, top-level declarations are **not** global; they are scoped to the module.
- Implicit globals (undeclared assignments) create global variables in non-strict mode but throw `ReferenceError` in strict mode.

**Constraints and Limitations:**
- Global variables persist for the lifetime of the page/application.
- Global namespace pollution can cause name collisions.
- Global variables are slower to access than local variables in some engines.

### Annotated Code Examples

#### Example 1: Global Scope Access

```js
// Step 1: Declare a global variable with var.
var appName = "MyApp";

// Step 2: Declare a global with let.
let version = "1.0.0";

// Step 3: Access from a function.
function showInfo() {
  console.log(appName); // "MyApp" — accessible
  console.log(version); // "1.0.0" — accessible
}

showInfo();

// Step 4: Verify var becomes a window property.
console.log(window.appName); // "MyApp"
console.log(window.version); // undefined (let is not a window property)
```

**Expected Output:**
```
MyApp
1.0.0
MyApp
undefined
```

**Why This Output:** `var` at the top level creates a property on the global object, while `let` creates a global binding that is not a property of `window`.

#### Example 2: Implicit Global Leakage

```js
// Step 1: Assignment without declaration (non-strict mode).
function leaky() {
  leakedVariable = "I escaped!"; // No var/let/const — becomes global!
}

leaky();
console.log(leakedVariable); // "I escaped!"
console.log(window.leakedVariable); // "I escaped!"

// Step 2: In strict mode, this throws.
"use strict";
function safe() {
  // strictVariable = "error"; // ReferenceError: strictVariable is not defined
}
```

**Expected Output:**
```
I escaped!
I escaped!
```

**Why This Output:** In non-strict mode, undeclared assignments create global variables. This is called **variable leakage** and is a form of scope pollution.

### Real-World Cases

- **Application configuration:** `const API_BASE_URL = "https://api.example.com";`.
- **Feature flags:** `window.FEATURE_ENABLED = true;`.
- **Third-party libraries:** jQuery uses `window.jQuery` and `window.$`.
- **Polyfills:** Adding missing methods to `window` for legacy browser support.

---

## Function Scope

### Definitions

**Core Definition:** Function scope is the scope created by a function, where parameters and variables declared with `var` (or function declarations) are accessible within that function and its nested functions.

**Technical Definition:** Per ECMA-262 §9.1.1.5, a function environment is a Lexical Environment that corresponds to the invocation of an ECMAScript function object. Each function call creates a fresh function environment record containing bindings for parameters and `var`-declared variables. Function declarations are also scoped to the enclosing function.

**Beginner-Friendly Explanation:** Every time you call a function, JavaScript creates a new "private room" for that function's variables. Variables declared with `var` inside the function are only visible inside that function — not outside it.

### Purposes

- To encapsulate variables so they don't interfere with other parts of the program.
- To create private state within functions.
- To enable recursion and closures.
- To limit the lifetime of temporary variables.

### Syntax Rules and Structure

```js
function myFunction(param1, param2) {
  // Function scope begins here
  var localVar = "only visible here";
  let blockVar = "also visible here";
  const constant = "also visible here";

  function innerFunction() {
    // Can access all of the above
  }

  // Function scope ends here
}
// localVar, blockVar, constant are NOT accessible here
```

**Syntax Rules:**
- Parameters are function-scoped.
- `var` declarations are function-scoped (hoisted to the top of the function).
- `let` and `const` declarations are block-scoped, but when declared at the top level of a function body, they are effectively function-scoped.
- Function declarations inside a function are scoped to that function.
- Each function call creates a new scope; closures capture the scope of their defining function.

**Constraints and Limitations:**
- `var` hoisting can lead to unexpected `undefined` values.
- Function scope is created per call, not per definition.
- Arrow functions do not have their own `arguments` object but do have their own scope for parameters and local variables.

### Annotated Code Examples

#### Example 1: Function Scope Isolation

```js
// Step 1: Global variable.
const message = "global";

function showMessage() {
  // Step 2: Local variable shadows global.
  const message = "local";
  console.log(message); // "local"
}

showMessage();
console.log(message); // "global" (unchanged)
```

**Expected Output:**
```
local
global
```

**Why This Output:** The `message` inside `showMessage` is a new binding in the function's scope, shadowing the global `message`. The global is unaffected.

#### Example 2: `var` Hoisting Inside Functions

```js
function example() {
  console.log(hoisted); // undefined (hoisted but not initialized)
  var hoisted = "value";
  console.log(hoisted); // "value"
}

example();
// console.log(hoisted); // ReferenceError: hoisted is not defined
```

**Expected Output:**
```
undefined
value
```

**Why This Output:** `var hoisted` is hoisted to the top of the function scope and initialised to `undefined`. The assignment happens later. Outside the function, `hoisted` is not defined.

#### Example 3: Each Call Creates a New Scope

```js
function counter() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
}

const c1 = counter();
const c2 = counter();

console.log(c1()); // 1
console.log(c1()); // 2
console.log(c2()); // 1 (separate scope)
```

**Expected Output:**
```
1
2
1
```

**Why This Output:** Each call to `counter()` creates a new function scope. `c1` and `c2` have independent `count` variables.

### Real-World Cases

- **Encapsulation:** Private variables in module patterns (IIFE).
- **Event handlers:** Each handler closure captures its own scope.
- **Recursion:** Each recursive call has its own scope.
- **Memoisation:** Caching results in a closure-scoped variable.

---

## Block Scope

### Definitions

**Core Definition:** Block scope is the scope created by a pair of curly braces `{}`, where variables declared with `let`, `const`, and `class` are accessible only within that block.

**Technical Definition:** Per ECMA-262 §13.2.2, a BlockStatement creates a new declarative environment record when it contains lexical declarations (`let`, `const`, `class`, `function` in strict mode). `var` declarations ignore block scope and bind to the nearest function or global environment.

**Beginner-Friendly Explanation:** Any time you write code inside curly braces `{ }` — like in an `if` statement or a `for` loop — variables declared with `let` and `const` are only visible inside those braces. `var`, however, ignores the braces and "escapes" to the surrounding function.

### Purposes

- To limit the lifetime and visibility of loop variables.
- To prevent variable collisions in conditional branches.
- To create temporary scopes for `switch` statements and `try...catch`.
- To enable block-level encapsulation.

### Syntax Rules and Structure

```js
{
  let blockLet = "visible only here";
  const blockConst = "also visible only here";
  var functionVar = "visible in the whole function";
}

// blockLet and blockConst are NOT accessible here
// functionVar IS accessible here (if in the same function)
```

**Key Keywords:**
- `let`: Block-scoped, can be reassigned.
- `const`: Block-scoped, cannot be reassigned.
- `class`: Block-scoped.
- `function` (strict mode): Block-scoped.
- `var`: NOT block-scoped; function-scoped.

**Constraints and Limitations:**
- `var` inside a block "leaks" to the enclosing function scope.
- Block-scoped variables are hoisted but remain in the **Temporal Dead Zone (TDZ)** until their declaration is evaluated.
- Accessing a `let`/`const` variable before its declaration throws `ReferenceError`.

### Annotated Code Examples

#### Example 1: Block Scope with `let` vs `var`

```js
// Step 1: Block with let.
{
  let blockVar = "block";
  console.log(blockVar); // "block"
}
// console.log(blockVar); // ReferenceError: blockVar is not defined

// Step 2: Block with var.
{
  var functionVar = "function";
}
console.log(functionVar); // "function" (var ignores block scope)
```

**Expected Output:**
```
block
function
```

**Why This Output:** `let` is confined to the block. `var` ignores the block and binds to the enclosing function or global scope.

#### Example 2: Loop Variable Scoping

```js
// Step 1: var in a loop leaks.
for (var i = 0; i < 3; i++) {}
console.log(i); // 3 (i leaked)

// Step 2: let in a loop is scoped per iteration.
for (let j = 0; j < 3; j++) {}
// console.log(j); // ReferenceError: j is not defined
```

**Expected Output:**
```
3
```

**Why This Output:** `var i` is hoisted to the enclosing scope and remains accessible after the loop. `let j` is scoped to each iteration of the loop and is not accessible outside.

#### Example 3: Temporal Dead Zone

```js
{
  // console.log(x); // ReferenceError: Cannot access 'x' before initialization
  let x = 5;
  console.log(x); // 5
}
```

**Expected Output:**
```
5
```

**Why This Output:** The `let x` declaration is hoisted but not initialised. Accessing it before the declaration line throws a `ReferenceError` (the Temporal Dead Zone).

### Real-World Cases

- **Loop counters:** Using `let` in `for` loops prevents leakage.
- **Conditional variables:** Scoping variables to `if` branches.
- **Switch statements:** Preventing fall-through variable access.
- **Try/catch:** Scoping the error variable to the catch block.

---

## Module Scope

### Definitions

**Core Definition:** Module scope is the scope of a JavaScript module, where top-level variables, functions, and classes are private to the module unless explicitly exported.

**Technical Definition:** Per ECMA-262, a module environment is a Lexical Environment that contains the bindings for the top-level declarations of a Module. Modules are executed within their own scope, not in the global scope. Top-level `var` declarations in a module do not create global properties. Variables must be explicitly exported to be accessible from other modules.

**Beginner-Friendly Explanation:** A module is a file that has its own private space. Variables you declare at the top of a module are only visible inside that module — unless you explicitly `export` them. This prevents naming conflicts between files.

### Purposes

- To encapsulate code at the file level.
- To prevent global namespace pollution.
- To explicitly control what is shared between files.
- To enable dependency management via imports and exports.

### Syntax Rules and Structure

```js
// module.js
// Top-level declarations are module-scoped (not global).
const privateVar = "only in this module";
export const publicVar = "shared with importers";

export function publicFunction() {
  return privateVar; // Can access module-scoped variables
}

// Top-level var does NOT create a global.
var moduleVar = "module-scoped";
// window.moduleVar is undefined in browsers
```

**Import Syntax:**
```js
// another-module.js
import { publicVar, publicFunction } from "./module.js";
console.log(publicVar); // "shared with importers"
// console.log(privateVar); // ReferenceError
```

**Syntax Rules:**
- A file is a module if it contains top-level `import` or `export`.
- Modules are always in strict mode by default.
- Top-level `var`, `let`, `const`, `function`, and `class` are module-scoped.
- `export` makes bindings available to other modules.
- `import` brings bindings from other modules into the current module.
- Modules are loaded with `<script type="module">` in browsers.

**Constraints and Limitations:**
- Module scope is not accessible from classic scripts.
- Module scripts are deferred by default.
- Circular imports have special semantics.

### Annotated Code Examples

#### Example 1: Module Encapsulation

```js
// utils.js
let counter = 0; // module-scoped, not global

export function increment() {
  counter++;
  return counter;
}

export function reset() {
  counter = 0;
}

// app.js
import { increment, reset } from "./utils.js";

console.log(increment()); // 1
console.log(increment()); // 2
console.log(reset());     // undefined
console.log(increment()); // 1
```

**Expected Output:**
```
1
2
undefined
1
```

**Why This Output:** `counter` is module-scoped in `utils.js`. The exported functions share access to it. External code cannot modify `counter` directly.

#### Example 2: Module Top-Level `var` Does Not Pollute Global

```js
// module.js
var moduleScoped = "I am module-scoped";
console.log(moduleScoped); // "I am module-scoped"

// In the browser console after loading the module:
// console.log(window.moduleScoped); // undefined
```

**Expected Output:**
```
I am module-scoped
```

**Why This Output:** Unlike classic scripts, modules do not share the global scope. Top-level `var` in a module is not accessible via `window`.

### Real-World Cases

- **Component libraries:** Exporting components and utilities.
- **Configuration modules:** Private config with public API.
- **State management:** Module-level state with controlled access.
- **Utility modules:** Pure functions exported for reuse.

---

## Lexical Scope

### Definitions

**Core Definition:** Lexical scope (also called static scope) is the scope model where a function's accessible variables are determined by where the function is **defined** in the source code, not where it is called.

**Technical Definition:** Per ECMA-262, JavaScript uses lexical scoping. A function's `[[Environment]]` internal slot is set to the Lexical Environment of its enclosing scope when the function is created, not when it is called. Name resolution walks the scope chain from the function's own environment outward to the global environment.

**Beginner-Friendly Explanation:** Lexical scope means "scope by location in the code." If you define a function inside another function, it can see the outer function's variables. If you define it at the top level, it can only see global variables — regardless of where you call it from.

### Purposes

- To make scope predictable and analysable by reading the code.
- To enable closures that capture their defining environment.
- To support modular and encapsulated code design.

### Syntax Rules and Structure

```js
const x = "global";

function outer() {
  const x = "outer";

  function inner() {
    // inner is defined inside outer, so it sees outer's x.
    console.log(x); // "outer"
  }

  inner();
}

function unrelated() {
  // unrelated is defined at top level, so it sees global x.
  console.log(x); // "global"
}

outer();
unrelated();
```

**Key Rule:** A function can access variables in its own scope and in any scope where it was **defined** (lexically enclosed). It cannot access variables in scopes where it is **called**.

### Annotated Code Examples

#### Example 1: Lexical Scope Determination

```js
const value = "global";

function outer() {
  const value = "outer";

  function inner() {
    console.log(value); // "outer" — determined by definition location
  }

  return inner;
}

const fn = outer();
fn(); // "outer" — even though called from global scope
```

**Expected Output:**
```
outer
```

**Why This Output:** `inner` was defined inside `outer`, so it lexically captures `outer`'s scope. Calling `fn()` from the global scope does not change its scope chain.

#### Example 2: Lexical Scope vs Call Location

```js
const a = "global-a";

function showA() {
  console.log(a);
}

function caller() {
  const a = "caller-a";
  showA(); // showA is defined globally, so it sees global a.
}

caller(); // "global-a"
```

**Expected Output:**
```
global-a
```

**Why This Output:** `showA` was defined at the top level, so its scope chain includes only the global scope. The fact that it is called from inside `caller` (which has its own `a`) does not matter — this is the essence of lexical scoping.

### Real-World Cases

- **Closures:** Capturing variables from the defining scope.
- **Event handlers:** Accessing variables from the surrounding code.
- **Module patterns:** Using IIFE to create private scope.
- **React hooks:** Closures capture props and state from the component's scope.

---

## Dynamic Scope

### Definitions

**Core Definition:** Dynamic scope is a scope model where a function's accessible variables are determined by the **call stack** at runtime, not by where the function is defined. JavaScript does **not** use dynamic scoping.

**Technical Definition:** In dynamic scoping, name resolution searches the calling function's environment, then its caller's environment, and so on up the call stack. JavaScript uses lexical scoping instead, but `this` behaves somewhat dynamically (determined by how a function is called).

**Beginner-Friendly Explanation:** Dynamic scope would mean "a function sees whatever variables its caller has." JavaScript doesn't work this way — but it's important to understand the difference because some languages (like Bash) do use dynamic scope.

### Purposes (Conceptual)

- To understand the contrast with lexical scoping.
- To recognise why JavaScript's behaviour may differ from dynamically scoped languages.
- To clarify that `this` is not scope.

### Syntax Rules and Structure

JavaScript does **not** implement dynamic scoping. The following is **not** valid JavaScript behaviour:

```js
// HYPOTHETICAL dynamic scope — NOT JavaScript!
const x = "global";
function foo() {
  console.log(x); // Would see caller's x in dynamic scope
}
function bar() {
  const x = "bar";
  foo(); // Would print "bar" in dynamic scope
}
bar(); // In JavaScript, prints "global" (lexical)
```

### Annotated Code Examples

#### Example 1: JavaScript is Not Dynamically Scoped

```js
const x = "global";

function foo() {
  console.log(x);
}

function bar() {
  const x = "bar";
  foo(); // Lexical: foo sees global x, not bar's x
}

bar(); // "global"
```

**Expected Output:**
```
global
```

**Why This Output:** JavaScript uses lexical scoping. `foo` was defined at the top level, so it sees the global `x`. If JavaScript were dynamically scoped, it would print `"bar"`.

#### Example 2: `this` Behaves Dynamically (But Is Not Scope)

```js
function showThis() {
  console.log(this.name);
}

const obj = { name: "object", showThis };
obj.showThis(); // "object" — this is determined by call site

const standalone = obj.showThis;
standalone(); // undefined (or global name in non-strict mode)
```

**Expected Output:**
```
object
undefined
```

**Why This Output:** `this` is determined by how a function is called (dynamic), but `this` is not part of the scope chain. Variables are resolved lexically; `this` is resolved dynamically.

### Real-World Cases

- **Understanding language differences:** Bash, Perl, and some Lisp dialects use dynamic scope.
- **Clarifying `this` vs scope:** `this` is dynamically bound, but scope is lexical.
- **Avoiding confusion:** Knowing that JavaScript does not use dynamic scope prevents incorrect mental models.

---

## Scope Chain

### Definitions

**Core Definition:** The scope chain is the linked list of lexical environments that a function has access to, from its own scope outward to the global scope.

**Technical Definition:** Per ECMA-262, each Lexical Environment has an `outer` reference pointing to the enclosing Lexical Environment. When an identifier is resolved, the engine searches the current environment record first, then follows the `outer` reference to the next environment, continuing until the identifier is found or the global environment is reached without a match. The chain is fixed at function creation time, not at call time.

**Beginner-Friendly Explanation:** When JavaScript looks up a variable, it searches in the current scope first. If it doesn't find it, it looks in the "parent" scope, then the grandparent, and so on until it reaches the global scope. This chain of scopes is the scope chain.

### Purposes

- To enable access to outer-scope variables from inner scopes.
- To provide a predictable resolution order for identifiers.
- To support closures by keeping outer environments alive.

### Syntax Rules and Structure

The scope chain is not written explicitly; it is created implicitly by nesting functions and blocks:

```js
const globalVar = "global";

function outer() {
  const outerVar = "outer";

  function inner() {
    const innerVar = "inner";
    // Scope chain for inner:
    // 1. inner's own scope (innerVar)
    // 2. outer's scope (outerVar)
    // 3. global scope (globalVar)
  }
}
```

**Resolution Order:**
1. Current scope's environment record.
2. Outer environment record (repeat until global).
3. Global environment record.
4. If not found: `ReferenceError`.

### Annotated Code Examples

#### Example 1: Walking the Scope Chain

```js
const level0 = "global";

function level1() {
  const level1Var = "level1";

  function level2() {
    const level2Var = "level2";

    function level3() {
      console.log(level0); // Found in global
      console.log(level1Var); // Found in level1
      console.log(level2Var); // Found in level2
      // console.log(level3Var); // Not defined anywhere
    }

    level3();
  }

  level2();
}

level1();
```

**Expected Output:**
```
global
level1
level2
```

**Why This Output:** The engine searches the scope chain from innermost to outermost. `level0` is found in global, `level1Var` in `level1`, and `level2Var` in `level2`.

#### Example 2: Scope Chain Is Fixed at Definition

```js
const x = "global";

function defineHere() {
  console.log(x);
}

function callHere() {
  const x = "callHere";
  defineHere(); // Scope chain of defineHere does NOT include callHere
}

callHere(); // "global"
```

**Expected Output:**
```
global
```

**Why This Output:** `defineHere`'s scope chain is determined at definition time. It includes only its own scope and the global scope — not `callHere`'s scope.

### Real-World Cases

- **Closures:** The scope chain keeps outer variables alive for inner functions.
- **Debugging:** Understanding which variable is resolved in nested scopes.
- **Performance:** Deep scope chains have slightly higher lookup costs.

---

## Variable Shadowing

### Definitions

**Core Definition:** Variable shadowing occurs when a variable declared in an inner scope has the same name as a variable in an outer scope, making the outer variable temporarily inaccessible from within the inner scope.

**Technical Definition:** When an inner Lexical Environment contains an environment record with a binding for an identifier that also exists in an outer environment record, the inner binding **shadows** the outer one. Name resolution finds the inner binding first and stops searching.

**Beginner-Friendly Explanation:** Shadowing is like having two people named "Alex" in a room. If you're standing next to one Alex, calling "Alex!" refers to the one nearest to you. Similarly, a local variable with the same name as a global variable "hides" the global one inside that local scope.

### Purposes (and Anti-Purposes)

- To intentionally override an outer variable within a specific scope (sometimes useful).
- To avoid accidental shadowing, which can cause confusing bugs.
- To understand why a variable has an unexpected value.

**Note:** Shadowing is often considered an anti-pattern. ESLint's `no-shadow` rule exists to disallow it.

### Syntax Rules and Structure

```js
const name = "Global";

function example() {
  const name = "Local"; // Shadows the global name
  console.log(name); // "Local"
}

example();
console.log(name); // "Global" (unchanged)
```

**Syntax Rules:**
- Shadowing occurs when an inner scope declares an identifier with the same name as an outer scope.
- `var`, `let`, `const`, `function`, `class`, and parameters can all shadow.
- A `let` declaration cannot shadow a `let` in the same scope (SyntaxError).
- Parameters shadow outer variables.

**Constraints and Limitations:**
- Shadowing can make code harder to read and debug.
- The outer variable is inaccessible (by name) within the shadowing scope.
- Tooling like ESLint can flag shadowing as a warning.

### Annotated Code Examples

#### Example 1: Simple Shadowing

```js
const city = "Paris";

function showCity() {
  const city = "London"; // Shadows global city
  console.log(city); // "London"
}

showCity();
console.log(city); // "Paris"
```

**Expected Output:**
```
London
Paris
```

**Why This Output:** The `city` inside `showCity` is a new binding that shadows the global `city`. The global remains unchanged.

#### Example 2: Parameter Shadowing

```js
const value = 100;

function process(value) { // Parameter shadows global value
  return value * 2;
}

console.log(process(5)); // 10
console.log(value); // 100 (unchanged)
```

**Expected Output:**
```
10
100
```

**Why This Output:** The parameter `value` shadows the global `value` inside the function. Outside, the global is still `100`.

#### Example 3: Block Shadowing

```js
let x = 1;

{
  let x = 2; // Shadows outer x
  console.log(x); // 2
}

console.log(x); // 1
```

**Expected Output:**
```
2
1
```

**Why This Output:** The block-scoped `x` shadows the outer `x` within the block. After the block, the outer `x` is accessible again.

### Real-World Cases

- **Loop variables:** `for (let i = 0; ...)` shadows any outer `i`.
- **Event handlers:** Parameter shadowing can cause unexpected behavior.
- **Testing:** Local mocks shadow imported modules.
- **Linting:** ESLint `no-shadow` catches problematic shadowing.

---

## Variable Leakage

### Definitions

**Core Definition:** Variable leakage is the unintended creation of a variable in an outer or global scope due to assignment without declaration, or due to `var` ignoring block scope.

**Technical Definition:** In non-strict mode, an assignment to an undeclared identifier creates a property on the global object (`window` in browsers) rather than throwing a `ReferenceError`. Additionally, `var` declarations inside blocks are hoisted to the enclosing function or global scope, effectively "leaking" out of the block.

**Beginner-Friendly Explanation:** Variable leakage happens when a variable you meant to be local accidentally becomes global. This usually happens when you forget to use `var`, `let`, or `const`, or when you use `var` inside a block and expect it to stay there.

### Purposes (and Anti-Purposes)

- **Anti-purpose:** Leakage is almost always unintentional and is considered a bug.
- To understand why strict mode is recommended.
- To recognise and prevent accidental globals.

### Syntax Rules and Structure

```js
// Leakage via undeclared assignment
function leaky() {
  accidentallyGlobal = "oops"; // No declaration keyword
}

// Leakage via var in a block
function blockLeak() {
  if (true) {
    var leaked = "I escape the block";
  }
  console.log(leaked); // "I escape the block"
}
```

**Prevention:**
- Use `"use strict";` to throw `ReferenceError` on undeclared assignments.
- Use `let`/`const` instead of `var`.
- Use linters (ESLint `no-implicit-globals`).

### Annotated Code Examples

#### Example 1: Undeclared Assignment Leak

```js
function accidentalGlobal() {
  leakedVar = "I am global now"; // No var/let/const
}

accidentalGlobal();
console.log(leakedVar); // "I am global now"
console.log(window.leakedVar); // "I am global now"
```

**Expected Output:**
```
I am global now
I am global now
```

**Why This Output:** Without a declaration keyword, JavaScript creates a global variable in non-strict mode.

#### Example 2: `var` Block Leak

```js
function blockLeak() {
  if (true) {
    var blockVar = "I escape!";
  }
  console.log(blockVar); // "I escape!"
}

blockLeak();
```

**Expected Output:**
```
I escape!
```

**Why This Output:** `var` ignores block scope and binds to the enclosing function scope.

#### Example 3: Strict Mode Prevents Leakage

```js
"use strict";

function safe() {
  // undeclared = "error"; // ReferenceError: undeclared is not defined
}

function stillSafe() {
  let local = "safe";
  console.log(local);
}
stillSafe();
```

**Expected Output:**
```
safe
```

**Why This Output:** Strict mode throws an error for undeclared assignments, preventing accidental globals.

### Real-World Cases

- **Legacy codebases:** Before strict mode and `let`, leakage was common.
- **Global namespace pollution:** Multiple scripts leaking variables cause collisions.
- **Security:** Leaked variables can be accessed by third-party scripts.
- **Debugging:** Leakage causes hard-to-trace bugs.

---

## Scope Pollution

### Definitions

**Core Definition:** Scope pollution (also called namespace pollution) is the accumulation of too many variables in the global scope, increasing the risk of name collisions and making code harder to maintain.

**Technical Definition:** Scope pollution occurs when scripts add variables, functions, or properties to the global object or global lexical environment. This is problematic because all scripts on a page share the same global scope, so a variable defined by one script can overwrite or conflict with a variable defined by another.

**Beginner-Friendly Explanation:** Scope pollution is like having too many things in a shared closet. If everyone keeps dumping stuff into the global scope, eventually two people will want to store different things under the same name — and one will overwrite the other.

### Purposes (and Anti-Purposes)

- **Anti-purpose:** Scope pollution is a code smell and a source of bugs.
- To understand why modules and IIFEs are preferred.
- To recognise the risks of global variables.

### Syntax Rules and Structure

**Causes of scope pollution:**
```js
// 1. Global variables declared with var
var globalConfig = {}; // Becomes window.globalConfig

// 2. Implicit globals (undeclared assignments)
function leaky() {
  accidental = "global";
}

// 3. Adding properties to the global object
window.myPlugin = {};

// 4. Multiple scripts sharing the global scope
// script1.js
var data = [1, 2, 3];
// script2.js
var data = ["a", "b"]; // Overwrites script1's data!
```

**Prevention:**
- Use ES modules (each module has its own scope).
- Use IIFEs to create private scope.
- Use `let`/`const` instead of `var`.
- Use strict mode.

### Annotated Code Examples

#### Example 1: Name Collision from Global Pollution

```js
// script1.js
var user = { name: "Alice" };

// script2.js (loaded after script1)
var user = { name: "Bob" }; // Overwrites Alice's user!

console.log(user.name); // "Bob"
```

**Expected Output:**
```
Bob
```

**Why This Output:** Both scripts declare `var user` in the global scope. The second declaration overwrites the first.

#### Example 2: IIFE Prevents Pollution

```js
// Wrap code in an IIFE to create private scope.
(function () {
  var privateVar = "not global";
  function helper() { return privateVar; }
  console.log(helper()); // "not global"
})();

console.log(typeof privateVar); // "undefined"
console.log(typeof helper); // "undefined"
```

**Expected Output:**
```
not global
undefined
undefined
```

**Why This Output:** The IIFE creates a function scope. Variables inside are not added to the global scope.

#### Example 3: Module Scope Prevents Pollution

```js
// module1.js
const config = { theme: "dark" };
export function getTheme() { return config.theme; }

// module2.js
const config = { theme: "light" }; // No conflict — module-scoped
export function getTheme() { return config.theme; }
```

**Why This Output:** Each module has its own top-level scope. The two `config` variables do not conflict.

### Real-World Cases

- **Multiple libraries:** jQuery, Lodash, and custom scripts colliding.
- **Legacy scripts:** Pre-module codebases with global variables.
- **Browser extensions:** Injecting scripts into pages with existing globals.
- **Testing:** Global state leaking between tests.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Lexical Environments — https://tc39.es/ecma262/multipage/executable-code-and-execution-contexts.html#sec-lexical-environments
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Environment Records — https://tc39.es/ecma262/multipage/executable-code-and-execution-contexts.html#sec-environment-records
- MDN Web Docs – Scope — https://developer.mozilla.org/en-US/docs/Glossary/Scope
- MDN Web Docs – Hoisting — https://developer.mozilla.org/en-US/docs/Glossary/Hoisting
- MDN Web Docs – Closures — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures
- MDN Web Docs – let — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let
- MDN Web Docs – const — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const
- MDN Web Docs – var — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var
- MDN Web Docs – JavaScript modules — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- TypeScript Handbook – Modules — https://www.typescriptlang.org/docs/handbook/2/modules.html
- V8 Blog – JavaScript modules — https://v8.dev/features/modules
- Cornell CS – Is JavaScript Statically or Dynamically Scoped? — https://www.cs.cornell.edu/~asampson/blog/scope.html
- ESLint – no-shadow — https://eslint.org/docs/latest/rules/no-shadow
- ESLint – no-implicit-globals — https://eslint.org/docs/latest/rules/no-implicit-globals
- You Don't Know JS – Scope & Closures — https://github.com/getify/You-Dont-Know-JS