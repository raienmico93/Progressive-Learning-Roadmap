# Function Patterns — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Function patterns are reusable, idiomatic solutions to common problems in JavaScript programming that leverage the language's treatment of functions as first-class values. They encompass scope isolation, encapsulation, object creation, behaviour enhancement, and execution control.

**Technical Definition:** Per the ECMAScript specification, functions are callable objects with a `[[Call]]` internal method and optionally a `[[Construct]]` method. Function patterns exploit this design through closures, the module system, prototype chains, and higher-order functions. They are not language features but architectural conventions built on the language's core semantics.

**Beginner-Friendly Explanation:** Function patterns are the "recipes" experienced developers use to solve common problems. Need to keep variables private? Use the module pattern. Need to create objects without `new`? Use a factory function. Need to prevent a button from being clicked twice? Use `once`. These patterns are tried-and-tested ways to write cleaner, safer JavaScript.

### Key Characteristics

- **Closure-driven:** Most patterns rely on closures to encapsulate state.
- **First-class functions:** Functions are passed as arguments and returned as values.
- **Scope isolation:** Patterns like IIFE and module pattern prevent global namespace pollution.
- **Behaviour enhancement:** Decorators, throttling, and debouncing wrap functions to add functionality.
- **Execution control:** `once`, `after`, and `before` control when and how often functions execute.
- **Object creation:** Factory functions and constructor functions provide different approaches to creating objects.

### Prerequisites

- Understanding of functions, scope, and closures.
- Familiarity with `this` binding and prototypes.
- Basic knowledge of asynchronous programming and event handling.

### Related Programming Areas

- **Functional programming:** Higher-order functions, composition, currying.
- **Object-oriented programming:** Constructors, prototypes, mixins.
- **Event-driven programming:** Debouncing, throttling, `once`.
- **Performance optimisation:** Memoization, lazy initialisation.

### Core Concepts / Features

---

## IIFE (Immediately Invoked Function Expression)

### Definitions

**Core Definition:** An IIFE is a JavaScript function that runs as soon as it is defined. It is also known as a self-executing anonymous function.

**Technical Definition:** Per MDN, "An IIFE (Immediately Invoked Function Expression) is an idiom in which a JavaScript function runs as soon as it is defined". It consists of two parts: a function expression enclosed in parentheses (the grouping operator), and an immediately invoked function call. The parentheses around the function are necessary because function declarations cannot be immediately invoked — only function expressions can. The IIFE pattern was named by Ben Alman.

**Beginner-Friendly Explanation:** An IIFE is a function that runs the moment you write it. You wrap a function in parentheses to turn it into an expression, then immediately call it with another set of parentheses. It's useful for creating a private scope — variables inside the IIFE don't leak into the global scope.

### Purposes

- To create a new scope and avoid polluting the global namespace.
- To create a new asynchronous context for using `await` in non-async contexts.
- To compute values using multiple statements as a single expression.
- To hide implementation details and expose only a public API (foundation of the module pattern).

### Syntax Rules and Structure

```js
// Standard IIFE:
(function() {
  // code runs immediately
})();

// Arrow function IIFE:
(() => {
  // code runs immediately
})();

// Async IIFE:
(async () => {
  await someAsyncOperation();
})();

// With arguments:
(function(name) {
  console.log(name);
})("Alice");
```

**Component Breakdown:**
- `(function() { ... })`: The function expression wrapped in grouping parentheses.
- `()`: The invocation parentheses that call the function immediately.
- Arguments can be passed inside the invocation parentheses.

**Syntax Rules:**
- The function must be an expression, not a declaration. Wrapping in parentheses is the most common way to achieve this.
- The grouping parentheses are required; without them, the `function` keyword would be parsed as a declaration.
- IIFEs can be async (`async () => {}`) or generators (`function* () {}`).

**Constraints and Limitations:**
- Modern JavaScript modules and block scoping (`let`/`const`) have reduced the need for IIFEs for scope isolation.
- IIFEs create a new `this` context (or capture the outer one if using arrow functions).

### Annotated Code Examples

#### Example 1: Basic IIFE for Scope Isolation

```js
// Step 1: Declare a global variable.
var globalVar = "I am global";

// Step 2: Create an IIFE to isolate private variables.
(function() {
  // Step 3: This variable is private to the IIFE.
  var privateVar = "I am private";
  console.log(privateVar); // "I am private"
  console.log(globalVar);  // "I am global" (accessible)
})();

// Step 4: The private variable is not accessible outside.
try {
  console.log(privateVar);
} catch (err) {
  console.log(err.name); // "ReferenceError"
}
```

**Expected Output:**
```
I am private
I am global
ReferenceError
```

**Why This Output:** The IIFE creates a new function scope. `privateVar` is declared inside this scope and is not accessible from the global scope. `globalVar` is accessible because the IIFE's scope chain includes the global scope.

#### Example 2: Async IIFE

```js
// Step 1: Use an async IIFE to await inside a non-async context.
(async () => {
  // Step 2: Simulate an async operation.
  const result = await Promise.resolve("Data loaded");
  console.log(result);
})();

// Step 3: The IIFE runs immediately.
console.log("After IIFE call");
```

