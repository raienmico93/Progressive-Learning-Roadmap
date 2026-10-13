# ES Modules — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** ES Modules (ECMAScript Modules) are the official standardised module system for JavaScript, introduced in ES2015 (ES6). They provide a syntax for dividing code into separate files that can import and export functionality between each other, using the `import` and `export` keywords.

**Technical Definition:** Per the ECMAScript specification, a module is a distinct source text that is parsed and evaluated as a `Module` record. Any file containing a top-level `import` or `export` declaration is considered a module; a file without any top-level import or export declarations is treated as a script whose contents are available in the global scope【10†L18-L19】. ES modules use a static structure, meaning imports and exports are determined at parse time, not runtime. This enables compile-time optimisation, including tree shaking and dead code elimination. Modules are executed within their own scope, not in the global scope, and are always in strict mode【2†L33-L36】. Modules are cached after their first evaluation; subsequent imports return the same module instance【8†L8-L8】.

**Beginner-Friendly Explanation:** ES Modules are the modern, standard way to split your JavaScript code into separate files. Instead of putting everything in one giant file, you create smaller files (modules) that each do one thing. You can then share code between them: a module can `export` things it wants to share, and other modules can `import` what they need. It's like organising a toolbox into separate drawers, each labelled with what's inside.

### Key Characteristics

- **Static structure:** Imports and exports are determined at parse time, enabling static analysis and tree shaking.
- **Module scope:** Modules have their own scope; variables and functions are private by default unless explicitly exported【2†L33-L36】.
- **Strict mode by default:** ES modules always run in strict mode; `"use strict"` is unnecessary.
- **Single evaluation:** A module is evaluated only once, regardless of how many times it is imported.
- **Caching:** Module instances are cached by URL; all importers share the same instance【8†L8-L8】.
- **Live bindings:** Imported bindings are live; when the exporting module changes a value, the change is visible to importers.
- **Deferred execution:** Module scripts are deferred by default in browsers; they execute after the document is parsed【2†L78-L78】.
- **Top-level await:** ES modules support `await` at the top level, outside any function.
- **Dynamic imports:** The `import()` function allows loading modules on demand at runtime.

### Prerequisites

- Basic understanding of JavaScript functions, variables, and scope.
- Familiarity with HTML `<script>` tags and how to include JavaScript in web pages.
- Awareness of file paths and URLs.
- Knowledge of Promises and `async/await` (for dynamic imports and top-level await).

### Related Programming Areas

- **Code organisation:** Breaking large codebases into manageable, reusable pieces.
- **Dependency management:** Modules declare and resolve their dependencies.
- **Bundling and build tools:** Webpack, Rollup, and Vite use module graphs for optimisation.
- **Tree shaking:** Eliminating unused code by analysing module exports.
- **Dynamic imports:** Lazy-loading modules on demand for performance.
- **Node.js module systems:** ES Modules (ESM) vs CommonJS (CJS).

### Core Concepts / Features

---

## `import`

### Definitions

**Core Definition:** The `import` statement is used to bring exported bindings from another module into the current module's scope.

**Technical Definition:** Per MDN, "The static `import` declaration is used to import read-only live bindings which are exported by another module"【2†L32-L32】. Imported bindings are called "live bindings" because they are updated by the module that exported the binding, but cannot be re-assigned by the importing module. The `import` statement can only be used in modules (files with a top-level `export` or `import`) and must be at the top level of the module.

**Beginner-Friendly Explanation:** `import` is how you get things from another module. If a module exports a function or variable, you use `import` to bring it into your own file so you can use it. It's like borrowing a tool from a friend's toolbox.

### Purposes

- To access functionality from other modules.
- To declare dependencies explicitly at the top of a module.
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
- In browsers, the module specifier must include the file extension (`.js`).

**Constraints and Limitations:**
- Import names must match the exported names (unless using `as` renaming).
- Default imports can be named arbitrarily.
- Static imports cannot be used conditionally; use dynamic `import()` for conditional loading.

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

## `export`

### Definitions

**Core Definition:** The `export` statement is used to make declarations in a module available to other modules.

**Technical Definition:** Per the ECMAScript specification, an `export` declaration is used to export values from a module. There are two types of exports: **named exports** and **default exports**. Named exports allow multiple values to be exported from a module; default exports represent a single main value. The `export` keyword can be applied to declarations or used as a standalone statement.

**Beginner-Friendly Explanation:** `export` is how you share things from your module. It's like putting items on a shelf for others to take. You can put many items on the shelf (named exports) or have one main item (default export).

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

### Real-World Cases

- **Utility libraries:** Exporting multiple helper functions as named exports.
- **React components:** Using default exports for component files.
- **Configuration:** Exporting a default configuration object.
- **Constants:** Exporting named constants for shared values.

---

## Named Exports

### Definitions

**Core Definition:** Named exports allow a module to export multiple values by name, which can then be imported by other modules using the same names.

**Technical Definition:** Per the freeCodeCamp guide, "Named exports allow you to export multiple values from a module. Each value is exported with a specific name, and importers must use that exact name (or an alias) to import it"【4†L67-L67】. Named exports are the preferred approach for modules that expose multiple related functions, constants, or classes.

