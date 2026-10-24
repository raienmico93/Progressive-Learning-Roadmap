# DefinitelyTyped and `@types` Type Packages — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **DefinitelyTyped** is a community-maintained repository of high-quality TypeScript **declaration files** (`.d.ts`) for JavaScript packages that do not bundle their own types. These declarations are automatically published to the **`@types` scope on npm**, allowing TypeScript users to install type definitions for untyped libraries using standard npm commands. The `@types` scope is the **canonical distribution channel** for community-authored ambient and module declarations.

**Technical Definition.** DefinitelyTyped is a **pnpm monorepo** whose packages are automatically published to npm under the `@types/` scope via the `types-publisher` tool. A type package named `@types/foo` provides declarations for the npm package `foo`. For **scoped packages**, the scope separator is replaced with a double underscore: `@types/scope__name`. When a package already includes its own declarations (via the `types` or `typings` field in `package.json`), installing the corresponding `@types` package is **unnecessary and may conflict**. TypeScript resolves `@types` packages automatically from `node_modules/@types` unless restricted by the `types` or `typesVersions` compiler options.

**Beginner-Friendly Explanation.** DefinitelyTyped is **a giant community library of type descriptions** for JavaScript packages that do not include their own TypeScript types. When you install a package like `lodash`, it does not know about TypeScript types. You run `npm install --save-dev @types/lodash`, and TypeScript automatically finds the type descriptions so you get autocomplete and type checking. **If the package already ships its own types, you do not need an `@types` package**—installing one anyway can cause conflicts.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Community-maintained** | Thousands of contributors maintain declarations for the JavaScript ecosystem. |
| **Auto-published to `@types`** | The `master` branch is automatically published to the npm `@types` scope. |
| **Dev dependencies** | Type packages are installed with `--save-dev` because they are compile-time only. |
| **Automatic inclusion** | TypeScript includes `node_modules/@types` by default; no manual import is needed. |
| **Scoped naming convention** | `@types/scope__name` for scoped packages like `@babel/preset-env`. |
| **Two-year support window** | DefinitelyTyped tests against TypeScript versions less than **2 years old**. |
| **Optional but recommended** | If a package bundles its own `.d.ts` files, no `@types` package is needed. |

---

### Prerequisites

- **TypeScript basics**: modules, declaration files, and `tsconfig.json`.
- **npm/yarn/pnpm**: installing dependencies and managing `package.json`.
- **Node.js module resolution**: `node_modules` and the `@types` folder.
- **Declaration file fundamentals**: `declare module`, `declare namespace`, and ambient declarations.

---

### Related Programming Areas

- **Declaration files** (`.d.ts`) — the content of `@types` packages.
- **Module augmentation** and **global augmentation** — extending third-party types.
- **`typeAcquisition`** — automatic type acquisition for JavaScript projects.
- **Publishing types** — bundling types vs. submitting to DefinitelyTyped.
- **`typesVersions`** — version-specific type resolution.

---

## Core Concepts / Features

---

## 1. DefinitelyTyped

### Definitions

**Core Definition.** **DefinitelyTyped** is a repository of community-authored TypeScript declaration files for JavaScript packages, automatically published to the `@types` npm scope.

**Technical Definition.** DefinitelyTyped is a **pnpm monorepo** of type packages. Each package under `types/` corresponds to an npm package and is published as `@types/<name>`. The repository is the primary source of ambient and module declarations for the JavaScript ecosystem. Contributions are made via pull requests and reviewed by maintainers.

**Beginner-Friendly Explanation.** DefinitelyTyped is **the place where the TypeScript community writes type definitions for JavaScript libraries**. If a library does not have types, someone has probably written them on DefinitelyTyped.

### Purposes

- **To provide type definitions for untyped JavaScript packages**.
- **To centralize community type maintenance** in a single, reviewable repository.
- **To automatically publish type packages** to the `@types` scope.
- **To enable TypeScript users to adopt untyped libraries safely**.
- **To support the two-year TypeScript version support window**.

### Syntax Rules and Structure

**Installing from DefinitelyTyped:**

```bash
npm install --save-dev @types/lodash
```

**Contributing (high-level):**

1. Fork the repository.
2. Add or modify declarations under `types/<package-name>/`.
3. Add tests in `types/<package-name>/<package-name>-tests.ts`.
4. Run the type checker and linter.
5. Submit a pull request.

**Syntax rules.**

- Package names must match the npm package name, prefixed with `@types/`.
- Scoped packages use `@types/scope__name`.
- Declarations are automatically published from `master`.
- The support window covers TypeScript versions **less than 2 years old**.

**Constraints and limitations.**

- DefinitelyTyped does not accept declarations for packages that already bundle their own types.
- Pull requests are reviewed by maintainers and may take time.
- Very old TypeScript versions are supported only through version-specific dist-tags.

### Annotated Complete Step by Step Code Examples

#### Example 1: Installing and using `@types/lodash`

```bash
npm install --save-dev @types/lodash
```

```typescript
// main.ts
import * as _ from "lodash";

const padded = _.padStart("Hello TypeScript!", 20, " ");
console.log(padded); // Expected: "     Hello TypeScript!"
```

**Setup.** Run `npm install --save-dev typescript @types/lodash` and compile with `npx tsc main.ts --strict`.

**Expected Output:**
```
     Hello TypeScript!
```

**Why this output occurs.** TypeScript automatically includes `node_modules/@types/lodash`, so `_.padStart` is typed and checked. No manual import of the declaration file is needed.

---

### Real-World Cases

- **`@types/node`** — Node.js APIs.
- **`@types/react`** — React APIs.
- **`@types/express`** — Express middleware and request types.
- **`@types/jest`** — Jest globals and matchers.

---

## 2. `@types`

### Definitions

**Core Definition.** The **`@types` scope** is the npm namespace where DefinitelyTyped publishes type declaration packages. A package named `@types/foo` provides declarations for the package `foo`.

**Technical Definition.** `@types` packages are **type-only npm packages**. TypeScript automatically includes declarations from `node_modules/@types` unless restricted by the `types` compiler option. Scoped packages use the `@types/scope__name` convention because npm scope names cannot contain slashes in a single package name.

**Beginner-Friendly Explanation.** `@types` is **the special npm scope for type definitions**. When you install `@types/foo`, TypeScript finds it automatically and gives you types for `foo`.

### Purposes

- **To distribute type declarations** through the standard npm ecosystem.
- **To enable automatic type inclusion** without manual references.
- **To provide a consistent naming convention** for type packages.
- **To support scoped package declarations** with the double-underscore convention.

### Syntax Rules and Structure

**Basic installation:**

```bash
npm install --save-dev @types/package-name
```

**Scoped package installation:**

```bash
# For @babel/preset-env
npm install --save-dev @types/babel__preset-env
```

**Manual reference (when not using modules):**

```typescript
/// <reference types="node" />
```

**Syntax rules.**

- The `@types/` prefix maps to the package name.
- Scoped packages replace `@scope/` with `@types/scope__`.
- TypeScript includes `node_modules/@types` automatically.
- `types` in `tsconfig.json` restricts which `@types` packages are included.

**Constraints and limitations.**

- Do not install `@types/foo` if `foo` already bundles its own types.
- The `types` compiler option, when set, **excludes all other `@types` packages**.
- Type packages should be in `devDependencies`, not `dependencies`, for applications.

### Annotated Complete Step by Step Code Examples

#### Example 1: Scoped package installation

```bash
npm install --save-dev @types/babel__preset-env
```

```typescript
// main.ts
import presetEnv from "@babel/preset-env";
console.log(presetEnv); // Expected: [Function: presetEnv]
```

**Expected Output:**
```
[Function: presetEnv]
```

**Why this output occurs.** The `@types/babel__preset-env` package provides declarations for `@babel/preset-env`. TypeScript automatically includes the types from `node_modules/@types/babel__preset-env`.

---

#### Example 2: Triple-slash reference for global types

```typescript
// main.ts
/// <reference types="node" />

console.log(process.version); // Expected: v20.x.x
```

**Expected Output (approximate):**
```
v20.11.0
```

**Why this output occurs.** The triple-slash reference tells TypeScript to include the `@types/node` declarations, making the global `process` object available.

---

### Real-World Cases

- **Global type packages**: `@types/node`, `@types/jest` for global APIs.
- **Module type packages**: `@types/lodash`, `@types/express`.
- **Scoped package types**: `@types/babel__core`, `@types/react-dom`.

---

## 3. Type Packages

### Definitions

**Core Definition.** A **type package** is an npm package that contains only TypeScript declaration files (`.d.ts`) and metadata, providing types for another package without runtime code.

**Technical Definition.** Type packages are published either as `@types/<name>` (community-authored on DefinitelyTyped) or as part of a package that bundles its own declarations (`types` field in `package.json`). They contain `.d.ts` files, a `package.json`, and optionally tests. They produce **no JavaScript** and are used only by the compiler.

**Beginner-Friendly Explanation.** A type package is **a package that contains only type descriptions**. It tells TypeScript about another package’s API. It has no runtime code.

### Purposes

- **To supply types for untyped JavaScript packages**.
- **To separate type maintenance from runtime code**.
- **To enable versioned type definitions** independent of the package version.
- **To support automatic type acquisition** in JavaScript projects.

### Syntax Rules and Structure

**Typical type package structure:**

```
node_modules/@types/lodash/
├── index.d.ts
├── package.json
└── ...
```

**Syntax rules.**

- Type packages contain `.d.ts` files and a `package.json`.
- The `types` field in `package.json` points to the main declaration file.
- Type packages produce no runtime output.

**Constraints and limitations.**

- Installing a type package for a library that already bundles types can cause duplicate or conflicting declarations.
- Type packages must be kept in sync with the runtime package’s API.

### Annotated Complete Step by Step Code Examples

#### Example 1: Inspecting a type package

```bash
ls node_modules/@types/lodash
# index.d.ts  package.json  common/
```

```typescript
// main.ts
import * as _ from "lodash";
const result: number = _.add(1, 2);
console.log(result); // Expected: 3
```

**Expected Output:**
```
3
```

**Why this output occurs.** The `@types/lodash` package contains declarations for `_.add`, allowing TypeScript to infer the return type as `number`.

---

### Real-World Cases

- **`@types/node`** — large ambient type package for Node.js.
- **`@types/react`** — module and namespace declarations for React.
- **Bundled types** — packages like `typescript` and `zod` ship their own declarations.

---

## 4. Type Package Installation

### Definitions

**Core Definition.** **Type package installation** is the process of adding an `@types` package (or a package that bundles types) to a project’s dependencies so the compiler can use its declarations.

**Technical Definition.** Type packages are installed with npm, yarn, or pnpm as **dev dependencies**. TypeScript automatically includes `node_modules/@types` declarations. The `types` compiler option can restrict inclusion. For JavaScript projects, `typeAcquisition` can automatically acquire types from DefinitelyTyped.

**Beginner-Friendly Explanation.** Type package installation means **running `npm install --save-dev @types/foo`** so TypeScript can find the types for `foo`.

### Purposes

- **To make types available to the compiler** for untyped packages.
- **To enable autocomplete and type checking** in editors.
- **To keep type dependencies explicit** in `package.json`.
- **To support automatic type acquisition** in JavaScript projects.

### Syntax Rules and Structure

**Installation commands:**

```bash
npm install --save-dev @types/lodash
npm install --save-dev @types/babel__preset-env
```

**`tsconfig.json` restriction:**

```json
{
  "compilerOptions": {
    "types": ["node", "jest"]
  }
}
```

**Automatic type acquisition (`jsconfig.json`):**

```json
{
  "typeAcquisition": {
    "enable": true,
    "include": ["jest"],
    "exclude": ["jquery"]
  }
}
```

**Syntax rules.**

- Use `--save-dev` for type packages in applications.
- Scoped packages use `@types/scope__name`.
- `types` in `tsconfig.json` restricts which `@types` packages are included.
- `typeAcquisition` controls automatic type acquisition for JavaScript projects.

**Constraints and limitations.**

- The `types` option, when set, excludes all other `@types` packages.
- Installing an `@types` package for a library that bundles its own types may cause conflicts.
- Type packages should match the runtime package version.

### Annotated Complete Step by Step Code Examples

#### Example 1: Restricted `types` array

```json
// tsconfig.json
{
  "compilerOptions": {
    "types": ["node"]
  }
}
```

```typescript
// main.ts
console.log(process.version); // Expected: v20.x.x

// console.log(_.padStart("x", 3)); // Error: lodash types not included.
```

**Expected Output (approximate):**
```
v20.11.0
```

**Why this output occurs.** The `types` array includes only `node`, so `@types/lodash` is excluded. Lodash is not available without an explicit import.

---

#### Example 2: Automatic type acquisition

```json
// jsconfig.json
{
  "typeAcquisition": {
    "include": ["jest"]
  }
}
```

**Expected behavior.** The editor automatically acquires Jest types for a JavaScript project.

---

### Real-World Cases

- **Node.js projects**: `npm install --save-dev @types/node`.
- **React projects**: `npm install --save-dev @types/react @types/react-dom`.
- **Restricting ambient types**: `"types": ["node"]` to exclude accidental globals.

---

## 5. DefinitelyTyped Best Practices

### Definitions

**Core Definition.** **DefinitelyTyped best practices** are guidelines for writing high-quality declaration files that accurately model JavaScript APIs, avoid common pitfalls, and remain maintainable.

**Technical Definition.** Best practices cover **ghost modules** (type-only namespaces), **extending built-in types**, **getter/setter overloads**, **fluent interfaces**, **callback signatures**, **function overloading**, and **overload ordering**. They are documented on DefinitelyTyped and the TypeScript Handbook.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced type authors use** to write declarations that are accurate, safe, and easy to maintain.

### Purposes

- **To model JavaScript APIs accurately** without over- or under-constraining.
- **To avoid global namespace pollution**.
- **To provide precise callback and overload types**.
- **To keep declarations maintainable and testable**.

### Syntax Rules and Structure

Best practices are conventions that interact with declaration syntax. The core rules are:

1. **Use ghost modules (type-only namespaces) to avoid global pollution.** Group interfaces inside a `declare namespace` without creating a runtime variable.
2. **Extend built-in types via interfaces.** Add members to `Date`, `Array`, etc., by declaring an interface with the same name.
3. **Use overloads for getters and setters.** Instead of `declare function duration(value?: number): any`, use `declare function duration(): number; declare function duration(value: number): void;`.
4. **Model fluent interfaces by returning the interface type.** `foo(): Something; bar(): Something;`.
5. **Do not mark callback arguments as optional if they are always passed.** Use `(bar: any, bas: any) => any`, not optional parameters.
6. **Use function overloading for functions and constructors.** Reserve union types for config object bags.
7. **Order overloads from most specific to most general.**
8. **Test declarations** with `tsc` and the DefinitelyTyped test harness before submitting.

**Component breakdown (declaration pattern decision tree):**

```
Is the library global or modular?
  ├── Global → Use `declare namespace` (ghost module) or global declarations.
  └── Modular → Use `declare module "name" { ... }`.
       ├── Does it have overloads? → Order most specific first.
       ├── Does it have callbacks? → Do not mark always-passed args optional.
       └── Does it extend built-ins? → Use interface merging.
```

**Constraints and limitations.**

- Ghost modules cannot be used as values at runtime.
- Extending built-in types can conflict with other declarations.
- Overload ordering affects resolution; incorrect order can make overloads unreachable.

### Annotated Complete Step by Step Code Examples

#### Example 1: Ghost module

```typescript
// types/foo/index.d.ts
declare namespace NodeJS {
  interface Foo {
    id: string;
  }
}

declare module "foo" {
  var foo: NodeJS.Foo;
  export = foo;
}
```

**Why this is good.** Interfaces are grouped under `NodeJS` without creating a runtime global. The module declaration uses the namespace type.

---

#### Example 2: Getter/setter overloads

```typescript
// ❌ Vague.
// declare function duration(value?: number): any;

// ✅ Precise overloads.
declare function duration(): number;
declare function duration(value: number): void;
```

**Why this is good.** The overloads distinguish reading (`duration()`) from writing (`duration(5)`), providing accurate types.

---

#### Example 3: Callback signatures

```typescript
// ❌ Callback arguments marked optional incorrectly.
// declare function foo(callback: (bar?: any, bas?: any) => void);

// ✅ Correct: arguments are always passed.
declare function foo(callback: (bar: any, bas: any) => any): void;
```

**Why this is good.** The callback is always called with both arguments, so marking them optional would incorrectly force callers to handle `undefined`.

---

### Real-World Cases

- **Contributing to DefinitelyTyped**: following the contribution guide and best practices.
- **Writing local declarations**: applying the same patterns in project `types/` folders.
- **Reviewing type PRs**: checking for ghost modules, overload ordering, and callback accuracy.

---

## References

- TypeScript Handbook — Consumption (Installing `@types`) - https://www.typescriptlang.org/docs/handbook/declaration-files/consumption.html
- TypeScript Handbook — Publishing (Bundling vs. DefinitelyTyped) - https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html
- TSConfig Reference — `typeAcquisition` - https://www.typescriptlang.org/tsconfig/typeAcquisition.html
- DefinitelyTyped — Home (`@types` Usage) - https://definitelytyped.org/
- DefinitelyTyped — Best Practices - https://definitelytyped.org/guides/best-practices.html
- DefinitelyTyped — Contribution Guide - https://definitelytyped.org/guides/contributing.html
- DefinitelyTyped — Creating a Definition File - https://definitelytyped.org/guides/creating-a-definition-file.html
- GitHub — DefinitelyTyped README (Installation, Scoped Packages, Support Window) - https://github.com/DefinitelyTyped/DefinitelyTyped
- TypeScript Handbook — Library Structures - https://www.typescriptlang.org/docs/handbook/declaration-files/library-structures.html
- TypeScript Handbook — Do’s and Don’ts - https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html