# AMD and UMD — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** AMD (Asynchronous Module Definition) is a JavaScript module specification designed for browsers that allows modules and their dependencies to be loaded asynchronously. UMD (Universal Module Definition) is a pattern that combines AMD, CommonJS, and global variable exports into a single module format that works in any environment.

**Technical Definition:** The AMD specification, maintained by the amdjs organization, defines a mechanism for authoring asynchronously loadable modules with associated dependencies. It extends the CommonJS Modules specification by providing a global `define()` function that takes an array of dependencies (specified as module names) and a factory function containing the module body. The AMD API consists of two primary components: the `define()` function for module definition and the `require()` function for dynamic, asynchronous loading of modules and resolving module ID-based strings to file paths. UMD is not a formal specification but a pattern that detects the environment (AMD, CommonJS, or browser global) and adapts the module's export mechanism accordingly.

**Beginner-Friendly Explanation:** AMD is a way to write JavaScript modules that load asynchronously, which is important for browsers because loading files synchronously would freeze the page. You use `define()` to create a module and `require()` to load one. UMD is a "universal" wrapper that makes a module work everywhere — in browsers with AMD loaders like RequireJS, in Node.js with CommonJS, or as a plain global variable. It's like a universal adapter that fits any outlet.

### Key Characteristics

- **Asynchronous loading:** AMD modules and their dependencies load asynchronously, preventing browser blocking.
- **Browser-first design:** AMD was designed specifically for the browser environment, where synchronous loading is impractical.
- **`define()` function:** The primary API for defining modules with dependencies and a factory function.
- **`require()` function:** Used for dynamically loading modules and resolving module IDs.
- **UMD universality:** UMD modules work in AMD, CommonJS, and browser global environments.
- **Environment detection:** UMD patterns detect the available module system at runtime and use the appropriate export mechanism.
- **Legacy status:** Both AMD and UMD are considered legacy systems, largely superseded by ES Modules in modern development.

### Prerequisites

- Basic understanding of JavaScript functions, closures, and the module concept.
- Familiarity with browser script loading and the `script` tag.
- Awareness of CommonJS (`require`/`module.exports`) and ES Modules (`import`/`export`).
- Knowledge of RequireJS (the most popular AMD loader) is helpful.

### Related Programming Areas

- **Browser module loading:** AMD was the primary solution before ES Modules.
- **Build tools:** RequireJS, curl.js, and Browserify supported AMD.
- **Library distribution:** UMD was widely used by libraries to support multiple environments.
- **Legacy codebases:** Many older projects still use AMD and UMD.
- **Migration to ESM:** Understanding AMD/UMD is necessary for migrating legacy code.

### Core Concepts / Features

---

## AMD

### Definitions

**Core Definition:** AMD (Asynchronous Module Definition) is a module specification for JavaScript that allows modules and their dependencies to be loaded asynchronously, primarily designed for browser environments.

**Technical Definition:** The AMD API specifies a mechanism for defining modules such that the module and its dependencies can be asynchronously loaded. The primary building block is the `define()` function, which takes an array of dependencies and a factory function. AMD was created to address the limitations of the CommonJS module format in browsers, where synchronous loading would block page rendering. The AMD format grew out of Dojo's real-world experience with browser module loading.

**Beginner-Friendly Explanation:** AMD is a way to write JavaScript modules that don't block the browser while loading. Instead of loading all your scripts at once and hoping they load in the right order, AMD lets you declare what each module needs, and the loader fetches them asynchronously.

### Purposes

- To enable asynchronous loading of JavaScript modules in browsers.
- To declare module dependencies explicitly, allowing the loader to fetch them in parallel.
- To avoid the blocking behaviour of synchronous script loading.
- To provide a scalable alternative to manually ordering `<script>` tags.
- To support on-demand loading of modules for better performance.

### Syntax Rules and Structure

```js
// Basic AMD module definition:
define(['dependency1', 'dependency2'], function (dependency1, dependency2) {
  // Module body
  return {
    // Exported values
  };
});

// Named module:
define('myModule', ['dep1'], function (dep1) {
  return { /* exports */ };
});

// Simplified CommonJS wrapper:
define(function (require, exports, module) {
  var dep = require('dependency');
  exports.value = dep.value;
});
```

