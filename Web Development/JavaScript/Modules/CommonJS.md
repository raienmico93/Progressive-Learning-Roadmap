# CommonJS — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** CommonJS is the original module system for Node.js. It provides a synchronous, file-based mechanism for splitting JavaScript code into reusable modules using `require()` to import and `module.exports` to export functionality.

**Technical Definition:** In Node.js, each file is treated as a separate module. CommonJS modules are wrapped in a function by Node.js, giving each module its own private scope. The `module` object represents the current module, and `module.exports` is the value returned when another module requires it. The `exports` variable is a reference to the initial `module.exports` object, provided as a convenience for adding properties. Modules are cached after their first load, so subsequent `require()` calls return the same object.

**Beginner-Friendly Explanation:** CommonJS is the "classic" way Node.js handles modules. Each JavaScript file is its own little world — variables inside are private unless you explicitly share them. You use `require()` to bring in code from other files and `module.exports` to share your own code. It's like each file is a shop: `module.exports` is the counter where you put things for others to take, and `require()` is how you go to another shop and pick things up.

### Key Characteristics

- **Synchronous loading:** `require()` loads modules synchronously — the module is fully evaluated before `require()` returns.
- **Module scope:** Each file is wrapped in a function, so top-level variables are private to that module.
- **Single value export:** `module.exports` can be assigned any value — an object, function, class, or primitive.
- **Caching:** Modules are cached after first load; repeated `require()` calls return the same instance.
- **Circular dependency support:** Node.js handles circular dependencies by returning partial exports.
- **Extension resolution:** `require()` can resolve modules without explicit file extensions and can load directories via `index.js`.
- **`exports` alias:** The `exports` variable is a shorthand for `module.exports`, but only works for property assignment, not reassignment.
- **Not the only system:** Node.js also supports ECMAScript Modules (ESM), and the two can interoperate.

### Prerequisites

- Basic understanding of JavaScript functions, objects, and scope.
- Familiarity with Node.js and how to run scripts (`node file.js`).
- Awareness of file paths and the `node_modules` directory.
- Understanding of Promises and `async/await` (helpful for ESM comparison).

### Related Programming Areas

- **Node.js development:** CommonJS is the default module system for Node.js.
- **Package distribution:** npm packages historically used CommonJS.
- **Build tools:** Webpack, Browserify, and others bundle CommonJS modules for browsers.
- **Migration to ESM:** Many projects are migrating from CommonJS to ES Modules.
- **Interoperability:** Understanding CommonJS is essential for working with legacy code and npm packages.

### Core Concepts / Features

---

## `require()`

### Definitions

**Core Definition:** `require()` is the function used in CommonJS modules to load and import other modules.

**Technical Definition:** When `require()` is called with a module specifier, Node.js resolves the specifier to a file path, loads the file, wraps it in a function, executes it, and returns the value of `module.exports` from that module. The resolution process follows a specific algorithm that checks core modules, relative paths, `node_modules` directories, and file extensions.

**Beginner-Friendly Explanation:** `require()` is how you bring code from another file into your current file. You give it a path or package name, and it gives you back whatever that module exported.

### Purposes

- To load and use code from other files or installed packages.
- To declare dependencies explicitly at the top of a module.
- To access Node.js built-in modules (e.g., `fs`, `http`, `path`).
- To import third-party packages from `node_modules`.

### Syntax Rules and Structure

```js
const moduleName = require('./module-path');
const { namedExport } = require('./module-path');
const defaultExport = require('./module-path');
```

**Component Breakdown:**
- `require('./module-path')`: The function call with a module specifier.
- The return value is the `module.exports` of the target module.

**Syntax Rules:**
- `require()` can be called anywhere in the code (not just at the top).
- It can be used inside conditionals and functions.
- The module specifier can be a relative path (`./`, `../`), a core module name (`fs`), or a package name (`lodash`).
- File extensions (`.js`, `.json`, `.node`) are optional; Node.js tries them in order.

**Constraints and Limitations:**
- `require()` is synchronous and blocking.
- It cannot be used in ES modules without `createRequire()`.
- It does not support loading ESM modules with top-level await.

### Annotated Code Examples

#### Example 1: Requiring Local Modules

```js
// math.js
const add = (a, b) => a + b;
const subtract = (a, b) => a - b;

module.exports = { add, subtract };

// app.js
const math = require('./math');
console.log(math.add(2, 3));      // 5
console.log(math.subtract(5, 3)); // 2
```

**Expected Output:**
```
5
2
```

**Why This Output:** `math.js` exports an object containing `add` and `subtract`. `app.js` requires it and accesses the functions as properties of the returned object.

#### Example 2: Requiring Core and Third-Party Modules

```js
// Core module:
const fs = require('fs');
console.log(typeof fs.readFile); // "function"

// Third-party module:
const express = require('express');
const app = express();
```

**Expected Output:**
```
function
```

**Why This Output:** Core modules are resolved by name. Third-party modules are resolved from `node_modules`.

### Real-World Cases

- **Web servers:** `const http = require('http')` to create an HTTP server.
- **File operations:** `const fs = require('fs')` for reading and writing files.
- **Utilities:** `const path = require('path')` for path manipulation.
- **Third-party libraries:** `const express = require('express')` for web frameworks.

---

## `module.exports`

### Definitions

**Core Definition:** `module.exports` is the property on the `module` object that defines what a CommonJS module exports to other modules.

**Technical Definition:** Each module in Node.js has a `module` object with an `exports` property. By default, `module.exports` is an empty object. When another module calls `require()` on this module, it receives the current value of `module.exports`. You can assign a new value (a function, class, object, or primitive) to replace the default object, or you can add properties to it.

**Beginner-Friendly Explanation:** `module.exports` is like the "shopping bag" you hand to other modules. Whatever you put in it is what they get when they `require()` your module. You can fill it with an object, a single function, or anything else.

### Purposes

- To define what a module makes available to other modules.
- To export a single main value (function, class, or object).
- To export multiple values as properties of an object.
- To control the public API of a module.

### Syntax Rules and Structure

```js
// Exporting a single value:
module.exports = function() { ... };
module.exports = class { ... };
module.exports = { key: value };

// Adding properties:
module.exports.name = "value";
module.exports.fn = function() { ... };
```

**Component Breakdown:**
- `module.exports = value`: Replaces the entire export object with `value`.
- `module.exports.property = value`: Adds a property to the existing export object.

**Syntax Rules:**
- Assigning to `module.exports` replaces the entire export.
- Adding properties to `module.exports` augments the existing export.
- The value of `module.exports` is what `require()` returns.

**Constraints and Limitations:**
- Once `module.exports` is reassigned, any previously added properties are lost.
- Mixing reassignment and property addition can lead to confusion.

### Annotated Code Examples

#### Example 1: Exporting an Object

```js
// user.js
const user = {
  name: "Alice",
  greet() { return `Hello, I'm ${this.name}`; },
};

module.exports = user;

// app.js
const user = require('./user');
console.log(user.name);    // "Alice"
console.log(user.greet()); // "Hello, I'm Alice"
```

**Expected Output:**
```
Alice
Hello, I'm Alice
```

**Why This Output:** `module.exports` is assigned the `user` object. `require()` returns that object.

#### Example 2: Exporting a Function

```js
// logger.js
module.exports = function log(message) {
  console.log(`[LOG] ${message}`);
};

// app.js
const log = require('./logger');
log("Hello"); // "[LOG] Hello"
```

**Expected Output:**
```
[LOG] Hello
```

**Why This Output:** `module.exports` is assigned a function. `require()` returns that function directly.

### Real-World Cases

- **Single-function modules:** Exporting a single function (e.g., a utility).
- **Class modules:** Exporting a class for instantiation.
- **Object modules:** Exporting an object with multiple methods and properties.

---

## `exports`

### Definitions

**Core Definition:** `exports` is a convenience variable that initially references the same object as `module.exports`, provided to make it easier to add properties to a module's exports.

**Technical Definition:** Node.js initialises a variable called `exports` that points to the same object as `module.exports`. You can add properties to this object, and they will be included in the module's exports. However, if you reassign `exports` to a new value, it breaks the reference to `module.exports`, and the new value will not be exported.

**Beginner-Friendly Explanation:** `exports` is a shortcut for `module.exports`. You can use it to add things to your module's export object. But if you try to replace it entirely (like `exports = something`), it won't work — you have to use `module.exports` for that.

### Purposes

- To provide a shorter syntax for adding properties to a module's exports.
- To avoid repeatedly typing `module.exports`.
- To export multiple named values conveniently.

### Syntax Rules and Structure

```js
// Adding properties (works):
exports.name = "value";
exports.fn = function() { ... };

// Reassigning (does NOT work):
exports = { name: "value" }; // This does nothing!
```

**Component Breakdown:**
- `exports.property = value`: Adds a property to the shared export object.
- `exports = value`: Reassigns the local `exports` variable, breaking the reference.

**Syntax Rules:**
- Use `exports.property` for adding properties.
- Use `module.exports = value` for replacing the entire export.
- Never reassign `exports` directly.

**Constraints and Limitations:**
- `exports` is just a reference to the original `module.exports` object.
- Reassigning `exports` does not affect `module.exports`.

### Annotated Code Examples

#### Example 1: Adding Properties with `exports`

```js
// math.js
exports.add = (a, b) => a + b;
exports.subtract = (a, b) => a - b;

// app.js
const math = require('./math');
console.log(math.add(2, 3));      // 5
console.log(math.subtract(5, 3)); // 2
```

**Expected Output:**
```
5
2
```

**Why This Output:** Properties added to `exports` are added to `module.exports` because they reference the same object.

#### Example 2: The Reassignment Trap

```js
// broken.js
exports = { name: "Alice" }; // This does NOT work!

// app.js
const broken = require('./broken');
console.log(broken); // {} (empty object)
```

**Expected Output:**
```
{}
```

**Why This Output:** Reassigning `exports` breaks the reference to `module.exports`. The new object is local to the module and is not exported.

### Real-World Cases

- **Utility modules:** Exporting multiple helper functions using `exports.fn = ...`.
- **Barrel files:** Aggregating exports from multiple modules.
- **Simple modules:** Quick exports without needing `module.exports` each time.

---

## CommonJS Modules

### Definitions

**Core Definition:** A CommonJS module is a JavaScript file that uses `require()` to import dependencies and `module.exports` (or `exports`) to export functionality.

**Technical Definition:** In Node.js, every file is treated as a separate module. The module's code is wrapped in a function (the "module wrapper") that provides `require`, `module`, `exports`, `__filename`, and `__dirname` as parameters. This wrapping creates a private scope for each module, preventing variable leakage into the global scope.

**Beginner-Friendly Explanation:** A CommonJS module is just a regular JavaScript file in Node.js. It automatically gets its own private scope, and it can use `require()` to bring in other modules and `module.exports` to share its own code.

### Purposes

- To encapsulate code in a private scope.
- To explicitly declare dependencies via `require()`.
- To control the public API via `module.exports`.
- To enable code reuse across files and projects.

### Syntax Rules and Structure

```js
// Module file:
const dependency = require('./dependency');
const privateVar = "private";
function privateFn() { ... }

