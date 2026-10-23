# JavaScript Variables — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A variable is a named storage location in memory that holds a value which can be referenced and manipulated throughout a program. In JavaScript, variables are declared using `var`, `let`, or `const`, each with distinct scoping and reassignment behaviours.

**Technical Definition:** Per the ECMAScript specification (ECMA-262 §14.3), a variable declaration creates a binding in a Lexical Environment (Declarative Environment Record or Object Environment Record). `var` statements create mutable bindings in the VariableEnvironment (function or global scope); `let` and `const` declarations create bindings in the LexicalEnvironment's declarative record (block scope). `const` creates an immutable binding after initialisation. All declarations are instantiated during the creation of the execution context — a process colloquially called hoisting — but lexical declarations (`let`, `const`) remain uninitialised in the Temporal Dead Zone (TDZ) until their declaration is evaluated.

**Beginner-Friendly Explanation:** A variable is a labelled box where you store information — a number, a name, a list, anything. You give the box a name so you can find it later and put something new inside, or read what's already there. JavaScript gives you three kinds of boxes: `var` (the old, quirky kind), `let` (the modern, changeable kind), and `const` (the modern, locked kind).

### Key Characteristics

- **Three declaration keywords:** `var` (function/global scope, hoisted and initialised to `undefined`), `let` (block scope, hoisted but uninitialised in TDZ), `const` (block scope, immutable binding, must be initialised at declaration).
- **Scope determines visibility:** Variables are visible only within their enclosing scope (global, function, or block). Blocks scope `let` and `const` but not `var`.
- **Hoisting is universal but behaviour differs:** All declarations are hoisted, but `var` is initialised to `undefined`, while `let` and `const` remain in the TDZ until evaluated.
- **Reassignment vs. mutability:** `let` and `var` bindings can be reassigned; `const` bindings cannot. However, `const` objects remain mutable — only the binding is locked.
- **Naming is case-sensitive and syntax-restricted:** Identifiers must start with a letter, `$`, or `_`, cannot start with a digit, and cannot be reserved words.
- **Strict mode changes behaviour:** In strict mode, assigning to an undeclared variable throws a `ReferenceError`; in sloppy mode, it creates a global property.

### Prerequisites

- Basic understanding of JavaScript execution contexts and the call stack.
- Familiarity with blocks (`{}`), functions, and the global object.
- Awareness of strict mode (`"use strict"`) and its effect on undeclared variables.

### Related Programming Areas

- **Scope and closures:** Variable scope underpins closure behaviour.
- **Memory management:** Long-lived variables (especially globals) affect garbage collection.
- **Module systems:** ES modules introduce module scope, distinct from global and function scope.
- **Testing and debugging:** Understanding hoisting and TDZ explains many common runtime errors.

### Core Concepts / Features

The following sections cover each requested concept using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## `var`

### Definitions

**Core Definition:** `var` declares a function-scoped or globally-scoped variable, optionally initialising it to a value, and is the original variable declaration mechanism in JavaScript.

**Technical Definition:** The `var` statement creates a mutable binding in the running execution context's VariableEnvironment. Declarations are hoisted to the top of their containing function or script, and the binding is initialised to `undefined` during hoisting. `var` is not block-scoped; it ignores block boundaries and is scoped only to the enclosing function or global environment.

**Beginner-Friendly Explanation:** `var` is the old way to make a variable. It's a bit sloppy: it can "leak" out of blocks like `if` statements and loops, and you can use it before you declare it (you'll just get `undefined`). Modern code usually avoids `var` in favour of `let` and `const`.

### Purposes

- To declare variables in legacy codebases and older codebases that predate ES2015.
- To create function-scoped variables that persist across block boundaries within a function.
- To redeclare a variable in the same scope (allowed with `var`, unlike `let`/`const`).
- To attach properties to the global object when declared at the top level in sloppy mode (a behaviour to be aware of, not recommended).

### Syntax Rules and Structure

**General Syntax:**
```js
var name;
var name = value;
var name1 = value1, name2 = value2;
```

**Component Breakdown:**
- `var`: The declaration keyword.
- `name`: A valid identifier (binding name).
- `= value`: Optional initialiser expression.
- Multiple declarations can be comma-separated.

**Hoisting Behaviour:** `var` declarations are hoisted with type 2 behaviour: the binding is created and initialised to `undefined` at the top of the function/script scope, before any code executes. The assignment (if any) occurs at the point of the declaration statement.

**Syntax Rules:**
- Can be used at the global, function, or block level.
- Redeclaration in the same scope is allowed (the later declaration is ignored for the binding, but the assignment executes).
- Can be called without an initialiser (`var x;` yields `undefined`).

**Constraints and Limitations:**
- Not block-scoped: variables leak out of `if`, `for`, and other blocks into the enclosing function.
- Hoisting with `undefined` can mask errors (e.g., using a variable before assignment silently yields `undefined`).
- In strict mode, assigning to an undeclared variable throws; in sloppy mode, it creates a global.

### Annotated Code Examples

#### Example 1: Function Scoping and Block Leakage