**Component Breakdown:**
- `id` (optional): The module identifier. When omitted, the module is anonymous and its ID is inferred from the file path.
- `dependencies` (optional): An array of module IDs that the module depends on.
- `factory`: The function that is executed to instantiate the module. Its parameters correspond to the resolved dependencies. It should return the module's export value.

**Syntax Rules:**
- `define()` is a global function provided by the AMD loader.
- The `id` parameter is usually omitted; anonymous modules are the norm.
- The `dependencies` array can contain relative paths (`./`, `../`), absolute paths, or module names.
- The factory function's parameters are positional; the first parameter corresponds to the first dependency, and so on.
- The factory can return any value: an object, a function, a class, or a primitive.

**Constraints and Limitations:**
- AMD requires an AMD loader (like RequireJS) to be present in the page.
- AMD is not supported natively by browsers; it requires a loader library.
- AMD modules must be loaded via the AMD loader; they cannot be loaded with a plain `<script>` tag (unless shimmed).
- AMD has largely been superseded by ES Modules.

### Annotated Code Examples

#### Example 1: Basic AMD Module Definition

```js
// Step 1: Define a module with no dependencies.
define(function () {
  // Step 2: Private variable.
  var count = 0;

  // Step 3: Public API returned as an object.
  return {
    increment: function () { return ++count; },
    getCount: function () { return count; }
  };
});

// Step 4: Define a module with dependencies.
define(['./counter'], function (counter) {
  counter.increment();
  counter.increment();
  return {
    report: function () {
      return 'Count is: ' + counter.getCount();
    }
  };
});
```

**Expected Output:** When the second module is loaded, it increments the counter twice and its `report()` method returns `"Count is: 2"`.

**Why This Output:** The first module is defined with no dependencies and returns an object with `increment` and `getCount` methods. The second module depends on `./counter`, receives the exported object as `counter`, and uses it.

#### Example 2: Using `define` with a Named Module and Dependencies

```js
// Step 1: Define a named module with dependencies.
define('mathUtils', ['dependencyA', 'dependencyB'], function (depA, depB) {
  // Step 2: Use the dependencies.
  function add(a, b) {
    return depA.calculate(a, b) + depB.constant;
  }

  // Step 3: Export the module.
  return {
    add: add,
    version: '1.0.0'
  };
});
```

**Expected Output:** The `mathUtils` module is registered with the AMD loader and can be required by other modules.

**Why This Output:** The named module `mathUtils` depends on `dependencyA` and `dependencyB`. The factory receives the resolved dependencies and returns an object with the `add` function and `version` property.

### Real-World Cases

- **Dojo Toolkit:** AMD grew out of the Dojo Toolkit's module system and was the first major implementation.
- **RequireJS:** The most popular AMD loader, used extensively in browser applications before ES Modules.
- **Backbone.js:** Backbone supports AMD via RequireJS for module definition.
- **jQuery:** jQuery 1.7+ registers itself as an AMD module when an AMD loader is present.

---

## `define()`

### Definitions

**Core Definition:** `define()` is the global function provided by AMD loaders that is used to define a module, specifying its dependencies and the factory function that produces its exports.

**Technical Definition:** The `define()` function is the primary API for the AMD specification. It can be called with different argument signatures: `define(factory)`, `define(dependencies, factory)`, or `define(id, dependencies, factory)`. The AMD specification requires that `define()` be a global function available in the AMD-compliant loader environment. The factory function is executed only after all dependencies have been resolved and loaded.

**Beginner-Friendly Explanation:** `define()` is how you tell the AMD loader "here is a module, here's what it needs, and here's what it provides." It's the entry point for creating an AMD module.

### Purposes

- To register a module with the AMD loader.
- To declare the module's dependencies so the loader can fetch them.
- To provide the factory function that produces the module's exports.
- To support named and anonymous modules.
- To enable the simplified CommonJS wrapper syntax for easier migration.

### Syntax Rules and Structure

