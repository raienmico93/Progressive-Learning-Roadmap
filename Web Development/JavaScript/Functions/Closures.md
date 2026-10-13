# Closures — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives a function access to its outer scope. In JavaScript, closures are created every time a function is created, at function creation time.

**Technical Definition:** Per the ECMAScript specification, a closure is formed when a function object retains a reference to its lexical environment (the `[[Environment]]` internal slot), allowing it to access variables from enclosing scopes even after the outer function has returned. A closure is the pairing of a function and the scope object in which it was created. The scope chain links from the inner function to its parent and all the way to the global object.

**Beginner-Friendly Explanation:** A closure is like a backpack that a function carries around. When you create a function inside another function, the inner function packs up the variables from its parent's scope into that backpack. Even after the parent function finishes running, the inner function still has access to those variables. It remembers where it came from.

### Key Characteristics

- **Lexical scoping:** Functions resolve variable names based on where they are declared in the source code, not where they are called.
- **Automatic creation:** Closures are created every time a function is created, at function creation time.
- **Reference capture:** Closures capture references to variables, not copies. If the outer variable changes, the closure sees the updated value.
- **State persistence:** Closures allow functions to maintain state between calls, even after the outer function has completed.
- **Encapsulation:** Closures enable private variables and methods in JavaScript.
- **Scope chain:** Every closure has access to three scopes: its own local scope, the outer function's scope, and the global scope.

### Prerequisites

- Understanding of JavaScript functions (declarations, expressions, arrow functions).
- Familiarity with scope and the scope chain.
- Basic knowledge of execution contexts and the call stack.

### Related Programming Areas

- **Functional programming:** Higher-order functions, function factories, currying.
- **Object-oriented programming:** Encapsulation, private fields, module pattern.
- **Asynchronous JavaScript:** Callbacks, event handlers, timers.
- **Memory management:** Garbage collection, memory leaks.

### Core Concepts / Features

---

## What Closures Are

### Definitions

**Core Definition:** A closure is a function that "remembers" the environment in which it was created, allowing it to access variables from that environment even after the environment no longer exists.

**Technical Definition:** A closure is formed when an inner function references a variable from an outer function's scope. The inner function retains a reference to the outer function's variable object (lexical environment), preventing it from being garbage-collected. Per MDN, "a closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment)".

**Beginner-Friendly Explanation:** When you define a function inside another function, the inner function can use the outer function's variables. Even after the outer function finishes and its variables would normally disappear, the inner function keeps them alive. That's a closure.

### Purposes

- To allow a function to access variables from its lexical scope even after that scope has exited.
- To enable data encapsulation and private state.
- To support function factories and partial application.
- To maintain state in event handlers and callbacks.

### Syntax Rules and Structure

Closures are not created by a specific syntax; they emerge automatically from JavaScript's lexical scoping rules. The general pattern is:

```js
function outer() {
  const outerVar = "I am from outer";

  function inner() {
    console.log(outerVar); // Accessing outer variable
  }

  return inner;
}

const closureFn = outer();
closureFn(); // "I am from outer"
```

**Component Breakdown:**
- `outer()`: The outer function that creates the lexical environment.
- `outerVar`: A variable in the outer function's scope.
- `inner()`: The inner function that references `outerVar`.
- `return inner`: Returns the inner function, carrying its closure with it.

**Syntax Rules:**
- Any function created inside another function forms a closure.
- Closures capture variables by reference, not by value.
- The closure retains access to the outer function's entire scope chain.

**Constraints and Limitations:**
- Closures can cause memory leaks if they hold references to large objects that are no longer needed.
- Overusing closures can impact performance due to increased memory usage.

### Annotated Code Examples

#### Example 1: Basic Closure

```js
// Step 1: Define an outer function.
function createGreeter(greeting) {
  // Step 2: Define an inner function that uses the outer variable.
  function greet(name) {
    return `${greeting}, ${name}!`;
  }
  // Step 3: Return the inner function.
  return greet;
}

// Step 4: Create a closure.
const sayHello = createGreeter("Hello");

// Step 5: Call the closure.
console.log(sayHello("Alice")); // "Hello, Alice!"
console.log(sayHello("Bob"));   // "Hello, Bob!"
```

**Expected Output:**
```
Hello, Alice!
Hello, Bob!
```

**Why This Output:** `createGreeter` returns the `greet` function, which closes over the `greeting` parameter. Even after `createGreeter` has finished executing, `sayHello` still remembers `"Hello"`. Each call to `sayHello` uses the captured `greeting` value.

#### Example 2: Counter with Closure

```js
// Step 1: Define a counter factory.
function createCounter() {
  let count = 0;

  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count,
  };
}

// Step 2: Create two independent counters.
const counter1 = createCounter();
const counter2 = createCounter();

// Step 3: Use them independently.
counter1.increment();
counter1.increment();
counter2.increment();

console.log(counter1.getCount()); // 2
console.log(counter2.getCount()); // 1
```

**Expected Output:**
```
2
1
```

**Why This Output:** Each call to `createCounter()` creates a new lexical environment with its own `count` variable. The returned methods form closures over their respective `count` variables. `counter1` and `counter2` are completely independent.

### Real-World Cases

- **React Hooks:** `useState` uses closures to maintain state between renders.
- **Event handlers:** Closures allow event handlers to remember data from the scope where they were created.
- **Module pattern:** Closures provide private state for modules.

---

## Lexical Scoping

### Definitions

**Core Definition:** Lexical scoping (also called static scoping) determines variable availability based on where variables are declared in the source code, not where functions are called.

**Technical Definition:** Lexical scoping describes how a parser resolves variable names when functions are nested. The word "lexical" refers to the fact that lexical scoping uses the location where a variable is declared within the source code to determine where that variable is available. Nested functions have access to variables declared in their outer scope. This is in contrast to dynamic scoping, where variable resolution depends on the call stack.

