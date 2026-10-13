# Function Fundamentals — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A function is a reusable block of code designed to perform a specific task. Functions accept input (arguments), execute a series of statements, and optionally return a value. In JavaScript, functions are first-class objects, meaning they can be assigned to variables, passed as arguments to other functions, and returned from functions.

**Technical Definition:** Per the ECMAScript specification (ECMA-262), a function is a callable object with a `[[Call]]` internal method, and optionally a `[[Construct]]` internal method for use with the `new` operator. Every function is an instance of the `Function` constructor; `(function(){}).constructor === Function` returns `true`. Functions are also objects, possessing properties such as `length` and `name`, and methods such as `call()`, `apply()`, and `bind()`. The specification defines four syntactic forms for creating functions: function declarations, function expressions, arrow function expressions, and the `Function` constructor.

**Beginner-Friendly Explanation:** A function is like a recipe: you write down the steps once, give it a name, and then "call" it whenever you need to perform those steps. You can hand it ingredients (arguments) and it can hand you back a result (return value). JavaScript treats functions as special objects — you can store them in variables, pass them around, and even return them from other functions.

### Key Characteristics

- **First-class citizens:** Functions can be assigned to variables, passed as arguments, and returned from other functions.
- **Object nature:** Functions are objects; they have properties (`length`, `name`) and methods (`call`, `apply`, `bind`).
- **Multiple creation syntaxes:** Function declarations, function expressions (named or anonymous), arrow functions, and the `Function` constructor.
- **Hoisting behaviour:** Function declarations are hoisted with their definitions; function expressions are not.
- **Closure support:** Functions capture their lexical environment, enabling closures.
- **`this` binding:** Traditional functions have dynamic `this`; arrow functions capture `this` lexically.

### Prerequisites

- Basic JavaScript syntax (variables, statements, expressions).
- Understanding of the call stack and execution contexts.
- Familiarity with scope and closures (helpful but not required).

### Related Programming Areas

- **Functional programming:** Higher-order functions, pure functions, function composition.
- **Asynchronous JavaScript:** Callbacks, Promises, `async/await`.
- **Object-oriented programming:** Methods, constructors, prototype-based inheritance.
- **Event handling:** Event listeners and callback functions.
- **Module systems:** Exporting and importing functions.

### Core Concepts / Features

---

## Functions

### Definitions

**Core Definition:** A function is a reusable, callable block of code that performs a task or calculates a value, accepting inputs and optionally returning an output.

**Technical Definition:** In ECMAScript, a function is an object that implements the `[[Call]]` internal method. It is created by evaluating a `FunctionDeclaration`, `FunctionExpression`, `ArrowFunction`, `MethodDefinition`, or by invoking the `Function` constructor. Functions have a `prototype` property (except arrow functions), a `length` property indicating the number of formal parameters, and a `name` property containing the function's name.

**Beginner-Friendly Explanation:** A function is a named block of code you can run whenever you want. Instead of writing the same code over and over, you write it once inside a function and then "call" it by name.

### Purposes

- To encapsulate reusable code into a single named unit.
- To avoid code duplication and improve maintainability.
- To break complex problems into smaller, manageable pieces.
- To create abstractions that hide implementation details.
- To enable higher-order programming through callbacks and composition.

### Syntax Rules and Structure

```js
// Function declaration:
function name(param1, param2) {
  // statements
  return value;
}

// Function expression:
const name = function(param1, param2) {
  // statements
  return value;
};

// Arrow function expression:
const name = (param1, param2) => {
  // statements
  return value;
};

// Function constructor (not recommended):
const name = new Function('param1', 'param2', 'return param1 + param2');
```

**Component Breakdown:**
- `function` keyword: Initiates a function declaration or expression.
- `name`: The identifier bound to the function (optional in expressions).
- `(param1, param2)`: The parameter list (formal parameters).
- `{ }`: The function body containing statements.
- `return`: Optional statement that exits the function and provides a value.

**Syntax Rules:**
- Function declarations must begin with the `function` keyword followed by a required name.
- Function expressions may be anonymous (name omitted).
- Arrow functions use `=>` and do not use the `function` keyword.
- Function bodies must be enclosed in curly braces (except arrow function expression bodies).

**Constraints and Limitations:**
- The `Function` constructor is discouraged due to security (similar to `eval`) and performance issues; it also creates functions only in the global scope.
- Functions declared with `function` have their own `this` binding; arrow functions do not.
- Maximum number of formal parameters varies across engines (typically 65535 in V8).

### Annotated Code Examples

#### Example 1: Function Declaration and Invocation

```js
// Step 1: Declare a function named 'greet' with one parameter.
function greet(name) {
  // Step 2: Return a greeting string using the parameter.
  return `Hello, ${name}!`;
}

// Step 3: Call the function and store the result.
const message = greet("Alice");

// Step 4: Log the result.
console.log(message); // Output: "Hello, Alice!"
```

**Expected Output:**
```
Hello, Alice!
```

**Why This Output:** The function `greet` is declared with one parameter (`name`). When called with the argument `"Alice"`, the parameter `name` is bound to `"Alice"`, and the function returns the string `"Hello, Alice!"`.

#### Example 2: Function Expression

```js
// Step 1: Assign a function expression to a variable.
const square = function(number) {
  return number * number;
};

// Step 2: Call the function.
console.log(square(4)); // Output: 16
console.log(square(7)); // Output: 49
```

**Expected Output:**
```
16
49
```

**Why This Output:** The function expression is assigned to `square`. When `square(4)` is called, `number` is `4`, and the function returns `16`. The same function can be called repeatedly with different arguments.

### Real-World Cases

- **Utility functions:** `formatCurrency(amount)`, `validateEmail(email)` — reusable helpers used across an application.
- **Event handlers:** `function handleClick(event) { ... }` — functions passed to `addEventListener`.
- **Data transformation:** `const doubled = numbers.map(function(n) { return n * 2; })` — functions passed to array methods.

---

## Function Declarations

### Definitions

**Core Definition:** A function declaration is a statement that creates a named function binding in the current scope.

**Technical Definition:** A `FunctionDeclaration` is a HoistableDeclaration in ECMAScript. It creates a binding of a new function to a given name in the current scope. In strict mode, block-level function declarations are scoped to the block and hoisted to the top of the block. At the top level of a script, function declarations behave like `var`: they can be redeclared by another `function` or `var` declaration, but not by `let`, `const`, or `class`.

**Beginner-Friendly Explanation:** A function declaration is the standard way to define a function with a name. You write `function myFunction() { ... }` and it becomes available in the scope where you declared it. Because of hoisting, you can call it even before the line where it's defined.

### Purposes

- To create named, reusable functions that are hoisted and available throughout their scope.
- To define methods or standalone functions with clear, self-documenting names.
- To support mutual recursion (two functions that call each other).

### Syntax Rules and Structure

```js
function functionName(param1, param2, ...paramN) {
  // function body
  return expression;
}
```

**Component Breakdown:**
- `function`: Required keyword.
- `functionName`: Required identifier. Must follow JavaScript naming rules.
- `(param1, ...)`: Optional comma-separated list of formal parameters.
- `{ }`: Required block statement containing the function body.
- `return`: Optional statement specifying the return value.

**Syntax Rules:**
- The `function` keyword must be followed by the function name.
- If the function has no parameters, use empty parentheses `()`.
- The function body must be enclosed in curly braces, even if empty.
- Function declarations must appear at the top level of a script, function, or block (in strict mode).

**Constraints and Limitations:**
- In strict mode, block-level function declarations are scoped to the block and are hoisted to the top of the block.
- Function declarations cannot be used as IIFEs (Immediately Invoked Function Expressions) because the parentheses are interpreted as a grouping operator.

### Annotated Code Examples

#### Example 1: Hoisting Demonstration

```js
// Step 1: Call the function before its declaration.
console.log(add(2, 3)); // Output: 5

// Step 2: Declare the function after the call.
function add(a, b) {
  return a + b;
}

// Step 3: The function is still callable after its declaration.
console.log(add(10, 20)); // Output: 30
```

**Expected Output:**
```
5
30
```

**Why This Output:** Function declarations are hoisted to the top of their enclosing scope. The entire function definition (not just the name) is hoisted, so `add` is available and callable before its textual declaration in the code.

#### Example 2: Block-Level Function Declaration in Strict Mode

```js
"use strict";

// Step 1: The function is scoped to the block.
{
  function blockScoped() {
    return "inside block";
  }
  console.log(blockScoped()); // Output: "inside block"
}

// Step 2: Outside the block, the function is not accessible.
try {
  blockScoped();
} catch (err) {
  console.log(err.name); // "ReferenceError"
}
```

**Expected Output:**
```
inside block
ReferenceError
```

**Why This Output:** In strict mode, function declarations inside blocks are scoped to that block. They are hoisted to the top of the block but are not visible outside it.

### Real-World Cases

- **Library APIs:** `function createUser(name, email) { ... }` — exported functions in a module.
- **Utility modules:** `function debounce(fn, delay) { ... }` — reusable helpers.
- **Recursive algorithms:** `function factorial(n) { return n <= 1 ? 1 : n * factorial(n - 1); }` — self-referencing functions.

---

## Function Expressions

### Definitions

**Core Definition:** A function expression is a function defined as part of an expression, typically assigned to a variable or passed as an argument. Function expressions may be anonymous or named.

**Technical Definition:** A `FunctionExpression` is evaluated to produce a function object. Unlike function declarations, function expressions are not hoisted — the variable to which they are assigned may be hoisted (if declared with `var`), but the function definition itself is not. A function expression can be used as an IIFE (Immediately Invoked Function Expression), running as soon as it is defined.

**Beginner-Friendly Explanation:** A function expression is when you create a function as part of a larger expression, usually by assigning it to a variable: `const myFunc = function() { ... }`. You can't call it before the line where it's defined because the function itself isn't hoisted.

### Purposes

- To create functions whose names are not needed (anonymous functions).
- To assign functions to variables or object properties.
- To pass functions as arguments to other functions (callbacks).
- To create IIFEs for scope isolation.

### Syntax Rules and Structure

```js
// Anonymous function expression:
const fn = function(param) {
  return param * 2;
};

// Named function expression:
const fn = function namedFn(param) {
  return param * 2;
};

// IIFE:
(function() {
  // code runs immediately
})();
```

**Component Breakdown:**
- `function`: Required keyword (for traditional function expressions).
- `name` (optional): If provided, the name is only accessible inside the function body.
- `(param)`: Parameter list.
- `{ }`: Function body.

**Syntax Rules:**
- A function expression can be anonymous; the `name` property will be `""` (empty string) for anonymous function expressions.
- The function name in a named function expression is local to the function body only and is useful for recursion and debugging.
- An expression statement cannot begin with the `function` keyword to avoid ambiguity with function declarations; wrap the expression in parentheses for IIFEs.

**Constraints and Limitations:**
- Function expressions are not hoisted; calling the variable before the assignment results in a `TypeError` (if `var`) or a `ReferenceError` (if `let`/`const`).
- Named function expressions have a subtle scoping quirk: the name is only available inside the function body, not outside.

### Annotated Code Examples

#### Example 1: Anonymous vs Named Function Expression

```js
// Step 1: Anonymous function expression.
const anon = function(x) {
  return x * 2;
};

// Step 2: Named function expression.
const named = function factorial(n) {
  return n <= 1 ? 1 : n * factorial(n - 1);
};

// Step 3: Test both.
console.log(anon(5));    // 10
console.log(named(5));   // 120

// Step 4: The name is only accessible inside the function.
try {
  factorial(5);
} catch (err) {
  console.log(err.name); // "ReferenceError"
}
```

**Expected Output:**
```
10
120
ReferenceError
```

**Why This Output:** The anonymous function expression is assigned to `anon` and works normally. The named function expression `factorial` can call itself recursively via its internal name, but `factorial` is not accessible outside the function body.

#### Example 2: IIFE for Scope Isolation

```js
// Step 1: Variables inside the IIFE do not leak to the outer scope.
(function() {
  const privateVar = "I am private";
  console.log(privateVar); // Output: "I am private"
})();

// Step 2: The variable is not accessible outside.
try {
  console.log(privateVar);
} catch (err) {
  console.log(err.name); // "ReferenceError"
}
```

**Expected Output:**
```
I am private
ReferenceError
```

