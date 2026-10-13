# Module Patterns — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Module patterns are reusable architectural and design solutions for organising JavaScript code into self-contained, encapsulated units with clear public and private boundaries. They address the fundamental challenge of structuring large codebases without polluting the global namespace.

**Technical Definition:** Module patterns in JavaScript leverage closures, IIFEs (Immediately Invoked Function Expressions), and modern ES module syntax to create encapsulated scopes. The module pattern uses closures to create private and public members, providing encapsulation and namespace management. Before ES6 introduced native modules, developers relied on these patterns to organise larger applications. Modern patterns extend these foundations with dependency injection for loose coupling and module federation for distributed module sharing across independently deployed applications.

**Beginner-Friendly Explanation:** Module patterns are the "blueprints" developers use to organise their JavaScript code into separate, manageable pieces. Instead of dumping everything into one giant file, you split your code into modules — each with its own private variables and a clear public interface. Think of it like organising a kitchen: pots go here, spices go there, and when you need something, you know exactly where to find it.

### Key Characteristics

- **Encapsulation:** Private variables and functions are hidden from the outside world.
- **Explicit interfaces:** Only explicitly exposed members are accessible to consumers.
- **Namespace management:** Patterns like the namespace pattern reduce global variable pollution.
- **Loose coupling:** Dependency injection allows components to be decoupled from their dependencies.
- **Scalability:** Module federation enables distributed module sharing across applications.
- **Testability:** Clear boundaries make unit testing easier.

### Prerequisites

- Understanding of JavaScript functions, scope, and closures.
- Familiarity with IIFEs (Immediately Invoked Function Expressions).
- Basic knowledge of ES module syntax (`import`/`export`).
- Awareness of the global namespace and its risks.

### Related Programming Areas

- **Code organisation:** Structuring large applications into manageable pieces.
- **Design patterns:** Module patterns are part of the broader JavaScript design pattern catalogue.
- **Micro-frontends:** Module federation enables distributed frontend architectures.
- **Dependency management:** DI and module federation manage dependencies across boundaries.
- **Build tooling:** Webpack, Rollup, and Vite support module patterns natively.

### Core Concepts / Features

---

## Module Pattern

### Definitions

**Core Definition:** The module pattern is a design pattern that uses closures to create private and public members, providing encapsulation and namespace management in JavaScript.

**Technical Definition:** Per CoreUI, "The module pattern uses closures to create private and public members, providing encapsulation and namespace management in JavaScript. The most maintainable approach uses IIFE (Immediately Invoked Function Expression) or ES6 modules for encapsulation". Modules should be Immediately-Invoked-Function-Expressions (IIFE) to allow for private scopes — a closure that protects variables and methods, returning an object instead of a function. Code outside the closure cannot access private variables since they are not in the same scope.

**Beginner-Friendly Explanation:** The module pattern is like a shop with a counter. Inside the shop (the module), you have private storage and workspaces. But customers can only interact with what you place on the counter (the public API). Everything else stays hidden.

### Purposes

- To encapsulate private implementation details and expose only a public API.
- To prevent global namespace pollution.
- To create private variables and methods.
- To organise code into logical, maintainable units.
- To provide a clean, reusable interface for related functionality.

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
- Private members cannot be accessed for unit testing.

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

**Technical Definition:** Per the revealing module pattern documentation, "The only difference is that the revealing module pattern was engineered as a way to ensure that all methods and variables are kept private until they are explicitly exposed". The returned object contains references to the functions defined inside the IIFE. The purpose is to maintain encapsulation and reveal certain variables and methods returned in an object literal.

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
- An obvious disadvantage is the inability to reference private methods from outside, which can pose unit testing challenges.

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

#### Example 2: Comparing Module Pattern vs Revealing Module Pattern

```js
// Module Pattern:
const Exposer1 = (function() {
  var privateVariable = 10;
  var privateMethod = function() {
    console.log('Inside a private method!');
    privateVariable++;
  };

  return {
    first: function() {
      console.log('This is a method I want to expose!');
    },
    second: function() {
      privateMethod();
    }
  };
})();

// Revealing Module Pattern (equivalent):
const Exposer2 = (function() {
  var privateVariable = 10;
  var privateMethod = function() {
    console.log('Inside a private method!');
    privateVariable++;
  };
  var methodToExpose = function() {
    console.log('This is a method I want to expose!');
  };
  var otherMethodIWantToExpose = function() {
    privateMethod();
  };

  return {
    first: methodToExpose,
    second: otherMethodIWantToExpose
  };
})();
```

**Expected Output:** Both modules behave identically. The revealing module pattern makes the public API explicit at the bottom of the module.

**Why This Output:** In the revealing module pattern, all functions are defined as private function declarations, and the return statement maps public names to private functions, making the public interface self-documenting.

### Real-World Cases

- **jQuery plugins:** Revealing module pattern for plugin APIs.
- **Framework modules:** AngularJS services used the revealing module pattern.
- **Namespace management:** Organising large codebases before ES6 modules.

---

## Namespace Pattern

### Definitions

**Core Definition:** The namespace pattern is a code organisation technique that uses a single global object as a container for methods and properties, reducing global namespace pollution by grouping related functionality under one global variable.

**Technical Definition:** Per the Compile-N-Run documentation, "The namespace pattern is one of the simplest approaches to code organization. It uses a single global object as a container for methods and properties". Per Oracle's developer companion, "Since app defines a unique name to unambiguously qualify its exported function names, JavaScript developers call it a namespace". The namespace pattern solves the problem of name conflicts, but not that of data encapsulation.

**Beginner-Friendly Explanation:** The namespace pattern is like having a labelled folder on your desk. Instead of scattering papers everywhere (global variables), you put everything related into one folder (namespace). Now you have one folder to manage instead of dozens of loose papers.

### Purposes

- To reduce global variables to just one.
- To organise related functionality together.
- To prevent naming conflicts between different scripts.
- To provide a simple, understandable structure for code organisation.
- To make code easier to navigate and maintain.

### Syntax Rules and Structure

```js
// Define a namespace (with fallback):
var MyApp = MyApp || {};

// Add functionality to namespace:
MyApp.utilities = {
  formatDate: function(date) {
    return date.toLocaleDateString();
  },
  calculateTax: function(amount) {
    return amount * 0.08;
  }
};

// Add another module to the namespace:
MyApp.models = {
  User: function(name, email) {
    this.name = name;
    this.email = email;
  }
};
```

**Component Breakdown:**
- `var MyApp = MyApp || {}`: Creates the namespace if it doesn't exist, or uses the existing one.
- `MyApp.utilities`: A sub-namespace for utility functions.
- `MyApp.models`: Another sub-namespace for data models.

**Syntax Rules:**
- Use the `||` operator to avoid overwriting an existing namespace.
- Nested namespaces can be created for deeper organisation.
- Functions and objects are attached as properties of the namespace object.

**Constraints and Limitations:**
- No true privacy or encapsulation — all members are publicly accessible.
- Dependencies are not explicit.
- Possible namespace collisions if not careful.
- Does not solve the problem of data encapsulation.

### Annotated Code Examples

#### Example 1: Basic Namespace Pattern

```js
// Step 1: Define the namespace (or reuse existing).
var MyApp = MyApp || {};

// Step 2: Add utility functions.
MyApp.utilities = {
  formatDate: function(date) {
    return date.toLocaleDateString();
  },
  calculateTax: function(amount) {
    return amount * 0.08;
  }
};

// Step 3: Add a data model.
MyApp.models = {
  User: function(name, email) {
    this.name = name;
    this.email = email;
  }
};

// Step 4: Use the namespace.
var today = new Date();
console.log(MyApp.utilities.formatDate(today)); // Current date
console.log(MyApp.utilities.calculateTax(100)); // 8
var user = new MyApp.models.User("Alice", "alice@example.com");
console.log(user.name); // "Alice"
```