**Beginner-Friendly Explanation:** Lexical scoping means that where you write a function in your code determines what variables it can see. If you write a function inside another function, the inner function can see the outer function's variables. It doesn't matter where you call the function — what matters is where you defined it.

### Purposes

- To provide predictable variable resolution based on code structure.
- To enable closures by preserving scope information at function creation time.
- To support nested function hierarchies with clear access rules.

### Syntax Rules and Structure

```js
const globalVar = "global";

function outer() {
  const outerVar = "outer";

  function inner() {
    const innerVar = "inner";
    console.log(globalVar); // accessible
    console.log(outerVar);  // accessible
    console.log(innerVar);  // accessible
  }

  inner();
}
```

**Component Breakdown:**
- `globalVar`: Accessible from all scopes.
- `outerVar`: Accessible from `outer` and `inner`.
- `innerVar`: Accessible only from `inner`.

**Syntax Rules:**
- Inner functions have access to all variables in their lexical scope chain.
- Outer functions do not have access to inner function variables.
- Scope is determined at function definition time, not call time.

**Constraints and Limitations:**
- Lexical scoping is fixed at definition time; it cannot be changed at runtime.
- Shadowing occurs when an inner variable has the same name as an outer one, hiding the outer variable within that scope.

### Annotated Code Examples

#### Example 1: Lexical Scoping Demonstration

```js
// Step 1: Define a global variable.
const name = "Global";

function outer() {
  // Step 2: Define a variable in outer scope.
  const name = "Outer";

  function inner() {
    // Step 3: Inner function sees "Outer", not "Global".
    console.log(name);
  }

  inner();
}

outer(); // "Outer"
console.log(name); // "Global"
```

**Expected Output:**
```
Outer
Global
```

**Why This Output:** The inner function `inner` resolves `name` lexically — it looks in its own scope first (not found), then in `outer`'s scope (found `"Outer"`). The global `name` is shadowed by the outer function's `name`. Outside `outer`, the global `name` is still `"Global"`.

#### Example 2: Lexical vs. Dynamic Scoping

```js
const value = "global";

function printValue() {
  console.log(value);
}

function run() {
  const value = "local";
  printValue(); // Lexical scoping: prints "global"
}

run();
```

**Expected Output:**
```
global
```

**Why This Output:** With lexical scoping, `printValue` resolves `value` based on where it was defined (the global scope), not where it was called (inside `run`). If JavaScript used dynamic scoping, it would print `"local"`.

### Real-World Cases

- **Nested functions:** Any function defined inside another function uses lexical scoping.
- **Modules:** ES modules use lexical scoping to isolate variables.
- **Global vs. local variables:** Understanding lexical scoping prevents accidental variable shadowing.

---

## Closure Scope

### Definitions

**Core Definition:** Closure scope is the set of variables that a closure can access — its own local variables, the variables of all enclosing functions, and global variables.

**Technical Definition:** Every closure has access to three scopes: its own block scope (local variables and arguments), its outer function's block scope (variables and arguments of the parent function), and the global scope (anything defined globally). The scope chain links these scopes together, and variable resolution proceeds from the innermost scope outward.

**Beginner-Friendly Explanation:** When a closure looks for a variable, it searches in its own scope first. If it doesn't find it there, it looks in the outer function's scope. If it still doesn't find it, it looks in the global scope. This chain of scopes is called the scope chain.

### Purposes

- To provide a structured hierarchy for variable resolution.
- To allow inner functions to access outer variables while keeping outer scopes isolated.
- To enable data encapsulation through scope isolation.

### Syntax Rules and Structure

```js
const globalVar = "global";

function outer() {
  const outerVar = "outer";

  function inner() {
    const innerVar = "inner";
    // Scope chain: inner -> outer -> global
    console.log(innerVar);  // 1st: found in inner
    console.log(outerVar);  // 2nd: found in outer
    console.log(globalVar); // 3rd: found in global
  }

  return inner;
}
```

**Component Breakdown:**
- **Local scope:** Variables declared in the closure's own function body.
- **Outer function scope:** Variables from the enclosing function(s).
- **Global scope:** Variables declared at the top level.

**Syntax Rules:**
- Variable lookup follows the scope chain from innermost to outermost.
- The first matching variable is used (shadowing).
- If no variable is found, a `ReferenceError` is thrown.

**Constraints and Limitations:**
- Deeper scope chains mean more lookup steps, which can slightly affect performance.
- Shadowing can cause confusion if not carefully managed.

### Annotated Code Examples

#### Example 1: Scope Chain Resolution

```js
// Step 1: Global variable.
const level = "global";

function outer() {
  // Step 2: Outer variable (shadows global).
  const level = "outer";

  function middle() {
    // Step 3: Middle variable (shadows outer).
    const level = "middle";

    function inner() {
      // Step 4: Inner has no 'level', so it searches the chain.
      console.log(level); // Finds "middle"
    }

    inner();
  }

  middle();
}

outer();
```

**Expected Output:**
```
middle
```

**Why This Output:** `inner` has no local `level` variable. It searches the scope chain: `middle` has `level = "middle"`, so that value is used. The outer and global `level` variables are shadowed.

#### Example 2: Accessing Global Variables Through the Chain

