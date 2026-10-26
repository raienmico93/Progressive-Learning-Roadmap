# Parameter Decorators in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **parameter decorator** is a decorator applied to a parameter of a method or constructor within a class. It executes when the class is defined and is used **exclusively to record metadata** about the parameter. Unlike method, accessor, or Stage 3 field decorators, parameter decorators **cannot alter the behavior** of the parameter, the method, or the constructor. Parameter decorators exist **only in TypeScript’s legacy experimental decorator system** and are **not part of the TC39 Stage 3 decorators proposal**.

**Technical Definition.** In the legacy system, a parameter decorator has the signature `(target: any, propertyKey: string | symbol | undefined, parameterIndex: number) => void`. The `target` is the prototype for instance methods or the constructor for static methods and constructors. The `propertyKey` is the method name — or **`undefined`** when the decorated parameter belongs to a **constructor**. The `parameterIndex` is the zero-based position of the parameter in the parameter list. The decorator’s **return value is ignored**; it cannot return a descriptor or a replacement. Its sole purpose is to call `Reflect.defineMetadata` (or mutate an external registry) so that a separate runtime mechanism — typically a dependency-injection container or a validation framework — can consult the metadata later. The TC39 notes explicitly record that parameter decorators were part of the original 2013 design but were **left out of the Stage 3 proposal** due to complexities around hoisting and circularity, and that **constructor-parameter-based dependency injection** remains the most heavily used ecosystem case.

**Beginner-Friendly Explanation.** A parameter decorator is a **sticker you put on a function parameter** (like `@inject` or `@required`). It runs when the class is defined, but it **cannot change what the parameter does** — it can only **write down a note** about that parameter. A separate system (such as a dependency-injection container) reads those notes later and acts on them. Parameter decorators exist **only in the old experimental decorator system**. The new Stage 3 decorators **do not support them at all**; frameworks that need constructor parameter injection must stay on legacy decorators.

---

### Key Characteristics

| Characteristic | Parameter Decorators |
|---|---|
| **Availability** | **Legacy experimental decorators only**; not part of Stage 3 |
| **Compiler flag** | `experimentalDecorators: true` required |
| **Signature** | `(target: any, propertyKey: string \| symbol \| undefined, parameterIndex: number) => void` |
| **`propertyKey` for constructors** | **`undefined`** |
| **Return value** | **Ignored**; cannot modify behavior |
| **Sole mechanism** | Metadata recording (`Reflect.defineMetadata` or external registry) |
| **Stage 3 equivalent** | **None** (not supported) |
| **Primary ecosystem** | Angular, NestJS, Inversify, constructor-based dependency injection |
| **Common use cases** | Dependency injection, request parameter binding, argument validation, ORM entity construction |

---

### Prerequisites

- **Classes, constructors, and methods**: understanding how constructor parameters and method parameters are declared.
- **Legacy decorator fundamentals**: the `experimentalDecorators` flag and the distinction between decorator **evaluation** and **execution**.
- **`reflect-metadata`**: the standard library for storing and retrieving parameter metadata (`Reflect.defineMetadata`, `Reflect.getMetadata`).
- **`emitDecoratorMetadata`** (optional): the compiler flag that emits `design:paramtypes` metadata, used by dependency-injection frameworks.
- **A TypeScript environment** (`tsc`) with legacy decorators enabled.

---

### Related Programming Areas

- **Dependency injection (DI)**: constructor parameter decorators are the backbone of Angular, NestJS, and Inversify DI containers.
- **Request parameter binding**: frameworks use parameter decorators (`@Param()`, `@Query()`, `@Body()`) to bind HTTP request data to method arguments.
- **Argument validation**: parameter decorators record validation rules (e.g., `@required`, `@min`) that a separate validation pass enforces.
- **ORM entity construction**: TypeORM and similar systems use parameter metadata during entity hydration.
- **Type erasure**: parameter decorators are runtime constructs, but their type signatures are erased; only the decorator function survives compilation.

---

### Core Concepts / Features

The following four sections cover the requested concepts. Because parameter decorators exist **only in the legacy system**, the entire treatment concerns legacy experimental decorators, with explicit notes on the Stage 3 gap.

---

## 1. Parameter Decorators

### Definitions

**Core Definition.** **Parameter decorators** are decorator functions applied to method or constructor parameters. They execute at class definition time and **record metadata** about the parameter — its position, the method it belongs to, and any configuration supplied by a decorator factory.

**Technical Definition.** A parameter decorator is invoked with `(target, propertyKey, parameterIndex)`. It **cannot** return a descriptor, a replacement function, or any value that alters runtime behavior. Its only sanctioned mechanism is **metadata storage**: `Reflect.defineMetadata(key, value, target, propertyKey)` or mutation of an external registry keyed by the target and method. When `propertyKey` is `undefined`, the decorated parameter belongs to a **constructor**. Because the return value is ignored, parameter decorators are **observational only**; enforcement or transformation must be implemented by a separate runtime system that reads the recorded metadata. The TC39 committee documented that parameter decorators were intentionally excluded from the Stage 3 proposal, and that this exclusion is a **migration blocker** for large TypeScript projects that rely on constructor parameter injection.

**Beginner-Friendly Explanation.** A parameter decorator is a **note-taker for a parameter**. It runs when the class is defined and writes down information — “this parameter needs a database connection,” “this parameter is required,” “this parameter comes from the request body.” It **cannot make the parameter do anything different**; it can only leave a note. A separate system (like a DI container) reads the notes and supplies the right value at runtime. This is why parameter decorators are **only useful with a framework that reads metadata**.

### Purposes

- **To mark constructor parameters for dependency injection**, enabling DI containers to resolve and supply the correct dependencies.
- **To record validation rules** for method parameters (e.g., `@required`, `@minLength`) that a separate validation pass enforces.
- **To bind request data to method parameters** in web frameworks (e.g., `@Param()`, `@Query()`, `@Body()`).
- **To record parameter positions and configurations** for ORM hydration, marshaling, and serialization.
- **To attach arbitrary metadata** that a downstream runtime system can consult without requiring changes to the method body.

### Syntax Rules and Structure

**General syntax (legacy parameter decorator).**

```
function parameterDecorator(
  target: any,
  propertyKey: string | symbol | undefined,
  parameterIndex: number
): void {
  // record metadata; return value is ignored
}
```

**Component breakdown.**

- `target: any` — the **prototype** for instance methods, or the **constructor** for static methods and constructors.
- `propertyKey: string | symbol | undefined` — the method name; **`undefined`** for constructor parameters.
- `parameterIndex: number` — the **zero-based position** of the parameter in the parameter list.
- **No descriptor** is provided, and **the return value is ignored**.

**Syntax rules.**

- The `@` sigil precedes the decorator expression, placed **immediately before** the parameter declaration: `method(@decorator param: Type) { ... }`.
- Parameter decorators may be applied to **method parameters** and **constructor parameters**.
- Parameter decorators **cannot** be applied to **function declarations outside classes**, **arrow functions**, or **standalone functions**.
- Multiple parameter decorators may be applied to the **same parameter**; they execute **bottom-to-top** relative to their position.
- The decorator must be a **function** (or a factory returning a function) with the signature above.

**Constraints and limitations.**

- **Return value is ignored**: returning a `PropertyDescriptor` or a replacement function has no effect.
- **Cannot modify parameter behavior** directly; only metadata recording is supported.
- **`propertyKey` is `undefined` for constructor parameters**; code must handle this case explicitly.
- **Not available in Stage 3**: the `experimentalDecorators` flag must be enabled, and Stage 3 decorators **do not support parameter decoration**.
- **Metadata reflection requires `reflect-metadata`** and, for type metadata, `emitDecoratorMetadata: true`.

### Annotated Complete Step by Step Code Examples

#### Example 1: `@required` parameter decorator for metadata recording

```typescript
// example_required.ts
// Step 1: Import reflect-metadata for metadata storage.
import "reflect-metadata";

// Step 2: Define a parameter decorator factory.
function required(target: any, propertyKey: string | undefined, parameterIndex: number) {
  // Step 3: Retrieve existing required-parameter indices for this method.
  const existing: number[] =
    Reflect.getMetadata("required", target, propertyKey!) ?? [];

  // Step 4: Add the current parameter index.
  existing.push(parameterIndex);

  // Step 5: Store the updated list as metadata.
  Reflect.defineMetadata("required", existing, target, propertyKey!);
}

// Step 6: Apply the decorator to method parameters.
class UserService {
  createUser(@required name: string, age: number, @required email: string) {
    return { name, age, email };
  }
}

// Step 7: Read the metadata separately.
const requiredParams = Reflect.getMetadata(
  "required",
  UserService.prototype,
  "createUser"
);

console.log(requiredParams); // Expected: [ 0, 2 ]
```

