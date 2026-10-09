# Node.js TypeScript Tooling — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Node.js TypeScript tooling encompasses the compilers, bundlers, runtime loaders, linters, and configuration files that transform, execute, and validate TypeScript code in Node.js environments.

**Technical Definition:** TypeScript tooling for Node.js includes: the TypeScript compiler (`tsc`) for type checking and transpilation; bundlers/transpilers (`esbuild`, `swc`, `tsx`) for fast compilation; runtime loaders (`ts-node`, `tsx`) for executing TypeScript directly; source map generation for debugging; type-checking separation (`tsc --noEmit`); ESLint integration via `@typescript-eslint`; and module resolution configuration (`moduleResolution: "NodeNext"`, `"Bundler"`) for handling ESM `.js`/`.ts` extensions and `package.json` `exports` fields.

**Beginner-Friendly Explanation:** TypeScript isn't JavaScript — it has to be compiled before Node.js can run it. Tooling is the set of tools that do this compilation, run the code during development, catch errors before runtime, and integrate with your editor. Think of it like a kitchen: `tsc` is the oven (slow but thorough), `esbuild` and `swc` are the microwave (fast), `tsx` is the ready-made meal (instant), and ESLint is the food critic (checks quality). You pick the right tool for each situation.

### Key Characteristics

- **Compiler vs. transpiler:** `tsc` does both type checking and transpilation; `esbuild`/`swc` only transpile.
- **Speed vs. correctness:** Fast transpilers (esbuild, swc) are ~10–100× faster than `tsc` but skip type checking.
- **Runtime loaders:** `ts-node` and `tsx` execute TypeScript directly without a build step.
- **Source maps:** Enable debugging TypeScript in production by mapping compiled JS back to original TS.
- **Separation of concerns:** Bundling and type checking are separate steps in modern toolchains.
- **ESM compatibility:** `moduleResolution: "NodeNext"` handles `.js` extensions in imports.
- **Linting integration:** `@typescript-eslint` enables type-aware linting rules.

### Prerequisites

- **TypeScript fundamentals:** Types, interfaces, generics, strict mode.
- **Node.js fundamentals:** Modules, `package.json`, npm scripts.
- **ES modules:** `import`/`export`, `type: "module"`.
- **Command line:** Running `tsc`, `node`, `npm` commands.
- **Editor setup:** VS Code or an editor with TypeScript support.

### Related Programming Areas

- **TypeScript for Backend Development:** The code that tooling compiles.
- **Build systems:** Webpack, Vite, Rollup, esbuild.
- **Testing:** Vitest, Jest with TypeScript support.
- **Monorepos:** Project references, shared `tsconfig.json`.
- **CI/CD:** Build pipelines, caching, artifact generation.

### Core Concepts

1. **Compiler Configuration** — tuning `tsconfig.json` properties like `target`, `module`, and `lib`.
2. **Build Systems** — compiling code using `tsc`, `esbuild`, `swc`, or `tsx`.
3. **Runtime Execution** — running development instances with `ts-node`, `tsx`, or modern multi-loader environments.
4. **Source Maps** — configuring generation maps for production stack trace mapping and direct source debugging.
5. **Type Checking** — separating bundling from type checking using flags like `tsc --noEmit`.
6. **ESLint Integration** — wiring up `@typescript-eslint/parser` and configuring strict linting plugins.
7. **ESM Module Resolution Configuration** — configuring `moduleResolution: "NodeNext"` or `Bundler`.

---

## Core Concept 1: Compiler Configuration

### Definitions

**Core Definition:** Compiler configuration is the `tsconfig.json` file that tells TypeScript how to compile the project — what target, module system, libraries, and strictness rules to apply.

**Technical Definition:** `tsconfig.json` is the configuration file read by `tsc`. It has two top-level sections: `compilerOptions` (compiler behaviour) and `include`/`exclude`/`files` (which files to compile). Key options include `target` (ECMAScript version — ES2022, ESNext), `module` (module system — CommonJS, ESNext, NodeNext), `moduleResolution` (module resolution strategy — Node, NodeNext, Bundler), `lib` (type definitions for built-in APIs — ES2022, DOM), `outDir` (output directory), `rootDir` (source directory), `strict` (strict mode), `esModuleInterop` (CommonJS interop), `skipLibCheck` (skip `.d.ts` checking), `declaration` (emit `.d.ts` files), and `sourceMap` (emit source maps).

**Beginner-Friendly Explanation:** `tsconfig.json` is like the settings on a camera. You tell it what resolution to shoot in (`target`), what format to save (`module`), what kind of lens to use (`lib`), and how strict to be about focus (`strict`). Different projects need different settings — a Node.js backend needs different settings than a browser frontend or an npm library.

### Purposes

- To configure how TypeScript compiles the project.
- To enforce consistent settings across the team.
- To enable strict type checking.
- To specify where to find source files and where to emit output.
- To support different environments (Node.js, browser, library).

### Syntax Rules and Structure

#### Node.js Backend Configuration (Recommended)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": false,
    "allowSyntheticDefaultImports": true,
    "verbatimModuleSyntax": true,
    "importHelpers": true,
    "noEmit": false,
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

#### Configuration Options Explained

| Option | Values | Purpose |
|--------|--------|---------|
| `target` | `ES2022`, `ESNext`, `ES2020` | ECMAScript output version |
| `module` | `NodeNext`, `ESNext`, `CommonJS` | Module system |
| `moduleResolution` | `NodeNext`, `Bundler`, `Node` | How imports are resolved |
| `lib` | `ES2022`, `DOM`, `DOM.Iterable` | Built-in type definitions |
| `outDir` | `./dist` | Output directory |
| `rootDir` | `./src` | Source directory |
| `strict` | `true`/`false` | Enable all strict checks |
| `esModuleInterop` | `true`/`false` | CommonJS interop |
| `skipLibCheck` | `true`/`false` | Skip `.d.ts` checking |
| `isolatedModules` | `true`/`false` | Compatibility with esbuild/swc |
| `declaration` | `true`/`false` | Emit `.d.ts` files |
| `sourceMap` | `true`/`false` | Emit source maps |
| `verbatimModuleSyntax` | `true`/`false` | Enforce explicit `type` imports |
| `incremental` | `true`/`false` | Faster incremental builds |

#### Browser (React) Configuration

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "jsx": "react-jsx",
    "strict": true,
    "noEmit": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true
  },
  "include": ["src"]
}
```

#### Library Configuration (npm Package)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true
  },
  "include": ["src/**/*"],
  "exclude": ["**/*.test.ts", "**/*.spec.ts"]
}
```

#### Syntax Rules

- **Use `target: "ES2022"`** for Node.js 18+ — modern features are supported.
- **Use `module: "NodeNext"` and `moduleResolution: "NodeNext"`** for ESM Node.js.
- **Use `moduleResolution: "Bundler"`** for bundler-based projects (Vite, esbuild).
- **Use `strict: true`** — baseline for all projects.
- **Use `skipLibCheck: true`** — for faster compilation.
- **Use `isolatedModules: true`** — for esbuild/swc compatibility.
- **Use `verbatimModuleSyntax: true`** — enforce explicit `type` imports.
- **Use `declaration: true`** for libraries — emits `.d.ts` files.
- **Use `sourceMap: true`** for debugging.
- **Use `incremental: true`** for faster rebuilds.

#### Constraints and Limitations

- **`module: "NodeNext"` requires `.js` extensions** — in relative imports.
- **`moduleResolution: "Bundler"` is not for Node.js** — it is for bundlers.
- **`strict: true` may break existing code** — gradual adoption.
- **`skipLibCheck: true` hides errors** — in dependencies.
- **`verbatimModuleSyntax` requires `type` imports** — `import type { X }`.
- **`isolatedModules` disallows `const enum`** — use `as const` instead.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Node.js ESM Project with TypeScript

```json
// package.json
{
  "name": "my-app",
  "version": "1.0.0",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "scripts": {
    "build": "tsc",
    "dev": "tsx watch src/index.ts",
    "start": "node dist/index.js",
    "typecheck": "tsc --noEmit",
    "lint": "eslint . --ext .ts"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.5.0",
    "tsx": "^4.0.0",
    "eslint": "^9.0.0",
    "@typescript-eslint/eslint-plugin": "^8.0.0",
    "@typescript-eslint/parser": "^8.0.0"
  }
}
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "declaration": true,
    "sourceMap": true,
    "verbatimModuleSyntax": true,
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

```typescript
// src/index.ts — with NodeNext, imports use .js extension
import { greet } from './greet.js';
import type { User } from './types.js';

const user: User = { id: '1', name: 'Alice' };
console.log(greet(user));
```

```typescript
// src/greet.ts
import type { User } from './types.js';

export function greet(user: User): string {
  return `Hello, ${user.name}!`;
}
```

```typescript
// src/types.ts
export interface User {
  id: string;
  name: string;
}
```

**Expected behaviour:** `npm run build` compiles TypeScript to `dist/`. `npm run dev` runs the source directly with `tsx`. `npm run typecheck` checks types without emitting. Imports use `.js` extensions (required by `NodeNext`), but refer to `.ts` source files.

**Why this works:** `module: "NodeNext"` matches Node.js's ESM resolution. `type: "module"` in `package.json` enables ESM. The `exports` field defines entry points. `verbatimModuleSyntax` enforces explicit type imports.

### Real-World Cases

- **Node.js APIs:** `module: "NodeNext"`, `target: "ES2022"`.
- **React apps:** `moduleResolution: "Bundler"`, `jsx: "react-jsx"`.
- **npm libraries:** `declaration: true`, `module: "NodeNext"`.
- **Monorepos:** Project references with shared `tsconfig.base.json`.

---

## Core Concept 2: Build Systems

### Definitions

**Core Definition:** Build systems are the tools that compile TypeScript to JavaScript, ranging from the official `tsc` compiler to high-speed transpilers like `esbuild` and `swc`.

**Technical Definition:** Build systems for TypeScript include: **`tsc`** (official compiler, full type checking, slow), **`esbuild`** (Go-based transpiler, ~10–100× faster than `tsc`, no type checking), **`swc`** (Rust-based transpiler, similar speed to esbuild), **`tsx`** (esbuild-based TypeScript runner for Node.js), and **bundlers** (Vite, Webpack, Rollup) that integrate TypeScript via loaders. Modern best practice: use `tsc` for type checking (`--noEmit`) and a fast transpiler (esbuild, swc) for bundling.

**Beginner-Friendly Explanation:** Building TypeScript is like translating a document. `tsc` is a careful human translator (slow but accurate). `esbuild` and `swc` are machine translators (fast but skip nuance). The best approach: use the machine translator for speed, and the human translator to check quality (type checking) separately.

### Purposes

- To compile TypeScript to JavaScript.
- To bundle multiple files into a single output.
- To transpile modern syntax for older environments.
- To enable fast development iteration.
- To generate type declarations for libraries.

### Syntax Rules and Structure

#### Build Tool Comparison

| Tool | Language | Type Checking | Speed | Use Case |
|------|----------|---------------|-------|----------|
| **tsc** | TypeScript | ✅ Yes | Slow | Type checking, libraries |
| **esbuild** | Go | ❌ No | Very fast | Bundling, transpilation |
| **swc** | Rust | ❌ No | Very fast | Bundling, transpilation |
| **tsx** | Go (esbuild) | ❌ No | Very fast | Development runtime |
| **ts-node** | TypeScript | ⚠️ Optional | Slow | Development runtime |
| **Vite** | Go/Rust | ❌ No | Very fast | Frontend bundling |
| **Webpack** | JavaScript | ⚠️ Via plugin | Moderate | Complex bundling |

#### tsc Commands

```bash
# Build (emit JavaScript)
tsc

# Type check only (no emit)
tsc --noEmit

# Watch mode
tsc --watch

# Build a specific project
tsc --project tsconfig.build.json

# Generate declarations only
tsc --emitDeclarationOnly
```

#### esbuild Commands

```bash
# Bundle for Node.js
esbuild src/index.ts --bundle --platform=node --target=node20 --outfile=dist/index.js

# Bundle with source maps
esbuild src/index.ts --bundle --platform=node --sourcemap --outfile=dist/index.js

# Watch mode
esbuild src/index.ts --bundle --platform=node --watch --outfile=dist/index.js

# Minify
esbuild src/index.ts --bundle --platform=node --minify --outfile=dist/index.js

# External dependencies (do not bundle node_modules)
esbuild src/index.ts --bundle --platform=node --packages=external --outfile=dist/index.js
```

#### swc Configuration

```json
// .swcrc
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "decorators": true,
      "dynamicImport": true
    },
    "target": "es2022",
    "transform": {
      "legacyDecorator": true,
      "decoratorMetadata": true
    }
  },
  "module": {
    "type": "es6"
  },
  "sourceMaps": true
}
```

#### Build Scripts (package.json)

```json
{
  "scripts": {
    "build": "tsc",
    "build:fast": "esbuild src/index.ts --bundle --platform=node --target=node20 --outfile=dist/index.js",
    "build:types": "tsc --emitDeclarationOnly",
    "typecheck": "tsc --noEmit",
    "check": "npm run typecheck && npm run build:fast"
  }
}
```

#### Syntax Rules

- **Use `tsc` for type checking** — `tsc --noEmit`.
- **Use `esbuild` or `swc` for fast builds** — no type checking.
- **Use `--packages=external`** — for Node.js backends (do not bundle node_modules).
- **Use `--platform=node`** — for Node.js.
- **Use `--target=node20`** — match your Node.js version.
- **Generate source maps** — `--sourcemap`.
- **Separate type checking from bundling** — in CI.
- **Use `--emitDeclarationOnly`** — for library `.d.ts` files.

#### Constraints and Limitations

- **esbuild and swc do not type check** — run `tsc --noEmit` separately.
- **esbuild does not support all TypeScript features** — `const enum`, `emitDecoratorMetadata`.
- **swc supports decorators** — with configuration.
- **Bundling Node.js code can break** — native modules, dynamic requires.
- **Minification can break reflection** — class names, metadata.
- **Build output must be tested** — bugs may be introduced by transpilation.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Fast Build with esbuild + tsc Type Checking

```json
// package.json
{
  "name": "fast-build-app",
  "type": "module",
  "scripts": {
    "build": "npm run typecheck && npm run bundle",
    "bundle": "esbuild src/index.ts --bundle --platform=node --target=node20 --packages=external --sourcemap --outfile=dist/index.js",
    "typecheck": "tsc --noEmit",
    "dev": "tsx watch src/index.ts",
    "start": "node --enable-source-maps dist/index.js"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "esbuild": "^0.23.0",
    "tsx": "^4.0.0",
    "typescript": "^5.5.0"
  }
}
```

```typescript
// src/index.ts
import express from 'express';
import { userRouter } from './routes/user.routes.js';

const app = express();
app.use(express.json());
app.use('/users', userRouter);

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

```bash
# Build output
$ npm run build

> typecheck
> tsc --noEmit

> bundle
> esbuild src/index.ts --bundle --platform=node --target=node20 --packages=external --sourcemap --outfile=dist/index.js

  dist/index.js      234.5kb
  dist/index.js.map  1.2mb

⚡ Done in 45ms
```

**Expected behaviour:** `tsc --noEmit` checks types (fails on errors). `esbuild` bundles the code (fast). Source maps are generated. `node --enable-source-maps` uses the source maps for stack traces.

**Why this works:** Type checking and bundling are separate. `esbuild` is fast because it skips type checking. `--packages=external` keeps `node_modules` unbundled (important for native modules). Source maps enable debugging.

### Real-World Cases

- **Node.js APIs:** `esbuild` for bundling, `tsc` for type checking.
- **Libraries:** `tsc` for `.d.ts` generation, `esbuild` for bundling.
- **Monorepos:** `swc` for fast builds across packages.
- **Frontend:** Vite (esbuild under the hood).

---

## Core Concept 3: Runtime Execution

### Definitions

**Core Definition:** Runtime execution tools run TypeScript code directly in Node.js without a separate build step, enabling fast development iteration.

**Technical Definition:** Runtime execution tools include: **`ts-node`** (TypeScript's official Node.js loader, supports type checking, slow), **`tsx`** (esbuild-based, fast, no type checking), **`bun`** (native TypeScript support), **`deno`** (native TypeScript support), and **Node.js native** (`--experimental-strip-types` in Node.js 22+). Modern practice favors `tsx` for speed and simplicity. For production, compile with `tsc` or a bundler and run plain JavaScript.

**Beginner-Friendly Explanation:** During development, you don't want to wait for a build every time you change a file. Runtime execution tools let you run TypeScript directly — like a live interpreter. `ts-node` is the official one (slow but thorough). `tsx` is the modern one (fast, skips type checking). For production, you compile to JavaScript for maximum performance.

### Purposes

- To run TypeScript directly during development.
- To enable fast iteration without a build step.
- To support watch mode and hot reloading.
- To simplify the development workflow.
- To enable scripting and CLI tools in TypeScript.

### Syntax Rules and Structure

#### tsx (Recommended)

```bash
# Install
npm install --save-dev tsx

# Run a file
tsx src/index.ts

# Watch mode
tsx watch src/index.ts

# Run with Node.js flags
tsx --inspect src/index.ts

# Pass arguments
tsx src/cli.ts --input file.txt
```

#### ts-node

```bash
# Install
npm install --save-dev ts-node

# Run a file
ts-node src/index.ts

# With ESM support
node --loader ts-node/esm src/index.ts

# With transpile-only (faster)
ts-node --transpile-only src/index.ts

# With files
ts-node --files src/index.ts
```

#### Node.js Native (22+)

```bash
# Node.js 22.6+ — experimental type stripping
node --experimental-strip-types src/index.ts

# Node.js 23.6+ — enabled by default
node src/index.ts
```

#### npm Scripts

```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "dev:debug": "tsx watch --inspect src/index.ts",
    "start": "node dist/index.js",
    "start:debug": "node --enable-source-maps --inspect dist/index.js"
  }
}
```

#### Syntax Rules

- **Use `tsx` for development** — fast and simple.
- **Use `ts-node` for legacy projects** — more configuration options.
- **Use `tsx watch`** — for hot reloading.
- **Use `--inspect`** — for debugging.
- **Use `node --experimental-strip-types`** — for Node.js 22+.
- **Do not use `tsx` in production** — compile to JavaScript.
- **Use `--enable-source-maps`** — for production stack traces.

#### Constraints and Limitations

- **`tsx` does not type check** — run `tsc --noEmit` separately.
- **`ts-node` is slow** — especially with type checking.
- **ESM support varies** — `ts-node` requires `--loader`.
- **Native type stripping is experimental** — Node.js 22+.
- **Production should use compiled JavaScript** — for performance and reliability.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Development with tsx and Production with tsc

```json
// package.json
{
  "name": "dev-prod-app",
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node --enable-source-maps dist/index.js",
    "typecheck": "tsc --noEmit",
    "test": "vitest"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "tsx": "^4.0.0",
    "typescript": "^5.5.0",
    "vitest": "^2.0.0"
  }
}
```

```typescript
// src/index.ts
import express from 'express';

const app = express();

app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

```bash
# Development
$ npm run dev
> tsx watch src/index.ts
Server running on port 3000
# File changes trigger automatic restart

# Production build
$ npm run build
$ npm start
> node --enable-source-maps dist/index.js
Server running on port 3000
```

**Expected behaviour:** `npm run dev` starts the server with hot reloading. `npm run build` compiles to `dist/`. `npm start` runs the compiled JavaScript with source map support.

**Why this works:** `tsx` provides fast development iteration. `tsc` produces optimized JavaScript for production. Source maps enable debugging in production.

### Real-World Cases

- **Development:** `tsx watch` for fast iteration.
- **Production:** Compiled JavaScript with `node`.
- **CLI tools:** `tsx` for TypeScript scripts.
- **Testing:** Vitest with native TypeScript support.

---

## Core Concept 4: Source Maps

### Definitions

**Core Definition:** Source maps are JSON files that map compiled JavaScript back to the original TypeScript source, enabling debugging and accurate stack traces.

**Technical Definition:** Source maps (`.js.map` files) contain a `mappings` field (VLQ-encoded positions), `sources` (original files), `sourcesContent` (embedded source), `names` (identifiers), and `file` (generated file). They are generated by `tsc` (`sourceMap: true`), esbuild (`--sourcemap`), and swc (`sourceMaps: true`). Node.js uses source maps with `--enable-source-maps`. Browsers use them automatically via the `//# sourceMappingURL=` comment. For production, `hidden-source-map` generates the map without the comment, and `inline-source-map` embeds it in the file.

**Beginner-Friendly Explanation:** Source maps are like a translation guide. When your compiled JavaScript throws an error, the stack trace points to line 42 of `index.js`. But you wrote the code in `index.ts`, line 87. Source maps tell the debugger "line 42 of `index.js` is actually line 87 of `index.ts`." Without them, debugging production code is nearly impossible.

### Purposes

- To enable debugging of compiled code.
- To provide accurate stack traces in production.
- To map error locations back to source.
- To support editor breakpoints in TypeScript.
- To improve developer experience.

### Syntax Rules and Structure

#### tsc Configuration

```json
{
  "compilerOptions": {
    "sourceMap": true,
    "inlineSourceMap": false,
    "inlineSources": false,
    "declarationMap": true
  }
}
```

| Option | Purpose |
|--------|---------|
| `sourceMap` | Emit `.js.map` files |
| `inlineSourceMap` | Embed source map in `.js` |
| `inlineSources` | Embed source content in `.map` |
| `declarationMap` | Emit `.d.ts.map` for declaration files |

#### esbuild Configuration

```bash
# External source map
esbuild src/index.ts --bundle --sourcemap --outfile=dist/index.js

# Inline source map
esbuild src/index.ts --bundle --sourcemap=inline --outfile=dist/index.js

# Hidden source map (no comment, for production)
esbuild src/index.ts --bundle --sourcemap=external --outfile=dist/index.js
```

#### Node.js Source Map Support

```bash
# Enable source maps
node --enable-source-maps dist/index.js

# Environment variable
NODE_OPTIONS="--enable-source-maps" node dist/index.js
```

#### Syntax Rules

- **Enable `sourceMap: true`** — in `tsconfig.json`.
- **Use `--enable-source-maps`** — in Node.js production.
- **Use `inlineSources: true`** — to embed source in the map.
- **Use `declarationMap: true`** — for library consumers.
- **Use `hidden-source-map`** — for production (no comment).
- **Do not ship source maps publicly** — they reveal your source code.
- **Upload source maps to error trackers** — Sentry, Rollbar, Bugsnag.
- **Use `inline-source-map` for development** — faster loading.

#### Constraints and Limitations

- **Source maps reveal source code** — do not expose them publicly.
- **Larger bundle size** — source maps add size.
- **Performance overhead** — `--enable-source-maps` adds latency.
- **Stack traces may be incomplete** — if source maps are missing.
- **Not all tools support source maps** — check compatibility.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Production Source Maps with Error Tracking

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "sourceMap": true,
    "inlineSources": true,
    "declaration": true,
    "declarationMap": true
  }
}
```

```typescript
// src/index.ts
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.RELEASE_VERSION,
});

function processUser(userId: string): void {
  if (!userId) {
    throw new Error('UserId is required');
  }
  // ...
}

try {
  processUser('');
} catch (err) {
  Sentry.captureException(err);
  console.error(err);
}
```

```bash
# Build with source maps
tsc

# Upload source maps to Sentry
npx @sentry/cli sourcemaps upload --release=$RELEASE_VERSION ./dist

# Delete source maps from the deployed bundle
rm -f dist/*.map

# Run with source map support
node --enable-source-maps dist/index.js
```

**Expected behaviour:** The error's stack trace points to `src/index.ts:12` instead of `dist/index.js:8`. Sentry shows the original TypeScript code.

**Why this works:** `sourceMap: true` generates maps. `inlineSources: true` embeds source content. The Sentry CLI uploads maps for error reporting. The `.map` files are deleted from the deployed bundle to prevent public exposure. `--enable-source-maps` enables stack trace mapping.

### Real-World Cases

- **Production debugging:** Mapping stack traces to source.
- **Error tracking:** Sentry, Rollbar, Bugsnag with source maps.
- **Browser debugging:** Chrome DevTools uses source maps automatically.
- **CI/CD:** Uploading source maps as artifacts.

---

## Core Concept 5: Type Checking

### Definitions

**Core Definition:** Type checking is the process of validating TypeScript code against its type annotations, typically run separately from bundling for performance.

**Technical Definition:** `tsc --noEmit` runs the TypeScript compiler in type-checking mode without emitting JavaScript. This separates type checking (slow, thorough) from transpilation (fast, superficial). In CI, type checking runs as a separate step; in development, it runs in watch mode alongside a fast transpiler. `--noEmit` is the recommended flag for type checking. Type checking catches errors that transpilers (esbuild, swc) miss.

**Beginner-Friendly Explanation:** Type checking is like a spell-checker. It reads your code and tells you about mistakes — typos, wrong types, missing properties — without changing anything. Bundling is like printing the document. You can run the spell-checker (type checking) and the printer (bundling) separately, which is faster than doing both at once.

### Purposes

- To catch type errors before runtime.
- To enforce type contracts across the codebase.
- To run as a fast CI check.
- To separate concerns (type checking vs. bundling).
- To enable incremental type checking.

### Syntax Rules and Structure

#### Type Checking Commands

```bash
# Type check only (no emit)
tsc --noEmit

# Type check with watch mode
tsc --noEmit --watch

# Type check a specific project
tsc --noEmit --project tsconfig.build.json

# Incremental type checking
tsc --noEmit --incremental
```

#### CI Configuration

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - name: Type check
        run: npm run typecheck
      - name: Lint
        run: npm run lint
      - name: Test
        run: npm test
      - name: Build
        run: npm run build
```

#### npm Scripts

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "typecheck:watch": "tsc --noEmit --watch",
    "typecheck:incremental": "tsc --noEmit --incremental",
    "build": "tsc",
    "check": "npm run typecheck && npm run lint && npm test"
  }
}
```

#### Syntax Rules

- **Use `tsc --noEmit`** — for type checking.
- **Run type checking in CI** — as a separate step.
- **Use `--incremental`** — for faster rebuilds.
- **Use `--watch`** — during development.
- **Separate type checking from bundling** — for performance.
- **Fail CI on type errors** — enforce type safety.
- **Use `tsc --noEmit` with esbuild/swc** — they do not type check.

#### Constraints and Limitations

- **Type checking is slow** — especially on large codebases.
- **`--noEmit` still reads all files** — may be slow.
- **Incremental builds require cache** — `.tsbuildinfo` file.
- **Some errors are only caught at runtime** — type checking is not exhaustive.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Type Checking with Fast Build

```json
// package.json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "npm run typecheck && npm run bundle",
    "bundle": "esbuild src/index.ts --bundle --platform=node --packages=external --outfile=dist/index.js",
    "typecheck": "tsc --noEmit",
    "typecheck:watch": "tsc --noEmit --watch",
    "test": "vitest run",
    "ci": "npm run typecheck && npm run lint && npm run test && npm run build"
  }
}
```

```typescript
// src/index.ts
interface User {
  id: string;
  name: string;
  email: string;
}

function createUser(data: Omit<User, 'id'>): User {
  return {
    id: crypto.randomUUID(),
    ...data,
  };
}

// Type checking catches this error:
// const user = createUser({ name: 'Alice' }); // Error: missing email

const user = createUser({ name: 'Alice', email: 'alice@example.com' });
console.log(user);
```

**Expected behaviour:** `npm run typecheck` catches the missing `email` error without emitting JavaScript. `npm run bundle` builds the code quickly with esbuild. The `ci` script runs all checks.

**Why this works:** Type checking is separated from bundling. `tsc --noEmit` is fast because it does not write files. esbuild is fast because it skips type checking. Together, they provide both speed and safety.

### Real-World Cases

- **CI pipelines:** Type checking as a separate job.
- **Pre-commit hooks:** Husky + lint-staged with `tsc --noEmit`.
- **Monorepos:** Project references with incremental type checking.
- **Large codebases:** Distributed type checking with Turborepo.

---

## Core Concept 6: ESLint Integration

### Definitions

**Core Definition:** ESLint integration for TypeScript uses `@typescript-eslint/parser` and `@typescript-eslint/eslint-plugin` to lint TypeScript code with type-aware rules.

**Technical Definition:** `@typescript-eslint` is a monorepo that includes `parser` (parses TypeScript into an ESTree-compatible AST), `eslint-plugin` (TypeScript-specific rules), `typescript-estree` (parser infrastructure), and `utils` (helpers for rules). Type-aware rules (e.g., `no-floating-promises`, `no-misused-promises`, `await-thenable`) require the `project` option in `parserOptions`. ESLint 9 uses flat config (`eslint.config.js`); ESLint 8 uses `.eslintrc.json`. Best practice: strict type-aware rules, Prettier integration, and CI enforcement.

**Beginner-Friendly Explanation:** ESLint is a code quality checker. It reads your code and points out problems — unused variables, missing `await`, wrong types, style issues. The `@typescript-eslint` plugin teaches ESLint how to understand TypeScript. Type-aware rules are especially powerful: they know the types of your variables and can catch subtle bugs like "you forgot to `await` this Promise."

### Purposes

- To enforce code quality and consistency.
- To catch bugs that TypeScript misses.
- To enforce style rules.
- To prevent common mistakes.
- To integrate with editors for real-time feedback.

### Syntax Rules and Structure

#### ESLint 9 Flat Config (Recommended)

```javascript
// eslint.config.js
import eslint from '@eslint/js';
import tseslint from 'typescript-eslint';
import prettier from 'eslint-config-prettier';

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  ...tseslint.configs.stylisticTypeChecked,
  prettier,
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/consistent-type-imports': ['error', { prefer: 'type-imports' }],
      '@typescript-eslint/no-floating-promises': 'error',
      '@typescript-eslint/no-misused-promises': 'error',
      '@typescript-eslint/await-thenable': 'error',
      '@typescript-eslint/require-await': 'error',
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/explicit-function-return-type': ['warn', {
        allowExpressions: true,
        allowTypedFunctionExpressions: true,
      }],
      '@typescript-eslint/naming-convention': [
        'error',
        { selector: 'interface', format: ['PascalCase'], prefix: ['I'] },
        { selector: 'typeAlias', format: ['PascalCase'] },
        { selector: 'enum', format: ['PascalCase'] },
      ],
      'no-console': ['warn', { allow: ['warn', 'error'] }],
      'eqeqeq': ['error', 'always'],
      'curly': ['error', 'all'],
    },
  },
  {
    ignores: ['dist/**', 'node_modules/**', 'coverage/**', '*.config.js'],
  },
);
```

#### ESLint 8 Legacy Config

```json
// .eslintrc.json
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": 2022,
    "sourceType": "module",
    "project": "./tsconfig.json",
    "tsconfigRootDir": "."
  },
  "plugins": ["@typescript-eslint"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/strict-type-checked",
    "plugin:@typescript-eslint/stylistic-type-checked",
    "prettier"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    "@typescript-eslint/no-floating-promises": "error",
    "@typescript-eslint/no-misused-promises": "error",
    "@typescript-eslint/no-explicit-any": "error"
  },
  "ignorePatterns": ["dist", "node_modules", "coverage", "*.config.js"]
}
```

#### npm Scripts

```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "lint:ci": "eslint . --max-warnings=0",
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  }
}
```

#### Syntax Rules

- **Use `@typescript-eslint/parser`** — to parse TypeScript.
- **Use `projectService: true`** — for type-aware rules (ESLint 9).
- **Use `strictTypeChecked`** — for strict rules.
- **Use `consistent-type-imports`** — to enforce `import type`.
- **Use `no-floating-promises`** — to catch missing `await`.
- **Use `no-misused-promises`** — to catch Promise misuse.
- **Use `no-explicit-any`** — to discourage `any`.
- **Integrate Prettier** — for formatting.
- **Run in CI with `--max-warnings=0`** — fail on warnings.
- **Configure editor integration** — for real-time feedback.

#### Constraints and Limitations

- **Type-aware rules are slow** — they require type information.
- **`projectService` requires `tsconfig.json`** — must be configured.
- **Some rules have false positives** — review carefully.
- **Prettier and ESLint may conflict** — use `eslint-config-prettier`.
- **Flat config is new** — some plugins do not support it yet.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full ESLint Setup for Node.js TypeScript

```javascript
// eslint.config.js
import eslint from '@eslint/js';
import tseslint from 'typescript-eslint';
import prettier from 'eslint-config-prettier';

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  ...tseslint.configs.stylisticTypeChecked,
  prettier,
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/consistent-type-imports': ['error', { prefer: 'type-imports' }],
      '@typescript-eslint/no-floating-promises': 'error',
      '@typescript-eslint/no-misused-promises': 'error',
      '@typescript-eslint/await-thenable': 'error',
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/prefer-nullish-coalescing': 'error',
      '@typescript-eslint/prefer-optional-chain': 'error',
    },
  },
  {
    files: ['**/*.test.ts', '**/*.spec.ts'],
    rules: {
      '@typescript-eslint/no-explicit-any': 'off',
      '@typescript-eslint/no-non-null-assertion': 'off',
    },
  },
  {
    ignores: ['dist/**', 'node_modules/**', 'coverage/**'],
  },
);
```

```typescript
// src/index.ts
// ESLint catches these issues:

// ❌ no-floating-promises
async function fetchUser(id: string): Promise<User> {
  return await fetch(`/api/users/${id}`).then((r) => r.json());
}
fetchUser('user-1'); // Error: Promise not awaited

// ✅ Fixed
await fetchUser('user-1');

// ❌ consistent-type-imports
import { User } from './types.js'; // Error: use `import type`
import type { User } from './types.js'; // ✅

// ❌ no-explicit-any
function process(data: any): void {} // Error: use `unknown`

// ✅ Fixed
function process(data: unknown): void {
  if (typeof data === 'string') {
    console.log(data.toUpperCase());
  }
}

// ❌ prefer-nullish-coalescing
const value = input || 'default'; // Error: use ??

// ✅ Fixed
const value = input ?? 'default';
```

**Expected behaviour:** ESLint catches missing `await`, explicit `any`, missing `type` imports, and other issues. `npm run lint:fix` auto-fixes many of them. CI fails on any warning.

**Why this works:** `strictTypeChecked` enables strict rules. `projectService` provides type information. Custom rules enforce project conventions. Test files have relaxed rules.

### Real-World Cases

- **CI enforcement:** `eslint --max-warnings=0` fails on any warning.
- **Pre-commit hooks:** Husky + lint-staged runs ESLint on changed files.
- **Editor integration:** VS Code shows errors in real time.
- **Monorepos:** Shared ESLint config across packages.

---

## Core Concept 7: ESM Module Resolution Configuration

### Definitions

**Core Definition:** ESM module resolution configuration determines how TypeScript resolves `import` statements — handling `.js` extensions in ESM, `package.json` `exports` fields, and the differences between Node.js and bundler resolution.

**Technical Definition:** `moduleResolution` options include: **`node10`** (legacy CommonJS resolution, uses `main` and `index.js`), **`node16`** (Node.js 16+ ESM resolution, uses `exports` and requires `.js` extensions), **`nodenext`** (alias for the latest Node.js resolution), and **`bundler`** (for bundlers — allows extensionless imports, uses `exports` and `imports`). With `NodeNext`, relative imports must include `.js` extensions in TypeScript source (`import { x } from './x.js'`), which map to `.ts` files. The `exports` field in `package.json` controls what is exported and how it is resolved. `verbatimModuleSyntax` enforces explicit `import type`.

**Beginner-Friendly Explanation:** When you write `import { x } from './x'`, how does TypeScript find `./x`? The `moduleResolution` setting answers this. Node.js requires `.js` extensions in ESM imports (`import { x } from './x.js'`). Bundlers allow extensionless imports. The `exports` field in `package.json` defines what a package exports and how. Getting this right is essential for ESM projects.

### Purposes

- To resolve `import` statements correctly.
- To support ESM in Node.js.
- To configure `package.json` `exports`.
- To enable bundler compatibility.
- To enforce explicit type imports.

### Syntax Rules and Structure

#### moduleResolution Options

| Option | Use Case | Extensions Required |
|--------|----------|---------------------|
| `node10` | Legacy CommonJS | No |
| `node16` | Node.js 16+ ESM | Yes (`.js`) |
| `nodenext` | Latest Node.js ESM | Yes (`.js`) |
| `bundler` | Vite, esbuild, webpack | No |

#### Node.js ESM Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "verbatimModuleSyntax": true
  }
}
```

```json
// package.json
{
  "name": "my-app",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    },
    "./utils": {
      "types": "./dist/utils.d.ts",
      "import": "./dist/utils.js"
    }
  }
}
```

```typescript
// src/index.ts — .js extension required
import { greet } from './greet.js';
import type { User } from './types.js';

export { greet };
export type { User };
```

#### Bundler Configuration

```json
// tsconfig.json (Vite, esbuild)
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "verbatimModuleSyntax": true,
    "noEmit": true
  }
}
```

```typescript
// src/index.ts — extensionless imports OK
import { greet } from './greet';
import type { User } from './types';

export { greet };
```

#### CommonJS Configuration

```json
// tsconfig.json (legacy)
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "moduleResolution": "Node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true
  }
}
```

```typescript
// src/index.ts — extensionless imports
import { greet } from './greet';
import { User } from './types';

export { greet };
export type { User };
```

#### Syntax Rules

- **Use `module: "NodeNext"` and `moduleResolution: "NodeNext"`** — for Node.js ESM.
- **Use `.js` extensions** — in relative imports with `NodeNext`.
- **Use `moduleResolution: "Bundler"`** — for bundler projects.
- **Use `verbatimModuleSyntax: true`** — enforce `import type`.
- **Configure `exports` in `package.json`** — for published packages.
- **Use `type: "module"`** — to enable ESM in Node.js.
- **Use `esModuleInterop: true`** — for CommonJS interop.
- **Do not mix `require` and `import`** — use one system.

#### Constraints and Limitations

- **`.js` extensions in TypeScript source are confusing** — they map to `.ts` files.
- **`NodeNext` is strict** — requires correct configuration.
- **`Bundler` is not for Node.js** — use `NodeNext` for backends.
- **`exports` is complex** — condition ordering matters.
- **CommonJS and ESM interop is tricky** — `esModuleInterop` helps.
- **Some libraries do not support ESM** — may require CommonJS.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Node.js ESM Package with Exports

```json
// package.json
{
  "name": "@myorg/my-library",
  "version": "1.0.0",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./utils": {
      "types": "./dist/utils.d.ts",
      "import": "./dist/utils.js",
      "require": "./dist/utils.cjs"
    },
    "./package.json": "./package.json"
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsc && tsc --project tsconfig.cjs.json",
    "typecheck": "tsc --noEmit"
  }
}
```

```json
// tsconfig.json (ESM build)
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "strict": true,
    "verbatimModuleSyntax": true
  },
  "include": ["src/**/*"]
}
```

```json
// tsconfig.cjs.json (CommonJS build)
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "module": "CommonJS",
    "moduleResolution": "Node",
    "outDir": "./dist",
    "verbatimModuleSyntax": false
  }
}
```

```typescript
// src/index.ts
export { greet } from './greet.js';
export type { User } from './types.js';
export { createUser } from './user.js';
```

```typescript
// src/greet.ts
import type { User } from './types.js';

export function greet(user: User): string {
  return `Hello, ${user.name}!`;
}
```

```typescript
// src/types.ts
export interface User {
  id: string;
  name: string;
  email: string;
}
```

```typescript
// Consumer code
import { greet } from '@myorg/my-library';
import type { User } from '@myorg/my-library';

const user: User = { id: '1', name: 'Alice', email: 'alice@example.com' };
console.log(greet(user)); // 'Hello, Alice!'
```

**Expected behaviour:** The package works with both ESM (`import`) and CommonJS (`require`) consumers. TypeScript resolves types via `exports.types`. The `.js` extensions in source map to `.ts` files during compilation.

**Why this works:** `module: "NodeNext"` matches Node.js ESM resolution. The `exports` field defines entry points for both module systems. Dual builds (ESM + CommonJS) support both consumer types. `verbatimModuleSyntax` enforces explicit type imports.

### Real-World Cases

- **npm libraries:** Dual ESM/CJS packages with `exports`.
- **Monorepos:** Shared packages with `workspace:*` dependencies.
- **Node.js APIs:** ESM with `NodeNext` resolution.
- **Frontend:** Bundler resolution with Vite.

---

## References

- TypeScript Documentation — tsconfig Reference — https://www.typescriptlang.org/tsconfig
- TypeScript Documentation — Module Resolution — https://www.typescriptlang.org/docs/handbook/modules/theory.html
- TypeScript Documentation — Modules Reference — https://www.typescriptlang.org/docs/handbook/modules/reference.html
- TypeScript Documentation — `verbatimModuleSyntax` — https://www.typescriptlang.org/tsconfig#verbatimModuleSyntax
- TypeScript Documentation — `moduleResolution` — https://www.typescriptlang.org/tsconfig#moduleResolution
- TypeScript Documentation — `isolatedModules` — https://www.typescriptlang.org/tsconfig#isolatedModules
- TypeScript Documentation — Source Maps — https://www.typescriptlang.org/tsconfig#sourceMap
- TypeScript Documentation — Project References — https://www.typescriptlang.org/docs/handbook/project-references.html
- esbuild Documentation — https://esbuild.github.io/
- esbuild — TypeScript Content Types — https://esbuild.github.io/content-types/#typescript
- swc Documentation — https://swc.rs/docs/getting-started
- tsx Documentation — https://tsx.is/
- ts-node Documentation — https://typestrong.org/ts-node/
- Node.js Documentation — Modules: Packages — https://nodejs.org/api/packages.html
- Node.js Documentation — `--enable-source-maps` — https://nodejs.org/api/cli.html#--enable-source-maps
- Node.js Documentation — TypeScript Support — https://nodejs.org/api/typescript.html
- ESLint Documentation — https://eslint.org/docs/latest/
- typescript-eslint Documentation — https://typescript-eslint.io/
- typescript-eslint — Getting Started — https://typescript-eslint.io/getting-started/
- typescript-eslint — Typed Linting — https://typescript-eslint.io/getting-started/typed-linting/
- typescript-eslint — Rules — https://typescript-eslint.io/rules/
- Prettier Documentation — https://prettier.io/docs/en/
- eslint-config-prettier — https://github.com/prettier/eslint-config-prettier
- Vitest Documentation — https://vitest.dev/
- Turborepo Documentation — https://turbo.build/repo/docs
- Sentry — Source Maps — https://docs.sentry.io/platforms/node/sourcemaps/
- tsconfig bases — https://github.com/tsconfig/bases
- Total TypeScript — https://www.totaltypescript.com/