```js
const globalCounter = { count: 0 };

function createCounter() {
  function increment() {
    globalCounter.count++;
    return globalCounter.count;
  }
  return increment;
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

**Expected Output:**
```
1
2
```

**Why This Output:** The `increment` closure has no local `globalCounter`, so it searches the scope chain and finds it in the global scope. Each call mutates the global object.

### Real-World Cases

- **Nested modules:** Inner functions access module-level configuration.
- **Event handling:** Event handlers access variables from their enclosing component scope.
- **Recursive functions:** Inner functions access outer scope for memoization caches.

---

## Capturing Variables

### Definitions

**Core Definition:** Capturing variables means that a closure retains a reference to the variables in its lexical scope, allowing it to read and modify them even after the outer function has finished executing.

**Technical Definition:** When a function is declared, variables in the context of its declaration are captured in its scope. Closures capture references to variables, not copies. If the outer variable changes, the closure sees the updated value. This concept of capturing scope is significant because the closure can use and modify variables from an outer scope even after the outer scope exits.

**Beginner-Friendly Explanation:** When a closure captures a variable, it doesn't take a snapshot of the value at that moment. Instead, it keeps a live connection to the variable. If the variable changes later, the closure sees the new value. This is like having a remote control to a TV — you're not carrying a picture of what's on the screen; you're carrying the ability to see what's currently playing.

### Purposes

- To allow functions to maintain state across multiple calls.
- To enable data encapsulation and private variables.
- To support function factories that produce customised functions.
- To maintain context in asynchronous callbacks.

### Syntax Rules and Structure

```js
function outer() {
  let capturedVar = "initial";

  function inner() {
    console.log(capturedVar); // Captures the variable
  }

  capturedVar = "modified"; // Closure sees this change
  return inner;
}

const fn = outer();
fn(); // "modified" (not "initial")
```

**Component Breakdown:**
- `capturedVar`: The variable being captured.
- `inner`: The function that captures `capturedVar`.
- The closure sees the latest value of `capturedVar`, not the value at capture time.

**Syntax Rules:**
- Closures capture variables by reference, not by value.
- All closures created in the same scope share access to the same variables.
- Changes to captured variables are visible to all closures sharing that scope.

**Constraints and Limitations:**
- Shared variable capture can lead to unintended behaviour, especially in loops.
- Capturing large objects can prevent garbage collection.

### Annotated Code Examples

#### Example 1: Capturing by Reference

```js
// Step 1: Outer function with a captured variable.
function createLogger() {
  let messageCount = 0;

  return function log(message) {
    messageCount++;
    console.log(`[${messageCount}] ${message}`);
  };
}

// Step 2: Create a logger.
const logger = createLogger();

// Step 3: Each call increments the captured variable.
logger("First");  // [1] First
logger("Second"); // [2] Second
logger("Third");  // [3] Third
```

**Expected Output:**
```
[1] First
[2] Second
[3] Third
```

**Why This Output:** The returned `log` function captures `messageCount` by reference. Each call increments the same variable, maintaining state across calls. The closure keeps `messageCount` alive even after `createLogger` has finished.

#### Example 2: Multiple Closures Sharing a Variable

```js
function createPair() {
  let sharedValue = 0;

  return {
    increment: () => ++sharedValue,
    decrement: () => --sharedValue,
    getValue: () => sharedValue,
  };
}

const pair = createPair();
pair.increment();
pair.increment();
pair.decrement();
console.log(pair.getValue()); // 1
```

**Expected Output:**
```
1
```

**Why This Output:** All three methods (`increment`, `decrement`, `getValue`) are closures over the same `sharedValue` variable. They share access to it, so changes made by one are visible to the others.

### Real-World Cases

- **State management:** Closures capture state in React hooks and Redux.
- **Memoization:** Closures capture cache objects for memoized functions.
- **Configuration:** Factory functions capture configuration parameters.

---

## Closure Use Cases

### Definitions

**Core Definition:** Closure use cases are the practical scenarios in which closures provide unique value, including data encapsulation, function factories, event handling, and callbacks.

**Technical Definition:** Closures are useful because they associate data (the lexical environment) with a function that operates on that data. This is similar to object-oriented programming, where an object associates data (properties) with one or more methods. Closures can be used to emulate private methods, create function factories, and maintain state in asynchronous operations.

**Beginner-Friendly Explanation:** Closures are used everywhere in JavaScript. They help you create private variables, make functions that remember things, handle events, and manage asynchronous code. Whenever you need a function to "remember" something from where it was created, you're using a closure.

### Purposes

- To create private variables and methods (encapsulation).
- To build function factories that produce customised functions.
- To maintain state in event handlers and callbacks.
- To implement the module pattern for code organisation.
- To support partial application and currying.

### Syntax Rules and Structure

Closures are used implicitly in many patterns. Common use cases include:

```js
// 1. Private variables:
function createAccount() {
  let balance = 0;
  return {
    deposit: (amount) => balance += amount,
    getBalance: () => balance,
  };
}

// 2. Function factories:
function makeMultiplier(factor) {
  return (number) => number * factor;
}

// 3. Event handlers:
function setupButton() {
  let clickCount = 0;
  document.getElementById("btn").addEventListener("click", () => {
    clickCount++;
    console.log(`Clicked ${clickCount} times`);
  });
}
```

**Component Breakdown:**
- **Private variables:** Variables in the outer function scope, accessible only through returned methods.
- **Function factories:** Outer functions that return customised inner functions.
- **Event handlers:** Closures that capture component state and respond to user actions.

**Syntax Rules:**
- Closures are created automatically; no special syntax is needed.
- Use closures when you need to associate state with a function.
- Avoid unnecessary closures in performance-critical code.

**Constraints and Limitations:**
- Overusing closures can increase memory consumption.
- Closures in loops can cause unexpected behaviour if not handled correctly.

### Annotated Code Examples

#### Example 1: Private Variables with Closures

```js
// Step 1: Create an account factory.
function createAccount(initialBalance) {
  // Step 2: Private variable — not accessible from outside.
  let balance = initialBalance;

  return {
    deposit: (amount) => {
      balance += amount;
      return balance;
    },
    withdraw: (amount) => {
      if (amount > balance) {
        throw new Error("Insufficient funds");
      }
      balance -= amount;
      return balance;
    },
    getBalance: () => balance,
  };
}