**Setup.** Enable legacy decorators and metadata emission:
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```
Install `reflect-metadata` and import it before any decorated class.

**Expected Output:**
```
[ 0, 2 ]
```

**Why this output occurs.** Each `@required` decorator call records the **index** of the parameter it decorates. `name` is at index `0`, `age` at `1`, and `email` at `2`. The decorator pushes `0` and `2` into the metadata array. The return value is ignored; the decorator only **records** which parameters are required. A separate validation system would read this metadata and enforce the requirement.

---

#### Example 2: Constructor parameter decorator for dependency injection metadata

```typescript
// example_constructor_di.ts
// Step 1: Import reflect-metadata.
import "reflect-metadata";

// Step 2: Define a parameter decorator factory for injection tokens.
function inject(token: string) {
  return function (
    target: any,
    propertyKey: string | symbol | undefined,
    parameterIndex: number
  ): void {
    // Step 3: propertyKey is undefined for constructor parameters.
    console.log(
      `Injecting "${token}" into parameter ${parameterIndex} of ${target.name}`
    );

    // Step 4: Record the token against the constructor parameter index.
    const tokens = Reflect.getMetadata("inject:tokens", target) ?? {};
    tokens[parameterIndex] = token;
    Reflect.defineMetadata("inject:tokens", tokens, target);
  };
}

// Step 5: Apply the decorator to constructor parameters.
class DatabaseService {
  constructor(
    @inject("db") private db: string,
    @inject("logger") private logger: string
  ) {}
}

// Expected:
// Injecting "db" into parameter 0 of DatabaseService
// Injecting "logger" into parameter 1 of DatabaseService
```

**Expected Output:**
```
Injecting "db" into parameter 0 of DatabaseService
Injecting "logger" into parameter 1 of DatabaseService
```

**Why this output occurs.** For constructor parameters, `propertyKey` is `undefined`, and `target` is the class constructor itself. The decorator records the injection token for each parameter index in metadata on the constructor. A DI container would later read `inject:tokens` and resolve the dependencies accordingly. This is the canonical parameter-decorator pattern used by Angular, NestJS, and Inversify.

---

### Real-World Cases

- **Angular `@Inject()`**: constructor parameter decorator that specifies the injection token for a dependency, enabling Angular’s DI container to resolve it.
- **NestJS `@Body()`, `@Param()`, `@Query()`**: parameter decorators that bind HTTP request data to controller method arguments.
- **Inversify `@inject()`**: records the service identifier for constructor parameters, read by the Inversify container at resolution time.
- **class-validator `@Validate()`**: parameter decorator that records validation rules for method arguments, enforced by a separate validation pass.

---

## 2. Parameter Decorator Declaration

### Definitions

**Core Definition.** **Parameter decorator declaration** is the act of writing the decorator function — defining its parameters, return type, and metadata-recording behavior — before it is applied to a parameter with `@`.

**Technical Definition.** A parameter decorator declaration is a function `(target: any, propertyKey: string | symbol | undefined, parameterIndex: number) => void`. The declaration may be a **direct decorator** (used as-is) or a **decorator factory** (a function that accepts configuration and returns the decorator function). Because the return value is ignored, the declaration’s **return type is `void`** (or is simply not used). The declaration must handle **constructor parameters** where `propertyKey` is `undefined`, and must use `Reflect.defineMetadata` or an external registry to persist information.

**Beginner-Friendly Explanation.** Declaring a parameter decorator means **writing the function that will run when the parameter is decorated**. You decide what metadata to record and how to record it. If you want options (like an injection token or a validation rule), you write a **factory** — a function that takes the options and returns the decorator. The decorator’s job is simply to **write down information**; it cannot change the parameter’s behavior.

### Purposes

- **To define reusable parameter-level metadata recording** that can be applied across many methods and constructors.
- **To encapsulate configuration** through decorator factories, keeping call sites concise.
- **To handle the constructor-parameter case** (`propertyKey === undefined`) correctly in shared decorators.
- **To separate metadata-recording logic** from the classes and frameworks that consume it.

### Syntax Rules and Structure

**Legacy declaration (direct decorator).**

```
function decoratorName(
  target: any,
  propertyKey: string | symbol | undefined,
  parameterIndex: number
): void {
  // record metadata
}
```

**Legacy declaration (factory).**

```
function decoratorFactory(config: ConfigType) {
  return function (
    target: any,
    propertyKey: string | symbol | undefined,
    parameterIndex: number
  ): void {
    // use config and record metadata
  };
}
```

**Syntax rules.**

- The **third parameter** is always `parameterIndex: number`.
- The **second parameter** is `propertyKey`, which is **`undefined` for constructor parameters** and a `string | symbol` for method parameters.
- The **first parameter** is `target`: the prototype for instance methods, the constructor for static methods and constructors.
- The **return type is `void`**; returning a value has no effect.
- Factories return the decorator function; the factory itself is **not** the decorator.

**Constraints and limitations.**

- **Cannot return a descriptor or replacement**; the return value is discarded.
- **Must handle `propertyKey === undefined`** when the decorator is intended for constructors.
- **Metadata storage requires `reflect-metadata`** or an external registry; without it, the decorator’s observations are lost.
- **Factories should be pure** where possible; avoid shared mutable state across decorations.

### Annotated Complete Step by Step Code Examples

#### Example 1: Direct and factory parameter decorator declarations

```typescript
// example_declaration.ts
// Step 1: Direct parameter decorator that logs the parameter index.
function logIndex(
  target: any,
  propertyKey: string | symbol | undefined,
  parameterIndex: number
): void {
  console.log(`Parameter ${parameterIndex} of ${String(propertyKey)}`);
}