**Why This Output:** The IIFE creates a new function scope. Variables declared inside it are not accessible from the outer scope, providing encapsulation.

### Real-World Cases

- **Callbacks:** `array.map(function(item) { return item * 2; })` — anonymous function expressions passed to array methods.
- **Event listeners:** `button.addEventListener("click", function() { ... })` — anonymous handler functions.
- **Module pattern:** `const Module = (function() { ... return { publicMethod }; })();` — IIFE-based module encapsulation.

---

## Named Function Expressions

### Definitions

**Core Definition:** A named function expression is a function expression that includes a name, which is accessible only within the function's own body.

**Technical Definition:** In a `FunctionExpression` with a `BindingIdentifier`, the identifier is bound in the function's own scope (the "function name scope"), creating a self-reference. This name is not accessible in the enclosing scope. The name is useful for recursion, debugging (appears in stack traces), and self-documentation.

**Beginner-Friendly Explanation:** A named function expression is like an anonymous function expression, but you give it a name. The name is only usable inside the function itself — it's like a private label that the function uses to refer to itself.

### Purposes

- To enable recursive calls without relying on an outer variable.
- To improve stack traces by giving the function a name for debugging.
- To provide a self-documenting name that describes the function's purpose.
- To avoid polluting the enclosing scope with the function name.

### Syntax Rules and Structure

```js
const variable = function functionName(params) {
  // functionName is accessible here
  // variable is also accessible here (if in scope)
};
```

**Component Breakdown:**
- `variable`: The outer binding that holds the function.
- `functionName`: The internal name, accessible only inside the function body.
- `params`: Parameter list.
- The function body can reference both `functionName` (for recursion) and `variable` (if in scope).

**Syntax Rules:**
- The function name in a named function expression is read-only within the function body.
- The name is not accessible in the enclosing scope.
- In ES2015+, the `name` property of the function is set to the function expression's name.

**Constraints and Limitations:**
- The internal name is not writable inside the function body (in strict mode, assigning to it throws a `TypeError`).
- Named function expressions are not hoisted.

### Annotated Code Examples

#### Example 1: Recursion with a Named Function Expression

```js
// Step 1: Define a named function expression for factorial.
const factorial = function fac(n) {
  // Step 2: Use the internal name 'fac' for recursion.
  return n <= 1 ? 1 : n * fac(n - 1);
};

// Step 3: Call the function.
console.log(factorial(5)); // Output: 120

// Step 4: The internal name is not accessible outside.
try {
  fac(5);
} catch (err) {
  console.log(err.name); // "ReferenceError"
}
```

**Expected Output:**
```
120
ReferenceError
```

**Why This Output:** The function uses `fac` internally to call itself recursively. The outer variable `factorial` also refers to the same function. However, `fac` is not defined in the outer scope, so calling it there throws a `ReferenceError`.

#### Example 2: Stack Trace Identification

```js
// Step 1: Anonymous function expression.
const anonFn = function() {
  throw new Error("Anonymous error");
};

// Step 2: Named function expression.
const namedFn = function descriptiveName() {
  throw new Error("Named error");
};

// Step 3: Compare stack traces.
try { anonFn(); } catch (e) { console.log(e.stack.split("\n")[1]); }
try { namedFn(); } catch (e) { console.log(e.stack.split("\n")[1]); }
```

**Expected Output (approximate):**
```
    at anonFn (<anonymous>:2:9)
    at descriptiveName (<anonymous>:7:9)
```

**Why This Output:** The stack trace uses the function's `name` property. The anonymous function's inferred name is `anonFn` (from the variable assignment), while the named function expression's name is explicitly `descriptiveName`. In real-world debugging, explicit names provide clearer stack traces.

### Real-World Cases

- **Recursive algorithms:** Factorial, Fibonacci, tree traversal — functions that need to call themselves.
- **Debugging:** Named function expressions produce more readable stack traces in error monitoring tools.
- **Event handlers:** Naming handler functions for easier identification in DevTools.

---

## Anonymous Functions

### Definitions

**Core Definition:** An anonymous function is a function that has no name identifier. It is created by omitting the name in a function expression or by using arrow function syntax.

**Technical Definition:** An anonymous function expression has an empty string (`""`) as its `name` property, unless it is assigned to a variable or property, in which case the engine infers a name from the assignment. Only function expressions can be anonymous; function declarations must have a name.

**Beginner-Friendly Explanation:** An anonymous function is a function without a name. You typically use it when you need a quick function for a specific purpose — like a callback — and you don't need to refer to it by name elsewhere.

### Purposes

- To create short, one-off functions for callbacks and event handlers.
- To avoid polluting the namespace with unnecessary names.
- To write concise code when the function's purpose is obvious from context.

### Syntax Rules and Structure

```js
// Anonymous function expression:
const fn = function() {
  // body
};

// Anonymous arrow function:
const fn = () => {
  // body
};

// Anonymous function as callback:
array.map(function(item) {
  return item * 2;
});

// Anonymous arrow function as callback:
array.map((item) => item * 2);
```

**Component Breakdown:**
- The `function` keyword is followed immediately by `(` (no name).
- For arrow functions, the parameter list (if any) is followed by `=>`.

**Syntax Rules:**
- Anonymous function expressions cannot be used for recursion (no internal name to refer to).
- If assigned to a variable, the `name` property is inferred from the variable name (e.g., `const fn = function() {}` → `fn.name === "fn"`).
- Anonymous functions passed as arguments are common in functional programming.

**Constraints and Limitations:**
- Anonymous functions cannot refer to themselves by name (unless the variable is in scope).
- Stack traces for anonymous functions are less informative than for named functions.
- Debugging anonymous functions in DevTools is more difficult because they appear as `(anonymous)`.

### Annotated Code Examples

#### Example 1: Anonymous Function as Callback

```js
// Step 1: Array of numbers.
const numbers = [1, 2, 3, 4];

// Step 2: Use an anonymous function with map().
const doubled = numbers.map(function(n) {
  return n * 2;
});

console.log(doubled); // [2, 4, 6, 8]
```

**Expected Output:**
```
[ 2, 4, 6, 8 ]
```

**Why This Output:** The anonymous function is passed to `map()` and called for each element. It receives each number as `n` and returns `n * 2`. The result is a new array of doubled values.

