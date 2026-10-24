# Declaration Files in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **declaration file** is a TypeScript file with the `.d.ts` extension that contains **only type information** — no executable code and no JavaScript output. It serves as the **type-level counterpart** to an existing JavaScript implementation, allowing TypeScript to type-check code that uses libraries or globals whose implementations live outside the current project. Declaration files are the primary mechanism by which TypeScript provides **static typing over existing JavaScript code** without requiring that code to be rewritten.

**Technical Definition.** Declaration files contain **ambient declarations** — declarations prefixed with `declare` or wrapped in `declare module` / `declare global` blocks — that describe the **types, values, and modules** that exist at runtime but are not defined in the TypeScript source. They participate in **declaration merging**, allowing interfaces, namespaces, and modules to be augmented across files. Declaration files are **erased** during compilation; they produce no `.js` output and exist solely for the type checker. They are resolved by the compiler through `typeRoots`, `types`, and package.json `types`/`typings` fields.

**Beginner-Friendly Explanation.** A declaration file is **a “type-only description” of JavaScript code that already exists**. When you use a JavaScript library that does not include TypeScript types, you write a `.d.ts` file that tells TypeScript what the library’s functions, classes, and variables look like — without writing any implementation. It is like a **menu** that describes the dishes without cooking them. TypeScript uses the menu to check that you order correctly, but the actual cooking happens in the JavaScript library.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Type-only** | Declaration files contain no executable code; they emit no JavaScript. |
| **Ambient by nature** | Top-level declarations must use `declare` or `export`; otherwise the file is treated as a script and errors occur. |
| **Non-emitting** | `.d.ts` files are used only for type checking; they never produce `.js` output. |
| **Mergeable** | Interfaces, namespaces, and modules can merge across declaration files. |
| **Resolution-aware** | The compiler finds them via `typeRoots`, `types`, package.json `types`, or triple-slash references. |
| **Global or modular** | A `.d.ts` file without top-level imports/exports is a **global** declaration file; with imports/exports it is a **module** declaration file. |

---

### Prerequisites

- **TypeScript basics**: type annotations, interfaces, type aliases, and modules.
- **JavaScript runtime knowledge**: how libraries are loaded (script tag, `require`, `import`).
- **`tsconfig.json`**: `typeRoots`, `types`, `include`, and `declaration` options.
- **Module systems**: CommonJS, ES Modules, UMD, and global scripts.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Ambient declarations** — the `declare` keyword and its forms.
- **Declaration merging** — augmenting existing types and modules.
- **Global augmentation** — adding properties to `Window`, `ProcessEnv`, and `globalThis`.
- **Module augmentation** — extending third-party module types.
- **Publishing types** — `types`/`typings` fields and `@types` packages.
- **Library structures** — modular, global, UMD, and module-plugin patterns.

---

## Core Concepts / Features

---

## 1. Declaration Files

### Definitions

**Core Definition.** A **declaration file** (`.d.ts`) is a TypeScript file containing only type-level declarations that describe the shape of existing JavaScript code.

**Technical Definition.** Declaration files provide **ambient declarations** — types, values, and modules — without implementations. They are used by the compiler to type-check imports and global references. They may be **global** (no top-level imports/exports) or **modular** (with imports/exports). They are resolved through package metadata, `typeRoots`, or explicit references.

**Beginner-Friendly Explanation.** A declaration file is **a description of JavaScript code that TypeScript cannot see**. It tells TypeScript what functions and variables exist and what types they use.

### Purposes

- **To type-check JavaScript libraries that ship without TypeScript definitions**.
- **To describe global variables and APIs** injected by scripts or the runtime.
- **To augment existing types** from libraries or the standard library.
- **To publish type information** alongside JavaScript packages.
- **To enable editor intelligence** (autocomplete, go-to-definition) for untyped code.

### Syntax Rules and Structure

**Global declaration file:**

```typescript
// global.d.ts — no top-level import/export
declare const VERSION: string;
declare function greet(name: string): string;
```

**Module declaration file:**

```typescript
// module.d.ts — has top-level export
export function greet(name: string): string;
export const VERSION: string;
```

**Component breakdown:**

- `declare` — marks an ambient declaration.
- `export` — marks a modular declaration.
- Top-level `import`/`export` — determines whether the file is a module.

**Syntax rules.**

- In a `.d.ts` file, top-level declarations must start with `declare` or `export`.
- A file **without** top-level imports/exports is a **global** declaration file.
- A file **with** top-level imports/exports is a **module** declaration file.
- Declaration files produce **no JavaScript output**.

**Constraints and limitations.**

- Declaration files cannot contain executable statements or implementations.
- Global declaration files pollute the global scope; use module files when possible.
- Misconfigured `typeRoots` or `types` can prevent declaration files from being found.

### Annotated Complete Step by Step Code Examples

#### Example 1: Global declaration file

```typescript
// global.d.ts
declare const APP_VERSION: string;
declare function logMessage(message: string): void;
```

```typescript
// main.ts
console.log(APP_VERSION); // Expected: 1.0.0 (injected at runtime)
logMessage("Hello");       // Expected: Hello
```

**Setup.** Place `global.d.ts` in the project and ensure it is included in `tsconfig.json`.

**Expected Output:**
```
1.0.0
Hello
```

**Why this output occurs.** `global.d.ts` is a global declaration file (no top-level imports/exports), so `APP_VERSION` and `logMessage` are available globally without imports.

---

#### Example 2: Module declaration file

```typescript
// math.d.ts
export function add(a: number, b: number): number;
export const PI: number;
```

```typescript
// main.ts
import { add, PI } from "./math.js";
console.log(add(2, 3)); // Expected: 5
console.log(PI);        // Expected: 3.14159
```

**Expected Output:**
```
5
3.14159
```

**Why this output occurs.** `math.d.ts` has top-level `export`, making it a module declaration file. Its exports are imported like any other module.

---

### Real-World Cases

- **`@types/node`** — ambient declarations for Node.js APIs.
- **`lib.dom.d.ts`** — built-in DOM declarations.
- **`@types/lodash`** — type definitions for Lodash.
- **Local `types/` folder** — project-specific global declarations.

---

## 2. `.d.ts`

### Definitions

**Core Definition.** The **`.d.ts` extension** identifies a **declaration file** — a TypeScript file containing only type information, never executable code.

**Technical Definition.** `.d.ts` files are parsed by the compiler but **never emitted** as JavaScript. They are included in the program for type checking and are excluded from runtime output. TypeScript bundles many `.d.ts` files for built-in APIs (`lib.[something].d.ts`). Projects can generate them with `"declaration": true` or `--emitDeclarationOnly`.

**Beginner-Friendly Explanation.** `.d.ts` is **the file extension for declaration files**. When you see a `.d.ts` file, you know it contains only type descriptions, not runnable code.

### Purposes

- **To signal that a file is type-only** and should not be executed.
- **To allow the compiler to include type information** without generating output.
- **To separate type declarations** from implementation files.
- **To publish types** alongside JavaScript in npm packages.

### Syntax Rules and Structure

```
filename.d.ts
```

**Syntax rules.**

- The extension is `.d.ts` (the `.ts` is part of the extension).
- `.d.ts` files are **not** compiled to `.js`.
- They may be generated by `tsc --declaration` or written manually.
- They may be global or modular depending on top-level imports/exports.

**Constraints and limitations.**

- `.d.ts` files cannot contain executable code or implementations.
- They are excluded from runtime bundles by design.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generating `.d.ts` from TypeScript

```json
// tsconfig.json
{
  "compilerOptions": {
    "declaration": true,
    "emitDeclarationOnly": true,
    "outDir": "types"
  }
}
```

```typescript
// src/math.ts
export function add(a: number, b: number): number {
  return a + b;
}
```

**Generated `types/math.d.ts`:**
```typescript
export declare function add(a: number, b: number): number;
```

**Expected Output:** No runtime output; only declaration files are emitted.

**Why this output occurs.** `emitDeclarationOnly` tells the compiler to generate only `.d.ts` files, not JavaScript.

---

### Real-World Cases

- **Publishing libraries** with `types` field pointing to `.d.ts` files.
- **Monorepos** sharing type definitions between packages.
- **Ambient declarations** for global scripts.

---

## 3. `declare`

### Definitions

**Core Definition.** The **`declare` keyword** creates an **ambient declaration** — it tells TypeScript that a variable, function, class, namespace, or module exists at runtime without providing an implementation.

**Technical Definition.** `declare` may be applied to variables, functions, classes, enums, namespaces, and modules. It creates an **ambient context** in which no implementation is required or allowed. Ambient declarations are **erased** and produce no JavaScript. They are the primary building block of `.d.ts` files.

**Beginner-Friendly Explanation.** `declare` means **“trust me, this exists at runtime — do not look for an implementation”** . It is how you tell TypeScript about code that lives outside your TypeScript files.

### Purposes

- **To describe runtime values without implementations**.
- **To type global variables** injected by scripts or the environment.
- **To declare external modules** that lack type definitions.
- **To augment existing declarations** in ambient contexts.
- **To separate type information** from implementation.

### Syntax Rules and Structure

```typescript
declare const name: Type;
declare function fn(params: Type): ReturnType;
declare class C { ... }
declare namespace N { ... }
declare module "module-name" { ... }
```

**Syntax rules.**

- `declare` is used at the top level of a `.d.ts` file or inside `declare module` / `declare global` blocks.
- Ambient declarations **cannot have implementations**.
- `declare` can be combined with `const`, `let`, `var`, `function`, `class`, `enum`, `namespace`, and `module`.

**Constraints and limitations.**

- `declare` declarations produce no JavaScript.
- Using `declare` inside a `.ts` file creates a local ambient declaration, not a global one.
- Incorrect `declare` usage can cause “Cannot redeclare block-scoped variable” errors.

### Annotated Complete Step by Step Code Examples

#### Example 1: `declare` for a global constant

```typescript
// global.d.ts
declare const DEBUG: {
  getState(): { id: string };
};
```

```typescript
// main.ts
const state = DEBUG.getState();
console.log(state.id); // Expected: "abc123" (injected at runtime)
```

**Expected Output:**
```
abc123
```

**Why this output occurs.** `declare const DEBUG` tells TypeScript that `DEBUG` exists globally with a `getState` method. No implementation is provided; the runtime injects it.

---

#### Example 2: `declare` inside a module (local ambient)

```typescript
// main.ts
declare const LOCAL_DEBUG: { log(): void };

LOCAL_DEBUG.log(); // Expected: "debug"
```

**Expected Output:**
```
debug
```

**Why this output occurs.** `declare` inside a `.ts` file creates a **local** ambient declaration. `LOCAL_DEBUG` is not added to the global scope; it is only visible in `main.ts`.

---

### Real-World Cases

- **`declare const __DEV__: boolean`** in React Native.
- **`declare global { interface Window { analytics: ... } }`** for analytics scripts.
- **`declare module "*.css"`** for CSS module imports.

---

## 4. Declaring External Libraries

### Definitions

**Core Definition.** **Declaring external libraries** means writing declaration files (or module declarations) that describe the types of JavaScript libraries or modules that do not ship with TypeScript definitions.

**Technical Definition.** External libraries may be **modular** (used via `import`/`require`), **global** (exposed on `window`), or **UMD** (both). The declaration file must match the library’s **structure**. For modular libraries, use `declare module "name" { ... }` with exports. For global libraries, use ambient global declarations.

**Beginner-Friendly Explanation.** When you use a JavaScript library that has no TypeScript types, you write a **declaration file that describes its API** so TypeScript can check your usage.

### Purposes

- **To type-check third-party JavaScript libraries** without type definitions.
- **To provide autocomplete and IntelliSense** for untyped libraries.
- **To integrate legacy JavaScript code** into TypeScript projects.
- **To describe non-JavaScript assets** (CSS, images, SVGs) with wildcard module declarations.

### Syntax Rules and Structure

**Modular library:**

```typescript
declare module "library-name" {
  export function doSomething(input: string): number;
  export class SomeClass { ... }
}
```

**Global library:**

```typescript
declare namespace MyLib {
  function makeGreeting(s: string): string;
}
```

**Wildcard module:**

```typescript
declare module "*.png" {
  const value: string;
  export default value;
}
```

**Syntax rules.**

- `declare module "name"` creates a module declaration.
- Wildcards (`*`) are allowed in module specifiers.
- Global libraries use `declare namespace` or global `declare` declarations.
- Declarations must match how the library is consumed (import vs. global).

**Constraints and limitations.**

- Relative module names are **not allowed** in ambient module declarations.
- Declaration files without top-level imports/exports are global; adding imports/exports changes them to modules.
- Incorrect structure causes “Cannot find module” errors.

### Annotated Complete Step by Step Code Examples

#### Example 1: Modular library declaration

```typescript
// types/old-carousel.d.ts
declare module "old-carousel-library" {
  export default class Carousel {
    constructor(element: HTMLElement, options?: any);
    next(): void;
    prev(): void;
    destroy(): void;
  }
}
```

```typescript
// main.ts
import Carousel from "old-carousel-library";
const carousel = new Carousel(document.body);
carousel.next(); // Expected: carousel advances
```

**Expected Output:** Carousel advances (runtime behavior).

**Why this output occurs.** The module declaration describes the library’s default export and methods, allowing TypeScript to check the usage.

---

#### Example 2: Wildcard declaration for assets

```typescript
// types/assets.d.ts
declare module "*.png" {
  const src: string;
  export default src;
}
```

```typescript
// main.ts
import logo from "./logo.png";
console.log(logo); // Expected: "/assets/logo.png"
```

**Expected Output:**
```
/assets/logo.png
```

**Why this output occurs.** The wildcard module declaration matches any `.png` import and types the default export as a string.

---

### Real-World Cases

- **CSS Modules**: `declare module "*.module.css"`.
- **SVG components**: `declare module "*.svg"`.
- **Untyped npm packages**: `declare module "old-library"`.

---

## 5. Declaration Merging

### Definitions

**Core Definition.** **Declaration merging** is TypeScript’s ability to merge **multiple declarations with the same name** into a single definition with the combined features of all declarations.

**Technical Definition.** Merging applies to **interfaces**, **namespaces**, **enums**, and **modules**. Non-function members must be unique or have identical types; function members become overloads. Namespaces merge with classes, functions, and enums to add static members or nested types. Global augmentation and module augmentation rely on declaration merging.

**Beginner-Friendly Explanation.** Declaration merging means **if you declare the same thing twice, TypeScript combines them**. It is how libraries let you add properties to existing types.

### Purposes

- **To augment existing interfaces** with new properties.
- **To add static members to classes** via namespace merging.
- **To extend third-party modules** with additional types.
- **To split large interfaces** across multiple files.
- **To implement global augmentation** safely.

### Syntax Rules and Structure

```typescript
interface Box {
  width: number;
}

interface Box {
  height: number;
}

// Box now has width and height.
```

**Syntax rules.**

- Interfaces merge if declared with the same name in the same scope.
- Non-function members must have identical types; otherwise the compiler errors.
- Function members become overloads; later declarations have higher precedence.
- Namespaces merge with classes, functions, and enums when the namespace follows the declaration.

**Constraints and limitations.**

- Type aliases **do not merge**.
- Non-exported namespace members do not merge across declarations.
- Over-merging can cause naming collisions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Interface merging

```typescript
// a.ts
interface User {
  id: string;
}

// b.ts
interface User {
  name: string;
}
```

```typescript
// main.ts
const user: User = { id: "u1", name: "Ada" };
console.log(user.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The two `User` interfaces merge into one requiring both `id` and `name`.

---

#### Example 2: Namespace merging with a class

```typescript
class Album {
  label: Album.AlbumLabel;
}

namespace Album {
  export class AlbumLabel {}
}
```

```typescript
const album = new Album();
album.label = new Album.AlbumLabel();
console.log(album.label instanceof Album.AlbumLabel); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** The namespace adds `AlbumLabel` as a static-side nested class on `Album`.

---

### Real-World Cases

- **Express Request augmentation**: adding `user` to `Express.Request`.
- **Window augmentation**: adding `analytics` to `Window`.
- **Redux module augmentation**: extending `DefaultRootState`.

---

## 6. Ambient Declarations

### Definitions

**Core Definition.** **Ambient declarations** are declarations that describe types and values existing at runtime without providing implementations, created with the `declare` keyword.

**Technical Definition.** Ambient declarations include `declare const`, `declare function`, `declare class`, `declare namespace`, and `declare module`. They are **non-emitting** and are typically placed in `.d.ts` files. They provide **static typing over existing JavaScript code**.

**Beginner-Friendly Explanation.** Ambient declarations are **type descriptions of code that already exists** — you declare the types, but you do not write the code.

### Purposes

- **To type existing JavaScript code** without rewriting it.
- **To declare global variables** injected by scripts.
- **To describe external modules** that lack types.
- **To separate type information** from implementation.

### Syntax Rules and Structure

```typescript
declare const VERSION: string;
declare function greet(name: string): string;
declare class LegacyClass { method(): void; }
declare module "legacy-lib" { ... }
```

**Syntax rules.**

- Ambient declarations use `declare`.
- They cannot have implementations.
- They are erased and produce no JavaScript.
- They can be global or module-scoped.

**Constraints and limitations.**

- Ambient declarations do not enforce runtime existence; they are assertions.
- Overusing ambient global declarations can pollute the global scope.

### Annotated Complete Step by Step Code Examples

#### Example 1: Ambient function declaration

```typescript
// ambient.d.ts
declare function trackEvent(name: string, data?: Record<string, unknown>): void;
```

```typescript
// main.ts
trackEvent("click", { button: "submit" }); // Expected: event tracked
```

**Expected Output:** Event is tracked at runtime.

**Why this output occurs.** The ambient declaration tells TypeScript that `trackEvent` exists globally with the given signature.

---

### Real-World Cases

- **Google Analytics**: `declare function gtag(...args: any[]): void`.
- **Stripe.js**: ambient declarations for `Stripe`.
- **Legacy global scripts**: `declare const $: JQueryStatic`.

---

## 7. Global Declarations

### Definitions

**Core Definition.** **Global declarations** add types or values to the **global scope**, making them available in every file without imports. They are created by global `.d.ts` files or `declare global` blocks inside modules.

**Technical Definition.** A global `.d.ts` file has **no top-level imports/exports** and uses `declare` at the top level. A `declare global` block inside a module augments the global scope. Global augmentation is used to extend `Window`, `ProcessEnv`, `globalThis`, and other global interfaces.

**Beginner-Friendly Explanation.** Global declarations are **types that are available everywhere** — you do not need to import them.

### Purposes

- **To declare global variables** injected by scripts.
- **To augment built-in global interfaces** (`Window`, `ProcessEnv`).
- **To provide project-wide types** without imports.
- **To integrate environment-specific globals** (Node.js, browser).

### Syntax Rules and Structure

**Global `.d.ts`:**

```typescript
// global.d.ts — no imports/exports
declare const MY_GLOBAL: string;
```

**`declare global` inside a module:**

```typescript
// augment.ts — must have an import/export
export {};

declare global {
  interface Window {
    analytics: { track(event: string): void };
  }
}
```

**Syntax rules.**

- Global `.d.ts` files must **not** have top-level imports/exports.
- `declare global` blocks must be inside a **module** (a file with at least one import/export).
- Global augmentations are visible in all files.

**Constraints and limitations.**

- Global declarations pollute the global namespace and can cause collisions.
- `declare global` does not work in a script file; the file must be a module.

### Annotated Complete Step by Step Code Examples

#### Example 1: Global `.d.ts` file

```typescript
// debug.d.ts
declare const DEBUG: {
  getState(): { id: string };
};
```

```typescript
// main.ts
const state = DEBUG.getState();
console.log(state.id); // Expected: "abc123"
```

**Expected Output:**
```
abc123
```

**Why this output occurs.** The global `.d.ts` file has no imports/exports, so `DEBUG` is available globally.

---

#### Example 2: `declare global` inside a module

```typescript
// analytics.ts
export {};

declare global {
  interface Window {
    analytics: {
      track(event: string): void;
    };
  }
}
```

```typescript
// main.ts
window.analytics.track("click"); // Expected: event tracked
```

**Expected Output:** Event tracked.

**Why this output occurs.** The `declare global` block augments the global `Window` interface with `analytics`.

---

### Real-World Cases

- **Environment variables**: augmenting `ProcessEnv`.
- **Analytics**: adding `window.dataLayer`.
- **Polyfills**: adding global functions.

---

## 8. Module Declarations

### Definitions

**Core Definition.** A **module declaration** describes the types of an **external module** (a JavaScript module identified by a specifier) using `declare module "name" { ... }`.

**Technical Definition.** Module declarations may describe **modular libraries**, **wildcard patterns** (`*.css`), **UMD libraries**, or **module augmentation** (extending an existing module). They may contain exports, functions, classes, interfaces, and nested namespaces. Module declarations are **ambient** and produce no JavaScript.

**Beginner-Friendly Explanation.** A module declaration is **a description of an external module** — it tells TypeScript what the module exports so you can import it safely.

### Purposes

- **To type external libraries** that lack declarations.
- **To type non-JavaScript assets** (CSS, images, SVGs).
- **To augment existing modules** with additional exports.
- **To support UMD libraries** that work as both global and modular.

### Syntax Rules and Structure

```typescript
declare module "module-name" {
  export function fn(): void;
  export class C { ... }
  export default function main(): void;
}
```

**Wildcard module:**

```typescript
declare module "*.css" {
  const styles: { [key: string]: string };
  export default styles;
}
```

**Syntax rules.**

- `declare module "name"` creates an ambient module declaration.
- Wildcards (`*`) match families of module names.
- Relative module names are **not allowed**.
- Default and named exports are supported.

**Constraints and limitations.**

- Module declarations do not create runtime modules; they only describe them.
- Wildcard declarations can accidentally match unintended modules if too broad.

### Annotated Complete Step by Step Code Examples

#### Example 1: Modular library declaration

```typescript
// types/mylib.d.ts
declare module "mylib" {
  export function greet(name: string): string;
}
```

```typescript
// main.ts
import { greet } from "mylib";
console.log(greet("Ada")); // Expected: "Hello, Ada"
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The module declaration tells TypeScript that `mylib` exports `greet` with the given signature.

---

#### Example 2: CSS module wildcard declaration

```typescript
// types/css-modules.d.ts
declare module "*.module.css" {
  const classes: { [key: string]: string };
  export default classes;
}
```

```typescript
// main.ts
import styles from "./Button.module.css";
console.log(styles.primary); // Expected: "primary_hash"
```

**Expected Output:**
```
primary_hash
```

**Why this output occurs.** The wildcard declaration matches any `.module.css` import and types the default export as a class-name map.

---

### Real-World Cases

- **CSS Modules**: `declare module "*.module.css"`.
- **SVG components**: `declare module "*.svg"`.
- **Untyped libraries**: `declare module "old-library"`.
- **Module augmentation**: extending `express` or `fastify`.

---

## 9. Declaration File Best Practices

### Definitions

**Core Definition.** **Declaration file best practices** are guidelines for writing `.d.ts` files that are **accurate, maintainable, and non-polluting**, following the TypeScript Handbook’s Do’s and Don’ts.

**Technical Definition.** Best practices include **starting from documentation rather than code**, **using the correct library structure** (modular, global, UMD), **avoiding global pollution**, **using `declare global` only when necessary**, **keeping declaration files type-only**, **using `export` in module files**, and **validating with `tsc`**. The TypeScript Handbook recommends: “The best way to write a `.d.ts` file is to start from the documentation of the library, not the code.”

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep declaration files accurate and safe.

### Purposes

- **To write accurate declarations** that match runtime behavior.
- **To avoid global namespace pollution**.
- **To choose the correct structure** for the library type.
- **To keep declaration files type-only** and non-executable.
- **To ensure declarations are included** in compilation via `typeRoots` or `include`.

### Syntax Rules and Structure

Best practices are conventions that interact with declaration syntax. The core rules are:

1. **Start from documentation, not code.** Use the library’s documented API to write declarations.
2. **Match the library structure.** Use modular declarations for modules, global declarations for globals, and UMD patterns for UMD.
3. **Avoid global pollution.** Prefer module declaration files over global `.d.ts` files when possible.
4. **Use `declare global` only in modules.** It requires at least one import/export in the file.
5. **Keep declarations type-only.** Do not put implementation code in `.d.ts` files.
6. **Use `export` in module files.** Top-level `export` makes the file a module and avoids global leakage.
7. **Validate with the compiler.** Run `tsc --noEmit` to ensure declarations are correct.
8. **Place declaration files in a `types/` directory** and include them in `tsconfig.json`.

**Component breakdown (declaration file decision tree):**

```
Does the library use import/require?
  ├── Yes → Use `declare module "name" { ... }`.
  └── No → Does it attach to a global object?
           ├── Yes → Use a global .d.ts file with `declare`.
           └── No → Does it support both?
                    └── Use a UMD-style declaration.
```

**Constraints and limitations.**

- Global declaration files cannot contain top-level imports/exports.
- `declare global` requires a module file.
- Overly broad wildcard declarations can hide errors.

### Annotated Complete Step by Step Code Examples

#### Example 1: Correct module declaration file

```typescript
// types/mylib.d.ts
export function greet(name: string): string;
export const VERSION: string;
```

```typescript
// main.ts
import { greet, VERSION } from "mylib";
console.log(greet("Ada")); // Expected: "Hello, Ada"
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The top-level `export` makes the file a module declaration file, avoiding global pollution and correctly describing the module’s API.

---

#### Example 2: Avoiding global pollution with `declare global`

```typescript
// analytics.ts
export {};

declare global {
  interface Window {
    analytics: { track(event: string): void };
  }
}
```

**Expected Output:** `window.analytics` is typed globally.

**Why this output occurs.** The `export {}` makes the file a module, and `declare global` augments the global `Window` interface.

---

### Real-World Cases

- **Publishing types** with `types`/`typings` in package.json.
- **DefinitelyTyped** contributions.
- **Monorepo shared types** in a `types/` folder.
- **Third-party module augmentation** (e.g., `express-session`).

---

## References

- TypeScript Handbook — Declaration Files: Introduction - https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html
- TypeScript Handbook — Type Declarations - https://www.typescriptlang.org/docs/handbook/2/type-declarations.html
- TypeScript Handbook — Declaration Merging - https://www.typescriptlang.org/docs/handbook/declaration-merging.html
- TypeScript Handbook — Library Structures - https://www.typescriptlang.org/docs/handbook/declaration-files/library-structures.html
- TypeScript Handbook — Modules .d.ts - https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-d-ts.html
- TypeScript Handbook — Deep Dive - https://www.typescriptlang.org/docs/handbook/declaration-files/deep-dive.html
- TypeScript Handbook — Do’s and Don’ts - https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html
- Total TypeScript — Ambient Context and the `declare` Keyword - https://www.totaltypescript.com/workshops/typescript-pro-essentials/modules-scripts-and-declaration-files/type-variables-declared-elsewhere/solution
- Total TypeScript — Understanding Global Declarations - https://www.totaltypescript.com/workshops/typescript-pro-essentials/modules-scripts-and-declaration-files/global-scope-declarations-in-typescript/solution
- Total TypeScript — Use Wildcards in TypeScript Module Declarations - https://www.totaltypescript.com/workshops/typescript-pro-essentials/types-you-don't-control/importing-and-typing-non-code-files-in-typescript/solution
- Steve Kinney — Modules and Declaration Files for React - https://stevekinney.com/courses/react-typescript/typescript-modules-declarations
- TypeScript Documentation — Global: Modifying Module - https://www.typescriptlang.org/docs/handbook/declaration-files/templates/global-modifying-module-d-ts.html
- TypeScript Documentation — `declare` Keyword (Ambient Declarations) - https://www.typescriptlang.org/docs/handbook/declaration-files/by-example.html
- TypeScript 7.1 Beta — Wildcard Ambient Module Declarations - https://devblogs.microsoft.com/typescript/announcing-typescript-7-1-beta/