// Step 3: Create an account.
const account = createAccount(100);
account.deposit(50);
console.log(account.getBalance()); // 150
console.log(account.balance);       // undefined (private)
```

**Expected Output:**
```
150
undefined
```

**Why This Output:** The `balance` variable is enclosed in `createAccount`'s scope. The returned methods form closures over `balance`, allowing them to read and modify it. However, `balance` is not a property of the returned object, so `account.balance` is `undefined`.

#### Example 2: Function Factory

```js
// Step 1: Define a factory function.
function createValidator(min, max) {
  return function(value) {
    if (value < min || value > max) {
      return `${value} is out of range [${min}, ${max}]`;
    }
    return `${value} is valid`;
  };
}

// Step 2: Create specialised validators.
const validateAge = createValidator(0, 120);
const validateScore = createValidator(0, 100);

// Step 3: Use them.
console.log(validateAge(25));    // "25 is valid"
console.log(validateAge(150));   // "150 is out of range [0, 120]"
console.log(validateScore(85));  // "85 is valid"
```

**Expected Output:**
```
25 is valid
150 is out of range [0, 120]
85 is valid
```

**Why This Output:** `createValidator` returns a function that closes over `min` and `max`. Each validator remembers its own bounds. `validateAge` and `validateScore` are independent closures with different captured values.

### Real-World Cases

- **React hooks:** `useState`, `useEffect`, and `useCallback` all rely on closures.
- **Redux:** Reducers and action creators use closures to access state.
- **Node.js middleware:** Express middleware functions are closures over configuration.
- **Memoization:** Caches are stored in closures for functions like `memoize`.

---

## Module Pattern

### Definitions

**Core Definition:** The module pattern is a design pattern that uses closures to create private and public members, providing encapsulation and namespace management in JavaScript.

**Technical Definition:** The module pattern uses closures to create private and public members, providing encapsulation and namespace management in JavaScript. The most maintainable approach uses IIFE (Immediately Invoked Function Expression) or ES6 modules for encapsulation. At its core, the module pattern leverages scope, closures, and (commonly) IIFE's. The module pattern allows for public and private access levels, with modules being Immediately-Invoked-Function-Expressions (IIFE) that create a private scope through closures.

**Beginner-Friendly Explanation:** The module pattern is a way to organise code into self-contained units. Inside a module, you have private variables that only the module can see, and public methods that outside code can use. The module wraps everything in a function that runs immediately, and the public API is returned as an object.

### Purposes

- To encapsulate implementation details and expose only a public API.
- To prevent global namespace pollution.
- To create private variables and methods.
- To organise code into logical, maintainable units.

### Syntax Rules and Structure

```js
const ModuleName = (function() {
  // Private variables and functions
  let privateVar = "private";

  function privateMethod() {
    return privateVar;
  }

  // Public API
  return {
    publicMethod: function() {
      return privateMethod();
    },
  };
})();
```

**Component Breakdown:**
- `(function() { ... })()`: An IIFE that creates a new scope.
- `privateVar`: A private variable accessible only inside the module.
- `return { ... }`: The public API object exposed to the outside world.

**Syntax Rules:**
- The IIFE runs immediately, creating the closure.
- Only the returned object's properties are accessible from outside.
- Private members are not accessible from outside the module.

**Constraints and Limitations:**
- The module pattern is less common with ES6 modules, which provide native encapsulation.
- Module pattern closures can cause memory leaks if not managed properly.

### Annotated Code Examples

#### Example 1: Basic Module Pattern

```js
// Step 1: Create a module using IIFE.
const Calculator = (function() {
  // Step 2: Private variable.
  let result = 0;

  // Step 3: Private function.
  function log(message) {
    console.log(`[Calculator] ${message}`);
  }

  // Step 4: Public API.
  return {
    add(x) {
      result += x;
      log(`Added ${x}, result: ${result}`);
      return this;
    },
    subtract(x) {
      result -= x;
      log(`Subtracted ${x}, result: ${result}`);
      return this;
    },
    getResult() {
      return result;
    },
    reset() {
      result = 0;
      log("Reset");
      return this;
    },
  };
})();

// Step 5: Use the module.
Calculator.add(5).add(3).subtract(2);
console.log(Calculator.getResult()); // 6
console.log(Calculator.result);      // undefined (private)
```

**Expected Output:**
```
[Calculator] Added 5, result: 5
[Calculator] Added 3, result: 8
[Calculator] Subtracted 2, result: 6
6
undefined
```

**Why This Output:** The IIFE creates a private scope where `result` and `log` live. The returned object exposes public methods that close over these private members. `Calculator.result` is `undefined` because `result` is not a property of the returned object.

#### Example 2: Revealing Module Pattern

```js
// Step 1: Create a module with the revealing pattern.
const UserManager = (function() {
  // Step 2: Private state.
  const users = [];
  let currentId = 1;

  // Step 3: Private functions.
  function generateId() {
    return currentId++;
  }

  function findById(id) {
    return users.find((user) => user.id === id);
  }

  // Step 4: Public functions.
  function addUser(name, email) {
    const user = { id: generateId(), name, email };
    users.push(user);
    return user;
  }

  function getUser(id) {
    return findById(id);
  }

  function getAllUsers() {
    return [...users]; // Return a copy
  }

  // Step 5: Reveal public API.
  return {
    add: addUser,
    get: getUser,
    getAll: getAllUsers,
  };
})();

// Step 6: Use the module.
const user = UserManager.add("John", "john@example.com");
console.log(user);           // { id: 1, name: 'John', email: '...' }
console.log(UserManager.getAll()); // [{ id: 1, ... }]
console.log(UserManager.generateId); // undefined (private)
```

**Expected Output:**
```
{ id: 1, name: 'John', email: 'john@example.com' }
[ { id: 1, name: 'John', email: 'john@example.com' } ]
undefined
```

**Why This Output:** The revealing module pattern defines all functions privately and then reveals only the ones intended for public use. `generateId` remains private, while `add`, `get`, and `getAll` are exposed.

### Real-World Cases

- **jQuery plugins:** Many jQuery plugins use the module pattern for encapsulation.
- **Utility libraries:** Lodash and Underscore use module-like patterns.
- **Legacy codebases:** The module pattern was the standard before ES6 modules.

---

## Private Variables

### Definitions

**Core Definition:** Private variables are variables that are accessible only within a specific scope (usually a function or module) and cannot be accessed or modified from outside that scope.

**Technical Definition:** JavaScript does not have built-in support for private variables in the classical sense, but closures can emulate private state. By declaring a variable within a function and returning methods that close over it, the variable becomes inaccessible from outside the function. The simplest way to make a property private is by creating a variable within the constructor function, changing its scope to be within the constructor function versus available globally.

**Beginner-Friendly Explanation:** A private variable is like a secret that only certain functions know about. You create it inside a function, and only the functions defined inside that same function can see or change it. Outside code has no way to access it directly.

### Purposes

- To hide implementation details from external code.
- To prevent accidental modification of internal state.
- To provide controlled access through public methods.
- To implement encapsulation similar to other programming languages.

### Syntax Rules and Structure

```js
function createObject() {
  // Private variable — not accessible from outside
  let privateVar = "secret";

  return {
    getPrivate: () => privateVar,
    setPrivate: (value) => { privateVar = value; },
  };
}

const obj = createObject();
console.log(obj.getPrivate()); // "secret"
console.log(obj.privateVar);   // undefined
```

**Component Breakdown:**
- `privateVar`: Declared inside the function, inaccessible from outside.
- `getPrivate` / `setPrivate`: Public methods that close over `privateVar`.

**Syntax Rules:**
- Private variables are declared in an outer function scope.
- Access is provided through closures (public methods).
- There is no direct property access to private variables.

**Constraints and Limitations:**
- Each instance of the object gets its own private variables (per-closure).
- Private variables cannot be accessed by subclasses or external code.
- Memory overhead: each closure keeps its own copy of private variables.

### Annotated Code Examples

#### Example 1: Private Variable with Controlled Access

```js
// Step 1: Create a factory with private state.
function createTemperature() {
  // Step 2: Private variable.
  let celsius = 0;

  return {
    getCelsius: () => celsius,
    getFahrenheit: () => (celsius * 9) / 5 + 32,
    setCelsius: (value) => {
      if (value < -273.15) {
        throw new RangeError("Temperature below absolute zero");
      }
      celsius = value;
    },
  };
}

// Step 3: Use the controlled access.
const temp = createTemperature();
temp.setCelsius(25);
console.log(temp.getCelsius());    // 25
console.log(temp.getFahrenheit()); // 77

try {
  temp.setCelsius(-300);
} catch (err) {
  console.log(err.name); // "RangeError"
}
```

**Expected Output:**
```
25
77
RangeError
```

**Why This Output:** The `celsius` variable is private. It can only be modified through `setCelsius`, which enforces validation. This prevents external code from setting invalid values.

#### Example 2: Private Counter with Multiple Instances

```js
function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    getCount: () => count,
  };
}

const a = createCounter();
const b = createCounter();

a.increment();
a.increment();
b.increment();

console.log(a.getCount()); // 2
console.log(b.getCount()); // 1
```

**Expected Output:**
```
2
1
```

**Why This Output:** Each call to `createCounter()` creates a new private `count` variable. `a` and `b` have completely independent state.

### Real-World Cases

- **Bank accounts:** Private balance with public deposit/withdraw methods.
- **User sessions:** Private session data with public access methods.
- **Caching:** Private cache stores with public get/set methods.

---

## Function Factories

### Definitions

**Core Definition:** A function factory is a function that creates and returns other functions, often using closures to capture configuration parameters.

**Technical Definition:** Function factories are higher-order functions that return functions tailored to a specific purpose. The returned function is a closure: it remembers the bounds from its outer scope and uses them every time it validates a value. Examples like this take advantage of JavaScript's closures to implement what is called a "factory". Factory functions are used in place of constructors, and the factory function pattern is similar to constructors.

**Beginner-Friendly Explanation:** A function factory is a function that makes other functions. You give it some settings, and it gives you back a customised function that remembers those settings. For example, `makeMultiplier(3)` returns a function that multiplies any number by 3.

### Purposes

- To create specialised functions from a general template.
- To reduce code duplication by generating similar functions programmatically.
- To encapsulate configuration in closures.
- To support partial application and currying.

### Syntax Rules and Structure

```js
function makeFunction(config) {
  return function(input) {
    // Use config and input
    return result;
  };
}
```

**Component Breakdown:**
- `makeFunction`: The factory that accepts configuration.
- `config`: Captured by the returned function's closure.
- The returned function uses `config` each time it's called.

**Syntax Rules:**
- The factory returns a function (usually an inner function).
- The returned function closes over the factory's parameters.
- Each call to the factory creates a new closure with its own captured values.

**Constraints and Limitations:**
- Each factory call creates a new closure, consuming memory.
- Function factories are not hoisted like function declarations.

### Annotated Code Examples

#### Example 1: Multiplier Factory

```js
// Step 1: Define a multiplier factory.
function makeMultiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

// Step 2: Create specialised multipliers.
const double = makeMultiplier(2);
const triple = makeMultiplier(3);
const quadruple = makeMultiplier(4);

// Step 3: Use them.
console.log(double(5));    // 10
console.log(triple(5));    // 15
console.log(quadruple(5)); // 20
```

**Expected Output:**
```
10
15
20
```

**Why This Output:** `makeMultiplier` returns a function that closes over `factor`. Each multiplier remembers its own `factor` value. `double` always multiplies by 2, `triple` by 3, and so on.

#### Example 2: Validator Factory

```js
// Step 1: Define a range validator factory.
function createRangeValidator(min, max) {
  return function(value) {
    if (typeof value !== "number") {
      return { valid: false, error: "Not a number" };
    }
    if (value < min || value > max) {
      return { valid: false, error: `Must be between ${min} and ${max}` };
    }
    return { valid: true };
  };
}

// Step 2: Create validators.
const validateAge = createRangeValidator(0, 120);
const validateScore = createRangeValidator(0, 100);

// Step 3: Use them.
console.log(validateAge(25));    // { valid: true }
console.log(validateAge(150));   // { valid: false, error: "Must be between 0 and 120" }
console.log(validateScore(85));  // { valid: true }
```

**Expected Output:**
```
{ valid: true }
{ valid: false, error: 'Must be between 0 and 120' }
{ valid: true }
```

**Why This Output:** The factory captures `min` and `max` in a closure. Each validator remembers its own bounds. `validateAge` and `validateScore` are independent closures with different captured values.

### Real-World Cases

- **Validation libraries:** Factories for creating field validators.
- **API clients:** Factories for creating configured HTTP clients.
- **Logging:** Factories for creating loggers with specific levels.
- **React:** Higher-order components (HOCs) are function factories.

---

## Event Handlers

### Definitions

**Core Definition:** Event handlers are functions that respond to user interactions or system events, and they commonly use closures to maintain state and access component data.

**Technical Definition:** Event handlers are callbacks registered to respond to events. The event handler function is a closure — a function in which non-local variables are bound to ("closed over") another scope. Event handlers can share their closing environments, just like other functions can. Closures in event listeners keep variables alive — they cannot be garbage collected while the listener is active.

**Beginner-Friendly Explanation:** When you attach an event handler to a button, the handler function remembers the variables from where it was defined. This lets the handler access things like a counter or configuration values. Even though the handler runs later (when the user clicks), it still has access to those variables.

### Purposes

- To maintain state across multiple event invocations.
- To access component data from within event handlers.
- To avoid global variables for event-related state.
- To create dynamic handlers with captured configuration.

### Syntax Rules and Structure

```js
function setupButton() {
  let clickCount = 0;

  document.getElementById("btn").addEventListener("click", function() {
    clickCount++;
    console.log(`Clicked ${clickCount} times`);
  });
}
```

**Component Breakdown:**
- `clickCount`: Captured variable maintained across clicks.
- The event handler is a closure over `clickCount`.
- Each click increments the same captured variable.

**Syntax Rules:**
- Event handlers are functions passed to `addEventListener`.
- They capture variables from their enclosing scope.
- The captured variables persist as long as the event listener is registered.

**Constraints and Limitations:**
- Closures in event listeners can cause memory leaks if not removed.
- Using `var` in loops can lead to all handlers sharing the same variable.

### Annotated Code Examples

#### Example 1: Click Counter with Closure

```js
// Step 1: Create a button and attach a handler with closure.
function setupClickCounter() {
  let count = 0;

  document.getElementById("myButton").addEventListener("click", () => {
    count++;
    console.log(`Button clicked ${count} time(s)`);
  });
}

// Step 2: Call the setup function.
setupClickCounter();

// When the button is clicked:
// "Button clicked 1 time(s)"
// "Button clicked 2 time(s)"
// ... and so on
```

**Expected Output (on clicks):**
```
Button clicked 1 time(s)
Button clicked 2 time(s)
```

**Why This Output:** The arrow function captures `count` from the enclosing scope. Each click increments the same variable, maintaining state across clicks. The closure keeps `count` alive for the lifetime of the event listener.

#### Example 2: Event Handler in a Loop (Pitfall and Fix)

```js
// WRONG: Using var — all handlers share the same variable.
const buttons = document.querySelectorAll(".btn");
for (var i = 0; i < buttons.length; i++) {
  buttons[i].addEventListener("click", function() {
    console.log(`Button ${i} clicked`); // Always logs the final value of i
  });
}

// CORRECT: Using let — each iteration gets its own binding.
for (let i = 0; i < buttons.length; i++) {
  buttons[i].addEventListener("click", function() {
    console.log(`Button ${i} clicked`); // Logs the correct index
  });
}
```

**Expected Output (with `let`):**
```
Button 0 clicked
Button 1 clicked
Button 2 clicked
```

**Why This Output:** With `var`, there is only one `i` binding for the entire loop. All handlers close over the same variable, which ends up being the final value. With `let`, each iteration creates a new `i` binding, so each handler captures its own value.

### Real-World Cases

- **UI components:** Buttons, forms, and interactive elements.
- **React:** Event handlers in functional components use closures over props and state.
- **Vue:** Methods and computed properties use closures.
- **Game development:** Keyboard and mouse event handlers.

---

## Callbacks

### Definitions

**Core Definition:** Callbacks are functions passed as arguments to other functions, which are then invoked inside the outer function to complete some action or routine.

**Technical Definition:** Callbacks are functions passed into other functions as arguments, which are then executed inside the outer function, providing a way to delay execution or maintain sequence in asynchronous operations. Closures are used with callbacks to reference variables from the outer scope, enabling the callback to maintain state and context. A callback using context variables is a closure.

**Beginner-Friendly Explanation:** A callback is a function you give to another function, saying "call me back when you're done." The callback remembers the variables from where it was created, so it can use them when it eventually runs. This is how JavaScript handles asynchronous operations.

### Purposes

- To handle asynchronous operations (timers, network requests, events).
- To customise the behaviour of functions like `map`, `filter`, and `reduce`.
- To maintain state and context across asynchronous boundaries.
- To implement event-driven programming.

### Syntax Rules and Structure

```js
function doSomething(callback) {
  // ... do work ...
  callback(result);
}

doSomething(function(result) {
  console.log(result);
});
```

**Component Breakdown:**
- `callback`: The function passed as an argument.
- The callback closes over variables from its creation scope.
- The outer function invokes the callback when appropriate.

**Syntax Rules:**
- Pass the function reference, not the invocation (`fn`, not `fn()`).
- Callbacks can be named, anonymous, or arrow functions.
- Callbacks capture variables from their lexical scope.

**Constraints and Limitations:**
- Deeply nested callbacks lead to "callback hell."
- Error handling requires explicit checking in each callback.
- Callbacks do not compose as cleanly as Promises.

### Annotated Code Examples

#### Example 1: Asynchronous Callback with Closure

```js
// Step 1: Function that simulates async work.
function fetchData(id, callback) {
  setTimeout(() => {
    const data = { id, name: `Item ${id}` };
    callback(null, data);
  }, 1000);
}

// Step 2: Use a callback that captures configuration.
function createFetcher(prefix) {
  return function(id) {
    fetchData(id, (err, data) => {
      if (err) {
        console.error(`${prefix}: Error`);
      } else {
        console.log(`${prefix}: ${data.name}`);
      }
    });
  };
}

// Step 3: Create a fetcher with a captured prefix.
const userFetcher = createFetcher("USER");
userFetcher(1); // After 1s: "USER: Item 1"
```

**Expected Output (after 1 second):**
```
USER: Item 1
```

**Why This Output:** The callback passed to `fetchData` closes over the `prefix` variable from `createFetcher`. Even though the callback runs asynchronously (after 1 second), it still has access to `prefix`.

#### Example 2: Callback with Accumulated State

```js
// Step 1: Create a function that accumulates state.
function createAccumulator() {
  let total = 0;

  return function(value, callback) {
    total += value;
    callback(total);
  };
}

// Step 2: Use the accumulator.
const addAndReport = createAccumulator();

addAndReport(10, (sum) => console.log(`Sum: ${sum}`)); // Sum: 10
addAndReport(20, (sum) => console.log(`Sum: ${sum}`)); // Sum: 30
addAndReport(5, (sum) => console.log(`Sum: ${sum}`));  // Sum: 35
```

**Expected Output:**
```
Sum: 10
Sum: 30
Sum: 35
```

**Why This Output:** The returned function closes over `total`. Each call updates `total` and passes it to the callback. The callback receives the updated value. The closure maintains state across multiple calls.

### Real-World Cases

- **Node.js file I/O:** `fs.readFile(path, (err, data) => { ... })`.
- **Array methods:** `[1, 2, 3].map(n => n * 2)`.
- **Timers:** `setTimeout(callback, delay)`.
- **Promises:** `.then(callback)` and `.catch(callback)`.

---

## Memory Considerations

### Definitions

**Core Definition:** Memory considerations for closures involve understanding how closures retain references to variables, which can prevent garbage collection and potentially lead to memory leaks.

**Technical Definition:** Closures retain references to outer scope variables. When a closure survives the function returning (which can happen for several reasons), the variable object for that function call is retained in memory by the reference from the closure. Closures that unintentionally keep references to large objects in their own scope can cause memory leaks. The most common causes are forgotten event listeners, unclosed timers, and circular references in closures.

**Beginner-Friendly Explanation:** A closure holds onto the variables it uses. If those variables are large (like a big array) or reference DOM elements, they stay in memory as long as the closure exists. If you're not careful, this can make your application use more and more memory over time.

### Purposes

- To understand when closures cause memory issues.
- To learn techniques for preventing closure-related memory leaks.
- To optimise applications that create many closures.

### Syntax Rules and Structure

Memory leaks occur when closures retain references to objects that are no longer needed:

```js
// Memory leak: closure holds entire array
function createHandler(data) {
  return function() {
    console.log(data[0]); // Only needs first element
  };
}
const handler = createHandler(hugeArray); // hugeArray stays in memory

// Fix: extract only needed data
function createHandler(data) {
  const firstItem = data[0]; // Extract only what's needed
  return function() {
    console.log(firstItem);
  };
}
const handler = createHandler(hugeArray); // Only firstItem kept
```

**Component Breakdown:**
- **Leak:** The closure captures `data`, keeping the entire array alive.
- **Fix:** Extract only the needed value (`firstItem`) before creating the closure.

**Syntax Rules:**
- Closures capture references, not copies.
- Large objects captured by closures prevent garbage collection.
- Event listeners and timers that are not cleaned up keep closures alive.

**Constraints and Limitations:**
- Memory leaks are difficult to detect without profiling tools.
- Chrome DevTools can identify memory leaks through heap snapshots.

### Annotated Code Examples

#### Example 1: Memory Leak with Large Arrays

```js
// Step 1: Create a function that leaks memory.
function createLeakyHandler() {
  const hugeArray = new Array(1000000).fill("data");

  return function() {
    console.log(hugeArray[0]); // Only needs first element
  };
}

// Step 2: The closure keeps hugeArray alive.
const handler = createLeakyHandler();
handler(); // "data"
// hugeArray remains in memory even though only [0] is needed
```

**Expected Output:**
```
data
```

**Why This Output (and the Problem):** The closure captures `hugeArray` by reference. Even though only `hugeArray[0]` is used, the entire million-element array stays in memory because the closure holds a reference to it. This is a memory leak.

**Fix:**
```js
function createFixedHandler() {
  const hugeArray = new Array(1000000).fill("data");
  const firstItem = hugeArray[0]; // Extract only needed data

  return function() {
    console.log(firstItem);
  };
}
```

#### Example 2: Event Listener Leak

```js
// Step 1: Leak — listener never removed.
function setupButton() {
  const data = new Array(100000).fill("x");
  const button = document.getElementById("btn");

  button.addEventListener("click", () => {
    console.log(data[0]);
  });

  button.remove(); // Button removed from DOM, but listener remains
}

// Step 2: Fix — remove listener before removing element.
function setupButtonFixed() {
  const data = new Array(100000).fill("x");
  const button = document.getElementById("btn");

  const handler = () => {
    console.log(data[0]);
  };

  button.addEventListener("click", handler);
  button.removeEventListener("click", handler);
  button.remove();
}
```

**Expected Output:** (No visible output, but the fix prevents the leak.)

**Why This Output:** In the leaky version, the event listener (and its closure over `data`) remains in memory even after the button is removed. In the fixed version, `removeEventListener` is called before `button.remove()`, allowing the closure and `data` to be garbage-collected.

### Real-World Cases

- **Single-page applications:** Long-running SPAs that create and destroy components.
- **Event-heavy interfaces:** Applications with many event listeners.
- **Timer-based polling:** Intervals that are not cleared.

---

## Closure Pitfalls

### Definitions

**Core Definition:** Closure pitfalls are common mistakes and unexpected behaviours that arise from misunderstanding how closures capture variables.

**Technical Definition:** Closures can lead to memory leaks if not managed properly, especially when they capture variables that are no longer needed. They can also make debugging more difficult due to the complexity of the scope chain, and can cause performance issues if overused or used inappropriately. Additionally, closures can cause unintended variable sharing, especially in loops.

**Beginner-Friendly Explanation:** Closures are powerful but tricky. Common mistakes include: using `var` in loops (all closures share the same variable), capturing large objects unnecessarily (memory leaks), and creating too many closures (performance overhead). Understanding these pitfalls helps you avoid bugs.

### Purposes

- To recognise and avoid common closure-related bugs.
- To write more predictable and maintainable code.
- To optimise performance by using closures judiciously.

### Syntax Rules and Structure

Common pitfalls and their solutions:

```js
// Pitfall 1: var in loops
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // Logs 3, 3, 3
}

// Fix: use let
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // Logs 0, 1, 2
}

// Pitfall 2: Capturing large objects
function leaky(data) {
  return () => console.log(data[0]); // Holds entire array
}

// Fix: extract only what's needed
function fixed(data) {
  const first = data[0];
  return () => console.log(first);
}

// Pitfall 3: Unintended sharing
function createFunctions() {
  const fns = [];
  for (var i = 0; i < 3; i++) {
    fns.push(() => console.log(i)); // All share same i
  }
  return fns;
}
```

**Component Breakdown:**
- **`var` in loops:** Single function-scoped binding shared by all closures.
- **Large object capture:** Closure holds reference to the entire object.
- **Unintended sharing:** Multiple closures share the same variable.

**Syntax Rules:**
- Use `let` and `const` for block-scoped variables in loops.
- Extract only needed values before creating closures.
- Be aware of which variables a closure captures.

**Constraints and Limitations:**
- Even experienced developers can miss closure pitfalls.
- Code reviews and linters help catch common mistakes.

### Annotated Code Examples

#### Example 1: The `var` Loop Pitfall

```js
// Step 1: WRONG — using var.
const functions = [];
for (var i = 0; i < 3; i++) {
  functions.push(function() {
    console.log(i);
  });
}

functions[0](); // 3 (not 0)
functions[1](); // 3 (not 1)
functions[2](); // 3 (not 2)
```

**Expected Output:**
```
3
3
3
```

**Why This Output:** With `var`, there is only one `i` variable for the entire loop. All closures capture the same variable, which ends up being `3` after the loop finishes.

**Fix:**
```js
const functions = [];
for (let i = 0; i < 3; i++) {
  functions.push(function() {
    console.log(i);
  });
}

functions[0](); // 0
functions[1](); // 1
functions[2](); // 2
```

#### Example 2: Memory Leak from Unnecessary Capture

```js
// Step 1: Function that captures too much.
function processData() {
  const rawData = fetchHugeDataset(); // Large object
  const summary = rawData.summary;

  return function() {
    // Only needs summary, but captures rawData too
    console.log(summary);
  };
}

// Step 2: Fix — extract only what's needed.
function processDataFixed() {
  const rawData = fetchHugeDataset();
  const summary = rawData.summary;
  // rawData goes out of scope and can be garbage-collected

  return function() {
    console.log(summary);
  };
}
```

**Expected Output:** (Both work, but the fixed version uses less memory.)

**Why This Output:** In the first version, the returned closure captures `rawData` because it's in the same scope. In the fixed version, only `summary` is captured, allowing `rawData` to be garbage-collected.

### Real-World Cases

- **Legacy code:** `var`-based loops with closures are common in older codebases.
- **Performance-critical code:** Overusing closures in tight loops can impact performance.
- **Memory-sensitive applications:** Mobile and embedded applications need careful closure management.

---

## References

- MDN Web Docs – Closures — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures
- MDN Web Docs – Functions — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions
- MDN Web Docs – Lexical scoping — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures#lexical_scoping
- Microsoft Learn – JavaScript, the underappreciated language — https://learn.microsoft.com/en-us/archive/blogs/ericlippert/javascript-the-underappreciated-language
- CoreUI – How to implement module pattern in JavaScript — https://coreui.io/answers/how-to-implement-module-pattern-in-javascript/
- CoreUI – How to avoid memory leaks in JavaScript — https://coreui.io/answers/how-to-avoid-memory-leaks-in-javascript/
- GreatFrontEnd – What are the potential pitfalls of using closures? — https://github.com/greatfrontend/top-javascript-interview-questions
- Pluralsight – Function Factories: Simplify Your Code and Reduce Redundancy — https://www.pluralsight.com/labs/codeLabs/function-factories-simplify-your-code-and-reduce-redundancy
- SitePoint – Demystifying JavaScript Closures, Callbacks and IIFEs — https://www.sitepoint.com/demystifying-javascript-closures-callbacks-iifes/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- Chrome DevTools – Memory Leak Debugging — https://developer.chrome.com/docs/devtools/memory-problems/
- Jake Archibald – Garbage collection and closures — https://jakearchibald.com/2024/garbage-collection-and-closures/
- Google JavaScript Style Guide – Closures — https://google.github.io/styleguide/jsguide.html