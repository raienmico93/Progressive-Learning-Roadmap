# JavaScript Object Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Object patterns are reusable design solutions that use JavaScript's object model (objects, prototypes, closures, and the module system) to organise code, encapsulate state, control instantiation, and manage resources.

**Technical Definition:** Object patterns in JavaScript emerge from the language's unique combination of features: first-class functions, closures, prototype-based inheritance, and (in ES2015+) the `class` syntax. Unlike classical object-oriented languages where patterns are often built on class hierarchies, JavaScript patterns frequently leverage closures for encapsulation and prototypes for shared behaviour. The patterns covered here — Namespace, Module, Singleton, Factory, Prototype, Builder, and Object Pooling — are implementations of classic design patterns adapted to JavaScript's idioms, as popularised by the "Gang of Four" (GoF) and refined for JavaScript by authors such as Addy Osmani, Stoyan Stefanov, and the MDN documentation.

**Beginner-Friendly Explanation:** Object patterns are proven "recipes" for organising your code with objects. They solve common problems: how to avoid polluting the global scope (Namespace), how to keep data private (Module), how to ensure only one instance exists (Singleton), how to create objects flexibly (Factory), how to share behaviour efficiently (Prototype), how to construct complex objects step by step (Builder), and how to reuse expensive objects (Object Pooling). Learning these patterns helps you write cleaner, more maintainable code.

### Key Characteristics

- **Idiomatic to JavaScript:** Patterns are adapted to JavaScript's features, not copied from Java.
- **Closure-based encapsulation:** Many patterns use closures to achieve privacy.
- **Prototype-based inheritance:** Patterns leverage prototypes for shared behaviour.
- **Composable:** Patterns can be combined (e.g., Factory + Singleton).
- **Trade-offs:** Every pattern has costs (complexity, memory, indirection).
- **Class-syntax compatible:** Modern patterns can be written with `class` or with functions.

### Prerequisites

- Solid understanding of objects, prototypes, and closures.
- Familiarity with `this`, `new`, and constructor functions.
- Understanding of scope and the module system.
- Basic knowledge of IIFEs (Immediately Invoked Function Expressions).
- Awareness of ES6 `class`, `import`/`export`.

### Related Programming Areas

- **Architecture:** Structuring large applications.
- **Encapsulation:** Hiding implementation details.
- **Instantiation control:** Managing how and when objects are created.
- **Resource management:** Reusing expensive objects.
- **Module systems:** ES modules, CommonJS, AMD.
- **Functional programming:** Factory functions and composition.

### Core Concepts / Features

The following sections cover each object pattern, each using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## Namespace Pattern

### Definitions

**Core Definition:** The Namespace pattern groups related functionality under a single global object to avoid polluting the global scope and prevent naming collisions.

**Technical Definition:** In JavaScript, the Namespace pattern is implemented by creating a single global object (the namespace) and attaching properties, functions, and sub-namespaces to it. Before ES modules, this was the primary way to organise code in large applications. The namespace object acts as a container, and its properties are accessed via dot notation.

**Beginner-Friendly Explanation:** The Namespace pattern is like using labelled folders instead of dumping everything on your desk. Instead of having `getUser`, `saveUser`, and `deleteUser` as separate global functions, you put them inside a `User` object: `User.get`, `User.save`, `User.delete`. This keeps the global scope clean and makes it clear which functions belong together.

### Purposes

- To avoid polluting the global scope with many variables.
- To prevent naming collisions between different parts of an application.
- To organise related functionality into logical groups.
- To provide a single access point for a library or module.
- To enable nested namespaces for hierarchical organisation.

### Syntax Rules and Structure

**Basic namespace:**
```js
const MyApp = {
  models: {},
  views: {},
  controllers: {},
  utils: {
    formatDate(date) { /* ... */ },
    parseDate(str) { /* ... */ },
  },
};
```

**Nested namespace:**
```js
const MyApp = MyApp || {};
MyApp.utils = MyApp.utils || {};
MyApp.utils.formatDate = function(date) { /* ... */ };
```

**Component Breakdown:**
- The namespace object (e.g., `MyApp`).
- Sub-objects for categories (e.g., `models`, `views`).
- Functions attached as properties.

**Syntax Rules:**
- Use a single global variable for the top-level namespace.
- Nested namespaces are created as properties.
- Use `MyApp = MyApp || {}` to avoid overwriting an existing namespace.
- Access members via dot notation: `MyApp.utils.formatDate()`.

**Constraints and Limitations:**
- Still pollutes the global scope with one variable.
- Long dot-notation chains can become verbose.
- ES modules are now preferred for new code.
- No built-in access control.

### Annotated Code Examples

#### Example 1: Basic Namespace

```js
// Step 1: Create a namespace object.
const MyApp = {
  // Sub-namespace for string utilities.
  stringUtils: {
    capitalize(str) {
      return str.charAt(0).toUpperCase() + str.slice(1);
    },
    reverse(str) {
      return str.split("").reverse().join("");
    },
  },

  // Sub-namespace for math utilities.
  mathUtils: {
    sum(...nums) {
      return nums.reduce((a, b) => a + b, 0);
    },
    average(...nums) {
      return nums.length ? this.sum(...nums) / nums.length : 0;
    },
  },
};

// Step 2: Use the namespace.
console.log(MyApp.stringUtils.capitalize("hello")); // "Hello"
console.log(MyApp.stringUtils.reverse("abc"));       // "cba"
console.log(MyApp.mathUtils.sum(1, 2, 3));            // 6
console.log(MyApp.mathUtils.average(10, 20, 30));     // 20
```

**Expected Output:**
```
Hello
cba
6
20
```

**Why This Output:** Related functions are grouped into sub-namespaces, and accessed via dot notation.

#### Example 2: Safe Namespace Extension

```js
// Step 1: Define or reuse the namespace.
const MyApp = window.MyApp || {};

// Step 2: Add a sub-namespace.
MyApp.utils = MyApp.utils || {};
MyApp.utils.now = function() { return Date.now(); };

console.log(typeof MyApp.utils.now); // "function"
```

**Expected Output:**
```
function
```

**Why This Output:** The `|| {}` pattern prevents overwriting existing namespaces when multiple scripts contribute to the same namespace.

### Real-World Cases

- **Legacy libraries:** jQuery (`$`), Lodash (`_`), D3 (`d3`).
- **Enterprise apps:** `CompanyName.App.Module`.
- **Plugin systems:** `MyPlugin.utils`, `MyPlugin.widgets`.
- **Pre-module codebases:** Before ES modules, this was standard.

---

## Module Pattern

### Definitions

**Core Definition:** The Module pattern encapsulates private state and exposes a public API using closures and an IIFE (Immediately Invoked Function Expression).

**Technical Definition:** The Module pattern uses an IIFE to create a private scope. Variables declared inside the IIFE are not accessible from outside; only the returned object's properties are exposed. The pattern was popularised by Eric Miraglia and Addy Osmani. Modern JavaScript uses ES modules (`import`/`export`) for the same purpose, but the IIFE-based Module pattern remains common in legacy code and for creating singleton modules.

**Beginner-Friendly Explanation:** The Module pattern is like a house with a public front door and private rooms. The front door (returned object) is what visitors can access; the private rooms (closed-over variables) are hidden from the outside.

### Purposes

- To achieve true privacy for internal state.
- To expose only a controlled public API.
- To avoid polluting the global scope.
- To create singleton modules with encapsulated state.
- To organise related functionality in a self-contained unit.

### Syntax Rules and Structure

```js
const Module = (function() {
  // Private variables and functions
  let privateVar = 0;
  function privateFunction() { /* ... */ }

  // Public API
  return {
    publicMethod() { /* ... */ },
    publicProperty: "value",
  };
})();
```

**Component Breakdown:**
- `(function() { ... })()`: The IIFE that creates the private scope.
- Private variables/functions inside the IIFE.
- A returned object exposing the public API.
- The IIFE's return value is assigned to a variable.