// Step 2: Factory parameter decorator with configuration.
function validate(min: number) {
  return function (
    target: any,
    propertyKey: string | symbol | undefined,
    parameterIndex: number
  ): void {
    const key = `validate:${String(propertyKey)}`;
    const rules = Reflect.getMetadata(key, target) ?? {};
    rules[parameterIndex] = { min };
    Reflect.defineMetadata(key, rules, target);
  };
}

// Step 3: Apply both.
class Example {
  method(@logIndex @validate(5) value: number) {
    return value;
  }
}

// Expected: Parameter 0 of method
```

**Expected Output:**
```
Parameter 0 of method
```

**Why this output occurs.** `@logIndex` is a **direct decorator** that logs the parameter index and method name. `@validate(5)` is a **factory** that records a `min: 5` rule for parameter `0` of `method`. Both return `void`; neither can change the method’s behavior. The decorators execute at class definition time, recording their observations.

---

#### Example 2: Constructor-aware parameter decorator

```typescript
// example_constructor_aware.ts
// Step 1: A parameter decorator that handles both methods and constructors.
function record(
  target: any,
  propertyKey: string | symbol | undefined,
  parameterIndex: number
): void {
  const location = propertyKey === undefined ? "constructor" : String(propertyKey);
  console.log(`Recorded parameter ${parameterIndex} of ${location}`);
}

// Step 2: Apply to a constructor parameter and a method parameter.
class Service {
  constructor(@record private name: string) {}

  greet(@record greeting: string) {
    return `${greeting}, ${this.name}`;
  }
}

// Expected:
// Recorded parameter 0 of constructor
// Recorded parameter 0 of greet
```

**Expected Output:**
```
Recorded parameter 0 of constructor
Recorded parameter 0 of greet
```

**Why this output occurs.** The decorator checks whether `propertyKey` is `undefined`. For the constructor parameter, it is `undefined`, so the decorator logs `"constructor"`. For the `greet` method parameter, `propertyKey` is `"greet"`. This pattern is essential for shared decorators that must handle both contexts.

---

### Real-World Cases

- **`@inject(token)` factory**: records an injection token for constructor parameters, read by a DI container.
- **`@required` direct decorator**: records that a parameter is required, read by a validation system.
- **`@Body()` / `@Query()` factories**: record request-binding configuration for controller method parameters, read by a routing framework.

---

## 3. Parameter Decorator Usage

### Definitions

**Core Definition.** **Parameter decorator usage** is the application of a declared parameter decorator to a method or constructor parameter via `@decoratorName` or `@decoratorFactory(args)`, triggering the decorator’s execution at class definition time.

**Technical Definition.** Usage consists of placing the `@` expression **immediately before** the parameter declaration inside the parameter list. The decorator expression is evaluated (including any factory call), and the resulting decorator function is invoked with `(target, propertyKey, parameterIndex)`. Parameter decorators may be stacked on the **same parameter**; they execute **bottom-to-top** relative to their position. The decorator’s return value is ignored, so usage is **purely observational** — the effect is entirely in the metadata it records.

**Beginner-Friendly Explanation.** Using a parameter decorator means **putting `@name` right before the parameter** inside the parentheses: `method(@inject("db") db: Database)`. When the class is defined, the decorator runs and records its metadata. You do not call it yourself. If it is a factory, you call the factory with parentheses (`@inject("db")`), and it returns the decorator that runs.

### Purposes

- **To apply reusable metadata recording** to method and constructor parameters.
- **To compose multiple decorators** on one parameter, each recording a different piece of metadata.
- **To configure decorators** through factory arguments at the usage site.
- **To declaratively attach injection, validation, or binding metadata** adjacent to the parameter declaration.

### Syntax Rules and Structure

**General usage syntax.**

```
methodName(@decorator paramName: Type) { ... }

methodName(@decoratorFactory(options) paramName: Type) { ... }

constructor(@first @second("option") paramName: Type) { ... }
```

**Component breakdown.**

- `@decoratorName` — direct decorator usage; the function is the decorator.
- `@decoratorFactory(options)` — factory usage; the factory is called, and its return value is the decorator.
- Stacking — multiple decorators on one parameter; execution is **bottom-to-top**.

**Syntax rules.**

- The decorator is placed **immediately before the parameter**, inside the parentheses.
- Decorator expressions are restricted to **identifiers, dotted names, and calls**.
- **Evaluation order**: top-to-bottom. **Execution order**: bottom-to-top.
- A parameter may have **multiple decorators**; each records its own metadata.
- Parameter decorators apply to **method parameters** and **constructor parameters** only.

**Constraints and limitations.**

- **Cannot be applied to standalone function parameters** outside a class.
- **Return value is ignored**; do not attempt to return a descriptor.
- **Legacy-only**: the `experimentalDecorators` flag must be enabled; Stage 3 does not support parameter decorators.
- **Metadata must be read by a separate system**; parameter decorators alone have no runtime effect.

### Annotated Complete Step by Step Code Examples

#### Example 1: Stacked parameter decorators

```typescript
// example_stacked.ts
// Step 1: Define two parameter decorators.
function first(target: any, key: string | undefined, index: number): void {
  console.log(`first: param ${index}`);
}

function second(target: any, key: string | undefined, index: number): void {
  console.log(`second: param ${index}`);
}

// Step 2: Stack them on one parameter.
class Example {
  method(@first @second value: string) {}
}

// Expected:
// second: param 0
// first: param 0
```

**Expected Output:**
```
second: param 0
first: param 0
```

**Why this output occurs.** `@second` is closest to the parameter, so it executes first. Then `@first` executes. Both receive the same `(target, propertyKey, parameterIndex)` arguments. The stacking order demonstrates bottom-to-top execution.

---

#### Example 2: Factory usage with different tokens

```typescript
// example_factory_usage.ts
// Step 1: Define a parameter decorator factory.
function inject(token: string) {
  return function (
    target: any,
    propertyKey: string | symbol | undefined,
    parameterIndex: number
  ): void {
    const tokens = Reflect.getMetadata("tokens", target) ?? {};
    tokens[parameterIndex] = token;
    Reflect.defineMetadata("tokens", tokens, target);
    console.log(`Token "${token}" registered for parameter ${parameterIndex}`);
  };
}

// Step 2: Apply with different tokens.
class App {
  constructor(@inject("db") db: string, @inject("logger") logger: string) {}
}

// Expected:
// Token "db" registered for parameter 0
// Token "logger" registered for parameter 1
```

**Expected Output:**
```
Token "db" registered for parameter 0
Token "logger" registered for parameter 1
```

**Why this output occurs.** Each `@inject(...)` call invokes the factory with a different token, producing an **independent decorator** that records its token for the parameter index it decorates. The metadata is stored on the constructor (`target`) under the `"tokens"` key, ready for a DI container to read.

---

### Real-World Cases

- **Angular**: `constructor(@Inject(HTTP) http: HttpClient)` — factory usage that specifies the injection token.
- **NestJS**: `getUser(@Param('id') id: string)` — factory usage that binds the route parameter `id` to the method argument.
- **Inversify**: `constructor(@inject('Weapon') weapon: Weapon)` — factory usage that registers the service identifier.

---

## 4. Parameter Decorator Best Practices

### Definitions

**Core Definition.** **Parameter decorator best practices** are guidelines for declaring and using parameter decorators in ways that preserve clarity, maintainability, and compatibility with the legacy decorator system — and that acknowledge the **fundamental limitation** that parameter decorators exist only in the legacy system.

**Technical Definition.** Best practices center on **accepting the legacy-only constraint** (Stage 3 does not support parameter decorators), **using parameter decorators for metadata only** (never attempting to modify behavior), **handling `propertyKey === undefined` for constructors**, **using `reflect-metadata` consistently**, **keeping decorators single-purpose and composable**, and **documenting the metadata contract** that downstream systems consume. Because parameter decorators cannot enforce anything by themselves, the best practice is to design them as part of a **two-phase system**: recording metadata at decoration time and enforcing it at runtime through a separate mechanism.

**Beginner-Friendly Explanation.** Best practices mean **understanding what parameter decorators can and cannot do**. They **cannot change behavior**; they can only **record notes**. They exist **only in the old system**, so if your project uses Stage 3 decorators, you cannot use them at all. Use them for **dependency injection, validation metadata, and request binding** — the things frameworks read later. Always handle the constructor case, keep each decorator focused on one piece of metadata, and document what the metadata means so the consuming system knows how to read it.

### Purposes

- **To avoid frustrated debugging** caused by expecting parameter decorators to alter behavior when they cannot.
- **To ensure correct handling of constructor parameters** where `propertyKey` is `undefined`.
- **To keep metadata contracts clear** so that DI containers, validators, and routers can reliably consume the recorded information.
- **To choose the right decorator system** — legacy for parameter decorators, Stage 3 for everything else where possible.

### Syntax Rules and Structure

**System selection checklist.**

```
Does the project require parameter decorators (DI, request binding, validation metadata)?
  ├── Yes → experimentalDecorators: true (legacy system)
  │         Use parameter decorators ONLY for metadata recording.
  │         Implement enforcement in a separate runtime system.
  └── No  → Stage 3 decorators (default in TS 5.0+)
            Parameter decorators are unavailable; use Stage 3 alternatives.
```

**Constructor-aware handling (mandatory).**

```
const key = propertyKey ?? "constructor";   // handle undefined for constructors
```

**Metadata recording (standard pattern).**

```
Reflect.defineMetadata(key, value, target, propertyKey);   // for methods
Reflect.defineMetadata(key, value, target);                // for constructors
```

**Constraints and limitations.**

- **Do not attempt to return a descriptor** from a parameter decorator; the return value is ignored.
- **Do not use parameter decorators in Stage 3 projects**; they are not supported.
- **Do not rely on parameter decorators to enforce anything** without a separate runtime enforcement mechanism.
- **Do not forget `reflect-metadata`**; without it, `Reflect.defineMetadata` is unavailable.

### Annotated Complete Step by Step Code Examples

#### Example 1: Correct metadata-only pattern with constructor handling

```typescript
// example_best_metadata.ts
// ✅ Parameter decorators record metadata; enforcement is separate.
import "reflect-metadata";

function required(
  target: any,
  propertyKey: string | symbol | undefined,
  parameterIndex: number
): void {
  const key = propertyKey ?? "constructor";
  const existing: number[] = Reflect.getMetadata("required", target, key) ?? [];
  existing.push(parameterIndex);
  Reflect.defineMetadata("required", existing, target, key);
}

class Service {
  constructor(@required private name: string) {}

  greet(@required greeting: string) {
    return `${greeting}, ${this.name}`;
  }
}

// Read the metadata (a separate validation system would do this).
console.log(Reflect.getMetadata("required", Service, "constructor")); // [ 0 ]
console.log(Reflect.getMetadata("required", Service.prototype, "greet")); // [ 0 ]
```

**Expected Output:**
```
[ 0 ]
[ 0 ]
```

**Why this output occurs.** The decorator handles the constructor case by substituting `"constructor"` for `undefined` `propertyKey`. It records parameter index `0` for both the constructor and the `greet` method. **No enforcement happens here**; the decorator only records. A separate validation system would read this metadata and enforce the requirement. This is the correct two-phase pattern.

---

#### Example 2: Correct DI metadata pattern

```typescript
// example_best_di.ts
// ✅ Constructor parameter decorators for DI metadata.
import "reflect-metadata";

function inject(token: string) {
  return function (
    target: any,
    propertyKey: string | symbol | undefined,
    parameterIndex: number
  ): void {
    // Constructor parameters have propertyKey === undefined.
    const tokens = Reflect.getMetadata("inject:tokens", target) ?? {};
    tokens[parameterIndex] = token;
    Reflect.defineMetadata("inject:tokens", tokens, target);
  };
}

class Database {
  constructor(@inject("db") private connection: string) {}
}

console.log(Reflect.getMetadata("inject:tokens", Database));
// Expected: { '0': 'db' }
```

**Expected Output:**
```
{ '0': 'db' }
```

**Why this output occurs.** The decorator records the token `"db"` for parameter index `0` on the `Database` constructor. A DI container would later read `inject:tokens` and resolve `"db"` to the appropriate connection. The decorator **does not create the connection**; it only records the metadata that tells the container what to inject.

---

#### Example 3: Single-responsibility parameter decorators

```typescript
// example_best_single.ts
// ✅ Each decorator records ONE piece of metadata.
import "reflect-metadata";

function required(target: any, key: string | undefined, index: number): void {
  const req = Reflect.getMetadata("required", target, key!) ?? [];
  req.push(index);
  Reflect.defineMetadata("required", req, target, key!);
}

function min(value: number) {
  return function (target: any, key: string | undefined, index: number): void {
    const rules = Reflect.getMetadata("min", target, key!) ?? {};
    rules[index] = value;
    Reflect.defineMetadata("min", rules, target, key!);
  };
}

class Form {
  submit(@required @min(3) name: string) {}
}

console.log(Reflect.getMetadata("required", Form.prototype, "submit")); // [ 0 ]
console.log(Reflect.getMetadata("min", Form.prototype, "submit"));      // { '0': 3 }
```

**Expected Output:**
```
[ 0 ]
{ '0': 3 }
```

**Why this output occurs.** `@required` records that parameter `0` is required. `@min(3)` records that parameter `0` has a minimum value of `3`. Each decorator records a **single, independent piece of metadata**. A validation system can read both and enforce them separately. This single-responsibility design makes the decorators composable and the metadata contracts clear.

---

### Real-World Cases

- **Angular, NestJS, and Inversify** rely on constructor parameter decorators for DI; these projects must keep `experimentalDecorators: true` and cannot migrate to Stage 3 until an equivalent parameter-decorator proposal advances.
- **class-validator and routing-controllers** use parameter decorators to record validation and binding metadata, enforced by separate runtime passes.
- **New projects without DI or parameter-level metadata requirements** should use Stage 3 decorators and avoid parameter decorators entirely, since they are not part of the standard.

---

## References

- Decorators — TypeScript (Advanced) - https://mintlify.wiki/Microsoft/typescript/advanced/decorators
- W3Schools — TypeScript Decorators (parameter decorator signature) - https://www.w3schools.com/typescript/typescript_decorators.php
- TC39 Notes — Parameter Decorators and the Stage 3 Gap - https://raw.githubusercontent.com/tc39/notes/d943c1fc75dd58129e968887fa64af1eddb855a4/meetings/2023-03/mar-23.md#8
- MikroORM Discussion — Stage 3 Decorators Do Not Allow Parameter Decoration - https://github.com/mikro-orm/mikro-orm/discussions/4148#1
- Parameter Decorators and Metadata — TypeScript Academy - https://www.coddykit.com
- TypeScript Handbook — Decorators (legacy) - https://www.typescriptlang.org/docs/handbook/decorators.html
- TC39 Proposal — Class Method Parameter Decorators (related proposal) - https://github.com/tc39/proposal-class-method-parameter-decorators