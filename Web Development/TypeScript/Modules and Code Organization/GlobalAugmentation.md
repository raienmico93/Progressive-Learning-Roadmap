# Global Augmentation in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Global augmentation** is a TypeScript mechanism that allows a **module** to **patch or extend declarations in the global scope** using the `declare global { }` syntax. It is the global-scope counterpart to module augmentation: where module augmentation re-opens an existing module, global augmentation re-opens existing global declarations such as `Window`, `Array`, `String`, `Console`, or `globalThis`. The feature exists to type runtime values that are injected into the global environment by scripts, polyfills, or platform runtimes, without modifying the original global declaration files.

**Technical Definition.** A global augmentation is a `declare global { ... }` block that must appear inside a **module**—a file with at least one top-level `import` or `export`. The declarations inside the block are merged into the global namespace using **declaration merging**. Global augmentations **cannot add new items to the top-level scope**; they can only “patch” existing declarations. The augmentation is **type-only**: it is erased during compilation and produces no JavaScript, meaning runtime behavior must be implemented separately. When the containing file is a **script** (no imports or exports), the `declare global` block is unnecessary because top-level declarations in a global `.d.ts` file are already global.

**Beginner-Friendly Explanation.** Global augmentation is **how you teach TypeScript about extra things that exist globally at runtime**—like a property your analytics script adds to `window`, or a method your polyfill adds to `Array.prototype`. You write a special block in your code that says “add this to the global types,” and TypeScript then recognizes it **everywhere in your project**. It is a powerful escape hatch, but because it affects the entire program, it must be used **sparingly and carefully**.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Module-scoped syntax** | `declare global` works only inside a module (a file with at least one `import` or `export`). |
| **Patch-only** | Can only extend **existing** global declarations; new top-level names cannot be introduced. |
| **Declaration merging** | Relies on interface and namespace merging to combine new members with existing global types. |
| **Type-only** | Produces **no JavaScript**; it does not change runtime behavior. |
| **Global scope impact** | Affects the **entire program**, including all files and packages, making collisions possible. |
| **`globalThis` support** | Can augment `globalThis` for non-browser globals in Node.js and other runtimes. |
| **Non-tree-shakeable** | Global augmentations are not removed by tree-shaking; they remain in the type graph. |

---

### Prerequisites

- **TypeScript modules**: `import`/`export` syntax and module detection.
- **Declaration merging**: how interfaces and namespaces combine.
- **Declaration files** (`.d.ts`) and ambient declarations.
- **Platform globals**: `Window`, `Array`, `String`, `Console`, `ProcessEnv`, and `globalThis`.
- **`tsconfig.json`**: `include` and `typeRoots` configuration.

---

### Related Programming Areas

- **Module augmentation** (`declare module`) — the module-scoped counterpart.
- **Declaration merging** — the underlying mechanism that makes augmentation work.
- **DefinitelyTyped and `@types`** — type packages that may declare the globals you want to augment.
- **Global `.d.ts` files** — the alternative when the declaration file is not a module.
- **Runtime patching** — the value-level counterpart to type-level augmentation.

---

## Core Concepts / Features

---

## 1. Global Augmentation

### Definitions

**Core Definition.** **Global augmentation** is the act of using a `declare global { ... }` block inside a module to **merge new declarations into the global scope**.

**Technical Definition.** A global augmentation patches existing global declarations—interfaces like `Window`, `Array`, `String`, and `Console`, or variables on `globalThis`—using declaration merging. It cannot add new top-level names; it can only extend what already exists. The containing file **must be a module**; otherwise, the block is either unnecessary or interpreted as a global declaration file. The augmentation is erased at compile time and has **no runtime effect**.

**Beginner-Friendly Explanation.** Global augmentation is **adding type information to things that are already globally available**. For example, if a script adds `window.analytics`, you augment `Window` to include `analytics` so TypeScript stops complaining and gives you autocomplete.

### Purposes

- **To add types for global variables injected by third-party scripts** such as `window.analytics`, `window.dataLayer`, or `window.posthog`.
- **To type polyfills that extend built-in prototypes**, such as `Array.prototype.mapToNumbers()` or `String.prototype.fancyFormat()`.
- **To augment platform globals** like `Window`, `Console`, `ProcessEnv`, or `globalThis` with application-specific properties.
- **To integrate untyped global libraries** into a TypeScript project without importing them.
- **To provide a single, centralized declaration** for global concerns in an application, improving discoverability and maintainability.

### Syntax Rules and Structure

**General syntax (global augmentation inside a module):**

```typescript
export {}; // Ensure this file is treated as a module.

declare global {
  interface Window {
    myGlobal: string;
  }

  var myGlobalVariable: number;
}
```

**Component breakdown:**

- `export {}` — a module marker that ensures the file is treated as a module; `declare global` requires module context.
- `declare global { ... }` — opens the global augmentation block.
- `interface Window { ... }` — merges new properties into the existing global `Window` interface.
- `var myGlobalVariable: number` — declares a global variable that appears on `globalThis`; `var` is required for `globalThis` properties.

**Syntax rules.**

- The containing file **must be a module** (top-level `import` or `export`). A bare `declare global` in a script file is invalid or unnecessary.
- Only **existing global declarations** can be patched; new top-level names cannot be introduced.
- Use `interface` for augmenting types and `var` for global variables.
- In a **global `.d.ts` file** (no imports/exports), you can declare `interface Window { ... }` directly without `declare global`.

**Constraints and limitations.**

- **Cannot add new top-level names**; only patch existing declarations.
- **No runtime effect**: the augmentation does not create the property; runtime assignment must exist separately.
- **Global pollution**: affects the entire program and can collide with other packages or browser extensions.
- **`let` and `const` do not add properties to `window`/`globalThis`** in the same way as `var`; use `var` for global variable declarations.

### Annotated Complete Step by Step Code Examples

#### Example 1: Augmenting `Window` with an analytics property

```typescript
// src/types/global.d.ts
export {}; // Step 1: Make this file a module.

declare global {
  // Step 2: Patch the global Window interface.
  interface Window {
    analytics?: {
      track(event: string): void;
      identify(userId: string): void;
    };
  }
}
```

```typescript
// src/app.ts
// Step 3: Use the augmented property with optional chaining.
window.analytics?.track("app_started");
// Expected: event tracked (if analytics is loaded)
```

**Setup.** Include `src/types/global.d.ts` in `tsconfig.json`.

**Expected Output:**
```
event tracked
```

**Why this output occurs.** `export {}` makes the file a module. `declare global` merges `analytics` into the existing `Window` interface via declaration merging. The optional property correctly models a script that may or may not have loaded.

---

#### Example 2: Augmenting `Array` with a polyfill method

```typescript
// src/types/array-polyfill.d.ts
export {};

declare global {
  interface Array<T> {
    mapToNumbers(): number[];
  }
}
```

```typescript
// src/polyfills.ts
// Runtime patch — required separately.
Array.prototype.mapToNumbers = function () {
  return this.map(Number);
};

// Usage
const nums = ["1", "2", "3"].mapToNumbers();
console.log(nums); // Expected: [1, 2, 3]
```

**Expected Output:**
```
[1, 2, 3]
```

**Why this output occurs.** The global augmentation adds `mapToNumbers` to `Array<T>`. The runtime patch in `polyfills.ts` makes the method actually callable. The augmentation is **type-only**; without the runtime patch, the call would fail at runtime.

---

#### Example 3: Augmenting `globalThis` for Node.js

```typescript
// src/types/global.d.ts
export {};

declare global {
  var __cache: Map<string, unknown>;
}
```

```typescript
// src/cache.ts
globalThis.__cache = new Map();
console.log(globalThis.__cache.size); // Expected: 0
```

**Expected Output:**
```
0
```

**Why this output occurs.** `var` declares a property on `globalThis`, making `globalThis.__cache` type-safe. The runtime assignment initializes the cache.

---

### Real-World Cases

- **Analytics scripts**: typing `window.dataLayer`, `window.ga`, `window.posthog`, or `window.analytics`.
- **Polyfills**: typing `Array.prototype.at`, `String.prototype.replaceAll`, or custom prototype extensions in older environments.
- **Node.js globals**: typing `globalThis.__cache`, `global.__rootdir`, or other application singletons.
- **Bundler-injected constants**: typing `__DEV__`, `__VERSION__`, or `process.env` extensions.
- **Test frameworks**: augmenting `jest` matchers or `expect` extensions.

---

## 2. `declare global`

### Definitions

**Core Definition.** `declare global` is the **syntax** used to open a **global augmentation block** inside a module. It tells the compiler that the declarations inside should be merged into the **global scope**.

**Technical Definition.** `declare global { ... }` is only valid inside a **module** (a file with at least one top-level `import` or `export`). It relies on **declaration merging** to patch existing global declarations. In a **global `.d.ts` file** (no imports/exports), top-level declarations are already global, so `declare global` is **unnecessary**. The block is erased at compile time and produces no JavaScript.

**Beginner-Friendly Explanation.** `declare global` is **the container you put global type additions in** when you are writing them inside a module file. It says: “Everything inside this block belongs to the global scope, not to this module.”

### Purposes

- **To add global types from within a module file**, keeping global declarations co-located with related code.
- **To augment existing global interfaces and variables** without creating a separate global `.d.ts` file.
- **To explicitly scope global additions** inside a module context, avoiding accidental global leakage.
- **To support co-location of runtime mutation and type augmentation** in the same file.
- **To enable module-based projects** to extend globals when a global `.d.ts` file is not practical.

### Syntax Rules and Structure

**General syntax:**

```typescript
export {}; // Module marker.

declare global {
  interface ExistingGlobalInterface {
    newMember: Type;
  }

  var newGlobalVariable: Type;
}
```

**Component breakdown:**

- `export {}` — ensures the file is a module; required for `declare global` to work.
- `declare global { ... }` — opens the global augmentation block.
- `interface ExistingGlobalInterface` — patches an existing global interface via declaration merging.
- `var newGlobalVariable: Type` — declares a global variable on `globalThis`.

**Syntax rules.**

- Must be inside a **module** (top-level `import` or `export`).
- Use `export {}` if the file has no other imports/exports.
- Cannot add new top-level names; can only patch existing declarations.
- Use `interface` for type augmentation and `var` for global variables.
- In a `.d.ts` file **without** imports/exports, `declare global` is unnecessary; declare interfaces directly.

**Constraints and limitations.**

- Does **not work in a script file** (no imports/exports).
- Cannot introduce new top-level names.
- Does not emit runtime code; runtime behavior must exist separately.

### Annotated Complete Step by Step Code Examples

#### Example 1: `declare global` in a `.ts` file

```typescript
// src/global-augment.ts
export {}; // Step 1: Make this file a module.

declare global {
  // Step 2: Augment Window.
  interface Window {
    myFeature: {
      enabled: boolean;
    };
  }
}

// Step 3: Runtime assignment (must exist separately).
window.myFeature = { enabled: true };
console.log(window.myFeature.enabled); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** `export {}` makes the file a module, so `declare global` is valid. The block merges `myFeature` into `Window`. The runtime assignment makes the property actually exist.

---

#### Example 2: `declare global` vs. global `.d.ts`

```typescript
// Option A: global .d.ts (no imports/exports)
interface Window {
  myFeature: { enabled: boolean };
}
```

```typescript
// Option B: module with declare global
export {};
declare global {
  interface Window {
    myFeature: { enabled: boolean };
  }
}
```

**Explanation.** Both achieve the same global augmentation. Option A is simpler for a standalone declaration file because top-level declarations are already global. Option B is necessary when the file is a module or when the augmentation is co-located with runtime code.

---

### Real-World Cases

- **Co-locating global augmentation with runtime patching**: keeping the type declaration and the `window.myFeature = ...` assignment in the same file.
- **Augmenting `jest` matchers** in a test setup file.
- **Adding `process.env` properties** in application code.
- **Extending `Console`** with custom logging methods.

---

## 3. Global Augmentation Best Practices

### Definitions

**Core Definition.** **Global augmentation best practices** are guidelines for using global augmentation **safely**, avoiding **global type pollution**, **collisions**, and **maintenance problems** across packages and applications.

**Technical Definition.** The discipline is to **(1) never publish global augmentations from libraries**, **(2) confine them to leaf applications**, **(3) namespace them with a project-specific brand**, **(4) keep them in a clearly named `.d.ts` file**, **(5) use `export {}` to ensure module status**, and **(6) augment `globalThis` for non-browser globals**. Global augmentation is **type-only** and should be aligned with runtime mutation in the same file when possible.

**Beginner-Friendly Explanation.** Best practices mean **keeping global additions rare, clearly labeled, and scoped to your app** so they do not break other packages. The biggest rule is: **libraries should never add to the global scope**—they should export types and let applications decide how to wire them up.

### Purposes

- **To prevent global type collisions** across packages in a monorepo or application.
- **To keep global additions discoverable and maintainable** through predictable file placement.
- **To avoid polluting the global namespace in libraries**, preserving composability.
- **To ensure global augmentations are module-scoped** and do not break tree-shaking.
- **To align type augmentations with runtime patches**, preventing type lies.

### Syntax Rules and Structure

Best practices are conventions that interact with global augmentation syntax. The core rules are:

1. **Libraries never `declare global`.** Export types and let consumers wire them up.
2. **Apps namespace globals under a project-specific key** (e.g., `__acme`, `__internal`, or the team’s short name). Avoid generic names like `analytics`, `auth`, or `flags`—they collide with browser extensions and other libraries.
3. **One file per global concern in a predictable path** such as `src/types/global.d.ts` or `src/types/window.d.ts`. New team members find it instantly.
4. **Use `export {}` to make the file a module**, not a script. Script files implicitly augment the global scope and can break tree-shaking in some bundlers.
5. **Augment `globalThis` for non-browser globals.** `declare global { var __cache: Map<string, unknown> }` works for Node.js globals.
6. **Perform global mutation and type augmentation in the same file** when possible, keeping them synchronized.

**Component breakdown (global augmentation decision tree):**

```
Are you writing a library?
  ├── Yes → Do NOT use `declare global`. Export types instead.
  └── No → Are you writing an application?
           ├── Yes → Namespace under a project-specific key.
           └── No → Reconsider whether global augmentation is necessary.
```

**Constraints and limitations.**

- **Generic global names** (`analytics`, `auth`, `flags`) collide with browser extensions and other libraries.
- **Script files** implicitly augment the global scope and can break tree-shaking.
- **Global augmentations are not tree-shakeable** and affect the entire program.
- **Augmentation alone does not create runtime values**; the runtime patch must exist separately.

### Annotated Complete Step by Step Code Examples

#### Example 1: Library exports, application augments

```typescript
// ❌ Library — do NOT do this.
// declare global {
//   interface Window {
//     analytics: { track(event: string): void };
//   }
// }

// ✅ Library — export types instead.
export interface AnalyticsClient {
  track(event: string): void;
  identify(userId: string): void;
}

export function getAnalytics(): AnalyticsClient {
  // ...
  return null as any;
}
```

```typescript
// ✅ Application — augment with a project-specific namespace.
import type { AnalyticsClient } from "analytics-sdk";

export {};

declare global {
  interface Window {
    __acme: {
      analytics: AnalyticsClient;
    };
  }
}

window.__acme.analytics.track("clicked"); // Expected: event tracked
```

**Expected Output:**
```
event tracked
```

**Why this output occurs.** The library exports `AnalyticsClient` instead of polluting the global scope. The application augments `Window` under `__acme`, avoiding collisions with other libraries or extensions. The `export {}` ensures the file is a module.

---

#### Example 2: One file per global concern

```typescript
// src/types/window.d.ts
export {};

declare global {
  interface Window {
    __appConfig: {
      apiUrl: string;
    };
  }
}
```

```typescript
// src/types/global.d.ts
export {};

declare global {
  var __cache: Map<string, unknown>;
}
```

**Expected behavior.** Each global concern lives in its own clearly named file, making global additions discoverable and maintainable.

---

#### Example 3: Aligning runtime mutation and type augmentation

```typescript
// src/analytics.ts
import type { AnalyticsClient } from "analytics-sdk";

declare global {
  interface Window {
    __acme: {
      analytics: AnalyticsClient;
    };
  }
}

export {};

// Runtime mutation in the same file.
window.__acme = {
  analytics: {
    track(event) {
      console.log(event);
    },
  },
};

window.__acme.analytics.track("app_started"); // Expected: app_started
```

**Expected Output:**
```
app_started
```

**Why this output occurs.** The type augmentation and runtime assignment are co-located, keeping the type and runtime behavior synchronized. The `export {}` ensures module status.

---

### Real-World Cases

- **Monorepo applications**: namespacing globals under `window.__acme.analytics` and `window.__acme.flags` to prevent collisions between packages.
- **Test setup files**: augmenting `jest` matchers in a single `setup.ts` file.
- **Node.js applications**: augmenting `globalThis.__cache` for singleton caches.
- **Bundler-injected constants**: `declare global { var __DEV__: boolean }` for development flags.
- **Code review checklists**: rejecting `declare global` in library code and requiring project-specific namespacing in application code.

---

## References

- TypeScript Handbook — Declaration Merging: Global Augmentation - https://www.typescriptlang.org/docs/handbook/declaration-merging.html#global-augmentation
- TypeScript Documentation — Global: Modifying Module - https://www.typescriptlang.org/docs/handbook/declaration-files/templates/global-modifying-module-d-ts.html
- TypeScript Wiki — What's New in TypeScript: Global Augmentation - https://github.com/microsoft/TypeScript/wiki/What's-new-in-TypeScript
- TypeScript Documentation — Global .d.ts - https://www.typescriptlang.org/docs/handbook/declaration-files/templates/global-d-ts.html
- Total TypeScript — How to Properly Type Window - https://www.totaltypescript.com/how-to-properly-type-window
- pproenca/dot-skills — Scope Global Type Augmentation to Avoid Conflicts - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-advanced-patterns/references/decl-global-augmentation-discipline.md
- Convex — Declare: TypeScript Guide - https://www.convex.dev/typescript/core-concepts/declare
- Stack Overflow — Global augmentation and mutation in the same file - https://stackoverflow.com/revisions/702ba099-51af-4c0a-a9df-40518408bcfe/view-source
- TypeScript Handbook — Declaration Merging: Module Augmentation - https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation
- TypeScript Documentation — Library Structures - https://www.typescriptlang.org/docs/handbook/declaration-files/library-structures.html