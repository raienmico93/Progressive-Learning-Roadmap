# Module Augmentation in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Module augmentation** is a TypeScript technique that allows a module to **re-open an existing module's declarations** and **merge new members into its exported types**, without modifying the original source code. It combines two TypeScript features: `declare module` syntax (to target another module) and **declaration merging** (to combine the new declarations with existing ones). Module augmentation is the type-level equivalent of runtime patching: it lets your code extend a third-party library's interfaces as if those extensions were defined in the library itself.

**Technical Definition.** A module augmentation is a `declare module "specifier" { ... }` block that appears inside a **module file** (a file with at least one top-level `import` or `export`). The specifier is resolved using the same rules as `import`/`export` specifiers. Declarations inside the block are **merged** with the existing declarations of the targeted module using declaration merging. Module augmentations **cannot add new items to the top-level scope**—they can only **patch existing declarations**. The augmentation file must be a module; a bare `declare module` in a script file augments the global scope instead. The modifications are **type-only**; runtime behavior must be patched separately.

**Beginner-Friendly Explanation.** Module augmentation is **how you add new types to an existing library without editing the library**. Imagine a library gives you a `Request` type that does not have a `userId` field, but your auth middleware adds one at runtime. Instead of casting `(req as any).userId` everywhere, you **re-open** the library's module once in your own code and **merge** a `userId` field into its `Request` interface. From then on, every file in your project sees `req.userId` as typed. It is like adding a page to a book without reprinting the whole book.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Merges, does not replace** | New declarations merge with existing ones; they do not override the module. |
| **Type-only** | Augmentations add compile-time types only; runtime behavior must be implemented separately. |
| **Requires module context** | The augmentation file must be a module (top-level `import`/`export`), or it becomes a global augmentation. |
| **Must import the target** | The target module must be imported first; otherwise the augmentation may replace rather than merge. |
| **Implementation module, not re-export** | Augment the module where the type is **declared**, not where it is re-exported. |
| **Cannot add top-level names** | Module augmentations patch existing declarations; they do not create new global names. |
| **No class merging** | Classes do not declaration-merge; augment a surrounding namespace or interface instead. |

---

### Prerequisites

- **TypeScript modules**: `import`/`export`, module detection, and `declare module`.
- **Declaration merging**: how interfaces, namespaces, and modules combine.
- **Declaration files** (`.d.ts`) and ambient contexts.
- **Third-party library structure**: understanding entry points vs. implementation modules.
- **`tsconfig.json`**: module resolution and `include` settings.

---

### Related Programming Areas

- **Declaration merging** — the underlying mechanism that makes augmentation work.
- **Global augmentation** (`declare global`) — the global-scope counterpart.
- **DefinitelyTyped and `@types`** — type packages that may already declare the module you want to augment.
- **Runtime patching** — the value-level counterpart to type-level augmentation.
- **Module resolution** — how the augmented specifier is resolved.

---

## Core Concepts / Features

---

## 1. Module Augmentation

### Definitions

**Core Definition.** **Module augmentation** is the act of using a `declare module` block inside a module to **merge new declarations into an existing module's exported types**.

**Technical Definition.** A module augmentation is a `declare module "specifier" { ... }` declaration nested inside a module (or another ambient external module). Its declarations are merged with the target module's declarations. The specifier resolves like an import specifier. Augmentations are **type-only** and **cannot create new top-level items**—they only patch existing declarations.

**Beginner-Friendly Explanation.** Module augmentation is **adding fields and types to a library's existing interfaces** from your own code, without touching `node_modules`.

### Purposes

- **To add fields to third-party interfaces** that are populated at runtime by middleware or plugins.
- **To narrow generic types** (e.g., replacing a broad `string` type with a strict union) without forking the library.
- **To extend library types** with project-specific properties without casts at every call site.
- **To layer project-specific types on top of upstream definitions** while preserving the originals.
- **To support plugin ecosystems** where each plugin contributes type extensions to a shared module.

### Syntax Rules and Structure

**General syntax:**

```typescript
import "target-module"; // Ensure the module is loaded and the file is a module.

declare module "target-module" {
  interface ExistingInterface {
    newProperty: Type;
  }
}
```

**Component breakdown:**

- `import "target-module"` — a **side-effect import** that ensures the module is loaded and the file is treated as a module.
- `declare module "target-module"` — re-opens the module for augmentation.
- `interface ExistingInterface { ... }` — merges new members into an existing interface.

**Syntax rules.**

- The augmentation file **must be a module** (top-level `import` or `export`). A bare `declare module` in a script file augments the **global scope** instead.
- The target module **must be imported first**; otherwise TypeScript may treat the declaration as a replacement rather than an augmentation.
- Only **existing declarations** can be patched; new top-level names cannot be introduced.
- Classes **do not merge**; augment an interface or namespace instead.
- Augment the **implementation module** where the type is declared, not a re-export.

**Constraints and limitations.**

- Augmentations are **type-only**; they do not add runtime behavior.
- Adding required properties can make early-pipeline code lie about its state; prefer **optional** properties for middleware-attached fields.
- Invasive changes (changing existing field types, removing fields) are **not possible**; fork the type or contribute upstream.

### Annotated Complete Step by Step Code Examples

#### Example 1: Augmenting Express's `Request` with `userId`

```typescript
// src/types/express.d.ts
import "express"; // Ensure module context and load the module.

declare module "express-serve-static-core" {
  interface Request {
    userId?: string;   // Attached by auth middleware — optional.
    requestId: string; // Attached by tracing middleware — always present after middleware.
  }
}
```

```typescript
// src/routes/user.ts
import express from "express";

const app = express();

app.get("/me", (req, res) => {
  const userId = req.userId;       // string | undefined — recognised.
  const reqId = req.requestId;     // string — recognised.
  res.json({ userId, reqId });
});
```

**Setup.** Compile with `npx tsc --strict`. The augmentation file must be included in `tsconfig.json`.

**Expected behavior.** `req.userId` is typed `string | undefined`; `req.requestId` is typed `string`. No casts are needed.

**Why this output occurs.** The augmentation targets `express-serve-static-core`, where Express's `Request` is **actually declared**, not the `express` re-export. The `import "express"` ensures the file is a module. The optional `userId` correctly models middleware that may or may not have run.

---

#### Example 2: Augmenting a local module

```typescript
// observable.ts
export class Observable<T> {
  // ...
}
```

```typescript
// map.ts
import { Observable } from "./observable";

declare module "./observable" {
  interface Observable<T> {
    map<U>(proj: (el: T) => U): Observable<U>;
  }
}

Observable.prototype.map = function <T, U>(
  this: Observable<T>,
  proj: (el: T) => U
): Observable<U> {
  // runtime implementation
  return new Observable<U>();
};
```

```typescript
// consumer.ts
import { Observable } from "./observable";
import "./map";

let o: Observable<number>;
o.map((x) => x.toFixed()); // ✅ map is recognised.
```

**Expected behavior.** The `Observable` class gains a `map` method **at the type level**, and the runtime prototype is patched separately.

**Why this output occurs.** The `declare module "./observable"` block **merges** an `Observable<T>` interface into the existing class declaration. Classes themselves do not merge, but interface merging adds the method to the class's instance type. The runtime patch in `map.ts` makes the method actually callable.

---

#### Example 3: Narrowing a generic type via augmentation

```typescript
// permissions.types.d.ts
import "@nx-permissions/permissions";

declare module "@nx-permissions/permissions" {
  export type Permission = "ALLOW_X" | "ALLOW_Y";
}
```

```typescript
// app.ts
import { Permission, PermissionsDirective } from "@nx-permissions/permissions";

const permissions: Permission[] = ["ALLOW_X"]; // ✅ Strict union.
// const bad: Permission[] = ["ALLOW_Z"]; // ❌ Error.
```

**Expected behavior.** `Permission` is narrowed to `"ALLOW_X" | "ALLOW_Y"` **without** breaking other exports like `PermissionsDirective`.

**Why this output occurs.** The `import "@nx-permissions/permissions"` is **required**; without it, TypeScript may treat the `declare module` block as a **replacement**, causing "no exported member" errors for other exports. With the import, the new `Permission` type **merges** with the existing export.

---

### Real-World Cases

- **Express middleware**: adding `userId`, `requestId`, or `tenant` to `Express.Request`.
- **Hardhat plugins**: adding `MyType` to `hardhat/types/network` or extending `NetworkConnection`.
- **Angular libraries**: narrowing a generic `Permission` type to application-specific strict unions.
- **Stream Chat**: extending `CustomChannelData` via module augmentation.
- **Payload CMS**: overriding request context types with stricter application-specific shapes.

---

## 2. `declare module`

### Definitions

**Core Definition.** `declare module "specifier" { ... }` is the syntactic form used for **both** ambient module declarations (describing an untyped module) and **module augmentations** (extending an existing module). The distinction depends on **whether the target module already exists** and **whether the file is a module**.

**Technical Definition.** When `declare module "specifier"` appears inside a **module file** and the specifier resolves to an **existing module**, it acts as an **augmentation**. When it appears in a **global `.d.ts` file** or the specifier does not resolve to an existing module, it acts as an **ambient declaration** describing a new module. The specifier is resolved using the same rules as `import`/`export` specifiers.

**Beginner-Friendly Explanation.** `declare module` is **the syntax you use to either describe a new module or extend an existing one**. Inside a module file, targeting an existing module, it **extends**; in a global declaration file, it **describes**.

### Purposes

- **To target an existing module for augmentation**.
- **To describe an untyped module** in a global declaration file.
- **To provide a scoped container** for merged declarations.
- **To distinguish augmentation from global pollution** by keeping declarations modular.

### Syntax Rules and Structure

**Augmentation form (inside a module):**

```typescript
import "target";
declare module "target" {
  interface Existing {
    added: string;
  }
}
```

**Ambient declaration form (global `.d.ts`):**

```typescript
declare module "untyped-lib" {
  export function fn(): void;
}
```

**Syntax rules.**

- Inside a module, `declare module "existing"` **augments**.
- In a global `.d.ts`, `declare module "name"` **declares** a new ambient module.
- The target module must be imported before augmentation to avoid replacement.
- Wildcards (`*.css`) are allowed in ambient declarations but **not** in augmentations of existing modules.

**Constraints and limitations.**

- Augmentations cannot introduce new top-level names.
- Classes do not merge; use interfaces or namespaces.
- Invasive type changes require forking or upstream contribution.

### Annotated Complete Step by Step Code Examples

#### Example 1: Distinguishing augmentation from ambient declaration

```typescript
// ✅ Ambient declaration (global .d.ts, no imports/exports):
declare module "legacy-lib" {
  export function oldApi(): void;
}
```

```typescript
// ✅ Module augmentation (inside a module file):
import "legacy-lib";
declare module "legacy-lib" {
  interface OldApiOptions {
    newOption: boolean;
  }
}
```

**Expected behavior.** The first block **describes** `legacy-lib` for TypeScript. The second block **extends** an existing interface within `legacy-lib`.

**Why this output occurs.** The presence of `import "legacy-lib"` makes the file a module and ensures the `declare module` block **merges** rather than **replaces**.

---

### Real-World Cases

- **Describing untyped libraries**: `declare module "old-carousel-library"` in a global `.d.ts`.
- **Extending library interfaces**: `declare module "express-serve-static-core"` for `Request` fields.
- **Plugin type extensions**: `declare module "hardhat/types/network"` for `MyType`.

---

## 3. Augmentation Best Practices

### Definitions

**Core Definition.** **Augmentation best practices** are guidelines for writing module augmentations that are **reliable, non-destructive, and maintainable**, avoiding silent no-ops, accidental replacements, and runtime/type mismatches.

**Technical Definition.** Best practices center on **importing the target module first**, **augmenting the implementation module rather than a re-export**, **ensuring the augmentation file is a module**, **making middleware-attached properties optional**, **avoiding class augmentation** (classes do not merge), **keeping runtime patches aligned with type augmentations**, and **avoiding invasive changes** that augmentation cannot express.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to make module augmentation work reliably. The big rules are: import the module first; augment the right module; make added fields optional when middleware sets them later; and remember that types do not change runtime behavior.

### Purposes

- **To avoid silent no-ops** from augmenting the wrong module.
- **To prevent accidental replacement** of existing exports.
- **To keep type augmentations aligned with runtime behavior**.
- **To avoid lying to the compiler** about properties that may not exist yet.
- **To keep augmentations minimal and non-invasive**.

### Syntax Rules and Structure

Best practices are conventions that interact with `declare module` syntax. The core rules are:

1. **Import the target module first.** A side-effect import (`import "target"`) ensures the file is a module and the target is loaded before augmentation.
2. **Augment the implementation module, not the re-export.** Express's `Request` is declared in `express-serve-static-core`, not `express`. Augmenting the wrong module silently no-ops.
3. **Ensure the augmentation file is a module.** A bare `declare module` in a script file augments the global scope instead. Add `import "target"` or `export {}`.
4. **Make middleware-attached properties optional.** A required property set mid-pipeline makes early-pipeline code lie about its state. Use `?` or a separate post-middleware type.
5. **Do not augment classes directly.** Classes do not declaration-merge. Augment an interface or namespace instead.
6. **Keep runtime patches aligned with type augmentations.** Module augmentation is type-only; add the runtime property separately.
7. **Avoid invasive changes.** Augmentation can only add; it cannot change existing field types or remove fields. Fork the type or contribute upstream.
8. **Prefer module augmentation over global augmentation** when the target is a module, to avoid global scope pollution.

**Component breakdown (augmentation decision tree):**

```
Do you need to extend an existing module's types?
  ├── Yes → Import the target module first.
  │        ├── Is the type declared in a re-export? → Augment the implementation module.
  │        ├── Is the file a module? → Add `import "target"` or `export {}`.
  │        └── Is the added property set mid-pipeline? → Make it optional.
  └── No → Use an ambient declaration (`declare module`) instead.
```

**Constraints and limitations.**

- Augmentations cannot add new top-level global names.
- Classes cannot be augmented directly; use interface merging on the class's instance type.
- Runtime behavior must be patched separately; types alone do not change values.

### Annotated Complete Step by Step Code Examples

#### Example 1: Correct vs. incorrect augmentation target

```typescript
// ❌ Augmenting the re-export — silently no-ops.
// declare module "express" {
//   interface Request { userId?: string }
// }

// ✅ Augmenting the implementation module.
import "express";
declare module "express-serve-static-core" {
  interface Request {
    userId?: string;
  }
}
```

**Expected behavior.** The correct augmentation is visible on `req.userId` everywhere; the incorrect one is ignored.

**Why this output occurs.** Express re-exports `Request` from `express-serve-static-core`; only the implementation module's declarations merge.

---

#### Example 2: Aligning runtime and type augmentation

```typescript
// type-extensions.ts
import "hardhat/types/network";

declare module "hardhat/types/network" {
  interface NetworkConnection {
    myProp: string;
  }
}

// Runtime patch — must exist separately.
export function addMyProp(connection: any) {
  connection.myProp = "value";
}
```

**Expected behavior.** `NetworkConnection` has `myProp` at the type level, and the runtime function attaches it.

**Why this output occurs.** Type extensions are **exclusively type-level**; adding a property to a type does not add it to runtime values.

---

#### Example 3: Optional property for middleware

```typescript
import "express";

declare module "express-serve-static-core" {
  interface Request {
    userId?: string;       // ✅ Optional — set by auth middleware.
    requestId: string;     // ✅ Always present after tracing middleware.
  }
}
```

**Expected behavior.** `req.userId` is `string | undefined`; `req.requestId` is `string`.

**Why this output occurs.** Optional properties correctly model the pipeline state; a required `userId` would falsely imply it is always present.

---

### Real-World Cases

- **Express augmentation files** in `src/types/` — importing `express` and augmenting `express-serve-static-core`.
- **Hardhat plugin type extensions** — importing the target module and adding interfaces.
- **Angular permission narrowing** — importing the library before re-declaring `Permission`.
- **Code review checklists** — verifying the import is present, the implementation module is targeted, and added properties are optional.

---

## References

- TypeScript Handbook — Declaration Merging: Module Augmentation - https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation
- TypeScript Wiki — What's New in TypeScript: Module Augmentation and Global Augmentation - https://github.com/microsoft/TypeScript/wiki/What's-new-in-TypeScript
- TypeScript Documentation — Global: Modifying Module - https://www.typescriptlang.org/docs/handbook/declaration-files/templates/global-modifying-module-d-ts.html
- pproenca/dot-skills — Declare Module Augmentation (Three Reliable Rules) - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-advanced-patterns/references/decl-module-augmentation.md
- Hardhat — Type Extensions (Module Augmentation and Declaration Merging) - https://hardhat.org/plugin-development/explanations/type-extensions
- Logto Blog — TypeScript Module Augmentation and Handling Nested JavaScript Files - https://blog.logto.io/typescript-module-augmentation
- Stack Overflow — How to Properly Augment TypeScript Module Types Without Overriding Other Exports? - https://stackoverflow.com/questions/79857360/how-to-properly-augment-typescript-module-types-without-overriding-other-exports
- DefinitelyTyped — Augmentations Merging Discussion - https://github.com/DefinitelyTyped/DefinitelyTyped/discussions/73801