```js
// Signature 1: Factory only (no dependencies):
define(function () {
  return { /* exports */ };
});

// Signature 2: Dependencies and factory:
define(['dep1', 'dep2'], function (dep1, dep2) {
  return { /* exports */ };
});

// Signature 3: ID, dependencies, and factory:
define('moduleId', ['dep1'], function (dep1) {
  return { /* exports */ };
});

// Simplified CommonJS wrapper:
define(function (require, exports, module) {
  var dep = require('dependency');
  exports.value = dep.value;
});
```

**Component Breakdown:**
- `id` (optional): A string identifying the module. If omitted, the module is anonymous.
- `dependencies` (optional): An array of strings, each a module ID or path.
- `factory`: A function that returns the module's export value. It receives resolved dependencies as arguments.

**Syntax Rules:**
- The `id` parameter is optional and usually omitted in favour of anonymous modules.
- The `dependencies` array is optional; if omitted, the factory has no dependency arguments.
- The factory function must return the module's export value.
- The special dependencies `"require"`, `"exports"`, and `"module"` are always available.

**Constraints and Limitations:**
- `define()` must be available globally (provided by the AMD loader).
- The factory function is called only once per module; the result is cached.
- Anonymous modules rely on the loader inferring the module ID from the file path.

### Annotated Code Examples

#### Example 1: `define()` with Dependencies

```js
// Step 1: Define a module that depends on jQuery and Underscore.
define(['jquery', 'underscore'], function ($, _) {
  // Step 2: Use the dependencies.
  var users = [
    { name: 'Alice', age: 30 },
    { name: 'Bob', age: 25 }
  ];

  // Step 3: Return the module's public API.
  return {
    getNames: function () {
      return _.pluck(users, 'name');
    },
    render: function (selector) {
      $(selector).html(users.map(function (u) {
        return '<li>' + u.name + '</li>';
      }).join(''));
    }
  };
});
```

**Expected Output:** The module is defined with `jquery` and `underscore` as dependencies. When required, it provides `getNames()` and `render()` methods.

**Why This Output:** The `define()` call declares the dependencies in the array. The loader resolves them and passes the resolved modules as `$` and `_` to the factory function. The factory returns an object with two methods.

#### Example 2: Simplified CommonJS Wrapper

```js
// Step 1: Define a module using the CommonJS-style wrapper.
define(function (require, exports, module) {
  // Step 2: Require dependencies using CommonJS syntax.
  var _ = require('underscore');

  // Step 3: Use exports to define the public API.
  exports.pluck = function (arr, key) {
    return _.pluck(arr, key);
  };

  // Step 4: Optionally use module.exports for a single export.
  // module.exports = { ... };
});
```

**Expected Output:** The module behaves like a CommonJS module but is loaded via AMD.

**Why This Output:** The simplified CommonJS wrapper is a special form of `define()` that provides `require`, `exports`, and `module` as parameters, allowing developers to use CommonJS-style code that the AMD loader can handle.

### Real-World Cases

- **RequireJS:** The primary implementation of AMD, used for defining modules in browser applications.
- **Dojo:** The original AMD implementation.
- **curl.js:** Another AMD loader that supports `define()`.
- **Library authors:** Many libraries provide AMD-compatible builds using `define()`.

---

## `require()`

### Definitions

**Core Definition:** In AMD, `require()` is the function used to load modules dynamically and asynchronously, and to resolve module IDs to file paths.

**Technical Definition:** The AMD `require()` function has two main forms: the synchronous `require(id)` form for resolving already-loaded modules, and the asynchronous `require(dependencies, callback)` form for loading modules on demand. The asynchronous form takes an array of module IDs and a callback function that is invoked with the resolved modules once they are loaded. The AMD `require()` is an API that allows dynamic, asynchronous loading of modules, and for resolving some module ID-based strings to file paths.

**Beginner-Friendly Explanation:** AMD's `require()` is how you load modules at runtime. You give it a list of module names and a callback, and it fetches those modules asynchronously, then calls your callback with the loaded modules.

### Purposes

- To dynamically load modules on demand, rather than all at once.
- To resolve module IDs to file paths.
- To support lazy loading and conditional module loading.
- To enable the simplified CommonJS wrapper syntax within `define()`.
- To provide a way to load modules after the initial page load.