#### Example 2: Inferred Name for Anonymous Functions

```js
// Step 1: Assign an anonymous function to a variable.
const myFunction = function() {};

// Step 2: Check the inferred name.
console.log(myFunction.name); // "myFunction"

// Step 3: Assign an anonymous function to an object property.
const obj = {
  myMethod: function() {}
};
console.log(obj.myMethod.name); // "myMethod"
```

**Expected Output:**
```
myFunction
myMethod
```

**Why This Output:** Although the functions are anonymous, JavaScript engines infer names from variable assignments and object property names. This improves debugging without requiring explicit names.

### Real-World Cases

- **Array methods:** `[1,2,3].filter(function(x) { return x > 1; })` — anonymous predicate functions.
- **Promise chains:** `fetch(url).then(function(response) { return response.json(); })` — anonymous handlers.
- **Timers:** `setTimeout(function() { console.log("done"); }, 1000)` — anonymous delayed callbacks.

---

## Arrow Functions

### Definitions

**Core Definition:** An arrow function is a compact function syntax that uses the `=>` operator. Arrow functions do not have their own `this`, `arguments`, `super`, or `new.target` bindings, and cannot be used as constructors.

**Technical Definition:** An `ArrowFunction` is defined in ECMAScript as an `AssignmentExpression` that produces a function object with no `prototype` property, no `[[Construct]]` internal method, and lexical `this` binding. Arrow functions cannot be used as generators (no `yield`) and cannot be used with the `new` operator — attempting to do so throws a `TypeError`. They are best suited for non-method functions and callbacks.

**Beginner-Friendly Explanation:** Arrow functions are a shorter way to write functions. Instead of `function(x) { return x * 2; }`, you write `x => x * 2`. They are especially useful for callbacks. However, they behave differently with `this` — they don't have their own `this`, so they use the `this` from the surrounding code.

### Purposes

- To write concise, readable function expressions.
- To preserve the lexical `this` binding (useful in callbacks and methods).
- To simplify array methods (`map`, `filter`, `reduce`).
- To create inline functions without the `function` keyword.

### Syntax Rules and Structure

```js
// Single parameter, implicit return:
const double = x => x * 2;

// Multiple parameters, explicit return:
const add = (a, b) => {
  return a + b;
};

// No parameters:
const greet = () => "Hello";

// Returning an object literal (wrap in parentheses):
const makeObj = () => ({ key: "value" });

// Async arrow function:
const fetchData = async () => {
  const res = await fetch("/api");
  return res.json();
};
```

**Component Breakdown:**
- Parameter list: `x`, `(a, b)`, or `()`.
- `=>`: The arrow operator.
- Body: Either an expression (implicit return) or a block `{ }` (explicit return required).

**Syntax Rules:**
- Parentheses around parameters can be omitted only if there is exactly one simple parameter.
- Braces around the body can be omitted only if the function directly returns an expression (implicit return).
- To return an object literal from an expression body, wrap it in parentheses: `() => ({ key: "value" })`.
- Arrow functions cannot contain `yield` and cannot be generators.

**Constraints and Limitations:**
- Arrow functions do not have their own `this` — they capture the `this` of the enclosing scope.
- Arrow functions do not have the `arguments` object; use rest parameters (`...args`) instead.
- Arrow functions cannot be used as constructors (`new ArrowFn()` throws `TypeError`).
- Arrow functions do not have a `prototype` property.
- Arrow functions are not suitable for methods that need dynamic `this`.

### Annotated Code Examples

#### Example 1: Implicit vs Explicit Return

```js
// Step 1: Implicit return — expression body.
const implicit = (x, y) => x + y;
console.log(implicit(2, 3)); // 5

// Step 2: Explicit return — block body.
const explicit = (x, y) => {
  const sum = x + y;
  return sum;
};
console.log(explicit(2, 3)); // 5

// Step 3: Returning an object literal requires parentheses.
const makeUser = (name) => ({ name, active: true });
console.log(makeUser("Alice")); // { name: 'Alice', active: true }
```

**Expected Output:**
```
5
5
{ name: 'Alice', active: true }
```

**Why This Output:** The implicit return expression `x + y` is automatically returned. The explicit version uses a block and must use the `return` keyword. The object literal is wrapped in parentheses so it is parsed as an expression, not a block.

#### Example 2: Lexical `this` Binding

```js
// Step 1: Object with a traditional method and an arrow function.
const counter = {
  count: 0,
  incrementTraditional: function() {
    setTimeout(function() {
      // 'this' is NOT the counter object — it's the global object (or undefined in strict mode).
      console.log("Traditional this:", this);
    }, 0);
  },
  incrementArrow: function() {
    setTimeout(() => {
      // 'this' is the counter object because arrow functions capture lexical 'this'.
      this.count++;
      console.log("Arrow this.count:", this.count);
    }, 0);
  },
};

counter.incrementTraditional(); // Logs: Traditional this: Window (or global)
counter.incrementArrow();       // Logs: Arrow this.count: 1
```

**Expected Output:**
```
Traditional this: [object Window] (or undefined in strict mode)
Arrow this.count: 1
```

**Why This Output:** The traditional function passed to `setTimeout` has its own `this` binding, which defaults to the global object (or `undefined` in strict mode). The arrow function captures the `this` of the enclosing `incrementArrow` method, which is the `counter` object, so `this.count` correctly increments.

### Real-World Cases

- **Array methods:** `[1, 2, 3].map(n => n * 2)` — concise transformations.
- **Promise chains:** `fetch(url).then(res => res.json()).then(data => console.log(data))`.
- **Event handlers (when `this` is not needed):** `button.addEventListener("click", () => this.handleClick())`.
- **React components (pre-Hooks):** Class methods using arrow functions to bind `this`.

---

## Function Hoisting

### Definitions

**Core Definition:** Function hoisting is the behaviour whereby function declarations are moved to the top of their enclosing scope before code execution, making them callable before their textual position in the code.

**Technical Definition:** In ECMAScript, `FunctionDeclaration`s are HoistableDeclarations. The specification defines a `HoistableDeclaration` production that includes function declarations, generator declarations, async function declarations, and async generator declarations. Function declarations are hoisted with their definitions (value hoisting), meaning both the name and the function body are available. Function expressions are not hoisted; only the variable to which they are assigned may be hoisted (with `undefined` as its initial value if declared with `var`).