**Beginner-Friendly Explanation:** Named exports are like labelling each item in your toolbox. When someone wants a specific tool, they ask for it by name. You can have many named exports in one module.

### Purposes

- To export multiple values from a single module.
- To provide a clear, explicit API with named functions and constants.
- To enable tree shaking by allowing bundlers to identify unused exports.
- To support IDE autocomplete and static analysis.

### Syntax Rules and Structure

```js
// Declaration form:
export const name = "value";
export function fn() { ... }
export class MyClass { ... }

// List form:
const a = 1, b = 2;
export { a, b };

// Renaming:
export { a as alpha, b as beta };
```

**Component Breakdown:**
- `export const/function/class`: Exports a declaration directly.
- `export { ... }`: Exports an explicit list of bindings.
- `as`: Renames the export.

**Syntax Rules:**
- Export names must be valid identifiers.
- A module can have multiple named exports.
- Importers must use the exact export name or an alias.

**Constraints and Limitations:**
- Named exports cannot be imported without curly braces.
- Exporting a value that is re-assigned later requires a live binding.

### Annotated Code Examples

#### Example 1: Multiple Named Exports

```js
// math.js
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export class Vector { constructor(x, y) { this.x = x; this.y = y; } }

// main.js
import { PI, add, Vector } from './math.js';
console.log(PI);              // 3.14159
console.log(add(2, 3));       // 5
const v = new Vector(1, 2);
console.log(v.x, v.y);        // 1 2
```

**Expected Output:**
```
3.14159
5
1 2
```

**Why This Output:** Each named export is imported by its exact name.

### Real-World Cases

- **Utility modules:** Exporting multiple helper functions.
- **Constants:** Exporting shared constants.
- **Types:** Exporting multiple classes or interfaces.

---

## Default Exports

### Definitions

**Core Definition:** A default export is a single value that a module exports as its "main" export, which can be imported without curly braces and given any name.

**Technical Definition:** Per MDN, "Default exports are used to export a single value from a module. There can only be one default export per module. When importing a default export, you can use any name you like"【2†L49-L49】. Default exports are often used for React components and other modules that have a primary export.

**Beginner-Friendly Explanation:** A default export is the "main thing" a module exports. If a module is a toolbox, the default export is the most important tool. You can give it any name when you import it.

### Purposes

- To export a single main value from a module.
- To simplify imports when a module has one primary export.
- To support frameworks like React that use default exports for components.

### Syntax Rules and Structure

```js
// Declaration form:
export default function() { ... }
export default class { ... }
export default expression;

// Named function:
export default function myFunc() { ... }
```

**Component Breakdown:**
- `export default`: Marks the value as the default export.
- The default export can be a function, class, or any expression.

**Syntax Rules:**
- A module can have at most one default export.
- Default exports are imported without curly braces.
- The importer can choose any name for the default import.

**Constraints and Limitations:**
- Default exports can make refactoring harder because the name is not enforced.
- Tree shaking is less effective with default exports.
- Some style guides discourage default exports.

### Annotated Code Examples

#### Example 1: Default Function Export

```js
// greet.js
export default function(name) {
  return `Hello, ${name}!`;
}

// main.js
import greet from './greet.js';
console.log(greet("Alice")); // "Hello, Alice!"
```

**Expected Output:**
```
Hello, Alice!
```

**Why This Output:** The default export is imported without curly braces and named `greet` by the importer.

#### Example 2: Default Class Export

```js
// User.js
export default class User {
  constructor(name) { this.name = name; }
  greet() { return `Hello, I'm ${this.name}`; }
}

// main.js
import User from './User.js';
const user = new User("Bob");
console.log(user.greet()); // "Hello, I'm Bob"
```

**Expected Output:**
```
Hello, I'm Bob
```

**Why This Output:** The default export is a class, imported and instantiated.

### Real-World Cases

- **React components:** `export default function Button() { ... }`.
- **Main module exports:** A library's primary export.
- **Configuration:** A default configuration object.

---

## Re-exports

### Definitions

**Core Definition:** Re-exports allow a module to export bindings from another module without importing them into its own scope first.

**Technical Definition:** Per MDN, "Re-exports allow you to export a binding from another module without having to import it first. This is useful for creating barrel files or aggregating exports from multiple modules"【2†L61-L61】. The syntax is `export { name } from './module.js'` or `export * from './module.js'`.

**Beginner-Friendly Explanation:** Re-exports let you pass through exports from one module to another. It's like a relay race: the baton (export) is passed from one module to another without stopping.

### Purposes

- To create barrel files (index.js) that aggregate exports from multiple modules.
- To provide a single entry point for a library or feature.
- To simplify imports for consumers by exposing a unified API.
- To avoid importing and re-exporting manually.

### Syntax Rules and Structure

```js
// Re-export specific names:
export { name1, name2 } from './module.js';

// Re-export with renaming:
export { name as alias } from './module.js';