**Expected Output:**
```
After IIFE call
Data loaded
```

**Why This Output:** The async IIFE starts executing immediately. The `await` pauses the IIFE's execution (not the entire script), so "After IIFE call" is logged first. When the promise resolves, "Data loaded" is logged.

### Real-World Cases

- **Legacy libraries:** jQuery and other pre-ES6 libraries use IIFEs to avoid global pollution.
- **Configuration scripts:** IIFEs run setup code once without leaving traces in the global scope.
- **Async initialisation:** Async IIFEs allow `await` in scripts that are not modules.

---

## Module Pattern

### Definitions

**Core Definition:** The module pattern is a design pattern that uses closures and IIFEs to create private and public members, providing encapsulation and namespace management in JavaScript.

**Technical Definition:** Per CoreUI, "The module pattern uses closures to create private and public members, providing encapsulation and namespace management in JavaScript. The most maintainable approach uses IIFE (Immediately Invoked Function Expression) or ES6 modules for encapsulation". The module pattern requires an outer enclosing function that is invoked at least once. At its core, the module pattern leverages scope, closures, and (commonly) IIFEs.

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
- `(function() { ... })()`: The IIFE that creates the module scope.
- `privateVar` / `privateMethod`: Private members inaccessible from outside.
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

#### Example 2: Module with Configuration

```js
// Step 1: Create a configurable logger module.
const Logger = (function(config) {
  // Step 2: Private settings merged with defaults.
  const settings = Object.assign(
    { level: "info", prefix: "[LOG]", timestamp: true },
    config
  );

  // Step 3: Private helper functions.
  function formatMessage(level, message) {
    const timestamp = settings.timestamp
      ? `[${new Date().toISOString()}] `
      : "";
    return `${timestamp}${settings.prefix} ${level.toUpperCase()}: ${message}`;
  }

  function shouldLog(level) {
    const levels = ["debug", "info", "warn", "error"];
    const currentLevel = levels.indexOf(settings.level);
    const messageLevel = levels.indexOf(level);
    return messageLevel >= currentLevel;
  }

  // Step 4: Public API.
  return {
    debug(message) {
      if (shouldLog("debug")) console.log(formatMessage("debug", message));
    },
    info(message) {
      if (shouldLog("info")) console.log(formatMessage("info", message));
    },
    warn(message) {
      if (shouldLog("warn")) console.warn(formatMessage("warn", message));
    },
    error(message) {
      if (shouldLog("error")) console.error(formatMessage("error", message));
    },
    setLevel(level) {
      settings.level = level;
    },
  };
})({ level: "debug", prefix: "[APP]" });

// Step 5: Use the logger.
Logger.debug("Debug message"); // Logs with [APP] prefix
Logger.info("Info message");
Logger.setLevel("error");
Logger.info("This won't show"); // Below error level
```

**Expected Output:**
```
[2026-10-13T...] [APP] DEBUG: Debug message
[2026-10-13T...] [APP] INFO: Info message
```

**Why This Output:** The module accepts a configuration object at creation time. The private `settings` object is merged with defaults. Public methods close over `settings` and `formatMessage`, providing a clean API while keeping implementation details private.

### Real-World Cases

- **jQuery plugins:** Encapsulated plugin code with private helpers.
- **Utility libraries:** Lodash and Underscore use module-like patterns.
- **Legacy codebases:** The module pattern was the standard before ES6 modules.

---

## Revealing Module Pattern

### Definitions

**Core Definition:** The revealing module pattern is a variation of the module pattern where all methods and variables are kept private until they are explicitly exposed through a returned object literal.

**Technical Definition:** The revealing module pattern "was engineered as a way to ensure that all methods and variables are kept private until they are explicitly exposed". The returned object contains references to the functions defined inside the IIFE. The only difference from the standard module pattern is that the revealing module pattern defines all functions privately and then reveals only the ones intended for public use.

**Beginner-Friendly Explanation:** The revealing module pattern is like the module pattern, but with a clearer separation between private implementation and public API. You define everything privately, then at the end, you return an object that "reveals" only the functions you want to make public.

### Purposes

- To make the public API explicit and self-documenting.
- To maintain encapsulation while clearly identifying what is public.
- To improve code readability by separating implementation from the public interface.
- To make it easy to change what is public without modifying the function definitions.

### Syntax Rules and Structure

```js
const ModuleName = (function() {
  // All functions and variables are private by default.
  function privateFunction() { /* ... */ }
  function publicFunction() { /* ... */ }

  // Reveal only the public API.
  return {
    publicMethod: publicFunction,
  };
})();
```

**Component Breakdown:**
- Private functions and variables are defined normally inside the IIFE.
- The returned object references the public functions by name.
- Only the referenced functions are accessible from outside.

**Syntax Rules:**
- All functions are defined as function declarations (not object methods).
- The return statement maps public names to private function references.
- Private functions can call each other directly.

**Constraints and Limitations:**
- The revealing module pattern has the same limitations as the standard module pattern.
- Renaming public methods requires changes in the return statement, not the function definitions.

### Annotated Code Examples