```js
// Step 1: Declare a var inside a block.
if (true) {
  var functionScoped = "I escape the block";
}

// Step 2: The variable is visible outside the block.
console.log(functionScoped); // "I escape the block"

// Step 3: Declare a var inside a function.
function test() {
  var localVar = "I stay inside";
  console.log(localVar);
}
test(); // "I stay inside"

// Step 4: The local variable is not visible outside the function.
try {
  console.log(localVar); // ReferenceError
} catch (err) {
  console.log(err.name); // "ReferenceError"
}
```

**Expected Output:**
```
I escape the block
I stay inside
ReferenceError
```

**Why This Output:** The first `var` leaks out of the `if` block because `var` is not block-scoped. The second `var` is scoped to the function `test` and is invisible outside it.

#### Example 2: Hoisting with `var`

```js
// Step 1: Log the variable before its declaration line.
console.log(hoistedVar); // undefined (not an error)

// Step 2: Declare and assign the variable.
var hoistedVar = "Now I have a value";

// Step 3: Log again.
console.log(hoistedVar); // "Now I have a value"
```

**Expected Output:**
```
undefined
Now I have a value
```

**Why This Output:** The `var` declaration is hoisted to the top of the scope and initialised to `undefined`. The assignment happens at the original line. This is why accessing it early does not throw but yields `undefined`.

### Real-World Cases

- **Legacy codebases:** Many older libraries and scripts still use `var`; understanding its scoping is necessary for maintenance.
- **Loop variables in non-block contexts:** `var` in a `for` loop leaks to the enclosing function, which can cause bugs with asynchronous callbacks (a classic interview question).
- **Global configuration objects:** In sloppy-mode scripts, top-level `var` declarations become properties of `window` (or `globalThis`).

---

## `let`

### Definitions

**Core Definition:** `let` declares a block-scoped local variable, optionally initialising it to a value. It was introduced in ES2015 (ES6) to address the scoping and hoisting issues of `var`.

**Technical Definition:** The `let` declaration creates a mutable binding in the declarative environment record of the current LexicalEnvironment. The binding is hoisted to the top of the block but remains uninitialised (in the Temporal Dead Zone) until the declaration is evaluated. After initialisation, the binding can be reassigned.

**Beginner-Friendly Explanation:** `let` is the modern, well-behaved way to make a variable you plan to change. It stays inside the block where you declare it — no leaking — and you can't use it before you declare it (you'll get a `ReferenceError` instead of `undefined`).

### Purposes

- To declare variables that need to be reassigned later.
- To confine variables to the block where they are needed, preventing accidental leakage.
- To avoid the hoisting pitfalls associated with `var` (using a variable before declaration yields a clear error rather than silent `undefined`).
- To serve as the default modern choice when `const` is not applicable.

### Syntax Rules and Structure

**General Syntax:**
```js
let name;
let name = value;
let name1 = value1, name2 = value2;
```

**Component Breakdown:**
- `let`: The block-scoped declaration keyword.
- `name`: A valid identifier.
- `= value`: Optional initialiser.

**Hoisting Behaviour:** `let` declarations are hoisted with type 3 behaviour: the binding is created at the top of the block, but remains uninitialised until the declaration is evaluated. Accessing it before initialisation throws a `ReferenceError` (the Temporal Dead Zone).

**Syntax Rules:**
- Block-scoped: visible only within the enclosing `{}` block.
- Cannot be redeclared in the same scope (SyntaxError).
- Can be declared without an initialiser (`let x;` yields `undefined` after declaration).
- Can be reassigned freely after initialisation.

**Constraints and Limitations:**
- The TDZ means `typeof x` throws for a `let` variable before declaration (unlike undeclared variables, where `typeof` returns `"undefined"`).
- Hoisted bindings from `let` can "shadow" outer bindings even before initialisation (the TDZ "taints" the entire block).

### Annotated Code Examples

#### Example 1: Block Scoping with `let`

```js
// Step 1: Declare a let inside a block.
if (true) {
  let blockScoped = "I stay inside";
  console.log(blockScoped); // "I stay inside"
}

// Step 2: Try to access it outside the block.
try {
  console.log(blockScoped); // ReferenceError
} catch (err) {
  console.log(err.name); // "ReferenceError"
}
```

**Expected Output:**
```
I stay inside
ReferenceError
```

**Why This Output:** `let` is block-scoped; the variable exists only inside the `if` block. Outside, the binding is not visible, and referencing it throws a `ReferenceError`.

#### Example 2: Reassignment with `let`

```js
// Step 1: Declare and initialise.
let counter = 0;
console.log(counter); // 0

// Step 2: Reassign.
counter = 1;
console.log(counter); // 1

// Step 3: Reassign again.
counter += 1;
console.log(counter); // 2
```

**Expected Output:**
```
0
1
2
```

**Why This Output:** `let` bindings are mutable; reassignment updates the value associated with the binding.

#### Example 3: Temporal Dead Zone with `let`

```js
// Step 1: Attempt to access before declaration.
try {
  console.log(tdzVar);
} catch (err) {
  console.log(err.name);    // "ReferenceError"
  console.log(err.message); // "Cannot access 'tdzVar' before initialization"
}

// Step 2: Declare the variable.
let tdzVar = "Initialised";
console.log(tdzVar); // "Initialised"
```

**Expected Output:**
```
ReferenceError
Cannot access 'tdzVar' before initialization
Initialised
```

**Why This Output:** The `let` binding is hoisted but uninitialised. Accessing it in the TDZ throws a `ReferenceError` with a message specific to lexical declarations.

### Real-World Cases

- **Loop variables in `for` loops:** `let` creates a new binding per iteration, solving the classic closure capture problem that `var` exhibits.
- **Conditional variables:** Declaring variables inside `if`/`else` blocks without polluting the outer scope.
- **Modern default:** `let` is the standard choice when a variable needs to change, per MDN and most style guides.

---

## `const`

### Definitions

**Core Definition:** `const` declares a block-scoped variable whose binding cannot be reassigned after initialisation. The value itself may be mutable if it is an object or array.

**Technical Definition:** The `const` declaration creates an immutable binding in the declarative environment record of the current LexicalEnvironment. Unlike `let`, the binding cannot be reassigned after initialisation; attempting to do so throws a `TypeError` ("Assignment to constant variable"). The binding is hoisted and remains in the TDZ until initialisation, and an initialiser is mandatory at the point of declaration.

**Beginner-Friendly Explanation:** `const` creates a variable that cannot be pointed at a different value later. If you put a number or string in a `const`, it's truly fixed. If you put an object or array, you can still change what's inside it — you just can't replace the whole object with a different one.

### Purposes

- To declare values that should never be reassigned (mathematical constants, configuration keys, module imports).
- To signal intent to readers: "this binding will not change."
- To prevent accidental reassignment bugs.
- To serve as the preferred default for variables that do not need to change (per modern style guides).

### Syntax Rules and Structure

**General Syntax:**
```js
const name = value;
const name1 = value1, name2 = value2;
```

**Component Breakdown:**
- `const`: The block-scoped immutable declaration keyword.
- `name`: A valid identifier.
- `= value`: Mandatory initialiser.

**Hoisting Behaviour:** `const` declarations are hoisted with type 3 behaviour (same as `let`): the binding is created but remains in the TDZ until initialisation. Accessing it before the declaration throws a `ReferenceError`.

**Syntax Rules:**
- An initialiser is required (`const x;` is a SyntaxError).
- Block-scoped, like `let`.
- Cannot be redeclared in the same scope.
- Reassignment throws a `TypeError` at runtime.
- Object and array contents remain mutable (the binding is immutable, not the value).

**Constraints and Limitations:**
- Not suitable for variables that need to change (use `let`).
- The immutability is shallow; deep immutability requires `Object.freeze()`.
- Reassignment attempt produces a `TypeError`, not a `SyntaxError` (because the declaration itself is valid; the violation occurs at runtime).

### Annotated Code Examples

#### Example 1: Immutable Binding

```js
// Step 1: Declare a const with a primitive value.
const PI = 3.14159;
console.log(PI); // 3.14159

// Step 2: Attempt reassignment.
try {
  PI = 3.14; // TypeError
} catch (err) {
  console.log(err.name);    // "TypeError"
  console.log(err.message); // "Assignment to constant variable."
}
```

**Expected Output:**
```
3.14159
TypeError
Assignment to constant variable.
```

**Why This Output:** The `const` binding cannot be reassigned. The attempt throws a `TypeError` at the assignment site.

#### Example 2: Mutable Object Contents

```js
// Step 1: Declare a const object.
const user = { name: "Alice" };

// Step 2: Mutate the object's properties (allowed).
user.name = "Bob";
user.age = 30;
console.log(user); // { name: "Bob", age: 30 }

// Step 3: Attempt to reassign the binding (not allowed).
try {
  user = { name: "Charlie" };
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
{ name: 'Bob', age: 30 }
TypeError
```

**Why This Output:** The `const` binding holds a reference to the object. The reference cannot be changed, but the object's contents are mutable. Reassigning the binding to a new object throws.

### Real-World Cases

- **Module imports:** ES module imports are effectively `const` bindings.
- **Configuration constants:** API base URLs, maximum retry counts, mathematical constants.
- **Function expressions:** `const fn = () => {}` prevents accidental overwriting of the function reference.
- **Object and array declarations:** Most modern style guides recommend `const` by default, switching to `let` only when reassignment is needed.

---

## Variable Declaration

### Definitions

**Core Definition:** Variable declaration is the act of introducing a new named binding into a scope using `var`, `let`, or `const`, without necessarily assigning a value.

**Technical Definition:** Declaration creates a binding in the appropriate Environment Record during the instantiation phase of the execution context. For `var`, the binding is created in the VariableEnvironment and initialised to `undefined`; for `let` and `const`, the binding is created in the LexicalEnvironment's declarative record and remains uninitialised (TDZ) until the declaration is evaluated.

**Beginner-Friendly Explanation:** Declaring a variable means telling JavaScript "I'm going to use this name." You're reserving the box. You might put something in it right away or leave it empty for now.

### Purposes

- To reserve a name in a scope before using it.
- To make the variable visible to the appropriate parts of the program.
- To distinguish declared variables from accidental globals (in strict mode).
- To enable hoisting behaviour (intentional or otherwise).

### Syntax Rules and Structure

```js
var name;          // declaration without initialiser
let name;          // declaration without initialiser (TDZ ends at this line)
const name = value; // declaration with mandatory initialiser
```

**Constraints and Limitations:**
- Redeclaring a `let`/`const` name in the same scope is a SyntaxError.
- Redeclaring a `var` name is allowed (the binding is reused; the initialiser still executes).
- `const` without an initialiser is a SyntaxError.

### Annotated Code Examples

#### Example 1: Declaration Without Initialisation

```js
// Step 1: Declare a let without a value.
let pending;
// Step 2: The variable exists but has the value undefined.
console.log(pending); // undefined

// Step 3: Assign a value later.
pending = "resolved";
console.log(pending); // "resolved"
```

**Expected Output:**
```
undefined
resolved
```

**Why This Output:** `let pending;` declares the binding and initialises it to `undefined` (the TDZ ends at the declaration line). The assignment later updates the value.

### Real-World Cases

- **Deferred initialisation:** Declaring a variable at the top of a function and assigning it inside a conditional branch.
- **Loop counters:** Declaring `let i` in a `for` loop header.
- **API design:** Declaring exports before defining them in module scope.

---

## Variable Initialization

### Definitions

**Core Definition:** Initialisation is the act of assigning an initial value to a variable at the point of its declaration.

**Technical Definition:** Initialisation occurs when a declaration with an initialiser (e.g., `let x = 5`) is evaluated. For `var`, the binding is pre-initialised to `undefined` during hoisting, and the initialiser (if present) assigns the actual value at the declaration statement. For `let` and `const`, initialisation ends the TDZ and assigns the value.

**Beginner-Friendly Explanation:** Initialising a variable means giving it a starting value when you create it — like putting something in the box the moment you make it.

### Purposes

- To provide a defined starting value for the variable.
- To end the TDZ for `let` and `const` bindings.
- To make code clearer by combining declaration and value assignment.

### Syntax Rules and Structure

```js
let x = 10;       // declaration + initialisation
const y = 20;     // declaration + mandatory initialisation
var z = 30;       // declaration + initialisation (after hoisting to undefined)
```

### Annotated Code Examples

#### Example 1: Declaration and Initialisation Together

```js
// Step 1: Declare and initialise in one statement.
let score = 100;

// Step 2: Verify the value.
console.log(score); // 100
```

**Expected Output:**
```
100
```

**Why This Output:** The initialiser `100` is evaluated and assigned to the binding at the declaration line.

### Real-World Cases

- **Configuration loading:** `const config = loadConfig()` initialises a binding with a computed value.
- **Function defaults:** `let result = defaultValue` before conditional logic.
- **Module exports:** `export const VERSION = "1.0.0"`.

---

## Variable Assignment

### Definitions

**Core Definition:** Assignment is the operation of storing a value into an existing variable binding using the assignment operator (`=`).

**Technical Definition:** Assignment evaluates the right-hand side expression and calls the binding's SetMutableBinding operation (for mutable bindings) or throws a TypeError (for immutable `const` bindings). Assignment does not create a binding; it updates an existing one.

**Beginner-Friendly Explanation:** Assignment means putting a value into a variable you've already declared — like putting something into a box that already exists.

### Purposes

- To store or update the value of an existing variable.
- To perform calculations and store the result.
- To reassign variables in loops and state machines.

### Syntax Rules and Structure

```js
variable = expression;
variable += expression; // compound assignment
```

### Annotated Code Examples

#### Example 1: Basic Assignment

```js
// Step 1: Declare without initialisation.
let message;

// Step 2: Assign a value.
message = "Hello";
console.log(message); // "Hello"

// Step 3: Reassign a new value.
message = "World";
console.log(message); // "World"
```

**Expected Output:**
```
Hello
World
```

**Why This Output:** The first assignment initialises the binding; the second reassigns it. Both are valid for `let`.

### Real-World Cases

- **State updates:** `currentUser = newUser`.
- **Accumulators:** `total += item.price`.
- **Flag toggling:** `isLoading = true`.

---

## Variable Reassignment

### Definitions

**Core Definition:** Reassignment is the act of assigning a new value to a variable that has already been declared and initialised.

**Technical Definition:** Reassignment invokes SetMutableBinding on the binding record. For `let` and `var`, the binding is mutable, and the value is replaced. For `const`, the binding is immutable, and reassignment throws a TypeError in strict mode (and is a silent no-op or throws depending on the context in sloppy mode; modern engines throw).

**Beginner-Friendly Explanation:** Reassignment means changing what's inside a variable. You can do this freely with `let` and `var`, but not with `const` — `const` locks the box so you can't swap its contents for a different value.

### Purposes

- To update state as a program runs.
- To accumulate results in loops.
- To swap values (e.g., swapping two variables).

### Syntax Rules and Structure

```js
let x = 1;
x = 2;      // reassignment — allowed
const y = 1;
y = 2;      // TypeError — not allowed
```

### Annotated Code Examples

#### Example 1: Reassignment with `let`

```js
let temperature = 20;
console.log(temperature); // 20

temperature = 25;
console.log(temperature); // 25

temperature = 30;
console.log(temperature); // 30
```

**Expected Output:**
```
20
25
30
```

**Why This Output:** `let` allows multiple reassignments; each assignment replaces the previous value.

#### Example 2: Reassignment Attempt with `const`

```js
const maxRetries = 3;
try {
  maxRetries = 5;
} catch (err) {
  console.log(err.name);    // "TypeError"
  console.log(err.message); // "Assignment to constant variable."
}
```

**Expected Output:**
```
TypeError
Assignment to constant variable.
```

**Why This Output:** The `const` binding is immutable; reassignment throws a TypeError.

### Real-World Cases

- **Game loops:** Updating player position, score, and health each frame.
- **Form state:** Updating field values as the user types.
- **Retry counters:** Decrementing remaining attempts.

---

## Block Scoping

### Definitions

**Core Definition:** Block scoping means that a variable declared inside a block (`{ ... }`) is visible only within that block, not outside it.

**Technical Definition:** Blocks (including `if`, `for`, `while`, and standalone `{}`) create a new LexicalEnvironment. Declarations using `let` and `const` (and `class` in strict mode) are added to this block's declarative environment record. `var` declarations bypass block scoping and are added to the enclosing function or global environment.

**Beginner-Friendly Explanation:** Block scoping means a variable created inside a set of curly braces stays inside those braces. It's like a room: what's in the room stays in the room. `let` and `const` respect walls; `var` walks through walls.

### Purposes

- To confine variables to the smallest necessary scope, reducing naming collisions.
- To prevent loop variables from leaking into the surrounding function.
- To enable safer refactoring (moving blocks does not change variable visibility).
- To support the principle of least privilege for data access.

### Syntax Rules and Structure

```js
{
  let blockVar = "inside";
  const blockConst = "also inside";
  var functionVar = "escapes";
}
// blockVar and blockConst are not visible here.
// functionVar is visible here (function-scoped).
```

### Annotated Code Examples

#### Example 1: Block Scope with `let` vs `var`

```js
// Step 1: Declare let and var inside a block.
if (true) {
  let blockOnly = "let: inside";
  var functionWide = "var: outside";
  console.log(blockOnly);   // "let: inside"
  console.log(functionWide); // "var: outside"
}

// Step 2: Access from outside.
try {
  console.log(blockOnly); // ReferenceError
} catch (err) {
  console.log(err.name); // "ReferenceError"
}
console.log(functionWide); // "var: outside"
```

**Expected Output:**
```
let: inside
var: outside
ReferenceError
var: outside
```

**Why This Output:** `let` is confined to the `if` block; `var` leaks to the enclosing scope.

### Real-World Cases

- **Loop bodies:** `for (let i = 0; ...)` creates a fresh `i` per iteration, preventing closure bugs.
- **Conditional blocks:** Variables needed only inside an `if` branch stay there.
- **Module patterns:** Block scoping replaces IIFEs for many isolation tasks.

---

## Function Scoping

### Definitions

**Core Definition:** Function scoping means that a variable declared inside a function is visible throughout that function (and its nested functions) but not outside it.

**Technical Definition:** Every function creates a new LexicalEnvironment (function scope). `var` declarations are added to the function's VariableEnvironment; `let` and `const` declarations are added to the function's declarative LexicalEnvironment. Parameters are also function-scoped. Nested functions can access outer function variables via the scope chain (closures).

**Beginner-Friendly Explanation:** Function scoping means a variable made inside a function belongs to that function. Other functions can't see it. It's like each function has its own private notebook.

### Purposes

- To encapsulate data and prevent global pollution.
- To enable closures (functions remembering their outer variables).
- To create private state in module patterns and IIFEs.

### Syntax Rules and Structure

```js
function myFunction() {
  var functionScoped = "inside";
  let alsoInside = "inside";
  function nested() {
    console.log(functionScoped); // accessible via closure
  }
  nested();
}
// functionScoped and alsoInside are not visible here.
```

### Annotated Code Examples

#### Example 1: Function Scope and Nested Access

```js
function outer() {
  const secret = "only for outer";
  function inner() {
    console.log(secret); // accessible via closure
  }
  inner();
  console.log(secret); // accessible in outer
}

outer();
try {
  console.log(secret); // ReferenceError
} catch (err) {
  console.log(err.name); // "ReferenceError"
}
```

**Expected Output:**
```
only for outer
only for outer
ReferenceError
```

**Why This Output:** `secret` is scoped to `outer`; `inner` can access it via closure, but code outside `outer` cannot.

### Real-World Cases

- **Private variables:** The module pattern uses function scope to hide implementation details.
- **Callbacks and closures:** Event handlers capture variables from their enclosing function scope.
- **Recursion:** Helper functions defined inside a recursive function stay private.

---

## Global Scoping

### Definitions

**Core Definition:** Global scoping refers to variables that are visible throughout the entire program, accessible from any function or block.

**Technical Definition:** In script mode (non-module), top-level `var` declarations and function declarations create properties on the global object (`globalThis`). Top-level `let`, `const`, and `class` declarations create bindings in the global LexicalEnvironment's declarative record — they are globally scoped but are not properties of the global object. In module mode, top-level declarations are scoped to the module, not the global environment.

**Beginner-Friendly Explanation:** A global variable is one that every part of your program can see and use. It's like a public whiteboard. Too many globals cause clutter and bugs, so use them sparingly.

### Purposes

- To share configuration or state across the entire application.
- To provide built-in global objects (`Math`, `JSON`, `console`).
- To support legacy scripts that rely on global variables.

### Syntax Rules and Structure

```js
var globalVar = "I am global";    // becomes a property of globalThis (in scripts)
let globalLet = "I am global too"; // global lexical binding, not on globalThis
const GLOBAL_CONST = "constant";   // global lexical binding
```

### Annotated Code Examples

#### Example 1: Global `var` vs Global `let`

```js
// Step 1: Declare at top level.
var globalVar = "var value";
let globalLet = "let value";

// Step 2: Access in a function.
function checkGlobals() {
  console.log(globalVar); // "var value"
  console.log(globalLet); // "let value"
}
checkGlobals();

// Step 3: Check global object properties.
console.log(globalThis.globalVar); // "var value"
console.log(globalThis.globalLet); // undefined
```

**Expected Output:**
```
var value
let value
var value
undefined
```

**Why This Output:** Top-level `var` becomes a property of the global object; top-level `let` creates a global lexical binding but does not attach to `globalThis`.

### Real-World Cases

- **Polyfills:** Adding missing features to global objects (with caution).
- **Configuration:** Global constants like `APP_VERSION`.
- **Legacy scripts:** Older libraries that expect global jQuery, Lodash, etc.

---

## Temporal Dead Zone (TDZ)

### Definitions

**Core Definition:** The Temporal Dead Zone is the period between the start of a scope and the point where a `let` or `const` declaration is evaluated, during which the variable cannot be accessed.

**Technical Definition:** When a block is entered, `let` and `const` bindings are instantiated but remain uninitialised. The binding is in the TDZ from the start of the block until the declaration is evaluated. Any attempt to read or write the binding while it is in the TDZ throws a `ReferenceError`. The TDZ exists to catch errors where a variable is used before it is declared, even though the declaration itself is hoisted.

**Beginner-Friendly Explanation:** The TDZ is a "no-go zone" at the beginning of a block where a `let` or `const` variable exists but isn't ready yet. If you try to use it there, JavaScript throws an error instead of giving you `undefined` like `var` would.

### Purposes

- To catch use-before-declaration errors early.
- To make `let` and `const` safer than `var` by preventing silent `undefined` values.
- To ensure that a variable is always initialised before it is read.

### Syntax Rules and Structure

The TDZ applies to `let` and `const` (and `class`) declarations. The zone begins at the start of the enclosing block and ends when the declaration is evaluated.

```js
{
  // TDZ starts here
  // console.log(x); // ReferenceError
  let x = 1; // TDZ ends here
  console.log(x); // 1
}
```

### Annotated Code Examples

#### Example 1: Accessing a `let` in the TDZ

```js
{
  // Step 1: Attempt to access before declaration.
  try {
    console.log(tdz);
  } catch (err) {
    console.log(err.name);    // "ReferenceError"
    console.log(err.message); // "Cannot access 'tdz' before initialization"
  }

  // Step 2: Declare and initialise.
  let tdz = "Now available";
  console.log(tdz); // "Now available"
}
```

**Expected Output:**
```
ReferenceError
Cannot access 'tdz' before initialization
Now available
```

**Why This Output:** The binding exists but is uninitialised until the `let` line. Accessing it in the TDZ throws.

#### Example 2: TDZ Taints the Entire Block

```js
let x = "outer";

{
  // The inner let x shadows the outer x for the whole block.
  try {
    console.log(x); // ReferenceError — not "outer"
  } catch (err) {
    console.log(err.name); // "ReferenceError"
  }
  let x = "inner";
  console.log(x); // "inner"
}
```

**Expected Output:**
```
ReferenceError
inner
```

**Why This Output:** The inner `let x` hoists to the top of the block and puts the entire block in the TDZ for `x`, shadowing the outer `x` even before the inner declaration is evaluated.

### Real-World Cases

- **Debugging:** TDZ errors immediately reveal use-before-declaration bugs that `var` would hide.
- **Refactoring:** Moving a `let` declaration later in a block can surface TDZ errors if earlier code depends on it.
- **Default parameters:** TDZ errors can occur if default parameter expressions reference later parameters.

---

## Hoisting

### Definitions

**Core Definition:** Hoisting is the behaviour whereby declarations appear to be moved to the top of their scope before code execution, allowing some declarations to be referenced before their textual position.

**Technical Definition:** Hoisting is not a normative term in the ECMAScript specification, but it describes the instantiation phase of execution contexts. `var` declarations are hoisted and initialised to `undefined` (type 2 behaviour). Function declarations are fully hoisted with their bodies (type 1 behaviour). `let`, `const`, and `class` are hoisted but remain uninitialised in the TDZ (type 3 behaviour).

**Beginner-Friendly Explanation:** Hoisting is JavaScript's habit of "lifting" declarations to the top of their scope before running code. With `var`, you can use a variable before you declare it and get `undefined`. With `let` and `const`, the variable exists early but is "locked" in the TDZ until its declaration line.

### Purposes

- To explain the behaviour of `var` and function declarations in legacy code.
- To understand why `let`/`const` throw TDZ errors instead of returning `undefined`.
- To aid debugging and code comprehension.

### Syntax Rules and Structure

Hoisting applies differently by declaration type:

| Declaration | Hoisted? | Initial Value | Access Before Declaration |
|---|---|---|---|
| `var` | Yes (type 2) | `undefined` | Returns `undefined` |
| `let` | Yes (type 3) | Uninitialised (TDZ) | `ReferenceError` |
| `const` | Yes (type 3) | Uninitialised (TDZ) | `ReferenceError` |
| `function` | Yes (type 1) | Function object | Callable |
| `class` | Yes (type 3) | Uninitialised (TDZ) | `ReferenceError` |

**Constraints and Limitations:**
- Hoisting is not a literal physical move; it describes the binding instantiation order.
- `var` hoisting only affects declarations, not initialisations (assignments stay in place).

### Annotated Code Examples

#### Example 1: `var` Hoisting vs `let` TDZ

```js
// Step 1: var is hoisted to undefined.
console.log(varVar); // undefined
var varVar = "var value";

// Step 2: let is hoisted but in TDZ.
try {
  console.log(letVar); // ReferenceError
} catch (err) {
  console.log(err.name); // "ReferenceError"
}
let letVar = "let value";
```

**Expected Output:**
```
undefined
ReferenceError
```

**Why This Output:** `var` is initialised to `undefined` during hoisting; `let` remains uninitialised and throws when accessed.

#### Example 2: Function Hoisting

```js
// Step 1: Call a function before its declaration.
hoistedFunction(); // "I was hoisted!"

function hoistedFunction() {
  console.log("I was hoisted!");
}
```

**Expected Output:**
```
I was hoisted!
```

**Why This Output:** Function declarations are fully hoisted with their bodies, so they can be called before their textual position.

### Real-World Cases

- **Understanding legacy code:** `var` hoisting explains why some old code "works" when it looks like it shouldn't.
- **Debugging TDZ errors:** Knowing that `let`/`const` are hoisted but uninitialised clarifies error messages.
- **Function declaration ordering:** Hoisting allows functions to be defined after they are used, enabling flexible code organisation.

---

## Naming Conventions

### Definitions

**Core Definition:** Naming conventions are agreed-upon rules for choosing identifiers (variable names) that improve readability, consistency, and maintainability.

**Technical Definition:** JavaScript identifiers must start with a letter, `$`, or `_`; subsequent characters may also include digits. Identifiers are case-sensitive. Reserved words (`let`, `const`, `class`, etc.) cannot be used as identifiers. Conventions such as camelCase, PascalCase, and UPPER_SNAKE_CASE are stylistic standards, not language rules.

**Beginner-Friendly Explanation:** Naming conventions are the "style rules" for variable names. They make code easier to read — like using `userName` instead of `u` or `USERNAME` for a regular variable.

### Purposes

- To make code self-documenting and readable.
- To distinguish between variable types (constants, classes, booleans).
- To reduce cognitive load for readers and maintainers.
- To align with community standards and tooling expectations.

### Syntax Rules and Structure

**Common Conventions:**

| Convention | Usage | Example |
|---|---|---|
| **camelCase** | Variables, functions, parameters | `userName`, `isActive`, `totalCount` |
| **PascalCase** | Classes, constructors | `UserProfile`, `HttpClient` |
| **UPPER_SNAKE_CASE** | True constants (immutable, known at development time) | `MAX_RETRIES`, `API_BASE_URL` |
| **Leading underscore** | Historically for "private" properties (now a convention only) | `_internalState` |
| **Boolean prefix** | `is`, `has`, `can`, `should` | `isLoading`, `hasError`, `canEdit` |

**Rules:**
- Start with a letter, `$`, or `_`. Never start with a digit.
- Use descriptive names; avoid single-letter names except for loop counters (`i`, `j`) or mathematical contexts.
- Avoid underscores at the start unless following a specific convention.
- Do not use reserved words.

**Constraints and Limitations:**
- Conventions are not enforced by the language; linters and formatters enforce them in practice.
- Overly long names hurt readability; overly short names hurt clarity — balance is key.
- Some conventions are ecosystem-specific (e.g., React uses PascalCase for components, camelCase for hooks).

### Annotated Code Examples

#### Example 1: Applying Conventions

```js
// Step 1: camelCase for a mutable variable.
let userName = "Alice";

// Step 2: UPPER_SNAKE_CASE for a true constant.
const MAX_RETRY_COUNT = 3;

// Step 3: PascalCase for a class.
class UserProfile {
  constructor(name) {
    this.name = name;
  }
}

// Step 4: Boolean prefix.
let isLoading = true;
let hasPermission = false;
```

**Expected Output:** No runtime output; the example illustrates naming style.

**Why This Output:** The conventions signal the role of each identifier: `userName` is a regular variable, `MAX_RETRY_COUNT` is a constant, `UserProfile` is a class, and boolean names communicate their type.

### Real-World Cases

- **Team codebases:** Consistent naming reduces review friction and onboarding time.
- **Library APIs:** Public names follow community expectations (e.g., `fetch`, `querySelector`).
- **Linting:** ESLint rules like `camelcase` and `id-length` enforce naming conventions automatically.

---

## Naming Best Practices

### Definitions

**Core Definition:** Naming best practices are the principles for choosing clear, meaningful, and maintainable variable names beyond mere casing conventions.

**Technical Definition:** Best practices encompass descriptiveness, avoiding abbreviations, using searchable names, avoiding mental mapping, and encoding meaning without redundancy. They are guided by principles such as "code is read more often than it is written" and "a name should reveal its intent."

**Beginner-Friendly Explanation:** Good naming is about making your code understandable at a glance. Instead of `d`, use `elapsedTimeInDays`. Instead of `x`, use `userAge`. The name should tell you what the variable holds and why it exists.

### Purposes

- To make code self-explanatory without needing comments.
- To reduce the time spent deciphering unfamiliar code.
- To prevent bugs caused by confusing similarly named variables.
- To improve searchability and refactoring.

### Key Best Practices

1. **Use descriptive, pronounceable names:** `elapsedTimeInDays` not `d`.
2. **Avoid disinformation:** Don't name a list `accountList` if it's not a `List`; use `accounts`.
3. **Make meaningful distinctions:** `productInfo` vs `productData` is vague; be specific.
4. **Use searchable names:** Avoid magic numbers and single letters; constants should be named.
5. **Avoid mental mapping:** `r` for "the URL without the host and scheme" is bad; use `urlWithoutHost`.
6. **Don't encode type in the name:** `strName`, `arrItems` are redundant; the value's type is evident from usage.
7. **Use one word per concept:** Don't mix `fetch`, `retrieve`, and `get` for the same operation.
8. **Prefer positive boolean names:** `isEnabled` rather than `isNotDisabled`.
9. **Use domain language:** Name variables using the vocabulary of the problem domain (e.g., `invoice`, `shipment`).
10. **Keep names proportional to scope:** Short names are acceptable for tiny scopes (loop counters); long, descriptive names for wide scopes.

### Annotated Code Examples

#### Example 1: Improving Names

```js
// BAD: cryptic names.
function calc(a, b) {
  let x = a * b;
  return x;
}

// GOOD: descriptive names.
function calculateTotalPrice(unitPrice, quantity) {
  const totalPrice = unitPrice * quantity;
  return totalPrice;
}
```

**Expected Output:** No runtime difference; the second version is self-documenting.

**Why This Output:** The good version communicates what the function does and what its parameters mean without comments.

#### Example 2: Boolean Naming

```js
// BAD: ambiguous or negative.
let notReady = false;
let flag = true;

// GOOD: positive and descriptive.
let isReady = false;
let hasUnsavedChanges = true;
```

**Expected Output:** No runtime difference; the good names convey meaning.

**Why This Output:** Boolean prefixes (`is`, `has`) make conditional logic read naturally (`if (isReady)`).

### Real-World Cases

- **Code reviews:** Reviewers frequently flag cryptic names; descriptive names speed up reviews.
- **Open-source libraries:** Public API names are scrutinised for clarity and consistency.
- **Refactoring tools:** Renaming variables is safe when names are unique and descriptive; cryptic names make refactoring risky.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Statements and Declarations, §14 — https://tc39.es/ecma262/2026/multipage/ecmascript-language-statements-and-declarations.html
- MDN Web Docs – Hoisting — https://developer.mozilla.org/en-US/docs/Glossary/Hoisting
- MDN Web Docs – Scope — https://developer.mozilla.org/en-US/docs/Glossary/Scope
- MDN Web Docs – Storing the information you need — Variables — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Variables
- MDN Web Docs – `let` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let
- MDN Web Docs – `const` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const
- MDN Web Docs – `var` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var
- MDN Web Docs – ReferenceError: can't access lexical declaration before initialization — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Cant_access_lexical_declaration_before_init
- MDN Web Docs – Camel case — https://developer.mozilla.org/en-US/docs/Glossary/Camel_case
- W3Schools – JavaScript let — https://www.w3schools.com/js/js_let.asp
- W3Schools – JavaScript const — https://www.w3schools.com/js/js_const.asp
- W3Schools – JavaScript var — https://www.w3schools.com/js/js_var.asp
- W3Schools – JavaScript Variables — https://www.w3schools.com/js/js_variables.asp