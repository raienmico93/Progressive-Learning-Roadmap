# Module Resolution in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Module resolution** is the compiler phase that determines **which file an import specifier refers to** — the process of “taking a string from the `import` or `require` statement, and determining what file that string refers to”. It is separate from type checking and happens during program creation. TypeScript supports multiple resolution strategies, selected via the `moduleResolution` compiler option, which must align with the project’s **runtime environment** (Node.js vs. bundler) and **module format** (ESM vs. CommonJS).

**Technical Definition.** TypeScript’s resolver is implemented in `moduleNameResolver.ts` and exposes a `ModuleResolutionKind` enum: `Classic = 1`, `Node10 = 2` (alias for `NodeJs`), `Node16 = 3`, `NodeNext = 99`, and `Bundler = 100`. Each strategy defines lookup rules for **relative** imports (`./`, `../`) and **non-relative** imports (bare package specifiers and path-mapped aliases). Modern strategies (`node16`, `nodenext`, `bundler`) support `package.json` `"exports"` and `"imports"` fields, while `node10` and `classic` do not. The `baseUrl` and `paths` options add a **pre-resolution rewrite layer** for non-relative specifiers.

**Beginner-Friendly Explanation.** Module resolution is **how TypeScript finds the file you are importing**. When you write `import { helper } from "./utils"`, TypeScript has to figure out whether that means `utils.ts`, `utils/index.ts`, or something else. When you write `import express from "express"`, it has to search `node_modules`. The `moduleResolution` setting tells TypeScript **which set of rules to follow** while searching — rules that must match how your code will actually run.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Separate from type checking** | Resolution happens during program creation; it determines which files enter the program. |
| **Relative vs. non-relative** | Relative imports start with `/`, `./`, or `../`; non-relative imports are bare specifiers or path-mapped aliases. |
| **Strategy-dependent** | Different `moduleResolution` values apply different lookup rules and feature support. |
| **Runtime alignment** | The chosen strategy must match the runtime (Node.js ESM/CJS or a bundler). |
| **Extension sensitivity** | `node16`/`nodenext` require explicit file extensions in ESM relative imports; `bundler` does not. |
| **Path mapping is a rewrite layer** | `baseUrl` and `paths` rewrite non-relative specifiers before resolution; runtime tools must agree. |
| **Modern strategies support `exports`** | `node16`, `nodenext`, and `bundler` respect `package.json` `"exports"` and `"imports"`. |

---

### Prerequisites

- **TypeScript modules**: `import`/`export` syntax and module detection.
- **`tsconfig.json`**: familiarity with `compilerOptions`.
- **Node.js module systems**: ESM vs. CommonJS, `package.json` fields (`"type"`, `"main"`, `"exports"`).
- **Bundlers**: Vite, webpack, esbuild, or Rollup basics.
- **File paths**: relative vs. absolute specifiers.

---

### Related Programming Areas

- **ES Modules vs. CommonJS** — the runtime module systems that resolution must match.
- **`package.json` `"exports"` and `"imports"`** — modern package entry-point and subpath mapping.
- **Bundler resolution** — Vite, webpack, esbuild, and Rollup use their own resolvers that must align with TypeScript’s.
- **Path aliases** — `baseUrl` and `paths` for cleaner imports.
- **Monorepos** — workspace resolution and project references.

---

## Core Concepts / Features

---

## 1. Module Resolution

### Definitions

**Core Definition.** **Module resolution** is the algorithm that maps an import specifier to a concrete file on disk.

**Technical Definition.** TypeScript applies the selected `ModuleResolutionKind` to each import. For **relative imports**, the resolver starts from the importing file’s directory. For **non-relative imports**, it searches `node_modules` (walking up the directory tree) and applies `baseUrl`/`paths` rewrites. Modern strategies add `package.json` `"exports"` condition matching (`types`, `import`, `require`) and enforce extension rules.

**Beginner-Friendly Explanation.** Module resolution is **the search process** that turns `"./utils"` or `"express"` into an actual `.ts` or `.d.ts` file.

### Purposes

- **To locate the correct source or declaration file** for every import.
- **To enforce runtime-consistent rules** so code runs where it is deployed.
- **To support package entry points** through `package.json` `"exports"`.
- **To enable path aliases** for cleaner imports.

### Syntax Rules and Structure

**Configuration:**

```json
{
  "compilerOptions": {
    "moduleResolution": "bundler" | "nodenext" | "node16" | "node10" | "classic"
  }
}
```

**Syntax rules.**

- `moduleResolution` is set in `tsconfig.json` under `compilerOptions`.
- If omitted, defaults depend on `module`: `node10` for `commonjs`, `classic` otherwise (legacy behavior).
- Modern projects should **explicitly set** both `module` and `moduleResolution`.

**Constraints and limitations.**

- Mismatched `module` and `moduleResolution` causes “Cannot find module” errors.
- `classic` is legacy and rarely appropriate for modern projects.

### Annotated Complete Step by Step Code Examples

#### Example 1: Bundler-style configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "module": "esnext",
    "moduleResolution": "bundler",
    "strict": true,
    "noEmit": true
  }
}
```

```typescript
// src/main.ts
import { helper } from "./utils"; // Extension omitted — bundler allows this.
console.log(helper());
```

**Expected behavior.** The bundler strategy resolves `./utils` to `src/utils.ts` without requiring a `.js` extension.

---

### Real-World Cases

- **Vite/React apps**: `module: "esnext"`, `moduleResolution: "bundler"`.
- **Node.js libraries**: `module: "nodenext"`, `moduleResolution: "nodenext"`.
- **Legacy CommonJS projects**: `module: "commonjs"`, `moduleResolution: "node10"`.

---

## 2. Relative Imports

### Definitions

**Core Definition.** A **relative import** is one whose specifier starts with `/`, `./`, or `../`, and is resolved **relative to the importing file’s location**.

**Technical Definition.** Relative imports never resolve to ambient module declarations or `node_modules` packages. The resolver joins the specifier to the importing file’s directory and searches for matching files (`.ts`, `.tsx`, `.d.ts`, `index.*`). Under `node16`/`nodenext` with ESM, relative imports **must include file extensions** (`.js`).

**Beginner-Friendly Explanation.** Relative imports are **file paths** — “the file next to me” (`./`) or “the file one folder up” (`../`).

### Purposes

- **To import sibling or nearby files** within the same project.
- **To avoid coupling to absolute project structure**.
- **To express local dependencies clearly**.

### Syntax Rules and Structure

```typescript
import { x } from "./module";      // Same directory
import { y } from "../parent/mod"; // Parent directory
import { z } from "/absolute";     // Root-relative (rare)
```

**Syntax rules.**

- Relative imports **cannot** resolve to `node_modules` packages.
- Under `node16`/`nodenext` ESM, add `.js` extensions to relative imports.
- Under `bundler`, extensions are optional.

**Constraints and limitations.**

- Deep relative paths (`../../../../`) become fragile and hard to read.
- Moving files requires updating relative paths.

### Annotated Complete Step by Step Code Examples

#### Example 1: Relative import with `nodenext`

```typescript
// src/math.ts
export function add(a: number, b: number): number { 
  return a + b; 
}
```

```typescript
// src/main.ts
import { add } from "./math.js"; // .js extension required under nodenext ESM.

console.log(add(2, 3)); // Expected: 5
```

**Expected Output:**
```
5
```

**Why this output occurs.** Under `nodenext`, TypeScript enforces Node ESM’s rule that relative specifiers include the **emitted** `.js` extension.

---

### Real-World Cases

- **Feature folders**: importing sibling components and utilities.
- **Tests**: importing the module under test.

---

## 3. Package Imports

### Definitions

**Core Definition.** A **package import** (non-relative import) is a bare specifier that resolves to a package in `node_modules` (or to a `baseUrl`/`paths` target). Examples: `"express"`, `"lodash/fp"`, `"@scope/pkg"`.

**Technical Definition.** Non-relative imports are resolved relative to `baseUrl`, through path mapping, or by searching `node_modules` upward from the importing file. Modern strategies match `package.json` `"exports"` conditions (`"types"`, `"import"`, `"require"`) to select the correct declaration and runtime files. `node10` ignores `"exports"` and uses `"types"`/`"main"` fields.

**Beginner-Friendly Explanation.** Package imports are **imports from installed libraries** — you write the package name, and TypeScript finds it in `node_modules`.

### Purposes

- **To consume third-party libraries** and workspace packages.
- **To use package subpath exports** (`"pkg/subpath"`).
- **To support package-internal aliases** via `"imports"` (`#/utils`).

### Syntax Rules and Structure

```typescript
import express from "express";
import { fp } from "lodash/fp";
import * as utils from "#/utils.js"; // package "imports" subpath
```

**Syntax rules.**

- Bare specifiers do not start with `./` or `../`.
- Subpath imports (`pkg/sub`) require `"exports"` support in the package.
- `#/` imports require `package.json` `"imports"` and `node16`/`nodenext`/`bundler`.

**Constraints and limitations.**

- `node10` does not support `"exports"` subpaths or `#/` imports.
- pnpm’s strict `node_modules` blocks phantom dependencies at resolution.

### Annotated Complete Step by Step Code Examples

#### Example 1: Package subpath import

```typescript
// package.json (dependency)
{
  "name": "mylib",
  "exports": {
    ".": { "types": "./dist/index.d.ts", "import": "./dist/index.js" },
    "./utils": { "types": "./dist/utils.d.ts", "import": "./dist/utils.js" }
  }
}
```

```typescript
// main.ts
import { helper } from "mylib/utils";
console.log(helper()); // Expected: "helper result"
```

**Expected Output:**
```
helper result
```

**Why this output occurs.** `node16`/`nodenext` matches the `"./utils"` export condition and selects the `types` declaration for checking and the `import` file for runtime.

---

### Real-World Cases

- **Importing scoped packages**: `@company/ui`.
- **Subpath exports**: `date-fns/fp`, `lodash/fp`.
- **Package-internal aliases**: `#/config` within a Node package.

---

## 4. Path Aliases

### Definitions

**Core Definition.** A **path alias** is a custom non-relative specifier (e.g., `@/components/Button`) that is rewritten by TypeScript to a real path using `baseUrl` and `paths`. It creates **shortcuts** for commonly imported directories.

**Technical Definition.** `paths` defines a mapping from alias patterns (which may include `*`) to arrays of target paths, resolved relative to `baseUrl`. TypeScript rewrites the specifier **before** module resolution. The **runtime** (bundler or Node loader) must be configured with the same aliases, or the aliases will compile but fail at runtime.

**Beginner-Friendly Explanation.** Path aliases let you write **`@/components/Button`** instead of **`../../../../components/Button`**. They are bookmarks for your project’s folders.

### Purposes

- **To eliminate fragile deep relative paths** (`../../../`).
- **To make imports refactor-safe** and easier to read.
- **To create a stable project-wide import convention** (`@/`).
- **To align TypeScript with bundler aliases** for a single source of truth.

### Syntax Rules and Structure

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"]
    }
  }
}
```

```typescript
import { Button } from "@/components/Button";
```

**Syntax rules.**

- `paths` is relative to `baseUrl`.
- `*` captures the matched segment and substitutes it into the target.
- Each alias may map to an **array** of fallback targets.
- Bundlers need matching `resolve.alias` configuration.

**Constraints and limitations.**

- Aliases are a **TypeScript-side rewrite**; runtime must be configured separately.
- Mixing `@/` and `~/` conventions confuses teams.
- Aliases do not change emitted import paths unless a bundler or loader handles them.

### Annotated Complete Step by Step Code Examples

#### Example 1: TypeScript and Vite alias alignment

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import path from "path";

export default defineConfig({
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

```typescript
// src/main.ts
import { Button } from "@/components/Button";
console.log(Button); // Expected: [Function: Button]
```

**Expected Output:**
```
[Function: Button]
```

**Why this output occurs.** TypeScript rewrites `@/components/Button` to `src/components/Button` for type checking. Vite applies the same alias at bundle time, so the import resolves consistently.

---

### Real-World Cases

- **React/Vue projects**: `@/` for `src/`.
- **Monorepos**: `@workspace/ui` for shared packages.
- **Next.js**: `~/` for project root.

---

## 5. `moduleResolution`

### Definitions

**Core Definition.** **`moduleResolution`** is the `tsconfig.json` compiler option that selects the **module resolution strategy** — the set of rules TypeScript uses to locate imported files.

**Technical Definition.** The option accepts `"classic"`, `"node10"` (alias `"node"`), `"node16"`, `"nodenext"`, or `"bundler"`. It must be paired with a compatible `module` setting. The strategy determines extension rules, `package.json` field support, `exports`/`imports` handling, and whether file extensions are required.

**Beginner-Friendly Explanation.** `moduleResolution` tells TypeScript **which rulebook to use** when searching for imports. Different rulebooks match different runtimes.

### Purposes

- **To match the runtime’s actual resolution behavior** (Node.js or bundler).
- **To enable modern package features** (`exports`, `imports`).
- **To avoid resolution mismatches** that cause “Cannot find module” errors.

### Syntax Rules and Structure

```json
{
  "compilerOptions": {
    "moduleResolution": "bundler"
  }
}
```

**Syntax rules.**

- Values: `"classic"`, `"node10"` / `"node"`, `"node16"`, `"nodenext"`, `"bundler"`.
- `"node"` is an alias for `"node10"`.
- Defaults are legacy: `node10` for `commonjs`, `classic` otherwise.
- Modern projects should set it explicitly.

**Constraints and limitations.**

- `classic` and `node10` do not support `"exports"` maps.
- Mismatched `module`/`moduleResolution` causes errors.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit modern pairing

```json
{
  "compilerOptions": {
    "module": "nodenext",
    "moduleResolution": "nodenext"
  }
}
```

**Expected behavior.** TypeScript applies Node.js ESM/CJS rules, including `"exports"` support and extension requirements.

---

### Real-World Cases

- **Node services**: `nodenext`.
- **Bundled frontends**: `bundler`.
- **Legacy libraries**: `node10`.

---

## 6. `node` / `node10`

### Definitions

**Core Definition.** **`node10`** (historically `"node"`, also called `NodeJs`) is a resolution strategy that **mimics Node.js CommonJS resolution** — searching `node_modules`, supporting directory `index` files and `package.json` `"types"`/`"main"` fields.

**Technical Definition.** `Node10 = 2` in `ModuleResolutionKind`; `"node"` is an alias. It does **not** support `package.json` `"exports"` maps or extension-explicit ESM rules. It is described as “most common for older projects”.

**Beginner-Friendly Explanation.** `node10` is **the old Node.js rulebook**. It works for CommonJS projects but does not understand modern package `exports`.

### Purposes

- **To support legacy CommonJS projects**.
- **To resolve older packages** that rely on `"main"` and `"types"`.
- **To maintain compatibility** during migration.

### Syntax Rules and Structure

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "moduleResolution": "node10"
  }
}
```

**Constraints and limitations.**

- **Ignores `"exports"` maps**; subpath imports may fail.
- Not recommended for new projects.
- Being phased out in favor of modern modes.

### Annotated Complete Step by Step Code Examples

#### Example 1: Legacy resolution

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "moduleResolution": "node10"
  }
}
```

**Expected behavior.** Resolves `import x from "pkg"` via `node_modules/pkg/package.json` `"types"`/`"main"`.

---

### Real-World Cases

- **Legacy Node.js APIs** written in CommonJS.
- **Older npm packages** without `"exports"`.

---

## 7. `node16`

### Definitions

**Core Definition.** **`node16`** is a resolution strategy that supports **both ESM and CommonJS** with `package.json` `"exports"` conditions, following Node.js 16+ semantics.

**Technical Definition.** `Node16 = 3`. It respects `"exports"` maps with `"types"`, `"import"`, and `"require"` conditions, and enforces ESM rules (including extension requirements) based on the package’s module type.

**Beginner-Friendly Explanation.** `node16` is **the modern Node.js rulebook** that understands `exports` and ESM.

### Purposes

- **To support dual ESM/CJS packages**.
- **To respect `package.json` `exports` conditions**.
- **To align with Node.js 16+ runtime behavior**.

### Syntax Rules and Structure

```json
{
  "compilerOptions": {
    "module": "node16",
    "moduleResolution": "node16"
  }
}
```

**Constraints and limitations.**

- Requires explicit extensions in ESM relative imports.
- Stricter than `bundler`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Node16 configuration

```json
{
  "compilerOptions": {
    "module": "node16",
    "moduleResolution": "node16"
  }
}
```

**Expected behavior.** ESM imports require `.js` extensions; `exports` conditions are matched.

---

### Real-World Cases

- **Node.js 16+ libraries** with conditional exports.

---

## 8. `nodenext`

### Definitions

**Core Definition.** **`nodenext`** is the **latest Node.js resolution strategy**, tracking Node.js’s evolving module rules and supporting `"exports"`/`"imports"` with ESM/CJS semantics.

**Technical Definition.** `NodeNext = 99`. It is the strictest mode: relative ESM imports **must include `.js` extensions**; `"exports"` and `"imports"` are fully supported; behavior follows the package’s `"type"` field. TypeScript “never wants to change runtime behavior,” so it enforces the extension you must write for Node ESM.

**Beginner-Friendly Explanation.** `nodenext` is **the “run it directly on Node.js” rulebook**. It is strict and accurate for Node ESM.

### Purposes

- **To run TypeScript output directly on Node.js** without a bundler.
- **To enforce Node.js ESM/CJS correctness**.
- **To use `"exports"` and `#/` imports** natively.

### Syntax Rules and Structure

```json
{
  "compilerOptions": {
    "module": "nodenext",
    "moduleResolution": "nodenext"
  }
}
```

```typescript
import { helper } from "./utils.js"; // .js required
```

**Constraints and limitations.**

- Requires explicit `.js` extensions on relative ESM imports.
- Not suitable for bundled apps that omit extensions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Extension requirement

```typescript
// ✅ Correct under nodenext
import { add } from "./math.js";

// ❌ Error under nodenext
import { add } from "./math";
```

**Expected behavior.** The extensionless import triggers error TS2835, suggesting `math.js`.

---

### Real-World Cases

- **Node.js ESM packages** (`"type": "module"`).
- **CLI tools** run with `node`.

---

## 9. `bundler`

### Definitions

**Core Definition.** **`bundler`** is a resolution strategy designed for **bundled applications** (Vite, webpack, esbuild, Rollup). It supports `"exports"` and `"imports"` but **does not require file extensions** on relative imports.

**Technical Definition.** `Bundler = 100`. It is “optimized for bundlers” and uses “relaxed rules” compared to Node.js modes. It supports `package.json` `"imports"` (`#/`) and `"exports"` like `node16`/`nodenext`, but never requires extensions.

**Beginner-Friendly Explanation.** `bundler` is **the “my bundler handles paths” rulebook**. It is relaxed and convenient for frontend projects.

### Purposes

- **To match bundler resolution behavior**.
- **To allow extensionless imports** in bundled code.
- **To support modern package `exports`** without Node’s strictness.

### Syntax Rules and Structure

```json
{
  "compilerOptions": {
    "module": "esnext",
    "moduleResolution": "bundler"
  }
}
```

**Constraints and limitations.**

- Does not enforce Node.js ESM extension rules.
- Runtime aliases must be configured in the bundler.

### Annotated Complete Step by Step Code Examples

#### Example 1: Extensionless import under bundler

```typescript
import { helper } from "./utils"; // ✅ Allowed under bundler.
```

**Expected behavior.** The resolver finds `utils.ts` without requiring `.js`.

---

### Real-World Cases

- **Vite + React/Vue** projects.
- **webpack/esbuild** applications.
- **Monorepo apps** using bundlers.

---

## 10. `classic`

### Definitions

**Core Definition.** **`classic`** is TypeScript’s **legacy resolution strategy**, included for backward compatibility and rarely used in modern projects.

**Technical Definition.** `Classic = 1`. It was the default when `module` was not `commonjs`. It does not support `node_modules` resolution in the same way and lacks modern `"exports"` support. It is effectively obsolete for new code.

**Beginner-Friendly Explanation.** `classic` is **the old, outdated rulebook**. Do not use it for new projects.

### Purposes

- **To support very old TypeScript projects**.
- **To provide historical compatibility**.

### Syntax Rules and Structure

```json
{
  "compilerOptions": {
    "moduleResolution": "classic"
  }
}
```

**Constraints and limitations.**

- Legacy and rarely appropriate.
- Removed or erroring in future TypeScript versions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Classic resolution

```json
{
  "compilerOptions": {
    "moduleResolution": "classic"
  }
}
```

**Expected behavior.** Applies legacy lookup rules; not recommended.

---

### Real-World Cases

- **Ancient codebases** only.

---

## 11. Path Mapping (`baseUrl` and `paths`)

### Definitions

**Core Definition.** **Path mapping** is the combination of `baseUrl` and `paths` that rewrites **non-relative** specifiers to real files before resolution.

**Technical Definition.** `baseUrl` sets the base directory for non-relative imports. `paths` defines alias patterns mapped to target arrays relative to `baseUrl`. This rewrite happens **before** the selected resolution strategy runs. TypeScript uses the mapping for type checking; the runtime needs its own alias configuration.

**Beginner-Friendly Explanation.** Path mapping lets you **create short names for folders** so imports are clean and stable.

### Purposes

- **To shorten deep relative paths**.
- **To create a consistent import convention**.
- **To align TypeScript with bundler aliases**.

### Syntax Rules and Structure

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    }
  }
}
```

**Constraints and limitations.**

- Aliases must be mirrored in the bundler or runtime.
- `paths` is relative to `baseUrl`.

### Annotated Complete Step by Step Code Examples

#### Example 1: `baseUrl` + `paths`

```json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@utils/*": ["utils/*"],
      "@components/*": ["components/*"]
    }
  }
}
```

```typescript
import { Button } from "@components/Button";
```

**Expected behavior.** Resolves to `src/components/Button.ts`.

---

### Real-World Cases

- **`@/` alias for `src/`**.
- **`~` alias for project root**.
- **Package-internal `#/` imports** for Node-native aliasing.

---

## 12. `baseUrl`

### Definitions

**Core Definition.** **`baseUrl`** sets the **base directory** for resolving non-relative imports and for evaluating `paths` mappings.

**Technical Definition.** When `baseUrl` is set, bare specifiers that are not found in `node_modules` are resolved relative to it. `paths` entries are interpreted relative to `baseUrl`.

**Beginner-Friendly Explanation.** `baseUrl` is **the folder TypeScript starts from** when resolving non-relative imports.

### Purposes

- **To define the root for path aliases**.
- **To resolve non-relative imports** from a known directory.
- **To simplify project structure**.

### Syntax Rules and Structure

```json
{
  "compilerOptions": {
    "baseUrl": "./src"
  }
}
```

**Constraints and limitations.**

- Changing `baseUrl` shifts alias resolution.
- Not needed if `paths` is not used, but often paired.

### Annotated Complete Step by Step Code Examples

#### Example 1: Base URL resolution

```json
{
  "compilerOptions": {
    "baseUrl": "./src"
  }
}
```

```typescript
import { config } from "config"; // Resolves to src/config
```

**Expected behavior.** The bare specifier `"config"` resolves to `src/config.ts`.

---

### Real-World Cases

- **Monorepos** with a shared `src` root.
- **Path alias configurations**.

---

## 13. `paths`

### Definitions

**Core Definition.** **`paths`** defines alias patterns that map non-relative specifiers to target paths, resolved relative to `baseUrl`.

**Technical Definition.** Each key is a pattern (may contain `*`), and each value is an array of targets. The `*` wildcard captures the matched substring and substitutes it into the target. This rewrite is TypeScript-side only.

**Beginner-Friendly Explanation.** `paths` is **the alias table** — it says “if you see `@/foo`, look in `src/foo`.”

### Purposes

- **To create readable aliases**.
- **To support wildcard mappings**.
- **To provide fallback targets**.

### Syntax Rules and Structure

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@lib": ["src/lib/index.ts"]
    }
  }
}
```

**Constraints and limitations.**

- Must be mirrored in the runtime tool.
- Wildcard patterns must be consistent.

### Annotated Complete Step by Step Code Examples

#### Example 1: Wildcard alias

```json
{
  "paths": {
    "@/*": ["src/*"]
  }
}
```

```typescript
import { Button } from "@/components/Button";
```

**Expected behavior.** `@/components/Button` → `src/components/Button`.

---

### Real-World Cases

- **`@/` → `src/`**.
- **`@components/` → `src/components/`**.
- **`#/` package imports** (Node-native alternative).

---

## 14. Module Resolution Best Practices

### Definitions

**Core Definition.** **Module resolution best practices** are guidelines for selecting and configuring resolution strategies so that **TypeScript, the bundler, and the runtime all agree**, avoiding “Cannot find module” errors and runtime failures.

**Technical Definition.** The dominant decision is **bundled app vs. Node.js runtime**. For bundled apps, use `module: "esnext"` with `moduleResolution: "bundler"`. For Node.js packages, use `module: "nodenext"` with `moduleResolution: "nodenext"`. Use `paths` for project-internal aliases and `"imports"` (`#/`) for package-native aliases; do not mix casually. Always mirror TypeScript aliases in the bundler or runtime configuration.

**Beginner-Friendly Explanation.** Best practices are **the habits that keep imports working everywhere**: pick the rulebook that matches where your code runs, configure aliases in both TypeScript and your build tool, and avoid mixing strategies.

### Purposes

- **To eliminate resolution mismatches** between compile time and runtime.
- **To choose the correct strategy** for the deployment environment.
- **To keep aliases consistent** across TypeScript and tooling.
- **To avoid deprecated strategies** (`classic`, `node10`) in new projects.

### Syntax Rules and Structure

**Decision tree:**

```
Where does the code run?
  ├── Bundler (Vite, webpack, esbuild, Rollup)
  │     → module: "esnext", moduleResolution: "bundler"
  └── Node.js directly
        → module: "nodenext", moduleResolution: "nodenext"
              (add .js extensions to relative imports)
```

**Rules.**

1. **Match the strategy to the runtime** — bundler for bundled apps, `nodenext` for Node.
2. **Set both `module` and `moduleResolution`** explicitly.
3. **Use `paths` for project aliases**, and mirror them in the bundler.
4. **Use `#/` package imports** for Node-native internal aliases when appropriate.
5. **Avoid `classic` and `node10`** for new code.
6. **Do not mix alias strategies casually** — keep runtime and compiler aligned.

**Constraints and limitations.**

- Extensions are required under `nodenext` but not `bundler`.
- Aliases that compile may still fail at runtime if the runtime is not configured.

### Annotated Complete Step by Step Code Examples

#### Example 1: Bundler project configuration

```json
{
  "compilerOptions": {
    "module": "esnext",
    "moduleResolution": "bundler",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "strict": true,
    "noEmit": true
  }
}
```

```typescript
import { Button } from "@/components/Button";
```

**Expected behavior.** TypeScript resolves the alias; the bundler mirrors it; no extension is needed.

---

#### Example 2: Node.js project configuration

```json
{
  "compilerOptions": {
    "module": "nodenext",
    "moduleResolution": "nodenext",
    "strict": true
  }
}
```

```typescript
import { add } from "./math.js"; // Extension required
```

**Expected behavior.** Node.js ESM rules are enforced; `"exports"` conditions are respected.

---

### Real-World Cases

- **Vite React app**: `bundler` + `@/` alias.
- **Node.js library**: `nodenext` + `.js` extensions.
- **Monorepo**: `bundler` for apps, `nodenext` for packages, with shared `paths` where appropriate.

---

## References

- TypeScript Documentation — Module Resolution - https://www.typescriptlang.org/docs/handbook/module-resolution.html
- TypeScript Documentation — Modules: Choosing Compiler Options - https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html
- TypeScript Documentation — Modules: Reference (`moduleResolution` values) - https://www.typescriptlang.org/docs/handbook/modules/reference.html
- TypeScript Documentation — `moduleResolution` TSConfig Reference - https://www.typescriptlang.org/tsconfig/moduleResolution
- TypeScript Documentation — `baseUrl` TSConfig Reference - https://www.typescriptlang.org/tsconfig/baseUrl
- TypeScript Documentation — `paths` TSConfig Reference - https://www.typescriptlang.org/tsconfig/paths
- Total TypeScript — The `moduleResolution` Option in `tsconfig.json` - https://www.totaltypescript.com/workshops/typescript-pro-essentials/configuring-typescript/the-moduleresolution-option-in-tsconfigjson/exercise
- Steve Kinney — Module Resolution and Path Aliases - https://stevekinney.com/courses/react-typescript/module-resolution-and-paths
- Guance — TypeScript 模块解析（moduleResolution）详解 - https://www.guance.com/learn/articles/typescript-module-resolution
- Mintlify Wiki — Module Resolution (TypeScript Compiler) - https://mintlify.wiki/Microsoft/typescript/compiler/module-resolution
- nodnarbnitram/claude-code-extensions — TypeScript 6 Module Resolution and Imports Reference - https://github.com/nodnarbnitram/claude-code-extensions/blob/main/.claude/skills/typescript-v6/references/module-resolution-imports-reference.md
- Nx Blog — Everything You Need to Know About TypeScript Project References - https://nx.dev/blog/typescript-project-references