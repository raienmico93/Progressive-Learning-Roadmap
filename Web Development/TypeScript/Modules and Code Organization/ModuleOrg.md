# TypeScript Module Organization: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Module organization in TypeScript is the practice of structuring code into files, directories, and packages so that dependencies are explicit, boundaries are clear, and the codebase remains maintainable as it grows. It encompasses decisions about file layout (by type vs. by feature), export strategies (barrel files, public API surfaces), and cross-package sharing (monorepos).

**Technical Definition**
TypeScript module organization operates on three levels: (1) **file-level modules**, where each file with a top-level `import` or `export` is its own scope; (2) **directory-level organization**, where related modules are grouped by feature, domain, or technical concern; and (3) **package-level architecture**, where multiple packages share code through workspace resolution (npm/pnpm/yarn workspaces) and TypeScript project references. The compiler's `moduleResolution` setting, `paths` aliases, and `exports` maps control how import specifiers resolve to files on disk. Effective organization minimizes the public API surface of each module, enforces unidirectional dependency flow (shared → features → app), and avoids circular dependencies that arise from barrel files and re-export chains.

**Beginner-Friendly Explanation**
Module organization is how you decide where to put your code files and how they're allowed to talk to each other. In small projects, it doesn't matter much—you can throw everything in a few folders. But as projects grow, bad organization causes circular imports, unclear boundaries, and code that's hard to change. The two main approaches are: **file-type-based** (all components together, all hooks together, all services together) and **feature-based** (everything for "auth" together, everything for "dashboard" together). Feature-based scales better because changing one feature means editing one folder. On top of that, you need to decide what each module exposes publicly (its API surface) and how packages in a monorepo share code.

### Key Characteristics

- **File-scoped isolation**: Each module has its own scope; nothing leaks unless explicitly exported.
- **Feature-based organization**: Grouping by domain/feature scales better than grouping by file type.
- **Minimal API surface**: Export only what consumers need; keep implementation details private.
- **Unidirectional flow**: `shared → features → app`; never import from a feature's internals.
- **Barrel files**: `index.ts` re-exports consolidate imports, but can cause circular dependencies.
- **Monorepo sharing**: Workspace packages resolve via package names, not relative paths.
- **Explicit extensions**: Node.js ESM requires `.js` extensions in relative imports.
- **Type-only imports**: `import type` erases type-only dependencies from the output.

### Prerequisites

- Basic knowledge of TypeScript modules (`import`/`export`)
- Familiarity with `tsconfig.json` (`module`, `moduleResolution`, `paths`)
- Understanding of npm packages and `package.json`
- Familiarity with bundlers or Node.js runtime resolution

### Related Programming Areas

- **ES Modules**: The underlying module system
- **Monorepo Architecture**: npm/pnpm/yarn workspaces, Nx, Turborepo
- **Package Design**: `exports` maps, public API contracts
- **Circular Dependencies**: Causes, detection, and avoidance
- **Tree-Shaking**: How module structure affects dead-code elimination

### Core Concepts / Features

1. File-Based Modules vs. Domain/Feature-Based Directory Structures
2. Shared Utilities and Organizing Cross-Cutting Concerns
3. Public API Boundaries and Enforcing Code Encapsulation
4. Barrel Exports (`index.ts`): Advantages and Circular Dependency Pitfalls
5. Monorepo Module Sharing and Workspace Package Resolution


## 1. File-Based Modules vs. Domain/Feature-Based Directory Structures

### Definitions

**Core Definition**
File-based organization groups code by technical file type (all components in `components/`, all hooks in `hooks/`, all services in `services/`). Feature-based organization groups code by business domain or feature (everything for `auth/` in one folder, everything for `dashboard/` in another). Feature-based scales better for medium-to-large projects because it colocates related code and minimizes cross-directory changes.

**Technical Definition**
File-based (type-based) organization creates top-level directories named after technical concerns: `components/`, `hooks/`, `utils/`, `services/`, `types/`. Feature-based (domain-based) organization creates top-level directories named after business features or bounded contexts: `features/auth/`, `features/dashboard/`, `features/settings/`. Each feature directory contains its own components, hooks, types, API clients, and utilities. Shared code lives in top-level `components/`, `hooks/`, `utils/`, and `types/` directories. The rule is: import from another feature only through its public `index.ts` barrel, never from its internal files. Dependency flow is strictly unidirectional: `shared → features → app`.

**Beginner-Friendly Explanation**
Imagine a library. File-based organization is like shelving all fiction books together, all non-fiction together, all reference books together—regardless of topic. Feature-based organization is like having a "Cooking" section, a "Travel" section, a "Science" section—everything about cooking (recipes, techniques, history) in one place. In code, feature-based means: if you're working on the login feature, all the login components, hooks, types, and API calls are in one folder. You don't have to jump between `components/`, `hooks/`, and `services/` every time you change something. Shared code (like a Button component or a date formatter) lives in top-level folders and is imported by features. This keeps features independent and makes the codebase easier to navigate.

### Purposes

- To colocate related code so that changing one feature affects a single directory.
- To reduce cross-directory dependencies and hidden coupling.
- To make code ownership and review boundaries clear (one feature = one team).
- To enable features to be moved, extracted, or deleted without affecting others.
- To enforce a unidirectional dependency flow that prevents circular imports.

### Syntax Rules and Structure

**General Syntax: File-Based (Type-Based) Structure**

```
src/
├── components/
│   ├── LoginForm.tsx
│   ├── DashboardCard.tsx
│   └── SettingsPanel.tsx
├── hooks/
│   ├── useAuth.ts
│   ├── useDashboard.ts
│   └── useSettings.ts
├── services/
│   ├── authService.ts
│   ├── dashboardService.ts
│   └── settingsService.ts
├── types/
│   ├── auth.ts
│   ├── dashboard.ts
│   └── settings.ts
└── utils/
    ├── formatters.ts
    └── validators.ts
```

**Component Breakdown**
- Related files for one feature are scattered across `components/`, `hooks/`, `services/`, and `types/`.
- Changing the `auth` feature requires editing 4–6 directories.

**General Syntax: Feature-Based Structure**

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   └── SignupForm.tsx
│   │   ├── hooks/
│   │   │   └── useAuth.ts
│   │   ├── api.ts
│   │   ├── types.ts
│   │   └── index.ts          # Barrel — public API of the feature
│   ├── dashboard/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── types.ts
│   │   └── index.ts
│   └── settings/
│       ├── components/
│       ├── hooks/
│       └── index.ts
├── components/               # Shared/generic components only
│   └── ui/
├── hooks/                    # Shared hooks only
├── utils/                    # Shared utilities only
└── types/                    # Global types only
```

**Component Breakdown**
- Everything for `auth` is in `features/auth/`.
- Shared code lives in top-level `components/`, `hooks/`, `utils/`.
- Each feature has an `index.ts` barrel that defines its public API.
- Features never import from another feature's internal files—only its barrel.

**Syntax Rules**

- Use feature-based organization for projects with 3+ distinct features.
- Keep shared/generic code in top-level `components/`, `hooks/`, `utils/`, `types/`.
- Never import from another feature's internal files—use only its `index.ts`.
- Enforce unidirectional flow: `shared → features → app`.
- Colocate tests and styles next to their component (`Button.test.tsx` next to `Button.tsx`).
- Migrate from flat structure when `components/` exceeds ~20 files.
- Use `tsconfig.json` `paths` aliases (e.g., `@features/*`, `@shared/*`) for clean imports.

**Constraints and Limitations**

- Feature-based requires discipline to avoid features importing from each other.
- Shared code can become a "dumping ground" if not curated.
- Deeply nested feature directories can be harder to navigate if over-segmented.
- Small projects (< 15 components) may be simpler with a flat structure.
- Feature boundaries require team agreement on what constitutes a "feature."

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Feature-Based Import with Barrel

```typescript
// Step 1: Feature internal structure.
// src/features/auth/components/LoginForm.tsx
export function LoginForm() {
  return "Login form";
}

// src/features/auth/hooks/useAuth.ts
export function useAuth() {
  return { user: null, login: () => {} };
}

// src/features/auth/index.ts (barrel — public API)
export { LoginForm } from "./components/LoginForm";
export { useAuth } from "./hooks/useAuth";
export type { AuthUser } from "./types";

// src/features/auth/types.ts
export interface AuthUser {
  id: string;
  name: string;
}

// Step 2: App imports from the feature's public API.
// src/App.tsx
import { LoginForm, useAuth } from "@features/auth";
import type { AuthUser } from "@features/auth";

function App() {
  const { user } = useAuth();
  return <LoginForm />;
}

// Step 3: Another feature must NOT import from auth's internals.
// src/features/dashboard/components/Dashboard.tsx
// ❌ WRONG: import { useAuth } from "../auth/hooks/useAuth";
// ✅ CORRECT: import { useAuth } from "@features/auth";

console.log("Feature-based imports work.");
```

**Expected Output:**
```
Feature-based imports work.
```

**Why This Output Occurs:** The `auth` feature exposes `LoginForm`, `useAuth`, and `AuthUser` through its `index.ts` barrel. The `App` imports from `@features/auth` (the public API), not from internal paths. This enforces encapsulation and allows the auth feature to refactor internals without breaking consumers.

#### Example 2: Unidirectional Flow Enforcement

```typescript
// Step 1: Shared code (no feature dependencies).
// src/utils/formatDate.ts
export function formatDate(date: Date): string {
  return date.toISOString().split("T")[0];
}

// src/components/ui/Button.tsx
export function Button({ label }: { label: string }) {
  return `<button>${label}</button>`;
}

// Step 2: Feature imports shared code (allowed).
// src/features/dashboard/components/DashboardCard.tsx
import { formatDate } from "@utils/formatDate";
import { Button } from "@components/ui/Button";

export function DashboardCard({ title, date }: { title: string; date: Date }) {
  return `${title} — ${formatDate(date)} ${Button({ label: "View" })}`;
}

// Step 3: App imports features (allowed).
// src/App.tsx
import { DashboardCard } from "@features/dashboard";

// Step 4: Shared code must NOT import from features (violation).
// src/utils/formatDate.ts
// import { useAuth } from "@features/auth";  // ❌ VIOLATION: shared → feature

// Step 5: Features must NOT import from other features' internals (violation).
// src/features/dashboard/index.ts
// import { LoginForm } from "../auth/components/LoginForm";  // ❌ VIOLATION
// ✅ CORRECT: import { LoginForm } from "@features/auth";

console.log("Unidirectional flow enforced.");
```

**Expected Output:**
```
Unidirectional flow enforced.
```

**Why This Output Occurs:** The dependency flow is `shared → features → app`. Shared code (`utils`, `components/ui`) does not import from features. Features import from shared. The app imports from features. Cross-feature imports go through public barrels, not internals.

### Real-World Cases

**Case 1: E-Commerce Platform**
An e-commerce app organizes by feature: `features/cart/`, `features/checkout/`, `features/product-listing/`, `features/user-account/`. Shared UI components live in `components/ui/`. Each feature team owns its directory.

**Case 2: SaaS Dashboard**
A SaaS dashboard organizes by domain: `features/analytics/`, `features/billing/`, `features/settings/`, `features/team-management/`. Cross-cutting concerns (auth, notifications) live in `features/auth/` and `features/notifications/`.

**Case 3: React Design System**
A design system uses feature-based organization for component categories: `components/forms/`, `components/layout/`, `components/feedback/`, with a root barrel `design-system/index.ts` for consumers.

### References

- TypeScript Documentation: Namespaces and Modules — https://www.typescriptlang.org/docs/handbook/namespaces-and-modules.html
- Google TypeScript Style Guide: Module Organization — https://google.github.io/styleguide/tsguide.html
- Feature-Based Folder Structure (Educative) — https://www.educative.io
- DEV Community: Why I Switched to a Feature-Based Folder Structure — https://dev.to


## 2. Shared Utilities and Organizing Cross-Cutting Concerns

### Definitions

**Core Definition**
Shared utilities are reusable functions, components, hooks, and types that are used across multiple features. Cross-cutting concerns are aspects that affect many parts of the application (logging, authentication, error handling, formatting) and are organized separately from feature code to avoid duplication and coupling.

**Technical Definition**
Shared code lives in top-level directories (`utils/`, `components/ui/`, `hooks/`, `types/`) outside the `features/` directory. It must not depend on any feature—it is the foundation layer. Cross-cutting concerns are implemented as independent modules (e.g., `auth/`, `logging/`, `notifications/`) that features import as needed. The dependency flow is strict: shared code depends on nothing feature-specific; features depend on shared code; the app depends on features. This layering prevents circular dependencies and ensures that shared utilities remain stable and reusable.

**Beginner-Friendly Explanation**
Shared utilities are the "common tools" of your codebase—things like a date formatter, a validation function, or a Button component that every feature uses. Cross-cutting concerns are things like authentication, logging, or notifications that affect many features. You don't want to copy-paste these into every feature, so you put them in shared folders. The rule is simple: shared code is the bottom layer. It can't depend on features. Features can depend on shared code. This keeps things organized and prevents the tangled dependencies that make codebases hard to change.

### Purposes

- To avoid duplicating common logic across features.
- To provide a stable foundation layer that features can depend on.
- To keep cross-cutting concerns isolated and reusable.
- To prevent circular dependencies by enforcing a clear dependency hierarchy.
- To make shared code independently testable and versionable.

### Syntax Rules and Structure

**General Syntax: Shared Directory Structure**

```
src/
├── shared/                   # Everything shared across features
│   ├── components/           # Reusable UI components
│   │   └── ui/
│   │       ├── Button/
│   │       │   ├── Button.tsx
│   │       │   └── index.ts
│   │       └── Input/
│   ├── hooks/                # Reusable React hooks
│   │   └── useDebounce.ts
│   ├── utils/                # Pure utility functions
│   │   ├── formatDate.ts
│   │   ├── validators.ts
│   │   └── apiClient.ts
│   ├── types/                # Global types
│   │   └── index.ts
│   └── constants/            # Global constants
│       └── config.ts
├── features/                 # Feature-based modules
│   ├── auth/
│   ├── dashboard/
│   └── settings/
└── app/                      # App-level composition
    ├── App.tsx
    └── main.tsx
```

**Component Breakdown**
- `shared/` contains all reusable code.
- `features/` contains feature-specific code.
- `app/` composes features into the application.
- Dependency flow: `shared → features → app`.

**Syntax Rules**

- Shared code must not import from `features/` or `app/`.
- Features may import from `shared/`.
- The app may import from both `shared/` and `features/`.
- Use `tsconfig.json` `paths` aliases: `@shared/*`, `@features/*`, `@app/*`.
- Cross-cutting concerns (auth, logging) can be features or shared modules depending on scope.
- Keep `shared/` curated—don't let it become a dumping ground.
- Prefer `export type` for type-only shared dependencies.

**Constraints and Limitations**

- Shared code must be truly generic—if it's used by only one feature, keep it in that feature.
- Over-abstracting shared code too early creates unnecessary complexity.
- Shared code changes affect all features—treat it as a stable API.
- Circular dependencies can still occur if shared code imports from features (violation).
- Versioning shared code in a monorepo requires package boundaries (see Section 5).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Shared Utility and Cross-Cutting Concern

```typescript
// Step 1: Shared utility — pure function.
// src/shared/utils/formatDate.ts
export function formatDate(date: Date): string {
  return date.toISOString().split("T")[0];
}

// Step 2: Shared UI component.
// src/shared/components/ui/Button/Button.tsx
export function Button({ label, onClick }: { label: string; onClick: () => void }) {
  return `<button onclick="${onClick}">${label}</button>`;
}

// Step 3: Cross-cutting concern — logging module.
// src/shared/logging/logger.ts
export function log(level: "info" | "warn" | "error", message: string): void {
  console.log(`[${level.toUpperCase()}] ${message}`);
}

// Step 4: Feature imports shared code.
// src/features/dashboard/components/Dashboard.tsx
import { formatDate } from "@shared/utils/formatDate";
import { Button } from "@shared/components/ui/Button";
import { log } from "@shared/logging/logger";

export function Dashboard({ title, date }: { title: string; date: Date }) {
  log("info", `Rendering dashboard: ${title}`);
  return `${title} — ${formatDate(date)} ${Button({ label: "Refresh", onClick: () => {} })}`;
}

// Step 5: Shared code must NOT import from features (violation check).
// src/shared/utils/formatDate.ts
// import { useAuth } from "@features/auth";  // ❌ VIOLATION

console.log("Shared utilities and cross-cutting concerns organized.");
```

**Expected Output:**
```
Shared utilities and cross-cutting concerns organized.
```

**Why This Output Occurs:** `formatDate`, `Button`, and `log` live in `shared/`. The `dashboard` feature imports them. Shared code does not depend on features. This enforces the dependency hierarchy.

#### Example 2: Cross-Cutting Auth Module

```typescript
// Step 1: Auth as a cross-cutting concern (or feature depending on scope).
// src/shared/auth/AuthContext.tsx
import type { ReactNode } from "react";

export interface AuthUser {
  id: string;
  name: string;
  role: "admin" | "user";
}

export function createAuthContext() {
  return {
    user: null as AuthUser | null,
    login: (user: AuthUser) => {},
    logout: () => {},
  };
}

// Step 2: Feature uses auth.
// src/features/dashboard/index.ts
export { Dashboard } from "./components/Dashboard";

// src/features/dashboard/components/Dashboard.tsx
import { createAuthContext } from "@shared/auth/AuthContext";

export function Dashboard() {
  const auth = createAuthContext();
  return auth.user ? `Welcome, ${auth.user.name}` : "Please log in";
}

// Step 3: Another feature uses the same auth.
// src/features/settings/index.ts
// src/features/settings/components/Settings.tsx
import { createAuthContext } from "@shared/auth/AuthContext";

export function Settings() {
  const auth = createAuthContext();
  return auth.user?.role === "admin" ? "Admin settings" : "User settings";
}

console.log("Cross-cutting auth organized.");
```

**Expected Output:**
```
Cross-cutting auth organized.
```

**Why This Output Occurs:** The auth context lives in `shared/auth/`, making it available to all features without duplication. Features import from `@shared/auth/AuthContext`. This is a cross-cutting concern organized at the shared layer.

### Real-World Cases

**Case 1: Formatters and Validators**
Date formatters, currency formatters, email validators, and phone number validators are shared utilities used across features.

**Case 2: API Client**
A shared API client (`apiClient.ts`) handles HTTP requests, authentication headers, error handling, and retries—used by all feature API modules.

**Case 3: Logging and Monitoring**
A shared logging module provides consistent logging across features, with feature-specific loggers composed from the shared base.

**Case 4: Design System Components**
Buttons, inputs, modals, and other UI primitives live in `shared/components/ui/` and are used by all features.

### References

- Google TypeScript Style Guide: Module Organization — https://google.github.io/styleguide/tsguide.html
- TypeScript Documentation: Modules — https://www.typescriptlang.org/docs/handbook/modules.html
- Feature-Based Organization (Impertio Studio) — https://github.com/Impertio-Studio


## 3. Public API Boundaries and Enforcing Code Encapsulation

### Definitions

**Core Definition**
A public API boundary is the set of exports that a module or package intentionally exposes to consumers. Everything else is internal and subject to change. Enforcing encapsulation means using barrel files, `package.json` `exports` maps, and linting rules to ensure that internal implementation details are not imported by external code.

**Technical Definition**
In TypeScript, public API boundaries are enforced through three mechanisms: (1) **barrel files** (`index.ts`) that re-export only the intended public API; (2) **`package.json` `exports` maps** that block deep imports into package internals; and (3) **ESLint rules** (e.g., `eslint-plugin-cooper`) that detect circular imports through barrel re-exports and enforce same-level export boundaries. The principle is: export only what consumers need. Internal implementation details should remain private to allow refactoring without breaking changes. Every exported type becomes part of the contract—consumers can reference it, structurally extend it, and break when it changes.

**Beginner-Friendly Explanation**
A public API boundary is like the front door of a building. Consumers can come through the front door (the `index.ts` barrel), but they can't wander into the back offices (internal files). You control what's visible by only exporting the front-door stuff. If you export everything, consumers might start depending on internal details, and then you can't change those details without breaking their code. The rule is: export only what's necessary for consumers to do their job. Keep everything else private. In packages, you can enforce this with `package.json` `exports` maps, which physically block deep imports like `my-package/internal/secret`.

### Purposes

- To reduce coupling between modules and their consumers.
- To allow internal refactoring without breaking changes.
- To make the public API easier to document and understand.
- To enforce encapsulation at the package boundary.
- To prevent accidental dependencies on implementation details.

### Syntax Rules and Structure

**General Syntax: Minimal Public API (Barrel)**

```typescript
// src/features/auth/index.ts
// Public API — only what consumers need
export { LoginForm } from "./components/LoginForm";
export { SignupForm } from "./components/SignupForm";
export { useAuth } from "./hooks/useAuth";
export type { AuthUser } from "./types";

// Internal — NOT exported
// src/features/auth/components/InternalHelper.tsx
// src/features/auth/utils/validators.ts
```

**Component Breakdown**
- `index.ts` exports only the public API.
- Internal files are not re-exported.
- Consumers import from `@features/auth`, not `@features/auth/components/InternalHelper`.

**General Syntax: Package `exports` Map**

```json
{
  "name": "@myorg/auth",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    },
    "./package.json": "./package.json"
  }
}
```

**Component Breakdown**
- `"."`: The package root (public API).
- Deep imports like `@myorg/auth/internal` are blocked (not listed in `exports`).
- `"./package.json"` is commonly exposed for tooling.

**General Syntax: ESLint Barrel Boundary Enforcement**

```json
{
  "rules": {
    "cooper/same-level-exports": "error",
    "cooper/no-circular-imports": "error"
  }
}
```

**Component Breakdown**
- `same-level-exports`: Barrel files can only re-export from same-level files.
- `no-circular-imports`: Detects circular imports through barrel re-exports.

**Syntax Rules**

- Export only what consumers need; keep implementation details private.
- Use `index.ts` barrels to define the public API of a feature.
- Use `package.json` `exports` maps to block deep imports into packages.
- Use ESLint rules to enforce barrel boundaries and detect circular imports.
- Every exported type becomes part of the contract—be intentional about exports.
- Use `export type` for type-only public API entries.
- Document the public API surface explicitly.

**Constraints and Limitations**

- Over-exporting makes refactoring painful because consumers depend on internals.
- `exports` maps are not respected by all bundlers and tools in all configurations.
- ESLint rules add build-time overhead.
- Feature teams may resist encapsulation if it slows down development.
- Public API changes still require semantic versioning discipline.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal Public API vs. Over-Exporting

```typescript
// ===== INCORRECT (over-exporting) =====
// src/features/auth/index.ts
export const API_ENDPOINT = "/api/auth";
export const MAX_RETRIES = 3;
export function validateUser(user: { name: string }): boolean {
  return user.name.length > 0;
}
export function formatUserForApi(user: { name: string; id: number }) {
  return { userName: user.name, userId: user.id };
}
export async function createUser(name: string) {
  const user = { name, id: "1" };
  if (!validateUser(user)) throw new Error("Invalid");
  const apiUser = formatUserForApi(user);
  return apiUser;
}

// ===== CORRECT (minimal exports) =====
// src/features/auth/index.ts
const API_ENDPOINT = "/api/auth";
const MAX_RETRIES = 3;
function validateUser(user: { name: string }): boolean {
  return user.name.length > 0;
}
function formatUserForApi(user: { name: string; id: number }) {
  return { userName: user.name, userId: user.id };
}
export async function createUser(name: string) {
  const user = { name, id: "1" };
  if (!validateUser(user)) throw new Error("Invalid");
  return formatUserForApi(user);
}

console.log("Minimal API surface enforced.");
```

**Expected Output:**
```
Minimal API surface enforced.
```

**Why This Output Occurs:** The incorrect version exports `API_ENDPOINT`, `MAX_RETRIES`, `validateUser`, and `formatUserForApi`—all internal details. The correct version keeps them private and exports only `createUser`. Consumers can refactor internal functions freely without breaking the public API.

#### Example 2: Package `exports` Map Blocking Deep Imports

```json
// package.json for @myorg/auth
{
  "name": "@myorg/auth",
  "version": "1.0.0",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    },
    "./package.json": "./package.json"
  }
}
```

```typescript
// Consumer code
import { createUser } from "@myorg/auth";        // ✅ Allowed (public API)
import { validateUser } from "@myorg/auth/dist/internal/validators";  // ❌ Blocked

// The exports map does not list "./dist/internal/validators",
// so the import fails at resolution time.
```

**Expected Output:** The first import succeeds; the second fails with a module resolution error.

**Why This Output Occurs:** The `exports` map only exposes `"."` (the package root) and `"./package.json"`. Deep imports into `dist/internal/` are physically blocked because they are not listed in the map. This enforces encapsulation at the package boundary.

### Real-World Cases

**Case 1: Library Packages**
npm libraries use `exports` maps to expose only the public API, blocking deep imports into compiled internals.

**Case 2: Design System Packages**
Design systems expose `Button`, `Input`, `Card` from the package root, but block imports into `src/internal/` or `src/utils/`.

**Case 3: Monorepo Shared Packages**
Shared packages in a monorepo use `exports` maps to enforce encapsulation, even though workspace resolution would otherwise allow deep imports.

**Case 4: SDK Packages**
SDKs expose `createClient`, `ClientConfig`, and public types, while keeping internal HTTP logic, retry logic, and serialization private.

### References

- Google TypeScript Style Guide: Export Visibility — https://google.github.io/styleguide/tsguide.html#export-visibility
- Minimize Exported API Surface (dot-skills) — https://github.com/pproenca/dot-skills
- Node.js Packages: `exports` Field — https://nodejs.org/api/packages.html#exports
- eslint-plugin-cooper — https://www.npmjs.com/package/eslint-plugin-cooper


## 4. Barrel Exports (`index.ts`): Advantages and Circular Dependency Pitfalls

### Definitions

**Core Definition**
A barrel file is an `index.ts` file that re-exports from multiple modules, consolidating imports into a single path. Barrels improve developer experience by reducing import verbosity, but they introduce significant risks: circular dependencies, tree-shaking failures, and slower test startup.

**Technical Definition**
Barrel files use `export { name } from "./module"` or `export * from "./module"` to re-export names from sibling modules. When a barrel re-exports a module that itself imports from the barrel (directly or transitively), a circular dependency is created. At bundle time, this non-determinism can result in undefined exports or order-sensitive bugs. Barrel files also confuse tree-shakers: importing one named export from a barrel may load every module re-exported by that barrel. ESLint rules like `cooper/same-level-exports` restrict barrels to re-export only from same-level files, preventing cross-feature circular dependencies.

**Beginner-Friendly Explanation**
A barrel file is a convenience: instead of importing from five different paths, you import from one `index.ts`. For example, `import { Button, Input, Card } from "./components"` instead of three separate imports. But barrels have a dark side: they create circular dependencies. Imagine `Button` imports from `Input`, and `Input` imports from the barrel that re-exports both. Now `Button` depends on the barrel, which depends on `Input`, which depends on the barrel—a cycle. This causes "undefined import" errors, breaks hot module replacement, and confuses bundlers. The rule of thumb: barrels are fine for public API entry points, but avoid using them for internal cross-module imports. Use direct imports for internal dependencies.

### Purposes

- To consolidate imports from a feature or package into a single path.
- To define a clear public API for a module or feature.
- To improve developer experience by reducing import path length.
- To enable consumers to import from a package root rather than deep paths.
- To organize re-exports in a predictable, consistent way.

### Syntax Rules and Structure

**General Syntax: Basic Barrel File**

```typescript
// src/components/index.ts
export { Button } from "./Button";
export { Input } from "./Input";
export { Card } from "./Card";
export type { ButtonProps } from "./Button";
```

**Component Breakdown**
- `export { name } from "./module"`: Re-exports specific names.
- Consumers import from `./components` instead of three separate files.

**General Syntax: `export *` Barrel**

```typescript
// src/components/index.ts
export * from "./Button";
export * from "./Input";
export * from "./Card";
```

**Component Breakdown**
- `export *`: Re-exports all named exports from each module.
- Does not re-export default exports.
- Can confuse tree-shakers.

**General Syntax: Circular Dependency (Anti-Pattern)**

```typescript
// src/components/index.ts
export { Button } from "./Button";
export { Input } from "./Input";

// src/components/Button.tsx
import { Input } from "./index";  // ❌ Circular: Button → index → Input → index

// src/components/Input.tsx
import { Button } from "./index";  // ❌ Circular
```

**Component Breakdown**
- `Button` imports from the barrel, which re-exports `Input`.
- `Input` imports from the barrel, which re-exports `Button`.
- This creates a cycle that can cause undefined exports at runtime.

**Syntax Rules**

- Use barrels for public API entry points (feature `index.ts`, package root).
- Avoid using barrels for internal cross-module imports—use direct imports.
- Prefer named re-exports (`export { name }`) over `export *` for tree-shaking.
- Restrict barrels to same-level exports (ESLint `cooper/same-level-exports`).
- Detect circular imports with ESLint `cooper/no-circular-imports`.
- In packages, combine barrels with `exports` maps for physical encapsulation.
- Consider per-family subpath exports instead of a single root barrel for large packages.

**Constraints and Limitations**

- Barrel files are the #1 cause of "Module undefined" circular dependency errors.
- They often confuse tree-shakers, risking entire libraries being bundled when one function is needed.
- Importing one named export from a barrel often loads every exported module, slowing tests.
- Circular dependencies break Hot Module Replacement (HMR) and cause unpredictable behavior.
- `export *` does not re-export default exports.
- Bundlers have known edge cases where cycles degrade or crash.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Barrel File with Circular Dependency

```typescript
// ===== ANTI-PATTERN: Circular dependency =====
// src/components/index.ts
export { Button } from "./Button";
export { Input } from "./Input";

// src/components/Button.tsx
import { Input } from "./index";  // ❌ Circular
export function Button() {
  return `Button with ${Input()}`;
}

// src/components/Input.tsx
import { Button } from "./index";  // ❌ Circular
export function Input() {
  return `Input with ${Button()}`;
}

// At runtime, one of these will be undefined due to the cycle.

// ===== CORRECT: Direct imports =====
// src/components/Button.tsx
import { Input } from "./Input";  // ✅ Direct import, no barrel
export function Button() {
  return `Button with ${Input()}`;
}

// src/components/Input.tsx
// Input does not need Button — no cycle.
export function Input() {
  return "Input";
}

console.log("Circular dependency avoided.");
```

**Expected Output:**
```
Circular dependency avoided.
```

**Why This Output Occurs:** The anti-pattern creates a cycle through the barrel. The correct pattern uses direct imports (`import { Input } from "./Input"`), breaking the cycle. The barrel remains for public API consumption, but internal modules don't import from it.

#### Example 2: Tree-Shaking Failure with `export *`

```typescript
// Step 1: Large utility module.
// src/utils/math.ts
export function add(a: number, b: number) { return a + b; }
export function multiply(a: number, b: number) { return a * b; }
export function divide(a: number, b: number) { return a / b; }
export function subtract(a: number, b: number) { return a - b; }
export const PI = 3.14159;
export const E = 2.71828;

// Step 2: Barrel with `export *`.
// src/utils/index.ts
export * from "./math";

// Step 3: Consumer imports only `add`.
// src/app.ts
import { add } from "./utils";
console.log(add(1, 2));  // 3

// Step 4: With `export *`, the bundler may keep `multiply`, `divide`,
// `subtract`, `PI`, and `E` because it cannot statically determine
// which names are used through the barrel.

// Step 5: Named re-exports fix tree-shaking.
// src/utils/index.ts
export { add, multiply, divide, subtract, PI, E } from "./math";

// Now the bundler knows exactly which names are exported and can
// tree-shake unused ones.

console.log("Tree-shaking demonstrated.");
```

**Expected Output:**
```
3
Tree-shaking demonstrated.
```

**Why This Output Occurs:** `export *` prevents bundlers from determining which exports are used, potentially including all of them in the bundle. Named re-exports provide the static information needed for tree-shaking.

### Real-World Cases

**Case 1: Design System Barrels**
Design systems use barrels for public API consumption (`import { Button, Input } from "@design-system"`), but internal components use direct imports to avoid cycles.

**Case 2: Icon Libraries**
Icon libraries use named re-exports in a barrel to expose hundreds of icons, enabling tree-shaking so consumers only bundle the icons they use.

**Case 3: Component Library Subpath Exports**
Large component libraries switch from a single root barrel to per-family subpath exports (e.g., `@lib/forms`, `@lib/layout`) to reduce circular dependency risk and improve tree-shaking.

**Case 4: Monorepo Package Barrels**
Monorepo packages use barrels for their public API, combined with `exports` maps to block deep imports and prevent consumers from bypassing the barrel.

### References

- Avoid Barrel Exports (Callstack Incubator) — https://github.com/callstackincubator/agent-skills
- no-barrel-file — https://www.npmjs.com/package/no-barrel-file
- eslint-plugin-cooper — https://www.npmjs.com/package/eslint-plugin-cooper
- Turbopack Barrel Chunk Failure (JSR Issue) — https://jsr.io
- Cloudpack: Circular Dependencies — https://microsoft.github.io


## 5. Monorepo Module Sharing and Workspace Package Resolution

### Definitions

**Core Definition**
A monorepo is a single repository containing multiple packages (apps, libraries, shared modules) that can depend on each other. Workspace package resolution allows TypeScript to resolve imports like `@myorg/shared` to the correct workspace package, either through npm/pnpm/yarn workspaces or through `tsconfig.json` `paths` aliases.

**Technical Definition**
Monorepo module sharing operates through two resolution mechanisms: (1) **`tsconfig.json` `paths` aliases**, which map import specifiers to directories (e.g., `"@shared/*": ["./shared/*"]`), applied in longest-match-first order; and (2) **npm/pnpm/yarn workspace packages**, where the root `package.json` `workspaces` field defines package locations, and each workspace's `package.json` `name`, `exports`, `module`, `main`, and `types` fields determine its entry points. Resolution priority is: relative path with extension fallback → `tsconfig` `paths` alias → npm workspace package by name → bare npm specifier. TypeScript project references enable incremental builds across packages. Tools like Nx and Turborepo add caching, task orchestration, and dependency graph visualization on top of workspaces.

**Beginner-Friendly Explanation**
A monorepo is a single Git repository that holds many packages—like a web app, an admin app, and a shared component library. Instead of publishing the shared library to npm and installing it in each app, the packages live in the same repo and can import from each other directly. Workspace resolution means that when `apps/web` imports `@myorg/ui`, the package manager (npm, pnpm, yarn) creates a symlink in `node_modules` so the import resolves to `packages/ui`. TypeScript's `paths` aliases provide a similar mechanism for compile-time resolution. The benefit is: change the shared library, and all apps see the change immediately—no publish-and-install cycle. The challenge is: you need to configure `tsconfig`, `package.json` `workspaces`, and build tooling correctly so everything resolves.

### Purposes

- To share code between applications without publishing to npm.
- To enable atomic changes across packages (one PR changes shared library and consumers).
- To reduce duplication of shared utilities, types, and components.
- To enable incremental builds with TypeScript project references.
- To provide a single source of truth for shared dependencies.

### Syntax Rules and Structure

**General Syntax: npm Workspaces `package.json`**

```json
{
  "name": "my-monorepo",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ]
}
```

**Component Breakdown**
- `"private": true`: The root package is not published.
- `"workspaces": ["apps/*", "packages/*"]`: All packages in `apps/` and `packages/` are workspace members.

**General Syntax: Workspace Package `package.json`**

```json
{
  "name": "@myorg/ui",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  }
}
```

**Component Breakdown**
- `"name": "@myorg/ui"`: The package name used in imports.
- `"exports"`: The public API entry points.
- `"types"`: The TypeScript declaration file.

**General Syntax: `tsconfig.json` `paths` Aliases**

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@myorg/ui": ["./packages/ui/src/index.ts"],
      "@shared/*": ["./shared/*"]
    }
  }
}
```

**Component Breakdown**
- `paths`: Maps import specifiers to source directories.
- `baseUrl`: The base directory for resolving paths.
- Aliases resolve in longest-match-first order.

**General Syntax: TypeScript Project References**

```json
{
  "references": [
    { "path": "./packages/ui" },
    { "path": "./packages/shared" }
  ]
}
```

**Component Breakdown**
- `references`: Lists project dependencies for incremental builds.
- Enables `tsc --build` to compile packages in dependency order.

**Syntax Rules**

- Use npm/pnpm/yarn workspaces for package-level sharing.
- Use `tsconfig.json` `paths` for compile-time alias resolution.
- Configure `exports` maps in each workspace `package.json` to block deep imports.
- Use TypeScript project references for incremental builds.
- Resolution priority: relative → `paths` alias → workspace package → bare specifier.
- Extension fallback: `.mts` → `.ts` → `.tsx` → `.mjs` → `.js` → `.jsx`.
- Keep external dependency versions uniform across packages (single version policy).

**Constraints and Limitations**

- Conflicting `paths` aliases across packages can leak if not properly scoped.
- Workspace resolution requires the package manager to create symlinks (`npm install` must be run).
- TypeScript does not automatically know about workspace packages—`tsconfig` must be configured.
- Deep imports into workspace packages bypass `exports` maps if not enforced by the package manager.
- Monorepos add build complexity (task orchestration, caching, CI configuration).
- Nested `node_modules` and symlink issues can cause resolution failures in some tools.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: npm Workspaces with Shared Package

```json
// Step 1: Root package.json
{
  "name": "my-monorepo",
  "private": true,
  "workspaces": ["apps/*", "packages/*"]
}
```

```
// Step 2: Directory structure
my-monorepo/
├── package.json
├── apps/
│   └── web/
│       ├── package.json
│       └── src/
│           └── App.tsx
└── packages/
    └── ui/
        ├── package.json
        └── src/
            └── index.ts
```

```json
// Step 3: packages/ui/package.json
{
  "name": "@myorg/ui",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts"
}
```

```typescript
// Step 4: packages/ui/src/index.ts
export function Button({ label }: { label: string }) {
  return `<button>${label}</button>`;
}
```

```typescript
// Step 5: apps/web/src/App.tsx
import { Button } from "@myorg/ui";

console.log(Button({ label: "Click me" }));
```

```bash
# Step 6: Install and run
npm install          # Creates symlinks in node_modules
npm run build --workspace @myorg/ui
```

**Expected Output:**
```
<button>Click me</button>
```

**Why This Output Occurs:** The root `package.json` defines workspaces. `npm install` creates a symlink from `node_modules/@myorg/ui` to `packages/ui`. The app imports `@myorg/ui`, which resolves to the workspace package. The shared Button component is used without publishing to npm.

#### Example 2: `tsconfig` Paths and Project References

```json
// Step 1: Root tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@myorg/ui": ["./packages/ui/src/index.ts"],
      "@myorg/shared": ["./packages/shared/src/index.ts"]
    }
  },
  "references": [
    { "path": "./packages/ui" },
    { "path": "./packages/shared" }
  ]
}
```

```json
// Step 2: packages/ui/tsconfig.json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "composite": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src"]
}
```

```typescript
// Step 3: apps/web/src/App.tsx
import { Button } from "@myorg/ui";
import { formatDate } from "@myorg/shared";

console.log(Button({ label: formatDate(new Date()) }));
```

```bash
# Step 4: Build with project references
tsc --build
# Compiles @myorg/shared first, then @myorg/ui, then apps/web
```

**Expected Output:**
```
<button>2024-01-01</button>
```

**Why This Output Occurs:** The `paths` aliases resolve `@myorg/ui` and `@myorg/shared` to source directories at compile time. Project references ensure that `@myorg/shared` is built before `@myorg/ui`, and `@myorg/ui` before `apps/web`. This enables incremental builds.

### Real-World Cases

**Case 1: Nx Monorepo**
Nx uses npm/pnpm/yarn workspaces plus its own project graph and task orchestration to manage TypeScript monorepos with caching and affected builds.

**Case 2: Turborepo**
Turborepo adds high-performance task caching and remote caching on top of npm/pnpm/yarn workspaces, with pipeline configuration for build, test, and lint.

**Case 3: Enterprise Design System**
A design system monorepo contains `packages/tokens`, `packages/icons`, `packages/components`, and `apps/docs`, sharing code through workspaces and `tsconfig` paths.

**Case 4: Full-Stack TypeScript Monorepo**
A monorepo with `apps/web`, `apps/api`, and `packages/shared-types` shares TypeScript types between frontend and backend, ensuring API contracts stay synchronized.

### References

- Nx Blog: Everything You Need to Know About TypeScript Project References — https://nx.dev/blog
- npm Workspaces Documentation — https://docs.npmjs.com/cli/using-npm/workspaces
- TypeScript Project References — https://www.typescriptlang.org/docs/handbook/project-references.html
- Monorepo Resolution (no-mistakes) — https://github.com/jonathanong/no-mistakes
- Set up a Monorepo (TypeScript Template) — https://raw.githubusercontent.com
- Turborepo Documentation — https://turbo.build


## Summary: Module Organization Decision Guide

| Decision | File-Based | Feature-Based | Recommendation |
|----------|------------|---------------|----------------|
| **< 15 components** | ✅ Simple | Overkill | File-based (flat) |
| **3+ features** | Hard to maintain | ✅ Scalable | Feature-based |
| **20+ files in `components/`** | ❌ Migrate | ✅ Use | Feature-based |
| **Shared code** | Top-level dirs | Top-level dirs | Both |
| **Public API** | Manual discipline | Barrel + `exports` map | `exports` map |
| **Internal imports** | Direct imports | Direct imports | Never use barrels internally |
| **Cross-feature imports** | N/A | Only through barrel | Enforce with ESLint |
| **Monorepo sharing** | N/A | N/A | npm workspaces + `tsconfig` paths |
| **Barrel files** | ❌ Avoid | ✅ Public API only | Named re-exports, same-level |
| **Circular dependencies** | ❌ Cause errors | ✅ Prevented by direct imports | ESLint detection |


## References

- TypeScript Documentation: Namespaces and Modules — https://www.typescriptlang.org/docs/handbook/namespaces-and-modules.html
- TypeScript Documentation: Modules — https://www.typescriptlang.org/docs/handbook/modules.html
- Google TypeScript Style Guide: Module Organization — https://google.github.io/styleguide/tsguide.html
- Google TypeScript Style Guide: Export Visibility — https://google.github.io/styleguide/tsguide.html#export-visibility
- Minimize Exported API Surface (dot-skills) — https://github.com/pproenca/dot-skills
- Avoid Barrel Exports (Callstack Incubator) — https://github.com/callstackincubator/agent-skills
- no-barrel-file — https://www.npmjs.com/package/no-barrel-file
- eslint-plugin-cooper — https://www.npmjs.com/package/eslint-plugin-cooper
- Nx Blog: TypeScript Project References — https://nx.dev/blog
- npm Workspaces Documentation — https://docs.npmjs.com/cli/using-npm/workspaces
- TypeScript Project References — https://www.typescriptlang.org/docs/handbook/project-references.html
- Monorepo Resolution (no-mistakes) — https://github.com/jonathanong/no-mistakes
- Node.js Packages: `exports` Field — https://nodejs.org/api/packages.html#exports
- Feature-Based Folder Structure (Educative) — https://www.educative.io
- Turborepo Documentation — https://turbo.build
- Pluralsight: Structure TypeScript Applications with Barrel Files — https://www.pluralsight.com/labs/codeLabs/guided-structure-typescript-applications-with-barrel-files-and-module-re-exports