### Syntax Rules and Structure

```js
// Asynchronous require (load modules and execute callback):
require(['module1', 'module2'], function (module1, module2) {
  // Use module1 and module2
});

// Synchronous require (within a define factory):
define(function (require) {
  var module = require('module');
  // Use module
});

// Configuration require:
require.config({
  baseUrl: '/js',
  paths: { 'jquery': 'lib/jquery' }
});
```

**Component Breakdown:**
- `dependencies`: An array of module IDs to load.
- `callback`: A function invoked after all dependencies are loaded. Receives the resolved modules as arguments.
- `require.config()`: Configures the AMD loader (base URL, paths, shims).

**Syntax Rules:**
- The asynchronous `require()` returns immediately; the callback is invoked later.
- The synchronous `require()` can only be used within a `define()` factory.
- `require.config()` should be called before any modules are loaded.
- The callback's parameters are positional, corresponding to the dependencies array.

**Constraints and Limitations:**
- The asynchronous `require()` does not return a value; the result is delivered via callback.
- The synchronous `require()` may throw if the module is not already loaded.
- `require.config()` must be called before any `define()` or `require()` calls.

### Annotated Code Examples

#### Example 1: Asynchronous Module Loading

```js
// Step 1: Load modules dynamically.
require(['./utils', './logger'], function (utils, logger) {
  // Step 2: Use the loaded modules.
  logger.log('Utils loaded: ' + utils.version);
  utils.doSomething();
});
```

**Expected Output:** The `utils` and `logger` modules are loaded asynchronously, and the callback logs a message and calls a utility method.

**Why This Output:** The `require()` call specifies the dependencies and a callback. The AMD loader fetches the modules, and once both are loaded, the callback is invoked with the resolved modules.

#### Example 2: Configuration and Dynamic Loading

```js
// Step 1: Configure the AMD loader.
require.config({
  baseUrl: '/js',
  paths: {
    'jquery': 'https://cdn.example.com/jquery-3.6.0.min',
    'underscore': 'lib/underscore'
  }
});

// Step 2: Load modules using the configured paths.
require(['jquery', 'underscore'], function ($, _) {
  $('body').append('<p>Loaded with AMD</p>');
});
```

**Expected Output:** The configured paths are used to resolve the module IDs. The callback appends a paragraph to the body.

**Why This Output:** `require.config()` sets the base URL and path mappings. When `require(['jquery', 'underscore'])` is called, the loader resolves the module IDs to the configured paths.

### Real-World Cases

- **Lazy loading:** Loading modules only when a user navigates to a specific section.
- **Conditional loading:** Loading different modules based on feature detection.
- **Configuration:** Setting up module paths and shims for legacy libraries.
- **Dynamic dependencies:** Loading modules whose names are determined at runtime.

---

## UMD

### Definitions

**Core Definition:** UMD (Universal Module Definition) is a pattern for writing JavaScript modules that work in multiple environments — AMD, CommonJS, and as browser global variables — using a single source file.

**Technical Definition:** UMD is a pattern that detects the available module system at runtime and uses the appropriate export mechanism. It typically checks for AMD (`define.amd`), then CommonJS (`module.exports`), and finally falls back to creating a global variable. The UMD pattern typically attempts to offer compatibility with the most popular script loaders of the day, using AMD as a base with special-casing added to handle CommonJS compatibility. It is not a formal specification but a convention adopted by library authors.

**Beginner-Friendly Explanation:** UMD is a "write once, run anywhere" pattern for JavaScript modules. It wraps a module in code that detects whether it's being used with an AMD loader, in Node.js (CommonJS), or as a plain browser script, and exports the module in the appropriate way.

### Purposes

- To create modules that work in any JavaScript environment.
- To support AMD, CommonJS, and browser globals from a single file.
- To simplify library distribution by providing one file that works everywhere.
- To bridge the gap between AMD and CommonJS ecosystems.
- To serve as a fallback when ES Modules are not available.

### Syntax Rules and Structure

