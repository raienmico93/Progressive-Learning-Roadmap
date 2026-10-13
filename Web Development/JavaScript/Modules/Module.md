# Module Fundamentals — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A module is a self-contained unit of JavaScript code that encapsulates related functions, classes, or variables, and explicitly controls what it exposes to other modules through exports and imports.

**Technical Definition:** Per the ECMAScript specification, a module is a distinct source text that is parsed and evaluated as a `Module` record. Any file containing a top-level `import` or `export` declaration is considered a module; a file without any top-level import or export declarations is treated as a script whose contents are available in the global scope. Modules are executed within their own scope, not in the global scope, meaning variables, functions, and classes declared in a module are not visible outside unless explicitly exported. ES modules use a static structure with compile-time optimisation, enabling tree shaking and predictable bundling. Modules are cached after their first evaluation; subsequent imports return the same module instance.

**Beginner-Friendly Explanation:** A module is like a separate file that contains related code. Instead of putting everything in one giant file, you split your code into smaller, focused files. Each module can choose what to share with others (exports) and what to keep private. Other modules can then import what they need. Think of modules as building blocks that you can combine to build a complete application.

### Key Characteristics

- **Encapsulation:** Modules have their own scope; variables and functions are private by default unless explicitly exported.
- **Explicit interfaces:** Modules declare what they expose via `export` and what they consume via `import`.
- **Static analysis:** ES module imports and exports are static (determined at parse time), enabling tree shaking and dead code elimination.
- **Single evaluation:** A module is evaluated only once, regardless of how many times it is imported.
- **Caching:** Module instances are cached by URL; all importers share the same instance.
- **Strict mode by default:** ES modules always run in strict mode; `"use strict"` directives are unnecessary.
- **Live bindings:** Imported bindings are live; when the exporting module changes a value, the change is visible to importers.
- **Deferred execution:** Module code runs after the HTML document is parsed (in browsers), and dependencies are resolved before execution.

### Prerequisites

- Basic understanding of JavaScript functions, variables, and scope.
- Familiarity with the file system and how to include JavaScript files in HTML.
- Awareness of build tools (helpful but not required for native ES modules).

### Related Programming Areas

- **Code organisation:** Breaking large codebases into manageable, reusable pieces.
- **Dependency management:** Modules declare and resolve their dependencies.
- **Bundling and build tools:** Webpack, Rollup, and Vite use module graphs for optimisation.
- **Tree shaking:** Eliminating unused code by analysing module exports.
- **Dynamic imports:** Lazy-loading modules on demand.
- **Node.js module systems:** CommonJS (CJS) and ES Modules (ESM).

### Core Concepts / Features

---

## Modules

### Definitions

**Core Definition:** A module is a self-contained unit of code that encapsulates related functionality and exposes a public API through exports.

**Technical Definition:** Per MDN, "JavaScript programs started off pretty small — most of its usage in the early days was to do isolated scripting tasks... Complex projects necessitate a mechanism for splitting JavaScript programs into separate modules that can be imported when needed". A module is defined by the presence of top-level `import` or `export` declarations. Modules are executed within their own scope, not in the global scope. The module system provides explicit imports and exports, enabling better code structure and reusability.

**Beginner-Friendly Explanation:** A module is a file containing JavaScript code that you can reuse in other files. It keeps its variables and functions private until you decide to share them. Think of a module as a toolbox: you keep your tools inside, and you can lend specific tools to others.

### Purposes

- To organise code into smaller, manageable, and reusable pieces.
- To prevent naming conflicts by encapsulating variables within module scope.
- To make dependencies explicit through import and export declarations.
- To enable code reuse across different parts of an application or across projects.
- To support maintainability by isolating changes to specific modules.
- To facilitate testing by allowing individual modules to be tested in isolation.

### Syntax Rules and Structure

```js
// Module file (module.js):
export const name = "MyModule";
export function greet(user) {
  return `Hello, ${user}!`;
}

// Importing module (main.js):
import { name, greet } from './module.js';
console.log(greet("Alice")); // "Hello, Alice!"
```

**Component Breakdown:**
- `export`: Marks a declaration as available to other modules.
- `import`: Brings exported bindings from another module into the current scope.
- `from './module.js'`: The module specifier (path or package name).

**Syntax Rules:**
- A file with any top-level `import` or `export` is a module.
- A file without any top-level import/export is a script (global scope).
- ES modules always run in strict mode.
- Module specifiers must be exact paths (no default extensions in Node.js ESM).

**Constraints and Limitations:**
- ES modules are not supported in older browsers without transpilation.
- Modules must be served with the correct MIME type (`text/javascript`).
- Module scripts are deferred by default in browsers.

### Annotated Code Examples

#### Example 1: Basic Module

```js
// Step 1: Create a module file (math.js).
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export const PI = 3.14159;

// Step 2: Import and use in another file (main.js).
import { add, PI } from './math.js';

console.log(add(2, 3));   // 5
console.log(PI);          // 3.14159
```

**Expected Output:**
```
5
3.14159
```

**Why This Output:** The `math.js` module exports `add`, `subtract`, and `PI`. The `main.js` module imports `add` and `PI` and uses them. The imported bindings are live and read-only.

#### Example 2: Module Scope

```js
// module.js
const privateVar = "I am private";
export const publicVar = "I am public";

// main.js
import { publicVar } from './module.js';
console.log(publicVar);   // "I am public"
console.log(privateVar);  // ReferenceError: privateVar is not defined
```

**Expected Output:**
```
I am public
ReferenceError: privateVar is not defined
```

**Why This Output:** `privateVar` is not exported, so it is not accessible outside the module. Only `publicVar` is available to importers.

### Real-World Cases

- **Utility modules:** A `math.js` module with reusable mathematical functions.
- **API clients:** A module that encapsulates HTTP request logic.
- **Component modules:** In frameworks like React, each component is often its own module.
- **Configuration modules:** A module that exports application settings.

---

## Why Modules Matter

### Definitions

**Core Definition:** Modules matter because they solve fundamental problems in software development: code organisation, reusability, encapsulation, and dependency management.

**Technical Definition:** Per the freeCodeCamp guide, "Having a large amount of code in a single file can be problematic, so it's helpful to split the code into multiple parts. This is where modules come in handy". Modules allow developers to break their code into smaller, reusable pieces that can be imported and exported between different parts of an application. They provide explicit interfaces, making dependencies clear and enabling static analysis for optimisation.

**Beginner-Friendly Explanation:** Imagine trying to build a house with all the materials dumped in one pile. It would be chaos. Modules are like organising those materials into labelled boxes: wood here, nails there, tools in another box. When you need something, you know exactly where to find it, and you can reuse the same box in multiple projects.

### Purposes

- To improve code organisation and readability by separating concerns.
- To enable code reusability across different parts of an application or across projects.
- To prevent naming conflicts by encapsulating code in module scope.
- To make dependencies explicit and manageable.
- To facilitate collaboration by allowing developers to work on separate modules independently.
- To enable build-time optimisations like tree shaking and dead code elimination.
- To support lazy loading and code splitting for performance.

### Syntax Rules and Structure

Modules are used by creating separate files and using `import`/`export`:

```js
// user.js — a module for user-related functionality.
export function createUser(name) {
  return { name, id: Date.now() };
}

// app.js — imports and uses the user module.
import { createUser } from './user.js';
const user = createUser("Alice");
```

**Component Breakdown:**
- Separation of concerns: Each module handles one responsibility.
- Explicit dependencies: `import` statements make dependencies clear.
- Encapsulation: Only exported members are accessible.

**Syntax Rules:**
- Each module should have a clear, single purpose.
- Dependencies should be explicitly declared via imports.
- Avoid circular dependencies where possible.

**Constraints and Limitations:**
- Too many small modules can increase complexity.
- Over-modularisation can make code harder to navigate.

### Annotated Code Examples

#### Example 1: Organising Code with Modules

```js
// Without modules — everything in one file:
// let userName = "Alice";
// function formatName(name) { ... }
// function saveUser(user) { ... }
// function sendEmail(user) { ... }
// ... hundreds of lines

// With modules — separated into focused files:

// utils.js
export function formatName(name) {
  return name.trim().toLowerCase();
}

// api.js
export function saveUser(user) {
  return fetch('/api/users', { method: 'POST', body: JSON.stringify(user) });
}

// email.js
export function sendEmail(user) {
  return fetch('/api/email', { method: 'POST', body: JSON.stringify(user) });
}

// app.js
import { formatName } from './utils.js';
import { saveUser } from './api.js';
import { sendEmail } from './email.js';

const user = { name: formatName("  Alice  ") };
await saveUser(user);
await sendEmail(user);
```

**Expected Output:** Each module is focused on a single responsibility, and the main file clearly shows the application's dependencies.

**Why This Output:** The code is organised into modules by concern. Each module can be tested, reused, and maintained independently.

### Real-World Cases

- **Large applications:** Breaking a monolithic codebase into modules for maintainability.
- **Library development:** Publishing reusable modules to npm.
- **Team collaboration:** Different developers can work on different modules without conflicts.
- **Performance:** Code splitting and lazy loading modules on demand.

---

## Module Scope

### Definitions

**Core Definition:** Module scope is the lexical environment created for each module, in which all top-level declarations are private to the module unless explicitly exported.

**Technical Definition:** Per the TypeScript handbook, "Modules are executed within their own scope, not in the global scope. This means that variables, functions, classes, etc. declared in a module are not visible outside the module unless they are explicitly exported". ES modules always run in strict mode; `"use strict"` directives are unnecessary and have no effect. The module scope is separate from the global scope and from other modules' scopes.

**Beginner-Friendly Explanation:** Module scope means that everything you declare in a module is private by default. It's like having your own private room — you can keep things inside, and only things you explicitly put on the shelf (export) are visible to others.

### Purposes

- To provide encapsulation and prevent global namespace pollution.
- To avoid naming conflicts between modules.
- To make it explicit what a module exposes to the outside world.
- To enable safe refactoring without affecting other parts of the application.

### Syntax Rules and Structure

```js
// module.js
const privateVariable = "not exported"; // Only accessible inside this module
export const publicVariable = "exported"; // Accessible to importers

function privateFunction() { ... } // Only accessible inside this module
export function publicFunction() { ... } // Accessible to importers
```

**Component Breakdown:**
- Top-level declarations without `export` are private.
- Top-level declarations with `export` are public.
- The module's scope is sealed; there is no way to access private members from outside.

**Syntax Rules:**
- All top-level code in a module runs in module scope.
- Variables declared with `var`, `let`, or `const` at the top level are module-scoped.
- Function and class declarations at the top level are module-scoped unless exported.

**Constraints and Limitations:**
- Module scope is not accessible from the global scope.
- There is no way to "reach into" a module and access private members (except through exported methods that expose them).

### Annotated Code Examples

#### Example 1: Private vs Public Members

```js
// counter.js
let count = 0; // Private — not exported

export function increment() {
  count++;
  return count;
}

export function decrement() {
  count--;
  return count;
}

export function getCount() {
  return count;
}

// main.js
import { increment, getCount } from './counter.js';

console.log(increment()); // 1
console.log(increment()); // 2
console.log(getCount());  // 2
console.log(count);       // ReferenceError: count is not defined
```

**Expected Output:**
```
1
2
2
ReferenceError: count is not defined
```

**Why This Output:** `count` is module-scoped and not exported, so it cannot be accessed from `main.js`. The exported functions close over `count` and provide controlled access.

#### Example 2: No Global Pollution

```js
// module.js
const name = "Module"; // Does not create a global variable
function helper() { ... } // Does not create a global function

// main.js
console.log(window.name);    // undefined (in browser)
console.log(typeof helper);  // "undefined"
```

**Expected Output:**
```
undefined
undefined
```

**Why This Output:** Module-scoped declarations do not leak into the global scope, preventing global namespace pollution.

### Real-World Cases

- **Utility libraries:** Private helper functions that are not part of the public API.
- **State management:** Module-level state that is accessible only through exported functions.
- **Configuration:** Private configuration that is not exposed to consumers.

---

## Module Exports

### Definitions

**Core Definition:** Module exports are the declarations in a module that are made available to other modules through the `export` keyword.

**Technical Definition:** Per the freeCodeCamp guide, "Exporting module features The first thing you need to do to get access to module features is export them. This is done using the `export` statement". There are two types of exports: **named exports** and **default exports**. Named exports allow multiple values to be exported from a module; default exports represent a single main value. The `export` keyword can be applied to declarations or used as a standalone statement.

**Beginner-Friendly Explanation:** Exports are the things you choose to share from your module. Think of a module as a shop: the `export` keyword puts items on the shelves for customers (other modules) to buy. You can put many items on the shelf (named exports) or have one main item (default export).

### Purposes

- To define the public API of a module.
- To share functions, objects, variables, or classes with other modules.
- To control what is visible and what remains private.
- To enable consumers to import only what they need.

### Syntax Rules and Structure

```js
// Named exports:
export const name = "value";
export function fn() { ... }
export class MyClass { ... }

// Default export:
export default function() { ... }
export default class { ... }
export default expression;

// Export list:
const a = 1, b = 2;
export { a, b };

// Renaming exports:
export { a as alpha, b as beta };
```

**Component Breakdown:**
- `export const/function/class`: Exports a declaration directly.
- `export default`: Exports a single value as the module's default.
- `export { ... }`: Exports an explicit list of bindings.

**Syntax Rules:**
- A module can have any number of named exports.
- A module can have at most one default export.
- Default exports are imported without curly braces; named exports require curly braces.
- Export declarations must be at the top level of the module (not inside blocks or functions).

**Constraints and Limitations:**
- Mixing default and named exports is allowed but discouraged by style guides.
- Default exports can make refactoring harder because importers can name them arbitrarily.
- Named exports are preferred for better tooling support and tree shaking.

### Annotated Code Examples

#### Example 1: Named Exports

```js
// utils.js
export const PI = 3.14159;
export function double(n) {
  return n * 2;
}
export class Calculator {
  add(a, b) { return a + b; }
}

// main.js
import { PI, double, Calculator } from './utils.js';
console.log(PI);          // 3.14159
console.log(double(5));   // 10
const calc = new Calculator();
console.log(calc.add(2, 3)); // 5
```

**Expected Output:**
```
3.14159
10
5
```

**Why This Output:** The named exports are imported by their exact names, enclosed in curly braces.

#### Example 2: Default Export

```js
// logger.js
export default function log(message) {
  console.log(`[LOG] ${message}`);
}

// main.js
import log from './logger.js';
log("Hello"); // "[LOG] Hello"
```

**Expected Output:**
```
[LOG] Hello
```

**Why This Output:** The default export is imported without curly braces. The importer can choose any name for the default import.

#### Example 3: Renaming Exports and Imports

```js
// math.js
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }

// main.js
import { add as sum, subtract as diff } from './math.js';
console.log(sum(5, 3));  // 8
console.log(diff(5, 3)); // 2
```

**Expected Output:**
```
8
2
```

**Why This Output:** The `as` keyword renames imports and exports, avoiding naming conflicts.

### Real-World Cases

- **Utility libraries:** Exporting multiple helper functions as named exports.
- **React components:** Using default exports for component files.
- **Configuration:** Exporting a default configuration object.
- **Constants:** Exporting named constants for shared values.

---

## Module Imports

### Definitions

**Core Definition:** Module imports are declarations that bring exported bindings from another module into the current module's scope, using the `import` keyword.

**Technical Definition:** Per MDN, "Use the `import` statement, followed by a comma-separated list of the features you want to import wrapped in curly braces, followed by the keyword `from`, followed by the module specifier". The static `import` declaration is used to import read-only live bindings which are exported by another module. The imported bindings are called live bindings because they are updated by the module that exported the binding, but cannot be re-assigned by the importing module.

**Beginner-Friendly Explanation:** Imports are how you get things from other modules. If a module exports a function, you can import it and use it in your own code. It's like borrowing a tool from a friend: you can use it, but you can't change the friend's tool.

### Purposes

- To access functionality from other modules.
- To declare dependencies explicitly.
- To bring in specific named exports or the default export.
- To rename imports to avoid naming conflicts.
- To import an entire module as a namespace object.

### Syntax Rules and Structure

```js
// Named imports:
import { name1, name2 } from './module.js';

// Default import:
import defaultExport from './module.js';

// Mixed import:
import defaultExport, { named1, named2 } from './module.js';

// Namespace import:
import * as namespace from './module.js';

// Renaming:
import { original as alias } from './module.js';

// Side-effect only import:
import './styles.css';
```

**Component Breakdown:**
- `import { ... }`: Named imports, enclosed in curly braces.
- `import defaultName`: Default import, no curly braces.
- `import * as ns`: Namespace import, importing everything as an object.
- `import './module.js'`: Side-effect-only import, executes the module without importing anything.

**Syntax Rules:**
- Imports are hoisted to the top of the module.
- Import statements must be at the top level of the module.
- Imported bindings are read-only (const-like).
- Imports are live bindings; if the exporting module changes the value, the imported binding reflects the change.

**Constraints and Limitations:**
- Import names must match the exported names (unless using `as` renaming).
- Default imports can be named arbitrarily.
- Circular imports can cause incomplete module evaluation.

### Annotated Code Examples

#### Example 1: Various Import Forms

```js
// module.js
export default function main() { return "main"; }
export const helper = "helper";
export function utility() { return "utility"; }

// main.js
import main, { helper, utility } from './module.js';
import * as everything from './module.js';

console.log(main());           // "main"
console.log(helper);           // "helper"
console.log(utility());        // "utility"
console.log(everything.helper); // "helper"
```

**Expected Output:**
```
main
helper
utility
helper
```

**Why This Output:** The default import (`main`) and named imports (`helper`, `utility`) are imported directly. The namespace import (`everything`) collects all exports into an object.

#### Example 2: Live Bindings

```js
// counter.js
export let count = 0;
export function increment() { count++; }

// main.js
import { count, increment } from './counter.js';
console.log(count); // 0
increment();
console.log(count); // 1
```

**Expected Output:**
```
0
1
```

**Why This Output:** The imported `count` is a live binding. When `increment()` modifies the exported `count` variable, the imported binding reflects the updated value.

### Real-World Cases

- **Framework components:** Importing components, hooks, and utilities.
- **Configuration:** Importing settings and constants.
- **Styles:** Side-effect imports for CSS files.
- **Dynamic imports:** `import()` for lazy loading.

---

## Module Resolution

### Definitions

**Core Definition:** Module resolution is the process by which a runtime or bundler determines the exact file path or URL for a module specifier used in an `import` statement.

**Technical Definition:** Per Node.js documentation, "The algorithm to load an ES module specifier is given through the `ESM_RESOLVE` method. It returns the resolved URL for a module specifier relative to a parentURL". Module resolution in ES modules differs from CommonJS: ES modules use URL-based resolution, require exact file extensions, do not support folder mains (like `index.js`), and resolve bare specifiers through `node_modules`.

**Beginner-Friendly Explanation:** Module resolution is how JavaScript figures out which file you mean when you write `import './utils.js'`. It's like looking up an address: the specifier is the address, and resolution is the process of finding the exact location.

### Purposes

- To translate human-readable module specifiers into exact file paths or URLs.
- To support relative, absolute, and bare (package) specifiers.
- To enable consistent module loading across different environments.
- To support both local files and installed packages.

### Syntax Rules and Structure

```js
// Relative specifier:
import { fn } from './utils.js';
import { fn } from '../lib/utils.js';

// Bare specifier (package):
import { fn } from 'lodash';

// URL specifier (in browsers):
import { fn } from 'https://example.com/module.js';

// Dynamic import:
const module = await import('./dynamic.js');
```

**Component Breakdown:**
- **Relative specifiers:** Start with `./` or `../`, resolved relative to the importing module.
- **Bare specifiers:** Package names resolved through `node_modules`.
- **Absolute specifiers:** Start with `/` (server root) in browsers.
- **URL specifiers:** Full URLs for remote modules.

**Syntax Rules:**
- ES modules require exact file extensions (`.js`, `.mjs`, etc.) in relative imports.
- No default extensions; `import './utils'` will fail in Node.js ESM.
- No folder mains; `import './utils'` will not automatically load `./utils/index.js`.
- Bare specifiers are resolved through `node_modules` and `package.json` exports.

**Constraints and Limitations:**
- Browser resolution requires CORS headers for cross-origin modules.
- Node.js ESM resolution is stricter than CommonJS resolution.
- Bundlers may have their own resolution algorithms (e.g., webpack's `resolve.extensions`).

### Annotated Code Examples

#### Example 1: Relative vs Bare Specifiers

```js
// Relative — resolved from the current module's location:
import { helper } from './helper.js';       // Same directory
import { config } from '../config/app.js';  // Parent directory

// Bare — resolved from node_modules:
import lodash from 'lodash';
import { z } from 'zod';
```

**Expected Output:** Both forms resolve to the correct modules, but relative specifiers are resolved from the current file's location, while bare specifiers are resolved from `node_modules`.

**Why This Output:** The resolution algorithm first checks if the specifier is relative or absolute (resolved from the parent URL), then checks if it's a bare specifier (resolved through `node_modules`).

#### Example 2: Dynamic Import

```js
// Conditional loading:
if (user.isAdmin) {
  const { adminPanel } = await import('./admin-panel.js');
  adminPanel.render();
}
```

**Expected Output:** The `admin-panel.js` module is loaded only when the condition is true.

**Why This Output:** Dynamic `import()` returns a Promise that resolves to the module namespace object. It allows lazy loading and conditional module loading.

### Real-World Cases

- **Node.js packages:** Resolving bare specifiers like `express` or `lodash`.
- **Browser applications:** Resolving relative paths to local modules.
- **CDN imports:** Using URL specifiers to load modules from CDNs.
- **Code splitting:** Dynamic imports for lazy loading routes and components.

---

## Module Caching

### Definitions

**Core Definition:** Module caching is the mechanism by which a module's evaluated result is stored after its first import, so that subsequent imports return the same cached instance rather than re-evaluating the module.

**Technical Definition:** Per the ES module specification, "ES modules are cached the first time they are imported, and subsequent import statements return the same object". This means that module-level state is shared across all importers, and the module's code executes only once. The same instance is used across all imports because the module is instantiated on its first import and then cached. ES modules are singletons by default — when you import a module, it is executed only once, and the exported value is cached by the JavaScript engine.

**Beginner-Friendly Explanation:** Module caching means that if you import the same module in multiple places, you get the same instance every time. It's like sharing a single copy of a book: everyone reads the same book, and if someone writes notes in it, everyone sees those notes.

### Purposes

- To prevent redundant module evaluation and improve performance.
- To ensure that module-level state is shared across all importers.
- To enable the singleton pattern without explicit implementation.
- To maintain consistent behaviour across multiple imports.

### Syntax Rules and Structure

Module caching is automatic; there is no syntax for it. It is a runtime behaviour:

```js
// config.js
console.log("Config module loaded");
export const config = { apiUrl: "https://api.example.com" };

// app.js
import { config } from './config.js'; // Logs: "Config module loaded"
import { config as config2 } from './config.js'; // Does NOT log again
console.log(config === config2); // true
```

**Component Breakdown:**
- The first import evaluates the module and caches it.
- Subsequent imports return the cached module namespace object.
- Module-level state is shared across all importers.

**Syntax Rules:**
- Modules are cached by their resolved URL.
- The cache is global to the runtime (per process in Node.js, per page in browsers).
- Query parameters can create separate cache entries (e.g., `./module.js?v=2`).

**Constraints and Limitations:**
- Cached modules cannot be re-evaluated without clearing the cache or using query parameters.
- Stale module state can cause bugs if the module was meant to be re-initialised.
- The cache is not exposed to JavaScript; there is no standard way to clear it.

### Annotated Code Examples

#### Example 1: Module Singleton Behaviour

```js
// store.js
let count = 0;
export function increment() { return ++count; }
export function getCount() { return count; }

// a.js
import { increment } from './store.js';
increment();
increment();
console.log("From a.js:", getCount()); // 2

// b.js
import { getCount } from './store.js';
console.log("From b.js:", getCount()); // 2 (same instance)
```

**Expected Output:**
```
From a.js: 2
From b.js: 2
```

**Why This Output:** Both `a.js` and `b.js` import the same cached instance of `store.js`. The `count` variable is shared, so changes made in `a.js` are visible in `b.js`.

#### Example 2: Module Execution Once

```js
// logger.js
console.log("Logger initialised");
export function log(msg) { console.log(msg); }

// module1.js
import { log } from './logger.js';
log("From module1");

// module2.js
import { log } from './logger.js';
log("From module2");

// main.js
import './module1.js';
import './module2.js';
```

**Expected Output:**
```
Logger initialised
From module1
From module2
```

**Why This Output:** The `logger.js` module is evaluated only once, even though it is imported by both `module1.js` and `module2.js`. The "Logger initialised" message appears only once.

### Real-World Cases

- **Shared state:** A module that holds application state shared across components.
- **Configuration:** A module that loads configuration once and shares it.
- **Database connections:** A module that creates a connection pool once.
- **Caching services:** A module that provides a shared cache instance.

---

## Circular Dependencies

### Definitions

**Core Definition:** A circular dependency occurs when two or more modules import each other, directly or indirectly, forming a cycle in the module dependency graph.

**Technical Definition:** Per the Microsoft Cloudpack guidance, "Circular dependencies (A imports B, B imports A) between files are legal in TypeScript and ES modules but introduce non-determinism at bundle time, which can result in `undefined` exports or order-sensitive bugs". ES modules use a module map and module records to handle cycles: when a module is being evaluated, it is marked as "evaluating", so circular references do not cause infinite loops. However, circular dependencies can cause one module to receive an incomplete export object, leading to `undefined` values and runtime errors that are difficult to debug.

**Beginner-Friendly Explanation:** A circular dependency is like two friends who each need something from the other before they can do anything. Module A needs something from Module B, and Module B needs something from Module A. This can work sometimes, but it often causes problems because one module might not be ready when the other tries to use it.

### Purposes

- To understand why circular dependencies are problematic.
- To recognise circular dependencies in a codebase.
- To learn strategies for avoiding and fixing circular dependencies.
- To use tooling to detect cycles in the dependency graph.

### Syntax Rules and Structure

Circular dependencies are not a syntax feature; they are a structural issue:

```js
// a.js
import { bFn } from './b.js';
export const aValue = 1;
export function aFn() { return bFn() + aValue; }

// b.js
import { aValue } from './a.js'; // Circular import
export function bFn() { return aValue; }
```

**Component Breakdown:**
- **Cycle:** `a.js` imports `b.js`, and `b.js` imports `a.js`.
- **Partial evaluation:** When `b.js` is evaluated, `a.js` may not have finished evaluating, so `aValue` may be `undefined`.

**Syntax Rules:**
- Circular dependencies are allowed by the specification but are dangerous.
- ES modules handle cycles by marking modules as "evaluating" during evaluation.
- The order of evaluation affects which exports are available when.

**Constraints and Limitations:**
- `undefined` exports are the most common symptom.
- Order-sensitive bugs are difficult to reproduce and fix.
- Some bundlers have edge cases with cycles that degrade tree shaking or cause execution order problems.

### Annotated Code Examples

#### Example 1: Circular Dependency Causing `undefined`

```js
// a.js
import { b } from './b.js';
export const a = "A";
console.log("a.js: b =", b); // b is undefined

// b.js
import { a } from './a.js';
export const b = "B";
console.log("b.js: a =", a); // a is "A" (if a.js evaluated first)

// main.js
import './a.js';
```

**Expected Output (depends on evaluation order):**
```
b.js: a = A
a.js: b = undefined
```

**Why This Output:** When `a.js` is evaluated first, it imports `b.js`. `b.js` is evaluated, imports `a.js` (which is still evaluating), and gets the current value of `a` (which may be `undefined` if `a` hasn't been initialised yet). The result depends on the exact evaluation order.

#### Example 2: Refactoring to Avoid Circular Dependencies

```js
// shared.js — extract shared values into a leaf module
export const aValue = 1;

// a.js
import { aValue } from './shared.js';
import { bFn } from './b.js';
export function aFn() { return bFn() + aValue; }

// b.js
import { aValue } from './shared.js';
export function bFn() { return aValue; }
```

**Expected Output:** No more circular dependency; `aValue` is available to both modules without a cycle.

**Why This Output:** By extracting the shared value into a separate `shared.js` module, the circular dependency is broken.

### Real-World Cases

- **Barrel files:** `index.ts` that re-exports siblings and is imported back by those siblings.
- **Cross-layer leakage:** UI layer importing data layer which imports UI types.
- **Large utility modules:** Re-exported widely, creating cycles.
- **Detection:** Use dependency graph tooling like `madge` to fail on new cycles in CI.

---

## References

- MDN Web Docs – JavaScript modules — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Scripts and Modules — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- Node.js Documentation – Modules: ECMAScript modules — https://nodejs.org/api/esm.html
- TypeScript Handbook – Modules — https://www.typescriptlang.org/docs/handbook/2/modules.html
- freeCodeCamp – JavaScript Modules: How to Create, Import, and Export a Module in JS — https://www.freecodecamp.org/news/javascript-modules/
- Microsoft Cloudpack – Repo guidance: Avoid circular dependencies — https://microsoft.github.io/cloudpack/repo-guidance/circular-dependencies
- webpack Documentation – ECMAScript Modules — https://webpack.js.org/api/module-methods/
- Node.js Documentation – Module System — https://mintlify.wiki/nodejs/node/concepts/modules
- ESLint – strict rule — https://eslint.org/docs/latest/rules/strict
- ES Modules in Depth (Fireship) — https://fireship.dev
- Google JavaScript Style Guide – Named vs Default Exports — https://google.github.io/styleguide/jsguide.html
- Node.js Documentation – ESM Resolution Algorithm — https://nodejs.org/api/esm.html#resolution-algorithm