// Re-export all (excluding default):
export * from './module.js';

// Re-export default:
export { default } from './module.js';
export { default as ModuleName } from './module.js';

// Re-export all as namespace:
export * as ns from './module.js';
```

**Component Breakdown:**
- `export { ... } from './module.js'`: Re-exports specific named exports.
- `export * from './module.js'`: Re-exports all named exports.
- `export { default } from './module.js'`: Re-exports the default export as a named export.

**Syntax Rules:**
- Re-exports do not create local bindings in the current module.
- `export *` does not re-export the default export.
- Re-exports can be renamed using `as`.

**Constraints and Limitations:**
- `export *` can cause naming conflicts if multiple modules export the same name.
- Re-exports add a layer of indirection that can complicate debugging.

### Annotated Code Examples

#### Example 1: Barrel File

```js
// utils/string.js
export function capitalize(s) { ... }
export function trim(s) { ... }

// utils/number.js
export function round(n) { ... }
export function format(n) { ... }

// utils/index.js (barrel file)
export { capitalize, trim } from './string.js';
export { round, format } from './number.js';

// main.js
import { capitalize, round } from './utils/index.js';
```

**Expected Output:** The consumer can import from a single entry point instead of individual files.

**Why This Output:** The barrel file re-exports all utilities, providing a clean API.

#### Example 2: Re-export All

```js
// index.js
export * from './math.js';
export * from './string.js';

// main.js
import { add, capitalize } from './index.js';
```

**Expected Output:** All named exports from both modules are available.

**Why This Output:** `export *` re-exports all named exports from each module.

### Real-World Cases

- **Library entry points:** `index.js` that re-exports from multiple source files.
- **Feature modules:** Aggregating exports from a feature's submodules.
- **Component libraries:** Exposing all components through a single entry point.

---

## Renaming Exports

### Definitions

**Core Definition:** Renaming exports allows a module to export a binding under a different name than its local identifier.

**Technical Definition:** Per MDN, "You can rename an export using the `as` keyword: `export { localName as exportedName }`. This is useful when the local name is not suitable for the public API or when avoiding naming conflicts"【2†L55-L56】.

**Beginner-Friendly Explanation:** Renaming exports lets you give a more public-friendly name to a local variable or function. It's like having a nickname: your friends know you by your nickname, but your ID has your real name.

### Purposes

- To provide a public name that differs from the local implementation name.
- To avoid naming conflicts when re-exporting from multiple modules.
- To expose a cleaner, more semantic API.
- To adapt internal names to external conventions.

### Syntax Rules and Structure

```js
// Export with renaming:
export { localName as publicName };

// Inline renaming:
export const publicName = value;
```

**Component Breakdown:**
- `export { localName as publicName }`: Exports `localName` under the name `publicName`.
- `as`: The renaming keyword.

**Syntax Rules:**
- The exported name must be a valid identifier.
- The local name can be any valid identifier.
- Renaming can be applied to both declarations and re-exports.

**Constraints and Limitations:**
- The renamed export is what importers use; the local name is not accessible.
- Renaming does not change the local binding's name.

### Annotated Code Examples

#### Example 1: Basic Renaming

```js
// internal.js
const _privateHelper = () => "helper";
export { _privateHelper as publicHelper };

// main.js
import { publicHelper } from './internal.js';
console.log(publicHelper()); // "helper"
```

**Expected Output:**
```
helper
```

**Why This Output:** The local `_privateHelper` is exported as `publicHelper`, providing a cleaner public name.

#### Example 2: Renaming in Re-exports

```js
// math.js
export function add(a, b) { return a + b; }

// index.js
export { add as sum } from './math.js';

// main.js
import { sum } from './index.js';
console.log(sum(2, 3)); // 5
```

**Expected Output:**
```
5
```

**Why This Output:** The `add` function is re-exported as `sum`, providing a more descriptive name.

### Real-World Cases

- **Library APIs:** Exposing internal functions under public names.
- **Conflict resolution:** Renaming conflicting exports when aggregating modules.
- **Naming conventions:** Adapting internal naming to external conventions.

---

## Renaming Imports

### Definitions

**Core Definition:** Renaming imports allows a module to import a binding under a different name than the one used by the exporting module.

**Technical Definition:** Per MDN, "You can rename an import using the `as` keyword: `import { originalName as alias } from './module.js'`. This is useful when the original name conflicts with a local name or when a more descriptive name is desired"【2†L42-L42】.

**Beginner-Friendly Explanation:** Renaming imports lets you call an imported value by a different name in your module. It's like giving a friend a nickname that's easier for you to remember.

### Purposes

- To avoid naming conflicts with local variables or other imports.
- To provide a more descriptive or convenient name.
- To adapt imported names to local conventions.
- To import multiple modules that export the same name.

### Syntax Rules and Structure

```js
import { originalName as alias } from './module.js';
```

**Component Breakdown:**
- `originalName`: The name of the export in the source module.
- `alias`: The local name to use in the importing module.

**Syntax Rules:**
- The alias must be a valid identifier.
- The alias is local to the importing module.
- Multiple imports can be renamed in the same statement.

**Constraints and Limitations:**
- The original name is not accessible after renaming.
- Renaming does not affect the exporting module.

### Annotated Code Examples

#### Example 1: Avoiding Naming Conflicts

```js
// math.js
export function add(a, b) { return a + b; }

// main.js
import { add as mathAdd } from './math.js';

function add(a, b) { return a + b + 1; } // Local function

console.log(mathAdd(2, 3)); // 5
console.log(add(2, 3));     // 6
```

**Expected Output:**
```
5
6
```

**Why This Output:** The imported `add` is renamed to `mathAdd` to avoid conflict with the local `add` function.

#### Example 2: Renaming for Clarity

```js
// api.js
export function get(url) { ... }

// main.js
import { get as httpGet } from './api.js';
```

**Expected Output:** The imported `get` is renamed to `httpGet` for clarity.

**Why This Output:** The alias provides a more descriptive name that clarifies the function's purpose.

### Real-World Cases

- **Conflict avoidance:** Renaming imports when multiple modules export the same name.
- **Clarity:** Using more descriptive names for imported functions.
- **Convention:** Adapting imported names to local naming conventions.

---

## Namespace Imports

### Definitions

**Core Definition:** A namespace import brings all of a module's exports into the current scope as properties of a single object.

**Technical Definition:** Per MDN, "A namespace import is used to import all exports from a module as an object. The syntax is `import * as name from './module.js'`. The `name` object contains all named exports as properties"【2†L45-L45】.

**Beginner-Friendly Explanation:** A namespace import is like importing the entire toolbox as one object. Instead of importing individual tools, you get a box with all the tools inside, and you access them by name.

### Purposes

- To import all exports from a module under a single namespace.
- To avoid naming conflicts by accessing exports as properties.
- To provide a clear, organised way to access many exports.
- To dynamically access exports by name.

### Syntax Rules and Structure

```js
import * as namespace from './module.js';

// Access exports:
namespace.exportName;
```

**Component Breakdown:**
- `* as namespace`: Imports all exports as properties of `namespace`.
- `namespace.exportName`: Accesses a specific export.

**Syntax Rules:**
- The namespace object contains all named exports.
- The default export is available as `namespace.default`.
- The namespace object is sealed; new properties cannot be added.

**Constraints and Limitations:**
- The namespace object is read-only.
- Tree shaking may be less effective with namespace imports.
- The namespace object includes all exports, even if only some are used.

### Annotated Code Examples

#### Example 1: Basic Namespace Import

```js
// math.js
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }

// main.js
import * as math from './math.js';
console.log(math.PI);          // 3.14159
console.log(math.add(2, 3));   // 5
console.log(math.subtract(5, 3)); // 2
```

**Expected Output:**
```
3.14159
5
2
```

**Why This Output:** All exports are available as properties of the `math` namespace object.

#### Example 2: Dynamic Access with Namespace

```js
import * as utils from './utils.js';

const operation = "add";
const result = utils[operation](2, 3);
console.log(result); // 5
```

**Expected Output:**
```
5
```

**Why This Output:** The namespace object allows dynamic access to exports by string name.

### Real-World Cases

- **Utility libraries:** Importing all utilities as a namespace.
- **Dynamic dispatch:** Accessing exports by string name.
- **Organisation:** Grouping related exports under a namespace.

---

## Dynamic Imports

### Definitions

**Core Definition:** Dynamic imports allow modules to be loaded on demand at runtime using the `import()` function, which returns a Promise.

**Technical Definition:** Per MDN, "Dynamic import allows you to load modules on demand. The `import()` function returns a Promise that resolves to the module namespace object"【9†L18-L19】. Dynamic imports are useful for code splitting, lazy loading, and conditional module loading.

**Beginner-Friendly Explanation:** Dynamic imports let you load a module only when you need it, instead of loading everything upfront. It's like ordering a pizza when you're hungry instead of buying all the ingredients in advance.

### Purposes

- To load modules on demand for better performance.
- To implement code splitting and lazy loading.
- To conditionally load modules based on runtime conditions.
- To reduce initial bundle size.
- To load modules dynamically based on user interaction.

### Syntax Rules and Structure

```js
// Basic dynamic import:
const module = await import('./module.js');

// Conditional:
if (condition) {
  const { fn } = await import('./module.js');
  fn();
}

// With promise:
import('./module.js')
  .then(module => module.default())
  .catch(err => console.error(err));
```

**Component Breakdown:**
- `import('./module.js')`: Returns a Promise that resolves to the module namespace object.
- `await`: Waits for the module to load.
- Destructuring: Accesses specific exports.

**Syntax Rules:**
- `import()` can be used anywhere in the code (not just top level).
- It returns a Promise.
- The module specifier can be a dynamic expression.

**Constraints and Limitations:**
- Dynamic imports are asynchronous; you must handle the Promise.
- Bundlers may need special configuration for dynamic imports.
- The module path must be resolvable at build time (or runtime in browsers).

### Annotated Code Examples

#### Example 1: Lazy Loading a Module

```js
// Step 1: Define a function that loads a module on demand.
async function loadChart() {
  const { Chart } = await import('./chart.js');
  return new Chart();
}

// Step 2: Call when needed.
document.getElementById('showChart').addEventListener('click', async () => {
  const chart = await loadChart();
  chart.render();
});
```

**Expected Output:** The chart module is loaded only when the button is clicked.

**Why This Output:** `import()` loads the module on demand, reducing the initial bundle size.

#### Example 2: Conditional Import

```js
async function getFormatter(locale) {
  if (locale === 'en') {
    return (await import('./formatters/en.js')).default;
  } else if (locale === 'fr') {
    return (await import('./formatters/fr.js')).default;
  }
  return (await import('./formatters/default.js')).default;
}
```

**Expected Output:** The appropriate formatter module is loaded based on the locale.

**Why This Output:** Only the needed module is loaded, optimising performance.

### Real-World Cases

- **Route-based code splitting:** Loading route modules on demand.
- **Feature flags:** Loading modules based on feature availability.
- **Heavy libraries:** Loading charts, editors, or maps only when needed.
- **Internationalisation:** Loading locale-specific modules.

---

## `import()`

### Definitions

**Core Definition:** `import()` is the function form of dynamic import that returns a Promise resolving to a module namespace object.

**Technical Definition:** Per MDN, "The `import()` function is a function-like expression that allows loading an ECMAScript module asynchronously and dynamically into a potentially non-module environment"【9†L16-L17】. Unlike the static `import` declaration, `import()` can be used anywhere in the code and returns a Promise.

**Beginner-Friendly Explanation:** `import()` is the function you call to load a module dynamically. It returns a Promise that gives you the module when it's ready.

### Purposes

- To load modules dynamically at runtime.
- To conditionally load modules based on runtime conditions.
- To implement lazy loading and code splitting.
- To load modules in non-module scripts.

### Syntax Rules and Structure

```js
const module = await import('./module.js');
// or
import('./module.js').then(module => { ... });
```

**Component Breakdown:**
- `import()`: The dynamic import function.
- `'./module.js'`: The module specifier.
- The return value is a Promise that resolves to the module namespace object.

**Syntax Rules:**
- `import()` can be used in both modules and scripts.
- The specifier can be a dynamic expression.
- The Promise resolves to the module namespace object.

**Constraints and Limitations:**
- The module specifier must be resolvable.
- Dynamic imports are asynchronous.
- Bundlers may need configuration to handle dynamic imports.

### Annotated Code Examples

#### Example 1: Basic `import()`

```js
// Step 1: Call import() with a static specifier.
import('./math.js')
  .then(math => {
    console.log(math.add(2, 3)); // 5
  })
  .catch(err => {
    console.error('Failed to load module:', err);
  });
```

**Expected Output:**
```
5
```

**Why This Output:** `import()` returns a Promise that resolves to the module namespace object.

#### Example 2: Dynamic Specifier

```js
async function loadModule(name) {
  const module = await import(`./modules/${name}.js`);
  return module.default;
}

const logger = await loadModule('logger');
logger("Hello");
```

**Expected Output:** The specified module is loaded dynamically.

**Why This Output:** The specifier is constructed dynamically, allowing flexible module loading.

### Real-World Cases

- **Plugin systems:** Loading plugins dynamically.
- **Themes:** Loading theme modules on demand.
- **Feature modules:** Loading features based on user permissions.

---

## Top-Level Await

### Definitions

**Core Definition:** Top-level await allows the `await` keyword to be used at the top level of an ES module, outside any async function.

**Technical Definition:** Per the TC39 proposal, "Top-level await enables modules to act as big async functions: With top-level await, ECMAScript Modules (ESM) can await resources, causing other modules who import them to wait before they start evaluating their body"【10†L12-L13】. Top-level await is Stage 4 (standardised) and works in ES modules.

**Beginner-Friendly Explanation:** Top-level await lets you use `await` directly in a module's top-level code, without wrapping it in an async function. It's like having a module that waits for something before it finishes loading.

### Purposes

- To await resources at module initialisation time.
- To eliminate the need for IIAFEs (Immediately Invoked Async Function Expressions).
- To enable dynamic imports and conditional module loading.
- To simplify module initialisation code.

### Syntax Rules and Structure

```js
// In an ES module:
const data = await fetch('/api/data');
export const result = await data.json();
```

**Component Breakdown:**
- `await` at the top level of a module.
- Imports of this module will wait until the await resolves.

**Syntax Rules:**
- Only works in ES modules (`.mjs` or `"type": "module"` in `package.json`).
- Works in `<script type="module">` in browsers.
- Not allowed in CommonJS or traditional scripts.

**Constraints and Limitations:**
- Not supported in CommonJS.
- A module that never resolves will cause the importing module to hang.
- Top-level await can introduce deadlocks if modules have circular dependencies.

### Annotated Code Examples

#### Example 1: Basic Top-Level Await

```js
// data.mjs
const response = await fetch('/api/data');
const data = await response.json();
export default data;