#### Example 1: Basic Revealing Module Pattern

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

  function addUser(name, email) {
    const user = { id: generateId(), name, email, createdAt: new Date() };
    users.push(user);
    return user;
  }

  function getUser(id) {
    return findById(id);
  }

  function getAllUsers() {
    return [...users]; // Return a copy
  }

  function deleteUser(id) {
    const index = users.findIndex((user) => user.id === id);
    if (index !== -1) {
      users.splice(index, 1);
      return true;
    }
    return false;
  }

  // Step 4: Reveal public API.
  return {
    add: addUser,
    get: getUser,
    getAll: getAllUsers,
    delete: deleteUser,
  };
})();

// Step 5: Use the module.
const user = UserManager.add("John", "john@example.com");
console.log(user);              // { id: 1, name: 'John', ... }
console.log(UserManager.getAll()); // [{ id: 1, ... }]
console.log(UserManager.generateId); // undefined (private)
```

**Expected Output:**
```
{ id: 1, name: 'John', email: 'john@example.com', createdAt: ... }
[ { id: 1, name: 'John', email: 'john@example.com', createdAt: ... } ]
undefined
```

**Why This Output:** All functions are defined privately. The return statement reveals only `add`, `get`, `getAll`, and `delete`. `generateId` remains private because it is not included in the returned object.

### Real-World Cases

- **jQuery plugins:** Revealing module pattern for plugin APIs.
- **Framework modules:** AngularJS services used the revealing module pattern.
- **Namespace management:** Organising large codebases before ES6 modules.

---

## Factory Functions

### Definitions

**Core Definition:** A factory function is a function that creates and returns a new object without using the `new` keyword.

**Technical Definition:** Per the curriculum source, "The factory function pattern is similar to constructors, but instead of using `new` to create an object, factory functions simply set up and return the new object when you call the function". Factory functions are used to produce objects. Unlike constructors, they do not require `new`, which eliminates the risk of forgetting it.

**Beginner-Friendly Explanation:** A factory function is a function that makes objects for you. You call it like a normal function, and it gives you back a ready-to-use object. No `new` keyword needed. This makes it safer and often simpler than constructor functions.

### Purposes

- To create objects without the complexity and risks of `new`.
- To encapsulate object creation logic and return customised objects.
- To avoid the `this` binding confusion associated with constructors.
- To support composition and flexible object creation.

### Syntax Rules and Structure

```js
function createPerson(name, age) {
  return {
    name,
    age,
    greet() {
      return `Hello, I'm ${this.name}`;
    },
  };
}

const person = createPerson("Alice", 30);
```

**Component Breakdown:**
- `createPerson`: The factory function.
- `return { ... }`: The object literal returned by the factory.
- Properties and methods are defined in the returned object.

**Syntax Rules:**
- Factory functions can be called without `new`.
- They return an object literal (or any object).
- `this` inside a factory function is not used for the returned object (unless explicitly bound).

**Constraints and Limitations:**
- Factory functions cannot use `instanceof` for type checking (unlike constructors).
- Each factory call creates a new object with its own methods (unless methods are shared via prototypes or mixins).

### Annotated Code Examples

#### Example 1: Basic Factory Function

```js
// Step 1: Define a factory function.
function createUser(name, email) {
  // Step 2: Return a new object.
  return {
    name,
    email,
    greet() {
      return `Hello, I'm ${this.name}`;
    },
    getInfo() {
      return `${this.name} (${this.email})`;
    },
  };
}

// Step 3: Create objects using the factory.
const alice = createUser("Alice", "alice@example.com");
const bob = createUser("Bob", "bob@example.com");

// Step 4: Use the objects.
console.log(alice.greet());   // "Hello, I'm Alice"
console.log(bob.getInfo());   // "Bob (bob@example.com)"
```

**Expected Output:**
```
Hello, I'm Alice
Bob (bob@example.com)
```

**Why This Output:** The factory function creates a new object with the provided `name` and `email` values. Each object has its own `greet` and `getInfo` methods that close over the object's properties.

#### Example 2: Factory with Closure-Based Privacy

```js
// Step 1: Factory function with private state.
function createCounter() {
  // Step 2: Private variable.
  let count = 0;

  // Step 3: Return object with methods that close over count.
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count,
  };
}

// Step 4: Create independent counters.
const counter1 = createCounter();
const counter2 = createCounter();

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

**Why This Output:** Each call to `createCounter()` creates a new closure with its own `count` variable. The returned methods are closures over that specific `count`. `counter1` and `counter2` are completely independent.

### Real-World Cases

- **React component factories:** Functions that return configured components.
- **API client factories:** `createClient(baseUrl, apiKey)`.
- **Game development:** `createEnemy(type, position)`.

---

## Constructor Functions

### Definitions

**Core Definition:** A constructor function is a regular function designed to be called with the `new` keyword, which creates a new object, binds `this` to that object, and returns it.

**Technical Definition:** Per MDN, "Constructor functions are ordinary functions that have a special purpose: to create object instances. You can consider them the more advanced version of factory functions. The difference with a factory function is the way to instantiate it". When called with `new`, JavaScript creates a new object, sets `this` to that object, runs the constructor body, and returns the object (unless the constructor explicitly returns an object).