**Expected Output:**
```
10/13/2026 (or current date)
8
Alice
```

**Why This Output:** All functionality is grouped under the `MyApp` namespace object. Only one global variable (`MyApp`) is created, reducing namespace pollution.

#### Example 2: Nested Namespaces

```js
// Step 1: Define nested namespaces.
var MyCompany = MyCompany || {};
MyCompany.Modules = MyCompany.Modules || {};
MyCompany.Modules.Utils = MyCompany.Modules.Utils || {};

// Step 2: Add functions to the nested namespace.
MyCompany.Modules.Utils.formatCurrency = function(amount) {
  return "$" + amount.toFixed(2);
};

MyCompany.Modules.Utils.parseCurrency = function(str) {
  return parseFloat(str.replace("$", ""));
};

// Step 3: Use the nested namespace.
console.log(MyCompany.Modules.Utils.formatCurrency(42.5)); // "$42.50"
console.log(MyCompany.Modules.Utils.parseCurrency("$99.99")); // 99.99
```

**Expected Output:**
```
$42.50
99.99
```

**Why This Output:** Nested namespaces provide deeper organisation, grouping related utilities under `MyCompany.Modules.Utils`.

### Real-World Cases

- **Legacy JavaScript libraries:** Many older libraries used namespaces to avoid global pollution.
- **jQuery plugins:** Plugins often use `$.fn.pluginName` as a namespace.
- **Enterprise applications:** Large codebases use namespaces for organisational clarity.

---

## Dependency Injection

### Definitions

**Core Definition:** Dependency injection (DI) is a design pattern where an object receives the instances it depends on from an external source rather than creating them itself.

**Technical Definition:** Per ExpressoTS documentation, "Dependency Injection (DI) is a design pattern where an object receives the instances it depends on rather than constructing them itself". Per the pair guide, "Dependency injection enables loose coupling, testability, and maintainable code by managing dependencies through external configuration rather than hard-coded instantiation". The most common pattern is constructor injection, where dependencies are declared as constructor parameters.

**Beginner-Friendly Explanation:** Dependency injection is like ordering a pizza instead of making it from scratch. Instead of your code creating all its dependencies internally, you "inject" them from outside. This makes it easier to swap ingredients (dependencies) and test different combinations.

### Purposes

- To achieve loose coupling between components.
- To improve testability by allowing mock dependencies to be injected.
- To manage dependencies through external configuration.
- To decouple component declaration from instantiation.
- To enable hierarchical dependency resolution and lifecycle management.

### Syntax Rules and Structure

```js
// Without DI (hard-coded dependency):
class UserService {
  constructor() {
    this.db = new Database(); // Hard-coded
  }
}

// With DI (injected dependency):
class UserService {
  constructor(database) {
    this.db = database; // Injected
  }
}

// Usage:
const db = new Database();
const service = new UserService(db);
```

**Component Breakdown:**
- **Constructor injection:** Dependencies are passed as constructor parameters.
- **Injector/container:** Manages the creation and resolution of dependencies.
- **Token/interface:** Identifies the dependency to be injected.

**Syntax Rules:**
- Dependencies should be declared as constructor parameters.
- An injector or container resolves and provides the dependencies.
- Dependencies should be registered with the container before use.

**Constraints and Limitations:**
- Adds complexity to small applications.
- Requires careful lifecycle management.
- Can make code harder to trace if overused.

### Annotated Code Examples

#### Example 1: Manual Dependency Injection

```js
// Step 1: Define a logger interface.
class ConsoleLogger {
  log(message) {
    console.log(`[LOG] ${message}`);
  }
}

class FileLogger {
  log(message) {
    // Write to file...
    console.log(`[FILE] ${message}`);
  }
}

// Step 2: Define a service that depends on a logger.
class UserService {
  constructor(logger) {
    this.logger = logger; // Injected dependency
  }

  createUser(name) {
    this.logger.log(`Creating user: ${name}`);
    return { name, id: Date.now() };
  }
}

// Step 3: Inject different loggers.
const consoleService = new UserService(new ConsoleLogger());
consoleService.createUser("Alice"); // [LOG] Creating user: Alice

const fileService = new UserService(new FileLogger());
fileService.createUser("Bob"); // [FILE] Creating user: Bob
```

**Expected Output:**
```
[LOG] Creating user: Alice
[FILE] Creating user: Bob
```

**Why This Output:** The `UserService` receives its logger dependency from outside. By injecting different loggers, the same service can be configured for different environments without changing its code.

#### Example 2: DI Container with `didi`

```js
import { Injector } from 'didi';

// Step 1: Create an injector.
const injector = new Injector();

// Step 2: Register services.
injector.register('logger', new ConsoleLogger());
injector.register('database', new Database());

// Step 3: Register a service that depends on others.
injector.register('userService', {
  $inject: ['logger', 'database'],
  construct: (logger, database) => {
    return new UserService(logger, database);
  }
});

// Step 4: Resolve the service — dependencies are injected automatically.
const userService = injector.get('userService');
userService.createUser("Charlie");
```

**Expected Output:** The `userService` is created with its dependencies automatically resolved and injected.

**Why This Output:** The `didi` injector manages the dependency graph, instantiating components as needed and caching them for future re-use.

### Real-World Cases

- **Angular:** Uses DI extensively for services and components.
- **NestJS:** DI is a core architectural principle.
- **Node.js libraries:** `didi` is used by Karma, diagram-js, and Wuffle.
- **Vue.js:** Uses `provide`/`inject` for dependency injection.

---

## Module Federation

### Definitions

**Core Definition:** Module Federation is a Webpack 5 feature that allows a JavaScript application to dynamically load code from another application at runtime, enabling distributed module sharing across independently built and deployed micro-frontends.

**Technical Definition:** Per the MakeMyTrip engineering blog, "Module Federation in Webpack 5 solves [the micro-frontend code sharing] problem by allowing apps to load code dynamically from other applications at runtime, so different teams can build and release features independently". Module Federation is officially supported starting from Webpack 5. It enables sharing dependencies between micro-frontends.

**Beginner-Friendly Explanation:** Module federation is like a library system where different branches (applications) can borrow books (modules) from each other at runtime. Each branch is independent — it can update its own collection without affecting others — but they can all share resources.

### Purposes

- To enable micro-frontend architectures where teams can deploy independently.
- To share code between applications without bundling it in advance.
- To reduce duplication of shared dependencies (e.g., React, React DOM).
- To allow runtime loading of remote modules.
- To support independent deployment and development of frontend features.

### Syntax Rules and Structure

```js
// webpack.config.js for the host (consumer):
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: "hostApp",
      remotes: {
        remoteApp: "remoteApp@http://localhost:3001/remoteEntry.js",
      },
      shared: {
        react: { singleton: true },
        "react-dom": { singleton: true },
      },
    }),
  ],
};
```

```js
// webpack.config.js for the remote (provider):
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: "remoteApp",
      filename: "remoteEntry.js",
      exposes: {
        "./Header": "./src/Header",
        "./Footer": "./src/Footer",
      },
      shared: {
        react: { singleton: true },
        "react-dom": { singleton: true },
      },
    }),
  ],
};
```

**Component Breakdown:**
- **`name`:** A unique identifier for the application.
- **`filename`:** The name of the remote entry file (default: `remoteEntry.js`).
- **`exposes`:** Modules that this application makes available to others.
- **`remotes`:** Modules that this application consumes from other applications.
- **`shared`:** Dependencies that should be shared between applications.

**Syntax Rules:**
- The remote application must be running for the host to load its modules.
- Shared dependencies should be configured as singletons to avoid duplicate instances.
- Remote modules are loaded dynamically via `import()`.

**Constraints and Limitations:**
- Requires Webpack 5 or higher.
- Both applications must be available at runtime.
- Caching and versioning of remote modules can be complex.
- Adds complexity to build configuration.

### Annotated Code Examples

#### Example 1: Exposing and Consuming a Remote Module

```js
// Step 1: In the remote app (app1), expose a Header component.
// app1/src/Header.js
const Header = () => <header>Hello from App 1</header>;
export default Header;

// Step 2: Configure Webpack for app1.
// app1/webpack.config.js
new ModuleFederationPlugin({
  name: "app1",
  filename: "remoteEntry.js",
  exposes: {
    "./Header": "./src/Header",
  },
  shared: {
    react: { singleton: true },
  },
});

// Step 3: In the host app (app2), consume the Header.
// app2/webpack.config.js
new ModuleFederationPlugin({
  name: "app2",
  remotes: {
    app1: "app1@http://localhost:3001/remoteEntry.js",
  },
  shared: {
    react: { singleton: true },
  },
});

// Step 4: Dynamically import the remote module.
// app2/src/App.js
import React, { Suspense } from "react";
const RemoteHeader = React.lazy(() => import("app1/Header"));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <RemoteHeader />
    </Suspense>
  );
}
```

**Expected Output:** The `Header` component from `app1` is rendered inside `app2` at runtime, without being bundled into `app2`'s build.

**Why This Output:** The `ModuleFederationPlugin` in `app1` exposes the `Header` component. The `remotes` configuration in `app2` points to `app1`'s `remoteEntry.js`. The dynamic `import()` loads the remote component at runtime.

#### Example 2: Sharing Dependencies

```js
// Both apps share React as a singleton:
shared: {
  react: { singleton: true },
  "react-dom": { singleton: true },
}
```

**Expected Output:** Both applications use the same instance of React, avoiding duplicate copies and ensuring that React features like Context work correctly across micro-frontends.

**Why This Output:** The `shared` configuration tells Webpack to use a single instance of React across both applications. Without this, each micro-frontend would bundle its own copy of React, leading to problems with React Context and hooks.

### Real-World Cases

- **Enterprise micro-frontends:** Large organisations use Module Federation to split monolithic frontends into independently deployable units.
- **Angular applications:** `@angular-architects/module-federation` integrates Module Federation with Angular.
- **React applications:** Module Federation is widely used in React micro-frontend architectures.
- **Supply chain applications:** Tencent Cloud documented splitting a supply chain system into independent sub-apps using Module Federation.

---

## References

- CoreUI – How to implement module pattern in JavaScript — https://coreui.io/answers/how-to-implement-module-pattern-in-javascript/
- DigitalOcean – Module Design Pattern in JavaScript — https://www.digitalocean.com/community/conceptual-articles/module-design-pattern-in-javascript
- Revealing Module Pattern – Glossary Documentation — https://raw.githubusercontent.com/PatternsDev/skills
- Compile-N-Run – JavaScript Module Patterns — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/javascript/10-javascript-modules/5-javascript-module-patterns.mdx
- Oracle – Developer's Companion: Namespace Pattern — https://docs.oracle.com
- Microsoft Learn – Namespacing in JavaScript — https://learn.microsoft.com
- ExpressoTS – Dependency Injection Documentation — https://doc.expresso-ts.com
- didi – Dependency Injection for JavaScript — https://github.com/nikku/didi
- Angular – Modular design with dependency injection — https://v20.angular.dev
- MakeMyTrip Engineering – Micro Frontend Architecture and Implementation with Module Federation — https://tech.makemytrip.com/micro-frontend-architecture-and-implementation-with-module-federation
- Webpack – Module Federation — https://webpack.js.org/concepts/module-federation/
- NGXS – Module Federation — https://www.ngxs.io
- AWS – Create a portal for micro-frontends by using AWS Amplify, Angular, and Module Federation — https://docs.aws.amazon.com
- Addy Osmani – Essential JS Design Patterns (Revealing Module Pattern) — https://addyosmani.com/resources/essentialjsdesignpatterns/book/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- James Edwards – Namespacing in JavaScript — https://learn.microsoft.com