**Beginner-Friendly Explanation:** Hoisting means JavaScript "moves" function declarations to the top of their scope before running the code. So you can call a function before you write it. But this only works for function declarations — not for function expressions or arrow functions.

### Purposes

- To allow functions to be called before their declaration in the source code.
- To enable mutual recursion without forward declarations.
- To simplify code organisation by placing helper functions after their usage.

### Syntax Rules and Structure

```js
// Function declaration — hoisted:
hoisted(); // Works
function hoisted() {
  console.log("Hoisted!");
}

// Function expression — NOT hoisted:
notHoisted(); // TypeError: notHoisted is not a function
var notHoisted = function() {
  console.log("Not hoisted!");
};
```

**Component Breakdown:**
- `function hoisted() {}`: A function declaration, fully hoisted.
- `var notHoisted = function() {}`: A function expression assigned to a `var`. The variable `notHoisted` is hoisted with value `undefined`; the function assignment happens at the line of execution.

**Syntax Rules:**
- Function declarations are hoisted to the top of their enclosing function or global scope.
- In strict mode, block-level function declarations are scoped to the block and hoisted to the top of the block.
- Function expressions are not hoisted; they behave like variable assignments.

**Constraints and Limitations:**
- Hoisting can lead to confusing code if functions are called before their declarations without clear documentation.
- `let` and `const` declarations are hoisted but remain in the Temporal Dead Zone (TDZ) until initialised; accessing them before declaration throws a `ReferenceError`.

### Annotated Code Examples

#### Example 1: Function Declaration Hoisting

```js
// Step 1: Call before declaration.
console.log(hoistedFn()); // Output: "I am hoisted"

// Step 2: Declare the function.
function hoistedFn() {
  return "I am hoisted";
}
```

**Expected Output:**
```
I am hoisted
```

**Why This Output:** The function declaration is hoisted to the top of the scope, so it is available and callable before its textual declaration.

#### Example 2: Function Expression Not Hoisted

```js
// Step 1: Attempt to call before assignment.
try {
  notHoisted();
} catch (err) {
  console.log(err.name); // "TypeError"
  console.log(err.message); // "notHoisted is not a function"
}

// Step 2: Assign the function expression.
var notHoisted = function() {
  return "Not hoisted";
};

// Step 3: Now it works.
console.log(notHoisted()); // "Not hoisted"
```

**Expected Output:**
```
TypeError
notHoisted is not a function
Not hoisted
```

**Why This Output:** The variable `notHoisted` is hoisted (as `undefined`), but the function assignment happens at the line where the expression appears. Calling it before that line attempts to call `undefined`, throwing a `TypeError`.

### Real-World Cases

- **Utility libraries:** Functions defined at the bottom of a file but used at the top benefit from hoisting.
- **Recursive algorithms:** Mutual recursion between two function declarations works because both are hoisted.
- **Code organisation:** Developers can place main logic first and helper functions later without breaking execution.

---

## Function Parameters

### Definitions

**Core Definition:** Parameters are named variables declared in a function's definition that receive values when the function is called.

**Technical Definition:** In ECMAScript, `FormalParameters` are part of the function syntax. They are bound in the function's scope upon invocation. Parameters are initialised to the values of the arguments supplied by the caller. JavaScript supports default parameters, rest parameters, and destructuring in parameter lists.

**Beginner-Friendly Explanation:** Parameters are the placeholders in a function definition. When you write `function greet(name)`, `name` is a parameter. When you call `greet("Alice")`, `"Alice"` is the argument that fills the `name` parameter.

### Purposes

- To define the inputs a function expects.
- To provide names for values used within the function body.
- To support default values and optional arguments.
- To enable variadic functions through rest parameters.

### Syntax Rules and Structure

```js
// Basic parameters:
function add(a, b) {
  return a + b;
}

// Default parameters:
function greet(name = "Guest") {
  return `Hello, ${name}!`;
}

// Rest parameters:
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

// Destructured parameters:
function createUser({ name, age = 18 }) {
  return { name, age };
}
```

**Component Breakdown:**
- `a, b`: Simple formal parameters.
- `name = "Guest"`: Default parameter — used when the argument is `undefined`.
- `...numbers`: Rest parameter — collects remaining arguments into an array.
- `{ name, age = 18 }`: Destructured parameter with default.

**Syntax Rules:**
- Parameters are comma-separated.
- Default parameters are evaluated at call time, not definition time.
- Rest parameters must be the last parameter.
- Destructured parameters can have defaults.

**Constraints and Limitations:**
- Default parameters are only applied when the argument is `undefined`, not when it is `null`.
- A function can have a maximum number of parameters (varies by engine; typically 65535 in V8).
- The `arguments` object is not available in arrow functions.

### Annotated Code Examples

#### Example 1: Default Parameters

```js
// Step 1: Define a function with a default parameter.
function greet(name = "Guest") {
  return `Hello, ${name}!`;
}

// Step 2: Call with an argument.
console.log(greet("Alice")); // "Hello, Alice!"

// Step 3: Call without an argument.
console.log(greet()); // "Hello, Guest!"

// Step 4: Call with null (null is not replaced by the default).
console.log(greet(null)); // "Hello, null!"
```

**Expected Output:**
```
Hello, Alice!
Hello, Guest!
Hello, null!
```

**Why This Output:** The default is applied only when the argument is `undefined` (or omitted). Passing `null` explicitly overrides the default.

#### Example 2: Rest Parameters

```js
// Step 1: Define a function with a rest parameter.
function sum(...numbers) {
  // Step 2: Use the numbers array.
  return numbers.reduce((total, n) => total + n, 0);
}

// Step 3: Call with varying numbers of arguments.
console.log(sum(1, 2, 3));       // 6
console.log(sum(10, 20));        // 30
console.log(sum());              // 0
```

**Expected Output:**
```
6
30
0
```

**Why This Output:** The rest parameter collects all arguments into an array. `reduce` sums the array, starting from `0`. With no arguments, the array is empty, and the result is `0`.

### Real-World Cases

- **Configuration objects:** `function createServer({ port = 3000, host = "localhost" } = {})`.
- **Math utilities:** `function multiply(...factors) { return factors.reduce((a, b) => a * b, 1); }`.
- **API clients:** `function request(url, { method = "GET", headers = {} } = {})`.

---

## Function Arguments

### Definitions

**Core Definition:** Arguments are the actual values passed to a function when it is called. They fill the parameters defined in the function's signature.

**Technical Definition:** In ECMAScript, arguments are the values supplied in a `CallExpression`'s `Arguments` production. Inside non-arrow functions, the `arguments` object provides access to all passed arguments as an array-like object, with `arguments.length` indicating the count. However, modern code should prefer rest parameters over the `arguments` object.

**Beginner-Friendly Explanation:** Arguments are the real values you give to a function when you call it. If a function is defined with `function add(a, b)`, then `add(2, 3)` passes `2` and `3` as arguments, which fill the `a` and `b` parameters.

### Purposes

- To provide input data to functions.
- To allow functions to operate on different values in different calls.
- To support variadic functions through the `arguments` object or rest parameters.

### Syntax Rules and Structure

```js
function example(a, b) {
  console.log(arguments[0]); // first argument
  console.log(arguments[1]); // second argument
  console.log(arguments.length); // number of arguments passed
}

example("x", "y"); // arguments: "x", "y"
```

**Component Breakdown:**
- `arguments`: An array-like object available in all non-arrow functions.
- `arguments[0]`: The first argument.
- `arguments.length`: The number of arguments passed.

**Syntax Rules:**
- The `arguments` object is available in all non-arrow functions.
- In strict mode, `arguments` is not linked to the formal parameters.
- Arrow functions do not have their own `arguments` object.

**Constraints and Limitations:**
- The `arguments` object is not a real array; it lacks array methods like `map`, `filter`, etc.
- Modern code should use rest parameters (`...args`) instead of `arguments`.
- `arguments` is deprecated in some contexts and is not available in arrow functions.

### Annotated Code Examples

#### Example 1: Using the `arguments` Object

```js
// Step 1: Define a function that accepts any number of arguments.
function longestString() {
  let longest = "";
  // Step 2: Iterate over the arguments object.
  for (let i = 0; i < arguments.length; i++) {
    if (arguments[i].length > longest.length) {
      longest = arguments[i];
    }
  }
  return longest;
}

// Step 3: Call with different numbers of arguments.
console.log(longestString("a", "abc", "ab")); // "abc"
console.log(longestString("hello", "hi"));    // "hello"
```

**Expected Output:**
```
abc
hello
```

**Why This Output:** The `arguments` object captures all passed arguments, regardless of how many were declared as parameters. The loop finds the longest string.

#### Example 2: Rest Parameters vs `arguments`

```js
// Step 1: Using arguments (old style).
function oldStyle() {
  // arguments is array-like, not a real array.
  console.log(Array.isArray(arguments)); // false
}

// Step 2: Using rest parameters (modern style).
function modernStyle(...args) {
  // args is a real array.
  console.log(Array.isArray(args)); // true
}

oldStyle(1, 2, 3);
modernStyle(1, 2, 3);
```

**Expected Output:**
```
false
true
```

**Why This Output:** The `arguments` object is array-like but not a true array. Rest parameters collect arguments into a real array, which supports array methods and is preferred in modern JavaScript.

### Real-World Cases

- **Variadic functions:** `Math.max(...numbers)` — functions that accept any number of arguments.
- **Logging utilities:** `function log(...messages) { console.log(...messages); }`.
- **Event emitters:** `emit(eventName, ...args)` — passing variable arguments to listeners.

---

## Return Values

### Definitions

**Core Definition:** A return value is the value that a function provides back to the code that called it, specified by the `return` statement.

**Technical Definition:** When a `return` statement is executed inside a function, the function's execution stops, and the expression following `return` is evaluated and returned to the caller. If no `return` statement is executed (or if `return` is used without an expression), the function returns `undefined`. In async functions, the returned value resolves the produced Promise. In generator functions, the returned value is placed in the `value` property of the iterator result object.

**Beginner-Friendly Explanation:** When a function finishes its work, it can "return" a result to whoever called it. If you write `return 42;`, the function gives back `42`. If you don't return anything, the function gives back `undefined`.

### Purposes

- To provide the result of a computation to the caller.
- To allow functions to be composed (the output of one becomes the input of another).
- To signal success or failure through the returned value.
- To exit a function early when a condition is met.

### Syntax Rules and Structure

```js
function functionName() {
  // ...
  return expression; // optional
}
```

**Component Breakdown:**
- `return`: The keyword that ends function execution.
- `expression` (optional): The value to return. If omitted, `undefined` is returned.

**Syntax Rules:**
- The `return` statement can only be used inside function bodies.
- A line terminator between `return` and the expression triggers ASI (Automatic Semicolon Insertion), causing the function to return `undefined`.
- To return an object literal on multiple lines, wrap it in parentheses.

**Constraints and Limitations:**
- ASI can cause unexpected `undefined` returns if `return` and the expression are on separate lines.
- A `return` inside a `try` block is deferred until the `finally` block executes.
- Functions without a `return` statement return `undefined`.

### Annotated Code Examples

#### Example 1: Basic Return Value

```js
// Step 1: Define a function that returns a value.
function square(n) {
  return n * n;
}

// Step 2: Capture the return value.
const result = square(5);
console.log(result); // 25

// Step 3: Use the return value in an expression.
console.log(square(3) + square(4)); // 9 + 16 = 25
```

**Expected Output:**
```
25
25
```

**Why This Output:** `square(5)` returns `25`, which is stored in `result`. The return value can also be used directly in expressions, as shown in the second call.

#### Example 2: ASI Hazard with `return`

```js
// Step 1: Incorrect — ASI inserts a semicolon after return.
function wrong() {
  return
    "this will not be returned";
}
console.log(wrong()); // undefined

// Step 2: Correct — expression on the same line.
function correct() {
  return "this will be returned";
}
console.log(correct()); // "this will be returned"
```

**Expected Output:**
```
undefined
this will be returned
```

**Why This Output:** In the first function, the line terminator after `return` triggers ASI, converting the code to `return; "this will not be returned";`. The function returns `undefined`. In the second function, the expression is on the same line as `return`, so the string is returned.

### Real-World Cases