**Beginner-Friendly Explanation:** A constructor function is a function you call with the `new` keyword to create objects. Inside the constructor, `this` refers to the new object being created. You set properties on `this`, and when the function finishes, the new object is returned automatically.

### Purposes

- To create object instances with shared methods via the prototype.
- To support `instanceof` type checking.
- To leverage prototype-based inheritance.
- To provide a standardised object creation pattern.

### Syntax Rules and Structure

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

const alice = new Person("Alice", 30);
```

**Component Breakdown:**
- `function Person(...)`: The constructor function (capitalised by convention).
- `this.name = name`: Sets properties on the new object.
- `new Person(...)`: Creates a new instance.

**Syntax Rules:**
- Constructor functions are called with `new`.
- By convention, constructor names start with a capital letter.
- `this` inside the constructor refers to the new object.
- If the constructor returns an object, that object replaces `this`.

**Constraints and Limitations:**
- Forgetting `new` causes `this` to follow default binding (global pollution).
- Methods defined inside the constructor are recreated for each instance (memory overhead).
- Methods should be defined on the prototype for sharing.

### Annotated Code Examples

#### Example 1: Basic Constructor Function

```js
// Step 1: Define a constructor function.
function Car(make, model) {
  this.make = make;
  this.model = model;
}

// Step 2: Add a method to the prototype (shared across instances).
Car.prototype.getInfo = function() {
  return `${this.make} ${this.model}`;
};

// Step 3: Create instances with new.
const car1 = new Car("Toyota", "Camry");
const car2 = new Car("Honda", "Civic");

// Step 4: Use the instances.
console.log(car1.getInfo()); // "Toyota Camry"
console.log(car2.getInfo()); // "Honda Civic"
console.log(car1 instanceof Car); // true
```

**Expected Output:**
```
Toyota Camry
Honda Civic
true
```

**Why This Output:** `new Car(...)` creates a new object, binds `this` to it, and runs the constructor. The `getInfo` method is on `Car.prototype`, so all instances share the same function. `instanceof` checks the prototype chain.

#### Example 2: Constructor vs Factory

```js
// Constructor function:
function PersonConstructor(name) {
  this.name = name;
}

// Factory function:
function personFactory(name) {
  return { name };
}

const p1 = new PersonConstructor("Alice");
const p2 = personFactory("Bob");

console.log(p1 instanceof PersonConstructor); // true
console.log(p2 instanceof personFactory);     // false
```

**Expected Output:**
```
true
false
```

**Why This Output:** `new PersonConstructor` creates an object whose prototype is `PersonConstructor.prototype`, so `instanceof` returns `true`. The factory function returns a plain object literal, which is not an instance of the factory function.

### Real-World Cases

- **ES5-style classes:** Before ES6 `class` syntax, constructors were the standard.
- **Prototype-based inheritance:** Setting up inheritance chains with `Parent.call(this, ...)`.
- **Built-in constructors:** `new Date()`, `new Map()`, `new Set()`.

---

## Mixins

### Definitions

**Core Definition:** A mixin is a pattern for adding reusable functionality to objects or classes without using inheritance. It is a faux-multiple inheritance pattern for classes in JavaScript.

**Technical Definition:** Per patterns.dev, "A mixin is a reusable bundle of behavior that can be folded into a class or object so it gains capabilities without sitting inside an inheritance chain". Mixins are typically implemented using `Object.assign()` to copy methods onto a class prototype or object. A mixin alters its subject, whereas a function decorator returns a new function that wraps the old one.

**Beginner-Friendly Explanation:** A mixin is like a set of extra abilities you can add to a class. If you have a `Dog` class and you want it to be able to swim, you can create a `SwimMixin` and mix it in. The dog doesn't inherit from "swimming" — it just gains the ability. Mixins let you share behaviour across unrelated classes.

### Purposes

- To share behaviour across classes without inheritance.
- To simulate multiple inheritance in JavaScript.
- To compose objects from small, reusable behaviour bundles.
- To avoid deep, fragile inheritance hierarchies.

### Syntax Rules and Structure

```js
// Mixin object:
const SwimMixin = {
  swim() {
    return `${this.name} is swimming.`;
  },
};

// Apply mixin to a class:
class Animal {
  constructor(name) {
    this.name = name;
  }
}

Object.assign(Animal.prototype, SwimMixin);

const dog = new Animal("Rex");
console.log(dog.swim()); // "Rex is swimming."
```

**Component Breakdown:**
- `SwimMixin`: An object containing the reusable methods.
- `Object.assign(Animal.prototype, SwimMixin)`: Copies the mixin's methods onto the class prototype.
- Instances of `Animal` now have the `swim` method.

**Syntax Rules:**
- Mixins are plain objects with methods.
- `Object.assign()` is the most common way to apply mixins.
- Multiple mixins can be applied by passing multiple source objects.

**Constraints and Limitations:**
- Mixins can cause naming conflicts if two mixins define the same method.
- Mixins are not a language feature; they are a convention.
- `instanceof` does not work with mixins (the object is not an instance of the mixin).

### Annotated Code Examples

#### Example 1: Basic Mixin

```js
// Step 1: Define mixin objects.
const canEat = {
  eat() {
    console.log(`${this.name} is eating.`);
  },
};

const canSleep = {
  sleep() {
    console.log(`${this.name} is sleeping.`);
  },
};

// Step 2: Define a class.
class Person {
  constructor(name) {
    this.name = name;
  }
}

// Step 3: Apply mixins to the class prototype.
Object.assign(Person.prototype, canEat, canSleep);

// Step 4: Create an instance and use the mixed-in methods.
const alice = new Person("Alice");
alice.eat();   // "Alice is eating."
alice.sleep(); // "Alice is sleeping."
```

**Expected Output:**
```
Alice is eating.
Alice is sleeping.
```

**Why This Output:** `Object.assign` copies the `eat` and `sleep` methods from the mixin objects onto `Person.prototype`. All instances of `Person` now have these methods. The mixins provide behaviour without inheritance.

#### Example 2: Mixin with State

```js
// Step 1: Mixin with initialisation logic.
const withCounter = {
  initCounter() {
    this.count = 0;
  },
  increment() {
    this.count++;
    return this.count;
  },
};

// Step 2: Class using the mixin.
class Button {
  constructor(label) {
    this.label = label;
    this.initCounter();
  }
}

Object.assign(Button.prototype, withCounter);

// Step 3: Use the mixed-in state and methods.
const btn = new Button("Submit");
btn.increment();
btn.increment();
console.log(btn.count); // 2
```

**Expected Output:**
```
2
```

**Why This Output:** The mixin provides both a method (`increment`) and an initialisation method (`initCounter`). The `Button` constructor calls `initCounter` to set up the `count` property. The mixed-in `increment` method operates on the instance's `count`.

### Real-World Cases

- **React:** Higher-order components (HOCs) are a form of mixin.
- **Vue:** Mixins are a first-class feature in Vue options API.
- **Utility libraries:** Lodash's `_.mixin()` adds methods to the Lodash object.
- **Game development:** Adding abilities like `Flyable`, `Swimmable`, `Attackable` to characters.

---

## Decorators

### Definitions

**Core Definition:** A decorator is a function that wraps another function to enhance its behaviour without modifying the original function's code.

**Technical Definition:** Per Telerik, "Decorators are just a wrapper around a function. They are used to enhance the functionality of the function without modifying the underlying function. Just like higher-order functions, decorators also add some functionality to the existing function without modifying the underlying code". Decorators are a Stage 3 TC39 proposal for JavaScript, and they are widely used through TypeScript and Babel. The `@` syntax allows declarative application of decorators to classes, methods, properties, and parameters.

**Beginner-Friendly Explanation:** A decorator is like a sticker you put on a function that adds extra behaviour. For example, a `@logged` decorator could log every time a function is called. You don't change the function's code — the decorator wraps around it and adds the logging automatically.

### Purposes

- To add behaviour (logging, validation, caching) to functions without modifying their source.
- To support aspect-oriented programming (AOP) concerns.
- To improve code readability by making cross-cutting concerns explicit.
- To enable metadata annotation for frameworks.

### Syntax Rules and Structure

```js
// Decorator function (function wrapper):
function logged(fn) {
  return function(...args) {
    console.log(`Calling with:`, args);
    const result = fn.apply(this, args);
    console.log(`Returned:`, result);
    return result;
  };
}

// Usage with @ syntax (requires transpiler):
class Example {
  @logged
  greet(name) {
    return `Hello, ${name}`;
  }
}
```

**Component Breakdown:**
- `logged`: The decorator function that takes the original function and returns a wrapped version.
- `@logged`: The decorator syntax (requires TypeScript or Babel).
- The wrapper function calls the original function and adds behaviour before/after.

**Syntax Rules:**
- Decorators can be applied to classes, methods, accessors, properties, and parameters.
- Multiple decorators execute in reverse order (bottom-up).
- Native browser support is incomplete; transpilation is typically required.

**Constraints and Limitations:**
- Decorators are a Stage 3 proposal; they are not yet standardised.
- Decorator syntax requires transpiler configuration.
- Overuse of decorators can make code harder to trace and debug.

### Annotated Code Examples

#### Example 1: Function Decorator (Without @ Syntax)

```js
// Step 1: Define a decorator function.
function withTiming(fn) {
  return function(...args) {
    const start = performance.now();
    const result = fn.apply(this, args);
    const end = performance.now();
    console.log(`Execution time: ${(end - start).toFixed(2)}ms`);
    return result;
  };
}

// Step 2: Decorate a function.
function slowFunction(n) {
  let sum = 0;
  for (let i = 0; i < n; i++) sum += i;
  return sum;
}

const timedSlowFunction = withTiming(slowFunction);

// Step 3: Call the decorated function.
timedSlowFunction(1000000);
```

**Expected Output:**
```
Execution time: 2.34ms (approximate)
499999500000
```

**Why This Output:** The `withTiming` decorator wraps `slowFunction`. It measures the time before and after execution and logs it. The original function's result is returned unchanged.

#### Example 2: Class Method Decorator (With @ Syntax)

```js
// Step 1: Define a decorator.
function readonly(target, key, descriptor) {
  descriptor.writable = false;
  return descriptor;
}

// Step 2: Apply the decorator.
class Config {
  @readonly
  getVersion() {
    return "1.0.0";
  }
}

// Step 3: Attempt to modify (fails silently in non-strict mode).
const config = new Config();
console.log(config.getVersion()); // "1.0.0"
```

**Expected Output:**
```
1.0.0
```

**Why This Output:** The `@readonly` decorator modifies the property descriptor to make the method non-writable. Attempts to reassign `getVersion` will fail.

### Real-World Cases

- **Angular:** `@Component`, `@Injectable`, `@Input` decorators.
- **NestJS:** `@Controller`, `@Get`, `@Post` decorators.
- **TypeScript:** Decorators for classes, methods, properties, and parameters.
- **MobX:** `@observable`, `@computed` decorators.

---

## Throttling

### Definitions

**Core Definition:** Throttling is a technique that limits the number of times a function can be called within a given time frame. It ensures a function executes at a consistent rate during an ongoing action.

**Technical Definition:** Per the GUVI guide, "The purpose of throttling is to limit the number of times a function can be called within a given time frame. Unlike debouncing, throttling does not delay the execution of the function but restricts the number of times it can be called". Lodash defines throttle as creating "a throttled function that only invokes func at most once per every wait milliseconds".

**Beginner-Friendly Explanation:** Throttling is like putting a speed limit on a function. If you throttle a scroll handler to once every 200ms, it will run at most five times per second, no matter how many scroll events fire. This is useful for performance-heavy operations like infinite scroll or mouse tracking.

### Purposes

- To limit the execution frequency of high-frequency events (scroll, resize, mousemove).
- To improve performance by reducing the number of function calls.
- To ensure consistent execution during ongoing actions.
- To implement rate limiting for API calls.

### Syntax Rules and Structure

```js
function throttle(fn, limit) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= limit) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}
```

**Component Breakdown:**
- `fn`: The function to throttle.
- `limit`: The minimum time (in ms) between calls.
- `lastCall`: Timestamp of the last execution.
- The returned function checks if enough time has passed before executing.

**Syntax Rules:**
- Throttling is implemented using a closure to track the last call time.
- The throttled function is a higher-order function that wraps the original.
- Arguments and `this` are passed through using `apply`.

**Constraints and Limitations:**
- Throttling does not guarantee execution at exact intervals; it guarantees at most once per interval.
- The first call may be delayed (unless a leading edge option is implemented).

### Annotated Code Examples

#### Example 1: Basic Throttle

```js
// Step 1: Define a throttle utility.
function throttle(fn, limit) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= limit) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}

// Step 2: Create a throttled scroll handler.
const handleScroll = throttle(() => {
  console.log("Scroll handled at:", new Date().toISOString());
}, 1000);

// Step 3: Attach to scroll event.
window.addEventListener("scroll", handleScroll);
```

**Expected Output:** (During rapid scrolling, the handler logs at most once per second.)

**Why This Output:** The throttle function tracks the last execution time. If less than 1000ms has passed since the last call, the function is not executed. This limits the handler to at most one execution per second.

### Real-World Cases

- **Scroll events:** Infinite scroll, lazy loading images.
- **Mouse movement:** Tracking cursor position at limited intervals.
- **API rate limiting:** Limiting requests to a server.
- **Game loops:** Updating game state at a fixed rate.

---

## Debouncing

### Definitions

**Core Definition:** Debouncing is a technique that postpones the execution of a function until after a certain amount of time has passed since its last invocation. It ensures a function is only called once after a sequence of rapid calls.

**Technical Definition:** Per the GUVI guide, "Debouncing in JavaScript makes sure that a function isn't called immediately after the other. Rather, it postpones the function's execution until after a certain amount of time has passed since its previous invocation". Lodash defines debounce as creating "a debounced function that delays invoking func until after wait milliseconds have elapsed since the last time the debounced function was invoked".

**Beginner-Friendly Explanation:** Debouncing is like waiting for someone to finish typing before you act. If you're typing in a search box, you don't want to search on every keystroke. Debouncing waits until you stop typing for, say, 500ms, and then runs the search. If you type another character, the timer resets.

### Purposes

- To delay function execution until after a burst of calls has stopped.
- To reduce the number of API calls triggered by user input.
- To prevent double-clicks on buttons.
- To optimise window resize and scroll handlers.

### Syntax Rules and Structure

```js
function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}
```

**Component Breakdown:**
- `fn`: The function to debounce.
- `delay`: The idle time (in ms) required before execution.
- `timeoutId`: The timer ID, cleared on each new call.
- The returned function resets the timer on every call.

**Syntax Rules:**
- Debouncing is implemented using `setTimeout` and `clearTimeout`.
- The timer is cleared and reset on each call.
- The function executes only after the delay has passed without any new calls.

**Constraints and Limitations:**
- Debouncing delays execution; it does not guarantee execution at regular intervals.
- If calls never stop, the function never executes (unless a maximum wait time is implemented).

### Annotated Code Examples

#### Example 1: Basic Debounce

```js
// Step 1: Define a debounce utility.
function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
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

**Why This Output:** Each call to `search` clears the previous timer and sets a new one. Only the last call — after the user stops typing for 500ms — actually triggers the search. This prevents excessive API calls while the user is typing.

### Real-World Cases

- **Search autocomplete:** Debouncing input to avoid firing a request on every keystroke.
- **Window resize:** Running resize logic after the user stops resizing.
- **Form validation:** Validating input after the user stops typing.
- **Button clicks:** Preventing double-clicks.

---

## Once

### Definitions

**Core Definition:** The `once` pattern creates a function that can only be executed one time, no matter how many times it is called. Subsequent calls return the result of the first invocation.

**Technical Definition:** Per Underscore's source code, `_.once` "Returns a function that will be executed at most one time, no matter how often you call it. Useful for lazy initialization". It is implemented as `_.partial(_.before, 2)`, meaning it uses the `before` pattern with a count of 2 (the function can be called once before the internal flag is set).

**Beginner-Friendly Explanation:** The `once` pattern is like a one-time-use coupon. After you use it once, it stops working. In code, `once(fn)` returns a function that runs `fn` only the first time it's called. Every subsequent call just returns the cached result of the first call.

### Purposes

- To ensure a function runs only once (lazy initialisation).
- To prevent duplicate side effects from multiple calls.
- To cache the result of an expensive one-time operation.

### Syntax Rules and Structure

```js
function once(fn) {
  let called = false;
  let result;
  return function(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}
```

**Component Breakdown:**
- `called`: A boolean flag tracking whether the function has been invoked.
- `result`: The cached result of the first invocation.
- The returned function checks `called` before executing `fn`.

**Syntax Rules:**
- The first call executes `fn` and stores the result.
- Subsequent calls return the stored result without executing `fn`.
- Arguments passed on subsequent calls are ignored.

**Constraints and Limitations:**
- `once` does not reset; the function can never be called again.
- The cached result is shared across all calls.

### Annotated Code Examples

#### Example 1: Basic Once

```js
// Step 1: Define a once utility.
function once(fn) {
  let called = false;
  let result;
  return function(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}

// Step 2: Create a once-only initialiser.
const initialise = once(() => {
  console.log("Initialising...");
  return { status: "ready" };
});

// Step 3: Call multiple times.
console.log(initialise()); // Logs "Initialising...", returns { status: "ready" }
console.log(initialise()); // Returns cached result, no log
console.log(initialise()); // Returns cached result, no log
```

**Expected Output:**
```
Initialising...
{ status: 'ready' }
{ status: 'ready' }
{ status: 'ready' }
```

**Why This Output:** The first call to `initialise` executes the function and logs "Initialising...". Subsequent calls return the cached `{ status: "ready" }` without executing the function again.

### Real-World Cases

- **Configuration loading:** Loading configuration only once.
- **Event listener setup:** Attaching a listener only once.
- **Database connection:** Creating a connection pool once.
- **Payment processing:** Ensuring a payment is processed only once.

---

## After

### Definitions

**Core Definition:** The `after` pattern creates a function that executes only after it has been called a specified number of times. Before that threshold, it does nothing.

**Technical Definition:** Per Underscore's source code, `_.after` "Returns a function that will only be executed on and after the Nth call". The implementation uses a counter that decrements on each call; when the counter reaches zero, the original function is invoked.

**Beginner-Friendly Explanation:** The `after` pattern is like a countdown timer. You specify a number, and the function only runs after it has been called that many times. For example, `after(3, fn)` returns a function that does nothing for the first two calls, then runs `fn` on the third call and every call after that.

### Purposes

- To wait for a specific number of calls before executing.
- To coordinate multiple asynchronous operations (e.g., wait for all to complete).
- To implement lazy initialisation after N calls.

### Syntax Rules and Structure

```js
function after(times, fn) {
  return function(...args) {
    if (--times < 1) {
      return fn.apply(this, args);
    }
  };
}
```

**Component Breakdown:**
- `times`: The number of calls required before execution.
- `--times`: Decrements the counter on each call.
- When `times` reaches 0, `fn` is executed.

**Syntax Rules:**
- The function does nothing until the Nth call.
- After the Nth call, the function executes on every subsequent call.
- The counter is decremented on each call.

**Constraints and Limitations:**
- The counter is shared across all calls; there is no way to reset it.
- Arguments passed before the Nth call are discarded.

### Annotated Code Examples

#### Example 1: Basic After

```js
// Step 1: Define an after utility.
function after(times, fn) {
  return function(...args) {
    if (--times < 1) {
      return fn.apply(this, args);
    }
  };
}

// Step 2: Create a function that runs after 3 calls.
const runAfterThree = after(3, () => {
  console.log("Executed after 3 calls!");
});

// Step 3: Call multiple times.
runAfterThree(); // Nothing
runAfterThree(); // Nothing
runAfterThree(); // "Executed after 3 calls!"
runAfterThree(); // "Executed after 3 calls!" (runs again)
```

**Expected Output:**
```
Executed after 3 calls!
Executed after 3 calls!
```

**Why This Output:** The function does nothing on the first two calls. On the third call, `times` becomes 0 (after decrementing from 2 to 1 to 0), and `fn` executes. On the fourth call, `times` is already below 1, so `fn` executes again.

### Real-World Cases

- **Coordinating async operations:** Wait for N callbacks before proceeding.
- **Batch processing:** Execute a cleanup after N items are processed.
- **Game development:** Trigger an event after N actions.

---

## Before

### Definitions

**Core Definition:** The `before` pattern creates a function that executes up to (but not including) the Nth call. After the Nth call, it returns the result of the last successful execution.

**Technical Definition:** Per Underscore's source code, `_.before` "Returns a function that will only be executed up to (but not including) the Nth call". The implementation stores the result of each call and stops executing after the counter reaches zero.

**Beginner-Friendly Explanation:** The `before` pattern is the opposite of `after`. It runs the function up to N-1 times, then stops executing it. After that, it just returns the last result. For example, `before(3, fn)` runs `fn` on the first two calls, then stops on the third and subsequent calls.

### Purposes

- To limit a function to a maximum number of executions.
- To cache the last result after the limit is reached.
- To implement one-time or few-time operations.

### Syntax Rules and Structure

```js
function before(times, fn) {
  let memo;
  return function(...args) {
    if (--times > 0) {
      memo = fn.apply(this, args);
    }
    if (times <= 1) fn = null;
    return memo;
  };
}
```

**Component Breakdown:**
- `times`: The number of calls allowed (the function runs on calls 1 through N-1).
- `memo`: Caches the result of the last execution.
- After the limit is reached, `fn` is set to `null` to release the reference.

**Syntax Rules:**
- The function executes on calls 1 through N-1.
- On the Nth call and beyond, it returns the cached result.
- `fn` is nullified after the limit to free memory.

**Constraints and Limitations:**
- The cached result is from the last execution before the limit.
- Arguments passed after the limit are ignored.

### Annotated Code Examples

#### Example 1: Basic Before

```js
// Step 1: Define a before utility.
function before(times, fn) {
  let memo;
  return function(...args) {
    if (--times > 0) {
      memo = fn.apply(this, args);
    }
    if (times <= 1) fn = null;
    return memo;
  };
}

// Step 2: Create a function that runs up to 3 times.
const runBeforeThree = before(3, (x) => {
  console.log(`Executing with ${x}`);
  return x * 2;
});

// Step 3: Call multiple times.
console.log(runBeforeThree(1)); // Executes: 2
console.log(runBeforeThree(2)); // Executes: 4
console.log(runBeforeThree(3)); // Does not execute: 4 (cached)
console.log(runBeforeThree(4)); // Does not execute: 4 (cached)
```

**Expected Output:**
```
Executing with 1
2
Executing with 2
4
4
4
```

**Why This Output:** The function executes on the first two calls (times decrements from 3 to 2 to 1). On the third call, `times` becomes 0, so `fn` is not executed; the cached result from the second call (4) is returned. On subsequent calls, `fn` is `null`, so only the cached result is returned.

### Real-World Cases

- **Limited retry logic:** Attempt an operation at most N times.
- **Rate limiting:** Allow a function to run a maximum number of times.
- **Caching:** Cache the result of a function after a limited number of calls.

---

## References

- MDN Web Docs – IIFE (Immediately Invoked Function Expression) — https://developer.mozilla.org/en-US/docs/Glossary/IIFE
- MDN Web Docs – Function expressions — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/function
- CoreUI – How to implement module pattern in JavaScript — https://coreui.io/answers/how-to-implement-module-pattern-in-javascript/
- DigitalOcean – Module Design Pattern in JavaScript — https://www.digitalocean.com/community/tutorials/js-module-design-pattern
- GUVI – Debouncing and Throttling in JavaScript: A Beginner's Guide — https://www.guvi.in/blog/debouncing-and-throttling-in-javascript/
- Underscore.js Source Code – once, after, before — https://raw.githubusercontent.com/jashkenas/underscore/1.9.1/underscore.js
- BMCARDONA Curriculum – Factory Functions — https://raw.githubusercontent.com/BMCARDONA/curriculum/49c1fcdeb5eaa60af2989e3444321135b5b1a20b/organizing-js/factory-functions.md
- patterns.dev – Mixin Pattern — https://www.patterns.dev/vanilla/mixin-pattern
- TypeScript Playground – Mixins — https://www.typescriptlang.org/play
- Telerik – Decorators in JavaScript — https://www.telerik.com/blogs/decorators-in-javascript
- Lodash Documentation – debounce — https://lodash.com/docs#debounce
- Lodash Documentation – throttle — https://lodash.com/docs#throttle
- TC39 – Decorators Proposal — https://github.com/tc39/proposal-decorators
- TypeScript Handbook – Decorators — https://www.typescriptlang.org/docs/handbook/decorators.html
- Google JavaScript Style Guide – Functions — https://google.github.io/styleguide/jsguide.html
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf