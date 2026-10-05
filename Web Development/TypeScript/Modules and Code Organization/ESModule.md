# TypeScript ES Modules: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
ES Modules (ECMAScript Modules) are the official, standardized module system built into the JavaScript language. They provide a way to split code into separate files, each with its own scope, and to explicitly share code between files using `import` and `export` statements. TypeScript fully supports ES Modules and adds type-aware features on top of the standard syntax.

**Technical Definition**
ES Modules (ESM) are defined by the ECMAScript specification and use dedicated `import` and `export` syntax. In TypeScript, any file containing a top-level `import` or `export` is considered a module and executes within its own scope, not the global scope. ESM was added to the JavaScript spec in 2015 and gained broad support in browsers and Node.js by 2020. TypeScript supports ESM output via the `module` compiler option (e.g., `"esnext"`, `"node16"`, `"nodenext"`), and the `moduleResolution` option (e.g., `"bundler"`, `"nodenext"`) controls how import specifiers are resolved to files on disk. TypeScript also provides type-only import/export syntax (`import type`, `export type`) to ensure that type-only dependencies are erased during compilation.

**Beginner-Friendly Explanation**
ES Modules are the modern way to organize JavaScript and TypeScript code into separate files. Instead of everything sharing one global scope (which causes naming conflicts), each file is its own module with its own private scope. To share code, you use `export` to make things available and `import` to bring them in. For example, a `math.ts` file can `export function add()`, and a `app.ts` file can `import { add } from "./math.js"`. TypeScript adds type safety to this system, and it also has special syntax for importing types that disappear when compiled. If you're building a modern web app or Node.js project, you're almost certainly using ES Modules.

### Key Characteristics

- **File-scoped by default**: Each module has its own scope; nothing is shared unless explicitly exported.
- **Standardized syntax**: Uses `import` and `export` statements defined by ECMAScript.
- **Static structure**: Imports and exports are resolved at compile time, enabling tree-shaking.
- **Type-aware**: TypeScript extends ESM with type annotations and type-only import/export syntax.
- **Extension-aware**: In native ESM environments (Node.js ESM, browsers without bundlers), relative imports require explicit file extensions (`.js`).
- **Multiple export forms**: Named exports, default exports, and namespace exports.
- **Tree-shakeable**: Named exports enable bundlers to eliminate unused code from the final bundle.
- **Module resolution modes**: TypeScript offers `"node16"`, `"nodenext"`, `"bundler"`, and legacy modes, each with different rules.

### Prerequisites

- Basic knowledge of JavaScript functions, variables, and objects
- Familiarity with TypeScript type annotations and interfaces
- Understanding of file systems and relative paths
- Basic familiarity with `tsconfig.json` compiler options

### Related Programming Areas

- **Module Systems**: CommonJS (CJS), AMD, UMD, SystemJS, and ES Modules
- **Bundlers**: Webpack, Rollup, Vite, esbuild, and their module handling
- **Node.js Runtime**: ESM support in Node.js (v12+), `package.json` `"type"` field
- **Tree-Shaking**: Dead code elimination based on ESM's static structure
- **TypeScript Compiler**: `module` and `moduleResolution` options

### Core Concepts / Features

1. `export` and `import` Syntax
2. Named Exports, Default Exports, and Namespace Imports (`import * as`)
3. Re-exporting Configurations and Tree-Shaking Patterns
4. Type-Only Imports and Exports (`import type` / `export type`)
5. The Impact of Explicit `.js` Extensions in ESM Import Paths


## 1. `export` and `import` Syntax

### Definitions

**Core Definition**
The `export` keyword makes declarations (variables, functions, classes, types, interfaces) available to other modules. The `import` keyword brings those exported declarations into the current module's scope. Together, they form the foundation of ES Modules.

**Technical Definition**
ES Module syntax uses dedicated `export` and `import` declarations at the top level of a module. `export` can be applied directly to declarations (`export function f() {}`) or used in an export list (`export { f, g }`). `import` declarations bring exported bindings into scope using named imports (`import { f } from "./module.js"`), default imports (`import f from "./module.js"`), or namespace imports (`import * as mod from "./module.js"`). In TypeScript, `export` and `import` can also be applied to TypeScript-specific declarations like type aliases and interfaces. The compiler uses the `module` option to determine how these statements are emitted in JavaScript.

**Beginner-Friendly Explanation**
The `export` keyword says "this thing can be used by other files." The `import` keyword says "I want to use this thing from another file." For example, if `math.ts` has `export function add(a, b) { return a + b; }`, then `app.ts` can do `import { add } from "./math.js"` and call `add(1, 2)`. The path in the import must point to the file, and in modern ESM environments, you need to include the `.js` extension even if the source file is `.ts`. TypeScript checks that the imported names actually exist and that their types match what you expect.

### Purposes

- To split code into separate, independently scoped files.
- To explicitly control what is shared between files (encapsulation).
- To enable static analysis for tree-shaking and refactoring.
- To provide type-safe imports with compiler-checked names and types.
- To support modern JavaScript runtimes (browsers, Node.js) natively.

### Syntax Rules and Structure

**General Syntax: Named Export**

```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export const PI = 3.14159;

export interface Point {
  x: number;
  y: number;
}
```

**Component Breakdown**
- `export function`: Exports a function declaration.
- `export const`: Exports a variable declaration.
- `export interface`: Exports a TypeScript interface.

**General Syntax: Named Import**

```typescript
// app.ts
import { add, PI, Point } from "./math.js";

console.log(add(1, 2));  // 3
console.log(PI);          // 3.14159
```

**Component Breakdown**
- `import { add, PI, Point }`: Imports the named exports.
- `from "./math.js"`: The module specifier (path).

**General Syntax: Renaming Imports and Exports**

```typescript
// Renaming on import
import { add as addNumbers, PI as pi } from "./math.js";

// Renaming on export
export { add as addNumbers, PI as pi };
```

**Component Breakdown**
- `as`: Renames the binding locally or externally.

**General Syntax: Export List**

```typescript
// utils.ts
function internalHelper() { }
export function publicApi() { }

export { publicApi, internalHelper as helper };
```

**Component Breakdown**
- `export { ... }`: An export list at the end of the module.
- `internalHelper as helper`: Renames on export.

**Syntax Rules**

- Any file with a top-level `import` or `export` is a module.
- Files without imports/exports are scripts with global scope (use `export {}` to force module mode).
- Named exports are declared with `export` before the declaration or via an export list.
- Imports use `import { name } from "specifier"` for named exports.
- The `as` keyword renames imports and exports.
- TypeScript-specific declarations (types, interfaces) can be exported and imported like values.
- The module specifier must be a string literal (not a variable).

**Constraints and Limitations**

- Import specifiers are string literals; dynamic imports require `import()`.
- Top-level `import`/`export` cannot appear inside conditional blocks or functions.
- Circular imports can cause runtime issues (values may be `undefined` during initialization).
- In native ESM (Node.js without bundler), relative imports require `.js` extensions.
- The `export =` syntax is CommonJS-only and incompatible with ESM.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Named Exports and Imports

```typescript
// Step 1: Define a module with named exports.
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

export const PI = 3.14159;

export interface Point {
  x: number;
  y: number;
}

export function distance(p1: Point, p2: Point): number {
  return Math.sqrt((p2.x - p1.x) ** 2 + (p2.y - p1.y) ** 2);
}

// Step 2: Import and use the named exports.
// app.ts
import { add, multiply, PI, distance } from "./math.js";
import type { Point } from "./math.js";

const result = add(5, 3);
console.log(result);  // 8

const product = multiply(4, 7);
console.log(product);  // 28

console.log(PI);  // 3.14159

const p1: Point = { x: 0, y: 0 };
const p2: Point = { x: 3, y: 4 };
console.log(distance(p1, p2));  // 5

// Step 3: Missing imports are compile errors.
// import { subtract } from "./math.js";
// ❌ Error: Module '"./math.js"' has no exported member 'subtract'.
```

**Expected Output:**
```
8
28
3.14159
5
```

**Why This Output Occurs:** The `math.ts` module exports `add`, `multiply`, `PI`, `Point`, and `distance`. The `app.ts` module imports the functions and the type separately (using `import type` for the type-only import). TypeScript checks that each imported name exists and has the correct type.

#### Example 2: Export Lists and Renaming

```typescript
// Step 1: Define a module with internal and public functions.
// utils.ts
function internalTrim(s: string): string {
  return s.trim();
}

function internalUpper(s: string): string {
  return s.toUpperCase();
}

function formatName(first: string, last: string): string {
  return `${internalTrim(first)} ${internalTrim(last)}`;
}

// Step 2: Export using an export list with renaming.
export {
  formatName,
  internalTrim as trim,
  internalUpper as upper,
};

// Step 3: Import the renamed exports.
// app.ts
import { formatName, trim, upper } from "./utils.js";

console.log(formatName("  Alice  ", "  Smith  "));  // "Alice Smith"
console.log(trim("  hello  "));                     // "hello"
console.log(upper("world"));                        // "WORLD"

// Step 4: Import with renaming.
import { trim as trimString } from "./utils.js";
console.log(trimString("  test  "));  // "test"
```

**Expected Output:**
```
Alice Smith
hello
WORLD
test
```

**Why This Output Occurs:** The `export { ... }` list exposes `formatName`, and renames `internalTrim` to `trim` and `internalUpper` to `upper`. The importing module can further rename with `import { trim as trimString }`. All imports are type-checked.

### Real-World Cases

**Case 1: Utility Libraries**
Utility libraries (e.g., lodash-es) export individual functions as named exports, enabling consumers to import only what they need and enabling tree-shaking.

**Case 2: Component Libraries**
React component libraries export components as named exports, allowing consumers to import specific components without pulling in the entire library.

**Case 3: API Clients**
API clients export typed functions and types, ensuring that consumers get both the runtime functions and the TypeScript types they need.

**Case 4: Domain Models**
Domain models export interfaces and types as named exports, providing a shared vocabulary across modules.

### References

- TypeScript Handbook: Modules — https://www.typescriptlang.org/docs/handbook/2/modules.html
- MDN: JavaScript Modules — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules


## 2. Named Exports, Default Exports, and Namespace Imports (`import * as`)

### Definitions

**Core Definition**
ES Modules support three primary export/import styles: named exports (multiple values per module), default exports (one primary value per module), and namespace imports (importing all exports as a single object). Each has distinct use cases and trade-offs.

**Technical Definition**
A named export exports a binding by name (`export function f()`), and importers must use the same name (`import { f }`). A default export exports a single value as the module's "main" export (`export default f`), and importers can choose any name (`import anything from "./module.js"`). A namespace import imports the entire module as an object (`import * as mod from "./module.js"`), with each named export accessible as a property (`mod.f`). TypeScript type-checks all three forms, and the `esModuleInterop` compiler option affects how default imports from CommonJS modules are handled.

**Beginner-Friendly Explanation**
Named exports are like a toolbox where each tool has a name—you pick the ones you need. Default exports are like a module that has one main thing to offer—you can call it whatever you want when you import it. Namespace imports bring in the whole toolbox as a single object, so you can access tools with dot notation. Most modern TypeScript codebases prefer named exports because they're better for tree-shaking, refactoring, and autocomplete. Default exports are useful for frameworks that expect them (like React lazy loading), and namespace imports are useful when you want to access many exports from a module with a clear prefix.

### Purposes

- To provide flexible import styles for different use cases.
- To enable tree-shaking through named exports.
- To support frameworks and tools that require default exports.
- To provide a single, clear "main" export for modules with one primary purpose.
- To allow namespace imports for modules with many related exports.

### Syntax Rules and Structure

**General Syntax: Default Export**

```typescript
// logger.ts
export default function log(message: string): void {
  console.log(`[LOG] ${message}`);
}
```

**Component Breakdown**
- `export default`: Marks a single value as the module's default export.
- The value can be a function, class, object, or expression.

**General Syntax: Default Import**

```typescript
// app.ts
import log from "./logger.js";  // Any name works
log("Hello");  // "[LOG] Hello"
```

**Component Breakdown**
- `import log`: The local name can be anything.
- No braces for default imports.

**General Syntax: Namespace Import**

```typescript
// app.ts
import * as math from "./math.js";

console.log(math.add(1, 2));  // 3
console.log(math.PI);          // 3.14159
```

**Component Breakdown**
- `import * as math`: Imports all named exports as properties of `math`.
- `math.add`, `math.PI`: Access via dot notation.

**General Syntax: Combining Default and Named Imports**

```typescript
// app.ts
import log, { formatMessage, LogLevel } from "./logger.js";
```

**Component Breakdown**
- `log`: The default export.
- `{ formatMessage, LogLevel }`: Named exports.

**Syntax Rules**

- A module can have at most one default export.
- Named exports are imported with braces: `import { name } from "module"`.
- Default exports are imported without braces: `import name from "module"`.
- Namespace imports use `* as` and access exports as properties.
- Default and named imports can be combined in one statement.
- Default exports can be anonymous (functions, classes) or named expressions.
- The `esModuleInterop` option enables default imports from CommonJS modules.

**Constraints and Limitations**

- Default exports are harder to refactor (the import name is local, not checked against the export).
- Default exports can hurt tree-shaking because the entire module must be loaded to access the default.
- Namespace imports can prevent tree-shaking because bundlers may keep the entire namespace.
- Namespace imports are not enumerable in the same way as object literals.
- Combining default and named imports in one statement is not allowed with `import type`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Default Export and Import

```typescript
// Step 1: Define a module with a default export.
// calculator.ts
export default class Calculator {
  private value: number = 0;

  add(n: number): this {
    this.value += n;
    return this;
  }

  subtract(n: number): this {
    this.value -= n;
    return this;
  }

  getValue(): number {
    return this.value;
  }
}

// Step 2: Import the default export with any name.
// app.ts
import Calculator from "./calculator.js";

const calc = new Calculator();
calc.add(10).subtract(3);
console.log(calc.getValue());  // 7

// Step 3: Default import name can be anything.
import MyCalc from "./calculator.js";
const calc2 = new MyCalc();
calc2.add(5);
console.log(calc2.getValue());  // 5
```

**Expected Output:**
```
7
5
```

**Why This Output Occurs:** The `calculator.ts` module has a default export (the `Calculator` class). The `app.ts` module imports it with the name `Calculator` in one case and `MyCalc` in another. Both work because the default export doesn't have a fixed name.

#### Example 2: Namespace Import

```typescript
// Step 1: Define a module with multiple named exports.
// stringUtils.ts
export function trim(s: string): string {
  return s.trim();
}

export function upper(s: string): string {
  return s.toUpperCase();
}

export function lower(s: string): string {
  return s.toLowerCase();
}

export const VERSION = "1.0.0";

// Step 2: Import all exports as a namespace.
// app.ts
import * as StringUtils from "./stringUtils.js";

console.log(StringUtils.trim("  hello  "));  // "hello"
console.log(StringUtils.upper("world"));     // "WORLD"
console.log(StringUtils.lower("HELLO"));     // "hello"
console.log(StringUtils.VERSION);            // "1.0.0"

// Step 3: Combine with default imports.
import defaultExport, { trim, upper } from "./stringUtils.js";
// Note: stringUtils.ts has no default export, so this would error.
// Use namespace or named imports instead.
```

**Expected Output:**
```
hello
WORLD
hello
1.0.0
```

**Why This Output Occurs:** The `stringUtils.ts` module exports `trim`, `upper`, `lower`, and `VERSION` as named exports. The `app.ts` module imports them all as a namespace object (`StringUtils`) and accesses each export as a property.

### Real-World Cases

**Case 1: React Components**
React component libraries typically use named exports for components, but default exports are still common in some codebases and are required for `React.lazy` dynamic imports.

**Case 2: Configuration Modules**
Configuration modules often use default exports for the main configuration object and named exports for related constants.

**Case 3: Utility Libraries**
Utility libraries prefer named exports because they enable better tree-shaking and are more refactor-friendly.

**Case 4: Namespace Imports for Math/Constants**
Namespace imports are useful for modules that export many related constants or utility functions, providing a clear prefix.

### References

- TypeScript Handbook: Modules — https://www.typescriptlang.org/docs/handbook/2/modules.html
- TypeScript `esModuleInterop` — https://www.typescriptlang.org/tsconfig#esModuleInterop
- MDN: `import` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import
- MDN: `export` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export


## 3. Re-exporting Configurations and Tree-Shaking Patterns

### Definitions

**Core Definition**
Re-exporting is the practice of importing a value from one module and immediately exporting it from another, creating a "barrel" or "facade" module that consolidates exports. Tree-shaking is a build-time optimization where bundlers remove unused exports from the final bundle. Re-exporting patterns have a significant impact on tree-shaking effectiveness.

**Technical Definition**
Re-exporting uses `export { name } from "./module.js"` or `export * from "./module.js"` to forward exports without creating local bindings. TypeScript supports these forms with full type checking. Tree-shaking relies on ESM's static structure: bundlers analyze the import/export graph and eliminate code that is never imported. Named exports enable tree-shaking because the bundler can see exactly which names are used. `export *` can hurt tree-shaking because the bundler cannot statically determine which names are actually used, especially when re-exporting from another library. Barrel files (index files that re-export many modules) are common but can negate tree-shaking if they use `export *`.

**Beginner-Friendly Explanation**
Re-exporting is when a module says "everything from this other module is also mine." You create a "barrel file" (often called `index.ts`) that re-exports from many files, so consumers can import from one place instead of many. Tree-shaking is when the bundler removes code you don't use. The problem is that `export *` makes it hard for the bundler to know what's used, so it often keeps everything. Named re-exports (`export { foo } from "./foo.js"`) are better because the bundler can see exactly what's exported and what's used. The rule of thumb: prefer named re-exports over `export *` if tree-shaking matters.

### Purposes

- To create a single entry point for a library or module (barrel file).
- To simplify imports for consumers (fewer paths to remember).
- To enable tree-shaking by using named re-exports.
- To expose a curated public API while hiding internal modules.
- To aggregate exports from many modules into a cohesive namespace.

### Syntax Rules and Structure

**General Syntax: Named Re-export**

```typescript
// index.ts
export { add, multiply } from "./math.js";
export { trim, upper } from "./stringUtils.js";
```

**Component Breakdown**
- `export { name } from "module"`: Re-exports specific names.
- The names are not available locally unless also imported.

**General Syntax: Re-export Everything (`export *`)**

```typescript
// index.ts
export * from "./math.js";
export * from "./stringUtils.js";
```

**Component Breakdown**
- `export *`: Re-exports all named exports.
- Does not re-export default exports.

**General Syntax: Re-export with Renaming**

```typescript
// index.ts
export { add as addNumbers, subtract as subtractNumbers } from "./math.js";
```

**Component Breakdown**
- `as`: Renames on re-export.

**General Syntax: Re-export Default as Named**

```typescript
// index.ts
export { default as Calculator } from "./calculator.js";
```

**Component Breakdown**
- `default as Calculator`: Re-exports the default export under a named export.

**Syntax Rules**

- `export { name } from "module"` re-exports specific names.
- `export * from "module"` re-exports all named exports (not default).
- Re-exports do not create local bindings.
- Named re-exports are tree-shakeable; `export *` may not be.
- Re-exporting from a module that re-exports from another can cause tree-shaking issues.
- Default exports must be explicitly named when re-exported.

**Constraints and Limitations**

- `export *` does not re-export default exports.
- `export *` with conflicting names across modules causes the conflicting names to be omitted.
- Barrel files with `export *` can hurt tree-shaking because bundlers cannot determine used names.
- Re-exporting from CommonJS modules in ESM can cause interop issues.
- Circular re-exports can cause runtime errors.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Barrel File with Named Re-exports

```typescript
// Step 1: Define individual modules.
// math.ts
export function add(a: number, b: number): number { return a + b; }
export function multiply(a: number, b: number): number { return a * b; }
export const PI = 3.14159;

// stringUtils.ts
export function trim(s: string): string { return s.trim(); }
export function upper(s: string): string { return s.toUpperCase(); }

// Step 2: Create a barrel file with named re-exports.
// index.ts
export { add, multiply, PI } from "./math.js";
export { trim, upper } from "./stringUtils.js";

// Step 3: Import from the barrel.
// app.ts
import { add, trim, PI } from "./index.js";

console.log(add(1, 2));    // 3
console.log(trim("  hello  "));  // "hello"
console.log(PI);           // 3.14159

// Step 4: Tree-shaking works: `multiply` and `upper` are not imported,
// so bundlers can eliminate them from the final bundle.
// (This is a build-time optimization, not visible at runtime.)
```

**Expected Output:**
```
3
hello
3.14159
```

**Why This Output Occurs:** The `index.ts` barrel file re-exports specific names from `math.ts` and `stringUtils.ts`. The `app.ts` imports from the barrel. Because the re-exports are named, the bundler can see exactly which exports are used and tree-shake the rest.

#### Example 2: `export *` and Tree-Shaking Pitfalls

```typescript
// Step 1: Define modules with many exports.
// math.ts
export function add(a: number, b: number): number { return a + b; }
export function multiply(a: number, b: number): number { return a * b; }
export function divide(a: number, b: number): number { return a / b; }
export function subtract(a: number, b: number): number { return a - b; }
export const PI = 3.14159;
export const E = 2.71828;

// Step 2: Barrel file with `export *`.
// index.ts
export * from "./math.js";

// Step 3: Import only `add` — but bundler may keep everything.
// app.ts
import { add } from "./index.js";
console.log(add(1, 2));  // 3

// Step 4: With `export *`, the bundler may not be able to tree-shake
// `multiply`, `divide`, `subtract`, `PI`, and `E` because it cannot
// statically determine which names are used through the barrel.

// Step 5: Named re-exports fix this.
// index.ts (fixed)
export { add, multiply, divide, subtract, PI, E } from "./math.js";

// Now the bundler knows exactly which names are exported and can
// tree-shake unused ones.
```

**Expected Output:**
```
3
```

**Why This Output Occurs:** `export *` makes it difficult for bundlers to determine which exports are used, potentially preventing tree-shaking. Named re-exports provide the static information needed for effective tree-shaking.

### Real-World Cases

**Case 1: Component Library Barrel Files**
React component libraries use barrel files to export all components from a single entry point. Named re-exports ensure tree-shaking works, so consumers only bundle the components they import.

**Case 2: Utility Library Aggregation**
Utility libraries aggregate functions from many modules into a single entry point. Named re-exports enable consumers to import only the functions they need.

**Case 3: Monorepo Package Exports**
Monorepos use barrel files to expose packages from a single entry point, with named re-exports to support tree-shaking.

**Case 4: Icon Libraries**
Icon libraries use named re-exports to expose hundreds of icons from a barrel file, enabling tree-shaking so consumers only bundle the icons they use.

### References

- TypeScript Handbook: Modules Reference (Export Syntax) — https://www.typescriptlang.org/docs/handbook/modules/reference.html
- MDN: `export` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export
- Webpack: Tree Shaking — https://webpack.js.org/guides/tree-shaking/
- Rollup: Tree Shaking — https://rollupjs.org/faqs/#tree-shaking


## 4. Type-Only Imports and Exports (`import type` / `export type`)

### Definitions

**Core Definition**
Type-only imports and exports (`import type` and `export type`) are TypeScript-specific syntax that explicitly marks imports and exports as type-only. These are guaranteed to be erased during compilation, so they produce no JavaScript code and create no runtime dependencies. They were introduced in TypeScript 3.8.

**Technical Definition**
The `import type` and `export type` syntax tells TypeScript that the imported/exported declarations are only used in type positions and should be completely erased from the emitted JavaScript. This is important for `isolatedModules` mode, `transpileModule` API, Babel, and other tools that transpile TypeScript file-by-file without type information. By default, TypeScript automatically elides imports that are only used in type positions, but `import type` makes this explicit and mandatory. Import specifiers can also be prefixed with the `type` keyword (`import { f, type SomeInterface } from "./module.js"`). The `verbatimModuleSyntax` compiler option (TypeScript 5.0+) further enforces explicit type-only annotations.

**Beginner-Friendly Explanation**
Type-only imports and exports are a way to say "this import is only for types, not for runtime code." When TypeScript compiles your code to JavaScript, these imports disappear completely—no `require()` call, no `import` statement, nothing. This is important because Babel and other tools that compile TypeScript file-by-file can't always tell which imports are type-only. With `import type`, you make it explicit. For example, `import type { User } from "./models.js"` tells TypeScript and Babel that `User` is only a type, so it should be removed from the output. This keeps your bundle smaller and avoids unnecessary runtime dependencies.

### Purposes

- To make type-only imports explicit and ensure they are erased during compilation.
- To improve compatibility with `isolatedModules`, Babel, and other file-by-file transpilers.
- To reduce bundle size by eliminating unnecessary runtime imports.
- To enable `verbatimModuleSyntax` for stricter module output.
- To prevent accidental runtime dependencies on types.

### Syntax Rules and Structure

**General Syntax: `import type` Declaration**

```typescript
import type { SomeType } from "./module.js";
```

**Component Breakdown**
- `import type`: Imports only for type positions.
- The import is completely erased at runtime.

**General Syntax: Inline Type Import**

```typescript
import { f, type SomeInterface } from "./module.js";
```

**Component Breakdown**
- `type SomeInterface`: Only this specifier is type-only.
- `f`: A regular (value) import.

**General Syntax: `export type` Declaration**

```typescript
export type { SomeType };
```

**Component Breakdown**
- `export type`: Exports a type-only binding.
- Erased at runtime.

**General Syntax: `import type` with Default and Named**

```typescript
import type { default as fs, BigIntOptions } from "fs";
```

**Component Breakdown**
- A type-only import can specify a default import or named bindings, but not both in one declaration.

**General Syntax: Inline `export type`**

```typescript
export { f, type SomeType };
```

**Component Breakdown**
- `type SomeType`: Only this export is type-only.

**Syntax Rules**

- `import type` and `export type` are guaranteed to be elided from the output.
- Type-only imports can only be used in type positions, not as values.
- A type-only import cannot declare both a default import and named bindings in one declaration.
- Import specifiers can be individually marked with `type`.
- `verbatimModuleSyntax` (TypeScript 5.0+) enforces explicit `type` annotations.
- Classes are both types and values; `import type` imports only the type side.

**Constraints and Limitations**

- `import type` imports cannot be used as values (e.g., `new`, function calls).
- Extending a class imported with `import type` is an error.
- `import type` cannot be combined with `import()` type syntax.
- Type-only imports are erased, so they cannot carry side effects.
- The `verbatimModuleSyntax` option may require code changes in existing projects.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `import type` and `export type`

```typescript
// Step 1: Define types in a module.
// models.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

export type UserRole = "admin" | "user" | "guest";

export function createUser(name: string): User {
  return { id: 1, name, email: `${name}@example.com` };
}

// Step 2: Import types with `import type` and values normally.
// app.ts
import { createUser } from "./models.js";
import type { User, UserRole } from "./models.js";

const user: User = createUser("Alice");
const role: UserRole = "admin";

console.log(user.name);  // "Alice"
console.log(role);       // "admin"

// Step 3: The compiled JavaScript contains only the value import.
// Compiled app.js:
// import { createUser } from "./models.js";
// const user = createUser("Alice");
// const role = "admin";
// console.log(user.name);
// console.log(role);

// Step 4: `export type` for re-exporting types.
// index.ts
export { createUser } from "./models.js";
export type { User, UserRole } from "./models.js";

// Step 5: Inline type imports.
import { createUser, type User } from "./models.js";
const user2: User = createUser("Bob");
```

**Expected Output:**
```
Alice
admin
```

**Why This Output Occurs:** The `import type` declaration imports `User` and `UserRole` as type-only bindings. These are erased during compilation, so the emitted JavaScript only contains the `createUser` import. The `export type` re-exports types without creating runtime exports.

#### Example 2: `verbatimModuleSyntax` and Type-Only Enforcement

```typescript
// Step 1: Enable verbatimModuleSyntax in tsconfig.json.
// { "compilerOptions": { "verbatimModuleSyntax": true } }

// Step 2: Without explicit `type`, TypeScript cannot elide imports.
// app.ts
import { User } from "./models.js";  // ⚠️ Warning: User is only used as a type.
// With verbatimModuleSyntax, this must be:
import type { User } from "./models.js";

// Step 3: Correct usage with verbatimModuleSyntax.
import { createUser } from "./models.js";
import type { User } from "./models.js";

const user: User = createUser("Alice");
console.log(user.name);

// Step 4: Inline type specifiers also work.
import { createUser, type User } from "./models.js";

// Step 5: The compiled output preserves the exact import syntax.
// app.js:
// import { createUser } from "./models.js";
// const user = createUser("Alice");
// console.log(user.name);
// (The `import type` line is completely absent.)
```

**Expected Output:**
```
Alice
```

**Why This Output Occurs:** With `verbatimModuleSyntax`, TypeScript requires explicit `type` annotations on type-only imports. The `import type` line is completely erased from the output, while the value import is preserved exactly as written.

### Real-World Cases

**Case 1: Babel and SWC Transpilation**
Projects using Babel or SWC to transpile TypeScript rely on `import type` to ensure type-only imports are correctly erased, since these tools do not have full type information.

**Case 2: `isolatedModules` Mode**
When `isolatedModules` is enabled, `import type` ensures that type-only imports are correctly identified and erased, preventing runtime errors.

**Case 3: Library Authoring**
Library authors use `export type` to re-export types without including them in the runtime bundle, keeping the package smaller.

**Case 4: `verbatimModuleSyntax` Projects**
Modern TypeScript projects enable `verbatimModuleSyntax` to enforce explicit type-only annotations, ensuring predictable module output.

### References

- TypeScript 3.8 Release Notes: Type-Only Imports and Exports — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html#type-only-imports-and-export
- TypeScript Modules Reference: Type-Only Imports and Exports — https://www.typescriptlang.org/docs/handbook/modules/reference.html#type-only-imports-and-exports
- TypeScript `verbatimModuleSyntax` — https://www.typescriptlang.org/tsconfig#verbatimModuleSyntax
- TypeScript `isolatedModules` — https://www.typescriptlang.org/tsconfig#isolatedModules


## 5. The Impact of Explicit `.js` Extensions in ESM Import Paths

### Definitions

**Core Definition**
In native ES Modules (Node.js ESM, browsers without bundlers), relative import paths must include the full file extension (`.js`). This means that even when writing TypeScript, you must import from `"./foo.js"` rather than `"./foo"`. TypeScript 4.7 introduced `"node16"` and `"nodenext"` module options that enforce this requirement.

**Technical Definition**
Node.js ESM requires explicit file extensions for relative import specifiers. Since TypeScript is designed to never transform valid JavaScript constructs, it enforces this requirement when the `module` option is set to `"node16"` or `"nodenext"`. When TypeScript compiles a `.ts` file to a `.js` file, the import path in the source must reference the output `.js` extension, not the source `.ts` extension. The `allowImportingTsExtensions` compiler option allows `.ts` extensions in imports, but it requires `noEmit` or `emitDeclarationOnly` (it's designed for bundler scenarios, not direct Node.js execution). Tools like `tsc-esm-fix` and `rollup-plugin-esnext-to-nodenext` can automatically add `.js` extensions to existing imports during build.

**Beginner-Friendly Explanation**
In the old CommonJS system, you could write `import { foo } from "./foo"` without an extension—Node.js would figure out the file. But in native ES Modules, Node.js requires the full extension: `import { foo } from "./foo.js"`. This means when you're writing TypeScript for Node.js ESM, you have to write `.js` even though the file on disk is `.ts`. It feels weird at first, but it's because Node.js loads the compiled `.js` file, not the `.ts` source. TypeScript tooling like auto-imports will usually add the extension for you. If you're using a bundler (like Vite or webpack), you can use `"moduleResolution": "bundler"` and skip the extensions—the bundler handles resolution.

### Purposes

- To comply with Node.js ESM's requirement for explicit file extensions.
- To ensure that compiled JavaScript can be loaded correctly by Node.js ESM.
- To enable direct execution of ESM output without a bundler.
- To provide compatibility with modern Node.js versions (v12+) that support ESM.
- To support tools like `tsx` and `ts-node` that run TypeScript directly.

### Syntax Rules and Structure

**General Syntax: ESM Import with `.js` Extension**

```typescript
// bar.ts (source)
import { helper } from "./foo.js";  // References the compiled output
helper();
```

**Component Breakdown**
- `"./foo.js"`: The path to the compiled JavaScript file.
- The source file is `foo.ts`, but the import references `foo.js`.

**General Syntax: `allowImportingTsExtensions`**

```json
{
  "compilerOptions": {
    "allowImportingTsExtensions": true,
    "noEmit": true
  }
}
```

**Component Breakdown**
- `allowImportingTsExtensions`: Allows `.ts` extensions in imports.
- Requires `noEmit` or `emitDeclarationOnly`.

**General Syntax: `tsc-esm-fix` Post-Build**

```bash
tsc-esm-fix --tsconfig tsconfig.json
```

**Component Breakdown**
- Automatically adds `.js` extensions to imports in the compiled output.

**Syntax Rules**

- Node.js ESM requires explicit file extensions for relative imports.
- TypeScript enforces this when `module` is `"node16"` or `"nodenext"`.
- The import path references the output `.js` file, not the source `.ts` file.
- `allowImportingTsExtensions` allows `.ts` extensions but requires `noEmit`.
- Bundler scenarios use `"moduleResolution": "bundler"` and can omit extensions.
- Auto-imports and path completion in IDEs typically add `.js` extensions automatically.
- `.d.ts` files are also affected—they must import from `.js` paths.

**Constraints and Limitations**

- The `.js` extension requirement does not apply to bare specifiers (e.g., `"lodash"`).
- It only applies to relative specifiers (`"./foo.js"`, `"../bar.js"`).
- Changing all imports to `.js` can be tedious in existing codebases.
- Tools like `tsc-esm-fix` automate the process but add a build step.
- The requirement can be confusing because the source file is `.ts` but the import is `.js`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: ESM Import with `.js` Extension
\
`foo.ts`
```typescript
// Step 1: Define a module in foo.ts.
export function helper(): string {
  return "Hello from foo";
}
```
\
`bar.ts`
```typescript
// Step 2: Import using the .js extension.
import { helper } from "./foo.js";  // References the compiled foo.js

console.log(helper());  // "Hello from foo"

// Step 3: Compiled output (bar.js) preserves the .js import.
// import { helper } from "./foo.js";
// console.log(helper());

// Step 4: Node.js runs the compiled .js files.
// node bar.js
// Output: "Hello from foo"

// Step 5: Without the extension, Node.js ESM fails.
// import { helper } from "./foo";
// ❌ Error: Cannot find module './foo' imported from bar.js
```

**Expected Output:**
```
Hello from foo
```

**Why This Output Occurs:** The `.ts` source imports from `"./foo.js"` (the compiled output path). The compiled JavaScript preserves this import. Node.js ESM loads `foo.js` successfully. Without the extension, Node.js ESM cannot resolve the module.

#### Example 2: Bundler vs. Node.js ESM Resolution

```typescript
// Step 1: Bundler scenario — extensions not required.
// vite.config.ts
// { "compilerOptions": { "moduleResolution": "bundler" } }

// app.ts (with bundler)
import { helper } from "./foo";  // ✅ Allowed with bundler resolution
console.log(helper());

// Step 2: Node.js ESM scenario — extensions required.
// tsconfig.json
// {
//   "compilerOptions": {
//     "module": "nodenext",
//     "moduleResolution": "nodenext"
//   }
// }

// app.ts (with NodeNext)
import { helper } from "./foo.js";   // ✅ Required with nodenext
// import { helper } from "./foo";   // ❌ Error with nodenext
console.log(helper());

// Step 3: The difference is in how the runtime resolves modules.
// - Bundlers (Vite, webpack, esbuild) implement their own resolution.
// - Node.js ESM requires full extensions by specification.

// Step 4: Use the appropriate resolution mode for your target.
// - Bundled apps: "bundler"
// - Node.js ESM apps: "nodenext"
// - Libraries: "nodenext" (for maximum compatibility)

console.log("Resolution modes demonstrated.");
```

**Expected Output:**
```
Hello from foo
Resolution modes demonstrated.
```

**Why This Output Occurs:** Bundler resolution allows extensionless imports because the bundler handles resolution. Node.js ESM requires explicit `.js` extensions because Node.js implements the ESM specification strictly.

### Real-World Cases

**Case 1: Node.js ESM Applications**
Node.js applications using `"type": "module"` in `package.json` must use `.js` extensions in relative imports for ESM output.

**Case 2: Library Publishing**
Library authors targeting both Node.js ESM and bundlers use `"nodenext"` resolution and include `.js` extensions in imports, ensuring maximum compatibility.

**Case 3: `tsx` and `ts-node`**
Tools like `tsx` (which uses esbuild) and `ts-node` (with ESM support) respect the `.js` extension requirement when running TypeScript directly.

**Case 4: Migration from CommonJS**
Projects migrating from CommonJS to ESM use tools like `tsc-esm-fix` to automatically add `.js` extensions to existing imports.

**Case 5: Monorepo ESM Packages**
Monorepos with multiple ESM packages use `.js` extensions in cross-package imports to ensure Node.js compatibility.

### References

- TypeScript 4.7 Release Notes: ECMAScript Module Support in Node.js — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-7.html
- TypeScript Modules Guide: Choosing Compiler Options — https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html
- Node.js ESM Documentation: Import Specifiers — https://nodejs.org/api/esm.html#import-specifiers
- `tsc-esm-fix` — https://www.npmjs.com/package/tsc-esm-fix
- `rollup-plugin-esnext-to-nodenext` — https://www.npmjs.com/package/rollup-plugin-esnext-to-nodenext


## Summary: ES Module Feature Comparison

| Feature | Syntax | Purpose | Key Detail |
|---------|--------|---------|------------|
| Named export | `export function f()` | Export multiple values | Tree-shakeable, refactor-safe |
| Default export | `export default f` | Export one primary value | Import name is local, harder to refactor |
| Namespace import | `import * as mod` | Import all exports as object | May hurt tree-shaking |
| Named import | `import { f }` | Import specific values | Type-checked names |
| Re-export (named) | `export { f } from "./mod.js"` | Forward specific exports | Tree-shakeable |
| Re-export (all) | `export * from "./mod.js"` | Forward all exports | May hurt tree-shaking |
| `import type` | `import type { T }` | Type-only import | Erased at runtime |
| `export type` | `export type { T }` | Type-only export | Erased at runtime |
| Inline `type` | `import { f, type T }` | Mixed value/type import | Explicit elision |
| `.js` extension | `import { f } from "./foo.js"` | Node.js ESM resolution | Required for relative imports |
| `moduleResolution: "bundler"` | Extensionless imports | Bundler resolution | Extensions optional |
| `verbatimModuleSyntax` | Enforces explicit `type` | Predictable output | TypeScript 5.0+ |


## References

- TypeScript Handbook: Modules — https://www.typescriptlang.org/docs/handbook/2/modules.html
- TypeScript Handbook: Modules Theory — https://www.typescriptlang.org/docs/handbook/modules/theory.html
- TypeScript Handbook: Modules Reference — https://www.typescriptlang.org/docs/handbook/modules/reference.html
- TypeScript Handbook: Choosing Compiler Options — https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html
- TypeScript 3.8 Release Notes: Type-Only Imports and Exports — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html
- TypeScript 4.7 Release Notes: ECMAScript Module Support in Node.js — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-7.html
- TypeScript 5.0 Release Notes — https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/
- MDN: JavaScript Modules — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- MDN: `import` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import
- MDN: `export` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export
- Node.js ESM Documentation: Import Specifiers — https://nodejs.org/api/esm.html#import-specifiers
- TypeScript `verbatimModuleSyntax` — https://www.typescriptlang.org/tsconfig#verbatimModuleSyntax
- TypeScript `isolatedModules` — https://www.typescriptlang.org/tsconfig#isolatedModules
- TypeScript `esModuleInterop` — https://www.typescriptlang.org/tsconfig#esModuleInterop
- Webpack: Tree Shaking — https://webpack.js.org/guides/tree-shaking/
- Rollup: Tree Shaking — https://rollupjs.org/faqs/#tree-shaking
- `tsc-esm-fix` — https://www.npmjs.com/package/tsc-esm-fix
- `rollup-plugin-esnext-to-nodenext` — https://www.npmjs.com/package/rollup-plugin-esnext-to-nodenext
- `node-ts-resolver` — https://www.npmjs.com/package/node-ts-resolver