```js
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD environment:
    define(['dependency'], factory);
  } else if (typeof module === 'object' && module.exports) {
    // CommonJS environment:
    module.exports = factory(require('dependency'));
  } else {
    // Browser global:
    root.MyModule = factory(root.Dependency);
  }
}(typeof self !== 'undefined' ? self : this, function (dependency) {
  // Module body
  return { /* exports */ };
}));
```

**Component Breakdown:**
- `root`: The global object (`self` in browsers, `global` in Node.js, `this` as fallback).
- `factory`: A function that contains the module's implementation and returns its exports.
- **Environment checks:** `define.amd` for AMD, `module.exports` for CommonJS, global variable fallback.

**Syntax Rules:**
- The `typeof define === 'function' && define.amd` check detects AMD loaders.
- The `typeof module === 'object' && module.exports` check detects CommonJS.
- The fallback assigns the module to a property on the global object.
- The pattern is typically wrapped in an IIFE (Immediately Invoked Function Expression).

**Constraints and Limitations:**
- UMD modules are more verbose than plain AMD or CommonJS modules.
- The pattern adds a small runtime overhead for environment detection.
- UMD is a legacy pattern; ES Modules are the modern standard.
- Some UMD variants have subtle differences in how they handle global exports.

### Annotated Code Examples

#### Example 1: Basic UMD Pattern

```js
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD
    define([], factory);
  } else if (typeof module === 'object' && module.exports) {
    // CommonJS
    module.exports = factory();
  } else {
    // Browser global
    root.MyModule = factory();
  }
}(typeof self !== 'undefined' ? self : this, function () {
  // Module implementation
  var version = '1.0.0';

  function greet(name) {
    return 'Hello, ' + name;
  }

  return {
    version: version,
    greet: greet
  };
}));
```

**Expected Output:** In AMD, the module is defined via `define()`. In CommonJS, it is exported via `module.exports`. In the browser, it creates a global `MyModule`.

**Why This Output:** The UMD wrapper detects the environment and uses the appropriate export mechanism. The factory function returns the module's exports.

#### Example 2: UMD with Dependencies

```js
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    define(['jquery', 'underscore'], factory);
  } else if (typeof module === 'object' && module.exports) {
    module.exports = factory(require('jquery'), require('underscore'));
  } else {
    root.MyPlugin = factory(root.jQuery, root._);
  }
}(typeof self !== 'undefined' ? self : this, function ($, _) {
  // Module depends on jQuery and Underscore.
  return {
    pluckNames: function (users) {
      return _.pluck(users, 'name');
    },
    render: function (selector, data) {
      $(selector).text(data.join(', '));
    }
  };
}));
```

**Expected Output:** The module works in AMD (with dependencies loaded), CommonJS (with dependencies required), and browser (with global `jQuery` and `_`).

**Why This Output:** The UMD pattern passes the dependencies through the appropriate mechanism for each environment.

### Real-World Cases

- **jQuery plugins:** Many jQuery plugins use UMD to work with AMD, CommonJS, and browser globals.
- **Lodash:** Lodash provides UMD builds that work in all environments.
- **Backbone.js:** Backbone uses UMD to support AMD, CommonJS, and browser globals.
- **Moment.js:** Moment.js historically used UMD for universal distribution.
- **Library distribution:** UMD was the standard for libraries that needed to support multiple module systems.

---

## Universal Modules

### Definitions

**Core Definition:** Universal modules are JavaScript modules that can be consumed in any environment, whether through AMD, CommonJS, or as browser global variables, typically implemented using the UMD pattern.

**Technical Definition:** The concept of "universal" modules is embodied in the UMD pattern, which is a pattern of universal module definition for JavaScript modules. These modules are capable of working everywhere, be it in the client, on the server, or elsewhere. Universal modules are not a formal specification but a design goal achieved through UMD wrappers. They provide maximum compatibility at the cost of additional boilerplate.

**Beginner-Friendly Explanation:** Universal modules are modules that "just work" no matter how you load them. You can use them with RequireJS, in Node.js, or by including them in a `<script>` tag. They're the Swiss Army knife of JavaScript modules.

### Purposes

