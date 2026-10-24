# Type Declarations in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **type declaration** is a TypeScript construct that describes the **shape, type, and existence** of a value or module without providing an implementation. Declarations are introduced with the `declare` keyword and are the foundation of **ambient declarations**—type-level assertions that tell the compiler “this entity exists at runtime, but its implementation is elsewhere.” Type declarations are the building blocks of `.d.ts` declaration files and are essential for typing JavaScript libraries, global APIs, and environment-injected values.

**Technical Definition.** The `declare` keyword creates an **ambient declaration**—a declaration that exists only in the type space, produces no JavaScript output, and asserts the existence of a runtime entity. TypeScript supports several declaration forms, each corresponding to a runtime construct: `declare var`/`declare let`/`declare const` for variables, `declare function` for functions, `declare class` for classes, `declare namespace` for namespaced values and types, `declare module` for external modules, and `declare global` for augmenting the global scope. All forms are **erased** during compilation and participate in **declaration merging** where applicable.

**Beginner-Friendly Explanation.** A type declaration is **you telling TypeScript that something exists at runtime, without writing the actual code**. You use the `declare` keyword to describe the shape of a variable, function, class, or module that lives outside your TypeScript files—like a global function injected by a script tag, or a JavaScript library with no types. TypeScript uses these declarations to type-check your code, but they produce **zero JavaScript output**. They are like a **casting call description**: they describe the actors (types) without hiring them (providing implementations).

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Non-emitting** | All `declare` declarations are erased at compile time and produce no JavaScript. |
| **Ambient by nature** | Declarations describe runtime entities that exist outside the current TypeScript compilation. |
| **Mergable** | Namespaces, interfaces, and modules can merge across multiple declarations. |
| **Form-specific** | Each declaration form (`var`, `function`, `class`, `namespace`, `module`, `global`) maps to a different runtime construct. |
| **Scope-sensitive** | A `.d.ts` file without top-level imports/exports is **global**; with imports/exports it is a **module**. |
| **No implementations** | `declare` declarations cannot contain executable code or function bodies. |

---

### Prerequisites

- **TypeScript basics**: type annotations, interfaces, type aliases, and modules.
- **JavaScript runtime knowledge**: how libraries are loaded (script tag, `require`, `import`).
- **`tsconfig.json`**: `declaration`, `typeRoots`, and `include` options.
- **Module systems**: CommonJS, ES Modules, UMD, and global scripts.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Declaration files** (`.d.ts`) — the primary container for type declarations.
- **Ambient declarations** — the `declare` keyword and its forms.
- **Declaration merging** — augmenting existing types and modules.
- **Global augmentation** — extending `Window`, `ProcessEnv`, and `globalThis`.
- **Module augmentation** — extending third-party module types.
- **Publishing types** — `types`/`typings` fields and `@types` packages.

---

## Core Concepts / Features

---

## 1. Type Declarations

### Definitions

**Core Definition.** A **type declaration** is a `declare`-prefixed statement that describes the type of a runtime entity without providing its implementation.

**Technical Definition.** Type declarations use the `declare` keyword to introduce **ambient** entities into the type space. They may declare variables, functions, classes, namespaces, modules, and global augmentations. All forms are **erased** and produce no JavaScript. Declarations are typically placed in `.d.ts` files but may also appear inside `declare module` or `declare global` blocks in regular `.ts` files.

**Beginner-Friendly Explanation.** A type declaration is **a type-only description of something that exists at runtime**. It tells TypeScript what to expect without writing the actual code.

### Purposes

- **To type-check existing JavaScript code** without rewriting it in TypeScript.
- **To describe global variables and APIs** injected by scripts or the runtime.
- **To type external libraries** that lack TypeScript definitions.
- **To augment existing types** from libraries or the standard library.
- **To separate type information from implementation** in library packaging.

### Syntax Rules and Structure

```typescript
declare const/let/var name: Type;
declare function name(params: Type): ReturnType;
declare class ClassName { ... }
declare namespace NamespaceName { ... }
declare module "module-name" { ... }
declare global { ... }
```

**Syntax rules.**

- `declare` must precede the declaration keyword.
- All forms are ambient and cannot include implementations.
- Declarations are erased and produce no JavaScript.
- Top-level imports/exports determine whether a `.d.ts` file is global or modular.

**Constraints and limitations.**

- `declare` declarations cannot contain executable code.
- Incorrect `declare` usage can cause “Cannot redeclare block-scoped variable” errors.
- Global declarations pollute the global scope.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic type declarations

```typescript
// globals.d.ts
declare const APP_VERSION: string;
declare function logMessage(message: string): void;
declare class LegacyWidget {
  constructor(element: HTMLElement);
  render(): void;
}
```

```typescript
// main.ts
console.log(APP_VERSION);          // Expected: "1.0.0"
logMessage("Hello");                // Expected: "Hello"
const widget = new LegacyWidget(document.body);
widget.render();                    // Expected: renders widget
```

**Expected Output:**
```
1.0.0
Hello
```

**Why this output occurs.** The `globals.d.ts` file describes `APP_VERSION`, `logMessage`, and `LegacyWidget` without implementing them. At runtime, the implementations are injected elsewhere.

---

### Real-World Cases

- **`@types/node`** — ambient declarations for Node.js APIs.
- **`lib.dom.d.ts`** — built-in DOM declarations.
- **Local `types/` folder** — project-specific global declarations.

---

## 2. `declare module`

### Definitions

**Core Definition.** `declare module "name" { ... }` declares an **external module** and its exports. It describes how a JavaScript module is shaped, allowing TypeScript to type-check `import` statements for that module.

**Technical Definition.** A module declaration is an **ambient module declaration** that may contain exports, functions, classes, interfaces, and nested namespaces. It may use **wildcards** (`*.css`) to match families of module specifiers. Module declarations are erased and produce no JavaScript. They are used for libraries without types, non-JavaScript assets, and module augmentation.

**Beginner-Friendly Explanation.** `declare module` is **how you describe a JavaScript module that TypeScript does not know about**. It tells TypeScript what the module exports so you can import it safely.

### Purposes

- **To type external libraries** that lack declaration files.
- **To type non-JavaScript assets** (CSS, images, SVGs) with wildcard declarations.
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

- Module names must be **string literals**; relative module names are not allowed.
- Wildcards (`*`) match families of module names.
- Default and named exports are supported.
- Module declarations must be in `.d.ts` files or ambient contexts.

**Constraints and limitations.**

- Module declarations do not create runtime modules; they only describe them.
- Overly broad wildcards can hide errors.

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

**Why this output occurs.** The module declaration describes `mylib`’s exports, allowing TypeScript to type-check the import.

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
- **Untyped npm packages**: `declare module "old-library"`.
- **Module augmentation**: extending `express` or `fastify`.

---

## 3. `declare global`

### Definitions

**Core Definition.** `declare global { ... }` augments the **global scope** from within a **module file**. It allows adding properties to global interfaces such as `Window`, `ProcessEnv`, `Array`, and `globalThis`.

**Technical Definition.** `declare global` is a **global augmentation** block that must appear inside a **module** (a file with at least one top-level import or export). It merges new declarations into the global scope. It is the standard way to extend built-in global types without polluting the module’s own scope.

**Beginner-Friendly Explanation.** `declare global` is **how you add things to the global scope from inside a module**. For example, you can add `window.analytics` or extend `Array.prototype` from a TypeScript file.

### Purposes

- **To augment built-in global interfaces** (`Window`, `ProcessEnv`, `Array`).
- **To declare environment-injected globals** from within a module.
- **To extend the global scope safely** without creating a global `.d.ts` file.
- **To add properties to `globalThis`** and other platform globals.

### Syntax Rules and Structure

```typescript
// augment.ts (must be a module)
export {};

declare global {
  interface Window {
    analytics: { track(event: string): void };
  }
  var __CACHE: Map<string, unknown>;
}
```

**Syntax rules.**

- `declare global` must be inside a **module** (file with at least one import/export).
- `export {}` is a common way to mark a file as a module.
- The block may contain interfaces, types, variables, and functions.
- Augmentations merge into the global scope.

**Constraints and limitations.**

- `declare global` does not work in a script file (no imports/exports).
- Global augmentation can cause naming collisions.
- Libraries should **never** publish global augmentations.

### Annotated Complete Step by Step Code Examples

#### Example 1: Augmenting `Window`

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

**Why this output occurs.** The `declare global` block merges `analytics` into the global `Window` interface, making `window.analytics` type-safe.

---

### Real-World Cases

- **Analytics scripts**: adding `window.dataLayer`.
- **Polyfills**: adding global functions.
- **Environment variables**: augmenting `ProcessEnv`.

---

## 4. `declare namespace`

### Definitions

**Core Definition.** `declare namespace Name { ... }` declares a **namespace** containing types, values, functions, classes, and nested namespaces. It is used to describe **global objects accessed by dotted notation** (e.g., `MyLib.makeGreeting`).

**Technical Definition.** A namespace declaration creates a **global value** and a **type container**. Exported members are accessible via dotted notation. Namespaces may merge across declarations and with classes, functions, and enums. They are typically used in **ambient declarations** for global libraries.

**Beginner-Friendly Explanation.** `declare namespace` is **how you describe a global object that has nested properties and types**. It is used for libraries that attach themselves to a global name.

### Purposes

- **To describe global libraries** accessed by dotted notation (e.g., `jQuery.ajax`).
- **To organize types** in ambient declaration files.
- **To merge with classes or functions** for static-side augmentation.
- **To provide a single global name** for many related declarations.

### Syntax Rules and Structure

```typescript
declare namespace MyLib {
  function makeGreeting(s: string): string;
  interface Options {
    verbose?: boolean;
  }
  namespace Sub {
    function nested(): void;
  }
}
```

**Syntax rules.**

- Members are accessed as `MyLib.makeGreeting`.
- Only `export`ed members (or all members in ambient namespaces) are visible.
- Namespaces may be nested.
- Namespaces merge with classes, functions, and enums.

**Constraints and limitations.**

- Namespaces pollute the global scope.
- Google’s Style Guide disallows `namespace Foo { ... }` except for third-party interop.

### Annotated Complete Step by Step Code Examples

#### Example 1: Ambient namespace

```typescript
// mylib.d.ts
declare namespace MyLib {
  function makeGreeting(s: string): string;
  interface Options {
    verbose?: boolean;
  }
}
```

```typescript
// main.ts
const greeting = MyLib.makeGreeting("Ada");
console.log(greeting); // Expected: "Hello, Ada"
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The ambient namespace declares the global `MyLib` object with `makeGreeting` and `Options`.

---

### Real-World Cases

- **jQuery**: `declare namespace $`.
- **Express**: `declare namespace Express`.
- **Legacy global libraries**: `declare namespace MyLib`.

---

## 5. `declare function`

### Definitions

**Core Definition.** `declare function name(params): ReturnType;` declares a **global function** that exists at runtime but has no TypeScript implementation.

**Technical Definition.** A function declaration creates an **ambient function** with a signature but no body. It may be **overloaded** with multiple signatures. The function is available globally (or within the ambient context) and is type-checked at call sites.

**Beginner-Friendly Explanation.** `declare function` is **how you describe a global function that TypeScript cannot see**. You write the signature, and TypeScript checks your calls.

### Purposes

- **To type global functions** injected by scripts or the runtime.
- **To declare function overloads** for untyped libraries.
- **To describe callbacks and function-type APIs** in ambient contexts.

### Syntax Rules and Structure

```typescript
declare function greet(name: string): string;
```

**Overloads:**

```typescript
declare function duration(): number;
declare function duration(value: number): void;
```

**Syntax rules.**

- The declaration ends with a semicolon, not a body.
- Multiple declarations with the same name create overloads.
- Parameters may be optional, defaulted, or rest.

**Constraints and limitations.**

- Cannot contain implementation logic.
- Must match the actual runtime behavior.

### Annotated Complete Step by Step Code Examples

#### Example 1: Global function declaration

```typescript
// globals.d.ts
declare function trackEvent(category: string, action: string): void;
```

```typescript
// main.ts
trackEvent("button", "click"); // Expected: event tracked
```

**Expected Output:** Event tracked.

**Why this output occurs.** The declaration tells TypeScript that `trackEvent` exists globally with the given signature.

---

### Real-World Cases

- **Google Analytics**: `declare function gtag(...args: any[]): void`.
- **Stripe.js**: ambient declarations for `Stripe`.
- **Legacy global scripts**: `declare function $(selector: string): HTMLElement`.

---

## 6. `declare class`

### Definitions

**Core Definition.** `declare class ClassName { ... }` declares a **global class** with a constructor, properties, and methods, but no implementation.

**Technical Definition.** A class declaration creates an **ambient class** with a constructor signature and member declarations. It may have **multiple constructors** (overloads) and may include static members. The class is available globally and is type-checked at instantiation and member access.

**Beginner-Friendly Explanation.** `declare class` is **how you describe a global class that exists at runtime**. You write its constructor and methods, and TypeScript checks your usage.

### Purposes

- **To type global classes** from libraries or the runtime.
- **To declare class overloads** for constructors.
- **To provide static-side types** for global objects.

### Syntax Rules and Structure

```typescript
declare class MyClass {
  constructor(id: number);
  constructor();
  method(): void;
  static staticMethod(): void;
}
```

**Syntax rules.**

- Constructors may be overloaded.
- Members are declared without bodies.
- Static members are supported.
- The class may merge with a namespace for nested types.

**Constraints and limitations.**

- Cannot contain method implementations.
- Must match the runtime class’s shape.

### Annotated Complete Step by Step Code Examples

#### Example 1: Global class declaration

```typescript
// globals.d.ts
declare class LegacyWidget {
  constructor(element: HTMLElement);
  render(): void;
  destroy(): void;
}
```

```typescript
// main.ts
const widget = new LegacyWidget(document.body);
widget.render(); // Expected: widget renders
```

**Expected Output:** Widget renders.

**Why this output occurs.** The declaration describes the constructor and methods of the global `LegacyWidget` class.

---

### Real-World Cases

- **Global UI libraries**: `declare class Carousel`.
- **Legacy frameworks**: `declare class Backbone.Model`.
- **Browser APIs**: ambient classes for non-standard globals.

---

## 7. `declare const` and `declare var`

### Definitions

**Core Definition.** `declare const name: Type;` and `declare var name: Type;` declare **global variables** without implementations. `const` creates a read-only binding; `var` creates a mutable global binding.

**Technical Definition.** Both forms create **ambient variables**. `declare const` is for immutable globals; `declare var` is for mutable globals (and is required for `globalThis` augmentation). `declare let` is also available for block-scoped mutable globals. The value is typed but not implemented.

**Beginner-Friendly Explanation.** `declare const` and `declare var` are **how you describe global variables**. Use `const` for things that never change and `var` for things that can be reassigned.

### Purposes

- **To type global constants** injected by the environment.
- **To type global mutable variables** on `window` or `globalThis`.
- **To declare configuration flags** like `__DEV__`.

### Syntax Rules and Structure

```typescript
declare const VERSION: string;
declare var __DEV__: boolean;
declare let mutableCount: number;
```

**Syntax rules.**

- `const` creates a read-only binding.
- `var` creates a mutable global binding.
- `let` creates a block-scoped mutable binding.
- `var` is required for some `globalThis` augmentations.

**Constraints and limitations.**

- The declared value must actually exist at runtime.
- `const` cannot be reassigned.

### Annotated Complete Step by Step Code Examples

#### Example 1: Global constant and variable

```typescript
// globals.d.ts
declare const APP_VERSION: string;
declare var featureFlags: {
  darkMode: boolean;
  beta: boolean;
};
```

```typescript
// main.ts
console.log(APP_VERSION); // Expected: "1.0.0"
if (featureFlags.darkMode) {
  console.log("Dark mode enabled"); // Expected: Dark mode enabled
}
```

**Expected Output:**
```
1.0.0
Dark mode enabled
```

**Why this output occurs.** The declarations describe the global `APP_VERSION` and `featureFlags` objects.

---

### Real-World Cases

- **Build-time constants**: `declare const __DEV__: boolean`.
- **Environment configuration**: `declare var process: { env: ... }`.
- **Global mutable state**: `declare var __CACHE: Map<string, unknown>`.

---

## 8. Type Declaration Best Practices

### Definitions

**Core Definition.** **Type declaration best practices** are guidelines for writing accurate, maintainable, and non-polluting declarations, following the TypeScript Handbook, DefinitelyTyped, and community standards.

**Technical Definition.** Best practices center on **starting from documentation, not code**, **matching the library’s structure** (modular, global, UMD), **avoiding global pollution**, **using `declare global` only in modules**, **keeping declarations type-only**, **validating with `tsc`**, and **shipping declarations with libraries**.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep type declarations accurate and safe.

### Purposes

- **To write accurate declarations** that match runtime behavior.
- **To avoid global namespace pollution**.
- **To choose the correct declaration form** for each entity.
- **To keep declaration files type-only** and non-executable.
- **To ensure declarations are included** in compilation.

### Syntax Rules and Structure

Best practices are conventions that interact with declaration syntax. The core rules are:

1. **Start from documentation, not code.** Use the library’s documented API to write declarations.
2. **Match the library structure.** Use `declare module` for modular libraries, global declarations for globals, and UMD patterns for UMD.
3. **Avoid global pollution.** Prefer module declaration files over global `.d.ts` files when possible.
4. **Use `declare global` only in modules.** It requires at least one import/export in the file.
5. **Never publish global augmentations from libraries.** Global augmentations can cause conflicts.
6. **Keep declarations type-only.** Do not put implementation code in `.d.ts` files.
7. **Validate with the compiler.** Run `tsc --noEmit` to ensure declarations are correct.
8. **Ship declaration files with libraries.** Set `declaration: true` and `declarationMap: true`.
9. **Use callback signatures accurately.** Do not mark callback arguments as optional if they are always passed.
10. **Prefer function overloads over union types** for function and constructor declarations.

**Component breakdown (declaration form decision tree):**

```
What entity are you declaring?
  ├── Variable → declare const / var / let.
  ├── Function → declare function.
  ├── Class → declare class.
  ├── Namespaced global → declare namespace.
  ├── External module → declare module "name".
  └── Global augmentation → declare global (inside a module).
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

**Why this output occurs.** The top-level `export` makes the file a module declaration file, avoiding global pollution.

---

#### Example 2: Safe global augmentation

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

**Why this output occurs.** The `export {}` makes the file a module, and `declare global` augments the global `Window` interface safely.

---

### Real-World Cases

- **Publishing types** with `types`/`typings` in package.json.
- **DefinitelyTyped** contributions.
- **Monorepo shared types** in a `types/` folder.
- **Third-party module augmentation** (e.g., `express-session`).

---

## References

- TypeScript Handbook — Declaration Files: Introduction - https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html
- TypeScript Handbook — Declaration Reference (`declare namespace`, `declare function`, `declare class`, `declare const`, `declare var`) - https://www.typescriptlang.org/docs/handbook/declaration-files/by-example.html
- TypeScript Handbook — Deep Dive (`declare module`, `declare namespace`) - https://www.typescriptlang.org/docs/handbook/declaration-files/deep-dive.html
- TypeScript Handbook — Global: Modifying Module (`declare global`) - https://www.typescriptlang.org/docs/handbook/declaration-files/templates/global-modifying-module-d-ts.html
- TypeScript Handbook — Library Structures - https://www.typescriptlang.org/docs/handbook/declaration-files/library-structures.html
- TypeScript Handbook — Do’s and Don’ts - https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html
- DefinitelyTyped — Best Practices - https://definitelytyped.org/guides/best-practices.html
- TypeScript Language Specification — Ambient Declarations - https://raw.githubusercontent.com/microsoft/TypeScript/refs/tags/v2.3.3/doc/spec.md#1.1
- Convex — Declare: TypeScript Guide (`declare function`) - https://www.convex.dev/typescript/core-concepts/declare
- Total TypeScript — Ambient Context and the `declare` Keyword - https://www.totaltypescript.com/workshops/typescript-pro-essentials/modules-scripts-and-declaration-files/type-variables-declared-elsewhere/solution
- Total TypeScript — Global Declarations - https://www.totaltypescript.com/workshops/typescript-pro-essentials/modules-scripts-and-declaration-files/global-scope-declarations-in-typescript/solution
- Steve Kinney — Publishing Types for Component Libraries - https://stevekinney.com/courses/react-typescript/publishing-types-for-component-libraries