**Syntax Rules:**
- Use an IIFE to create a new scope.
- Return an object literal with the public API.
- Private members are closed over by the public methods.
- Can be extended using the "augmentation" pattern.
- Named IIFEs help with debugging.

**Constraints and Limitations:**
- Singleton by nature; cannot create multiple instances.
- Memory: closures keep private variables alive.
- ES modules are now preferred for new code.
- Testing private members is difficult.

### Annotated Code Examples

#### Example 1: Basic Module Pattern

```js
// Step 1: Define the module with an IIFE.
const Counter = (function() {
  // Private state
  let count = 0;

  // Private helper
  function log(action) {
    console.log(`${action}: count is now ${count}`);
  }

  // Public API
  return {
    increment() {
      count++;
      log("increment");
      return count;
    },
    decrement() {
      count--;
      log("decrement");
      return count;
    },
    getCount() {
      return count;
    },
  };
})();

// Step 2: Use the module.
console.log(Counter.increment()); // "increment: count is now 1" then 1
console.log(Counter.increment()); // "increment: count is now 2" then 2
console.log(Counter.getCount());  // 2
console.log(Counter.count);       // undefined (private)
```

**Expected Output:**
```
increment: count is now 1
1
increment: count is now 2
2
2
undefined
```

**Why This Output:** The `count` variable and `log` function are private; only the returned methods can access them.

#### Example 2: Module Augmentation

```js
// Step 1: Start with a base module.
const MyModule = (function() {
  return {
    greet(name) { return `Hello, ${name}`; },
  };
})();

// Step 2: Augment the module.
(function(mod) {
  mod.farewell = function(name) {
    return `Goodbye, ${name}`;
  };
})(MyModule);

console.log(MyModule.greet("Alice"));    // "Hello, Alice"
console.log(MyModule.farewell("Alice")); // "Goodbye, Alice"
```

**Expected Output:**
```
Hello, Alice
Goodbye, Alice
```

**Why This Output:** Augmentation adds new methods to the existing module without modifying its original definition.

#### Example 3: Revealing Module Pattern

```js
const UserService = (function() {
  // Private state
  const users = [];

  // Private functions
  function validate(user) {
    return user.name && user.email;
  }

  // Public functions
  function addUser(user) {
    if (!validate(user)) throw new Error("Invalid user");
    users.push(user);
    return user;
  }

  function getUsers() {
    return [...users];
  }

  // Reveal public API
  return {
    add: addUser,
    list: getUsers,
  };
})();

UserService.add({ name: "Alice", email: "alice@example.com" });
console.log(UserService.list()); // [{ name: "Alice", email: "alice@example.com" }]
```

**Expected Output:**
```
[ { name: 'Alice', email: 'alice@example.com' } ]
```

**Why This Output:** The revealing module pattern defines all functions privately and reveals only the chosen ones.

### Real-World Cases

- **jQuery plugins:** `$.fn.myPlugin = (function() { ... })()`.
- **Node.js (pre-ES modules):** CommonJS modules use a similar pattern.
- **Singleton services:** Configuration, logging, caching.
- **Encapsulated state:** Counters, stores, registries.

---

## Singleton Pattern

### Definitions

**Core Definition:** The Singleton pattern ensures that a class or module has only one instance and provides a global point of access to it.

**Technical Definition:** In JavaScript, the Singleton pattern is typically implemented by caching the first instance created. In ES6+, `export` a single instance from a module (modules are cached after first evaluation). Alternatively, an IIFE-based Module pattern naturally produces a singleton. A class-based singleton can use a static instance property and a static `getInstance()` method.

**Beginner-Friendly Explanation:** A Singleton is a "one and only one" object — like the president of a country. No matter how many times you ask for it, you get the same one.

### Purposes

- To ensure a single shared instance of a resource.
- To provide a global access point for that instance.
- To control access to shared state (e.g., configuration, logging).
- To avoid the cost of creating multiple expensive objects.

### Syntax Rules and Structure

**ES module singleton:**
```js
// logger.js
const logger = {
  logs: [],
  log(msg) { this.logs.push(msg); console.log(msg); },
};
export default logger;
```

**Class-based singleton:**
```js
class Singleton {
  static #instance;
  constructor() {
    if (Singleton.#instance) return Singleton.#instance;
    Singleton.#instance = this;
  }
}
```

**IIFE singleton:**
```js
const singleton = (function() {
  let instance;
  function create() { return { /* ... */ }; }
  return {
    getInstance() {
      if (!instance) instance = create();
      return instance;
    },
  };
})();
```

**Constraints and Limitations:**
- Global state can make testing harder.
- Hides dependencies (harder to inject mocks).
- Overused in some codebases; consider dependency injection instead.
- Module singletons are the modern preferred approach.

### Annotated Code Examples

#### Example 1: ES Module Singleton

```js
// logger.js
const logger = {
  logs: [],
  log(message) {
    this.logs.push(message);
    console.log(`[LOG] ${message}`);
  },
  getLogs() {
    return [...this.logs];
  },
};

export default logger;

// app.js
import logger from "./logger.js";
logger.log("App started"); // "[LOG] App started"
logger.log("User logged in"); // "[LOG] User logged in"
console.log(logger.getLogs()); // ["App started", "User logged in"]
```

**Expected Output:**
```
[LOG] App started
[LOG] User logged in
[ 'App started', 'User logged in' ]
```

**Why This Output:** ES modules are cached after first evaluation, so all importers share the same `logger` instance.

#### Example 2: Class-Based Singleton

```js
class Config {
  static #instance = null;

  constructor() {
    if (Config.#instance) {
      return Config.#instance;
    }
    this.settings = {};
    Config.#instance = this;
  }

  set(key, value) {
    this.settings[key] = value;
  }

  get(key) {
    return this.settings[key];
  }
}

const a = new Config();
const b = new Config();
a.set("theme", "dark");
console.log(b.get("theme")); // "dark"
console.log(a === b);        // true
```

**Expected Output:**
```
dark
true
```

**Why This Output:** The constructor returns the cached instance, so `a` and `b` are the same object.

### Real-World Cases

- **Configuration:** A single config object shared across an app.
- **Logging:** A centralised logger.
- **Database connections:** A shared connection pool (though pools are different).
- **Redux store:** One store per application.

---

## Factory Pattern

### Definitions

**Core Definition:** The Factory pattern provides an interface for creating objects, allowing subclasses or calling code to decide which class to instantiate.

**Technical Definition:** In JavaScript, a factory function is a function that returns a new object without using `new`. Factories can return different object shapes based on input, hide construction complexity, and leverage closures for encapsulation. They are a fundamental alternative to constructor functions and classes.

**Beginner-Friendly Explanation:** A factory is like a vending machine: you put in a request ("I want a coffee"), and it gives you back the right product without you needing to know how it's made.

### Purposes

- To encapsulate and abstract object creation.
- To return different types based on input.
- To avoid the `new` keyword and its pitfalls.
- To enable composition over inheritance.
- To provide a clean API for object creation.

### Syntax Rules and Structure

```js
function createThing(type, options) {
  switch (type) {
    case "a": return { /* ... */ };
    case "b": return { /* ... */ };
    default: throw new Error("Unknown type");
  }
}
```

**Component Breakdown:**
- The factory function takes parameters describing what to create.
- It returns a new object (or an existing one, in some cases).
- The caller does not use `new`.

**Syntax Rules:**
- Factory functions conventionally start with `create` or `make`.
- They return objects directly.
- They can return different shapes.
- They can use closures for private state.

**Constraints and Limitations:**
- `instanceof` checks don't work as expected with factories (all return plain objects).
- Can obscure the object's type.
- May hide complexity that would be clearer with classes.

### Annotated Code Examples

#### Example 1: Basic Factory

```js
// Step 1: Define a factory function.
function createUser(role) {
  const base = {
    greet() { return `Hello, I'm ${this.name}`; },
  };

  switch (role) {
    case "admin":
      return Object.assign(Object.create(base), {
        role: "admin",
        name: "Admin",
        canDelete: () => true,
      });
    case "user":
      return Object.assign(Object.create(base), {
        role: "user",
        name: "User",
        canDelete: () => false,
      });
    default:
      throw new Error(`Unknown role: ${role}`);
  }
}

// Step 2: Use the factory.
const admin = createUser("admin");
const user = createUser("user");

console.log(admin.greet());   // "Hello, I'm Admin"
console.log(admin.canDelete()); // true
console.log(user.greet());    // "Hello, I'm User"
console.log(user.canDelete()); // false
```

**Expected Output:**
```
Hello, I'm Admin
true
Hello, I'm User
false
```

**Why This Output:** The factory returns different objects based on the `role` argument.

#### Example 2: Factory with Private State

```js
function createCounter() {
  let count = 0; // Private state
  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; },
  };
}

const counter1 = createCounter();
const counter2 = createCounter();

console.log(counter1.increment()); // 1
console.log(counter1.increment()); // 2
console.log(counter2.increment()); // 1 (independent)
```

**Expected Output:**
```
1
2
1
```

**Why This Output:** Each call to `createCounter` creates a new closure with its own `count`.

#### Example 3: Factory with Type Discrimination

```js
function createShape(type, ...args) {
  switch (type) {
    case "circle":
      return { type, radius: args[0], area: () => Math.PI * args[0] ** 2 };
    case "rectangle":
      return { type, width: args[0], height: args[1], area: () => args[0] * args[1] };
    default:
      throw new Error("Unknown shape");
  }
}

console.log(createShape("circle", 5).area().toFixed(2));       // "78.54"
console.log(createShape("rectangle", 4, 6).area());            // 24
```

**Expected Output:**
```
78.54
24
```

**Why This Output:** The factory hides the construction details and returns the appropriate object.

### Real-World Cases

- **React elements:** `React.createElement()` is a factory.
- **Document creation:** `document.createElement("div")`.
- **Connection factories:** Creating DB/HTTP clients.
- **Test fixtures:** `createUser()`, `createPost()`.

---

## Prototype Pattern

### Definitions

**Core Definition:** The Prototype pattern creates new objects by cloning an existing object (the prototype) rather than instantiating a class.

**Technical Definition:** In JavaScript, the Prototype pattern is natively supported via `Object.create(proto)`, which creates a new object whose `[[Prototype]]` is `proto`. The classic implementation also includes a `clone()` method on the prototype. Prototype-based inheritance allows objects to share behaviour without classes.

**Beginner-Friendly Explanation:** The Prototype pattern is like photocopying a document. Instead of writing a new document from scratch, you copy an existing one and modify it.

### Purposes

- To create new objects efficiently by copying.
- To share behaviour via the prototype chain.
- To avoid class hierarchies.
- To implement inheritance without `class`.

### Syntax Rules and Structure

```js
const prototype = {
  greet() { return `Hello, I'm ${this.name}`; },
};

const obj = Object.create(prototype);
obj.name = "Alice";
```

**Component Breakdown:**
- `prototype`: The object to inherit from.
- `Object.create(prototype)`: Creates a new object with the given prototype.
- The new object can add its own properties.

**Syntax Rules:**
- Use `Object.create()` to create objects with a specific prototype.
- Use `Object.getPrototypeOf()` to access an object's prototype.
- Properties on the prototype are shared; own properties are per-instance.
- `Object.create(null)` creates an object with no prototype.

**Constraints and Limitations:**
- Prototype chains can become complex.
- `this` binding depends on how methods are called.
- Deep prototype chains have lookup overhead.
- ES6 classes are syntactic sugar over prototypes.

### Annotated Code Examples

#### Example 1: Basic Prototype Inheritance

```js
// Step 1: Define a prototype object.
const animalProto = {
  breathe() { return `${this.name} is breathing`; },
  describe() { return `${this.name} is a ${this.species}`; },
};

// Step 2: Create objects that inherit from it.
const dog = Object.create(animalProto);
dog.name = "Rex";
dog.species = "dog";

const cat = Object.create(animalProto);
cat.name = "Whiskers";
cat.species = "cat";

console.log(dog.breathe());  // "Rex is breathing"
console.log(cat.describe()); // "Whiskers is a cat"
```

**Expected Output:**
```
Rex is breathing
Whiskers is a cat
```

**Why This Output:** Both `dog` and `cat` inherit `breathe` and `describe` from `animalProto`.

#### Example 2: Clone Method

```js
const prototype = {
  clone() {
    return Object.create(Object.getPrototypeOf(this), Object.getOwnPropertyDescriptors(this));
  },
  greet() { return `Hello, I'm ${this.name}`; },
};

const original = Object.create(prototype);
original.name = "Alice";

const copy = original.clone();
copy.name = "Bob";

console.log(original.greet()); // "Hello, I'm Alice"
console.log(copy.greet());     // "Hello, I'm Bob"
console.log(copy instanceof Object); // true
```

**Expected Output:**
```
Hello, I'm Alice
Hello, I'm Bob
true
```

**Why This Output:** `clone()` creates a new object with the same prototype and own properties.

### Real-World Cases

- **Object.create:** Standard way to set prototypes.
- **Library inheritance:** Backbone.js, older frameworks.
- **Mixins:** Combining behaviours via prototypes.
- **Polyfills:** Emulating class behaviour.

---

## Builder Pattern

### Definitions

**Core Definition:** The Builder pattern constructs complex objects step by step, separating the construction process from the representation.

**Technical Definition:** In JavaScript, the Builder pattern is typically implemented as a class or object with chainable methods that accumulate state, ending with a `build()` method that returns the final object. It is especially useful for creating objects with many optional configuration parameters.

**Beginner-Friendly Explanation:** The Builder pattern is like ordering a custom sandwich: you specify each ingredient one at a time (`addBread()`, `addCheese()`, `addLettuce()`), and at the end, you say "build it" to get your sandwich.

### Purposes

- To construct complex objects step by step.
- To avoid constructors with many parameters.
- To provide a readable, chainable API.
- To allow different representations from the same construction process.
- To defer object creation until all options are set.

### Syntax Rules and Structure

```js
class Builder {
  constructor() { this.parts = []; }
  addPart(part) { this.parts.push(part); return this; }
  build() { return { parts: this.parts }; }
}

const obj = new Builder().addPart("a").addPart("b").build();
```

**Component Breakdown:**
- Builder class/object with methods for each configurable part.
- Methods return `this` for chaining.
- `build()` returns the final object.

**Syntax Rules:**
- Methods typically return `this` to enable chaining.
- The final method (often `build()` or `create()`) returns the constructed object.
- Can be class-based or function-based.
- Can validate the object in `build()`.

**Constraints and Limitations:**
- Adds boilerplate for simple objects.
- Not always necessary; object literals suffice for simple cases.
- May be overkill for objects with few properties.

### Annotated Code Examples

#### Example 1: Fluent Builder

```js
class QueryBuilder {
  constructor() {
    this.query = { select: ["*"], from: "", where: [], orderBy: null, limit: null };
  }

  select(...fields) {
    this.query.select = fields;
    return this;
  }

  from(table) {
    this.query.from = table;
    return this;
  }

  where(condition) {
    this.query.where.push(condition);
    return this;
  }

  orderBy(field, direction = "ASC") {
    this.query.orderBy = `${field} ${direction}`;
    return this;
  }

  limit(n) {
    this.query.limit = n;
    return this;
  }

  build() {
    const { select, from, where, orderBy, limit } = this.query;
    let sql = `SELECT ${select.join(", ")} FROM ${from}`;
    if (where.length) sql += ` WHERE ${where.join(" AND ")}`;
    if (orderBy) sql += ` ORDER BY ${orderBy}`;
    if (limit) sql += ` LIMIT ${limit}`;
    return sql;
  }
}

const sql = new QueryBuilder()
  .select("id", "name")
  .from("users")
  .where("age > 18")
  .where("active = 1")
  .orderBy("name", "DESC")
  .limit(10)
  .build();

console.log(sql);
// "SELECT id, name FROM users WHERE age > 18 AND active = 1 ORDER BY name DESC LIMIT 10"
```

**Expected Output:**
```
SELECT id, name FROM users WHERE age > 18 AND active = 1 ORDER BY name DESC LIMIT 10
```

**Why This Output:** Each method adds to the internal state and returns `this`, enabling chaining. `build()` assembles the final SQL string.

#### Example 2: Form Data Builder

```js
class FormBuilder {
  constructor() { this.fields = {}; }

  text(name, value) { this.fields[name] = { type: "text", value }; return this; }
  email(name, value) { this.fields[name] = { type: "email", value }; return this; }
  number(name, value) { this.fields[name] = { type: "number", value }; return this; }

  build() {
    return { fields: { ...this.fields }, createdAt: new Date().toISOString() };
  }
}

const form = new FormBuilder()
  .text("name", "Alice")
  .email("email", "alice@example.com")
  .number("age", 30)
  .build();

console.log(form.fields);
// { name: { type: "text", value: "Alice" }, ... }
```

**Expected Output (approximate):**
```
{
  name: { type: 'text', value: 'Alice' },
  email: { type: 'email', value: 'alice@example.com' },
  age: { type: 'number', value: 30 }
}
```

**Why This Output:** Each method configures a field, and `build()` returns the final structure.

### Real-World Cases

- **SQL query builders:** Knex.js, Sequelize.
- **HTTP request builders:** Axios config, Fetch wrappers.
- **DOM builders:** Creating complex elements.
- **Test data builders:** `UserBuilder().withName("Alice").build()`.

---

## Object Pooling

### Definitions

**Core Definition:** Object pooling is a performance pattern that reuses a set of pre-created objects instead of creating and destroying them repeatedly.

**Technical Definition:** An object pool maintains a collection of reusable objects. When a consumer requests an object, the pool provides one from its available set (or creates one if none are available). When the consumer is done, the object is returned to the pool for reuse. This pattern is useful when object creation is expensive (e.g., database connections, game entities, DOM elements).

**Beginner-Friendly Explanation:** Object pooling is like a car rental service. Instead of buying a new car every time you need one, you borrow from a pool of cars and return them when you're done. The pool manages a limited supply and reuses them.

### Purposes

- To reduce the cost of frequent object creation/destruction.
- To limit the number of simultaneously active objects.
- To reduce garbage collection pressure.
- To improve performance in games, graphics, and network programming.
- To manage limited resources (connections, threads).

### Syntax Rules and Structure

```js
class ObjectPool {
  constructor(factory, reset, maxSize = 10) {
    this.factory = factory;
    this.reset = reset;
    this.maxSize = maxSize;
    this.available = [];
    this.inUse = new Set();
  }

  acquire() { /* return an object */ }
  release(obj) { /* return object to pool */ }
  get size() { /* total objects */ }
}
```

**Component Breakdown:**
- `factory`: A function that creates a new object when the pool is empty.
- `reset`: A function that resets an object before reuse.
- `available`: Array of idle objects.
- `inUse`: Set of objects currently checked out.

**Syntax Rules:**
- `acquire()` returns an object, creating one if necessary.
- `release(obj)` returns an object to the pool.
- Objects should be reset before reuse.
- Pool size is typically bounded.

**Constraints and Limitations:**
- Objects must be properly reset to avoid state leakage.
- Not beneficial for cheap objects (GC handles them fine).
- Adds complexity to the codebase.
- Releasing an object twice can corrupt the pool.

### Annotated Code Examples

#### Example 1: Basic Object Pool

```js
class ObjectPool {
  constructor(factory, reset, maxSize = 10) {
    this.factory = factory;
    this.reset = reset;
    this.maxSize = maxSize;
    this.available = [];
    this.inUse = new Set();
  }

  acquire() {
    let obj;
    if (this.available.length) {
      obj = this.available.pop();
      console.log("Reused object");
    } else if (this.inUse.size < this.maxSize) {
      obj = this.factory();
      console.log("Created new object");
    } else {
      throw new Error("Pool exhausted");
    }
    this.inUse.add(obj);
    return obj;
  }

  release(obj) {
    if (!this.inUse.has(obj)) throw new Error("Object not from this pool");
    this.inUse.delete(obj);
    this.reset(obj);
    this.available.push(obj);
    console.log("Released object");
  }

  get size() {
    return this.available.length + this.inUse.size;
  }
}

// Step 1: Define factory and reset functions.
const factory = () => ({ value: 0, data: null });
const reset = (obj) => { obj.value = 0; obj.data = null; };

// Step 2: Create the pool.
const pool = new ObjectPool(factory, reset, 3);

// Step 3: Acquire, use, release.
const a = pool.acquire(); // "Created new object"
a.value = 42;
pool.release(a);          // "Released object"

const b = pool.acquire(); // "Reused object"
console.log(b.value);     // 0 (reset)
console.log(a === b);     // true (same object reused)
```

**Expected Output:**
```
Created new object
Released object
Reused object
0
true
```

**Why This Output:** The pool reuses the released object, resetting its state before returning it.

#### Example 2: DOM Element Pool

```js
class DOMElementPool {
  constructor(tagName, maxSize = 20) {
    this.tagName = tagName;
    this.maxSize = maxSize;
    this.available = [];
  }

  acquire() {
    return this.available.pop() || document.createElement(this.tagName);
  }

  release(element) {
    element.textContent = "";
    element.className = "";
    element.removeAttribute("style");
    if (this.available.length < this.maxSize) {
      this.available.push(element);
    }
  }
}

// Usage in a list renderer:
const liPool = new DOMElementPool("li");
const list = document.getElementById("list");

function addItem(text) {
  const li = liPool.acquire();
  li.textContent = text;
  list.appendChild(li);
}

function removeItem(li) {
  list.removeChild(li);
  liPool.release(li);
}
```

**Note:** This example requires a DOM environment.

### Real-World Cases

- **Game development:** Bullets, particles, enemies.
- **Database connections:** Connection pools (e.g., `pg-pool` in Node.js).
- **Web workers:** Reusing workers.
- **Node.js Buffer pools:** Reusing buffers.
- **Graphics:** Sprite pools, texture atlases.

---

## References

- MDN Web Docs – Working with objects — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects
- MDN Web Docs – Object.create() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create
- MDN Web Docs – Classes — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- MDN Web Docs – Closures — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures
- MDN Web Docs – JavaScript modules — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- Addy Osmani – Learning JavaScript Design Patterns — https://www.patterns.dev/
- Addy Osmani – JavaScript Design Patterns (free book) — https://addyosmani.com/resources/essentialjsdesignpatterns/book/
- Patterns.dev – Design Patterns — https://www.patterns.dev/
- Patterns.dev – Module Pattern — https://www.patterns.dev/vanilla/module-pattern/
- Patterns.dev – Factory Pattern — https://www.patterns.dev/vanilla/factory-pattern/
- Patterns.dev – Singleton Pattern — https://www.patterns.dev/vanilla/singleton-pattern/
- Patterns.dev – Prototype Pattern — https://www.patterns.dev/vanilla/prototype-pattern/
- Refactoring Guru – Design Patterns — https://refactoring.guru/design-patterns
- Refactoring Guru – Builder — https://refactoring.guru/design-patterns/builder
- Refactoring Guru – Object Pool — https://refactoring.guru/design-patterns/object-pool
- Stoyan Stefanov – JavaScript Patterns — https://www.oreilly.com/library/view/javascript-patterns/9781449399115/
- Eric Miraglia – A JavaScript Module Pattern — https://yuiblog.com/blog/2007/06/12/module-pattern/
- Ben Cherry – JavaScript Module Pattern: In-Depth — https://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html
- Node.js Docs – Modules: CommonJS modules — https://nodejs.org/api/modules.html
- Wikipedia – Software design pattern — https://en.wikipedia.org/wiki/Software_design_pattern
- Wikipedia – Object pool pattern — https://en.wikipedia.org/wiki/Object_pool_pattern