module.exports = { publicFn, publicVar };
```

**Component Breakdown:**
- The module wrapper function provides `require`, `module`, `exports`, `__filename`, `__dirname`.
- Top-level variables are private to the module.
- `module.exports` defines the public API.

**Syntax Rules:**
- Every file is a module; there is no need to declare it.
- Module scope is private; global variables are not automatically accessible.
- The module wrapper function is `function(exports, require, module, __filename, __dirname) { ... }`.

**Constraints and Limitations:**
- CommonJS modules are synchronous; they cannot use top-level `await`.
- Module scope is separate from the global scope.

### Annotated Code Examples

#### Example 1: Module Wrapper in Action

```js
// module.js
console.log(__filename);   // Full path to this file
console.log(__dirname);    // Directory of this file
console.log(typeof require); // "function"
console.log(typeof module);  // "object"
console.log(typeof exports); // "object"
```

**Expected Output:**
```
/path/to/module.js
/path/to
function
object
object
```

**Why This Output:** Node.js wraps the module code in a function that provides these variables.

#### Example 2: Private Scope

```js
// counter.js
let count = 0; // Private

exports.increment = () => ++count;
exports.getCount = () => count;

// app.js
const counter = require('./counter');
counter.increment();
counter.increment();
console.log(counter.getCount()); // 2
console.log(counter.count);      // undefined (private)
```

**Expected Output:**
```
2
undefined
```

**Why This Output:** `count` is private to the module. Only the exported functions can access it.

### Real-World Cases

- **Node.js applications:** All files are CommonJS modules by default.
- **npm packages:** Many packages are published as CommonJS.
- **Legacy codebases:** CommonJS is the traditional Node.js module system.

---

## CommonJS Caching

### Definitions

**Core Definition:** CommonJS caching is the mechanism by which a module's evaluated result is stored after its first `require()` call, so that subsequent `require()` calls return the same cached instance without re-executing the module.

**Technical Definition:** Per Node.js documentation, "Modules are cached after the first time they are loaded. This means (among other things) that every call to `require('foo')` will get exactly the same object returned, if it would resolve to the same file". The cache is stored in `require.cache`, keyed by the resolved filename. Modules are cached based on their resolved filename, so if a module resolves to a different filename (e.g., from different `node_modules` locations), it will be loaded again.

**Beginner-Friendly Explanation:** CommonJS caching means that if you `require()` the same module multiple times, you get the same object every time. The module's code runs only once. It's like sharing a single copy of a book — everyone reads the same book, and if someone writes notes in it, everyone sees those notes.

### Purposes

- To prevent redundant module evaluation and improve performance.
- To ensure module-level state is shared across all importers.
- To enable the singleton pattern without explicit implementation.
- To reduce memory usage by sharing module instances.

### Syntax Rules and Structure

Caching is automatic. The cache can be accessed via `require.cache`:

```js
// Access the cache:
console.log(Object.keys(require.cache));

// Clear a module from the cache:
delete require.cache[require.resolve('./module.js')];

// Then require it again to get a fresh instance:
const freshModule = require('./module.js');
```

**Component Breakdown:**
- `require.cache`: An object mapping resolved filenames to module objects.
- `require.resolve('./module.js')`: Returns the resolved filename.
- `delete require.cache[filename]`: Removes the module from the cache.

**Syntax Rules:**
- Modules are cached by their resolved filename.
- The cache is global to the Node.js process.
- Clearing the cache forces re-evaluation on the next `require()`.

**Constraints and Limitations:**
- Cached modules cannot be re-evaluated without clearing the cache.
- Stale module state can cause bugs if the module was meant to be re-initialised.
- The cache is not exposed to JavaScript in ESM.

### Annotated Code Examples

#### Example 1: Module Singleton Behaviour

```js
// store.js
console.log("Store module loaded");
let count = 0;
exports.increment = () => ++count;
exports.getCount = () => count;

// a.js
const store = require('./store');
store.increment();
store.increment();
console.log("From a.js:", store.getCount()); // 2

// b.js
const store = require('./store');
console.log("From b.js:", store.getCount()); // 2
```

**Expected Output:**
```
Store module loaded
From a.js: 2
From b.js: 2
```

**Why This Output:** "Store module loaded" appears only once. Both `a.js` and `b.js` share the same cached instance of `store.js`.

#### Example 2: Clearing the Cache

```js
// config.js
console.log("Config loaded");
module.exports = { version: "1.0" };

// main.js
require('./config'); // Logs "Config loaded"
delete require.cache[require.resolve('./config')];
require('./config'); // Logs "Config loaded" again
```

**Expected Output:**
```
Config loaded
Config loaded
```

**Why This Output:** Deleting the module from the cache forces Node.js to re-evaluate the module on the next `require()`.

### Real-World Cases

- **Shared state:** A module that holds application state shared across components.
- **Configuration:** A module that loads configuration once and shares it.
- **Database connections:** A module that creates a connection pool once.
- **Hot reloading:** Clearing the cache to reload modules during development.

---

## CommonJS Resolution

### Definitions

**Core Definition:** CommonJS resolution is the algorithm Node.js uses to determine the exact file path for a module specifier passed to `require()`.

**Technical Definition:** The resolution algorithm follows a specific order: (1) If the specifier is a core module, return it; (2) If it begins with `/`, treat as absolute; (3) If it begins with `./` or `../`, resolve as a file or directory; (4) If it's a bare specifier, search `node_modules` directories up the file system tree. The algorithm also attempts to load files with extensions `.js`, `.json`, and `.node`, and directories via `package.json` `main` field or `index.js`.

**Beginner-Friendly Explanation:** Module resolution is how Node.js figures out which file you mean when you write `require('./utils')`. It checks different locations and extensions until it finds the right file.

### Purposes

- To translate human-readable module specifiers into exact file paths.
- To support relative, absolute, and bare (package) specifiers.
- To enable extension-free requires and directory-based modules.
- To provide a consistent module loading experience in Node.js.

### Syntax Rules and Structure

```js
require('./utils');        // Relative path
require('../lib/helper');  // Parent directory
require('/absolute/path'); // Absolute path
require('lodash');         // Bare specifier (node_modules)
require('fs');             // Core module
```

**Component Breakdown:**
- **Core modules:** Resolved by name.
- **Relative/absolute paths:** Resolved against the current file's location.
- **Bare specifiers:** Searched in `node_modules` directories.

**Syntax Rules:**
- Extensions are optional; Node.js tries `.js`, `.json`, `.node`.
- Directories are supported if they contain `package.json` with `main` or an `index.js`.
- If no file is found, a `MODULE_NOT_FOUND` error is thrown.

**Constraints and Limitations:**
- Resolution is synchronous and blocking.
- The `node_modules` search can be slow for deeply nested dependencies.
- Symlinks can affect resolution; Node.js uses `fs.realpath()` to normalise paths.

### Annotated Code Examples

#### Example 1: Extension Resolution

```js
// Directory structure:
// /app
//   /utils
//     index.js
//   app.js

// app.js
const utils = require('./utils'); // Resolves to ./utils/index.js
```

**Expected Output:** The `utils` directory's `index.js` is loaded.

**Why This Output:** `require()` checks for `utils.js`, then `utils/index.js` (directory resolution).

#### Example 2: `node_modules` Resolution

```js
// /app
//   node_modules
//     lodash
//       index.js
//   app.js

// app.js
const lodash = require('lodash'); // Resolves to node_modules/lodash/index.js
```

**Expected Output:** The `lodash` package is loaded from `node_modules`.

**Why This Output:** Bare specifiers are resolved by searching `node_modules` directories.

### Real-World Cases

- **Package imports:** Resolving `express`, `lodash`, and other npm packages.
- **Local modules:** Resolving `./utils`, `../lib`, etc.
- **Configuration files:** Resolving `./config.json`.
- **Native addons:** Resolving `.node` files.

---

## CommonJS vs ES Modules

### Definitions

**Core Definition:** CommonJS and ES Modules (ESM) are two distinct module systems in JavaScript. CommonJS is the original Node.js module system, while ESM is the standardised JavaScript module system.

**Technical Definition:** CommonJS uses `require()` and `module.exports` with synchronous, runtime-resolved loading. ESM uses `import` and `export` with asynchronous, statically-analysed loading. ESM supports top-level `await`, tree shaking, and live bindings. CommonJS is the default in Node.js for `.js` files without `"type": "module"`; ESM is the default for `.mjs` files and packages with `"type": "module"`.

**Beginner-Friendly Explanation:** CommonJS is the old way (`require`/`module.exports`); ESM is the new standard way (`import`/`export`). ESM is what browsers use and what modern JavaScript projects prefer, but CommonJS is still widely used in Node.js and npm packages.

### Purposes

- To understand the differences between the two module systems.
- To choose the right system for a project.
- To handle interoperability between CommonJS and ESM.
- To plan migration from CommonJS to ESM.

### Key Differences

| Feature | CommonJS | ES Modules |
|---------|----------|------------|
| **Syntax** | `require()` / `module.exports` | `import` / `export` |
| **Loading** | Synchronous | Asynchronous |
| **Resolution** | Runtime | Parse time (static) |
| **Tree shaking** | No | Yes |
| **Top-level await** | No | Yes |
| **Live bindings** | No (copies) | Yes |
| **File extensions** | Optional | Required |
| **Folder mains** | Supported (`index.js`) | Not supported |
| **`this` at top level** | `module.exports` | `undefined` |
| **Default in Node.js** | `.js` without `"type": "module"` | `.mjs`, or `"type": "module"` |

### Annotated Code Examples

#### Example 1: Same Module in Both Systems

```js
// CommonJS:
// math.js
exports.add = (a, b) => a + b;

// app.js
const { add } = require('./math');

// ES Modules:
// math.mjs
export const add = (a, b) => a + b;

// app.mjs
import { add } from './math.mjs';
```

**Expected Output:** Both export and import the `add` function, but with different syntax.

**Why This Output:** CommonJS and ESM achieve the same goal with different syntax and semantics.

#### Example 2: Top-Level Await (ESM Only)

```js
// CommonJS: This does NOT work.
// const data = await fetch('/api/data'); // SyntaxError

// ESM: This works.
// const data = await fetch('/api/data');
```

**Expected Output:** Top-level await works only in ESM.

**Why This Output:** CommonJS does not support top-level await.

### Real-World Cases

- **New projects:** Most new Node.js projects use ESM.
- **Legacy code:** Existing Node.js code often uses CommonJS.
- **npm packages:** Many packages still publish CommonJS, some publish both.
- **Bundlers:** Webpack and Rollup support both systems.

---

## Interoperability

### Definitions

**Core Definition:** Interoperability is the ability of CommonJS and ES Modules to work together — importing each other and sharing code across module systems.

**Technical Definition:** Node.js supports interoperability between CommonJS and ESM with specific rules. ESM can import CommonJS modules using `import`, receiving the `module.exports` value as the default export. CommonJS can load ESM using dynamic `import()`, but not with `require()` (unless the ESM is synchronous and Node.js version supports it). Since Node.js v20.19.0/v22.12.0, `require()` can load ESM if it does not use top-level await.

**Beginner-Friendly Explanation:** Interoperability means that even though CommonJS and ESM are different systems, they can still work together. ESM can import CommonJS modules, and in newer Node.js versions, CommonJS can even load ESM in some cases.

### Purposes

- To use CommonJS packages in ESM projects.
- To use ESM packages in CommonJS projects (via dynamic import).
- To migrate gradually from CommonJS to ESM.
- To publish packages that work with both systems.

### Syntax Rules and Structure

```js
// ESM importing CommonJS:
import cjsModule from './commonjs-module.cjs';
// cjsModule is the value of module.exports

// CommonJS importing ESM (dynamic import):
const esmModule = await import('./esm-module.mjs');
// esmModule.default is the default export

// CommonJS importing ESM (require, newer Node.js):
const esmModule = require('./esm-module.mjs'); // Only if no top-level await
```

**Component Breakdown:**
- **ESM importing CJS:** The `module.exports` value becomes the default export.
- **CJS importing ESM:** Use dynamic `import()` which returns a Promise.
- **CJS `require()` of ESM:** Supported in newer Node.js versions if the ESM is synchronous.

**Syntax Rules:**
- ESM can import CJS without special configuration.
- CJS must use dynamic `import()` for ESM (or `require()` in newer versions).
- Named exports from CJS may be detectable by static analysis in ESM imports.

**Constraints and Limitations:**
- Named exports from CommonJS are not always detectable by ESM's static analysis.
- CommonJS cannot use top-level await.
- Interoperability rules differ between Node.js versions.

### Annotated Code Examples

#### Example 1: ESM Importing CommonJS

```js
// commonjs-module.cjs
module.exports = {
  greet: (name) => `Hello, ${name}`,
  version: "1.0",
};

// esm-module.mjs
import cjs from './commonjs-module.cjs';
console.log(cjs.greet("Alice")); // "Hello, Alice"
console.log(cjs.version);        // "1.0"
```

**Expected Output:**
```
Hello, Alice
1.0
```

**Why This Output:** The ESM default import receives the `module.exports` object from the CommonJS module.

#### Example 2: CommonJS Importing ESM via Dynamic Import

```js
// esm-module.mjs
export const greet = (name) => `Hello, ${name}`;
export const version = "1.0";

// commonjs-module.cjs
async function loadEsm() {
  const esm = await import('./esm-module.mjs');
  console.log(esm.greet("Bob")); // "Hello, Bob"
  console.log(esm.version);      // "1.0"
}
loadEsm();
```

**Expected Output:**
```
Hello, Bob
1.0
```

**Why This Output:** `import()` returns a Promise that resolves to the ESM namespace object. Named exports are accessed as properties.

### Real-World Cases

- **Migrating projects:** Gradually converting CommonJS files to ESM while keeping dependencies working.
- **Publishing dual packages:** Packages that provide both CommonJS and ESM entry points.
- **Using ESM-only packages:** CommonJS projects can use dynamic `import()` to load ESM-only packages.
- **Using CommonJS packages:** ESM projects can import CommonJS packages via the default import.

---

## References

- Node.js Documentation – Modules: CommonJS modules — https://nodejs.org/api/modules.html
- Node.js Documentation – CommonJS modules (Chromium mirror) — https://chromium.googlesource.com/v8/node/+/e78742012d03eef1b89ab19bb8048b95a09d6a5a/doc/api/modules.md
- Node.js Documentation – All together (Resolution Algorithm) — https://beta.docs.nodejs.org/modules/all-together
- Node.js Documentation – Module resolution and loading — https://nodejs.org/api/packages.html
- Node.js Documentation – Modules: ECMAScript modules — https://nodejs.org/api/esm.html
- TypeScript Documentation – ESM/CJS Interoperability — https://www.typescriptlang.org/docs/handbook/modules/appendices/esm-cjs-interop.html
- Node.js Package Configuration Guide – CommonJS and ESM — https://nodejs.github.io/package-json-guide/
- Scaler Topics – Node.js Require vs Import — https://www.scaler.com/topics/nodejs/require-vs-import-nodejs/
- Builder.io – Node.js module.exports vs. exports: The Right Choice — https://site.builder.io/blog/nodejs-module-exports-vs-exports
- Node.js Documentation – Modules: CommonJS (Chinese mirror) — https://url.nodejs.cn/api/packages/5.html
- Node.js Documentation – Caching — https://nodejs.org/api/modules.html#modules_caching
- Stack Overflow – module.exports vs exports — https://stackoverflow.com/questions/7137397/module-exports-vs-exports-in-node-js
- Stack Overflow – CommonJS module cache — https://stackoverflow.com/questions/8887310/nodejs-require-cache