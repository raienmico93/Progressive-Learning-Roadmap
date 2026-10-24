# Modules in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **module** is a source file with its **own lexical scope** that explicitly **exports** declarations for use by other files and **imports** declarations from other files. TypeScript adopts the ECMAScript module (ESM) system as its primary module model, while also supporting CommonJS interoperability. Modules are the fundamental unit of code organization, encapsulation, and dependency management in modern TypeScript applications.

**Technical Definition.** In TypeScript, any file containing a **top-level `import` or `export`** is treated as a module; a file without either is treated as a **script** whose declarations live in the global scope. Modules execute in their own scope, and their declarations are invisible outside unless explicitly exported. TypeScript supports **ES Modules** (the standard `import`/`export` syntax added to JavaScript in 2015) as well as **CommonJS** (`require`/`module.exports`) for Node.js interoperability. TypeScript adds **type-only imports and exports**, **module resolution configuration**, and compiler options such as `verbatimModuleSyntax` and `isolatedModules` to align module syntax with runtime and bundler behavior.

**Beginner-Friendly Explanation.** A module is **a file that keeps its contents private by default**. If you want another file to use something from a module, you must **export** it; if you want to use something from another module, you must **import** it. This is how large programs are split into small, manageable pieces without polluting a global namespace. TypeScript uses the standard JavaScript `import`/`export` syntax and adds type safety on top of it.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **File-level scope** | Each module has its own scope; declarations do not leak globally unless exported. |
| **Explicit dependencies** | Modules declare what they import and export, making dependencies visible and analyzable. |
| **ES Modules first** | TypeScript’s primary module syntax is standard ECMAScript `import`/`export`. |
| **Type-only imports/exports** | TypeScript can erase imports/exports used only for types, avoiding runtime dependencies. |
| **Interoperability** | TypeScript supports both ESM and CommonJS, with compiler options controlling interop. |
| **Static analysis** | Imports and exports are statically analyzable, enabling bundling, tree-shaking, and refactoring. |
| **Module resolution** | The compiler resolves module specifiers to files using configurable strategies (`bundler`, `nodenext`, etc.). |
| **Erasable by design** | Type-only syntax is removed; value imports/exports remain in the emitted JavaScript. |

---

### Prerequisites

- **TypeScript basics**: type annotations, interfaces, and type aliases.
- **JavaScript modules**: familiarity with `import` and `export` syntax.
- **File system and paths**: relative and bare module specifiers.
- **Node.js or a bundler**: to run or bundle the emitted JavaScript.
- **`tsconfig.json`**: understanding of `module`, `moduleResolution`, `target`, and `strict`.

---

### Related Programming Areas

- **ES Modules (ESM)** and **CommonJS (CJS)** — the two dominant module systems.
- **Bundlers** (Vite, webpack, esbuild, Rollup) — consume module graphs and tree-shake.
- **Type-only imports** and **`verbatimModuleSyntax`** — modern TypeScript module hygiene.
- **Module resolution** — how specifiers map to files on disk.
- **Barrel files and re-exports** — aggregating public APIs.
- **Namespaces** — TypeScript’s legacy internal module construct, now discouraged.

---

### Core Concepts / Features

The following sections cover modules, ES modules, `import`, `export`, named exports, default exports, re-exports, and module best practices. Each follows a uniform structure.

---

## 1. Modules

### Definitions

**Core Definition.** A **module** is a file with its own scope that explicitly exports and imports declarations, forming a dependency graph of reusable code units.

**Technical Definition.** A TypeScript file is a module if it contains a **top-level `import` or `export`** (or top-level `await`). Modules have their own scope; declarations are not global unless exported. A file with no imports or exports is a **script** and shares the global scope with other scripts. Adding `export {}` to a file forces it to be treated as a module exporting nothing. Modules are the foundation for dependency management, encapsulation, and tooling.

**Beginner-Friendly Explanation.** A module is **a file that minds its own business**. Nothing inside it is visible to other files unless it is explicitly exported. If a file has no imports or exports, TypeScript treats it as an old-fashioned script that shares global variables—usually not what you want. Adding `export {}` is a quick way to mark a file as a module.

### Purposes

- **To encapsulate code** in file-level scopes and prevent global namespace pollution.
- **To make dependencies explicit** through imports and exports.
- **To enable code reuse** across files, packages, and applications.
- **To support tooling** such as bundlers, tree-shaking, refactoring, and type checking.
- **To provide a unit of compilation and distribution** for libraries and applications.

### Syntax Rules and Structure

**Module detection:**

```typescript
// A file with a top-level import or export is a module.
import { something } from "./somewhere.js";
export const value = 42;
```

**Forcing a file to be a module:**

```typescript
export {}; // Marks the file as a module that exports nothing.
```

**Component breakdown:**

- `import` / `export` — top-level module syntax that marks the file as a module.
- `export {}` — an empty export list; makes a file a module without adding runtime exports.

**Syntax rules.**

- Any file with a top-level `import` or `export` is a module.
- A file without module syntax is a **script** and shares the global scope.
- Module declarations are not global; they must be imported to be used elsewhere.
- `export {}` is a no-op export that marks a file as a module.

**Constraints and limitations.**

- Scripts can accidentally pollute the global scope; prefer modules everywhere.
- Module detection can be affected by `moduleDetection` compiler settings.
- Ambient declarations and global augmentation require care in module files.

### Annotated Complete Step by Step Code Examples

#### Example 1: Script vs. module

```typescript
// file: script.ts (no import/export — a script)
const greeting = "Hello from script";

// file: module.ts (has export — a module)
export const greeting = "Hello from module";
```

```typescript
// file: main.ts
import { greeting } from "./module.js";
console.log(greeting); // Expected: Hello from module
```

**Setup.** Compile with `npx tsc --module nodenext --target es2022 module.ts main.ts` and run `node main.js`.

**Expected Output:**
```
Hello from module
```

**Why this output occurs.** `module.ts` has a top-level `export`, so it is a module with its own scope. `main.ts` imports the exported `greeting`. The `script.ts` file has no exports and is treated as a script; its `greeting` would be global and is not imported here.

---

#### Example 2: Forcing a file to be a module with `export {}`

```typescript
// file: standalone.ts
const privateValue = 42;
export {}; // Now this file is a module.
```

**Expected Output:**
```
(no runtime output; module classification is compile-time behavior)
```

**Why this output occurs.** `export {}` contains a top-level `export`, so TypeScript treats the file as a module even though nothing is exported. `privateValue` remains module-scoped.

---

### Real-World Cases

- **Application structure**: every `.ts` file is a module with explicit exports.
- **Library packaging**: modules define the public API surface.
- **Monorepos**: modules and packages form a dependency graph.
- **Testing**: modules can be imported and mocked in isolation.

---

## 2. ES Modules

### Definitions

**Core Definition.** **ES Modules (ESM)** is the standard JavaScript module system built into the language, using static `import` and `export` declarations. It is supported in modern browsers and Node.js.

**Technical Definition.** ESM uses dedicated `import` and `export` syntax and is supported in modern browsers and Node.js since v12. ESM is **static**: imports and exports are analyzed at parse time, enabling tree-shaking and early error detection. TypeScript emits ESM when configured with `"module": "esnext"`, `"nodenext"`, or similar options. ESM files may be executed directly by runtimes or bundled by tools.

**Beginner-Friendly Explanation.** ES Modules are **the modern, standard way to split JavaScript into files**. You use `import` to bring things in and `export` to send things out. Browsers and Node.js understand this natively, and bundlers can optimize it.

### Purposes

- **To use the standard JavaScript module system** supported by browsers, Node.js, and bundlers.
- **To enable static analysis** and tree-shaking through static `import`/`export` syntax.
- **To interoperate with modern tooling** such as Vite, Rollup, and esbuild.
- **To replace legacy CommonJS** in new code.
- **To support top-level `await`** in module files.

### Syntax Rules and Structure

**ESM import/export:**

```typescript
// math.ts
export function add(a: number, b: number): number { return a + b; }

// main.ts
import { add } from "./math.js";
console.log(add(2, 3));
```

**Component breakdown:**

- `export function add(...)` — exports a named declaration.
- `import { add } from "./math.js"` — imports the named export.
- `.js` extension — required in Node ESM for relative specifiers.

**Syntax rules.**

- ESM uses `import`/`export`, not `require`/`module.exports`.
- Imports are **static** and hoisted; they cannot be conditionally placed inside blocks.
- Under Node ESM, relative imports require **file extensions** (e.g., `./math.js`).
- TypeScript’s `module` option controls the emitted module format.
- `verbatimModuleSyntax` helps preserve or erase imports consistently.

**Constraints and limitations.**

- ESM and CommonJS interoperability requires configuration (`esModuleInterop`, `module: nodenext`).
- Top-level `await` is available only in modules, not scripts.
- Default export interop between ESM and CJS can be confusing.

### Annotated Complete Step by Step Code Examples

#### Example 1: ESM file pair

```typescript
// file: math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export const PI = 3.14159;
```

```typescript
// file: main.ts
import { add, PI } from "./math.js";

console.log(add(2, 3)); // Expected: 5
console.log(PI);        // Expected: 3.14159
```

**Setup.** Compile with `npx tsc --module nodenext --target es2022 math.ts main.ts` and run `node main.js`.

**Expected Output:**
```
5
3.14159
```

**Why this output occurs.** `math.ts` exports two named bindings. `main.ts` imports them using ESM syntax. The `.js` extension is required under Node ESM module resolution.

---

#### Example 2: Dynamic `import()`

```typescript
// file: dynamic.ts
async function loadMath() {
  const math = await import("./math.js");
  console.log(math.add(10, 20)); // Expected: 30
}

loadMath();
```

**Expected Output:**
```
30
```

**Why this output occurs.** Dynamic `import()` returns a promise for the module namespace object. It is evaluated at runtime, unlike static imports.

---

### Real-World Cases

- **Modern frontend apps**: Vite and Rollup consume ESM.
- **Node.js packages**: `"type": "module"` enables ESM.
- **Library distribution**: ESM builds for bundlers and browsers.
- **Code splitting**: dynamic `import()` loads modules on demand.

---

## 3. `import`

### Definitions

**Core Definition.** The **`import` declaration** brings exported bindings from another module into the current module’s scope.

**Technical Definition.** TypeScript recognizes standard ECMAScript `import` forms: **named imports** (`import { x }`), **default imports** (`import x`), **namespace imports** (`import * as ns`), and **side-effect imports** (`import "mod"`). It adds **type-only imports** (`import type`) and inline `type` specifiers (`import { type T, value }`) that are guaranteed to be erased from the output. Imports are **static** and hoisted. TypeScript also supports the legacy CommonJS-interop syntax `import x = require("x")`, which should be avoided in new ESM code.

**Beginner-Friendly Explanation.** `import` is **how you bring code from another file into yours**. You can import specific named things with braces, a default thing without braces, or everything at once as a namespace. You can also import a module just for its side effects. TypeScript lets you mark type-only imports so they disappear from the final JavaScript.

### Purposes

- **To access exported values and types** from other modules.
- **To declare dependencies explicitly** at the top of a file.
- **To enable static analysis, bundling, and tree-shaking**.
- **To separate type-only imports** from runtime imports for cleaner output.
- **To support namespace imports** for convenient access to many exports.

### Syntax Rules and Structure

**Named import:**

```typescript
import { add, PI as pi } from "./math.js";
```

**Default import:**

```typescript
import greet from "./greeter.js";
```

**Namespace import:**

```typescript
import * as math from "./math.js";
```

**Side-effect import:**

```typescript
import "./polyfills.js";
```

**Type-only import:**

```typescript
import type { User } from "./types.js";
import { type Config, loadConfig } from "./config.js";
```

**Component breakdown:**

- `import` — declaration keyword.
- `{ add, PI as pi }` — named specifiers, optionally renamed with `as`.
- `greet` — default binding.
- `* as math` — namespace binding.
- `type` — type-only modifier.

**Syntax rules.**

- Named imports use braces; default imports do not.
- `as` renames a binding locally.
- Namespace imports bind the entire module namespace object.
- Side-effect imports execute the module without binding anything.
- `import type` is guaranteed to be erased from JavaScript.
- Type-only imports cannot combine a default import and named bindings without splitting or using `default as`.

**Constraints and limitations.**

- `import type` bindings cannot be used as values.
- `import x = require("x")` is TypeScript-specific and CommonJS-only; avoid it in ESM.
- Static imports must be top-level; dynamic `import()` is available for runtime loading.
- Node ESM requires explicit file extensions for relative imports.

### Annotated Complete Step by Step Code Examples

#### Example 1: Named, namespace, and side-effect imports

```typescript
// file: utils.ts
export const version = "1.0.0";
export function log(message: string) { console.log(message); }
console.log("utils loaded");
```

```typescript
// file: main.ts
import { version, log } from "./utils.js";
import * as utils from "./utils.js";
import "./side-effect.js";

log(`Version: ${version}`); // Expected: Version: 1.0.0
console.log(utils.version); // Expected: 1.0.0
```

**Expected Output (assuming `side-effect.js` logs “side effect”):**
```
utils loaded
side effect
Version: 1.0.0
1.0.0
```

**Why this output occurs.** Named imports bind `version` and `log`. The namespace import binds the entire module object. The side-effect import executes `side-effect.js` without binding anything. The module is evaluated only once.

---

#### Example 2: Type-only import

```typescript
// file: types.ts
export interface User { id: string; name: string }
export const defaultUser: User = { id: "u1", name: "Ada" };
```

```typescript
// file: main.ts
import { defaultUser, type User } from "./types.js";

const user: User = defaultUser;
console.log(user.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `type User` is erased from the emitted JavaScript. Only `defaultUser` remains as a runtime import. This avoids a runtime dependency on a type-only binding.

---

### Real-World Cases

- **Importing services and utilities** in application code.
- **Namespace imports** for large utility modules (`import * as _ from "lodash"`).
- **Side-effect imports** for CSS, polyfills, and global setup.
- **Type-only imports** for interfaces and type aliases that do not exist at runtime.

---

## 4. `export`

### Definitions

**Core Definition.** The **`export` declaration** makes a module’s declarations available to other modules.

**Technical Definition.** TypeScript supports **export declarations** (`export function f() {}`), **export lists** (`export { x, y as z }`), **default exports** (`export default expr`), **type exports** (`export type { T }`), and **re-exports** (`export { x } from "./mod.js"`, `export * from "./mod.js"`). Exports are static and can be analyzed by tooling. TypeScript-specific declarations such as interfaces and type aliases can be exported like JavaScript declarations.

**Beginner-Friendly Explanation.** `export` is **how you make something available to other files**. You can export a function, class, variable, or type directly, or list several exports at once. You can also re-export things from other modules.

### Purposes

- **To define a module’s public API**.
- **To make declarations available to importers**.
- **To control what is exposed** and keep everything else private.
- **To support type-only exports** that are erased from JavaScript.
- **To aggregate and re-export modules** through barrel files.

### Syntax Rules and Structure

**Export declaration:**

```typescript
export function add(a: number, b: number): number { return a + b; }
export const PI = 3.14159;
export interface User { id: string; name: string }
```

**Export list:**

```typescript
function internal() {}
export { internal as publicName };
```

**Default export:**

```typescript
export default function greet(name: string): string {
  return `Hello, ${name}`;
}
```

**Type-only export:**

```typescript
export type { User };
export type Config = { debug: boolean };
```

**Re-export:**

```typescript
export { add } from "./math.js";
export * from "./models.js";
export * as utils from "./utils.js";
```

**Component breakdown:**

- `export` — declaration keyword.
- `default` — marks the single default export.
- `type` — type-only modifier.
- `from "./mod.js"` — re-export source.

**Syntax rules.**

- A module may have **one** default export.
- Named exports may be multiple.
- `export type` is erased from JavaScript.
- `export *` re-exports all named exports, but **not** the default export.
- `export * as ns` re-exports the entire module as a namespace object.

**Constraints and limitations.**

- `export =` is TypeScript-specific CommonJS syntax and should be avoided in ESM.
- Default exports have no canonical name at the import site, weakening refactoring.
- Re-exporting a default requires `export { default } from` or `export { default as X } from`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Export declaration and list

```typescript
// file: math.ts
export function add(a: number, b: number): number {
  return a + b;
}

function subtract(a: number, b: number): number {
  return a - b;
}

export { subtract as minus };
```

```typescript
// file: main.ts
import { add, minus } from "./math.js";
console.log(add(5, 3));  // Expected: 8
console.log(minus(5, 3)); // Expected: 2
```

**Expected Output:**
```
8
2
```

**Why this output occurs.** `add` is exported directly; `subtract` is exported under the alias `minus` via an export list. Both are available to importers.

---

#### Example 2: Type-only export

```typescript
// file: models.ts
export interface User { id: string; name: string }
export const users: User[] = [];
```

```typescript
// file: main.ts
import { users, type User } from "./models.js";
const user: User = { id: "u1", name: "Ada" };
users.push(user);
console.log(users.length); // Expected: 1
```

**Expected Output:**
```
1
```

**Why this output occurs.** `User` is exported as a type only and is erased; `users` remains a runtime value.

---

### Real-World Cases

- **Library entry points**: exporting the public API.
- **Barrel files**: aggregating multiple modules.
- **Type-only exports**: exposing interfaces without runtime code.
- **Namespace re-exports**: grouping utilities under a namespace.

---

## 5. Named Exports

### Definitions

**Core Definition.** **Named exports** are exports that are identified by a **stable name** in the exporting module and imported using braces (`import { name }`). A module may have **any number** of named exports.

**Technical Definition.** Named exports are static bindings with a canonical name in the exporting module. They support **renaming on import** (`import { x as y }`) and **renaming on export** (`export { x as y }`). Named exports enable precise static analysis: importing a non-existent name is a compile-time error. They are the recommended export style by the Google TypeScript Style Guide and many other authorities.

**Beginner-Friendly Explanation.** Named exports are **exports with names**. You export `add` and `subtract`, and other files import exactly `add` and `subtract` using braces. If someone tries to import something that does not exist, TypeScript catches it.

### Purposes

- **To provide multiple, clearly named exports** from a module.
- **To enable compile-time errors** for misspelled or missing imports.
- **To support reliable refactoring and auto-import** because names are stable.
- **To avoid naming inconsistencies** at import sites.
- **To allow aliasing** when a local name conflicts.

### Syntax Rules and Structure

```typescript
// math.ts
export function add(a: number, b: number): number { return a + b; }
export const PI = 3.14159;
```

```typescript
// main.ts
import { add, PI } from "./math.js";
```

**Component breakdown:**

- `export function add` — named export declaration.
- `export const PI` — named export declaration.
- `import { add, PI }` — named import.

**Syntax rules.**

- Named exports use braces in imports.
- A module may have any number of named exports.
- `as` renames on import or export.
- Missing named exports produce compile-time errors.
- Type-only named exports use `export type` or inline `type`.

**Constraints and limitations.**

- Named exports require importers to know the exact names.
- Barrel files that re-export many names can create large public surfaces.
- Names must be unique within a module’s export list.

### Annotated Complete Step by Step Code Examples

#### Example 1: Multiple named exports

```typescript
// file: math.ts
export function add(a: number, b: number): number { return a + b; }
export function multiply(a: number, b: number): number { return a * b; }
export const PI = 3.14159;
```

```typescript
// file: main.ts
import { add, multiply, PI } from "./math.js";

console.log(add(2, 3));      // Expected: 5
console.log(multiply(2, 3)); // Expected: 6
console.log(PI);             // Expected: 3.14159
```

**Expected Output:**
```
5
6
3.14159
```

**Why this output occurs.** All three bindings are named exports with stable identities. Importing them by name is checked at compile time.

---

#### Example 2: Named export with alias

```typescript
// file: math.ts
function subtract(a: number, b: number): number { return a - b; }
export { subtract as minus };
```

```typescript
// file: main.ts
import { minus } from "./math.js";
console.log(minus(5, 2)); // Expected: 3
```

**Expected Output:**
```
3
```

**Why this output occurs.** The export list renames `subtract` to `minus` for importers.

---

### Real-World Cases

- **Utility libraries**: exporting many named functions.
- **Component libraries**: exporting `Button`, `Input`, and `Modal`.
- **Type packages**: exporting named interfaces and type aliases.
- **Barrel files**: re-exporting named APIs from submodules.

---

## 6. Default Exports

### Definitions

**Core Definition.** A **default export** is a single export per module that is imported **without braces** and whose local name is chosen by the importer. It is intended for modules with one primary export.

**Technical Definition.** A module may have **at most one** default export, declared with `export default`. Importers may bind it to any local name (`import Foo from "./mod.js"`), so the name is not canonical across the codebase. Default exports are **not included** in `export *` re-exports; re-exporting a default requires `export { default } from` or `export { default as X } from`. The Google TypeScript Style Guide explicitly discourages default exports because they weaken refactoring and naming consistency.

**Beginner-Friendly Explanation.** A default export is **the “main thing” a module exports**. You import it without braces and can call it whatever you like. Only one default is allowed per module. Many style guides recommend against it because different files can import it under different names.

### Purposes

- **To designate a single primary export** for a module.
- **To simplify imports** for modules that conceptually export one thing.
- **To interoperate with ecosystems** that conventionally use default exports (e.g., some frameworks).
- **To support default-import interoperability** with CommonJS modules when configured.

### Syntax Rules and Structure

```typescript
// greeter.ts
export default function greet(name: string): string {
  return `Hello, ${name}`;
}
```

```typescript
// main.ts
import greet from "./greeter.js";
console.log(greet("Ada"));
```

**Component breakdown:**

- `export default` — declares the default export.
- `import greet from` — imports the default without braces.
- The local name `greet` is chosen by the importer.

**Syntax rules.**

- At most one default export per module.
- Imported without braces.
- The importer chooses the local name.
- `export *` does not re-export the default.
- Re-export with `export { default } from "./mod.js"` or `export { default as X } from "./mod.js"`.

**Constraints and limitations.**

- No canonical name at the import site; refactoring and auto-import are weaker.
- Different files may import the same default under different names.
- Default exports encourage bundling many unrelated things into one object.
- Google TypeScript Style Guide recommends **not** using default exports.

### Annotated Complete Step by Step Code Examples

#### Example 1: Default export and import

```typescript
// file: greeter.ts
export default function greet(name: string): string {
  return `Hello, ${name}`;
}
```

```typescript
// file: main.ts
import greet from "./greeter.js";
console.log(greet("Ada")); // Expected: Hello, Ada

// A different file may import it under a different name:
// import sayHello from "./greeter.js";
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The default export is bound to the local name `greet` in `main.ts`. The module has no canonical export name, so another file could call it `sayHello` without a compile-time error.

---

#### Example 2: Re-exporting a default

```typescript
// file: index.ts
export { default as greet } from "./greeter.js";
```

```typescript
// file: main.ts
import { greet } from "./index.js";
console.log(greet("Grace")); // Expected: Hello, Grace
```

**Expected Output:**
```
Hello, Grace
```

**Why this output occurs.** The default export is re-exported under the named export `greet`, restoring a canonical name for importers.

---

### Real-World Cases

- **Framework conventions**: some frameworks expect default exports for pages or components.
- **Single-class modules**: a module exporting one class.
- **Interop wrappers**: re-exporting CommonJS defaults.
- **Legacy codebases**: existing default-export patterns.

---

## 7. Re-exports

### Definitions

**Core Definition.** A **re-export** is an export declaration that exports bindings **from another module** without first importing them into local scope. Re-exports are used to aggregate and forward APIs, commonly in **barrel files**.

**Technical Definition.** TypeScript supports `export { x } from "./mod.js"`, `export { x as y } from "./mod.js"`, `export * from "./mod.js"`, and `export * as ns from "./mod.js"`. `export *` re-exports all **named** exports but **not** the default export. Type-only re-exports use `export type { T } from "./mod.js"`. Re-exports do not introduce local names, so they cannot be used within the re-exporting module itself.

**Beginner-Friendly Explanation.** A re-export is **forwarding something from another file**. It is how a barrel file (`index.ts`) collects exports from many files and presents a single public API. You do not have to import the thing first; you just re-export it directly.

### Purposes

- **To aggregate exports** from multiple modules into a single entry point.
- **To create stable public APIs** for libraries and packages.
- **To avoid importing and re-exporting in two steps**.
- **To rename exports** during re-export for compatibility.
- **To group related modules** under a namespace with `export * as ns`.

### Syntax Rules and Structure

**Named re-export:**

```typescript
export { add } from "./math.js";
```

**Renamed re-export:**

```typescript
export { subtract as minus } from "./math.js";
```

**Wildcard re-export:**

```typescript
export * from "./models.js";
```

**Namespace re-export:**

```typescript
export * as utils from "./utils.js";
```

**Default re-export:**

```typescript
export { default } from "./greeter.js";
export { default as greet } from "./greeter.js";
```

**Type re-export:**

```typescript
export type { User } from "./types.js";
export type { User as UserShape } from "./types.js";
```

**Component breakdown:**

- `export { ... } from "..."` — re-exports specific bindings.
- `export * from "..."` — re-exports all named bindings.
- `export * as ns from "..."` — re-exports the module namespace as `ns`.
- `type` — marks the re-export as type-only.

**Syntax rules.**

- Re-exported bindings are not introduced into the local scope.
- `export *` does not re-export the default export.
- Defaults must be re-exported explicitly with `export { default } from` or aliased.
- `export type` guarantees erasure of type-only re-exports.
- `export * as ns` creates a namespace object named `ns`.

**Constraints and limitations.**

- Wildcard re-exports can create name collisions if multiple modules export the same name.
- Barrel files can create circular dependencies if not managed carefully.
- Re-exported types must use `export type` under strict module settings.

### Annotated Complete Step by Step Code Examples

#### Example 1: Barrel file with named and wildcard re-exports

```typescript
// file: models/user.ts
export interface User { id: string; name: string }
export const defaultUser: User = { id: "u1", name: "Ada" };
```

```typescript
// file: models/post.ts
export interface Post { id: string; title: string }
export const defaultPost: Post = { id: "p1", title: "Hello" };
```

```typescript
// file: models/index.ts
export { defaultUser, type User } from "./user.js";
export * from "./post.js";
```

```typescript
// file: main.ts
import { defaultUser, defaultPost } from "./models/index.js";
console.log(defaultUser.name); // Expected: Ada
console.log(defaultPost.title); // Expected: Hello
```

**Expected Output:**
```
Ada
Hello
```

**Why this output occurs.** `models/index.ts` aggregates the two modules. `defaultUser` is explicitly re-exported with its type, and `export *` forwards all named exports from `post.ts`. Importers use a single entry point.

---

#### Example 2: Namespace re-export

```typescript
// file: utils/string.ts
export function capitalize(s: string): string { return s.toUpperCase(); }
```

```typescript
// file: utils/index.ts
export * as stringUtils from "./string.js";
```

```typescript
// file: main.ts
import { stringUtils } from "./utils/index.js";
console.log(stringUtils.capitalize("hello")); // Expected: HELLO
```

**Expected Output:**
```
HELLO
```

**Why this output occurs.** `export * as stringUtils` re-exports the entire `string.ts` module as a namespace object. Importers access its exports through `stringUtils`.

---

### Real-World Cases

- **Package entry points**: `index.ts` re-exports the public API.
- **Feature modules**: aggregating components, hooks, and types.
- **Type-only barrels**: re-exporting interfaces with `export type`.
- **Namespace grouping**: `export * as utils` for organized utilities.

---

## 8. Module Best Practices

### Definitions

**Core Definition.** **Module best practices** are guidelines for writing modules that are **predictable, refactorable, tooling-friendly, and compatible with modern runtimes and bundlers**.

**Technical Definition.** Best practices center on **preferring named exports over default exports**, **using ES modules rather than legacy TypeScript/CommonJS syntax**, **marking type-only imports and exports explicitly**, **enabling `verbatimModuleSyntax` (or `isolatedModules`)**, **minimizing the exported API surface**, and **avoiding TypeScript-specific module syntax such as `export =` and `import = require`**. They also include correct module resolution configuration (`module`, `moduleResolution`) and explicit file extensions under Node ESM.

**Beginner-Friendly Explanation.** Module best practices are **the habits experienced TypeScript developers use** to keep imports and exports clean. The big rules are: use named exports instead of default exports; use `import`/`export`, not `require`; mark type-only imports with `import type`; enable `verbatimModuleSyntax`; and keep your public API small.

### Purposes

- **To make imports predictable and refactorable** with stable names.
- **To ensure compatibility with bundlers and single-file transpilers**.
- **To avoid runtime dependencies on type-only bindings**.
- **To keep the public API surface minimal and intentional**.
- **To align emitted JavaScript with the actual module system**.
- **To prevent accidental global-scope pollution**.

### Syntax Rules and Structure

Best practices are conventions that interact with module syntax. The core rules are:

1. **Prefer named exports over default exports.** Named exports provide canonical names, catch import typos at compile time, and refactor reliably. The Google TypeScript Style Guide says: “Use named exports in all code” and “Do not use default exports”.
2. **Use ES modules, not `namespace` or `require`.** Use `import`/`export` consistently and avoid legacy TypeScript `namespace` and CommonJS `require`/`module.exports` in new code.
3. **Use `import type` and `export type` for type-only bindings.** This makes erasure explicit and avoids runtime dependencies.
4. **Enable `verbatimModuleSyntax`.** It preserves non-type imports and drops type-only ones, simplifying module output. It also enables `isolatedModules` behavior.
5. **Use `isolatedModules` when `verbatimModuleSyntax` is not available.** It ensures each file can be transpiled independently by tools like esbuild, SWC, and Babel.
6. **Avoid `export =` and `import = require`.** They are TypeScript-specific CommonJS syntax and should not be used in new ESM code.
7. **Minimize the exported API surface.** Only export what consumers need; keep internals private.
8. **Use explicit `.js` extensions for relative imports under Node ESM.** `module: nodenext` requires them.
9. **Configure module resolution correctly.** Use `"module": "esnext"` and `"moduleResolution": "bundler"` for bundlers; `"module": "nodenext"` for Node.js.
10. **Be careful with barrel files.** They improve ergonomics but can create circular dependencies and increase bundle size; keep them intentional.

**Component breakdown (recommended `tsconfig.json` excerpt for a bundler project):**

```json
{
  "compilerOptions": {
    "module": "esnext",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,
    "strict": true
  }
}
```

- `module` — emitted module format.
- `moduleResolution` — how specifiers resolve to files.
- `esModuleInterop` — improves ESM/CJS default import interop.
- `verbatimModuleSyntax` — preserves non-type imports and erases type-only ones.
- `noEmit` — bundler handles output.

**Constraints and limitations.**

- `verbatimModuleSyntax` requires explicit `type` modifiers for type-only imports/exports.
- `isolatedModules` rejects cross-file type re-exports without `export type`.
- Default exports remain necessary in some framework conventions (e.g., Next.js pages), where best practices may be relaxed.
- Barrel files can introduce circular dependencies if not managed.

### Annotated Complete Step by Step Code Examples

#### Example 1: Named exports over default exports

```typescript
// ❌ Default export: importer chooses the name.
// file: user.ts
export default class User {
  constructor(public name: string) {}
}

// file: main.ts
import AnyName from "./user.js"; // Name is arbitrary.
```

```typescript
// ✅ Named export: canonical name, typo-safe.
// file: user.ts
export class User {
  constructor(public name: string) {}
}

// file: main.ts
import { User } from "./user.js";
console.log(new User("Ada").name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The named export has a canonical name. Importing `User` is checked at compile time. A typo such as `import { Usr }` would fail immediately, unlike a default import that can be renamed arbitrarily.

---

#### Example 2: Type-only imports and exports

```typescript
// file: types.ts
export interface User { id: string; name: string }
export const defaultUser: User = { id: "u1", name: "Ada" };
```

```typescript
// file: main.ts
import { defaultUser, type User } from "./types.js";

const user: User = defaultUser;
console.log(user.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `type User` is erased from JavaScript; only `defaultUser` is emitted. This avoids a runtime import for an interface that does not exist at runtime.

---

#### Example 3: `verbatimModuleSyntax` in practice

```typescript
// file: main.ts
import type { User } from "./types.js";       // Erased entirely.
import { defaultUser, type Config } from "./types.js"; // Becomes import { defaultUser }.
```

**Expected Output:**
```
(no runtime output; this is a compile-time transformation)
```

**Why this output occurs.** Under `verbatimModuleSyntax`, imports without `type` are preserved, and imports with `type` are dropped. This makes the emitted JavaScript predictable across compilers.

---

### Real-World Cases

- **Library APIs**: named exports for stable public interfaces.
- **Bundler-based apps**: `moduleResolution: "bundler"` with `verbatimModuleSyntax`.
- **Node.js ESM packages**: `"module": "nodenext"`, `.js` extensions, `"type": "module"`.
- **Code review**: rejecting default exports and `export =` in new code.

---

## References

- TypeScript Handbook — Modules - https://www.typescriptlang.org/docs/handbook/2/modules.html
- TypeScript Handbook — Modules: Theory - https://www.typescriptlang.org/docs/handbook/modules/theory.html
- TypeScript Handbook — Modules: Reference - https://www.typescriptlang.org/docs/handbook/modules/reference.html
- TypeScript Handbook — Modules: Choosing Compiler Options - https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html
- TSConfig Reference — `verbatimModuleSyntax` - https://www.typescriptlang.org/tsconfig/verbatimModuleSyntax
- TSConfig Reference — `isolatedModules` - https://www.typescriptlang.org/tsconfig/isolatedModules
- TSConfig Reference — `module` - https://www.typescriptlang.org/tsconfig/module
- TSConfig Reference — `moduleResolution` - https://www.typescriptlang.org/tsconfig/moduleResolution
- TypeScript 1.5 Release Notes — ES6 Modules, Re-exports, Default Exports - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-5.html
- TypeScript 3.8 Release Notes — Type-Only Imports and Exports - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html
- TypeScript 5.0 Release Notes — `verbatimModuleSyntax` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html
- Google TypeScript Style Guide — Modules and Exports - https://google.github.io/styleguide/tsguide.html
- Microsoft Cloudpack — Module Settings and `isolatedModules` - https://microsoft.github.io/cloudpack/repo-guidance/tsconfig-isolated-modules
- MDN — JavaScript Modules - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules