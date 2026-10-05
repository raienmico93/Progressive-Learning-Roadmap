# TypeScript Third-Party Type Definitions: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Third-party type definitions are TypeScript's mechanism for describing the shape of JavaScript code that exists outside your project—whether from the browser environment (`DOM`), the JavaScript language itself (`ES2022`), community-maintained packages (`@types/*`), or untyped JavaScript libraries you need to integrate. They bridge the gap between runtime JavaScript and TypeScript's compile-time type system, enabling type checking, autocompletion, and error detection even when the underlying code was not written in TypeScript.

**Technical Definition**
TypeScript's type system incorporates external type information from three sources: (1) **built-in library files** (`lib.*.d.ts`) shipped with the TypeScript compiler, which describe the JavaScript standard library and host environments (DOM, WebWorker, ES2022, etc.); (2) **community-maintained declaration packages** in the `@types/*` scope on npm, sourced from the DefinitelyTyped repository, which describe popular JavaScript libraries; and (3) **local declaration files** (`.d.ts`) written by developers to type untyped libraries or augment existing types. The compiler resolves these types through a structured lookup process governed by the `lib`, `types`, and `typeRoots` compiler options. The `skipLibCheck` option controls whether the compiler type-checks these declaration files, which has significant implications for build performance.

**Beginner-Friendly Explanation**
TypeScript doesn't live in a vacuum—it needs to know about the JavaScript around it. When you use `document.querySelector()` in a browser, TypeScript needs to know that `document` exists. When you use `Math.max()` in any JavaScript runtime, it needs to know that too. This knowledge comes from three places: **built-in types** (shipped with TypeScript for the browser and language), **community types** (installed from npm as `@types/packages`), and **your own types** (written in `.d.ts` files for libraries that don't have types). Understanding how to configure and manage these types is essential for any real-world TypeScript project—especially as your project grows and more third-party dependencies enter the picture.

### Key Characteristics

- **Multi-source**: Built-in (`lib`), community (`@types`), and local (`.d.ts`) type definitions.
- **Configurable**: The `lib`, `types`, `typeRoots`, and `skipLibCheck` compiler options control type discovery and checking.
- **Performance-sensitive**: Type checking declaration files is expensive; `skipLibCheck` trades correctness for speed.
- **Security-relevant**: `typeRoots` and `types` control which `@types` packages enter the global scope, reducing supply-chain risk.
- **Extensible**: Module augmentation and `declare module` allow patching existing types or stubbing untyped libraries.

### Prerequisites

- Basic knowledge of TypeScript `tsconfig.json` configuration
- Familiarity with npm packages and `package.json`
- Understanding of `.d.ts` declaration files (see the Declaration Files cheat sheet)
- Basic knowledge of module resolution

### Related Programming Areas

- **Declaration Files**: `.d.ts` files are the format for all third-party type definitions
- **Module Resolution**: How TypeScript locates type definition files
- **Compiler Performance**: `skipLibCheck` directly impacts build times
- **Supply-Chain Security**: `@types` packages are dependencies that enter your build environment
- **Monorepos**: Type definition management across packages

### Core Concepts / Features

1. Built-in JavaScript Execution Context Library Types (`DOM`, `ES2022`, etc. via `tsconfig.json` `lib`)
2. Community-Maintained Typings and Managing `@types` Packages in `devDependencies`
3. Typing Untyped JavaScript Libraries and Writing Local Type Overrides
4. The `typeRoots` and `types` Configuration Settings for Controlling Global Scope Visibility
5. Configuring `skipLibCheck` and Its Critical Impact on Compiler Build Performance


## 1. Built-in JavaScript Execution Context Library Types (`DOM`, `ES2022`, etc. via `tsconfig.json` `lib`)

### Definitions

**Core Definition**
Built-in library types are the `.d.ts` files shipped with the TypeScript compiler that describe the JavaScript standard library and host environments. They are controlled by the `lib` compiler option in `tsconfig.json`. If `lib` is not specified, TypeScript uses a default set based on the `target` option.

**Technical Definition**
TypeScript ships with a collection of declaration files (`lib.*.d.ts`) that describe the JavaScript language standard (ES5, ES2015, ES2016, ..., ES2022, ESNext) and host environments (DOM, DOM.Iterable, WebWorker, ScriptHost, etc.). The `lib` compiler option specifies which of these files to include in the compilation. When `lib` is omitted, TypeScript automatically includes a default set based on `target`: `ES5` target includes `DOM`, `ES5`, `ScriptHost`; `ES2015` target includes `DOM`, `ES2015`, `ScriptHost`; and so on. Since TypeScript 4.5, when including `dom` in `lib`, TypeScript will use the types in `node_modules/@typescript/lib-dom` if available, allowing custom DOM type overrides. The `lib` option accepts friendly names like `"ES2022"`, `"DOM"`, `"DOM.Iterable"`, which TypeScript normalizes internally.

**Beginner-Friendly Explanation**
TypeScript comes with built-in "dictionaries" of type information for JavaScript itself and for the browser. When you write `document.querySelector()`, TypeScript looks up `document` in the DOM dictionary. When you write `Math.max()`, it looks in the ES dictionary. The `lib` option in `tsconfig.json` controls which dictionaries are loaded. For a browser application, you need `"DOM"` and `"DOM.Iterable"`. For a Node.js application, you probably don't need DOM types (and including them can cause confusion by making browser globals appear available). The `target` option sets a default, but you can override it with `lib` for fine-grained control.

### Purposes

- To enable type checking for JavaScript standard library APIs (`Math`, `Promise`, `Array.prototype.map`, etc.).
- To enable type checking for browser APIs (`document`, `window`, `fetch`, `localStorage`, etc.).
- To exclude host environment types that are not relevant (e.g., excluding DOM for Node.js projects).
- To explicitly include modern JavaScript features not present in the default `target` set.
- To allow custom overrides of built-in DOM types (via `node_modules/@typescript/lib-dom`).

### Syntax Rules and Structure

**General Syntax: `lib` Configuration**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"]
  }
}
```

**Component Breakdown**
- `"target": "ES2022"`: Sets the JavaScript output level and the default `lib` set.
- `"lib": ["ES2022", "DOM", "DOM.Iterable"]`: Explicitly includes ES2022, DOM, and DOM.Iterable types.
- `DOM.Iterable`: Adds `Symbol.iterator` to DOM collections (NodeList, etc.), making them iterable in `for...of` loops.

**General Syntax: Default `lib` Based on `target`**

```json
// If `lib` is omitted:
// target: ES5    → lib: DOM, ES5, ScriptHost
// target: ES2015 → lib: DOM, ES2015, ScriptHost
// target: ES2022 → lib: DOM, ES2022, ScriptHost
```

**Component Breakdown**
- The default `lib` always includes `DOM` and `ScriptHost` unless overridden.
- Node.js projects should explicitly set `lib` to exclude `DOM`.

**General Syntax: Node.js Configuration (No DOM)**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022"],
    "types": ["node"]
  }
}
```

**Component Breakdown**
- `"lib": ["ES2022"]`: Excludes DOM types.
- `"types": ["node"]`: Includes Node.js types from `@types/node`.

**Syntax Rules**

- The `lib` option accepts an array of library names (e.g., `"ES2022"`, `"DOM"`, `"DOM.Iterable"`, `"WebWorker"`, `"ScriptHost"`, `"ESNext"`).
- Friendly names like `"ES2022"` encompass all features from earlier ECMAScript versions.
- If `lib` is omitted, TypeScript uses a default based on `target`.
- `DOM.Iterable` is required to make DOM collections iterable in `for...of` loops.
- The `DOM` library includes `fetch` (since TypeScript 2.8).
- Custom DOM types can override built-in DOM types via `node_modules/@typescript/lib-dom`.

**Constraints and Limitations**

- The DOM type definitions file (`lib.dom.d.ts`) is thousands of lines long and cannot be easily subsetted—you either include it or you don't.
- Including `DOM` in a Node.js project makes browser globals appear available, leading to runtime errors.
- Excluding `DOM` from a browser project removes `document`, `window`, `fetch`, etc., causing compile errors.
- The `lib` option cannot selectively include individual DOM APIs—it's all or nothing.
- Custom DOM overrides require `node_modules/@typescript/lib-dom` to be installed.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Browser Application Configuration

```json
// Step 1: Configure tsconfig.json for a browser application.
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "strict": true
  }
}
```

```typescript
// Step 2: Use browser and ES2022 APIs.
const button = document.querySelector("button");  // DOM type
const numbers = [1, 2, 3];
const doubled = numbers.map((n) => n * 2);        // ES2022 Array.prototype.map

// DOM.Iterable makes NodeList iterable.
const links = document.querySelectorAll("a");
for (const link of links) {  // ✅ Requires DOM.Iterable
  console.log(link.href);
}

// Step 3: Without DOM.Iterable, for...of on NodeList fails.
// for (const link of links) { }  // ❌ Error: Type 'NodeListOf<HTMLAnchorElement>' is not an array type.
```

**Expected Output:** The code compiles successfully with `DOM.Iterable` included.

**Why This Output Occurs:** `DOM` provides the base DOM types (`document`, `window`, `HTMLElement`), while `DOM.Iterable` adds `Symbol.iterator` to DOM collections, enabling `for...of` loops.

#### Example 2: Node.js Configuration (Excluding DOM)

```json
// Step 1: Configure tsconfig.json for a Node.js application.
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022"],
    "types": ["node"]
  }
}
```

```typescript
// Step 2: Use Node.js and ES2022 APIs.
import { readFileSync } from "fs";  // Node.js type (from @types/node)
const content = readFileSync("data.txt", "utf-8");

const numbers = [1, 2, 3];
const sum = numbers.reduce((a, b) => a + b, 0);  // ES2022

console.log(content, sum);

// Step 3: Browser APIs are NOT available.
// document.querySelector("button");  // ❌ Error: Cannot find name 'document'.
```

**Expected Output:** The Node.js code compiles; browser APIs produce compile errors.

**Why This Output Occurs:** The `lib: ["ES2022"]` excludes DOM types, so `document` is not available. The `types: ["node"]` includes `@types/node`, providing Node.js APIs like `readFileSync`.

### Real-World Cases

**Case 1: React Web Applications**
React web applications use `"lib": ["ES2022", "DOM", "DOM.Iterable"]` to get browser and modern JavaScript types.

**Case 2: Node.js API Servers**
Node.js servers use `"lib": ["ES2022"]` and `"types": ["node"]` to get Node.js types without browser types.

**Case 3: Universal (Isomorphic) Applications**
Universal apps use `"lib": ["ES2022", "DOM"]` and conditional imports to support both browser and server environments.

**Case 4: Web Workers**
Web Worker scripts use `"lib": ["ES2022", "WebWorker"]` instead of DOM to get worker-specific APIs.

### References

- TypeScript TSConfig Reference: `lib` — https://www.typescriptlang.org/tsconfig#lib
- Total TypeScript: Update tsconfig.json to Include DOM Typings — https://www.totaltypescript.com/workshops/typescript-pro-essentials/types-you-don't-control/dom-typing-configuration-in-typescript/solution
- TypeScript 4.5 Release Notes — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html
- Stack Overflow: Why web types are there by default — https://stackoverflow.com


## 2. Community-Maintained Typings and Managing `@types` Packages in `devDependencies`

### Definitions

**Core Definition**
Community-maintained typings are TypeScript declaration packages published under the `@types/*` scope on npm. They are sourced from the DefinitelyTyped repository—a community-maintained monorepo of type definitions for JavaScript packages that do not ship their own types. They should be installed in `devDependencies` because they are needed only at development time and are erased during compilation.

**Technical Definition**
The `@types/*` packages on npm are automatically generated from the DefinitelyTyped repository by a publishing bot. Each package contains `.d.ts` declaration files describing the API surface of a JavaScript library. Contributors submit pull requests, maintainers review them, and the bot pushes accepted changes to the `@types` scope. These packages contain no runtime code—only type declarations that are erased during compilation. The versioning of `@types/*` packages loosely follows the major and minor versions of the underlying library; keeping the first two version segments aligned is critical to avoid type drift. Since TypeScript 6.0, the `types` compiler option controls which `@types` packages are included in the global scope; without it, all visible `@types` packages are included.

**Beginner-Friendly Explanation**
Many JavaScript libraries don't have TypeScript types. Instead of writing types yourself, you can install a community-maintained `@types/*` package. For example, `npm install --save-dev @types/jest` gives you types for Jest. These packages come from DefinitelyTyped, a community project where volunteers write and maintain type definitions. They go in `devDependencies` because they're only needed during development—they don't affect the runtime behavior of your app. The version numbers matter: if you use Jest 29, install `@types/jest` in the 29 range. Otherwise, TypeScript might complain about APIs that don't match.

### Purposes

- To get type safety for JavaScript libraries that don't ship their own types.
- To enable IDE autocompletion and inline documentation for third-party libraries.
- To catch type errors when using library APIs incorrectly.
- To manage type dependencies separately from runtime dependencies.
- To ensure type definitions stay aligned with library versions.

### Syntax Rules and Structure

**General Syntax: Installing `@types` Packages**

```bash
npm install --save-dev @types/jest @types/node @types/express
```

**Component Breakdown**
- `--save-dev`: Installs to `devDependencies`.
- `@types/jest`, `@types/node`, `@types/express`: Type packages for Jest, Node.js, and Express.

**General Syntax: `package.json` Entry**

```json
{
  "devDependencies": {
    "@types/jest": "^29.5.0",
    "@types/node": "^20.0.0",
    "@types/express": "^4.17.0",
    "jest": "^29.7.0",
    "typescript": "^5.4.0"
  }
}
```

**Component Breakdown**
- `@types/*` packages are in `devDependencies`.
- Version ranges should align with the underlying library (e.g., `@types/jest` 29.x for Jest 29.x).

**General Syntax: Version Alignment**

```json
{
  "devDependencies": {
    "jest": "^29.7.0",
    "@types/jest": "^29.5.0"
  }
}
```

**Component Breakdown**
- Both are in the 29.x range, ensuring API compatibility.
- If Jest is upgraded to 30.x, `@types/jest` should also be upgraded to 30.x.

**Syntax Rules**

- Install `@types/*` packages as `devDependencies`, not `dependencies`.
- Version ranges should align with the underlying library's major and minor versions.
- If a library ships its own types (e.g., `zod`, `axios`), do NOT install `@types/*` for it.
- The `types` compiler option (see Section 4) controls which `@types` packages are included in the global scope.
- Since TypeScript 6.0, the default is to include all visible `@types` packages unless `types` is specified.
- Types-only packages can be a supply-chain risk because they are installed and executed in your build environment.

**Constraints and Limitations**

- Version drift between the library and `@types` package causes type errors or missed errors.
- Not all libraries have `@types` packages—some require custom declarations.
- Some libraries ship their own types and should not have `@types` installed.
- `@types` packages can become abandoned if maintainers stop updating them.
- Installing `@types` packages adds dependencies to your build environment, increasing supply-chain surface area.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Installing and Using `@types/jest`

```bash
# Step 1: Install Jest and its types.
npm install --save-dev jest ts-jest @types/jest typescript
```

```json
// Step 2: package.json devDependencies.
{
  "devDependencies": {
    "@types/jest": "^29.5.0",
    "jest": "^29.7.0",
    "ts-jest": "^29.1.0",
    "typescript": "^5.4.0"
  }
}
```

```typescript
// Step 3: Write a test with full type safety.
// src/math.test.ts
import { add } from "./math";

describe("add", () => {
  it("should add two numbers", () => {
    expect(add(1, 2)).toBe(3);  // ✅ Type-safe
  });

  it("should handle negatives", () => {
    expect(add(-1, -2)).toBe(-3);
  });
});
```

**Expected Output:** The test compiles and runs with full type checking.

**Why This Output Occurs:** `@types/jest` provides the type definitions for `describe`, `it`, `expect`, and `toBe`. Without it, TypeScript would report "Cannot find name 'describe'" and similar errors.

#### Example 2: Version Alignment

```json
// Step 1: Correct alignment — Jest 29 with @types/jest 29.
{
  "devDependencies": {
    "jest": "^29.7.0",
    "@types/jest": "^29.5.0"
  }
}
```

```json
// Step 2: Incorrect alignment — Jest 29 with @types/jest 28.
{
  "devDependencies": {
    "jest": "^29.7.0",
    "@types/jest": "^28.0.0"
  }
}
// ⚠️ Type errors: New APIs in Jest 29 are missing from @types/jest 28.
```

```typescript
// Step 3: The misalignment causes type errors.
// jest.mocked() was added in Jest 29 and is not in @types/jest 28.
const mockedFn = jest.mocked(myFunction);
// ❌ Error: Property 'mocked' does not exist on type 'typeof jest'.
```

**Expected Output:** Correct alignment compiles without errors; misalignment produces type errors.

**Why This Output Occurs:** The `@types/jest` version must align with the `jest` version to ensure that all APIs used at runtime are described in the type definitions.

### Real-World Cases

**Case 1: Node.js Projects**
Node.js projects install `@types/node` in `devDependencies` to get types for `fs`, `path`, `http`, `process`, etc.

**Case 2: Express Applications**
Express applications install `@types/express` and `@types/node` to get types for request/response objects and middleware.

**Case 3: React Projects**
React projects install `@types/react` and `@types/react-dom` to get types for JSX, hooks, and DOM APIs.

**Case 4: Testing Libraries**
Projects install `@types/jest` or `@types/vitest` to get types for testing globals.

### References

- @types/jest: What It Is and How to Use It Safely — https://safeguard.sh/resources/blog/types-jest
- TypeScript TSConfig Reference: `types` — https://www.typescriptlang.org/tsconfig#types
- DefinitelyTyped Repository — https://github.com/DefinitelyTyped/DefinitelyTyped
- Stack Overflow: Why NPM recommends @types packages as --save dependencies? — https://stackoverflow.com


## 3. Typing Untyped JavaScript Libraries and Writing Local Type Overrides

### Definitions

**Core Definition**
Typing untyped JavaScript libraries means writing local `.d.ts` declaration files to describe the shape of JavaScript code that has no TypeScript types and no `@types/*` package. Local type overrides use the `paths` compiler option to replace a library's bundled types with a custom declaration file.

**Technical Definition**
When a JavaScript library has no TypeScript types, TypeScript produces the `TS7016` error under `noImplicitAny`: "Could not find a declaration file for module 'library-name'." The solution is to create a local `.d.ts` file with a `declare module "library-name" { ... }` block that describes the library's exports. This scopes the type gap to the exact functions used, rather than casting the entire import to `any`. For libraries that have types but need modification, the `paths` compiler option can override the library's entry point with a local declaration file: `"redux": ["typings/redux"]`. Libraries with wildcard exports (e.g., CSS modules, SVGs) use `declare module "*.module.css" { ... }` to provide types for asset imports.

**Beginner-Friendly Explanation**
When you install a JavaScript library without TypeScript types, TypeScript complains. You have three options: (1) install an `@types/*` package if one exists, (2) write a local `.d.ts` file that describes the library, or (3) use a shorthand declaration (`declare module "library-name";`) that types everything as `any`. Option 2 is the best: you get some type safety without fully writing types for the whole library. You can also override a library's existing types if they're wrong or incomplete, using the `paths` compiler option to point to your own declaration file.

### Purposes

- To enable type checking for JavaScript libraries that lack TypeScript types.
- To provide IDE autocompletion and inline documentation for untyped libraries.
- To scope type gaps to specific APIs rather than casting the entire library to `any`.
- To override incorrect or incomplete types from a library's bundled declarations.
- To provide types for asset imports (CSS, SVG, images) in bundler environments.

### Syntax Rules and Structure

**General Syntax: Basic `declare module` Stub**

```typescript
// types/untyped-libs.d.ts
declare module "old-carousel-library" {
  export default class Carousel {
    constructor(element: HTMLElement, options?: any);
    next(): void;
    prev(): void;
    goTo(index: number): void;
    destroy(): void;
  }
}
```

**Component Breakdown**
- `declare module "old-carousel-library"`: Declares the shape of the module.
- `export default class Carousel`: Describes the default export.
- This scopes the type gap to the exact APIs used.

**General Syntax: Shorthand Declaration (All `any`)**

```typescript
// types/shorthand.d.ts
declare module "untyped-library";
// All imports have type `any` — quick fix, no type safety.
```

**Component Breakdown**
- No type information—everything is `any`.
- Useful as a temporary fix or for libraries with no types available.

**General Syntax: Wildcard Declaration for Assets**

```typescript
// types/assets.d.ts
declare module "*.module.css" {
  const classes: { [key: string]: string };
  export default classes;
}

declare module "*.svg" {
  import React from "react";
  const SVG: React.FC<React.SVGProps<SVGSVGElement>>;
  export default SVG;
}
```

**Component Breakdown**
- `*.module.css`: Matches CSS module imports, typing class names as strings.
- `*.svg`: Types SVG imports as React components (for Vite/webpack).

**General Syntax: Overriding Library Types via `paths`**

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "redux": ["typings/redux"]
    }
  }
}
```

**Component Breakdown**
- `"redux": ["typings/redux"]`: Overrides the `redux` module's types with a local declaration file.
- The local file `typings/redux.d.ts` provides custom types.

**Syntax Rules**

- Create `.d.ts` files in a `types/` directory and include them via `tsconfig.json` `include` or `typeRoots`.
- Use `declare module "library-name"` with the exact name used in `import` statements.
- Shorthand declarations (`declare module "library-name";`) type everything as `any`.
- Wildcard declarations (`declare module "*.css";`) handle asset imports.
- The `paths` compiler option can override a library's bundled types with a local declaration file.
- When the upstream library publishes types later, delete your local declaration to avoid drift.

**Constraints and Limitations**

- Handwritten declarations must be accurate—incorrect types cause runtime errors.
- Shorthand declarations provide no type safety.
- `paths` overrides can silently break if the library's structure changes.
- Local declarations for large libraries are time-consuming to write.
- Wildcard declarations can mask missing types for non-asset imports.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Typing an Untyped Library

```typescript
// Step 1: JavaScript library (no types).
// old-carousel-library.js
function Carousel(element, options) { }
Carousel.prototype.next = function() { };
Carousel.prototype.prev = function() { };
Carousel.prototype.goTo = function(index) { };
Carousel.prototype.destroy = function() { };
module.exports = Carousel;
```

```typescript
// Step 2: Local declaration file.
// types/old-carousel-library.d.ts
declare module "old-carousel-library" {
  export default class Carousel {
    constructor(element: HTMLElement, options?: { autoplay?: boolean; duration?: number });
    next(): void;
    prev(): void;
    goTo(index: number): void;
    destroy(): void;
  }
}
```

```typescript
// Step 3: Consumer uses the typed library.
// src/gallery.ts
import Carousel from "old-carousel-library";

const element = document.getElementById("gallery")!;
const carousel = new Carousel(element, { autoplay: true, duration: 5000 });
carousel.next();
carousel.goTo(2);
carousel.destroy();
```

**Expected Output:** The code compiles with type safety for `Carousel`.

**Why This Output Occurs:** The `declare module` block describes the shape of the JavaScript library. TypeScript uses this description for type checking, providing autocompletion and error detection.

#### Example 2: Overriding Library Types via `paths`

```json
// Step 1: Override Redux types with a local declaration.
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "redux": ["typings/redux"]
    }
  }
}
```

```typescript
// Step 2: Local override file.
// typings/redux.d.ts
export interface Action<T = any> {
  type: T;
}

export interface Store<S = any, A extends Action = Action> {
  getState(): S;
  dispatch(action: A): A;
}

export function createStore<S, A extends Action>(reducer: (state: S, action: A) => S): Store<S, A>;
```

```typescript
// Step 3: Consumer uses the overridden types.
import { createStore, Action, Store } from "redux";

const store: Store<{ count: number }> = createStore((state = { count: 0 }, action: Action) => {
  switch (action.type) {
    case "INCREMENT": return { count: state.count + 1 };
    default: return state;
  }
});

console.log(store.getState());  // { count: 0 }
```

**Expected Output:**
```
{ count: 0 }
```

**Why This Output Occurs:** The `paths` override replaces Redux's bundled types with the local declaration. The consumer uses the custom types, which may be simplified or tailored to the project's needs.

### Real-World Cases

**Case 1: Legacy JavaScript Libraries**
Projects integrating legacy libraries (jQuery plugins, old charting libraries) write local declarations to get type safety.

**Case 2: CSS Modules**
Projects using CSS modules write `declare module "*.module.css"` to type class names as strings.

**Case 3: SVG Imports**
React projects importing SVGs as components write `declare module "*.svg"` with `React.FC<React.SVGProps<SVGSVGElement>>`.

**Case 4: Overriding Incorrect Types**
When a library's bundled types are wrong or incomplete, the `paths` override provides corrected types without forking the library.

### References

- Steve Kinney: Modules and Declaration Files for React — https://stevekinney.com/courses/react-typescript/typescript-modules-declarations
- React Redux TypeScript Guide: Custom Type Definitions — https://raw.githubusercontent.com/steve-kim-96/react-redux-typescript-guide
- Stack Overflow: TypeScript 2 — custom typings for untyped npm module — https://stackoverflow.com
- TypeScript Handbook: Ambient Declarations — https://www.typescriptlang.org/docs/handbook/declaration-files/ambient-declarations.html


## 4. The `typeRoots` and `types` Configuration Settings for Controlling Global Scope Visibility

### Definitions

**Core Definition**
`typeRoots` and `types` are compiler options that control which `@types/*` packages are included in the global scope. `typeRoots` specifies the folders where type packages are located; `types` specifies which exact packages are included. They differ in granularity: `typeRoots` is folder-based, `types` is package-based. Controlling these options is a security and performance concern because `@types` packages enter your build environment and global scope.

**Technical Definition**
The `typeRoots` compiler option specifies an array of directories that TypeScript scans for type packages. Each subdirectory within a `typeRoot` is treated as a separate type package. By default, TypeScript scans `node_modules/@types` in all enclosing folders. If `typeRoots` is specified, the default is overwritten—`node_modules/@types` must be explicitly included or all `@types` packages are lost. The `types` compiler option specifies an exact list of `@types` packages to include in the global scope. When `types` is specified, only the listed packages are included; all others are excluded. This is the recommended approach for controlling global scope visibility, as it explicitly declares which packages are trusted to add globals.

**Beginner-Friendly Explanation**
`typeRoots` and `types` control which `@types` packages are automatically loaded into your project. By default, TypeScript loads ALL `@types` packages it can find in `node_modules/@types`. This is convenient but can be a security and performance problem: you might be loading types for packages you don't even use, and any `@types` package can add global declarations (like `describe` or `expect`) that pollute your global scope. `typeRoots` lets you say "look in these folders." `types` lets you say "include exactly these packages." The `types` option is more precise and is the recommended way to control which `@types` packages enter the global scope.

### Purposes

- To control which `@types` packages are included in the global scope.
- To reduce build-time overhead by excluding unused type packages.
- To reduce supply-chain risk by explicitly declaring trusted type packages.
- To support custom type package directories (e.g., a local `types/` folder).
- To prevent conflicts between global declarations from different packages.

### Syntax Rules and Structure

**General Syntax: `types` — Exact Package List**

```json
{
  "compilerOptions": {
    "types": ["node", "jest", "express"]
  }
}
```

**Component Breakdown**
- Only `@types/node`, `@types/jest`, and `@types/express` are included.
- All other `@types` packages in `node_modules/@types` are excluded.
- This is the recommended approach for global scope control.

**General Syntax: `typeRoots` — Folder-Based Scanning**

```json
{
  "compilerOptions": {
    "typeRoots": ["./node_modules/@types", "./types"]
  }
}
```

**Component Breakdown**
- Scans both `./node_modules/@types` and `./types` for type packages.
- Each subdirectory in these folders is treated as a package.
- The default `node_modules/@types` must be explicitly included, or it is overwritten.

**General Syntax: Combination — `typeRoots` + `types`**

```json
{
  "compilerOptions": {
    "typeRoots": ["./node_modules/@types", "./types"],
    "types": ["node", "jest"]
  }
}
```

**Component Breakdown**
- `typeRoots` specifies where to look.
- `types` specifies which packages to include from those locations.
- This is the most controlled configuration.

**Syntax Rules**

- `types` is an array of package names (without the `@types/` prefix).
- `typeRoots` is an array of directory paths.
- By default, all visible `@types` packages in `node_modules/@types` are included.
- When `types` is specified, only the listed packages are included in the global scope.
- When `typeRoots` is specified, the default `node_modules/@types` is overwritten unless explicitly included.
- Type packages for custom declarations should be placed in a folder included in `typeRoots`.
- Global declaration files in `typeRoots` folders should not have `export` statements (or they lose global scope).

**Constraints and Limitations**

- Overwriting `typeRoots` without including `node_modules/@types` loses all `@types` packages.
- `types` only controls global scope inclusion—it does not prevent `@types` packages from being used for module imports.
- Custom type packages must follow the package structure (a folder with `index.d.ts` or `package.json`).
- Global declaration files with `export` statements are treated as modules and do not pollute the global scope.
- Misconfigured `types` can cause "Cannot find name 'describe'" errors for testing globals.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Controlling Global Scope with `types`

```json
// Step 1: Without `types` — all @types packages are included.
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM"]
  }
}
// ⚠️ All @types packages in node_modules/@types are loaded, including unused ones.
```

```json
// Step 2: With `types` — only specified packages are included.
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM"],
    "types": ["node", "jest"]
  }
}
// ✅ Only @types/node and @types/jest are included in the global scope.
// ✅ Other @types packages (e.g., @types/lodash) are excluded.
```

```typescript
// Step 3: Testing globals from @types/jest are available.
describe("my test", () => {
  it("should work", () => {
    expect(true).toBe(true);
  });
});

// Step 4: Node.js globals from @types/node are available.
console.log(process.env.NODE_ENV);

// Step 5: Globals from excluded packages are NOT available.
// expect is from @types/jest (included) ✅
// describe is from @types/jest (included) ✅
// If @types/lodash provided globals, they would NOT be available.
```

**Expected Output:** The test and Node.js code compile; globals from excluded packages are not available.

**Why This Output Occurs:** The `types: ["node", "jest"]` option explicitly includes only `@types/node` and `@types/jest` in the global scope. All other `@types` packages are excluded, reducing global scope pollution and build-time overhead.

#### Example 2: Custom Type Packages with `typeRoots`

```json
// Step 1: Configure typeRoots to include custom types.
{
  "compilerOptions": {
    "typeRoots": ["./node_modules/@types", "./types"]
  }
}
```

```typescript
// Step 2: Custom type package structure.
// types/
// ├── global/
// │   └── index.d.ts
// └── legacy-lib/
//     └── index.d.ts
```

```typescript
// Step 3: Custom global declaration.
// types/global/index.d.ts
// No export statement — this is a global declaration.
interface Window {
  __APP_VERSION__: string;
  analytics: {
    track(event: string): void;
  };
}
```

```typescript
// Step 4: Custom module declaration.
// types/legacy-lib/index.d.ts
declare module "legacy-lib" {
  export function legacyFunction(input: string): number;
}
```

```typescript
// Step 5: Consumer uses the custom types.
// src/app.ts
window.__APP_VERSION__ = "1.0.0";  // ✅ Global type from custom package
window.analytics.track("page_view");

import { legacyFunction } from "legacy-lib";  // ✅ Custom module declaration
const result = legacyFunction("test");
console.log(result);
```

**Expected Output:** The custom global and module types are available.

**Why This Output Occurs:** The `typeRoots` option includes `./types`, which contains custom type packages. Each subdirectory is treated as a type package. The `global` package provides global declarations, and `legacy-lib` provides module declarations.

### Real-World Cases

**Case 1: Explicit Global Scope Control**
Projects with many `@types` packages use `types` to explicitly include only the packages that add globals (e.g., `node`, `jest`), reducing global scope pollution.

**Case 2: Custom Global Types**
Projects with custom global declarations (e.g., `window.__CONFIG__`) place them in a `types/` directory and include it via `typeRoots`.

**Case 3: Monorepo Type Management**
Monorepos use `typeRoots` to include a shared `types/` directory across packages, ensuring consistent global type definitions.

**Case 4: Security-Conscious Configuration**
Security-conscious projects use `types` to explicitly declare which `@types` packages are trusted, reducing supply-chain risk from unused type packages.

### References

- TypeScript TSConfig Reference: `types` — https://www.typescriptlang.org/tsconfig#types
- TypeScript TSConfig Reference: `typeRoots` — https://www.typescriptlang.org/tsconfig#typeRoots
- TypeScript Declaration File Management — https://raw.githubusercontent.com/Alessandro-Pang/fe-interview
- Stack Overflow: TypeScript how to declare global variable in every file? — https://stackoverflow.com


## 5. Configuring `skipLibCheck` and Its Critical Impact on Compiler Build Performance

### Definitions

**Core Definition**
`skipLibCheck` is a TypeScript compiler option that, when enabled, skips type checking of all declaration files (`.d.ts`). Since these files are pre-verified by their authors, checking them is redundant and wastes compilation time. Enabling `skipLibCheck` provides a 20–50% improvement in compilation speed and IDE performance.

**Technical Definition**
The `skipLibCheck` compiler option, when set to `true`, instructs TypeScript to bypass the type checking of declaration files (files with the `.d.ts` extension). This includes declaration files in `node_modules`, the built-in `lib.*.d.ts` files, and any other `.d.ts` files in the compilation. Since these files are authored and verified by library maintainers and the TypeScript team, checking them in every compilation is redundant. Disabling `skipLibCheck` (the default, `false`) forces TypeScript to verify every declaration file, which can add 50% or more to type-checking time and slow down IDE responsiveness. The Chromium project measured a ~29% build time improvement when enabling `skipLibCheck` (from ~24.5 seconds to ~17.4 seconds). Total TypeScript reports that without `skipLibCheck`, type-checking time increases by 50%, making the IDE 50% slower.

**Beginner-Friendly Explanation**
`skipLibCheck` is a performance optimization. By default, TypeScript checks the type correctness of ALL declaration files, including those in `node_modules` and the built-in DOM types. But these files are already checked by their authors—you don't need to check them again. Enabling `skipLibCheck` tells TypeScript "don't check the type definitions themselves, just use them." This can speed up compilation by 20–50% and make your IDE much more responsive. The downside is that if there's a bug in a library's type definitions, you won't catch it. But since you can't fix those types anyway, the trade-off is almost always worth it. Modern templates (Next.js, Vite) enable `skipLibCheck` by default.

### Purposes

- To significantly speed up TypeScript compilation (20–50% faster).
- To improve IDE responsiveness (autocompletion, error checking).
- To avoid type errors caused by incompatible declaration files in `node_modules`.
- To focus type checking on your own code rather than library internals.
- To work around declaration file incompatibilities between libraries.

### Syntax Rules and Structure

**General Syntax: Enabling `skipLibCheck`**

```json
{
  "compilerOptions": {
    "skipLibCheck": true
  }
}
```

**Component Breakdown**
- `"skipLibCheck": true`: Skips type checking of all `.d.ts` files.
- Default value is `false` (all declaration files are checked).

**General Syntax: Recommended Configuration**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,
    "strict": true
  }
}
```

**Component Breakdown**
- `skipLibCheck: true` is recommended for almost all projects.
- Modern frameworks (Next.js, Vite, CRA) enable it by default.

**General Syntax: When to Disable `skipLibCheck`**

```json
{
  "compilerOptions": {
    "skipLibCheck": false
  }
}
// Use only when:
// - Debugging type definition issues in your own .d.ts files
// - Auditing library type definitions for correctness
// - Working on the TypeScript compiler itself
```

**Component Breakdown**
- `skipLibCheck: false` is useful for debugging custom declaration files.
- Not recommended for general development.

**Syntax Rules**

- `skipLibCheck` is a boolean option (default: `false`).
- When `true`, TypeScript skips checking all `.d.ts` files.
- When `false`, TypeScript checks all `.d.ts` files, including `lib.*.d.ts` and `node_modules/@types`.
- The option does not affect type resolution—types are still used for checking your code.
- The option only skips checking the *internal correctness* of declaration files.
- As of TypeScript 6.0, `skipLibCheck` defaults to `true` in new projects.

**Constraints and Limitations**

- Enabling `skipLibCheck` may hide type errors in your own `.d.ts` files (not just libraries).
- It does not skip type checking of your `.ts` files—only `.d.ts` files.
- Some projects with custom declaration files may want to temporarily disable it to debug.
- The performance benefit is most pronounced in projects with many dependencies.
- The performance benefit is less significant in small projects with few dependencies.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Measuring the Performance Impact

```json
// Step 1: Configuration without skipLibCheck.
{
  "compilerOptions": {
    "skipLibCheck": false
  }
}
// Compilation time: ~24.5 seconds (Chromium measurement)
```

```json
// Step 2: Configuration with skipLibCheck.
{
  "compilerOptions": {
    "skipLibCheck": true
  }
}
// Compilation time: ~17.4 seconds (~29% improvement)
```

```bash
# Step 3: Measure your own project.
time npx tsc --noEmit  # With skipLibCheck: false
time npx tsc --noEmit  # With skipLibCheck: true
# Compare the results.
```

**Expected Output:** The `skipLibCheck: true` configuration compiles ~29% faster in large projects.

**Why This Output Occurs:** Without `skipLibCheck`, TypeScript must check every `.d.ts` file, including thousands of lines in `lib.dom.d.ts` and all `node_modules/@types` packages. With `skipLibCheck`, it skips these checks, focusing only on your code.

#### Example 2: Impact on IDE Responsiveness

```json
// Step 1: Without skipLibCheck.
{
  "compilerOptions": {
    "skipLibCheck": false
  }
}
// IDE type checking: ~50% slower
// Autocompletion: slower to appear
// Error checking: slower to update
```

```json
// Step 2: With skipLibCheck.
{
  "compilerOptions": {
    "skipLibCheck": true
  }
}
// IDE type checking: ~50% faster
// Autocompletion: near-instant
// Error checking: fast updates
```

**Expected Output:** The IDE is significantly more responsive with `skipLibCheck: true`.

**Why This Output Occurs:** The IDE runs TypeScript's type checker continuously. If `skipLibCheck` is disabled, every keystroke triggers re-checking of all declaration files, which is expensive. With `skipLibCheck: true`, the IDE only checks your code.

### Real-World Cases

**Case 1: Next.js Projects**
Next.js's default `tsconfig.json` includes `"skipLibCheck": true` to ensure fast compilation and IDE responsiveness.

**Case 2: Vite Projects**
Vite's React and Vue templates include `"skipLibCheck": true` by default.

**Case 3: Chromium**
The Chromium project measured a ~29% build time improvement when enabling `skipLibCheck` and is working to make it the default for TypeScript targets.

**Case 4: Large Monorepos**
Monorepos with hundreds of packages and thousands of dependencies benefit most from `skipLibCheck`, as it eliminates redundant checking of declaration files across all packages.

**Case 5: Library Development**
Library authors may temporarily disable `skipLibCheck` to verify the correctness of their own emitted `.d.ts` files.

### References

- Total TypeScript: Understanding skipLibCheck in TypeScript — https://www.totaltypescript.com/workshops/typescript-pro-essentials/types-you-don't-control/tsconfig-options-and-declaration-files/solution
- Chromium Issue 397737230: Investigate speeding up TypeScript invocations with skipLibCheck — https://issues.chromium.org/issues/397737230
- dot-skills: skipLibCheck performance reference — https://raw.githubusercontent.com/pproenca/dot-skills
- TypeScript TSConfig Reference: `skipLibCheck` — https://www.typescriptlang.org/tsconfig#skipLibCheck


## Summary: Third-Party Type Definition Configuration Comparison

| Option | Type | Default | Purpose | Performance Impact | Security Impact |
|---|---|---|---|---|---|
| `lib` | Array of library names | Based on `target` | Include built-in JS/DOM types | Minimal | None |
| `types` | Array of package names | All visible `@types` | Control which `@types` enter global scope | Moderate | High (limits supply-chain) |
| `typeRoots` | Array of folder paths | `node_modules/@types` | Control where type packages are found | Moderate | High (limits lookup locations) |
| `skipLibCheck` | Boolean | `false` (TS < 6.0) | Skip checking `.d.ts` files | High (20–50% faster) | Low (hides library type bugs) |
| `paths` | Object mapping | `{}` | Override module resolution (incl. type overrides) | Minimal | Low (can bypass library types) |
| `@types` packages | npm packages | N/A | Community-maintained library types | Moderate | High (build environment trust) |
| Local `.d.ts` files | Files in `types/` | N/A | Custom types for untyped libraries | Minimal | None |


## References

- TypeScript TSConfig Reference: `lib` — https://www.typescriptlang.org/tsconfig#lib
- TypeScript TSConfig Reference: `types` — https://www.typescriptlang.org/tsconfig#types
- TypeScript TSConfig Reference: `typeRoots` — https://www.typescriptlang.org/tsconfig#typeRoots
- TypeScript TSConfig Reference: `skipLibCheck` — https://www.typescriptlang.org/tsconfig#skipLibCheck
- TypeScript TSConfig Reference: `paths` — https://www.typescriptlang.org/tsconfig#paths
- TypeScript Handbook: Declaration Files — https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html
- TypeScript Handbook: Ambient Declarations — https://www.typescriptlang.org/docs/handbook/declaration-files/ambient-declarations.html
- TypeScript 4.5 Release Notes (DOM library overrides) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html
- Total TypeScript: Update tsconfig.json to Include DOM Typings — https://www.totaltypescript.com/workshops/typescript-pro-essentials/types-you-don't-control/dom-typing-configuration-in-typescript/solution
- Total TypeScript: Understanding skipLibCheck in TypeScript — https://www.totaltypescript.com/workshops/typescript-pro-essentials/types-you-don't-control/tsconfig-options-and-declaration-files/solution
- @types/jest: What It Is and How to Use It Safely — https://safeguard.sh/resources/blog/types-jest
- Steve Kinney: Modules and Declaration Files for React — https://stevekinney.com/courses/react-typescript/typescript-modules-declarations
- Chromium Issue 397737230: Investigate speeding up TypeScript invocations with skipLibCheck — https://issues.chromium.org/issues/397737230
- dot-skills: skipLibCheck performance reference — https://raw.githubusercontent.com/pproenca/dot-skills
- TypeScript Declaration File Management — https://raw.githubusercontent.com/Alessandro-Pang/fe-interview
- React Redux TypeScript Guide: Custom Type Definitions — https://raw.githubusercontent.com/steve-kim-96/react-redux-typescript-guide
- DefinitelyTyped Repository — https://github.com/DefinitelyTyped/DefinitelyTyped
- Stack Overflow: Why web types are there by default — https://stackoverflow.com
- Stack Overflow: TypeScript 2 — custom typings for untyped npm module — https://stackoverflow.com
- Stack Overflow: TypeScript how to declare global variable in every file? — https://stackoverflow.com