- To distribute a single module that works in any JavaScript environment.
- To reduce the maintenance burden of maintaining separate builds for different module systems.
- To simplify the consumer experience by removing the need for environment-specific configuration.
- To support legacy environments while still being usable in modern ones.

### Syntax Rules and Structure

Universal modules follow the UMD pattern, which checks for AMD, CommonJS, and global environments:

```js
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    define([], factory);
  } else if (typeof exports === 'object') {
    module.exports = factory();
  } else {
    root.UniversalModule = factory();
  }
}(globalThis, function () {
  // Module implementation
  return { /* exports */ };
}));
```

**Component Breakdown:**
- The IIFE checks for the environment and routes to the appropriate export.
- `globalThis` is the ES2020 standard for accessing the global object.

**Syntax Rules:**
- The AMD check must come first because some CommonJS environments also define `define`.
- The CommonJS check uses `typeof exports === 'object'` or `typeof module === 'object' && module.exports`.
- The fallback assigns to the global object.

**Constraints and Limitations:**
- Universal modules are larger due to the wrapper boilerplate.
- The wrapper adds a small runtime cost.
- Universal modules are a legacy solution; ES Modules with proper configuration (or dual packages) are preferred in modern development.

### Annotated Code Examples

#### Example 1: Universal Module with Global Object Fallback

```js
// myModule.js — a universal module.
;(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD
    define([], function () { return factory(root); });
  } else if (typeof exports === 'object') {
    // CommonJS
    module.exports = factory(root);
  } else {
    // Browser global
    root.myPlugin = factory(root);
  }
})(globalThis, function (window) {
  'use strict';

  // Module implementation
  var version = '2.0.0';

  function init(options) {
    console.log('Initialising version ' + version);
  }

  return {
    version: version,
    init: init
  };
});
```

**Expected Output:** In AMD environments, the module is defined via `define()`. In CommonJS, it is exported via `module.exports`. In browsers, it creates a global `myPlugin` object.

**Why This Output:** The UMD wrapper detects the environment and exports the module appropriately. The factory function receives the global object (`window` in browsers) for access to global variables.

#### Example 2: UMD with Dependency Injection

```js
;(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    define(['jquery', 'underscore'], factory);
  } else if (typeof exports === 'object') {
    module.exports = factory(require('jquery'), require('underscore'));
  } else {
    root.MyWidget = factory(root.jQuery, root._);
  }
})(typeof self !== 'undefined' ? self : this, function ($, _) {
  // Module that depends on jQuery and Underscore.
  function Widget(options) {
    this.$el = $(options.el);
    this.data = options.data || [];
  }

  Widget.prototype.render = function () {
    var names = _.pluck(this.data, 'name');
    this.$el.html(names.join(', '));
  };

  return Widget;
});
```

**Expected Output:** The `Widget` constructor works in AMD (with jQuery and Underscore loaded as dependencies), CommonJS (with them required), and browser (with global `jQuery` and `_`).

**Why This Output:** The UMD wrapper passes the dependencies through the appropriate mechanism for each environment, and the factory returns the `Widget` constructor.

### Real-World Cases

- **jQuery plugins:** The UMD pattern is widely used for jQuery plugins to support AMD and CommonJS.
- **Lodash:** Lodash's modular builds use UMD for universal compatibility.
- **Backbone.js:** Backbone's UMD wrapper supports AMD, CommonJS, and browser globals.
- **Moment.js:** Moment.js historically distributed UMD builds.
- **D3.js:** D3 provides UMD builds for universal consumption.

---

## Legacy Module Systems

### Definitions

**Core Definition:** Legacy module systems are JavaScript module specifications and patterns that predate ES Modules, including CommonJS, AMD, UMD, and the Revealing Module Pattern.

**Technical Definition:** Before ES Modules (ES2015), JavaScript had no native module system. Several community-driven solutions emerged: CommonJS (used by Node.js), AMD (designed for browsers), UMD (a universal pattern combining both), and earlier patterns like the Revealing Module Pattern and IIFE-based modules. These systems are now considered "legacy" because ES Modules are the official standard, but they remain in widespread use in existing codebases and npm packages.

**Beginner-Friendly Explanation:** Legacy module systems are the "old ways" of organising JavaScript code before the language had built-in modules. They still exist in many projects, especially older ones, but modern development has largely moved to ES Modules.

### Purposes

- To understand the historical context of JavaScript module systems.
- To maintain and migrate legacy codebases that use these systems.
- To choose the right module format when working with older libraries or environments.
- To appreciate the improvements that ES Modules bring.

### Key Legacy Systems

| System | Syntax | Loading | Primary Use | Status |
|--------|--------|---------|-------------|--------|
| **CommonJS** | `require()` / `module.exports` | Synchronous | Node.js | Legacy but still used |
| **AMD** | `define()` / `require()` | Asynchronous | Browsers | Legacy |
| **UMD** | Wrapper pattern | Both | Universal | Legacy |
| **Revealing Module** | IIFE + return object | N/A | Browsers | Legacy |
| **ES Modules** | `import` / `export` | Asynchronous | Standard | Current |

### Annotated Code Examples

#### Example 1: Comparing Legacy Systems

```js
// CommonJS (Node.js):
var math = require('./math');
math.add(2, 3);

// AMD (RequireJS):
define(['./math'], function (math) {
  math.add(2, 3);
});

// UMD (Universal):
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    define(['./math'], factory);
  } else if (typeof exports === 'object') {
    module.exports = factory(require('./math'));
  } else {
    root.mathModule = factory(root.math);
  }
}(this, function (math) {
  math.add(2, 3);
}));

// ES Modules (Modern):
import { add } from './math.js';
add(2, 3);
```

**Expected Output:** All four systems call the `add` function from a `math` module, but with different syntax and loading mechanisms.

**Why This Output:** Each module system provides a different way to define dependencies and export functionality. ES Modules is the modern standard.

#### Example 2: Migrating from AMD to ES Modules

```js
// AMD (old):
define(['jquery', './utils'], function ($, utils) {
  return {
    init: function () {
      $('body').append(utils.message);
    }
  };
});

// ES Modules (new):
import $ from 'jquery';
import { message } from './utils.js';

export function init() {
  $('body').append(message);
}
```

**Expected Output:** Both modules provide an `init` function, but the ES Module version uses modern syntax.

**Why This Output:** Migrating from AMD to ES Modules involves replacing `define()` with `import`/`export` and updating the dependency syntax.

### Real-World Cases

- **Legacy codebases:** Many older projects still use CommonJS, AMD, or UMD.
- **npm packages:** Some packages still ship CommonJS or UMD builds.
- **Migration projects:** Converting legacy module systems to ES Modules is a common task.
- **Browser compatibility:** AMD and UMD were necessary before ES Modules gained widespread browser support.

---

## References

- Asynchronous Module Definition (AMD) API — https://github.com/amdjs/amdjs-api
- AMD Specification (AMD.md) — https://github.com/amdjs/amdjs-api/blob/master/AMD.md
- AMD require() API — https://github.com/amdjs/amdjs-api/blob/master/require.md
- RequireJS — Why AMD? — https://requirejs.org/docs/whyamd
- UMD (Universal Module Definition) Repository — https://github.com/umdjs/umd
- UMD Patterns (CommonJS, AMD, and browser globals) — https://github.com/umdjs/umd/tree/master/templates
- MDN Web Docs – JavaScript modules — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- TypeScript Documentation – Ambient Declarations and AMD — https://cdn.jsdelivr.net/gh/microsoft/TypeScript@v2.3.3/doc/spec.md
- JS模块化系统 (CJS, AMD, UMD, ESM) — https://developer.aliyun.com/article/1521487
- CJS, AMD, UMD, ESM 区别 — https://developer.aliyun.com/article/1214756
- FreeCodeCamp – JavaScript Modules — https://www.freecodecamp.org/news/javascript-modules/
- RequireJS – AMD API — https://requirejs.org/docs/api.html
- Stack Overflow – AMD vs CommonJS vs UMD — https://stackoverflow.com/questions/16521471/relation-between-commonjs-amd-and-requirejs
- James Fisher – What are UMD modules? — https://jameshfisher.com/2020/10/04/what-are-umd-modules/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf