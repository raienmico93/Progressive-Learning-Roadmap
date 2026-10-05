# TypeScript Module Resolution: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Module resolution is the process by which TypeScript (and JavaScript runtimes/bundlers) determines which file a given import specifier (e.g., `"./utils"`, `"@myorg/ui"`, `"#lib/foo"`) refers to on disk. It answers the question: "When I write `import { X } from "specifier"`, where does TypeScript look for `X`?"

**Technical Definition**
Module resolution in TypeScript is governed by the `moduleResolution` compiler option, which selects an algorithm for mapping import specifiers to file paths. Each algorithm models a different environment: `"classic"` (TypeScript pre-1.6, deprecated), `"node10"` (legacy CommonJS Node.js), `"node16"`/`"nodenext"` (modern Node.js with ESM and CJS dual support), and `"bundler"` (bundler-style resolution that never requires file extensions and supports `package.json` `"imports"` and `"exports"`). Resolution priorities differ per mode, but generally: relative paths resolve from the importing file's directory; bare specifiers resolve through `node_modules`; path aliases (`paths` + `baseUrl`) provide project-internal shortcuts; `package.json` `"exports"` maps define a package's public entry points and block deep imports; and `package.json` `"imports"` maps define package-internal aliases prefixed with `#`.

**Beginner-Friendly Explanation**
Module resolution is TypeScript's way of figuring out what file you're importing. When you write `import { Button } from "./components/Button"`, TypeScript looks for `Button.ts` in the `components` folder. When you write `import { Button } from "@myorg/ui"`, TypeScript looks in `node_modules` for a package called `@myorg/ui`. The rules for *how* TypeScript searches depend on the `moduleResolution` setting in your `tsconfig.json`. Modern projects use either `"bundler"` (if you use Vite, webpack, or esbuild) or `"nodenext"` (if you run your code directly in Node.js). Getting this setting right is crucial—wrong settings cause "Cannot find module" errors, broken imports, and build failures.

### Key Characteristics

- **Strategy-driven**: The `moduleResolution` option determines the resolution algorithm.
- **Environment-specific**: Different strategies model different runtimes (Node.js CJS, Node.js ESM, bundlers).
- **Priority-ordered**: Resolution follows a specific order (relative → aliases → packages → bare specifiers).
- **Extension-aware**: Node.js ESM and `node16`/`nodenext` require explicit `.js` extensions on relative imports; `bundler` does not.
- **Exports-field-aware**: `node16`, `nodenext`, and `bundler` honor `package.json` `"exports"` and `"imports"` maps; `node10` does not.
- **Alias-capable**: `baseUrl` + `paths` provide project-internal path shortcuts.
- **Package-internal aliases**: `#`-prefixed subpath imports (Node.js `"imports"` field) provide native, runtime-resolvable internal aliases.

### Prerequisites

- Basic knowledge of TypeScript `import`/`export` syntax
- Familiarity with `tsconfig.json` and compiler options
- Understanding of npm packages and `node_modules`
- Basic familiarity with Node.js runtime and bundlers (Vite, webpack, esbuild)

### Related Programming Areas

- **ES Modules**: The module system underlying resolution
- **Node.js Runtime**: Native ESM resolution rules (`node16`/`nodenext`)
- **Bundlers**: Vite, webpack, Rollup, esbuild resolution (`bundler`)
- **Monorepos**: Workspace package resolution
- **Package Design**: `exports` and `imports` maps for library authors

### Core Concepts / Features

1. Module Resolution Strategies (`classic`, `node10`, `node16`, `nodenext`, `bundler`)
2. Relative Imports vs. Absolute Package Imports
3. Path Aliases, `baseUrl`, and the `paths` Compiler Configuration
4. Modern Package Entry Points via `exports` Fields (Conditional Exports)
5. Native Node.js Subpath Imports (`#` Prefixes) and TypeScript Configuration Integration


## 1. Module Resolution Strategies (`classic`, `node10`, `node16`, `nodenext`, `bundler`)

### Definitions

**Core Definition**
TypeScript offers five `moduleResolution` strategies, each modeling a different environment. The right choice depends on whether your code runs in Node.js (and which version), in a bundler (Vite, webpack), or in a legacy environment. The strategy determines how TypeScript searches for files, whether extensions are required, and whether `package.json` `"exports"` maps are honored.

**Technical Definition**
The `moduleResolution` compiler option accepts the following values:
- `"classic"`: The original TypeScript resolution strategy (pre-1.6). Deprecated and should not be used.
- `"node10"` (previously `"node"`): Models Node.js versions older than v10, which only support CommonJS `require`. It ignores `package.json` `"exports"` maps and allows extensionless imports and directory index resolution.
- `"node16"`: Models modern Node.js (v16+) with dual ESM/CJS support. It requires explicit file extensions on relative imports, honors `"exports"` and `"imports"` maps, and picks the right algorithm based on whether Node.js will see an `import` or `require` in the output.
- `"nodenext"`: An alias for `"node16"` that stays up to date with the latest Node.js resolution behavior. Recommended for Node.js applications.
- `"bundler"`: Models how modern bundlers (Vite, webpack, esbuild) resolve imports. Like `node16`/`nodenext`, it honors `"exports"` and `"imports"` maps, but unlike the Node.js modes, it never requires file extensions on relative paths. It also enables extensionless and directory index resolution.

The `module` option must be set consistently with `moduleResolution`. For example, `"module": "nodenext"` pairs with `"moduleResolution": "nodenext"`, while `"module": "esnext"` pairs with `"moduleResolution": "bundler"`.

**Beginner-Friendly Explanation**
Think of `moduleResolution` as choosing the "rulebook" TypeScript uses to find files. Different environments have different rules:
- **`bundler`**: Use this if you're using Vite, webpack, or esbuild. It's the most permissive—you can write `import { Button } from "./Button"` without the `.js` extension.
- **`nodenext`**: Use this if you're running code directly in Node.js. It's stricter—you must write `import { Button } from "./Button.js"` even though the file is `Button.ts`. It also respects `package.json` `"exports"` maps.
- **`node10`**: Legacy. Only use if you're stuck on old CommonJS Node.js. Avoid it for new projects.
- **`classic`**: Deprecated. Don't use it.

If you're unsure, ask: "Does my code go through a bundler, or does Node.js run it directly?" Bundler → `bundler`. Node.js → `nodenext`.

### Purposes

- To match TypeScript's resolution behavior to the actual runtime or bundler environment.
- To enable or disable file extension requirements, `exports` map support, and directory index resolution.
- To prevent runtime errors caused by TypeScript resolving imports differently from the runtime.
- To provide optimal developer experience (fewer extensions to type) for bundled projects.
- To enforce Node.js ESM compliance for direct Node.js execution.

### Syntax Rules and Structure

**General Syntax: `tsconfig.json` Configuration**

```json
{
  "compilerOptions": {
    "module": "nodenext",
    "moduleResolution": "nodenext"
  }
}
```

**Component Breakdown**
- `"module": "nodenext"`: Emits ESM or CJS based on `package.json` `"type"`.
- `"moduleResolution": "nodenext"`: Resolves imports using modern Node.js rules.

**General Syntax: Bundler Configuration**

```json
{
  "compilerOptions": {
    "module": "esnext",
    "moduleResolution": "bundler"
  }
}
```

**Component Breakdown**
- `"module": "esnext"`: Emits ESM syntax.
- `"moduleResolution": "bundler"`: Resolves imports using bundler rules.

**General Syntax: Legacy CommonJS Configuration**

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "moduleResolution": "node10"
  }
}
```

**Component Breakdown**
- `"module": "commonjs"`: Emits CommonJS `require` syntax.
- `"moduleResolution": "node10"`: Legacy Node.js resolution.

**Syntax Rules**

- `moduleResolution` must be compatible with `module`.
- `"bundler"` never requires file extensions on relative imports.
- `"node16"` and `"nodenext"` require explicit `.js` extensions on relative imports.
- `"node16"`, `"nodenext"`, and `"bundler"` honor `package.json` `"exports"` and `"imports"` maps.
- `"node10"` ignores `"exports"` maps.
- `"classic"` should not be used.
- TypeScript 7.0 will drop support for `node10`/`node`.

**Constraints and Limitations**

- `"bundler"` is more permissive than Node.js's actual resolver—code correct under `bundler` is not proven correct for direct Node.js execution.
- `"node16"`/`"nodenext"` require `.js` extensions even for `.ts` source files.
- `"node10"` is deprecated and will be removed.
- `"classic"` is deprecated and should not be used.
- The `module` and `moduleResolution` options must be paired correctly; invalid combinations are rejected.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Bundler vs. NodeNext Resolution

```json
// Step 1: Bundler configuration — extensions optional.
// tsconfig.json (bundler)
{
  "compilerOptions": {
    "module": "esnext",
    "moduleResolution": "bundler"
  }
}
```

```typescript
// src/app.ts (bundler)
import { Button } from "./components/Button";  // ✅ Allowed (no extension)
console.log(Button);
```

```json
// Step 2: NodeNext configuration — extensions required.
// tsconfig.json (nodenext)
{
  "compilerOptions": {
    "module": "nodenext",
    "moduleResolution": "nodenext"
  }
}
```

```typescript
// src/app.ts (nodenext)
import { Button } from "./components/Button.js";  // ✅ Required (.js extension)
// import { Button } from "./components/Button";  // ❌ Error: Missing extension
console.log(Button);
```

**Expected Output:** The bundler version compiles without extensions. The NodeNext version requires the `.js` extension and produces a compile error without it.

**Why This Output Occurs:** `"bundler"` models how bundlers resolve imports—they handle extension resolution internally. `"nodenext"` models Node.js ESM, which requires explicit file extensions for relative imports.

#### Example 2: Exports Map Support

```json
// Step 1: Package with an exports map.
// node_modules/my-lib/package.json
{
  "name": "my-lib",
  "exports": {
    ".": "./dist/index.js",
    "./utils": "./dist/utils.js"
  }
}
```

```typescript
// Step 2: With node16/nodenext/bundler, the exports map is honored.
import { helper } from "my-lib";           // ✅ Resolves to ./dist/index.js
import { format } from "my-lib/utils";     // ✅ Resolves to ./dist/utils.js
// import { internal } from "my-lib/dist/internal";  // ❌ Blocked by exports map
```

```typescript
// Step 3: With node10, the exports map is ignored.
// The resolver falls back to main/types and allows deep imports.
import { internal } from "my-lib/dist/internal";  // ⚠️ Allowed under node10
```

**Expected Output:** Under `node16`/`nodenext`/`bundler`, deep imports are blocked by the exports map. Under `node10`, deep imports are allowed because the exports map is ignored.

**Why This Output Occurs:** `node10` predates the `exports` field and resolves packages by looking at `main`, `types`, and the file system directly. `node16`/`nodenext`/`bundler` honor the `exports` map, which restricts which paths are accessible.

### Real-World Cases

**Case 1: Vite/React Projects**
Vite projects use `"moduleResolution": "bundler"` with `"module": "esnext"`, allowing extensionless imports and fast development builds.

**Case 2: Node.js ESM APIs**
Node.js APIs using `"type": "module"` in `package.json` use `"module": "nodenext"` and `"moduleResolution": "nodenext"`, ensuring that compiled output runs correctly in Node.js with explicit `.js` extensions.

**Case 3: Library Publishing**
Libraries targeting both bundlers and Node.js use `"module": "nodenext"` and `"moduleResolution": "nodenext"` with an `exports` map, ensuring maximum compatibility.

**Case 4: Legacy CommonJS**
Older Node.js projects using `require` use `"module": "commonjs"` and `"moduleResolution": "node10"`, though this is deprecated for new projects.

### References

- TSConfig Reference: `moduleResolution` — https://www.typescriptlang.org/tsconfig/moduleResolution.html
- TypeScript Handbook: Modules Reference (Module Resolution) — https://www.typescriptlang.org/docs/handbook/modules/reference.html
- TypeScript 6.0 Release Notes — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-6-0.html
- TypeScript GitHub Issue #61454: `moduleResolution` defaults — https://github.com/microsoft/TypeScript/issues/61454
- TypeScript PR #51669: `--moduleResolution bundler` — https://github.com/microsoft/TypeScript/pull/51669


## 2. Relative Imports vs. Absolute Package Imports

### Definitions

**Core Definition**
Relative imports begin with `./` or `../` and resolve from the importing file's directory. Absolute package imports are bare specifiers (e.g., `"lodash"`, `"@myorg/ui"`) that resolve through `node_modules` or path aliases. The distinction determines which resolution rules apply.

**Technical Definition**
Relative imports (`"./utils"`, `"../components/Button"`) resolve relative to the importing file's directory. In `node16`/`nodenext`, they require explicit file extensions (`.js`). Absolute package imports (`"lodash"`, `"@myorg/ui"`, `"react"`) are bare specifiers that do not start with `.` or `/`. They resolve through `node_modules` by package name, respecting the package's `package.json` `"exports"`, `"main"`, and `"types"` fields. Path aliases (`paths` + `baseUrl`) also produce absolute-style imports (e.g., `"@/components/Button"`) but resolve through TypeScript's alias configuration, not `node_modules`.

**Beginner-Friendly Explanation**
Relative imports are like giving directions from where you are: "go up one folder, then into `components`." They work but become messy: `../../../components/Button`. Absolute imports are like using a name: "`@myorg/ui`" or "`lodash`." They're cleaner and don't break when you move files. The trade-off is that absolute imports need configuration (either `node_modules` or path aliases). In modern TypeScript, most projects use a mix: relative imports for files in the same feature, absolute imports for shared code and external packages.

### Purposes

- To choose the right import style based on file proximity and project structure.
- To avoid fragile `../../../` paths that break on refactoring.
- To leverage package resolution for external dependencies.
- To use path aliases for clean, project-internal absolute imports.
- To understand extension requirements for each import type in each resolution mode.

### Syntax Rules and Structure

**General Syntax: Relative Import**

```typescript
import { helper } from "./utils";           // Same directory
import { Button } from "../components/Button";  // Parent directory
import { config } from "../../config";      // Two levels up
```

**Component Breakdown**
- `./`: Current directory.
- `../`: Parent directory.
- Resolution is relative to the importing file.

**General Syntax: Absolute Package Import**

```typescript
import { useState } from "react";           // Bare specifier
import { Button } from "@myorg/ui";         // Scoped package
import { helper } from "lodash-es";         // Named package
```

**Component Breakdown**
- Does not start with `.` or `/`.
- Resolves through `node_modules`.

**General Syntax: Path Alias Import**

```typescript
import { Button } from "@/components/Button";  // Alias for src/components
```

**Component Breakdown**
- `@/`: A configured alias (via `tsconfig.json` `paths`).
- Resolves through TypeScript's alias configuration.

**Syntax Rules**

- Relative imports start with `./` or `../`.
- Absolute package imports are bare specifiers.
- In `node16`/`nodenext`, relative imports require `.js` extensions.
- In `bundler`, relative imports may omit extensions.
- Path aliases are configured in `tsconfig.json` `paths`.
- Bare specifiers resolve through `node_modules` (or `imports` maps for `#` prefixes).

**Constraints and Limitations**

- Relative imports break when files are moved or directories are refactored.
- Deep relative paths (`../../../../`) reduce readability.
- Absolute package imports require the package to be installed in `node_modules`.
- Path aliases require configuration in both `tsconfig.json` and the bundler.
- `node16`/`nodenext` require `.js` extensions on relative imports, which can be confusing for `.ts` source files.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Relative vs. Absolute vs. Alias

```typescript
// Step 1: Relative imports (fragile, breaks on refactoring).
// src/features/dashboard/components/Dashboard.tsx
import { Button } from "../../../components/ui/Button";  // ❌ Hard to read

// Step 2: Absolute package imports (external dependencies).
import { useState } from "react";  // ✅ Clean, package-based

// Step 3: Path alias imports (project-internal absolute).
// With tsconfig paths: { "@/*": ["src/*"] }
import { Button } from "@/components/ui/Button";  // ✅ Clean, refactor-safe

// Step 4: Configure paths in tsconfig.json.
// {
//   "compilerOptions": {
//     "baseUrl": ".",
//     "paths": {
//       "@/*": ["src/*"],
//       "@/components/*": ["src/components/*"],
//       "@/utils/*": ["src/utils/*"]
//     }
//   }
// }

// Step 5: Configure the bundler to match.
// vite.config.ts
// resolve: {
//   alias: {
//     "@": path.resolve(__dirname, "./src"),
//   },
// }

console.log("Import styles demonstrated.");
```

**Expected Output:**
```
Import styles demonstrated.
```

**Why This Output Occurs:** Relative imports are functional but fragile. Absolute package imports resolve through `node_modules`. Path aliases provide clean, project-internal absolute imports. The key is that TypeScript and the bundler must agree on alias meanings.

#### Example 2: Extension Requirements by Mode

```typescript
// Step 1: Bundler mode — extensions optional.
// tsconfig: { "moduleResolution": "bundler" }
import { Button } from "./components/Button";  // ✅ No extension

// Step 2: NodeNext mode — extensions required.
// tsconfig: { "moduleResolution": "nodenext" }
import { Button } from "./components/Button.js";  // ✅ Required
// import { Button } from "./components/Button";  // ❌ Error

// Step 3: The extension references the compiled output, not the source.
// Source file: components/Button.ts
// Compiled file: components/Button.js
// Import path: "./components/Button.js"

// Step 4: Package imports never require extensions.
import { useState } from "react";  // ✅ Always fine
import { Button } from "@myorg/ui";  // ✅ Always fine

console.log("Extension requirements demonstrated.");
```

**Expected Output:**
```
Extension requirements demonstrated.
```

**Why This Output Occurs:** `bundler` mode allows extensionless imports because the bundler handles resolution. `nodenext` mode requires `.js` extensions because Node.js ESM requires them. Package imports never require extensions because they resolve through `node_modules`.

### Real-World Cases

**Case 1: React Feature-Based Projects**
Feature-based React projects use path aliases (`@/features/auth`, `@/shared/ui`) for cross-feature imports and relative imports within a feature.

**Case 2: Node.js API Libraries**
Node.js API libraries use relative imports with `.js` extensions (required by `nodenext`) and absolute imports for dependencies like `express`, `zod`, and `prisma`.

**Case 3: Monorepo Packages**
Monorepo packages use workspace package imports (`@myorg/ui`) for cross-package dependencies and path aliases for intra-package shortcuts.

### References

- TypeScript Handbook: Module Resolution Theory — https://www.typescriptlang.org/docs/handbook/modules/theory.html
- TypeScript Handbook: Choosing Compiler Options — https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html
- Steve Kinney: Module Resolution and Path Aliases — https://stevekinney.com/courses/react-typescript/module-resolution-and-paths


## 3. Path Aliases, `baseUrl`, and the `paths` Compiler Configuration

### Definitions

**Core Definition**
Path aliases are project-internal shortcuts defined in `tsconfig.json` using the `paths` option, which maps import specifiers (e.g., `"@/components/*"`) to directories on disk (e.g., `"src/components/*"`). The `baseUrl` option provides the base directory for resolving non-relative module names and is traditionally required for `paths` to work.

**Technical Definition**
The `baseUrl` compiler option sets the base directory for resolving non-relative module specifiers. The `paths` option defines a mapping from import specifiers to file paths, resolved relative to `baseUrl`. TypeScript applies `paths` mappings before falling back to `node_modules` resolution. `paths` entries are matched in longest-prefix-first order. Since TypeScript 4.1, `baseUrl` is no longer strictly required—`paths` can be used without it, with paths resolved relative to `tsconfig.json`. However, bundlers (Vite, webpack) require their own alias configuration to match TypeScript's `paths` behavior, since TypeScript only handles type checking and does not rewrite import paths at runtime.

**Beginner-Friendly Explanation**
Path aliases let you use short, clean import paths instead of long relative paths. Instead of `import { Button } from "../../../components/Button"`, you write `import { Button } from "@/components/Button"`. You configure this in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/components/*": ["src/components/*"]
    }
  }
}
```

The catch: TypeScript handles the type checking, but your bundler (Vite, webpack, etc.) needs its own alias configuration to make the imports work at runtime. If you configure only one, you'll get "module not found" errors in the other.

### Purposes

- To replace fragile relative paths (`../../../`) with clean absolute imports.
- To create project-internal shortcuts for commonly used directories.
- To make imports refactor-safe (moving a file doesn't break its imports).
- To improve code readability and reduce cognitive load.
- To standardize import conventions across a team.

### Syntax Rules and Structure

**General Syntax: `paths` + `baseUrl` Configuration**

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/components/*": ["src/components/*"],
      "@/utils/*": ["src/utils/*"],
      "@/hooks/*": ["src/hooks/*"]
    }
  }
}
```

**Component Breakdown**
- `"baseUrl": "."`: The base directory for resolving paths.
- `"@/*": ["src/*"]`: Maps `@/anything` to `src/anything`.
- `"@/components/*": ["src/components/*"]`: More specific mapping.

**General Syntax: Common Alias Patterns**

```json
{
  "paths": {
    "@/*": ["src/*"],
    "~/*": ["src/*"],
    "@/components/*": ["src/components/*"],
    "@/utils/*": ["src/utils/*"]
  }
}
```

**Component Breakdown**
- `@/`: Popular with Vue/Nuxt.
- `~/`: Popular with Next.js.
- Descriptive names: `components/*`, `utils/*`.

**General Syntax: Bundler Alias Configuration (Vite)**

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import path from "path";

export default defineConfig({
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
      "@/components": path.resolve(__dirname, "./src/components"),
    },
  },
});
```

**Component Breakdown**
- `resolve.alias`: Must match `tsconfig.json` `paths`.

**Syntax Rules**

- `baseUrl` sets the base directory for `paths` resolution.
- `paths` maps specifiers to file paths, resolved relative to `baseUrl`.
- `paths` entries are matched longest-prefix-first.
- Since TypeScript 4.1, `baseUrl` is optional if `paths` entries start with `./`.
- Bundlers require their own alias configuration to match `paths`.
- In TypeScript 7.0, `baseUrl` is deprecated; `paths` entries must start with `./`.
- Use one alias convention consistently across the project.

**Constraints and Limitations**

- `paths` only affects type checking; bundlers must be configured separately.
- If bundler aliases don't match `paths`, runtime errors occur.
- `baseUrl` is deprecated in TypeScript 7.0.
- Overly broad aliases can mask import errors.
- Nested `tsconfig.json` files may need `paths` in the correct config file.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Setting Up Path Aliases

```json
// Step 1: Configure tsconfig.json.
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/components/*": ["src/components/*"],
      "@/utils/*": ["src/utils/*"]
    }
  }
}
```

```typescript
// Step 2: Use aliases in imports.
// src/features/dashboard/Dashboard.tsx
import { Button } from "@/components/ui/Button";
import { formatDate } from "@/utils/date";
import { useAuth } from "@/hooks/useAuth";

console.log("Clean imports with aliases.");
```

```typescript
// Step 3: Configure Vite to match.
// vite.config.ts
import { defineConfig } from "vite";
import path from "path";

export default defineConfig({
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
      "@/components": path.resolve(__dirname, "./src/components"),
      "@/utils": path.resolve(__dirname, "./src/utils"),
    },
  },
});
```

**Expected Output:**
```
Clean imports with aliases.
```

**Why This Output Occurs:** The `tsconfig.json` `paths` configuration tells TypeScript how to resolve `@/components/ui/Button`. The Vite `resolve.alias` configuration tells the bundler the same. Both must agree, or the import fails at one stage or the other.

#### Example 2: `baseUrl` Deprecation and Migration

```json
// Step 1: Traditional configuration (baseUrl + paths).
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

```json
// Step 2: TypeScript 7.0 configuration (baseUrl deprecated).
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

```typescript
// Step 3: Imports work the same way.
import { Button } from "@/components/Button";
console.log(Button);
```

**Expected Output:**
```
[Function: Button]
```

**Why This Output Occurs:** In TypeScript 7.0, `baseUrl` is deprecated. The `paths` entries must start with `./` to be resolved relative to `tsconfig.json`. The behavior is otherwise identical.

### Real-World Cases

**Case 1: Vue/Nuxt Projects**
Vue and Nuxt projects commonly use `@/` as the alias for `src/`, configured in `tsconfig.json` and `vite.config.ts` or `nuxt.config.ts`.

**Case 2: Next.js Projects**
Next.js projects use `~/*` for `src/*` or `@/*` for the project root, configured in `tsconfig.json` and `next.config.js`.

**Case 3: React + Vite Projects**
React + Vite projects use `@/` for `src/`, with matching configuration in `tsconfig.json` and `vite.config.ts`.

**Case 4: Monorepo Packages**
Monorepo packages use workspace package names (`@myorg/ui`) for cross-package imports and `paths` aliases for intra-package shortcuts.

### References

- TypeScript TSConfig Reference: `paths` — https://www.typescriptlang.org/tsconfig#paths
- TypeScript TSConfig Reference: `baseUrl` — https://www.typescriptlang.org/tsconfig#baseUrl
- TypeScript GitHub Issue #62207: Deprecate `baseUrl` — https://github.com/microsoft/TypeScript/issues/62207
- Steve Kinney: Module Resolution and Path Aliases — https://stevekinney.com/courses/react-typescript/module-resolution-and-paths


## 4. Modern Package Entry Points via `exports` Fields (Conditional Exports)

### Definitions

**Core Definition**
The `exports` field in `package.json` defines a package's public entry points, replacing the legacy `main` and `types` fields. Conditional exports allow a package to provide different entry points for different environments (ESM `import`, CommonJS `require`, TypeScript `types`, browser, Node.js, etc.), evaluated top-to-bottom with the first match winning.

**Technical Definition**
The `exports` field, introduced in Node.js 12 and supported by webpack 5 and modern bundlers, explicitly declares which import paths a package allows. It supports conditional exports, where each condition (e.g., `"import"`, `"require"`, `"types"`, `"node"`, `"browser"`, `"default"`) maps to a different file. TypeScript's `node16`, `nodenext`, and `bundler` resolution modes honor `exports` maps, while `node10` ignores them. The `types` condition must be listed first in each exports condition block to ensure TypeScript resolves type declarations correctly. When a package has an `exports` field, deep imports not listed in the map are blocked, enforcing encapsulation.

**Beginner-Friendly Explanation**
The `exports` field is how modern npm packages say "here's what you can import from me." Instead of allowing any file to be imported (`my-lib/dist/internal`), the package author explicitly lists the allowed entry points: `"."` (the root), `"./utils"`, `"./hooks"`. Each entry can have conditions: if you're importing with ESM (`import`), use `./dist/index.mjs`; if you're using CommonJS (`require`), use `./dist/index.cjs`; if TypeScript is looking for types, use `./dist/index.d.ts`. This is crucial for dual-format packages that support both ESM and CJS. It also blocks deep imports, keeping internal implementation details private.

### Purposes

- To define a package's public API explicitly and block deep imports.
- To support dual ESM/CJS packages with different entry points per format.
- To provide TypeScript type declarations per condition (`types` condition).
- To enable environment-specific resolution (browser vs. Node.js).
- To replace the legacy `main`/`module`/`types` fields with a single, authoritative map.

### Syntax Rules and Structure

**General Syntax: Basic `exports` Map**

```json
{
  "name": "my-lib",
  "exports": {
    ".": "./dist/index.js",
    "./utils": "./dist/utils.js"
  }
}
```

**Component Breakdown**
- `"."`: The package root (main entry).
- `"./utils"`: A subpath export.
- Deep imports not listed are blocked.

**General Syntax: Conditional Exports (ESM/CJS)**

```json
{
  "name": "my-lib",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs",
      "default": "./dist/index.js"
    }
  }
}
```

**Component Breakdown**
- `"types"`: TypeScript type declarations (must be first).
- `"import"`: ESM entry point.
- `"require"`: CommonJS entry point.
- `"default"`: Fallback.

**General Syntax: Subpath Exports**

```json
{
  "name": "my-lib",
  "exports": {
    ".": "./dist/index.js",
    "./utils": {
      "types": "./dist/utils.d.ts",
      "import": "./dist/utils.mjs",
      "require": "./dist/utils.cjs"
    }
  }
}
```

**Component Breakdown**
- `"./utils"`: A subpath entry point with its own conditions.
- Consumers import from `my-lib/utils`.

**General Syntax: Browser vs. Node Conditions**

```json
{
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "browser": "./dist/browser.js",
      "node": "./dist/node.js",
      "default": "./dist/index.js"
    }
  }
}
```

**Component Breakdown**
- `"browser"`: Used by bundlers targeting browsers.
- `"node"`: Used by Node.js.

**Syntax Rules**

- `exports` replaces `main`/`module`/`types` as the primary entry point definition.
- Conditions are evaluated top-to-bottom; first match wins.
- `"types"` must be listed first in each condition block.
- `"import"` and `"require"` toggle ESM vs. CJS per consumer.
- Subpath exports (`"./utils"`) create public API boundaries.
- Deep imports not listed in `exports` are blocked in `node16`/`nodenext`/`bundler`.
- `node10` ignores `exports` maps.

**Constraints and Limitations**

- `exports` maps are not supported by `node10` resolution.
- Existing packages that add `exports` may break consumers using deep imports.
- Different APIs for `import` vs. `require` conditions are error-prone.
- `"types"` must be first in each condition block, or TypeScript fails to resolve types.
- `exports` maps are not universally supported by all tools in all configurations.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Dual-Format Package with `exports`

```json
// Step 1: Package.json with conditional exports.
{
  "name": "@myorg/sdk",
  "version": "1.0.0",
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs"
    },
    "./utils": {
      "types": "./dist/utils.d.ts",
      "import": "./dist/utils.mjs",
      "require": "./dist/utils.cjs"
    }
  }
}
```

```typescript
// Step 2: ESM consumer (import).
import { createClient } from "@myorg/sdk";       // ✅ Resolves to index.mjs
import { formatUrl } from "@myorg/sdk/utils";    // ✅ Resolves to utils.mjs

// Step 3: CJS consumer (require).
const { createClient } = require("@myorg/sdk");  // ✅ Resolves to index.cjs

// Step 4: Deep imports are blocked.
// import { internal } from "@myorg/sdk/dist/internal";  // ❌ Blocked
```

**Expected Output:** ESM consumers get `.mjs` files; CJS consumers get `.cjs` files. Deep imports are blocked.

**Why This Output Occurs:** The `exports` map defines `"."` and `"./utils"` as the only allowed entry points. Each has `"import"` and `"require"` conditions, so ESM and CJS consumers get the correct files. Deep imports are blocked because they're not listed.

#### Example 2: TypeScript Type Resolution with `exports`

```json
// Step 1: Package.json with types condition.
{
  "name": "@myorg/sdk",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs"
    }
  }
}
```

```typescript
// Step 2: TypeScript resolves types through the "types" condition.
import { createClient } from "@myorg/sdk";
// TypeScript uses ./dist/index.d.ts for type checking
const client = createClient({ url: "https://api.example.com" });
```

```json
// Step 3: Without types in exports, TypeScript cannot resolve types.
{
  "exports": {
    ".": "./dist/index.js"
  }
}
// TypeScript falls back to the top-level "types" field, if present.
// If no "types" field exists, type resolution fails.
```

**Expected Output:** With the `"types"` condition, TypeScript resolves type declarations. Without it, TypeScript may fail to find types.

**Why This Output Occurs:** TypeScript's `node16`, `nodenext`, and `bundler` resolution modes strictly follow the `exports` map and ignore the top-level `"types"` field when an `exports` map exists. The `"types"` condition must be inside the `exports` map.

### Real-World Cases

**Case 1: Dual-Format Libraries**
Modern libraries (e.g., `zod`, `@tanstack/query`) use conditional exports to ship both ESM and CJS builds with TypeScript types, ensuring compatibility across runtimes.

**Case 2: Browser/Node Packages**
Packages that have different implementations for browsers and Node.js use `"browser"` and `"node"` conditions in their `exports` map.

**Case 3: SDK Packages**
SDKs use subpath exports (`"./utils"`, `"./hooks"`) to expose specific APIs while blocking deep imports into internal modules.

**Case 4: Monorepo Packages**
Monorepo packages use `exports` maps combined with workspace resolution to provide clean public APIs for internal consumers.

### References

- Node.js Packages: `exports` Field — https://nodejs.org/api/packages.html#exports
- Node.js Packages: Conditional Exports — https://nodejs.org/api/packages.html#conditional-exports
- TypeScript Handbook: Modules Reference (Package Exports) — https://www.typescriptlang.org/docs/handbook/modules/reference.html
- TypeScript TSConfig: `customConditions` — https://www.typescriptlang.org/tsconfig#customConditions
- Ship Library Types with `exports` and `typesVersions` Maps — https://raw.githubusercontent.com/pproenca/dot-skills
- Package Publishing Skill — https://github.com/oakoss/agent-skills


## 5. Native Node.js Subpath Imports (`#` Prefixes) and TypeScript Configuration Integration

### Definitions

**Core Definition**
Subpath imports are a Node.js feature (stable since Node.js 14.6.0) that allows package-internal aliases prefixed with `#`. Defined in the `"imports"` field of `package.json`, they provide a native, runtime-resolvable alternative to TypeScript's `paths` aliases for package-internal imports. TypeScript 6.0 supports `#/*`-style subpath imports under `bundler` and `nodenext` resolution modes.

**Technical Definition**
The `"imports"` field in `package.json` defines private package-internal mappings that must start with `#`. Unlike `exports` (which defines external entry points), `imports` defines internal aliases that are only accessible within the package. Subpath imports support conditional resolution (e.g., `"node"`, `"default"`) and pattern matching (`"#lib/*": "./dist/lib/*"`). TypeScript added auto-import support for subpath imports in 5.4, and TypeScript 6.0 extended this to support `#/*` prefix syntax under `nodenext` and `bundler` moduleResolution. Unlike `tsconfig.json` `paths`, subpath imports are resolved by Node.js at runtime, so they work without bundler alias configuration.

**Beginner-Friendly Explanation**
Subpath imports are Node.js's native way to create internal aliases. Instead of `import { helper } from "../../../utils/helper"`, you write `import { helper } from "#utils/helper"`. You define these in `package.json`:

```json
{
  "imports": {
    "#utils/*": "./dist/utils/*",
    "#components/*": "./dist/components/*"
  }
}
```

The key difference from `tsconfig.json` `paths` is that subpath imports are resolved by Node.js itself—no bundler configuration needed. They must start with `#`. TypeScript 6.0 supports them in `nodenext` and `bundler` modes. This is the modern standard for package-internal aliases, replacing the older `paths` approach.

### Purposes

- To provide package-internal aliases that resolve natively in Node.js.
- To replace `tsconfig.json` `paths` aliases with a runtime-resolvable standard.
- To avoid bundler-specific alias configuration for internal imports.
- To enable conditional internal resolution (e.g., development vs. production builds).
- To align TypeScript resolution with Node.js's native resolution behavior.

### Syntax Rules and Structure

**General Syntax: Basic Subpath Imports**

```json
{
  "name": "my-package",
  "imports": {
    "#lib/*": "./dist/lib/*",
    "#utils/*": "./dist/utils/*"
  }
}
```

**Component Breakdown**
- `"#lib/*"`: The internal alias.
- `"./dist/lib/*"`: The target path.
- Imports must start with `#`.

**General Syntax: Subpath Imports with Conditions**

```json
{
  "imports": {
    "#dep": {
      "node": "dep-node-native",
      "default": "./dep-polyfill.js"
    }
  }
}
```

**Component Breakdown**
- `"node"`: Node.js-specific resolution.
- `"default"`: Fallback for other environments.

**General Syntax: TypeScript Configuration**

```json
{
  "compilerOptions": {
    "module": "nodenext",
    "moduleResolution": "nodenext"
  }
}
```

**Component Breakdown**
- TypeScript 6.0+ supports `#/*` subpath imports under `nodenext` and `bundler`.

**General Syntax: Using Subpath Imports in Code**

```typescript
import { helper } from "#utils/helper";
import { Button } from "#components/Button";
```

**Component Breakdown**
- Imports start with `#`.
- Node.js resolves them natively.

**Syntax Rules**

- Subpath imports are defined in `package.json` `"imports"` field.
- They must start with `#`.
- They are private—only accessible within the package.
- TypeScript 6.0+ supports `#/*` under `nodenext` and `bundler`.
- They resolve natively in Node.js without bundler configuration.
- Conditional resolution is supported (`"node"`, `"default"`, etc.).
- Pattern matching is supported (`"#lib/*": "./dist/lib/*"`).

**Constraints and Limitations**

- Subpath imports require Node.js 14.6.0+ or TypeScript 6.0+ for `#/*` syntax.
- They are package-internal only—they cannot be imported by external consumers.
- They require the `"imports"` field in `package.json`.
- Some older tools may not support `#`-prefixed imports.
- They are not a replacement for `exports` maps (which define external entry points).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Subpath Imports for Internal Aliases

```json
// Step 1: Package.json with subpath imports.
{
  "name": "my-app",
  "imports": {
    "#utils/*": "./dist/utils/*",
    "#components/*": "./dist/components/*",
    "#lib/*": "./dist/lib/*"
  }
}
```

```typescript
// Step 2: Use subpath imports in code.
// src/app.ts
import { formatDate } from "#utils/formatDate";
import { Button } from "#components/Button";
import { apiClient } from "#lib/apiClient";

console.log("Subpath imports work natively.");
```

```json
// Step 3: TypeScript 6.0+ supports #/* under nodenext.
{
  "compilerOptions": {
    "module": "nodenext",
    "moduleResolution": "nodenext"
  }
}
```

**Expected Output:**
```
Subpath imports work natively.
```

**Why This Output Occurs:** Node.js resolves `#utils/formatDate` to `./dist/utils/formatDate` at runtime. TypeScript 6.0 resolves the same import for type checking under `nodenext` resolution. No bundler alias configuration is needed.

#### Example 2: Conditional Subpath Imports

```json
// Step 1: Package.json with conditional subpath imports.
{
  "imports": {
    "#config": {
      "development": "./config/development.js",
      "production": "./config/production.js",
      "default": "./config/default.js"
    }
  }
}
```

```typescript
// Step 2: Use the conditional import.
import { config } from "#config";
console.log(config.apiUrl);
```

```bash
# Step 3: Run with different conditions.
node --conditions=development app.js  # Loads development.js
node --conditions=production app.js   # Loads production.js
node app.js                           # Loads default.js
```

**Expected Output:** The configuration loaded depends on the `--conditions` flag.

**Why This Output Occurs:** Subpath imports support conditional resolution, enabling environment-specific internal modules. The `"development"`, `"production"`, and `"default"` conditions are evaluated based on the `--conditions` flag.

### Real-World Cases

**Case 1: SvelteKit**
SvelteKit automatically creates a `#lib` subpath import for `src/lib`, using Node.js's native subpath imports feature.

**Case 2: Node.js API Packages**
Node.js API packages use subpath imports (`#utils/*`, `#services/*`) for internal aliases, avoiding the need for bundler alias configuration.

**Case 3: Electron Applications**
Electron main-process applications use subpath imports combined with TypeScript's path mapping for clean internal imports.

**Case 4: Monorepo Packages**
Monorepo packages use subpath imports for intra-package aliases, while using workspace package names for cross-package imports.

### References

- Node.js Packages: Subpath Imports — https://nodejs.org/api/packages.html#subpath-imports
- Node.js Packages: `imports` Field — https://nodejs.org/api/packages.html#imports
- TypeScript 6.0 Release Notes: Subpath Imports — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-6-0.html
- TypeScript GitHub Issue #40062: Support Node.js subpath imports — https://github.com/microsoft/playwright/issues/40062
- SvelteKit: `#lib` Subpath Import — https://svelte.dev/docs/kit/$lib


## Summary: Module Resolution Feature Comparison

| Strategy | Environment | Extensions Required | `exports` Maps | `imports` Maps | Recommended For |
|---|---|---|---|---|---|
| `classic` | TypeScript pre-1.6 | No | No | No | Deprecated — do not use |
| `node10` | Node.js < v10 (CJS) | No | No | No | Legacy CommonJS only |
| `node16` | Node.js v16+ (ESM/CJS) | Yes (`.js`) | Yes | Yes | Node.js apps and libraries |
| `nodenext` | Latest Node.js | Yes (`.js`) | Yes | Yes | Node.js apps and libraries (recommended) |
| `bundler` | Vite, webpack, esbuild | No | Yes | Yes | Bundled apps (recommended) |


## References

- TSConfig Reference: `moduleResolution` — https://www.typescriptlang.org/tsconfig/moduleResolution.html
- TypeScript Handbook: Modules Reference — https://www.typescriptlang.org/docs/handbook/modules/reference.html
- TypeScript Handbook: Module Resolution Theory — https://www.typescriptlang.org/docs/handbook/modules/theory.html
- TypeScript Handbook: Choosing Compiler Options — https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html
- TypeScript TSConfig Reference: `paths` — https://www.typescriptlang.org/tsconfig#paths
- TypeScript TSConfig Reference: `baseUrl` — https://www.typescriptlang.org/tsconfig#baseUrl
- TypeScript TSConfig Reference: `customConditions` — https://www.typescriptlang.org/tsconfig#customConditions
- TypeScript 6.0 Release Notes — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-6-0.html
- Node.js Packages: `exports` Field — https://nodejs.org/api/packages.html#exports
- Node.js Packages: Conditional Exports — https://nodejs.org/api/packages.html#conditional-exports
- Node.js Packages: Subpath Imports — https://nodejs.org/api/packages.html#subpath-imports
- Node.js Packages: `imports` Field — https://nodejs.org/api/packages.html#imports
- TypeScript GitHub Issue #61454: `moduleResolution` defaults — https://github.com/microsoft/TypeScript/issues/61454
- TypeScript GitHub Issue #62207: Deprecate `baseUrl` — https://github.com/microsoft/TypeScript/issues/62207
- TypeScript PR #51669: `--moduleResolution bundler` — https://github.com/microsoft/TypeScript/pull/51669
- Steve Kinney: Module Resolution and Path Aliases — https://stevekinney.com/courses/react-typescript/module-resolution-and-paths
- Ship Library Types with `exports` and `typesVersions` Maps — https://raw.githubusercontent.com/pproenca/dot-skills
- Package Publishing Skill — https://github.com/oakoss/agent-skills