- **Data transformation:** `function parseJSON(str) { return JSON.parse(str); }`.
- **Validation:** `function isValidEmail(email) { return /regex/.test(email); }` — returning boolean values.
- **Factory functions:** `function createUser(name) { return { name, createdAt: new Date() }; }`.

---

## `return` Statement

### Definitions

**Core Definition:** The `return` statement ends the execution of a function and specifies a value to be returned to the function caller.

**Technical Definition:** The `return` statement is defined in ECMAScript as `return [no LineTerminator here] Expressionopt ;`. When executed, it causes the function to cease execution and return the value of the expression (or `undefined` if the expression is omitted) to the caller. In async functions, the returned value resolves the Promise. In generator functions, the returned value is placed in the `value` property of the result object.

**Beginner-Friendly Explanation:** The `return` statement is how a function gives back a result. When JavaScript sees `return`, it stops running the function and sends the value back to whoever called it.

### Purposes

- To exit a function and provide a result.
- To terminate a function early when a condition is met.
- To return different values based on different conditions.
- To return `undefined` explicitly when needed.

### Syntax Rules and Structure

```js
return;
return expression;
```

**Component Breakdown:**
- `return;`: Returns `undefined`.
- `return expression;`: Returns the value of `expression`.

**Syntax Rules:**
- No line terminator is allowed between `return` and the expression (ASI hazard).
- `return` can only appear inside function bodies.
- In a `try` block, `finally` executes before the value is actually returned.

**Constraints and Limitations:**
- ASI can silently change the meaning of `return` statements.
- A `return` in a `finally` block overrides any previous `return` or `throw`.

### Annotated Code Examples

#### Example 1: Early Return

```js
// Step 1: Define a function with an early return.
function divide(a, b) {
  // Step 2: Check for division by zero.
  if (b === 0) {
    return "Cannot divide by zero";
  }
  // Step 3: Normal return.
  return a / b;
}

console.log(divide(10, 2)); // 5
console.log(divide(10, 0)); // "Cannot divide by zero"
```

**Expected Output:**
```
5
Cannot divide by zero
```

**Why This Output:** The early `return` in the `if` block exits the function immediately when `b` is zero. Otherwise, the normal return provides the division result.

#### Example 2: `return` Inside `try...finally`

```js
function test() {
  try {
    return "from try";
  } finally {
    console.log("finally executed");
  }
}

console.log(test());
```

**Expected Output:**
```
finally executed
from try
```

**Why This Output:** The `finally` block executes before the function actually returns. The order of output shows "finally executed" before "from try", demonstrating that `finally` runs first but does not change the return value.

### Real-World Cases

- **Guard clauses:** `if (!user) return null;` — early returns to avoid deep nesting.
- **Multiple conditions:** Returning different values based on a series of `if` statements.
- **Cleanup:** Using `try...finally` with `return` to ensure resources are released.

---

## Implicit Return in Arrow Functions

### Definitions

**Core Definition:** Implicit return is the behaviour of arrow functions where a single expression in the function body is automatically returned without using the `return` keyword.

**Technical Definition:** When an arrow function has a `ConciseBody` (an expression rather than a `FunctionBody` block), the value of that expression becomes the function's return value. This is called an implicit return. If the body is a block (`{ }`), the `return` keyword is required to return a value.

**Beginner-Friendly Explanation:** Arrow functions can be written in a short form where the expression you write is automatically returned. Instead of `x => { return x * 2; }`, you can write `x => x * 2`. The `return` is implicit.

### Purposes

- To write concise function expressions.
- To simplify callbacks and array method arguments.
- To reduce visual noise in functional-style code.

### Syntax Rules and Structure

```js
// Implicit return (expression body):
const double = x => x * 2;

// Explicit return (block body):
const double = x => {
  return x * 2;
};

// Implicit return of an object literal (parentheses required):
const makeObj = () => ({ key: "value" });
```

**Component Breakdown:**
- `x => x * 2`: The expression `x * 2` is implicitly returned.
- `x => { return x * 2; }`: The block body requires an explicit `return`.
- `() => ({ key: "value" })`: Parentheses prevent the `{` from being parsed as a block.

**Syntax Rules:**
- Implicit return works only when the arrow function body is a single expression.
- To return an object literal implicitly, wrap it in parentheses.
- If the body contains statements, braces and explicit `return` are required.

**Constraints and Limitations:**
- Implicit return cannot be used for multi-statement functions.
- Arrow functions with block bodies do not have implicit return.
- Returning `undefined` implicitly is not possible without an expression.

### Annotated Code Examples

#### Example 1: Implicit vs Explicit Return

```js
// Step 1: Implicit return — concise body.
const implicit = (a, b) => a + b;

// Step 2: Explicit return — block body.
const explicit = (a, b) => {
  return a + b;
};

console.log(implicit(3, 4)); // 7
console.log(explicit(3, 4)); // 7
```

**Expected Output:**
```
7
7
```

**Why This Output:** Both functions return `7`. The implicit version returns the expression `a + b` directly. The explicit version uses a block and must use `return`.

#### Example 2: Returning Object Literals

```js
// Step 1: Incorrect — parsed as a block, not an object.
const wrong = () => { name: "Alice" };
console.log(wrong()); // undefined

// Step 2: Correct — wrapped in parentheses.
const correct = () => ({ name: "Alice" });
console.log(correct()); // { name: "Alice" }
```

**Expected Output:**
```
undefined
{ name: 'Alice' }
```

**Why This Output:** Without parentheses, `{ name: "Alice" }` is parsed as a block statement containing a label (`name:`) and a string expression (`"Alice"`). The function returns `undefined`. With parentheses, it is parsed as an object literal expression.

### Real-World Cases

- **Array methods:** `[1, 2, 3].map(n => n * 2)` — concise transformations.
- **React components:** `const Component = () => <div>Hello</div>` — implicit return of JSX.
- **Promise chains:** `.then(data => data.json())` — concise data extraction.

---

## Function `length`

### Definitions

**Core Definition:** The `length` property of a function specifies the number of formal parameters the function expects.

**Technical Definition:** `Function.prototype.length` is a configurable property that returns an integer indicating the number of formal parameters declared in the function. It does not count rest parameters or parameters after the first default parameter. The `length` property of the `Function` constructor itself is `1` (because it takes one parameter: the function body).

**Beginner-Friendly Explanation:** The `length` property tells you how many parameters a function declares. If you write `function add(a, b)`, then `add.length` is `2`. It's a way to introspect a function's expected input count.

### Purposes

- To introspect a function's arity (number of declared parameters).
- To implement argument validation or function overloading.
- To support libraries and frameworks that adapt behaviour based on parameter count.

### Syntax Rules and Structure

```js
function example(a, b, c) {}
console.log(example.length); // 3
```

**Component Breakdown:**
- `example.length`: Returns the number of formal parameters.

**Syntax Rules:**
- Parameters after the first default parameter are not counted.
- Rest parameters are not counted.
- Destructured parameters are counted as one parameter.

**Constraints and Limitations:**
- `length` is a configurable property but is not writable.
- Arrow functions also have a `length` property.
- The `length` of a function with a rest parameter only counts parameters before the rest.

### Annotated Code Examples

#### Example 1: `length` with Various Parameter Types

```js
// Step 1: Simple parameters.
function add(a, b) {}
console.log(add.length); // 2

// Step 2: Default parameter stops counting.
function greet(name = "Guest", greeting) {}
console.log(greet.length); // 0 (first parameter has a default)

// Step 3: Rest parameter is not counted.
function sum(a, ...rest) {}
console.log(sum.length); // 1

// Step 4: Destructured parameter counts as one.
function createUser({ name, age }) {}
console.log(createUser.length); // 1
```

**Expected Output:**
```
2
0
1
1
```

**Why This Output:** `add` has two simple parameters, so `length` is `2`. `greet` has a default parameter first, so counting stops at `0`. `sum` has one parameter before the rest, so `length` is `1`. The destructured parameter counts as one formal parameter.

#### Example 2: `Function.length` vs `arguments.length`

```js
function example(a, b, c) {
  // Function.length is the number of declared parameters.
  console.log("Declared:", example.length); // 3
  // arguments.length is the number of arguments actually passed.
  console.log("Passed:", arguments.length);
}

example(1, 2); // Passed: 2
example(1, 2, 3, 4, 5); // Passed: 5
```

**Expected Output:**
```
Declared: 3
Passed: 2
Declared: 3
Passed: 5
```

**Why This Output:** `example.length` is always `3` because the function declares three parameters. `arguments.length` varies based on how many arguments are passed at each call.

### Real-World Cases

- **Function overloading:** Libraries like Ramda use `fn.length` to curry functions based on arity.
- **Argument validation:** Checking `fn.length` to ensure a callback provides the expected number of parameters.
- **Framework internals:** React and other frameworks use `length` to detect functional components.

---

## Function `name`

### Definitions

**Core Definition:** The `name` property of a function returns the function's name as a string.

**Technical Definition:** `Function.prototype.name` is a configurable, non-writable property that returns the name of the function. For function declarations, it is the declared name. For function expressions, it is the variable name (inferred) or the internal name of a named function expression. For anonymous functions, it may be `""` (empty string) or inferred from context. Arrow functions inherit the name of the variable they are assigned to.

**Beginner-Friendly Explanation:** The `name` property tells you what a function is called. If you write `function greet() {}`, then `greet.name` is `"greet"`. If you assign an anonymous function to a variable, JavaScript infers the name from the variable.

### Purposes

- To identify functions in stack traces and debugging tools.
- To support reflection and introspection in frameworks.
- To enable named function references for logging and error reporting.

### Syntax Rules and Structure

```js
function foo() {}
console.log(foo.name); // "foo"

const bar = function() {};
console.log(bar.name); // "bar"

const baz = () => {};
console.log(baz.name); // "baz"

const obj = {
  method: function() {}
};
console.log(obj.method.name); // "method"
```

**Component Breakdown:**
- `fn.name`: Returns the function's name as a string.
- Inferred names come from variable assignments, object properties, and default parameters.

**Syntax Rules:**
- Function declarations have the declared name.
- Named function expressions have the internal name.
- Anonymous function expressions assigned to variables have the variable name inferred.
- Arrow functions have the variable name inferred.
- Bound functions have the name `"bound " + originalName`.

**Constraints and Limitations:**
- `name` is not writable but is configurable.
- Anonymous functions may have `""` as their name if no inference context exists.
- Transpiled code may not preserve function names accurately.

### Annotated Code Examples

#### Example 1: Inferred Names

```js
// Step 1: Variable assignment inference.
const myFunc = function() {};
console.log(myFunc.name); // "myFunc"

// Step 2: Object property inference.
const obj = {
  myMethod: function() {}
};
console.log(obj.myMethod.name); // "myMethod"

// Step 3: Arrow function inference.
const arrow = () => {};
console.log(arrow.name); // "arrow"
```

**Expected Output:**
```
myFunc
myMethod
arrow
```

**Why This Output:** JavaScript engines infer function names from the context in which anonymous functions are created, improving debugging without requiring explicit names.

#### Example 2: Named Function Expression Name

```js
// Step 1: Named function expression.
const factorial = function fac(n) {
  return n <= 1 ? 1 : n * fac(n - 1);
};

// Step 2: The name property is "fac".
console.log(factorial.name); // "fac"
```

**Expected Output:**
```
fac
```

**Why This Output:** The `name` property of a named function expression returns the internal name (`fac`), not the variable name (`factorial`). This is useful for debugging because the function identifies itself by its internal name.

### Real-World Cases

- **Debugging:** Stack traces and DevTools display function names for easier identification.
- **Frameworks:** React uses `name` to display component names in warnings.
- **Logging:** Error logs include function names for context.

---

## References

- MDN Web Docs – Functions — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions
- MDN Web Docs – function declaration — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function
- MDN Web Docs – function expression — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/function
- MDN Web Docs – Arrow function expressions — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions
- MDN Web Docs – Hoisting — https://developer.mozilla.org/en-US/docs/Glossary/Hoisting
- MDN Web Docs – The arguments object — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments
- MDN Web Docs – return — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/return
- MDN Web Docs – Function — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function
- MDN Web Docs – Function: length — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/length
- MDN Web Docs – Function: name — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/name
- MDN Web Docs – Default parameters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters
- MDN Web Docs – Rest parameters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf