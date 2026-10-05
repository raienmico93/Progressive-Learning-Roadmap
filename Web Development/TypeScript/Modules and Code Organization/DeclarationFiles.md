# TypeScript Declaration Files: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A declaration file is a TypeScript file with the `.d.ts` extension that contains only type information—no executable code. It describes the shape of JavaScript code (whether from a library, a global script, or your own compiled output) so that TypeScript can type-check code that imports or uses it.

**Technical Definition**
Declaration files (`.d.ts`) are TypeScript source files that contain only *ambient* declarations—type-level constructs that describe existing runtime values without providing implementations. The TypeScript compiler uses them during type checking but never emits JavaScript from them. They follow the same grammar as `.ts` files but are restricted to declarations (no statements, no runtime expressions). Declaration files serve three primary purposes: (1) describing the types of external JavaScript libraries, (2) augmenting or modifying existing types via declaration merging, and (3) providing type information for compiled output when the `declaration` compiler option is enabled.

**Beginner-Friendly Explanation**
A declaration file is a "type-only" file that tells TypeScript what shape some JavaScript code has. If you're using a JavaScript library that wasn't written in TypeScript, you write a `.d.ts` file that describes its functions, classes, and variables—without any actual implementation. Think of it as a contract: "This library has a function called `add` that takes two numbers and returns a number." TypeScript reads this contract and checks your code against it. Declaration files never produce JavaScript output; they exist purely to help TypeScript understand code that it can't see the implementation of.

### Key Characteristics

- **Type-only**: `.d.ts` files contain no executable code and emit no JavaScript.
- **Ambient**: Declarations describe code that exists elsewhere (libraries, globals, compiled output).
- **Non-emitting**: The compiler never generates `.js` from `.d.ts` files.
- **Mergeable**: Declaration files participate in declaration merging (interfaces, namespaces, modules).
- **Three purposes**: Describe external libraries, augment existing types, and publish types for compiled output.
- **Convention**: Named after the library or module they describe (e.g., `lodash.d.ts`, `express.d.ts`).

### Prerequisites

- Basic knowledge of TypeScript types (interfaces, type aliases, generics)
- Familiarity with ES module syntax (`import`/`export`)
- Understanding of TypeScript's compilation model
- Familiarity with `tsconfig.json` compiler options

### Related Programming Areas

- **TypeScript Compiler**: The `declaration`, `declarationDir`, and `declarationMap` options
- **DefinitelyTyped**: The community repository for `@types/*` packages
- **Module Augmentation**: Extending third-party module types
- **Ambient Declarations**: Declaring globals and module shapes
- **Declaration Merging**: Combining multiple declarations with the same name

### Core Concepts / Features

1. `.d.ts` File Anatomy, Purpose, and Relationship to Output JavaScript
2. Declaring External Non-TypeScript Libraries
3. Ambient Declarations and Configuring Global Environment Additions (`declare global`)
4. Module Declarations and Module Augmentation (Patching Existing Third-Party Types)
5. Triple-Slash Directives (`/// <reference path="..." />`)


## 1. `.d.ts` File Anatomy, Purpose, and Relationship to Output JavaScript

### Definitions

**Core Definition**
A `.d.ts` file is a declaration file that contains only type information—no runtime code. It describes the shape of JavaScript code that exists elsewhere, whether from a compiled TypeScript project, a third-party library, or a global script. The compiler uses these files for type checking but never emits JavaScript from them.

**Technical Definition**
Declaration files are TypeScript source files with the `.d.ts` extension. They are a strict subset of implementation files (`.ts`): they contain only declarations (interfaces, type aliases, `declare` statements, module declarations) and no executable statements. When the `declaration` compiler option is enabled, TypeScript emits a `.d.ts` file alongside each `.js` output, mirroring the public API of the corresponding `.ts` file. These emitted files are the "contract" that consumers of a compiled package use for type checking. In addition to being *emitted* by the compiler, declaration files can be *handwritten* to describe libraries that have no TypeScript source.

**Beginner-Friendly Explanation**
A `.d.ts` file is like a user manual for JavaScript code. It doesn't contain the actual implementation—just the type information: "this function takes a string and returns a number," "this class has a `name` property." When you compile TypeScript with `declaration: true`, the compiler generates a `.d.ts` file next to each `.js` file. These `.d.ts` files are what other TypeScript projects use to understand your code's types without reading your implementation. You can also write `.d.ts` files by hand to describe JavaScript libraries that don't have TypeScript types.

### Purposes

- To provide type information for JavaScript code that lacks TypeScript types.
- To publish type declarations alongside compiled JavaScript (via `declaration: true`).
- To enable type checking across package boundaries without exposing implementation.
- To document the public API surface of a library or module.
- To support IDE autocompletion and inline documentation for external libraries.

### Syntax Rules and Structure

**General Syntax: Emitted Declaration File (Generated by Compiler)**

```typescript
// math.ts (source)
export function add(a: number, b: number): number {
  return a + b;
}
export interface Point {
  x: number;
  y: number;
}
```

```typescript
// math.d.ts (emitted by tsc --declaration)
export declare function add(a: number, b: number): number;
export interface Point {
  x: number;
  y: number;
}
```

**Component Breakdown**
- `export declare function`: The `declare` keyword indicates the function exists elsewhere.
- `export interface`: Interfaces are already type-only and are emitted as-is.
- The `.d.ts` file contains no implementation—only signatures.

**General Syntax: Handwritten Declaration File**

```typescript
// lodash.d.ts
declare module "lodash" {
  export function chunk<T>(array: T[], size: number): T[][];
  export function debounce(func: Function, wait: number): Function;
}
```

**Component Breakdown**
- `declare module "lodash"`: Declares the shape of the `lodash` module.
- `export function`: Describes the module's exports.

**Syntax Rules**

- `.d.ts` files contain only declarations, never executable code.
- The `declare` keyword is used for values (variables, functions, classes) that exist at runtime.
- Interfaces and type aliases are inherently type-only and do not require `declare`.
- The compiler emits `.d.ts` files when `declaration: true` is set in `tsconfig.json`.
- `declarationDir` controls where `.d.ts` files are emitted.
- `declarationMap: true` enables go-to-definition for consumers.
- `.d.ts` files are never emitted as `.js` files.

**Constraints and Limitations**

- `.d.ts` files cannot contain runtime code (function bodies, variable initializers).
- Handwritten `.d.ts` files must accurately reflect the runtime behavior of the described code.
- Emitted `.d.ts` files may omit private members (depending on `stripInternal`).
- Declaration files for complex generic libraries can be difficult to write by hand.
- Incorrect declarations cause type errors in consumers without runtime protection.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Compiler-Emitted Declaration File

```typescript
// Step 1: Source file with public API.
// src/calculator.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export interface CalculationResult {
  value: number;
  operation: string;
}

export function calculate(
  a: number,
  b: number,
  operation: "add" | "subtract"
): CalculationResult {
  const value = operation === "add" ? add(a, b) : subtract(a, b);
  return { value, operation };
}
```

```json
// Step 2: Enable declaration emission.
// tsconfig.json
{
  "compilerOptions": {
    "declaration": true,
    "declarationDir": "./dist/types",
    "outDir": "./dist"
  }
}
```

```typescript
// Step 3: Emitted declaration file.
// dist/types/calculator.d.ts
export declare function add(a: number, b: number): number;
export declare function subtract(a: number, b: number): number;
export interface CalculationResult {
  value: number;
  operation: string;
}
export declare function calculate(
  a: number,
  b: number,
  operation: "add" | "subtract"
): CalculationResult;
```

```typescript
// Step 4: Consumer uses the declaration file.
// app.ts
import { calculate, CalculationResult } from "./dist/types/calculator";

const result: CalculationResult = calculate(5, 3, "add");
console.log(result.value);  // 8
```

**Expected Output:**
```
8
```

**Why This Output Occurs:** The compiler emits `calculator.d.ts` alongside `calculator.js`. The declaration file contains the signatures of the exported functions and interfaces. The consumer imports from the declaration file, and TypeScript type-checks against it.

#### Example 2: Handwritten Declaration for a JavaScript Library

```typescript
// Step 1: JavaScript library (no types).
// legacy-charts.js
function renderChart(element, data, options) {
  // implementation...
}
function updateChart(chart, newData) {
  // implementation...
}
module.exports = { renderChart, updateChart };
```

```typescript
// Step 2: Handwritten declaration file.
// types/legacy-charts.d.ts
declare module "legacy-charts" {
  export interface ChartOptions {
    width?: number;
    height?: number;
    color?: string;
  }

  export interface Chart {
    id: string;
    update(data: unknown[]): void;
  }

  export function renderChart(
    element: HTMLElement,
    data: unknown[],
    options?: ChartOptions
  ): Chart;

  export function updateChart(chart: Chart, newData: unknown[]): void;
}
```

```typescript
// Step 3: Consumer uses the typed library.
// app.ts
import { renderChart, Chart, ChartOptions } from "legacy-charts";

const options: ChartOptions = { width: 800, height: 600 };
const chart: Chart = renderChart(document.getElementById("chart")!, [1, 2, 3], options);
console.log(chart.id);
```

**Expected Output:** The chart renders and `chart.id` is logged (assuming the library is loaded).

**Why This Output Occurs:** The handwritten `legacy-charts.d.ts` file describes the shape of the JavaScript library. TypeScript uses this description for type checking, even though the library has no TypeScript source.

### Real-World Cases

**Case 1: npm Package Publishing**
Libraries compiled with `declaration: true` ship `.d.ts` files alongside `.js` files. Consumers of the package get full type safety without reading the implementation.

**Case 2: DefinitelyTyped**
The `@types/*` packages on npm are declaration files maintained by the community for JavaScript libraries that don't ship their own types.

**Case 3: Legacy Code Migration**
When migrating JavaScript to TypeScript incrementally, handwritten `.d.ts` files provide types for un-migrated modules, enabling gradual migration.

**Case 4: Internal Shared Libraries**
Monorepo packages emit declaration files that other packages consume for type checking, enabling cross-package type safety without source access.

### References

- TypeScript Handbook: Type Declarations — https://www.typescriptlang.org/docs/handbook/2/type-declarations.html
- TypeScript Documentation: Declaration Files Introduction — https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html
- TypeScript TSConfig: `declaration` — https://www.typescriptlang.org/tsconfig#declaration
- TypeScript TSConfig: `declarationMap` — https://www.typescriptlang.org/tsconfig#declarationMap


## 2. Declaring External Non-TypeScript Libraries

### Definitions

**Core Definition**
Declaring external non-TypeScript libraries means writing a declaration file (`.d.ts`) that describes the shape of a JavaScript library that has no TypeScript types. This enables TypeScript to type-check code that imports and uses the library, even though the library's source is plain JavaScript.

**Technical Definition**
External library declarations use ambient module declarations (`declare module "library-name" { ... }`) to describe the exports and shape of a JavaScript module. The declaration file can be placed in a `types/` directory and referenced via `tsconfig.json` `typeRoots` or `include`. For libraries with complex APIs, the declaration file mirrors the library's structure with interfaces, type aliases, and function signatures. When a library has no types and no declaration file, TypeScript produces a `TS7016` error ("Could not find a declaration file"). A shorthand ambient module declaration (`declare module "library-name";`) suppresses the error by typing all imports as `any`.

**Beginner-Friendly Explanation**
If you're using a JavaScript library that doesn't have TypeScript types, TypeScript will complain: "Could not find a declaration file for module 'some-library'." To fix this, you write a `.d.ts` file that describes what the library exports. For example, if the library has a function `renderChart` that takes an HTML element and an array, you write `declare module "legacy-charts" { export function renderChart(element: HTMLElement, data: unknown[]): void; }`. Now TypeScript knows the shape of the library and can type-check your code. If you don't want to write full types, you can use a shorthand declaration (`declare module "some-library";`) that types everything as `any`.

### Purposes

- To enable type checking for JavaScript libraries that lack TypeScript types.
- To provide IDE autocompletion and inline documentation for external libraries.
- To catch type errors when using untyped libraries.
- To serve as a migration path from JavaScript to TypeScript.
- To document the expected usage of a JavaScript library.

### Syntax Rules and Structure

**General Syntax: Ambient Module Declaration**

```typescript
declare module "library-name" {
  export function someFunction(param: ParamType): ReturnType;
  export class SomeClass {
    constructor(param: string);
    method(): number;
  }
  export interface SomeInterface {
    property: string;
  }
}
```

**Component Breakdown**
- `declare module "library-name"`: Declares the shape of the module.
- `export function`: Describes exported functions.
- `export class`: Describes exported classes.
- `export interface`: Describes exported types.

**General Syntax: Shorthand Ambient Module Declaration**

```typescript
declare module "library-name";
// All imports from this module have type `any`.
```

**Component Breakdown**
- No type information—all imports are `any`.
- Useful as a quick fix or for libraries with no types available.

**General Syntax: Wildcard Module Declaration**

```typescript
declare module "*.css";
declare module "*.svg" {
  const content: string;
  export default content;
}
```

**Component Breakdown**
- `*.css`: Matches any import ending in `.css`.
- `*.svg`: Provides a default export type for SVG imports.

**Syntax Rules**

- Declaration files for external libraries should be placed in a `types/` directory.
- The `tsconfig.json` `include` or `typeRoots` must include the declaration file.
- `declare module` uses the exact module name as it appears in `import` statements.
- Exports inside the module declaration mirror the library's public API.
- Shorthand declarations type everything as `any` (no type safety).
- Wildcard declarations handle asset imports (CSS, images, SVGs).
- For libraries with `export =` (CommonJS), use `export =` in the declaration.

**Constraints and Limitations**

- Handwritten declarations must be accurate—incorrect types cause runtime errors.
- Shorthand declarations provide no type safety.
- Wildcard declarations can mask missing types for non-asset imports.
- Maintaining declarations for large libraries is time-consuming.
- Declaration files for libraries with complex generics are difficult to write correctly.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Ambient Module Declaration for a Library

```typescript
// Step 1: JavaScript library (no types).
// simple-charts.js
function renderChart(element, data, options) {
  // ...
}
function updateChart(chart, newData) {
  // ...
}
module.exports = { renderChart, updateChart };
```

```typescript
// Step 2: Declaration file.
// types/simple-charts.d.ts
declare module "simple-charts" {
  export interface ChartOptions {
    width?: number;
    height?: number;
    color?: string;
  }

  export interface Chart {
    id: string;
    update(data: unknown[]): void;
  }

  export function renderChart(
    element: HTMLElement,
    data: unknown[],
    options?: ChartOptions
  ): Chart;

  export function updateChart(chart: Chart, newData: unknown[]): void;
}
```

```json
// Step 3: Include the declaration file in tsconfig.json.
{
  "compilerOptions": {
    "typeRoots": ["./node_modules/@types", "./types"]
  },
  "include": ["src/**/*", "types/**/*"]
}
```

```typescript
// Step 4: Consumer uses the library with types.
import { renderChart, Chart, ChartOptions } from "simple-charts";

const options: ChartOptions = { width: 800, height: 600 };
const chart: Chart = renderChart(document.getElementById("chart")!, [1, 2, 3], options);
chart.update([4, 5, 6]);
console.log(chart.id);
```

**Expected Output:** The chart renders and updates; `chart.id` is logged.

**Why This Output Occurs:** The declaration file `simple-charts.d.ts` describes the shape of the JavaScript library. TypeScript uses it for type checking, providing autocompletion and error detection.

#### Example 2: Shorthand and Wildcard Declarations

```typescript
// Step 1: Shorthand declaration for a library with no types.
// types/untyped-library.d.ts
declare module "untyped-library";
// All imports from "untyped-library" have type `any`.

// Step 2: Wildcard declarations for assets.
// types/assets.d.ts
declare module "*.css";
declare module "*.svg" {
  const content: string;
  export default content;
}
declare module "*.png" {
  const content: string;
  export default content;
}

// Step 3: Consumer uses the untyped library.
import { something } from "untyped-library";
// something is `any` — no type safety, but no error.

// Step 4: Consumer imports CSS and SVG.
import "./styles.css";  // ✅ No error
import logo from "./logo.svg";  // logo: string
console.log(logo);  // "/assets/logo.svg"
```

**Expected Output:**
```
/assets/logo.svg
```

**Why This Output Occurs:** The shorthand declaration suppresses the "Could not find declaration file" error by typing everything as `any`. The wildcard declarations provide default export types for asset imports, enabling type-safe asset handling.

### Real-World Cases

**Case 1: Legacy JavaScript Libraries**
Projects using legacy JavaScript libraries (jQuery, Moment.js, etc.) use `@types/*` packages or handwritten declaration files to get type safety.

**Case 2: CSS Modules**
Projects using CSS modules write `declare module "*.module.css"` to type CSS class names as strings.

**Case 3: Image and SVG Imports**
Projects importing SVGs or images write `declare module "*.svg"` or `declare module "*.png"` to type asset imports as strings or React components.

**Case 4: Environment Variables**
Projects write `declare module "*.env"` or use `declare const` to type environment variables injected by bundlers.

### References

- TypeScript Handbook: Library Structures — https://www.typescriptlang.org/docs/handbook/declaration-files/library-structures.html
- TypeScript Handbook: Ambient Declarations — https://www.typescriptlang.org/docs/handbook/declaration-files/ambient-declarations.html
- TypeScript Handbook: Shorthand Ambient Modules — https://www.typescriptlang.org/docs/handbook/declaration-files/by-example.html
- DefinitelyTyped — https://github.com/DefinitelyTyped/DefinitelyTyped


## 3. Ambient Declarations and Configuring Global Environment Additions (`declare global`)

### Definitions

**Core Definition**
Ambient declarations are type-level descriptions of code that exists at runtime but has no TypeScript implementation. `declare global` is a special ambient declaration that adds or modifies types in the global scope from within a module file. It enables augmenting global types (e.g., `Window`, `Array`, `String`) and declaring global variables from module files.

**Technical Definition**
Ambient declarations use the `declare` keyword to describe existing runtime values without providing implementations. They appear in `.d.ts` files or in `.ts`/`.tsx` files. `declare global` is a global augmentation construct that must be used inside a module (a file with at least one top-level `import` or `export`). It adds declarations to the global scope—typically augmenting existing global interfaces like `Window`, `Array`, or `String.prototype`, or declaring new global variables. A file containing `declare global` must be a module (use `export {}` if necessary). Global augmentations are often used for project-specific globals, third-party script integrations, and extending built-in types.

**Beginner-Friendly Explanation**
Ambient declarations tell TypeScript about things that exist at runtime but aren't in your TypeScript code. For example, if a script adds a `DEBUG` variable to `window`, you declare it: `declare const DEBUG: boolean;`. `declare global` is a special form used inside module files to add things to the global scope. For example, if a third-party analytics script adds `window.analytics`, you write `declare global { interface Window { analytics: { track(event: string): void } } }` in a module file. This tells TypeScript that `window.analytics` exists and has a `track` method. The key rule: `declare global` only works in modules (files with imports/exports).

### Purposes

- To declare global variables, functions, and classes that exist at runtime.
- To augment global interfaces (`Window`, `Array`, `String`) from module files.
- To integrate third-party scripts that add globals.
- To provide types for environment-specific globals (e.g., `process.env` in Node.js).
- To extend built-in types with project-specific methods.

### Syntax Rules and Structure

**General Syntax: Ambient Variable/Function/Class Declaration**

```typescript
declare const VERSION: string;
declare function log(message: string): void;
declare class User {
  name: string;
  constructor(name: string);
  greet(): string;
}
```

**Component Breakdown**
- `declare const/let/var`: Declares a global variable.
- `declare function`: Declares a global function.
- `declare class`: Declares a global class.

**General Syntax: `declare global` (Module Context)**

```typescript
export {};  // Makes this file a module

declare global {
  interface Window {
    analytics: {
      track(event: string, props?: Record<string, unknown>): void;
      identify(userId: string): void;
    };
  }

  var DEBUG: boolean;
  function gtag(command: string, ...args: unknown[]): void;
}
```

**Component Breakdown**
- `export {}`: Makes the file a module (required for `declare global`).
- `interface Window`: Augments the global `Window` interface.
- `var DEBUG`: Declares a global variable.
- `function gtag`: Declares a global function.

**General Syntax: Global Declaration in `.d.ts` File (No `declare global` Needed)**

```typescript
// window.d.ts
interface Window {
  X: number;
}
// No `declare global` needed—everything in .d.ts is global by default.
```

**Component Breakdown**
- In `.d.ts` files, top-level interfaces automatically go into the global scope.

**Syntax Rules**

- `declare` is used for runtime values (variables, functions, classes).
- Interfaces and type aliases do not require `declare`—they are type-only.
- `declare global` must be inside a module (a file with `import`/`export`).
- Add `export {}` to make a file a module if it has no imports/exports.
- In `.d.ts` files, top-level declarations are global by default (no `declare global` needed).
- Global augmentations use interface merging to add members to existing interfaces.
- `declare global` cannot be used in ambient module declarations (only in module files).

**Constraints and Limitations**

- `declare global` only works in modules—using it in a script file is an error.
- Global augmentations can conflict with other libraries' augmentations.
- Overusing global declarations can lead to naming conflicts and maintenance issues.
- Global augmentations cannot change existing property types (only add).
- `declare const` at the top level of a module does not create a global—it only declares a local ambient binding.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `declare global` for Window Augmentation

```typescript
// Step 1: Module file with global augmentation.
// src/types/global.d.ts
export {};  // Makes this a module

declare global {
  interface Window {
    analytics: {
      track(event: string, props?: Record<string, unknown>): void;
      identify(userId: string): void;
    };
    __APP_VERSION__: string;
  }
}

// Step 2: Consumer uses the augmented Window.
// src/app.ts
window.analytics.track("page_view", { page: "/home" });
window.analytics.identify("user-123");
console.log(window.__APP_VERSION__);  // "1.0.0"

// Step 3: TypeScript now knows about window.analytics and window.__APP_VERSION__.
// Without the augmentation, these would produce TS2339 errors.
```

**Expected Output:**
```
1.0.0
```

**Why This Output Occurs:** The `declare global` block adds `analytics` and `__APP_VERSION__` to the global `Window` interface. TypeScript now recognizes these properties, enabling type-safe access.

#### Example 2: Ambient Declarations in `.d.ts` Files

```typescript
// Step 1: Ambient declaration file (no declare global needed).
// types/globals.d.ts
declare const VERSION: string;
declare function log(message: string): void;
declare class MathHelper {
  static random(): number;
  static round(value: number, precision: number): number;
}

interface Window {
  customProperty: string;
}
```

```typescript
// Step 2: Consumer uses the globals.
// src/app.ts
console.log(VERSION);  // "1.0.0"
log("Application started");
console.log(MathHelper.random());
console.log(MathHelper.round(3.14159, 2));  // 3.14
window.customProperty = "hello";

// Step 3: All globals are available without imports.
```

**Expected Output:**
```
1.0.0
3.14
```

**Why This Output Occurs:** The `.d.ts` file declares global values and augments the `Window` interface. In `.d.ts` files, top-level declarations automatically go into the global scope, so no `declare global` wrapper is needed.

### Real-World Cases

**Case 1: Third-Party Analytics Scripts**
Google Analytics, Segment, and Mixpanel add global objects to `window`. `declare global` provides types for these objects.

**Case 2: Environment-Specific Globals**
Node.js applications declare `process.env` variables, and browser applications declare `window.__CONFIG__` objects.

**Case 3: Build-Time Constants**
Bundlers inject build-time constants (e.g., `__DEV__`, `__VERSION__`). Ambient declarations provide types for these constants.

**Case 4: Extending Built-in Types**
Projects extend `Array.prototype` or `String.prototype` with custom methods. `declare global` augments the built-in interfaces.

**Case 5: Test Globals**
Testing frameworks add globals like `describe`, `it`, and `expect`. Declaration files provide types for these globals.

### References

- TypeScript Handbook: Global Augmentation — https://www.typescriptlang.org/docs/handbook/declaration-merging.html#global-augmentation
- TypeScript Handbook: Ambient Declarations — https://www.typescriptlang.org/docs/handbook/declaration-files/ambient-declarations.html
- Total TypeScript: How to Properly Type Window — https://www.totaltypescript.com/how-to-properly-type-window
- TypeScript Handbook: Global .d.ts Template — https://www.typescriptlang.org/docs/handbook/declaration-files/templates/global-d-ts.html


## 4. Module Declarations and Module Augmentation (Patching Existing Third-Party Types)

### Definitions

**Core Definition**
Module augmentation is a TypeScript feature that allows you to reopen a third-party module's types from your own code and add new properties, methods, or types to its existing interfaces. It uses the `declare module "module-name" { ... }` syntax and relies on declaration merging to combine your additions with the original types.

**Technical Definition**
Module augmentation (also called "module expansion") lets you extend a third-party module's type definitions without modifying the library's source. The augmentation file must be a module (a file with at least one top-level `import` or `export`). You import the module you want to augment (a side-effect import like `import "express"`), then use `declare module "module-name" { ... }` to add or modify its types. Declaration merging combines your additions with the original interface. This is the primary mechanism for adding project-specific properties to library types—such as adding `userId` to Express's `Request` interface after authentication middleware attaches it.

**Beginner-Friendly Explanation**
Module augmentation lets you add types to a third-party library without editing the library. For example, if you use Express and your authentication middleware adds a `userId` property to `req`, TypeScript doesn't know about it. You write a `.d.ts` file that says `declare module "express-serve-static-core" { interface Request { userId?: string } }`. Now TypeScript knows that `req.userId` exists. The key rules: the augmentation file must be a module (add `import "express"` at the top), and you augment the *implementation* module (e.g., `express-serve-static-core`), not the re-export module (`express`). This is one of the most powerful patterns for integrating third-party libraries with your application's types.

### Purposes

- To add project-specific properties to third-party library types.
- To extend library interfaces without modifying the library's source.
- To integrate middleware and plugins that add runtime properties.
- To narrow or refine library types for project-specific usage.
- To avoid unsafe type assertions (`as`) scattered throughout the codebase.

### Syntax Rules and Structure

**General Syntax: Module Augmentation**

```typescript
// types/express.d.ts
import "express";  // Side-effect import to make this a module

declare module "express-serve-static-core" {
  interface Request {
    userId?: string;
    requestId: string;
    tenant?: {
      id: string;
      plan: "free" | "pro" | "enterprise";
    };
  }
}
```

**Component Breakdown**
- `import "express"`: Makes the file a module (required for augmentation).
- `declare module "express-serve-static-core"`: Reopens the module's types.
- `interface Request`: Declaration-merges with the original `Request` interface.

**General Syntax: Augmenting a Re-Exported Module**

```typescript
// types/inertia.d.ts
import "@inertiajs/core";

declare module "@inertiajs/core" {
  export interface InertiaConfig {
    sharedPageProps: {
      auth: { user: { id: number; name: string } };
      flash: { success?: string; error?: string };
    };
  }
}
```

**Component Breakdown**
- `import "@inertiajs/core"`: Side-effect import.
- `declare module "@inertiajs/core"`: Augments the module's types.

**General Syntax: `export {}` to Make a File a Module**

```typescript
// types/window.d.ts
export {};  // Makes this a module

declare global {
  interface Window {
    analytics: { track(event: string): void };
  }
}
```

**Component Breakdown**
- `export {}` is an alternative to a side-effect import for making a file a module.

**Syntax Rules**

- The augmentation file must be a module (has `import` or `export`).
- Use `import "module-name"` (side-effect import) or `export {}` to make the file a module.
- `declare module "module-name"` reopens the module's types.
- Augmented interfaces must already exist in the original module.
- Augmented properties should be optional (`?`) if they're added conditionally (e.g., by middleware).
- Augment the implementation module, not the re-export module (e.g., `express-serve-static-core`, not `express`).
- Classes cannot be augmented—only interfaces and namespaces.

**Constraints and Limitations**

- Augmentation only *adds* members—it cannot remove or change existing types.
- The augmentation file must be included in the compilation (via `include` or `files`).
- Augmenting the wrong module silently no-ops (e.g., augmenting `express` instead of `express-serve-static-core`).
- Required properties that are actually set by middleware mid-pipeline can make early-pipeline code lie about its state.
- Declaration merging conflicts can occur when multiple augmentations define the same property.
- Module augmentation does not work with `export =`-style CommonJS modules in all configurations.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Augmenting Express Request

```typescript
// Step 1: Auth middleware adds `userId` to req.
// src/middleware/auth.ts
import { Request, Response, NextFunction } from "express";

export function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization;
  if (token) {
    (req as any).userId = "user-123";  // Attaches userId
  }
  next();
}
```

```typescript
// Step 2: Module augmentation for Express Request.
// src/types/express.d.ts
import "express";

declare module "express-serve-static-core" {
  interface Request {
    userId?: string;
    requestId: string;
  }
}
```

```typescript
// Step 3: Consumer uses req.userId without casts.
// src/routes/user.ts
import { Router } from "express";

const router = Router();

router.get("/me", (req, res) => {
  const userId = req.userId;  // string | undefined — recognized!
  const reqId = req.requestId;  // string — recognized!
  res.json({ userId, requestId: reqId });
});

// Step 4: No more casts like (req as any).userId.
```

**Expected Output:** The route handler accesses `req.userId` and `req.requestId` without type errors.

**Why This Output Occurs:** The module augmentation adds `userId` and `requestId` to the `Request` interface. TypeScript now recognizes these properties at every call site, eliminating the need for type assertions.

#### Example 2: Augmenting a Plugin Interface

```typescript
// Step 1: Third-party library defines an interface.
// node_modules/my-lib/index.d.ts
export interface MyLibraryConfig {
  apiUrl: string;
  timeout?: number;
}
```

```typescript
// Step 2: Project augments the config.
// src/types/my-lib.d.ts
import "my-lib";

declare module "my-lib" {
  interface MyLibraryConfig {
    customHeader?: string;
    retryPolicy?: "none" | "linear" | "exponential";
  }
}
```

```typescript
// Step 3: Consumer uses the augmented config.
// src/app.ts
import { MyLibraryConfig } from "my-lib";

const config: MyLibraryConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  customHeader: "X-Custom-Header",
  retryPolicy: "exponential",
};

console.log(config.retryPolicy);  // "exponential"
```

**Expected Output:**
```
exponential
```

**Why This Output Occurs:** The module augmentation adds `customHeader` and `retryPolicy` to `MyLibraryConfig`. The consumer can now use these properties with full type safety.

### Real-World Cases

**Case 1: Express Middleware**
Express applications augment `Request` with `user`, `session`, `requestId`, and other middleware-attached properties.

**Case 2: Passport.js**
Passport augments the `Express.User` interface to include application-specific user properties.

**Case 3: Vue Plugins**
Vue plugins augment `ComponentCustomProperties` to add global properties like `$axios` or `$store`.

**Case 4: Inertia.js**
Inertia.js augments `InertiaConfig` with shared page props, flash messages, and auth data.

**Case 5: Hardhat**
Hardhat augments `hardhat/types/network` to add custom network types and properties.

**Case 6: Jest/Vitest**
Testing libraries augment `jest.Matchers` or `vitest` types to add custom matchers.

### References

- TypeScript Handbook: Module Augmentation — https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation
- TypeScript Handbook: Declaration Merging — https://www.typescriptlang.org/docs/handbook/declaration-merging.html
- Augment Third-Party Module Types Without Patching Source — https://raw.githubusercontent.com/pproenca/dot-skills
- Hardhat: Type Extensions — https://hardhat.org/hardhat-runner/docs/advanced/type-extensions
- Inertia.js: TypeScript — https://inertiajs.com/typescript


## 5. Triple-Slash Directives (`/// <reference path="..." />`)

### Definitions

**Core Definition**
Triple-slash directives are single-line comments containing a single XML tag that TypeScript interprets as compiler directives. The most common form, `/// <reference path="..." />`, declares a dependency on another file, instructing the compiler to include it in the compilation. Triple-slash directives are only valid at the top of a file, before any statements or declarations.

**Technical Definition**
Triple-slash directives are processed during a preprocessing pass before compilation. The `/// <reference path="..." />` directive adds the referenced file to the compilation and establishes a dependency ordering. The `/// <reference types="..." />` directive declares a dependency on an `@types` package (e.g., `/// <reference types="node" />` includes `@types/node`). The `/// <reference lib="..." />` directive explicitly includes a built-in library file (e.g., `/// <reference lib="es2017.string" />`). Since TypeScript 5.5, the compiler does not generate reference directives and does not emit handwritten directives to output files unless marked with `preserve="true"`. If the `--noResolve` compiler flag is specified, triple-slash references are ignored entirely.

**Beginner-Friendly Explanation**
Triple-slash directives are special comments that start with `///`. They tell the TypeScript compiler to include additional files or packages in the compilation. The most common one is `/// <reference path="./other-file.d.ts" />`, which says "include this other file." Another is `/// <reference types="node" />`, which says "include the Node.js type definitions." These directives must appear at the very top of the file, before any code. In modern TypeScript, they're less common because `import` statements and `tsconfig.json` `include`/`files` are preferred. But they're still used in `.d.ts` files for global dependencies and in older projects.

### Purposes

- To declare dependencies between declaration files.
- To include `@types` packages in a compilation (e.g., `/// <reference types="node" />`).
- To explicitly include built-in library files (e.g., `/// <reference lib="dom" />`).
- To order output when using `--out` or `--outFile`.
- To support legacy projects and global declaration file dependencies.

### Syntax Rules and Structure

**General Syntax: `/// <reference path="..." />`**

```typescript
/// <reference path="./other-file.d.ts" />
// Declares a dependency on another file.
// The compiler includes it in the compilation.
```

**Component Breakdown**
- `path="./other-file.d.ts"`: Relative path to the referenced file.
- Resolved relative to the containing file.

**General Syntax: `/// <reference types="..." />`**

```typescript
/// <reference types="node" />
// Declares a dependency on the @types/node package.
```

**Component Breakdown**
- `types="node"`: Includes `@types/node` in the compilation.
- Similar to an `import` for declaration packages.

**General Syntax: `/// <reference lib="..." />`**

```typescript
/// <reference lib="es2017.string" />
// Explicitly includes the ES2017 string library.
```

**Component Breakdown**
- `lib="es2017.string"`: Includes the specified built-in library file.

**General Syntax: `preserve="true"`**

```typescript
/// <reference path="./other.d.ts" preserve="true" />
// Emits the directive to the output file (TypeScript 5.5+).
```

**Component Breakdown**
- `preserve="true"`: Marks the directive for preservation in output.

**Syntax Rules**

- Triple-slash directives must be at the top of the file.
- Only comments may precede them; statements/declarations invalidate them.
- `path` resolves relative to the containing file.
- `types` resolves to `@types/*` packages.
- `lib` includes built-in library files.
- `preserve="true"` (TypeScript 5.5+) emits the directive to output.
- `--noResolve` ignores all triple-slash references.
- Multiple directives can be combined (one per line).

**Constraints and Limitations**

- Triple-slash directives are legacy—prefer `import` and `tsconfig.json` `include`.
- `--noResolve` disables all triple-slash reference processing.
- Referencing a non-existent file is an error.
- A file cannot reference itself.
- Since TypeScript 5.5, generated output omits triple-slash directives unless `preserve="true"`.
- `types` directives are ignored in `.ts` files if `types` is specified in `tsconfig.json`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `/// <reference types="node" />`

```typescript
// Step 1: Declaration file with a types reference.
// types/global.d.ts
/// <reference types="node" />

declare global {
  namespace NodeJS {
    interface ProcessEnv {
      NODE_ENV: "development" | "staging" | "production";
      API_URL: string;
      DATABASE_URL: string;
    }
  }
}

export {};
```

```typescript
// Step 2: Consumer uses typed process.env.
// src/config.ts
const env = process.env.NODE_ENV;  // "development" | "staging" | "production"
const apiUrl = process.env.API_URL;  // string
const dbUrl = process.env.DATABASE_URL;  // string

console.log(env);  // "development"
```

**Expected Output:**
```
development
```

**Why This Output Occurs:** The `/// <reference types="node" />` directive includes `@types/node`, which provides the `NodeJS.ProcessEnv` interface. The `declare global` block augments `ProcessEnv` with project-specific variables. TypeScript now recognizes `process.env.NODE_ENV`, `API_URL`, and `DATABASE_URL`.

#### Example 2: `/// <reference path="..." />`

```typescript
// Step 1: A declaration file that references another.
// types/index.d.ts
/// <reference path="./globals.d.ts" />
/// <reference path="./express.d.ts" />

// This file depends on globals.d.ts and express.d.ts.
// The compiler includes them automatically.
```

```typescript
// Step 2: The referenced files.
// types/globals.d.ts
declare const VERSION: string;
declare function log(message: string): void;

// types/express.d.ts
import "express";
declare module "express-serve-static-core" {
  interface Request {
    userId?: string;
  }
}
```

```typescript
// Step 3: Consumer uses the globals.
// src/app.ts
console.log(VERSION);  // "1.0.0"
log("Application started");
```

**Expected Output:**
```
1.0.0
```

**Why This Output Occurs:** The `/// <reference path="..." />` directives in `types/index.d.ts` instruct the compiler to include `globals.d.ts` and `express.d.ts`. The consumer can use `VERSION` and `log` without explicit imports.

### Real-World Cases

**Case 1: Node.js Projects**
Node.js projects use `/// <reference types="node" />` to include Node.js type definitions when not using `tsconfig.json` `types`.

**Case 2: Legacy Global Libraries**
Legacy projects use `/// <reference path="..." />` to declare dependencies between global declaration files.

**Case 3: Test Files**
Test files sometimes use `/// <reference types="jest" />` or `/// <reference types="vitest" />` to include testing framework types.

**Case 4: Library Publishing**
Libraries that bundle `.d.ts` files use `/// <reference path="..." />` to declare dependencies between declaration files in the package.

**Case 5: `--outFile` Ordering**
Projects using `--outFile` use triple-slash references to control the order in which files are concatenated.

### References

- TypeScript Handbook: Triple-Slash Directives — https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html
- TypeScript 5.5 Release Notes — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-5.html
- TypeScript ESLint: `triple-slash-reference` — https://typescript-eslint.io/rules/triple-slash-reference/


## Summary: Declaration File Feature Comparison

| Feature | Syntax / Mechanism | Purpose | Key Rule |
|---|---|---|---|
| `.d.ts` file | `declare function f(): void;` | Describe external code's types | No runtime code |
| Emitted declarations | `"declaration": true` | Publish types for compiled JS | `.d.ts` alongside `.js` |
| Ambient module | `declare module "lib" { }` | Declare external library shape | Must match import name |
| Shorthand module | `declare module "lib";` | Quick fix for untyped libs | Imports typed as `any` |
| Wildcard module | `declare module "*.css";` | Type asset imports | Matches file extensions |
| `declare global` | `declare global { }` | Augment global types from modules | Must be in a module |
| Ambient declaration | `declare const X: T;` | Declare global values | No implementation |
| Module augmentation | `declare module "lib" { }` | Add members to existing module types | Must be a module file |
| Global augmentation | `declare global { }` | Add members to global scope | `export {}` if needed |
| `/// <reference path="..." />` | Triple-slash directive | Declare file dependency | Top of file only |
| `/// <reference types="..." />` | Triple-slash directive | Include `@types` package | Equivalent to import |
| `/// <reference lib="..." />` | Triple-slash directive | Include built-in library | Explicit lib inclusion |


## References

- TypeScript Handbook: Type Declarations — https://www.typescriptlang.org/docs/handbook/2/type-declarations.html
- TypeScript Handbook: Declaration Files Introduction — https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html
- TypeScript Handbook: Declaration Files Theory — https://www.typescriptlang.org/docs/handbook/declaration-files/deep-dive.html
- TypeScript Handbook: Library Structures — https://www.typescriptlang.org/docs/handbook/declaration-files/library-structures.html
- TypeScript Handbook: Ambient Declarations — https://www.typescriptlang.org/docs/handbook/declaration-files/ambient-declarations.html
- TypeScript Handbook: Triple-Slash Directives — https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html
- TypeScript Handbook: Declaration Merging — https://www.typescriptlang.org/docs/handbook/declaration-merging.html
- TypeScript Handbook: Global Augmentation — https://www.typescriptlang.org/docs/handbook/declaration-merging.html#global-augmentation
- TypeScript Handbook: Module Augmentation — https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation
- TypeScript TSConfig: `declaration` — https://www.typescriptlang.org/tsconfig#declaration
- TypeScript TSConfig: `declarationMap` — https://www.typescriptlang.org/tsconfig#declarationMap
- TypeScript 5.5 Release Notes — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-5.html
- Total TypeScript: How to Properly Type Window — https://www.totaltypescript.com/how-to-properly-type-window
- DefinitelyTyped — https://github.com/DefinitelyTyped/DefinitelyTyped
- Augment Third-Party Module Types Without Patching Source — https://raw.githubusercontent.com/pproenca/dot-skills
- Hardhat: Type Extensions — https://hardhat.org/hardhat-runner/docs/advanced/type-extensions
- Inertia.js: TypeScript — https://inertiajs.com/typescript
- TypeScript ESLint: `triple-slash-reference` — https://typescript-eslint.io/rules/triple-slash-reference/