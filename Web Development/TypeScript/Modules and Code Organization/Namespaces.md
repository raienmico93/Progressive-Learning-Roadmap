# Namespaces in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A TypeScript **namespace** is a TypeScript-specific construct for organizing code by wrapping related declarations in a named, globally accessible JavaScript object. Namespaces were historically called **“internal modules”**; the terminology changed in TypeScript 1.5, and the `module X { ... }` form was superseded by the preferred `namespace X { ... }` form. Namespaces are **not** ECMAScript modules and exist outside the standard module system.

**Technical Definition.** A `namespace` declaration creates a **named value** in the global scope and a **namespace container** for nested types, values, and sub-namespaces. Members are accessible via **dotted notation** (`Namespace.Member`) only if marked `export`; unexported members are private to the namespace. Namespaces may **span multiple files** and be merged via declaration merging, and can be concatenated with the `outFile` compiler option. Because namespaces compile to global objects, they require either `<script>` inclusion or the `outFile` concatenation strategy, and are **not compatible with ES module semantics**.

**Beginner-Friendly Explanation.** A namespace is **a named box that groups related code together**. You put functions, classes, and interfaces inside the box, and you access them using the box’s name, like `Validation.EmailValidator`. Namespaces were TypeScript’s original way to organize code before JavaScript had standard modules. Today, **they are largely obsolete** — the TypeScript team recommends **ES modules instead** for new code, and Google’s TypeScript Style Guide **prohibits namespaces entirely** except when interfacing with third-party code.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **TypeScript-specific** | Namespaces are not part of ECMAScript; they exist only in TypeScript. |
| **Global object** | A namespace compiles to a **named JavaScript object in the global scope**. |
| **Dotted access** | Members are accessed as `Namespace.Member`. |
| **Explicit exports** | Only members marked `export` are visible outside the namespace. |
| **Multi-file merging** | Namespaces with the same name **merge** across files. |
| **`outFile` concatenation** | Multiple namespace files can be concatenated into one output file. |
| **No module isolation** | Namespaces pollute the global scope and make dependencies hard to track. |
| **Discouraged in modern code** | ES modules are the recommended mechanism; namespaces are legacy/architectural niche. |

---

### Prerequisites

- **TypeScript basics**: declarations, type annotations, and interfaces.
- **JavaScript scope and global objects**.
- **ES modules**: `import`/`export` syntax (for contrast).
- **`tsconfig.json`**: `outFile`, `module`, and compilation basics.
- **`/// <reference path="..." />`** syntax for multi-file namespaces.

---

### Related Programming Areas

- **ES Modules** — the modern, standard replacement for namespaces.
- **Declaration merging** — namespaces merge with classes, functions, enums, and other namespaces.
- **Ambient declarations** (`.d.ts`) — `declare namespace` describes global libraries accessed by dotted notation.
- **Legacy codebases** — older TypeScript projects that predate ES modules.
- **`outFile` concatenation** — the only build mode where namespaces work naturally.

---

## Core Concepts / Features

---

## 1. Namespaces

### Definitions

**Core Definition.** A **namespace** is a TypeScript construct that groups related types and values under a single global name, preventing naming collisions and providing a dotted access path.

**Technical Definition.** A namespace declaration creates a **namespace container** and a **global value** (the namespace object). Exported members become properties of that object; unexported members remain private. Namespaces merge when they share a name, and can span multiple files. They are **erased into global object assignments** in the emitted JavaScript.

**Beginner-Friendly Explanation.** A namespace is **a labeled container for related code**. Everything inside is accessed through the container’s name.

### Purposes

- **To group related types and values** under a single identifier and avoid global name collisions.
- **To control visibility** so only explicitly exported members are accessible externally.
- **To organize legacy code** that predates ES modules.
- **To describe global libraries** in ambient declaration files with dotted access.
- **To merge declarations** across multiple files when `outFile` concatenation is used.

### Syntax Rules and Structure

**General syntax:**

```typescript
namespace NamespaceName {
  export interface TypeName { ... }
  export class ClassName { ... }
  const privateValue = ...; // not exported
}
```

**Component breakdown:**

- `namespace` — declaration keyword (replaces the legacy `module` keyword for internal modules).
- `NamespaceName` — the global identifier.
- `export` — marks a member as externally visible.
- Non-exported members are private to the namespace.

**Syntax rules.**

- Only exported members are accessible via `Namespace.Member`.
- Namespaces may be nested: `namespace A { export namespace B { ... } }`.
- Same-named namespaces merge across declarations and files.
- Namespaces may contain types, values, functions, classes, enums, and other namespaces.

**Constraints and limitations.**

- Namespaces **pollute the global scope** and make dependencies hard to identify.
- They are **not compatible with ES module isolation**.
- Google’s TypeScript Style Guide **disallows** the `namespace Foo { ... }` construct except for third-party interop.
- Namespaces cannot be used with modern bundlers in the same way as modules.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic namespace with exported and private members

```typescript
// example_namespace_basic.ts
namespace Validation {
  // Exported interface — visible outside.
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }

  // Private implementation detail — not exported.
  const lettersRegexp = /^[A-Za-z]+$/;

  // Exported class.
  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
      return lettersRegexp.test(s);
    }
  }
}

// Usage: dotted access to exported members.
const validator = new Validation.LettersOnlyValidator();
console.log(validator.isAcceptable("Hello")); // Expected: true
console.log(validator.isAcceptable("123"));   // Expected: false

// Validation.lettersRegexp; // Error: not exported.
```

**Setup.** Compile with `npx tsc example_namespace_basic.ts --strict` and run `node example_namespace_basic.js`.

**Expected Output:**
```
true
false
```

**Why this output occurs.** `LettersOnlyValidator` is exported and accessible as `Validation.LettersOnlyValidator`. The regex `lettersRegexp` is unexported and therefore private. The namespace compiles to a global object assignment.

---

#### Example 2: Nested namespaces

```typescript
// example_namespace_nested.ts
namespace Shapes {
  export namespace Polygons {
    export class Triangle {
      area(base: number, height: number): number {
        return (base * height) / 2;
      }
    }
  }
}

const triangle = new Shapes.Polygons.Triangle();
console.log(triangle.area(4, 5)); // Expected: 10
```

**Expected Output:**
```
10
```

**Why this output occurs.** Nested namespaces create a hierarchy accessed by dotted notation. `Triangle` is exported from the inner namespace, which is itself exported from `Shapes`.

---

### Real-World Cases

- **Legacy TypeScript projects** organized before ES modules became standard.
- **Ambient declarations** for global libraries (`declare namespace jQuery`).
- **`outFile`-based builds** that concatenate namespace files into a single script.
- **Third-party interop** where a library requires namespace-style access.

---

## 2. `namespace`

### Definitions

**Core Definition.** The **`namespace` keyword** is the TypeScript token that declares a namespace. It replaced the older `module` keyword for internal modules in TypeScript 1.5.

**Technical Definition.** `namespace Name { ... }` introduces a **namespace container** and a **value** into the global scope. It is distinct from `module` (external modules) and from ambient `declare module`. The keyword may be combined with `export`, `declare`, and nesting.

**Beginner-Friendly Explanation.** `namespace` is **the word you type to create a namespace box**. It is the modern replacement for the old `module` keyword used for internal code organization.

### Purposes

- **To declare a namespace container** with a single, clear keyword.
- **To replace the legacy `module` keyword** for internal organization.
- **To introduce both a namespace and a global value** in one declaration.

### Syntax Rules and Structure

```typescript
namespace Name {
  // members
}
```

**Syntax rules.**

- `namespace` must be followed by an identifier and a block.
- `export` controls external visibility of members.
- `declare namespace` is used in ambient contexts.
- `export namespace` exports the namespace from a module.
- The legacy `module Name { ... }` form is equivalent but discouraged.

**Constraints and limitations.**

- Google Style Guide: `namespace Foo { ... }` is **disallowed** except for third-party interop.
- Namespaces are not modules; do not mix them with ES module semantics.

### Annotated Complete Step by Step Code Examples

#### Example 1: `namespace` keyword with export

```typescript
// example_namespace_keyword.ts
namespace Geometry {
  export function areaOfCircle(radius: number): number {
    return Math.PI * radius ** 2;
  }
}

console.log(Geometry.areaOfCircle(2).toFixed(2)); // Expected: 12.57
```

**Expected Output:**
```
12.57
```

**Why this output occurs.** `namespace` creates the `Geometry` global object; `export` makes `areaOfCircle` accessible via `Geometry.areaOfCircle`.

---

### Real-World Cases

- **Ambient library declarations**: `declare namespace Express { ... }`.
- **Legacy internal organization** in pre-ESM code.

---

## 3. Namespace Declaration

### Definitions

**Core Definition.** A **namespace declaration** is the complete statement that defines a namespace, including its name, body, and exported members.

**Technical Definition.** A namespace declaration introduces a **namespace entity** that participates in **declaration merging**. Multiple declarations with the same namespace name merge their exported members. Namespaces may merge with classes, functions, and enums to add static-side members or nested types. Multi-file namespaces require `/// <reference path="..." />` directives and are concatenated with `outFile`.

**Beginner-Friendly Explanation.** A namespace declaration is **the full statement that defines a namespace**. If you declare the same namespace twice, the declarations **merge** into one.

### Purposes

- **To define a named group** of related declarations.
- **To merge namespace declarations** across files or with classes/functions/enums.
- **To support multi-file organization** through namespace spanning.
- **To add static members or nested types** to an existing class or function via merging.

### Syntax Rules and Structure

```typescript
namespace Name {
  export const value = 1;
  export interface Shape { ... }
}
```

**Syntax rules.**

- Repeated namespace declarations with the same name **merge**.
- Namespaces may merge with classes, functions, and enums.
- Multi-file namespaces use `/// <reference path="..." />` and `outFile`.
- Unexported members do **not** merge across declarations.

**Constraints and limitations.**

- Merging is global and can cause **name collisions** if not controlled.
- Multi-file namespaces depend on `outFile` and reference directives, which are legacy mechanisms.

### Annotated Complete Step by Step Code Examples

#### Example 1: Namespace declaration merging

```typescript
// example_namespace_merge.ts
namespace Animals {
  export class Zebra {}
}

namespace Animals {
  export interface Legged {
    numberOfLegs: number;
  }
}

const zebra = new Animals.Zebra();
const legInfo: Animals.Legged = { numberOfLegs: 4 };
console.log(legInfo.numberOfLegs); // Expected: 4
```

**Expected Output:**
```
4
```

**Why this output occurs.** The two `Animals` namespace declarations merge, exposing both `Zebra` and `Legged` through `Animals`.

---

#### Example 2: Multi-file namespace spanning

```typescript
// file: validators/string-validator.ts
namespace Validation {
  export interface StringValidator {
    isValid(s: string): boolean;
  }
}
```

```typescript
// file: validators/email-validator.ts
/// <reference path="string-validator.ts" />
namespace Validation {
  export class EmailValidator implements StringValidator {
    isValid(s: string): boolean {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(s);
    }
  }
}
```

```typescript
// file: main.ts
/// <reference path="validators/email-validator.ts" />

const validator: Validation.StringValidator = new Validation.EmailValidator();
console.log(validator.isValid("ada@example.com")); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** The `/// <reference>` directives tell the compiler to include the other files, and the same-named `Validation` namespaces merge. Compilation with `outFile` concatenates them into one script.

---

### Real-World Cases

- **Ambient declaration merging**: augmenting `declare namespace Express` with custom properties.
- **Class + namespace merging**: adding static helpers to a class.
- **Legacy multi-file libraries**: concatenated with `outFile`.

---

## 4. Namespace Usage

### Definitions

**Core Definition.** **Namespace usage** refers to accessing namespace members through dotted notation (`Namespace.Member`) and organizing code within namespaces using `export`.

**Technical Definition.** Only `export`ed members are accessible outside the namespace. Access uses **dotted notation**. Namespaces may contain nested namespaces, and the emitted JavaScript contains a global object hierarchy. Namespaces are **not imported**; they are referenced globally, which requires the namespace to be loaded before use.

**Beginner-Friendly Explanation.** You use a namespace by **writing its name, a dot, and the member name** — like `Validation.EmailValidator`. Only exported members are reachable.

### Purposes

- **To access grouped declarations** through a qualified name.
- **To avoid naming collisions** by qualifying names with the namespace.
- **To control the public surface** of a namespace via `export`.
- **To organize large codebases** into hierarchical groups.

### Syntax Rules and Structure

```typescript
Namespace.Member
Namespace.SubNamespace.Member
```

**Syntax rules.**

- Dotted notation accesses exported members.
- Unexported members are inaccessible outside.
- Nested namespaces require dotted access at each level.
- Namespaces are global; no `import` is required (or possible) for plain namespaces.

**Constraints and limitations.**

- Namespaces require global loading order; using a namespace before it is defined causes runtime errors.
- Namespace access does not benefit from module isolation or tree-shaking.

### Annotated Complete Step by Step Code Examples

#### Example 1: Dotted access with nested namespaces

```typescript
// example_namespace_usage.ts
namespace App {
  export namespace Models {
    export class User {
      constructor(public name: string) {}
    }
  }
}

const user = new App.Models.User("Ada");
console.log(user.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `User` is exported from `Models`, which is exported from `App`. Dotted access follows the hierarchy.

---

### Real-World Cases

- **Global library access**: `$.ajax(...)` in ambient jQuery declarations.
- **Hierarchical code organization** in legacy projects.
- **OutFile bundles** where namespaces are loaded globally.

---

## 5. Namespace Best Practices

### Definitions

**Core Definition.** **Namespace best practices** are guidelines for using—or **avoiding**—namespaces in TypeScript, reflecting the modern consensus that **ES modules are the standard** and namespaces are a **legacy mechanism**.

**Technical Definition.** The TypeScript Handbook states: “for new projects modules would be the recommended code organization mechanism” and “we recommended modules over namespaces in modern code”. Google’s TypeScript Style Guide **prohibits** `namespace Foo { ... }` except when interfacing with external third-party code. Namespaces remain legitimate only for **ambient declarations** of global libraries and for **legacy code** that cannot be migrated. When namespaces are used, they should be **type-only** where possible, avoid runtime code, and never be mixed with ES modules in the same codebase.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to decide when—if ever—to use namespaces. The big rule is: **do not use namespaces in new code; use ES modules**. Namespaces are only acceptable for describing old global libraries or maintaining legacy projects that cannot be migrated.

### Purposes

- **To prefer ES modules** for all new code organization.
- **To restrict namespaces** to ambient declarations and unavoidable legacy interop.
- **To prevent global scope pollution** and improve tree-shaking.
- **To avoid mixing namespaces and modules**, which is a well-known bad practice.
- **To keep namespaces type-only** when they must exist, avoiding runtime code inside them.

### Syntax Rules and Structure

Best practices are conventions that interact with namespace syntax. The core rules are:

1. **Do not use `namespace` in new code.** Use ES modules (`import`/`export`) instead.
2. **Use namespaces only for ambient declarations** of global libraries (`declare namespace jQuery`).
3. **Never mix namespaces and ES modules** in the same codebase.
4. **Keep namespaces type-only** if they must exist; avoid runtime values, functions, and classes inside them.
5. **Use `outFile` only for legacy builds** that cannot adopt a bundler or ES modules.
6. **Migrate namespaces to modules incrementally**, replacing `namespace` with separate files and `import`/`export`.

**Component breakdown (namespace decision tree):**

```
Are you writing new application code?
  ├── Yes → Use ES modules (import/export). Do NOT use namespaces.
  └── No → Are you describing a global third-party library?
           ├── Yes → Use `declare namespace` in a .d.ts file.
           └── No → Are you maintaining legacy code that cannot be migrated?
                    ├── Yes → Keep namespaces but plan migration.
                    └── No → Do not use namespaces.
```

**Constraints and limitations.**

- Namespaces pollute the global scope and undermine module isolation.
- Namespaces are incompatible with modern bundler-based tree-shaking.
- Google Style Guide disallows them except for third-party interop.
- Mixing namespaces and modules causes confusion and dependency-tracking problems.

### Annotated Complete Step by Step Code Examples

#### Example 1: Preferred ES module instead of a namespace

```typescript
// ❌ Namespace: global, non-isolated, discouraged.
// namespace Validation {
//   export class EmailValidator { ... }
// }

// ✅ ES module: isolated, tree-shakeable, standard.
// file: validators/email-validator.ts
export class EmailValidator {
  isValid(s: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(s);
  }
}

// file: main.ts
import { EmailValidator } from "./validators/email-validator.js";
const validator = new EmailValidator();
console.log(validator.isValid("ada@example.com")); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** The ES module exports `EmailValidator` with a canonical name and isolates it from the global scope. It works with bundlers, supports tree-shaking, and requires no global load-order management.

---

#### Example 2: Legitimate ambient namespace declaration

```typescript
// example_ambient_namespace.d.ts
declare namespace MyLib {
  function makeGreeting(s: string): string;
  interface Options {
    verbose?: boolean;
  }
}
```

```typescript
// usage.ts
const greeting = MyLib.makeGreeting("Ada");
console.log(greeting); // Expected: Hello, Ada
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** `declare namespace` describes the shape of a global library that is loaded externally. This is one of the few legitimate remaining uses of namespaces.

---

### Real-World Cases

- **Ambient library declarations**: `declare namespace Express`, `declare namespace JQuery`.
- **Legacy codebases**: maintaining existing namespace-based code while migrating to modules.
- **OutFile builds**: rare legacy setups that concatenate namespace files.

---

## References

- TypeScript Handbook — Namespaces - https://www.typescriptlang.org/docs/handbook/namespaces.html
- TypeScript Handbook — Namespaces and Modules - https://www.typescriptlang.org/docs/handbook/namespaces-and-modules.html
- TypeScript Handbook — Declaration Merging (Merging Namespaces) - https://www.typescriptlang.org/docs/handbook/declaration-merging.html
- TypeScript Handbook — Declaration Reference (`declare namespace`) - https://www.typescriptlang.org/docs/handbook/declaration-files/by-example.html#reusable-types
- Google TypeScript Style Guide — Namespaces vs Modules (Namespaces Disallowed) - https://google.github.io/styleguide/tsguide.html
- TSConfig Reference — `outFile` - https://www.typescriptlang.org/tsconfig/outFile
- TypeScript Deep Dive — Namespaces - https://basarat.gitbook.io/typescript/main-1/namespaces
- Total TypeScript — Merging Namespaces in TypeScript - https://www.totaltypescript.com/workshops/typescript-pro-essentials/declaration-merging/merging-namespaces
- Compile-N-Run — TypeScript Namespaces (Syntax and Namespace Splitting) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/typescript/2-typescript-modules--namespaces/1-typescript-namespaces.mdx
- Microsoft Learn — Organize Code by Using Single-File Namespaces - https://learn.microsoft.com/en-us/training/modules/typescript-namespaces/