// main.mjs
import data from './data.mjs';
console.log(data);
```

**Expected Output:** The fetched data is logged after the module loads.

**Why This Output:** `data.mjs` awaits the fetch before exporting. `main.mjs` waits for `data.mjs` to finish loading.

#### Example 2: Dynamic Import with Top-Level Await

```js
// module.mjs
const moduleName = 'utils';
const { default: utils } = await import(`./${moduleName}.mjs`);
export default utils;
```

**Expected Output:** The dynamically imported module is available.

**Why This Output:** Top-level await allows dynamic imports to be resolved before the module's exports are used.

### Real-World Cases

- **Configuration loading:** Fetching config before exporting it.
- **Database connections:** Establishing a connection at module load time.
- **Conditional imports:** Loading different modules based on environment.

---

## Module Scripts

### Definitions

**Core Definition:** Module scripts are `<script>` elements with `type="module"`, which tell the browser to treat the script as an ES module.

**Technical Definition:** Per MDN, "The `type="module"` attribute in the `<script>` tag tells the browser that the script is a module. Module scripts are deferred by default, execute in strict mode, and have their own scope"【2†L78-L78】. Module scripts can use `import` and `export` statements and support top-level await.

**Beginner-Friendly Explanation:** A module script is just a `<script>` tag with `type="module"`. It tells the browser to treat your JavaScript as a module, enabling imports, exports, and other module features.

### Purposes

- To load ES modules in the browser.
- To enable module features like `import` and `export`.
- To defer script execution until after the document is parsed.
- To execute scripts in strict mode automatically.
- To load modules with CORS support.

### Syntax Rules and Structure

```html
<script type="module" src="./main.js"></script>

<script type="module">
  import { fn } from './module.js';
  fn();
</script>
```

**Component Breakdown:**
- `type="module"`: The attribute that marks the script as a module.
- `src`: The module file to load.
- Inline module scripts are also supported.

**Syntax Rules:**
- Module scripts are deferred by default.
- Module scripts execute in strict mode.
- Module scripts are fetched with CORS.
- Module scripts have their own scope (not global).

**Constraints and Limitations:**
- Not supported in older browsers (IE11 and earlier).
- Module scripts must be served with the correct MIME type.
- Local file:// URLs may not work with modules due to CORS restrictions.

### Annotated Code Examples

#### Example 1: Basic Module Script

```html
<!DOCTYPE html>
<html>
<head>
  <title>ES Modules</title>
</head>
<body>
  <script type="module" src="./main.js"></script>
</body>
</html>
```

**main.js:**
```js
import { greet } from './utils.js';
greet('World');
```

**Expected Output:** The module loads and executes after the document is parsed.

**Why This Output:** The `type="module"` attribute marks the script as a module, enabling imports and deferred execution.

#### Example 2: Inline Module Script

```html
<script type="module">
  import { add } from './math.js';
  console.log(add(2, 3)); // 5
</script>
```

**Expected Output:**
```
5
```

**Why This Output:** Inline module scripts can use imports and exports.

### Real-World Cases

- **Modern web applications:** Using `type="module"` for all scripts.
- **Progressive enhancement:** Loading modules in supported browsers.
- **Development:** Using native modules without a bundler.

---

## `type="module"`

### Definitions

**Core Definition:** `type="module"` is the HTML attribute that tells the browser to treat a `<script>` element as an ES module.

**Technical Definition:** Per MDN, "The `type` attribute of the `<script>` element indicates the type of script. Setting `type="module"` makes the script an ES module, enabling `import` and `export` statements"【2†L78-L78】.

**Beginner-Friendly Explanation:** `type="module"` is how you tell the browser "this is a module, not a regular script." It unlocks all the module features.

### Purposes

- To enable ES module features in the browser.
- To defer script execution.
- To execute scripts in strict mode.
- To scope scripts to their own module scope.

### Syntax Rules and Structure

```html
<script type="module" src="./main.js"></script>
```

**Component Breakdown:**
- `type="module"`: The attribute value that marks the script as a module.

**Syntax Rules:**
- Module scripts are deferred by default.
- Module scripts execute in strict mode.
- Module scripts have their own scope.

**Constraints and Limitations:**
- Not supported in IE11.
- Requires CORS for cross-origin loading.

### Annotated Code Examples

#### Example 1: Module vs Regular Script

```html
<!-- Regular script: global scope, not deferred -->
<script src="./regular.js"></script>

<!-- Module script: module scope, deferred, strict mode -->
<script type="module" src="./module.js"></script>
```

**Expected Output:** The regular script runs immediately; the module script runs after the document is parsed.

**Why This Output:** The `type="module"` attribute changes the script's behaviour.

### Real-World Cases

- **All modern web applications:** Using `type="module"` for ES modules.
- **Native ESM:** Loading modules without a bundler.

---

## Module Resolution

### Definitions

**Core Definition:** Module resolution is the process by which a runtime or bundler determines the exact file path or URL for a module specifier used in an `import` statement.

**Technical Definition:** Per Node.js documentation, "The algorithm to load an ES module specifier is given through the `ESM_RESOLVE` method. It returns the resolved URL for a module specifier relative to a parentURL"【8†L13-L13】. ES modules use URL-based resolution, require exact file extensions, and do not support folder mains (like `index.js`).

**Beginner-Friendly Explanation:** Module resolution is how JavaScript figures out which file you mean when you write `import './utils.js'`. It's like looking up an address.

### Purposes

- To translate human-readable module specifiers into exact file paths or URLs.
- To support relative, absolute, and bare (package) specifiers.
- To enable consistent module loading across different environments.

### Syntax Rules and Structure

```js
// Relative specifier:
import { fn } from './utils.js';
import { fn } from '../lib/utils.js';

// Bare specifier (package):
import { fn } from 'lodash';

// URL specifier (in browsers):
import { fn } from 'https://example.com/module.js';
```

**Component Breakdown:**
- **Relative specifiers:** Start with `./` or `../`, resolved relative to the importing module.
- **Bare specifiers:** Package names resolved through `node_modules`.
- **Absolute specifiers:** Start with `/` (server root) in browsers.
- **URL specifiers:** Full URLs for remote modules.

**Syntax Rules:**
- ES modules require exact file extensions (`.js`, `.mjs`, etc.).
- No default extensions; `import './utils'` will fail in Node.js ESM.
- No folder mains; `import './utils'` will not automatically load `./utils/index.js`.

**Constraints and Limitations:**
- Browser resolution requires CORS headers for cross-origin modules.
- Node.js ESM resolution is stricter than CommonJS resolution.
- Bundlers may have their own resolution algorithms.

### Annotated Code Examples

#### Example 1: Relative vs Bare Specifiers

```js
// Relative — resolved from the current module's location:
import { helper } from './helper.js';
import { config } from '../config/app.js';

// Bare — resolved from node_modules:
import lodash from 'lodash';
import { z } from 'zod';
```

**Expected Output:** Both forms resolve to the correct modules.

**Why This Output:** Relative specifiers are resolved from the current file's location; bare specifiers are resolved through `node_modules`.

#### Example 2: Dynamic Import

```js
if (user.isAdmin) {
  const { adminPanel } = await import('./admin-panel.js');
  adminPanel.render();
}
```

**Expected Output:** The `admin-panel.js` module is loaded only when the condition is true.

**Why This Output:** Dynamic `import()` returns a Promise that resolves to the module namespace object.

### Real-World Cases

- **Node.js packages:** Resolving bare specifiers like `express` or `lodash`.
- **Browser applications:** Resolving relative paths to local modules.
- **CDN imports:** Using URL specifiers to load modules from CDNs.
- **Code splitting:** Dynamic imports for lazy loading routes.

---

## Import Maps

### Definitions

**Core Definition:** Import maps are a browser feature that allows developers to control how module specifiers are resolved, enabling the use of bare specifiers in browsers without a bundler.

**Technical Definition:** Per MDN, "An import map is a JSON object that allows developers to control how the browser resolves module specifiers when importing JavaScript modules"【5†L9-L9】. Import maps are a useful feature for using bare module specifiers in browsers, allowing developers to write cleaner imports like `import { fn } from 'lodash'` instead of relative or absolute paths.

**Beginner-Friendly Explanation:** Import maps let you use short, clean names for modules in the browser, just like you would in Node.js. Instead of writing long paths, you can say `import { fn } from 'utils'` and the import map tells the browser where to find it.

### Purposes

- To use bare module specifiers in browsers.
- To map module names to URLs.
- To simplify imports by avoiding long relative paths.
- To support dependency management without a bundler.
- To enable scoped module resolution.

### Syntax Rules and Structure

```html
<script type="importmap">
{
  "imports": {
    "lodash": "https://cdn.jsdelivr.net/npm/lodash-es/lodash.js",
    "utils/": "/js/utils/",
    "react": "https://esm.sh/react@18"
  }
}
</script>
```

**Component Breakdown:**
- `"imports"`: The main mapping object.
- Keys: Module specifiers (bare names, prefixes ending with `/`).
- Values: URLs to the actual modules.

**Syntax Rules:**
- Import maps must be declared before any module scripts.
- There can only be one import map per document (in most browsers).
- The `imports` object maps specifiers to URLs.
- Prefix mappings (ending with `/`) map a path prefix.

**Constraints and Limitations:**
- Browser support is limited to modern browsers (Chrome 89+, Firefox 108+, Safari 16.4+).
- Only one import map is allowed per document (though multiple are being considered).
- Import maps must be inline (cannot be loaded from external URLs).

### Annotated Code Examples

#### Example 1: Basic Import Map

```html
<script type="importmap">
{
  "imports": {
    "lodash": "https://cdn.jsdelivr.net/npm/lodash-es/lodash.js"
  }
}
</script>

<script type="module">
  import { chunk } from 'lodash';
  console.log(chunk([1, 2, 3, 4], 2));
</script>
```

**Expected Output:**
```
[[1, 2], [3, 4]]
```

**Why This Output:** The import map maps `'lodash'` to the CDN URL. The module script can use the bare specifier.

#### Example 2: Prefix Mapping

```html
<script type="importmap">
{
  "imports": {
    "utils/": "/js/utils/",
    "utils/format": "/js/utils/format.js"
  }
}
</script>

<script type="module">
  import { format } from 'utils/format';
  import { parse } from 'utils/parse';
</script>
```

**Expected Output:** The `utils/format` and `utils/parse` modules are resolved using the prefix mapping.

**Why This Output:** The prefix mapping `"utils/"` maps to `/js/utils/`, so `utils/parse` resolves to `/js/utils/parse.js`.

### Real-World Cases

- **CDN-based applications:** Using import maps to load dependencies from CDNs.
- **Micro-frontends:** Mapping shared dependencies to consistent URLs.
- **Development:** Using bare specifiers without a bundler.
- **Deno:** Import maps are a core feature of Deno's module resolution.

---

## Module Best Practices

### Definitions

**Core Definition:** Module best practices are guidelines for writing clean, maintainable, and performant ES modules.

**Technical Definition:** Per the web.dev guide, "When writing ES modules, it's important to follow best practices for maintainability, performance, and correctness. Key practices include keeping modules focused, using named exports, avoiding circular dependencies, and leveraging dynamic imports for code splitting"【1†L26-L26】.

**Beginner-Friendly Explanation:** Best practices are the rules of thumb for writing good modules. Keep your modules small and focused, use named exports, avoid circular dependencies, and use dynamic imports when you need them.

### Purposes

- To write maintainable and readable modules.
- To optimise performance through code splitting and tree shaking.
- To avoid common pitfalls like circular dependencies.
- To ensure consistent module structure across a codebase.
- To improve collaboration and code quality.

### Key Practices

**Practice 1: Keep Modules Focused**

```js
// GOOD: A module that does one thing well.
export function formatDate(date) { ... }
export function parseDate(str) { ... }

// BAD: A module that tries to do everything.
export function formatDate(date) { ... }
export function sendEmail(user) { ... }
export function calculateTax(amount) { ... }
```

**Practice 2: Prefer Named Exports**

```js
// GOOD: Named exports are explicit and tree-shakeable.
export function add(a, b) { ... }
export function subtract(a, b) { ... }

// LESS GOOD: Default exports are harder to refactor.
export default function add(a, b) { ... }
```

**Practice 3: Avoid Circular Dependencies**

```js
// BAD: a.js imports b.js, b.js imports a.js.

// GOOD: Extract shared code into a third module.
// shared.js
export const sharedValue = 1;

// a.js
import { sharedValue } from './shared.js';

// b.js
import { sharedValue } from './shared.js';
```

**Practice 4: Use Dynamic Imports for Code Splitting**

```js
// Load heavy modules only when needed.
async function loadChart() {
  const { Chart } = await import('./chart.js');
  return new Chart();
}
```

**Practice 5: Use Import Maps for Browser Dependencies**

```html
<script type="importmap">
{
  "imports": {
    "lodash": "https://cdn.jsdelivr.net/npm/lodash-es/lodash.js"
  }
}
</script>
```

**Practice 6: Avoid Side Effects in Modules**

```js
// BAD: Module-level side effects.
console.log("Module loaded");
document.body.innerHTML = "Hello";

// GOOD: Export functions, let the consumer decide when to run them.
export function init() {
  document.body.innerHTML = "Hello";
}
```

**Practice 7: Use Module Scope for Encapsulation**

```js
// Private variable — not exported.
let privateState = {};

// Public API.
export function getState() { return { ...privateState }; }
export function setState(newState) { privateState = { ...newState }; }
```

**Practice 8: Follow Consistent Naming Conventions**

```js
// Use .js for modules, .mjs for ESM in Node.js (or "type": "module" in package.json).
// Name modules after their primary export or purpose.
```

### Real-World Cases

- **Large applications:** Following best practices for maintainability.
- **Libraries:** Publishing well-structured modules for consumers.
- **Team collaboration:** Consistent module structure for collaboration.

---

## References

- MDN Web Docs – JavaScript modules — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- MDN Web Docs – import — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import
- MDN Web Docs – export — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export
- MDN Web Docs – import() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import
- MDN Web Docs – Import maps — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script/type/importmap
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- Node.js Documentation – Modules: ECMAScript modules — https://nodejs.org/api/esm.html
- Node.js Documentation – Module System — https://mintlify.wiki/nodejs/node/concepts/modules
- freeCodeCamp – JavaScript Modules: How to Create, Import, and Export a Module in JS — https://www.freecodecamp.org/news/javascript-modules/
- TC39 – Top-level await proposal — https://www.proposals.es/proposals/Top-level%20await
- TC39 – Import maps proposal — https://github.com/WICG/import-maps
- web.dev – JavaScript modules — https://web.dev/learn/javascript/modules
- TypeScript Handbook – Modules — https://www.typescriptlang.org/docs/handbook/2/modules.html
- Google JavaScript Style Guide – Modules — https://google.github.io/styleguide/jsguide.html