# Higher-Order Functions — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A higher-order function is a function that either takes one or more functions as arguments, returns a function as its result, or both. This capability is built on JavaScript's treatment of functions as first-class values.

**Technical Definition:** In ECMAScript, functions are objects that can be assigned to variables, passed as arguments, and returned from other functions. A higher-order function is any function whose parameter list includes a `Function` type or whose return value is a `Function` type. The specification does not define a special syntax for higher-order functions; they emerge naturally from the language's first-class function semantics.

**Beginner-Friendly Explanation:** A higher-order function is a function that works with other functions. It can accept a function as input, give back a function as output, or do both. Think of it like a factory that takes a tool (function) and produces a new, improved tool, or a manager that delegates tasks to workers (functions) you provide.

### Key Characteristics

- **First-class functions:** Functions are values that can be stored in variables, passed around, and returned.
- **Abstraction:** Higher-order functions abstract over actions, not just values, enabling powerful code reuse.
- **Composition:** Functions can be combined to build more complex behaviour from simple pieces.
- **Closure support:** Returned functions capture their lexical environment, enabling patterns like currying and partial application.
- **Ubiquity in JavaScript:** Array methods (`map`, `filter`, `reduce`), event listeners, and promise handlers all rely on higher-order functions.

### Prerequisites

- Understanding of function declarations, expressions, and arrow functions.
- Familiarity with scope, closures, and the call stack.
- Basic knowledge of arrays and objects.

### Related Programming Areas

- **Functional programming:** Pure functions, immutability, composition, currying.
- **Asynchronous JavaScript:** Callbacks, Promises, `async/await`.
- **Event-driven programming:** Event listeners and handlers.
- **Middleware and decorators:** Express middleware, class decorators, AOP (aspect-oriented programming).

### Core Concepts / Features

---

## Functions as Values

### Definitions

**Core Definition:** In JavaScript, functions are values — they can be assigned to variables, stored in data structures, passed as arguments, and returned from other functions, just like numbers or strings.

**Technical Definition:** JavaScript is said to have **first-class functions**. Per MDN, "A programming language is said to have First-class functions when functions in that language are treated like any other variable. For example, in such a language, a function can be passed as an argument to other functions, can be returned by another function and can be assigned as a value to a variable". Functions are objects (instances of `Function`), possessing properties (`length`, `name`) and methods (`call`, `apply`, `bind`).

**Beginner-Friendly Explanation:** In JavaScript, a function is not just a block of code — it is a value, like a number or a string. You can store it in a variable, put it in an array, or hand it to another function. This is what makes higher-order functions possible.

### Purposes

- To enable functions to be stored and manipulated like any other data.
- To allow functions to be passed as arguments to other functions (callbacks).
- To allow functions to be returned from other functions (closures, factories).
- To support functional programming patterns such as composition and currying.

### Syntax Rules and Structure

```js
// Assigning a function to a variable:
const greet = function(name) { return `Hello, ${name}`; };
const greetArrow = (name) => `Hello, ${name}`;

// Storing functions in data structures:
const operations = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
};
const fns = [() => 1, () => 2, () => 3];

// Passing a function as an argument:
array.map(function(item) { return item * 2; });

// Returning a function:
function makeMultiplier(factor) {
  return function(number) {
    return number * factor;
  };
}
```

**Component Breakdown:**
- `const greet = function(...) {}`: Function expression assigned to a variable.
- `operations.add`: Function stored as an object property.
- `fns[0]`: Function stored in an array.
- `array.map(fn)`: Function passed as an argument.
- `makeMultiplier(factor)`: Returns a function that closes over `factor`.

**Syntax Rules:**
- Functions can be assigned to any variable, including `let`, `const`, and `var`.
- Functions can be stored as object properties and array elements.
- Functions can be passed as arguments wherever an expression is expected.
- Functions can be returned from any function.

**Constraints and Limitations:**
- Function declarations are hoisted; function expressions are not.
- Named function expressions have a name only inside their own body.

### Annotated Code Examples

#### Example 1: Assigning and Using Functions as Values

```js
// Step 1: Assign a function to a variable.
const double = (n) => n * 2;

// Step 2: Store functions in an object.
const mathOps = {
  double: double,
  triple: (n) => n * 3,
  square: (n) => n * n,
};

// Step 3: Store functions in an array.
const transformations = [double, mathOps.triple, mathOps.square];

// Step 4: Apply each transformation.
transformations.forEach((fn) => {
  console.log(fn(5));
});
```

**Expected Output:**
```
10
15
25
```

**Why This Output:** Each function is stored as a value. `double(5)` is `10`, `triple(5)` is `15`, and `square(5)` is `25`. The `forEach` method receives each function and calls it with `5`.

#### Example 2: Returning a Function (Closure)

```js
// Step 1: Define a function that returns a function.
function makeCounter(start = 0) {
  let count = start;
  return function() {
    count++;
    return count;
  };
}

// Step 2: Create two independent counters.
const counter1 = makeCounter();
const counter2 = makeCounter(10);

// Step 3: Each counter maintains its own state.
console.log(counter1()); // 1
console.log(counter1()); // 2
console.log(counter2()); // 11
```

**Expected Output:**
```
1
2
11
```

**Why This Output:** `makeCounter` returns a function that closes over the `count` variable. Each call to `makeCounter()` creates a new scope with its own `count`. The returned functions are values that can be stored and called independently.

### Real-World Cases

- **Event listeners:** `element.addEventListener("click", handleClick)` — `handleClick` is a function value passed as an argument.
- **Array methods:** `numbers.map(double)` — passing a function reference.
- **Factory functions:** `const createValidator = (schema) => (value) => schema.validate(value);`.

---

## Functions as Arguments

### Definitions

**Core Definition:** Passing a function as an argument means supplying a function value to another function, which can then call that function inside its body. The passed function is called a **callback**.

**Technical Definition:** When a function is passed as an argument, the receiving function can invoke it at any point during its execution. Per MDN, "A callback function is a function passed into another function as an argument, which is then invoked inside the outer function to complete some kind of routine or action". This is the foundation of callbacks and is enabled by first-class functions.

**Beginner-Friendly Explanation:** When you pass a function to another function, you're giving it instructions to run later. The outer function decides when to run your function. For example, `setTimeout(myFunction, 1000)` tells JavaScript: "Run `myFunction` after one second." The function you pass is called a callback because the outer function "calls back" to it.

### Purposes

- To customise the behaviour of a function without modifying its source code.
- To handle asynchronous operations (timers, network requests, events).
- To implement array iteration methods (`map`, `filter`, `reduce`).
- To decouple the "what" (the callback) from the "when" (the higher-order function).

### Syntax Rules and Structure

```js
// Passing a named function:
function greet(name) { return `Hello, ${name}`; }
function processUser(name, callback) {
  return callback(name);
}
processUser("Alice", greet); // "Hello, Alice"

// Passing an anonymous function:
processUser("Bob", function(name) { return `Hi, ${name}`; });

// Passing an arrow function:
processUser("Carol", (name) => `Hey, ${name}`);
```

**Component Breakdown:**
- `callback`: The parameter that receives the function value.
- `callback(name)`: The invocation of the callback inside the outer function.
- `greet`: A function reference passed as an argument (not called at the call site).

**Syntax Rules:**
- Pass the function **reference** (e.g., `greet`), not the result of calling it (e.g., `greet()`).
- The callback function can be named, anonymous, or an arrow function.
- The outer function decides when and how many times to call the callback.

**Constraints and Limitations:**
- Passing `greet()` instead of `greet` passes the return value, not the function itself.
- Callbacks can lead to "callback hell" if deeply nested; Promises and `async/await` are preferred for complex async flows.

### Annotated Code Examples

#### Example 1: Custom Array Iteration

```js
// Step 1: Define a custom forEach-like function.
function myForEach(array, callback) {
  for (let i = 0; i < array.length; i++) {
    callback(array[i], i, array);
  }
}

// Step 2: Use it with an arrow function callback.
myForEach(["a", "b", "c"], (item, index) => {
  console.log(`Index ${index}: ${item}`);
});
```

**Expected Output:**
```
Index 0: a
Index 1: b
Index 2: c
```

**Why This Output:** `myForEach` iterates over the array and calls the callback for each element, passing the element, its index, and the array itself. The callback receives these arguments and logs them.

#### Example 2: Callback with Error-First Pattern

```js
// Step 1: Simulate an async operation.
function fetchData(callback) {
  setTimeout(() => {
    const success = Math.random() > 0.5;
    if (success) {
      callback(null, { data: "Hello" });
    } else {
      callback(new Error("Network failure"));
    }
  }, 100);
}

// Step 2: Call with an error-first callback.
fetchData((err, result) => {
  if (err) {
    console.error("Error:", err.message);
  } else {
    console.log("Data:", result.data);
  }
});
```

**Expected Output (varies — either):**
```
Data: Hello
```
or
```
Error: Network failure
```

**Why This Output:** The callback receives two arguments: an error (or `null`) and the result data. This "error-first" convention (adopted by Node.js) allows the callback to handle both success and failure cases.

### Real-World Cases

- **Array methods:** `[1, 2, 3].map(n => n * 2)` — the arrow function is the callback.
- **Event handling:** `button.addEventListener("click", () => { ... })`.
- **Node.js file I/O:** `fs.readFile(path, (err, data) => { ... })` — error-first callback.
- **Timers:** `setTimeout(() => { ... }, 1000)`.

---

## Functions as Return Values

### Definitions

**Core Definition:** Returning a function from another function means the outer function produces a new function as its result, which can then be stored, passed around, or called later.

**Technical Definition:** In ECMAScript, a `return` statement can return any value, including a function object. When the returned function references variables from the outer function's scope, it forms a **closure**. Per MDN, "A function that returns a function or takes other functions as arguments is called a higher-order function".

**Beginner-Friendly Explanation:** When a function returns another function, it's like a factory that produces customised tools. You give the factory some settings, and it gives you back a tool that's pre-configured. For example, `makeMultiplier(3)` returns a function that multiplies any number by 3.

### Purposes

- To create specialised functions from general ones (factories).
- To encapsulate private state through closures.
- To enable currying and partial application.
- To implement the module pattern and configuration functions.

### Syntax Rules and Structure

```js
function outerFunction(config) {
  // ... setup logic ...
  return function innerFunction(input) {
    // ... use config and input ...
    return result;
  };
}
```

**Component Breakdown:**
- `outerFunction`: The factory that accepts configuration.
- `return function innerFunction`: The returned function that closes over `config`.
- The returned function can be anonymous or named.

**Syntax Rules:**
- The returned function captures variables from the outer function's scope (closure).
- The returned function can be called immediately or stored for later use.
- Each call to the outer function creates a new closure with its own captured variables.

**Constraints and Limitations:**
- Closures can cause memory leaks if the returned function is retained and holds large objects.
- Returned functions are not hoisted; they must be returned before use.

### Annotated Code Examples

#### Example 1: Function Factory

```js
// Step 1: Define a factory function.
function makeGreeter(greeting) {
  // Step 2: Return a function that uses the greeting.
  return function(name) {
    return `${greeting}, ${name}!`;
  };
}

// Step 3: Create specialised greeters.
const sayHello = makeGreeter("Hello");
const sayHi = makeGreeter("Hi");

// Step 4: Use the returned functions.
console.log(sayHello("Alice")); // "Hello, Alice!"
console.log(sayHi("Bob"));      // "Hi, Bob!"
```

**Expected Output:**
```
Hello, Alice!
Hi, Bob!
```

**Why This Output:** `makeGreeter` returns a function that closes over the `greeting` parameter. Each call to `makeGreeter` creates a new closure. `sayHello` and `sayHi` are independent functions with their own captured `greeting` values.

#### Example 2: Module Pattern with Private State

```js
// Step 1: Define a module factory.
const createCounter = () => {
  // Step 2: Private state — not accessible from outside.
  let count = 0;

  // Step 3: Return an object with methods that close over count.
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count,
  };
};

// Step 4: Create a counter instance.
const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.getCount()); // 2
console.log(counter.count);       // undefined (private)
```

**Expected Output:**
```
2
undefined
```

**Why This Output:** The `count` variable is enclosed in the factory function's scope. The returned methods form closures over `count`, allowing them to read and modify it. However, `count` is not a property of the returned object, so `counter.count` is `undefined`.

### Real-World Cases

- **Configuration factories:** `const createLogger = (level) => (message) => console.log(`[${level}] ${message}`);`.
- **Middleware:** `const authMiddleware = (role) => (req, res, next) => { ... };`.
- **React hooks (conceptual):** `useState` returns a state value and a setter function.

---

## Callbacks

### Definitions

**Core Definition:** A callback is a function passed as an argument to another function, which is then invoked inside that outer function to complete some action or routine.

**Technical Definition:** Per MDN, "A callback function is a function passed into another function as an argument, which is then invoked inside the outer function to complete some kind of routine or action". Callbacks are the primary mechanism for asynchronous programming in JavaScript before Promises and `async/await`. Node.js popularised the **error-first callback** convention, where the first parameter of the callback is an error object (or `null`).

**Beginner-Friendly Explanation:** A callback is a function you hand to another function and say, "Call this when you're done." For example, when you click a button, the browser calls your event handler — that handler is a callback. When a file finishes reading, Node.js calls your callback with the data. Callbacks let JavaScript do things asynchronously without blocking.

### Purposes

- To handle asynchronous operations (timers, network requests, file I/O, events).
- To customise the behaviour of functions like `map`, `filter`, and `reduce`.
- To implement event-driven programming.
- To decouple the producer of an event from its consumer.

### Syntax Rules and Structure

```js
// Named callback:
function handleResult(result) {
  console.log(result);
}
asyncOperation(handleResult);

// Anonymous callback:
asyncOperation(function(result) {
  console.log(result);
});

// Arrow function callback:
asyncOperation((result) => console.log(result));

// Error-first callback (Node.js convention):
fs.readFile(path, (err, data) => {
  if (err) { /* handle error */ }
  else { /* use data */ }
});
```

**Component Breakdown:**
- `handleResult`: A named function reference passed as a callback.
- `(err, data) => {}`: An error-first callback where `err` is `null` on success.
- The outer function (e.g., `fs.readFile`) calls the callback when the operation completes.

**Syntax Rules:**
- Pass the function reference, not the invocation (`fn`, not `fn()`).
- Callbacks can be synchronous or asynchronous depending on the outer function.
- In error-first callbacks, the first parameter is always the error (or `null`).

**Constraints and Limitations:**
- Deeply nested callbacks lead to "callback hell" (pyramid of doom).
- Error handling with callbacks requires explicit checking in each callback.
- Callbacks do not compose as cleanly as Promises.

### Annotated Code Examples

#### Example 1: Event Listener Callback

```js
// Step 1: Define a callback function.
function handleClick(event) {
  console.log("Button clicked!", event.target.id);
}

// Step 2: Register the callback.
document.getElementById("myButton")
  .addEventListener("click", handleClick);
```

**Expected Output (on click):**
```
Button clicked! myButton
```

**Why This Output:** `handleClick` is registered as a callback for the `click` event. When the user clicks the button, the browser invokes `handleClick` with the event object, which contains information about the event including the target element.

#### Example 2: Error-First Callback

```js
// Step 1: Simulate a Node.js-style async operation.
function readData(id, callback) {
  setTimeout(() => {
    if (id < 0) {
      callback(new Error("Invalid ID"), null);
    } else {
      callback(null, { id, name: "Item " + id });
    }
  }, 200);
}

// Step 2: Call with error-first callback.
readData(1, (err, data) => {
  if (err) {
    console.error("Error:", err.message);
  } else {
    console.log("Data:", data);
  }
});

readData(-1, (err, data) => {
  if (err) {
    console.error("Error:", err.message);
  } else {
    console.log("Data:", data);
  }
});
```

**Expected Output:**
```
Data: { id: 1, name: 'Item 1' }
Error: Invalid ID
```

**Why This Output:** The callback receives `(err, data)`. On success, `err` is `null` and `data` contains the result. On failure, `err` contains an `Error` object and `data` is `null`. This convention makes error handling explicit and consistent.

### Real-World Cases

- **DOM events:** `addEventListener("click", handler)`.
- **Timers:** `setTimeout(callback, delay)` and `setInterval(callback, interval)`.
- **Node.js file system:** `fs.readFile(path, (err, data) => { ... })`.
- **HTTP requests:** `xhr.onreadystatechange = () => { ... }`.

---

## Function Composition

### Definitions

**Core Definition:** Function composition is the process of combining two or more functions so that the output of one function becomes the input of the next, creating a new function.

**Technical Definition:** In mathematics, composition is defined as `(f ∘ g)(x) = f(g(x))`. In JavaScript, composition is implemented via higher-order functions like `compose` (right-to-left) and `pipe` (left-to-right). The `compose` function takes any number of functions and returns a new function that applies them in right-to-left order. The `pipe` function does the same in left-to-right order. Composition works best with unary functions (functions taking one argument).

**Beginner-Friendly Explanation:** Function composition is like an assembly line. You put raw material in one end, and it goes through a series of machines, each doing one small job. The output of one machine becomes the input of the next. The result is a finished product. In code, you chain small functions together to build complex behaviour from simple pieces.

### Purposes

- To build complex functions from simple, reusable building blocks.
- To improve readability by eliminating deeply nested function calls.
- To enable point-free programming style.
- To make data transformation pipelines explicit and testable.

### Syntax Rules and Structure

```js
// compose (right-to-left):
const compose = (...fns) => (x) =>
  fns.reduceRight((acc, fn) => fn(acc), x);

// pipe (left-to-right):
const pipe = (...fns) => (x) =>
  fns.reduce((acc, fn) => fn(acc), x);

// Usage:
const process = pipe(trim, toLowerCase, capitalize);
process("  HELLO  "); // "Hello"
```

**Component Breakdown:**
- `...fns`: Rest parameter collecting the functions to compose.
- `reduceRight`: For `compose`, functions are applied from right to left.
- `reduce`: For `pipe`, functions are applied from left to right.
- `x`: The initial input value.

**Syntax Rules:**
- Composition functions are higher-order functions that return a new function.
- The rightmost (for `compose`) or leftmost (for `pipe`) function can accept multiple arguments.
- All other functions in the chain should be unary (accept one argument).

**Constraints and Limitations:**
- Composition works best with pure functions (no side effects).
- Non-unary functions require partial application before composition.
- Debugging composed functions can be harder because stack traces may be less direct.

### Annotated Code Examples

#### Example 1: Pipe (Left-to-Right Composition)

```js
// Step 1: Define utility functions.
const trim = (str) => str.trim();
const toLowerCase = (str) => str.toLowerCase();
const capitalize = (str) => str.charAt(0).toUpperCase() + str.slice(1);

// Step 2: Define a pipe function.
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);

// Step 3: Compose the functions.
const formatName = pipe(trim, toLowerCase, capitalize);

// Step 4: Use the composed function.
console.log(formatName("  ALICE  ")); // "Alice"
```

**Expected Output:**
```
Alice
```

**Why This Output:** `pipe` applies functions left-to-right. `"  ALICE  "` is trimmed to `"ALICE"`, then lowercased to `"alice"`, then capitalised to `"Alice"`. The result is a clean, formatted name.

#### Example 2: Compose (Right-to-Left Composition)

```js
// Step 1: Define a compose function.
const compose = (...fns) => (x) =>
  fns.reduceRight((acc, fn) => fn(acc), x);

// Step 2: Define utility functions.
const add = (a) => (b) => a + b; // curried
const multiply = (a) => (b) => a * b; // curried

// Step 3: Compose with curried functions.
const add5 = add(5);
const multiply3 = multiply(3);
const process = compose(add5, multiply3);

// Step 4: Use the composed function.
console.log(process(10)); // 35 (10 * 3 + 5)
```

**Expected Output:**
```
35
```

**Why This Output:** `compose` applies functions right-to-left. `multiply3(10)` is `30`, then `add5(30)` is `35`. The curried functions allow easy partial application before composition.

### Real-World Cases

- **Data validation pipelines:** `const validate = pipe(trim, checkLength, checkFormat);`.
- **Redux middleware:** `applyMiddleware(logger, thunk, promise)`.
- **Ramda and Lodash/fp:** Provide `compose` and `pipe` utilities for functional programming.

---

## Currying

### Definitions

**Core Definition:** Currying is the technique of transforming a function that takes multiple arguments into a sequence of functions, each taking a single argument.

**Technical Definition:** Currying converts a function of arity N into N functions of arity 1. Per Refine, "Currying is a way of transforming a multi-argument function into a sequence of functions with a single argument. In other words, if a function will generally receive multiple arguments, it can be represented as a series of functions in which the first one receives one argument and then returns another function to which the next argument is passed as an argument, and so on". For example, `add(a, b)` becomes `add(a)(b)`.

**Beginner-Friendly Explanation:** Currying takes a function like `add(2, 3)` and turns it into `add(2)(3)`. Instead of giving all arguments at once, you give them one at a time. Each time you give an argument, you get back a new function that remembers what you've already given it. When you've given all the arguments, the original operation runs.

### Purposes

- To create specialised functions by partially applying arguments.
- To enable cleaner function composition (unary functions compose well).
- To improve code modularity and reusability.
- To support functional programming patterns and point-free style.

### Syntax Rules and Structure

```js
// Non-curried:
function add(a, b) {
  return a + b;
}

// Curried:
function curriedAdd(a) {
  return function(b) {
    return a + b;
  };
}

// Arrow function curried:
const curriedAdd = (a) => (b) => a + b;

// Usage:
curriedAdd(2)(3); // 5
```

**Component Breakdown:**
- `(a) => (b) => a + b`: The first function takes `a` and returns a function that takes `b`.
- The returned function closes over `a`, remembering it.
- The final call `(b)` executes the operation.

**Syntax Rules:**
- Each function in the curried chain takes exactly one argument.
- Curried functions use closures to remember previously provided arguments.
- Currying is distinct from partial application: currying always produces unary functions; partial application can fix multiple arguments at once.

**Constraints and Limitations:**
- Currying every function is impractical; use it where it improves clarity and composition.
- Deeply curried functions can be harder to read for developers unfamiliar with the pattern.
- Performance overhead from multiple function calls is usually negligible in practice.

### Annotated Code Examples

#### Example 1: Basic Currying

```js
// Step 1: Non-curried function.
function volume(length, width, height) {
  return length * width * height;
}

// Step 2: Curried version.
function curriedVolume(length) {
  return function(width) {
    return function(height) {
      return length * width * height;
    };
  };
}

// Step 3: Compare results.
console.log(volume(2, 3, 4));          // 24
console.log(curriedVolume(2)(3)(4));    // 24
```

**Expected Output:**
```
24
24
```

**Why This Output:** Both functions produce the same result. The curried version takes arguments one at a time, returning a new function at each step until all arguments are provided.

#### Example 2: Creating Specialised Functions with Currying

```js
// Step 1: Curried greeting function.
const greet = (greeting) => (name) => `${greeting}, ${name}!`;

// Step 2: Create specialised greeters.
const sayHello = greet("Hello");
const sayGoodMorning = greet("Good morning");

// Step 3: Use them.
console.log(sayHello("Alice"));          // "Hello, Alice!"
console.log(sayGoodMorning("Bob"));      // "Good morning, Bob!"
```

**Expected Output:**
```
Hello, Alice!
Good morning, Bob!
```

**Why This Output:** `greet("Hello")` returns a function that remembers `"Hello"`. That function can then be called with different names to produce consistent greetings. Each specialised greeter is reusable.

### Real-World Cases

- **Event handlers with configuration:** `const handle = (role) => (event) => { ... };`.
- **API request builders:** `const get = (baseUrl) => (path) => fetch(baseUrl + path);`.
- **Functional libraries:** Ramda provides `curry()` to auto-curry any function.

---

## Partial Application

### Definitions

**Core Definition:** Partial application is the process of fixing some arguments of a function, producing a new function that accepts the remaining arguments.

**Technical Definition:** Per Microsoft Learn, "Partial application can be described as taking a function that accepts some number of arguments, binding values to one or more of those arguments, and returning a new function that only accepts the remaining, un-bound arguments". Unlike currying, partial application can fix multiple arguments at once. In JavaScript, `Function.prototype.bind()` provides built-in partial application.

**Beginner-Friendly Explanation:** Partial application is like pre-filling part of a form. If you have a function that needs three pieces of information, you can fill in one or two now and save the rest for later. You get a new function that only needs the remaining information. Unlike currying (which always takes one argument at a time), partial application can take several arguments at once.

### Purposes

- To create specialised functions by fixing some arguments in advance.
- To reduce repetition when calling the same function with some common arguments.
- To prepare functions for composition by reducing their arity.
- To simplify callback signatures.

### Syntax Rules and Structure

```js
// Using bind for partial application:
function greet(greeting, name) {
  return `${greeting}, ${name}!`;
}
const sayHello = greet.bind(null, "Hello");
sayHello("Alice"); // "Hello, Alice!"

// Manual partial application:
function partial(fn, ...fixedArgs) {
  return function(...remainingArgs) {
    return fn(...fixedArgs, ...remainingArgs);
  };
}
```

**Component Breakdown:**
- `fn.bind(null, ...fixedArgs)`: `bind` creates a new function with `fixedArgs` pre-applied. The first argument to `bind` is `this` (use `null` for standalone functions).
- `partial(fn, ...fixedArgs)`: A custom utility that returns a function accepting the remaining arguments.

**Syntax Rules:**
- `bind` can fix any number of leading arguments.
- Manual partial application utilities use rest parameters to collect remaining arguments.
- Partial application does not require the function to be curried.

**Constraints and Limitations:**
- `bind` also changes `this`, which may be undesirable; pass `null` if `this` is not needed.
- Partial application is not built into JavaScript as a dedicated operator (though a proposal exists).

### Annotated Code Examples

#### Example 1: Partial Application with `bind`

```js
// Step 1: Define a function with multiple parameters.
function multiply(a, b, c) {
  return a * b * c;
}

// Step 2: Fix the first argument.
const double = multiply.bind(null, 2);
console.log(double(3, 4)); // 24 (2 * 3 * 4)

// Step 3: Fix the first two arguments.
const doubleOf = multiply.bind(null, 2, 3);
console.log(doubleOf(4)); // 24 (2 * 3 * 4)
```

**Expected Output:**
```
24
24
```

**Why This Output:** `bind` creates a new function with the specified arguments pre-bound. `double` always multiplies by 2 first. `doubleOf` always multiplies by 2 and 3 first. Both produce the same result for the given inputs.

#### Example 2: Manual Partial Application Utility

```js
// Step 1: Define a partial utility.
function partial(fn, ...fixedArgs) {
  return (...remainingArgs) => fn(...fixedArgs, ...remainingArgs);
}

// Step 2: Define a general function.
function log(level, timestamp, message) {
  return `[${level}] ${timestamp}: ${message}`;
}

// Step 3: Create specialised loggers.
const logInfo = partial(log, "INFO");
const logError = partial(log, "ERROR");

// Step 4: Use them.
console.log(logInfo("2026-10-13", "Server started"));
console.log(logError("2026-10-13", "Connection failed"));
```

**Expected Output:**
```
[INFO] 2026-10-13: Server started
[ERROR] 2026-10-13: Connection failed
```

**Why This Output:** The `partial` utility fixes the `level` argument, creating specialised loggers. Each logger still accepts the remaining `timestamp` and `message` arguments.

### Real-World Cases

- **Logging:** `const logInfo = log.bind(null, "INFO");`.
- **Event handlers:** `const handleUserClick = handleClick.bind(null, "user");`.
- **API requests:** `const apiGet = partial(fetch, "GET");`.

---

## Memoization

### Definitions

**Core Definition:** Memoization is an optimisation technique that caches the results of function calls and returns the cached result when the same inputs occur again.

**Technical Definition:** Per Wikipedia (as cited by multiple sources), "memoization is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls and returning the cached result when the same inputs occur again". In JavaScript, memoization is typically implemented as a higher-order function that wraps a target function with a caching layer, using a `Map` or plain object to store results keyed by argument signature.

**Beginner-Friendly Explanation:** Memoization is like keeping a cheat sheet. The first time you solve a problem, you write down the answer. Next time the same problem comes up, you just look at your cheat sheet instead of solving it again. This makes repeated calls with the same inputs much faster.

### Purposes

- To optimise expensive function calls (e.g., recursive calculations, data processing).
- To avoid redundant computations when the same inputs are used repeatedly.
- To improve performance in applications that call the same function many times.

### Syntax Rules and Structure

```js
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

**Component Breakdown:**
- `cache`: A `Map` that stores results keyed by serialised arguments.
- `JSON.stringify(args)`: Creates a string key from the arguments array.
- `cache.has(key)`: Checks if the result is already cached.
- `fn.apply(this, args)`: Calls the original function with the same context and arguments.

**Syntax Rules:**
- The key must uniquely represent the arguments; `JSON.stringify` works for simple cases.
- The cache is typically a `Map` or plain object.
- Memoization works best with pure functions (same input → same output).

**Constraints and Limitations:**
- Memoization only works for pure functions; impure functions (with side effects) may produce incorrect results.
- The cache can grow unbounded, leading to memory issues; consider LRU (Least Recently Used) eviction.
- `JSON.stringify` fails for circular references and is slow for large arguments.

### Annotated Code Examples

#### Example 1: Memoizing a Recursive Fibonacci Function

```js
// Step 1: Non-memoized Fibonacci (exponential time).
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// Step 2: Memoized version.
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = args.join(",");
    if (cache.has(key)) {
      console.log(`Cache hit for ${key}`);
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const memoFib = memoize(function fib(n) {
  if (n <= 1) return n;
  return memoFib(n - 1) + memoFib(n - 2);
});

// Step 3: Compute Fibonacci numbers.
console.log(memoFib(10)); // 55
console.log(memoFib(10)); // 55 (cache hit)
```

**Expected Output:**
```
Cache hit for 10
55
Cache hit for 10
55
```

**Why This Output:** The first call computes `fib(10)` and caches it. The second call finds `10` in the cache and returns the cached result immediately, demonstrating the performance benefit.

#### Example 2: Memoizing an Expensive Calculation

```js
// Step 1: Define an expensive function.
function expensiveSquare(n) {
  console.log(`Computing square of ${n}...`);
  return n * n;
}

// Step 2: Memoize it.
const memoizedSquare = memoize(expensiveSquare);

// Step 3: Call multiple times.
console.log(memoizedSquare(5)); // Computes and logs
console.log(memoizedSquare(5)); // Returns cache, no log
console.log(memoizedSquare(6)); // Computes and logs
```

**Expected Output:**
```
Computing square of 5...
25
25
Computing square of 6...
36
```

**Why This Output:** The first call computes and caches the result. The second call returns the cached result without logging. The third call with a new argument computes and caches a new result.

### Real-World Cases

- **React performance:** `React.memo` and `useMemo` memoize component renders and computed values.
- **API response caching:** Memoizing identical API requests to avoid redundant network calls.
- **Dynamic programming:** Fibonacci, factorial, and other recursive algorithms.

---

## Decorators

### Definitions

**Core Definition:** A decorator is a function that modifies or extends the behaviour of a class, method, property, or parameter without changing the original source code.

**Technical Definition:** Decorators are a JavaScript language feature proposed for standardisation at TC39 (currently Stage 3). Per the proposal, "Decorators are a JavaScript language feature, proposed for standardization at TC39... Decorators make class declarations programmable". A decorator is syntactically prefixed with `@` and can be applied to classes, methods, accessors, properties, or parameters. In practice, decorators are widely used through transpilers like TypeScript and Babel. The decorator function receives metadata about the target and can return a modified version.

**Beginner-Friendly Explanation:** A decorator is like a sticker you put on a class or method that adds extra behaviour. For example, you could put a `@logged` sticker on a method, and every time that method is called, it logs a message. You don't change the method's code — the decorator wraps around it and adds the logging automatically.

### Purposes

- To add behaviour (logging, validation, caching) to classes and methods without modifying their source.
- To support aspect-oriented programming (AOP) concerns like authentication and authorization.
- To improve code readability by making cross-cutting concerns explicit.
- To enable metadata annotation for frameworks and libraries.

### Syntax Rules and Structure

```js
// Decorator function:
function logged(target, key, descriptor) {
  const original = descriptor.value;
  descriptor.value = function(...args) {
    console.log(`Calling ${key} with`, args);
    return original.apply(this, args);
  };
  return descriptor;
}

// Usage:
class Example {
  @logged
  greet(name) {
    return `Hello, ${name}`;
  }
}
```

**Component Breakdown:**
- `@logged`: The decorator syntax (prefixed with `@`).
- `target`: The class prototype (for methods) or constructor (for classes).
- `key`: The name of the method/property being decorated.
- `descriptor`: The property descriptor object.

**Syntax Rules:**
- Decorators are prefixed with `@` and placed immediately before the declaration.
- Multiple decorators can be applied to the same declaration; they execute in reverse order (bottom-up).
- Decorators are not yet natively supported in all environments; transpilation (TypeScript, Babel) is typically required.

**Constraints and Limitations:**
- Decorators are a Stage 3 proposal; native browser support is incomplete.
- Decorator syntax requires transpiler configuration (e.g., TypeScript's `experimentalDecorators`).
- Overuse of decorators can make code harder to trace and debug.

### Annotated Code Examples

#### Example 1: Simple Logging Decorator

```js
// Step 1: Define a decorator function.
function logged(target, key, descriptor) {
  const originalMethod = descriptor.value;
  descriptor.value = function(...args) {
    console.log(`[LOG] ${key} called with:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`[LOG] ${key} returned:`, result);
    return result;
  };
  return descriptor;
}

// Step 2: Apply the decorator.
class Calculator {
  @logged
  add(a, b) {
    return a + b;
  }
}

// Step 3: Use the decorated method.
const calc = new Calculator();
calc.add(2, 3);
```

**Expected Output:**
```
[LOG] add called with: [ 2, 3 ]
[LOG] add returned: 5
```

**Why This Output:** The `@logged` decorator wraps the `add` method. Every call to `add` logs the arguments before execution and the result after execution, without modifying the original `add` method's code.

#### Example 2: Decorator Factory for Configuration

```js
// Step 1: Define a decorator factory.
function delay(ms) {
  return function(target, key, descriptor) {
    const original = descriptor.value;
    descriptor.value = function(...args) {
      return new Promise((resolve) => {
        setTimeout(() => resolve(original.apply(this, args)), ms);
      });
    };
    return descriptor;
  };
}

// Step 2: Apply with configuration.
class ApiClient {
  @delay(1000)
  fetchData() {
    return "data";
  }
}
```

**Expected Output (after 1 second):**
```
data
```

**Why This Output:** The `@delay(1000)` decorator factory returns a decorator that wraps the method in a `setTimeout`, delaying its execution by 1000 milliseconds. The decorator factory pattern allows parameterised decorators.

### Real-World Cases

- **Angular:** Uses decorators extensively (`@Component`, `@Injectable`, `@Input`).
- **NestJS:** Uses `@Controller`, `@Get`, `@Post` decorators for routing.
- **TypeScript:** Supports decorators for classes, methods, properties, and parameters.
- **MobX:** Uses `@observable` and `@computed` decorators for reactive state.

---

## Higher-Order Function Patterns

### Definitions

**Core Definition:** Higher-order function patterns are recurring design solutions that leverage functions as arguments or return values to solve common programming problems.

**Technical Definition:** Common higher-order function patterns include `map`, `filter`, `reduce` (collection transformation), `compose`/`pipe` (function composition), `curry`/`partial` (argument specialisation), `memoize` (caching), `debounce`/`throttle` (rate limiting), and middleware chains (request processing). These patterns are idiomatic in JavaScript and form the backbone of functional programming in the language.

**Beginner-Friendly Explanation:** Higher-order function patterns are the "recipes" developers use over and over. `map` transforms every item in an array. `filter` keeps only the items that pass a test. `reduce` combines all items into one value. `debounce` waits until the user stops typing before running a search. These patterns are everywhere in JavaScript.

### Purposes

- To provide reusable solutions to common programming problems.
- To abstract over iteration, transformation, and combination of data.
- To enable declarative, readable code.
- To encapsulate cross-cutting concerns like timing, caching, and logging.

### Syntax Rules and Structure

```js
// map — transform each element:
const doubled = [1, 2, 3].map((n) => n * 2);

// filter — keep elements that pass a test:
const evens = [1, 2, 3, 4].filter((n) => n % 2 === 0);

// reduce — combine elements into a single value:
const sum = [1, 2, 3].reduce((acc, n) => acc + n, 0);

// debounce — delay execution until calls stop:
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

// throttle — limit execution frequency:
function throttle(fn, limit) {
  let inThrottle;
  return (...args) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}
```

**Component Breakdown:**
- `map(callback)`: Calls the callback for each element and returns a new array of results.
- `filter(callback)`: Returns a new array with elements for which the callback returns `true`.
- `reduce(callback, initial)`: Accumulates a single value by calling the callback for each element.
- `debounce(fn, delay)`: Returns a function that delays `fn` until `delay` ms after the last call.
- `throttle(fn, limit)`: Returns a function that runs `fn` at most once per `limit` ms.

**Syntax Rules:**
- Array methods (`map`, `filter`, `reduce`) do not mutate the original array.
- `reduce` requires an initial value to avoid errors on empty arrays.
- `debounce` and `throttle` use closures to maintain timer state.

**Constraints and Limitations:**
- `map` and `filter` always create new arrays; use `forEach` for side effects only.
- `reduce` can be harder to read than `map` + `filter` for simple cases.
- `debounce` delays execution; `throttle` limits frequency — choose based on the use case.

### Annotated Code Examples

#### Example 1: Map, Filter, Reduce Pipeline

```js
// Step 1: Start with an array of numbers.
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Step 2: Filter even numbers.
const evens = numbers.filter((n) => n % 2 === 0);

// Step 3: Double each even number.
const doubled = evens.map((n) => n * 2);

// Step 4: Sum the results.
const sum = doubled.reduce((acc, n) => acc + n, 0);

console.log({ evens, doubled, sum });
// { evens: [2, 4, 6, 8, 10], doubled: [4, 8, 12, 16, 20], sum: 60 }
```

**Expected Output:**
```
{ evens: [2, 4, 6, 8, 10], doubled: [4, 8, 12, 16, 20], sum: 60 }
```

**Why This Output:** `filter` keeps only even numbers. `map` doubles each. `reduce` sums the doubled values. Each step returns a new array, leaving the original untouched.

#### Example 2: Debounce for Search Input

```js
// Step 1: Define a debounce utility.
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Step 2: Create a debounced search function.
const search = debounce((query) => {
  console.log(`Searching for: ${query}`);
}, 500);

// Step 3: Simulate rapid typing.
search("a");
search("ab");
search("abc");
// After 500ms, only "abc" is logged.
```

**Expected Output (after 500ms):**
```
Searching for: abc
```

**Why This Output:** Each call to `search` resets the timer. Only the last call — after the user stops typing for 500ms — actually triggers the search. This prevents excessive API calls while the user is typing.

### Real-World Cases

- **Search autocomplete:** Debouncing input to avoid firing a request on every keystroke.
- **Scroll handling:** Throttling scroll event handlers for performance.
- **Data pipelines:** Chaining `filter`, `map`, and `reduce` to transform API responses.
- **Redux middleware:** Composing middleware functions for logging, error handling, and async actions.

---

## References

- MDN Web Docs – First-class function — https://developer.mozilla.org/en-US/docs/Glossary/First-class_Function
- MDN Web Docs – Callback function — https://developer.mozilla.org/en-US/docs/Glossary/Callback_function
- MDN Web Docs – Functions — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions
- MDN Web Docs – Array.prototype.map() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map
- MDN Web Docs – Array.prototype.filter() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter
- MDN Web Docs – Array.prototype.reduce() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce
- MDN Web Docs – Function.prototype.bind() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind
- Node.js Learn – JavaScript Asynchronous Programming and Callbacks — https://nodejs.org/learn/asynchronous-work/javascript-asynchronous-programming-and-callbacks
- Refine – JavaScript Currying — https://refine.dev/blog/javascript-variadic-currying/
- SitePoint – JavaScript Decorators: What They Are and When to Use Them — https://www.sitepoint.com/javascript-decorators-what-they-are/
- TC39 – Decorators Proposal — https://github.com/tc39/proposal-decorators
- TypeScript Handbook – Decorators — https://www.typescriptlang.org/docs/handbook/decorators.html
- MDN Web Docs – Closures — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures
- Eloquent JavaScript – Higher-Order Functions — https://eloquentjavascript.net/05_higher_order.html
- Wikipedia – Memoization — https://en.wikipedia.org/wiki/Memoization
- Wikipedia – Currying — https://en.wikipedia.org/wiki/Currying
- Wikipedia – Partial application — https://en.wikipedia.org/wiki/Partial_application
- Wikipedia – Function composition (computer science) — https://en.wikipedia.org/wiki/Function_composition_(computer_science)