# Class Decorators in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **class decorator** is a special declaration applied to a class declaration using `@expression` syntax. The expression evaluates to a function that is invoked **at class definition time** — not at instantiation time — with information about the class. Class decorators may **observe, modify, or replace** the class definition. TypeScript supports two incompatible class-decorator systems: **legacy experimental decorators** (based on an early Stage 2 proposal, enabled by `experimentalDecorators: true`) and **Stage 3 decorators** (the TC39-standard system, default in TypeScript 5.0+).

**Technical Definition.** In the **legacy system**, a class decorator is a function with the signature `(constructor: Function) => Function | void`. It receives the class constructor as its sole argument. If it returns a value, that value **replaces** the class; if it returns `void`, the class is modified in place. In the **Stage 3 system**, a class decorator has the signature `(value: T, context: ClassDecoratorContext<T>) => T | void`, where `T` is a constructor type (`new (...args: any[]) => any`). The `context` object provides `kind: "class"`, `name`, and `addInitializer`. Returning a replacement class is permitted, but the replacement **must maintain the original prototype chain** and is subject to constructor-type constraints. In neither system can a class decorator alter the **static type structure** of the decorated class — added members are invisible to the type checker unless merged via an interface.

**Beginner-Friendly Explanation.** A class decorator is a **function you stick above a class with `@`** that runs when the class is defined. It receives the class itself (the constructor function) and can **seal it, log it, replace it, or wrap it**. It is like putting a tamper-proof seal on a machine: the machine is the class, and the decorator is the seal that runs at the factory (definition time), not each time the machine is used (instantiation). TypeScript has an **old style** (experimental, used by Angular and NestJS) and a **new standard style** (Stage 3, the default since TypeScript 5.0). They look the same but receive different arguments, and you cannot mix them.

---

### Key Characteristics

| Characteristic | Legacy (Experimental) | Stage 3 (Standard) |
|---|---|---|
| **Compiler flag** | `experimentalDecorators: true` required | No flag required in TS 5.0+ with `target: ES2022`+ |
| **Signature** | `(constructor: Function) => Function \| void` | `(value: T, context: ClassDecoratorContext<T>) => T \| void` |
| **Context object** | None (relies on `reflect-metadata` for metadata) | Typed `ClassDecoratorContext` with `kind`, `name`, `addInitializer` |
| **Return semantics** | Return a constructor to replace the class | Return a constructor to replace the class |
| **Constructor type constraint** | `Function` (broad) | `T extends abstract new (...args: any) => any` (precise) |
| **Prototype maintenance** | Must be done manually if returning a replacement | Must be done manually if returning a replacement |
| **Static type modification** | Not permitted | Not permitted |
| **Primary ecosystem** | Angular, NestJS, TypeORM, class-validator | New projects, modern libraries |

---

### Prerequisites

- **Classes and constructors**: understanding that a class is a constructor function at runtime.
- **Function types and generics**: `new (...args: any[]) => {}` constructor type syntax.
- **Basic decorator syntax** (`@expression`) and the distinction between decorator **evaluation** and **execution**.
- **A TypeScript environment** (`tsc` 5.0+ or the TypeScript Playground) and familiarity with `tsconfig.json`.

---

### Related Programming Areas

- **Aspect-oriented programming (AOP)**: class decorators inject cross-cutting concerns (logging, sealing, registration).
- **Dependency injection**: legacy class decorators plus `reflect-metadata` underpin Angular and NestJS DI containers.
- **Metadata-driven frameworks**: ORMs and validators use class decorators to register entities and rules.
- **Type erasure**: class decorators are runtime constructs, but their **type signatures** are erased; only the decorator function survives.
- **The decorator pattern**: class decorators are a native syntactic implementation of the classic wrapper pattern.

---

### Core Concepts / Features

The following four sections cover the requested concepts. Legacy and Stage 3 class decorators are treated together where their behavior aligns, and distinguished explicitly where they differ.

---

## 1. Class Decorators

### Definitions

**Core Definition.** **Class decorators** are decorator functions applied to a class declaration. They execute when the class is defined and may **observe, modify, or replace** the class constructor.

**Technical Definition.** A class decorator is invoked with the class constructor (legacy) or the constructor plus a `ClassDecoratorContext` (Stage 3). It may return a **new constructor** to replace the original class, or `void` to leave the class in place after in-place modification. Because TypeScript’s type system is erased, a class decorator **cannot change the static type** of the decorated class. If a decorator adds members at runtime, those members are invisible to the type checker unless the class is augmented through **declaration merging** (an `interface` with the same name). Returning a replacement class requires **maintaining the original prototype chain**, or existing prototype methods will be lost.

**Beginner-Friendly Explanation.** A class decorator is **a function that gets the class itself** when the class is defined. It can **seal the class** (prevent modification), **log when instances are created**, **register the class** in a registry, or **return a new class** that wraps the original. It cannot tell TypeScript’s type checker about new methods it adds — those additions are invisible unless you explicitly declare them. And if you return a new class, you must be careful to keep the original class’s prototype intact, or its methods will disappear.

### Purposes

- **To seal or freeze a class** and its prototype, preventing runtime modification and subclassing.
- **To register classes automatically** in a central registry (e.g., ORM entities, DI providers, route controllers).
- **To wrap or replace a class constructor** with augmented behavior (logging, singleton enforcement, validation).
- **To apply cross-cutting concerns at the class level** (e.g., marking a class as injectable, observable, or serializable).
- **To enforce constructor-level policies** (e.g., preventing direct instantiation, tracking instance counts).

### Syntax Rules and Structure

**General syntax (legacy class decorator).**

```
function ClassDecoratorName(constructor: Function): Function | void
```

**Component breakdown.**

- `constructor: Function` — the class constructor; the **only** argument in the legacy system.
- Returning a `Function` — **replaces** the class with the returned constructor.
- Returning `void` — leaves the class in place; modify it in-place instead.

**General syntax (Stage 3 class decorator).**

```
function ClassDecoratorName<T extends abstract new (...args: any) => any>(
  value: T,
  context: ClassDecoratorContext<T>
): T | void
```

**Component breakdown.**

- `value: T` — the class constructor, typed as a constructor function.
- `context: ClassDecoratorContext<T>` — metadata: `kind: "class"`, `name`, `addInitializer`.
- Returning `T` — **replaces** the class; the returned constructor must be assignable to `T`.
- Returning `void` — leaves the class in place.

**Syntax rules.**

- The `@` sigil precedes the decorator expression, placed **immediately before** the class declaration.
- Multiple class decorators may be stacked; **execution order is bottom-to-top** (the decorator closest to the class runs first).
- Legacy class decorators are **called after** method, accessor, and property decorators have been applied.
- Stage 3 class decorators receive a **typed context**; legacy class decorators do not.
- A class decorator **cannot** be applied to an interface, a type alias, or a standalone function.

**Constraints and limitations.**

- **Static type structure cannot be changed.** Adding methods at runtime does not make them visible to the type checker; use interface merging to declare them.
- **Prototype maintenance is mandatory** when returning a replacement class. TypeScript will not preserve the original prototype for you.
- **Constructor constraint.** In Stage 3, the returned constructor must be assignable to the original constructor type `T`; returning a structurally broader constructor may fail type checking.
- **Legacy and Stage 3 systems are incompatible**; the `experimentalDecorators` flag selects one and disables the other.
- **Class decorators do not run at instantiation time**; they run once, when the class definition is evaluated.

### Annotated Complete Step by Step Code Examples

#### Example 1: Legacy `@sealed` class decorator

```typescript
// example_legacy_sealed.ts
// Step 1: Define a legacy class decorator.
// It receives the constructor and seals it and its prototype.
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

// Step 2: Apply the decorator to a class.
@sealed
class BugReport {
  type = "report";
  title: string;
  constructor(title: string) {
    this.title = title;
  }
}

// Step 3: Verify sealing.
console.log(Object.isSealed(BugReport));          // Expected: true
console.log(Object.isSealed(BugReport.prototype)); // Expected: true

// Step 4: Attempt to modify the prototype (will fail silently or throw in strict mode).
(BugReport.prototype as any).newMethod = () => {};
console.log((BugReport.prototype as any).newMethod); // Expected: undefined
```

**Setup.** Enable legacy decorators:
```json
{ "compilerOptions": { "experimentalDecorators": true } }
```
Compile with `npx tsc` and run with Node.js.

**Expected Output:**
```
true
true
undefined
```

**Why this output occurs.** `@sealed` receives the `BugReport` constructor and calls `Object.seal` on both the constructor and its prototype. Sealing prevents adding or removing properties. The attempted `newMethod` assignment fails because the prototype is sealed, so the property remains `undefined`. The decorator returns `void`, so the class itself is not replaced.

---

#### Example 2: Stage 3 `@logged` class decorator with a replacement class

```typescript
// example_stage3_logged.ts
// Step 1: Define a Stage 3 class decorator.
// It receives the constructor and a typed context, and returns a replacement class.
function logged<T extends new (...args: any[]) => any>(
  target: T,
  context: ClassDecoratorContext<T>
) {
  return class extends target {
    constructor(...args: any[]) {
      super(...args);
      console.log(`[LOG] Instance of ${context.name} created`);
    }
  };
}

// Step 2: Apply it to a class.
@logged
class UserService {
  constructor(public readonly name: string) {}
}

// Step 3: Instantiate the class.
const service = new UserService("Alice");
// Expected: [LOG] Instance of UserService created
console.log(service.name); // Expected: Alice
```

**Setup.** No special flag is needed in TypeScript 5.0+ with `target: ES2022` or `ESNext`.

**Expected Output:**
```
[LOG] Instance of UserService created
Alice
```

**Why this output occurs.** The decorator returns a **subclass** of the original `UserService`. The subclass’s constructor calls `super(...args)` to preserve the original initialization, then logs the class name from `context.name`. Because the subclass extends the original, the prototype chain is maintained and `name` is accessible. The returned class replaces the original at the definition site.

---

#### Example 3: Singleton class decorator (Stage 3)

```typescript
// example_stage3_singleton.ts
// Step 1: Define a Stage 3 singleton decorator.
function singleton<T extends new (...args: any[]) => any>(
  target: T,
  _context: ClassDecoratorContext<T>
) {
  let instance: InstanceType<T> | undefined;
  return class extends target {
    constructor(...args: any[]) {
      if (instance) return instance;   // return the cached instance
      super(...args);
      instance = this as InstanceType<T>;
    }
  } as T;
}

// Step 2: Apply it.
@singleton
class DatabaseConnection {
  constructor(public readonly url: string) {}
}

// Step 3: Instantiate twice.
const a = new DatabaseConnection("postgres://localhost/a");
const b = new DatabaseConnection("postgres://localhost/b");
console.log(a === b);        // Expected: true
console.log(a.url);          // Expected: postgres://localhost/a
```

**Expected Output:**
```
true
postgres://localhost/a
```

**Why this output occurs.** The decorator returns a subclass that caches the first instance. When `new DatabaseConnection(...)` is called the second time, the subclass constructor returns the cached `instance` instead of creating a new object. The `as T` assertion is necessary because the subclass constructor’s return type is not automatically inferred as `InstanceType<T>`. The second URL argument is ignored because the cached instance is returned.

---

### Real-World Cases

- **`@Component`, `@Injectable`, `@NgModule`** in Angular: legacy class decorators register components and services with the framework’s metadata system.
- **`@Controller`, `@Module`** in NestJS: legacy class decorators define routing and dependency-injection boundaries.
- **`@Entity`** in TypeORM: a legacy class decorator registers a class as a database entity.
- **`@singleton`** as a Stage 3 class decorator: enforces single-instance semantics for connection pools, configuration objects, and registries without relying on legacy metadata.

---

## 2. Class Decorator Declaration

### Definitions

**Core Definition.** **Class decorator declaration** is the act of writing the decorator function — defining its parameters, its return type, and its behavior — before it is applied to a class with `@`.

**Technical Definition.** In the **legacy system**, a class decorator declaration is a function `(constructor: Function) => Function | void`. In the **Stage 3 system**, it is a generic function `(value: T, context: ClassDecoratorContext<T>) => T | void`, where `T` is constrained to a constructor type. The declaration may be a **direct decorator** (a function used as-is) or a **decorator factory** (a function that returns a decorator function, allowing configuration through arguments).

**Beginner-Friendly Explanation.** Declaring a class decorator means **writing the function that will run when the class is defined**. You decide what arguments it receives, what it does, and whether it returns a new class or nothing. If you want to pass options (like a log prefix), you write a **factory** — a function that takes the options and returns the actual decorator.

### Purposes

- **To define reusable class-level behavior** that can be applied to many classes.
- **To encapsulate configuration** through decorator factories, keeping call sites concise.
- **To type the decorator precisely** so the compiler can check its compatibility with the decorated class.
- **To separate the decorator’s logic** from the classes it decorates, preserving single responsibility.

### Syntax Rules and Structure

**Legacy declaration (direct decorator).**

```
function decoratorName(constructor: Function): Function | void {
  // observe or modify constructor
}
```

**Legacy declaration (factory).**

```
function decoratorFactory(config: ConfigType) {
  return function (constructor: Function): Function | void {
    // use config
  };
}
```

**Stage 3 declaration (direct decorator).**

```
function decoratorName<T extends abstract new (...args: any) => any>(
  value: T,
  context: ClassDecoratorContext<T>
): T | void {
  // observe or replace
}
```

**Stage 3 declaration (factory).**

```
function decoratorFactory(config: ConfigType) {
  return function <T extends abstract new (...args: any) => any>(
    value: T,
    context: ClassDecoratorContext<T>
  ): T | void {
    // use config
  };
}
```

**Syntax rules.**

- The **constructor type constraint** `T extends abstract new (...args: any) => any` (or `new (...args: any[]) => any`) ensures `T` is a class constructor.
- `abstract new` is used when the decorator may be applied to abstract classes; `new` suffices for concrete classes.
- The `context` parameter may be named `_context` if unused; TypeScript does not require its use.
- Factory declarations return a decorator function; the factory itself is **not** the decorator.
- Legacy decorators receive `Function`; Stage 3 decorators receive the precise constructor type `T`.

**Constraints and limitations.**

- **Legacy `Function` type is too broad**: it permits non-constructor values and provides no type safety for constructor arguments or instance type.
- **Stage 3 constructor constraints** can be restrictive when returning a subclass; `as T` assertions are sometimes necessary.
- **Decorator factories should be pure** where possible; avoid shared mutable state across decorations.
- **Return type must be compatible** with the original constructor in Stage 3; returning an unrelated class fails type checking.

### Annotated Complete Step by Step Code Examples

#### Example 1: Legacy declaration — direct and factory

```typescript
// example_legacy_declaration.ts
// Step 1: Direct legacy class decorator.
function freeze(constructor: Function) {
  Object.freeze(constructor);
  Object.freeze(constructor.prototype);
}

// Step 2: Legacy decorator factory with configuration.
function tagged(version: string) {
  return function (constructor: Function) {
    (constructor as any).__version = version;
  };
}

// Step 3: Apply both.
@freeze
@tagged("1.0.0")
class Config {
  value = 42;
}

console.log((Config as any).__version); // Expected: 1.0.0
console.log(Object.isFrozen(Config));    // Expected: true
```

**Expected Output:**
```
1.0.0
true
```

**Why this output occurs.** `@tagged("1.0.0")` is a **factory**: it returns a decorator that attaches `__version` to the constructor. `@freeze` is a **direct decorator** that freezes the constructor and prototype. Because decorators execute **bottom-to-top**, `tagged` runs first (attaching the version), then `freeze` runs (freezing the constructor). The version property is set before freezing, so it survives.

---

#### Example 2: Stage 3 declaration with typed context

```typescript
// example_stage3_declaration.ts
// Step 1: Direct Stage 3 class decorator with precise typing.
function countable<T extends new (...args: any[]) => any>(
  target: T,
  context: ClassDecoratorContext<T>
) {
  let count = 0;
  context.addInitializer(function () {
    count++;
    console.log(`[${context.name}] initializer ran (count=${count})`);
  });
  return class extends target {
    constructor(...args: any[]) {
      super(...args);
      count++;
      console.log(`[${context.name}] instance created (count=${count})`);
    }
  };
}

// Step 2: Apply it.
@countable
class Widget {
  constructor(public id: number) {}
}

// Step 3: Instantiate twice.
new Widget(1);
new Widget(2);
// Expected:
// [Widget] initializer ran (count=1)
// [Widget] instance created (count=2)
// [Widget] instance created (count=3)
```

**Expected Output:**
```
[Widget] initializer ran (count=1)
[Widget] instance created (count=2)
[Widget] instance created (count=3)
```

**Why this output occurs.** `context.addInitializer` registers a callback that runs **after the class is defined** (before the first instance is created). The initializer increments the count to 1. Each constructor call increments it further. The initializer runs **once**, while the constructor runs per instance.

---

### Real-World Cases

- **Angular `@Component` factory**: `@Component({ selector, template })` is a factory that returns a class decorator configuring the component’s metadata.
- **NestJS `@Module` factory**: `@Module({ imports, controllers, providers })` configures module boundaries through a decorator factory.
- **Stage 3 `@registered` factory**: a factory that takes a registry key and returns a class decorator that registers the class in a central map.

---

## 3. Class Decorator Usage

### Definitions

**Core Definition.** **Class decorator usage** is the application of a declared class decorator to a class via `@decoratorName` or `@decoratorFactory(args)`, triggering the decorator’s execution at class definition time.

**Technical Definition.** Usage consists of placing the `@` expression **immediately before** a class declaration. The decorator expression is evaluated (including any factory call), and the resulting decorator function is invoked with the class constructor (legacy) or the constructor and context (Stage 3). Multiple decorators may be stacked; their **evaluation order** is top-to-bottom, but their **execution order** is bottom-to-top.

**Beginner-Friendly Explanation.** Using a class decorator means **putting `@name` right above the class**. When the class is defined, the decorator runs automatically. You do not call it yourself. If the decorator is a factory, you call the factory with parentheses (`@name(options)`), and it returns the actual decorator that runs.

### Purposes

- **To apply reusable behavior** to a class without modifying its body.
- **To compose multiple decorators** on one class, each handling a distinct concern.
- **To configure decorators** through factory arguments at the usage site.
- **To declaratively register or mark classes** for frameworks and registries.

### Syntax Rules and Structure

**General usage syntax.**

```
@decoratorName
class ClassName { ... }

@decoratorFactory(options)
class ClassName { ... }

@first
@second("option")
class ClassName { ... }
```

**Component breakdown.**

- `@decoratorName` — direct decorator usage; the function is the decorator.
- `@decoratorFactory(options)` — factory usage; the factory is called, and its return value is the decorator.
- Stacking — multiple decorators on separate lines or the same line; execution is bottom-to-top.

**Syntax rules.**

- The decorator must be **immediately above** the class declaration (or on the same line, before `class`).
- Decorator expressions are restricted to **identifiers, dotted names, and calls**; arbitrary expressions are not permitted.
- **Evaluation order**: top-to-bottom. **Execution order**: bottom-to-top (the decorator closest to the class runs first).
- A class may have **multiple decorators**; each receives the class (or the result of the decorator below it).
- Decorators **cannot** be applied to a class expression used as a value; they apply to class **declarations**.

**Constraints and limitations.**

- **Decorator factories must be called** with parentheses; `@factory` without `()` passes the factory itself as the decorator, which is usually incorrect.
- **Execution order matters**: a decorator that freezes the class should run **last** (be closest to the class) if earlier decorators need to modify it.
- **Legacy and Stage 3 usage is syntactically identical** but semantically incompatible; the same `@` syntax selects different behavior based on the compiler flag.
- **Type errors are not caught at the usage site** if the decorator’s signature is too broad; precise typing in the declaration is essential.

### Annotated Complete Step by Step Code Examples

#### Example 1: Stacked decorators and execution order

```typescript
// example_usage_stacked.ts
// Step 1: Define two legacy class decorators.
function first(constructor: Function) {
  console.log("first executed");
  (constructor as any).order = "first";
}

function second(constructor: Function) {
  console.log("second executed");
  (constructor as any).order = "second";
}

// Step 2: Stack them (evaluation top-to-bottom, execution bottom-to-top).
@first
@second
class Ordered {}

// Expected:
// second executed
// first executed
console.log((Ordered as any).order); // Expected: first
```

**Expected Output:**
```
second executed
first executed
first
```

**Why this output occurs.** The decorators are **evaluated** top-to-bottom (`first` is parsed first, then `second`), but they are **executed** bottom-to-top: `second` runs first, then `first`. The last decorator to execute (`first`) sets `order` to `"first"`, overwriting the value set by `second`.

---

#### Example 2: Factory usage with configuration

```typescript
// example_usage_factory.ts
// Step 1: Define a Stage 3 decorator factory.
function labeled(label: string) {
  return function <T extends new (...args: any[]) => any>(
    target: T,
    context: ClassDecoratorContext<T>
  ) {
    (target as any).__label = label;
    console.log(`Labeled ${context.name} as "${label}"`);
  };
}

// Step 2: Use the factory with an argument.
@labeled("service")
class UserService {}

@labeled("repository")
class UserRepository {}

// Expected:
// Labeled UserService as "service"
// Labeled UserRepository as "repository"
```

**Expected Output:**
```
Labeled UserService as "service"
Labeled UserRepository as "repository"
```

**Why this output occurs.** `@labeled("service")` calls the factory with `"service"`, which returns the actual decorator. That decorator receives the class and its context, attaches `__label`, and logs the class name from `context.name`. Each usage creates an independent decorator with its own captured configuration.

---

### Real-World Cases

- **Angular templates**: `@Component({ selector: "app-root", template: "..." })` — a factory usage that configures component metadata.
- **NestJS modules**: `@Module({ imports: [...], providers: [...] })` — factory usage that defines dependency-injection boundaries.
- **TypeORM entities**: `@Entity("users")` — factory usage that specifies the database table name.
- **Stage 3 custom registration**: `@registered("plugin")` — factory usage that registers a plugin class in a central registry.

---

## 4. Class Decorator Best Practices

### Definitions

**Core Definition.** **Class decorator best practices** are guidelines for declaring and using class decorators in ways that preserve type safety, maintainability, composability, and compatibility with the chosen decorator system.

**Technical Definition.** Best practices center on **precise typing** (using constructor-constrained generics in Stage 3, avoiding broad `Function` where possible), **prototype preservation** when returning replacement classes, **interface merging** when decorators add members, **single responsibility** (one concern per decorator), **predictable composition** (awareness of bottom-to-top execution), and **system selection** (legacy for Angular/NestJS/TypeORM dependencies, Stage 3 for new projects). Decorators should be **pure** where possible and should not rely on hidden shared state.

**Beginner-Friendly Explanation.** Best practices mean **writing class decorators that are safe and easy to understand**. Use precise types instead of `Function`. If your decorator adds a method at runtime, declare it with an interface so TypeScript knows about it. Keep each decorator focused on one job. Remember that decorators run bottom-to-top when stacked. And choose the right system: legacy if you depend on Angular or NestJS, Stage 3 for new code.

### Purposes

- **To preserve type safety** by using constructor-constrained generics and avoiding `any`.
- **To prevent runtime breakage** by maintaining the prototype chain when replacing classes.
- **To make added members visible to the type checker** through interface merging.
- **To keep decorators composable and predictable** by following execution-order rules and single responsibility.
- **To choose a future-proof decorator system** aligned with the project’s ecosystem dependencies.

### Syntax Rules and Structure

**Precise Stage 3 typing (recommended).**

```
function decorator<T extends abstract new (...args: any) => any>(
  value: T,
  context: ClassDecoratorContext<T>
): T | void
```

**Interface merging for added members (recommended).**

```
interface MyClass {
  addedMethod(): void;   // declared, implemented at runtime by decorator
}
```

**Prototype preservation when returning a replacement.**

```
return class extends target {
  constructor(...args: any[]) {
    super(...args);   // preserves prototype chain
  }
};
```

**System selection checklist.**

```
Depends on Angular, NestJS, TypeORM, or class-validator?
  ├── Yes → experimentalDecorators: true (legacy system)
  └── No  → Stage 3 decorators (default in TS 5.0+)
```

**Constraints and limitations.**

- **Do not mix systems**; the `experimentalDecorators` flag selects one and disables the other.
- **Do not return an unrelated class**; the returned constructor must be assignable to the original constructor type in Stage 3.
- **Do not rely on decorator-added members without interface merging**; the type checker will not see them.
- **Do not freeze before other decorators run**; freeze should be the last-executing decorator (closest to the class).

### Annotated Complete Step by Step Code Examples

#### Example 1: Precise typing and interface merging (Stage 3)

```typescript
// example_best_interface_merge.ts
// Step 1: Define a decorator that adds a method at runtime.
function withGreet<T extends new (...args: any[]) => any>(
  target: T,
  _context: ClassDecoratorContext<T>
) {
  (target.prototype as any).greet = function () {
    return "hello";
  };
}

// Step 2: Declare the added member via interface merging.
interface Greeter {
  greet(): string;
}

// Step 3: Apply the decorator and merge the interface.
@withGreet
class Greeter {}

// Step 4: Now the type checker sees greet().
const g = new Greeter();
console.log(g.greet()); // Expected: hello
```

**Expected Output:**
```
hello
```

**Why this output occurs.** Without the interface merging, `g.greet()` would be a compile-time error because the decorator adds `greet` at runtime but does not change the static type. The `interface Greeter { greet(): string; }` declaration **merges** with the class type, making the added member visible. This is the recommended pattern for decorators that augment classes.

---

#### Example 2: Prototype preservation when replacing a class

```typescript
// example_best_prototype.ts
// Step 1: A decorator that returns a replacement class.
function timed<T extends new (...args: any[]) => any>(
  target: T,
  context: ClassDecoratorContext<T>
) {
  return class extends target {
    constructor(...args: any[]) {
      super(...args);   // MUST call super to preserve prototype chain
      console.log(`[${context.name}] constructed at ${Date.now()}`);
    }
  };
}

// Step 2: Apply to a class with a prototype method.
@timed
class Report {
  generate() {
    return "report content";
  }
}

// Step 3: Verify the prototype method still works.
const r = new Report();
// Expected: [Report] constructed at <timestamp>
console.log(r.generate()); // Expected: report content
```

**Expected Output:**
```
[Report] constructed at <timestamp>
report content
```

**Why this output occurs.** The decorator returns a subclass that extends the original `Report`. Calling `super(...args)` preserves the original constructor’s initialization and, critically, maintains the prototype chain so that `generate` remains available on instances. Omitting `super` would break the prototype chain and cause `r.generate` to be `undefined`.

---

#### Example 3: Single-responsibility decorators in composition

```typescript
// example_best_single.ts
// ✅ Each decorator does ONE thing.
function frozen(constructor: Function) {
  Object.freeze(constructor);
  Object.freeze(constructor.prototype);
}

function tagged(version: string) {
  return function (constructor: Function) {
    (constructor as any).__version = version;
  };
}

// Correct order: tagged runs first (closer to class), frozen runs last.
@frozen
@tagged("2.0.0")
class Settings {
  theme = "dark";
}

console.log((Settings as any).__version); // Expected: 2.0.0
console.log(Object.isFrozen(Settings));    // Expected: true
```

**Expected Output:**
```
2.0.0
true
```

**Why this output occurs.** `@tagged("2.0.0")` attaches the version before `@frozen` freezes the constructor. Because execution is bottom-to-top, `tagged` runs first, then `frozen`. If the order were reversed (`@tagged` above `@frozen`), `frozen` would run first, and `tagged`’s attempt to add `__version` would fail because the constructor would already be frozen.

---

### Real-World Cases

- **Angular and NestJS codebases** correctly select legacy decorators and use precise typing through framework-provided decorator types.
- **TypeORM entities** use factory class decorators (`@Entity("table")`) with interface merging for relations and columns that are added via metadata.
- **Stage 3 singleton and logging decorators** use constructor-constrained generics and return subclasses with preserved prototypes, avoiding `Function` and manual prototype fixes.
- **Libraries targeting both ecosystems** publish separate entry points (e.g., legacy vs. standard decorators) rather than attempting to unify incompatible systems.

---

## References

- TypeScript Documentation — Decorators - https://www.typescriptlang.org/docs/handbook/decorators.html
- TypeScript 5.0 Release Notes — Decorators - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html
- TypeScript Advanced — Decorators (Stage 3 vs. Legacy) - https://mintlify.wiki/Microsoft/typescript/advanced/decorators
- TC39 Decorators Proposal (Stage 3) - https://github.com/tc39/proposal-decorators
- Refine — TypeScript Decorators in Brief (Best Practices) - https://refine.dev/blog/typescript-decorators/
- TypeScript 5.x Features — Standard Decorators - https://www.gaultonlab.org/posts/typescript-5-new-features/
- TypeScript Decorators in 2026 — Old vs. New - https://blog.codercops.com/blog/typescript-decorators-stable-2026
- Microsoft/TypeScript Issue #58022 — Constructor Constraint Limitations - https://github.com/microsoft/TypeScript/issues/58022
- Stack Overflow — Returning a New Constructor Must Maintain Prototype - https://stackoverflow.com/revisions/72bb6c37-ba56-407e-8581-6d7d6949074a/view-source