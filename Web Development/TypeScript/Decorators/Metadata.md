# Decorator Metadata in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Decorator metadata** is a runtime reflection mechanism that allows decorators to attach and retrieve arbitrary information about classes, methods, properties, and parameters. In TypeScript, this capability is provided by two cooperating pieces: the compiler flag `emitDecoratorMetadata`, which causes the compiler to emit **design-time type metadata** (`design:type`, `design:paramtypes`, `design:returntype`) for decorated declarations, and the **`reflect-metadata`** polyfill, which supplies the `Reflect.defineMetadata` and `Reflect.getMetadata` APIs used to read and write metadata at runtime.

**Technical Definition.** Decorator metadata is a **side-channel information store** keyed by a metadata key, a target object, and an optional property key. The `reflect-metadata` library implements the ECMAScript Metadata Reflection API proposal, extending the global `Reflect` object with methods such as `defineMetadata`, `getMetadata`, `getOwnMetadata`, `hasMetadata`, `hasOwnMetadata`, `deleteMetadata`, `getMetadataKeys`, and `getOwnMetadataKeys`. When `emitDecoratorMetadata` is enabled, the TypeScript compiler emits calls to `__metadata("design:type", …)`, `__metadata("design:paramtypes", …)`, and `__metadata("design:returntype", …)` adjacent to each decorated declaration, allowing runtime consumers to inspect the **erased types** of decorated elements. This is a **legacy-decorator-only** feature: the TC39 Stage 3 decorators proposal deliberately excludes type metadata emission, providing instead a `context.metadata` bag that carries no type information.

**Beginner-Friendly Explanation.** Decorator metadata is a **filing cabinet** that decorators can write notes into and that your code can read later at runtime. Because TypeScript erases types when it compiles to JavaScript, the runtime normally has no idea what types you wrote. `reflect-metadata` gives you the **filing cabinet**, and `emitDecoratorMetadata` tells the compiler to **automatically file away** the types of decorated properties, parameters, and return values. Frameworks like NestJS and TypeORM use this filing cabinet to figure out what to inject or how to map database columns. The new Stage 3 decorators do **not** get this automatic type filing, which is why legacy frameworks remain on the old system.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Requires `reflect-metadata`** | The polyfill must be installed and imported before any decorated code runs. |
| **Requires `emitDecoratorMetadata`** | The compiler flag must be `true` in `tsconfig.json` to emit `design:*` metadata. |
| **Requires legacy decorators** | `experimentalDecorators: true` is mandatory; Stage 3 decorators do not support `design:*` emission. |
| **Metadata is not typed** | Values retrieved from `Reflect.getMetadata` are `any`; TypeScript cannot verify correctness. |
| **Prototype-chain aware** | `getMetadata` walks the prototype chain; `getOwnMetadata` inspects only the target object. |
| **Ecosystem-critical** | NestJS, TypeORM, class-validator, class-transformer, InversifyJS, and tsyringe hard-require `emitDecoratorMetadata`. |
| **Erasure-adjacent** | Metadata **values** survive at runtime, but the **types** of those values are erased; metadata is the bridge between erased types and runtime behavior. |
| **Stage 3 has a different mechanism** | `context.metadata` is a plain object assigned to `Class[Symbol.metadata]`; it carries **no type information**. |

---

### Prerequisites

- **Decorators fundamentals**: legacy method, property, parameter, and class decorators.
- **TypeScript configuration**: `tsconfig.json` with `experimentalDecorators: true` and `emitDecoratorMetadata: true`.
- **`reflect-metadata` package**: installed via `npm install reflect-metadata` and imported at the application entry point.
- **Basic reflection concepts**: reading and writing properties on objects; prototype chains.
- **A TypeScript environment** (`tsc` 5.0+ or the TypeScript Playground) and Node.js for runtime execution.

---

### Related Programming Areas

- **Dependency injection (DI)**: NestJS, Angular (legacy), InversifyJS, and tsyringe read `design:paramtypes` to resolve constructor dependencies.
- **Object-relational mapping (ORM)**: TypeORM reads `design:type` to infer column types.
- **Runtime validation**: class-validator and class-transformer read property metadata to validate and serialize objects.
- **GraphQL schema generation**: TypeGraphQL infers GraphQL field types from `design:type`.
- **Stage 3 metadata bag**: the TC39 decorator-metadata proposal provides `context.metadata` and `Symbol.metadata` for decorator-authored metadata without type reflection.

---

### Core Concepts / Features

The following four sections cover the requested concepts. **Decorator Metadata**, **`reflect-metadata`**, and **`emitDecoratorMetadata`** are treated as primary features, and **Metadata Best Practices** consolidates the guidelines that apply across all three.

---

## 1. Decorator Metadata

### Definitions

**Core Definition.** **Decorator metadata** is the information that decorators attach to classes, methods, properties, and parameters, and that can be read at runtime through the reflection API. It encompasses both **custom metadata** (written by decorators via `Reflect.defineMetadata`) and **compiler-emitted design metadata** (written by the compiler when `emitDecoratorMetadata` is enabled).

**Technical Definition.** Decorator metadata is stored in an internal **metadata map** associated with the target object and property key. The `reflect-metadata` library implements this map using a `WeakMap`-based structure, allowing garbage collection of targets that are no longer referenced. The metadata map supports the same key as both the target object and the optional property key, enabling metadata to be scoped to a class, a method, a property, or a parameter (via `parameterIndex` in the property key position). When `emitDecoratorMetadata` is active, the compiler emits `__metadata` helper calls that populate this map with `design:type` (the type of a decorated property or the function type of a method), `design:paramtypes` (an array of parameter types for a decorated method or constructor), and `design:returntype` (the return type of a decorated method).

**Beginner-Friendly Explanation.** Decorator metadata is **any note a decorator leaves behind for later**. You can leave your own notes (custom metadata) or let the compiler automatically leave notes about the types you wrote (design metadata). Those notes are stored in a hidden filing cabinet attached to the class, method, or property. Later, your framework — or your own code — can open the filing cabinet and read the notes. This is how NestJS knows what to inject into a constructor and how TypeORM knows what database type to use for a column.

### Purposes

- **To bridge compile-time types and runtime behavior**, allowing frameworks to make decisions based on types that would otherwise be erased.
- **To record custom configuration** (injection tokens, validation rules, route paths, column options) that decorators attach to declarations.
- **To enable dependency injection** by recording constructor parameter types and injection tokens.
- **To support validation, serialization, and ORM mapping** by recording property types and constraints.
- **To decouple metadata producers from consumers**, so decorators can record information without knowing how it will be used.

### Syntax Rules and Structure

**General syntax (writing custom metadata).**

```
Reflect.defineMetadata(metadataKey, metadataValue, target, propertyKey?);
```

**Component breakdown.**

- `metadataKey` — a string, symbol, or object identifying the metadata entry.
- `metadataValue` — the value to store (can be any JavaScript value).
- `target` — the object on which to define metadata (class constructor, prototype, or instance).
- `propertyKey` — optional; the property or method name. For parameter metadata, the property key is the method name, and the parameter index is stored in the metadata value.

**General syntax (reading metadata).**

```
const value = Reflect.getMetadata(metadataKey, target, propertyKey?);
```

**Component breakdown.**

- `getMetadata` walks the **prototype chain**; `getOwnMetadata` inspects only the target object.
- `hasMetadata` and `hasOwnMetadata` check for presence without retrieving the value.
- `deleteMetadata` removes a metadata entry.

**Compiler-emitted design metadata (when `emitDecoratorMetadata: true`).**

| Key | Emitted for | Value |
|---|---|---|
| `design:type` | Decorated property or method | The type constructor (`String`, `Number`, `Function`, a class reference, etc.) |
| `design:paramtypes` | Decorated method or constructor | An array of parameter type constructors |
| `design:returntype` | Decorated method | The return type constructor (`void` becomes `undefined`) |

**Syntax rules.**

- `reflect-metadata` must be imported **once, at the application entry point**, before any decorated code executes.
- `emitDecoratorMetadata` requires `experimentalDecorators: true`; it has no effect in Stage 3 mode.
- For **parameters**, the `propertyKey` is the method name (or `undefined` for constructor parameters), and the parameter index is stored as part of the metadata value or used as an additional key.
- Metadata values are **not type-checked**; retrieving metadata yields `any`.

**Constraints and limitations.**

- **No type safety**: `Reflect.getMetadata` returns `any`, and `Reflect.defineMetadata` accepts any value.
- **No typo protection**: string keys are unchecked; typos are discovered only at runtime.
- **Circular dependencies**: `design:paramtypes` can contain `undefined` for circularly referenced classes, because the referenced class may not yet be initialized when metadata is emitted.
- **`import type` hazards**: marking an imported class as `import type` causes the compiler to emit `Function` instead of the class reference in `design:paramtypes`, silently changing runtime metadata.
- **No Stage 3 equivalent**: `design:*` metadata is not emitted for Stage 3 decorators; only the `context.metadata` bag is available.

### Annotated Complete Step by Step Code Examples

#### Example 1: Custom metadata with `reflect-metadata`

```typescript
// example_custom_metadata.ts
// Step 1: Import reflect-metadata at the entry point.
import "reflect-metadata";

// Step 2: Define a custom decorator that writes metadata.
function LogMethod(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  // Step 3: Record that logging is enabled for this method.
  Reflect.defineMetadata("logEnabled", true, target, propertyKey);
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with:`, args);
    const result = original.apply(this, args);
    console.log(`${propertyKey} returned:`, result);
    return result;
  };
  return descriptor;
}

// Step 4: Apply the decorator.
class Calculator {
  @LogMethod
  add(a: number, b: number): number {
    return a + b;
  }
}

// Step 5: Read the custom metadata.
const calc = new Calculator();
calc.add(5, 3);
// Expected:
// Calling add with: [ 5, 3 ]
// add returned: 8

console.log(
  Reflect.getMetadata("logEnabled", Calculator.prototype, "add")
);
// Expected: true
```

**Setup.** Install `reflect-metadata` and enable legacy decorators:
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

**Expected Output:**
```
Calling add with: [ 5, 3 ]
add returned: 8
true
```

**Why this output occurs.** The `LogMethod` decorator both wraps the method and writes `logEnabled: true` into the metadata map for `Calculator.prototype` under the `"add"` key. The wrapper logs the call and result. `Reflect.getMetadata` retrieves the custom metadata, confirming it was stored.

---

#### Example 2: Compiler-emitted design metadata

```typescript
// example_design_metadata.ts
// Step 1: Import reflect-metadata.
import "reflect-metadata";

// Step 2: Define a class used as a return type.
class User {
  constructor(public name: string, public age: number) {}
}

// Step 3: Define a decorator that reads design metadata.
function LogTypes(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  // Step 4: Read the emitted parameter types.
  const paramTypes = Reflect.getMetadata(
    "design:paramtypes",
    target,
    propertyKey
  );
  console.log("Parameter types:", paramTypes.map((t: any) => t.name));

  // Step 5: Read the emitted return type.
  const returnType = Reflect.getMetadata(
    "design:returntype",
    target,
    propertyKey
  );
  console.log("Return type:", returnType.name);

  return descriptor;
}

// Step 6: Apply the decorator to a method.
class UserService {
  @LogTypes
  createUser(name: string, age: number): User {
    return new User(name, age);
  }
}

// Step 7: Instantiate and call.
const service = new UserService();
service.createUser("John", 30);
// Expected:
// Parameter types: [ 'String', 'Number' ]
// Return type: User
```

**Expected Output:**
```
Parameter types: [ 'String', 'Number' ]
Return type: User
```

**Why this output occurs.** Because `emitDecoratorMetadata` is enabled, the compiler emits `__metadata("design:paramtypes", [String, Number])` and `__metadata("design:returntype", User)` for the decorated `createUser` method. The `LogTypes` decorator reads these entries and logs the constructor names. The parameter types are the **constructors** for `String` and `Number`, and the return type is the `User` class constructor.

---

### Real-World Cases

- **NestJS dependency injection**: reads `design:paramtypes` to determine what to inject into a controller or service constructor.
- **TypeORM column mapping**: reads `design:type` to infer the database column type for a decorated property.
- **class-validator**: reads property metadata written by validation decorators to enforce rules at runtime.
- **TypeGraphQL**: infers GraphQL field types from `design:type`, enabling schema generation without explicit type annotations.

---

## 2. `reflect-metadata`

### Definitions

**Core Definition.** **`reflect-metadata`** is a polyfill for the ECMAScript **Metadata Reflection API** proposal. It extends the global `Reflect` object with methods for defining, retrieving, checking, and deleting metadata on objects and their properties.

**Technical Definition.** The library implements the abstract operations `[[DefineOwnMetadata]]`, `[[GetOwnMetadata]]`, `[[HasOwnMetadata]]`, `[[GetMetadata]]`, `[[HasMetadata]]`, `[[DeleteMetadata]]`, and related key-listing operations. Metadata is stored in an internal `[[Metadata]]` slot on each object, implemented as a `WeakMap`-based map to avoid memory leaks. The API mirrors the proposal's signatures: `Reflect.defineMetadata(metadataKey, metadataValue, target, propertyKey?)`, `Reflect.getMetadata(metadataKey, target, propertyKey?)`, `Reflect.getOwnMetadata(metadataKey, target, propertyKey?)`, `Reflect.hasMetadata(metadataKey, target, propertyKey?)`, `Reflect.hasOwnMetadata(metadataKey, target, propertyKey?)`, `Reflect.deleteMetadata(metadataKey, target, propertyKey?)`, `Reflect.getMetadataKeys(target, propertyKey?)`, and `Reflect.getOwnMetadataKeys(target, propertyKey?)`. The library also provides `@Reflect.metadata(metadataKey, metadataValue)` as a convenient decorator factory.

**Beginner-Friendly Explanation.** `reflect-metadata` is the **filing cabinet** itself. Without it, there is nowhere to store metadata and no way to read it. Once you import it, every object gains a hidden metadata slot, and the `Reflect` object gains methods like `defineMetadata` and `getMetadata` so you can write and read notes. It is a **polyfill** because the feature is not yet built into JavaScript engines natively; the library adds it.

### Purposes

- **To provide a standardized API** for storing and retrieving metadata on objects and properties.
- **To enable decorators to record information** that persists beyond the decorator's execution.
- **To support prototype-chain-aware lookup** through `getMetadata` and `hasMetadata`.
- **To allow per-object metadata without polluting the object's own properties**, keeping metadata invisible to `Object.keys` and JSON serialization.

### Syntax Rules and Structure

**General syntax (core API).**

```
// Define metadata
Reflect.defineMetadata(metadataKey, metadataValue, target, propertyKey?);

// Read metadata (prototype-chain aware)
const value = Reflect.getMetadata(metadataKey, target, propertyKey?);

// Read own metadata only
const ownValue = Reflect.getOwnMetadata(metadataKey, target, propertyKey?);

// Check presence
const has = Reflect.hasMetadata(metadataKey, target, propertyKey?);
const hasOwn = Reflect.hasOwnMetadata(metadataKey, target, propertyKey?);

// Delete metadata
Reflect.deleteMetadata(metadataKey, target, propertyKey?);

// List keys
const keys = Reflect.getMetadataKeys(target, propertyKey?);
const ownKeys = Reflect.getOwnMetadataKeys(target, propertyKey?);
```

**Component breakdown.**

- `metadataKey` — a `string`, `symbol`, or object identifying the entry.
- `metadataValue` — any JavaScript value.
- `target` — the object (class constructor, prototype, or instance).
- `propertyKey` — optional; the property or method name.
- `getMetadata` traverses the prototype chain; `getOwnMetadata` does not.

**Syntax rules.**

- `reflect-metadata` must be imported **before any metadata operations**; the polyfill installs the `Reflect` methods on first import.
- `getMetadata` and `hasMetadata` are **prototype-chain aware**; `getOwnMetadata` and `hasOwnMetadata` are **own-property only**.
- Metadata keys can be **any value**, but strings and symbols are conventional.
- The library is a **global polyfill**; it should be imported once at the application entry point.

**Constraints and limitations.**

- **No type safety**: all metadata values are `any`; TypeScript cannot verify correctness.
- **String-key typos** are not caught at compile time.
- **Prototype-chain confusion**: `getMetadata` may return metadata from a base class when the current class has none; `getOwnMetadata` is required for per-class metadata.
- **Memory considerations**: metadata is stored in a `WeakMap`, so targets that are garbage-collected release their metadata, but long-lived class constructors retain metadata indefinitely.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic define and retrieve

```typescript
// example_reflect_basic.ts
// Step 1: Import reflect-metadata.
import "reflect-metadata";

// Step 2: Create a target object.
const target = {};

// Step 3: Define metadata on the object.
Reflect.defineMetadata("role", "admin", target);

// Step 4: Metadata is invisible to normal property enumeration.
console.log(Object.keys(target)); // Expected: []

// Step 5: Retrieve the metadata.
console.log(Reflect.getMetadata("role", target)); // Expected: admin
console.log(Reflect.getOwnMetadata("role", target)); // Expected: admin
```

**Expected Output:**
```
[]
admin
admin
```

**Why this output occurs.** `Reflect.defineMetadata` stores `"role": "admin"` in the object's internal metadata slot, not as an own property. `Object.keys` returns an empty array because metadata is not enumerable. `getMetadata` and `getOwnMetadata` both retrieve the value because the target itself has the metadata.

---

#### Example 2: Prototype-chain-aware retrieval

```typescript
// example_reflect_inheritance.ts
// Step 1: Import reflect-metadata.
import "reflect-metadata";

// Step 2: Define metadata on a base class.
class Base {}
Reflect.defineMetadata("type", "base", Base);

// Step 3: Define metadata on a derived class.
class Derived extends Base {}
Reflect.defineMetadata("type", "derived", Derived);

// Step 4: Retrieve with getMetadata (walks prototype chain).
console.log(Reflect.getMetadata("type", Derived)); // Expected: derived

// Step 5: Retrieve with getOwnMetadata (own only).
console.log(Reflect.getOwnMetadata("type", Derived)); // Expected: derived

// Step 6: A class without own metadata.
class Plain extends Base {}
console.log(Reflect.getMetadata("type", Plain));      // Expected: base
console.log(Reflect.getOwnMetadata("type", Plain));  // Expected: undefined
```

**Expected Output:**
```
derived
derived
base
undefined
```

**Why this output occurs.** `getMetadata` on `Derived` returns `"derived"` because `Derived` has its **own** metadata, which shadows the base class's metadata. On `Plain`, which has no own metadata, `getMetadata` walks the prototype chain to `Base` and returns `"base"`, while `getOwnMetadata` returns `undefined` because `Plain` itself has no metadata.

---

### Real-World Cases

- **NestJS**: imports `reflect-metadata` at the entry point and uses `Reflect.defineMetadata`/`getMetadata` to store and read injection tokens, route paths, and guard configurations.
- **TypeORM**: uses `Reflect.defineMetadata` to store column options and `Reflect.getMetadata` to read them during schema generation.
- **Custom validation frameworks**: use `reflect-metadata` to attach validation rules to properties and read them during a validation pass.

---

## 3. `emitDecoratorMetadata`

### Definitions

**Core Definition.** **`emitDecoratorMetadata`** is a TypeScript compiler option that, when enabled alongside `experimentalDecorators`, causes the compiler to emit **runtime metadata** about the **types** of decorated declarations. The emitted metadata is stored via `reflect-metadata` under the keys `design:type`, `design:paramtypes`, and `design:returntype`.

**Technical Definition.** When `emitDecoratorMetadata` is `true`, the compiler emits `__metadata` helper calls adjacent to each decorated declaration. For a decorated method, it emits `__metadata("design:type", Function)`, `__metadata("design:paramtypes", [/* parameter constructors */])`, and `__metadata("design:returntype", /* return constructor */)`. For a decorated property, it emits `__metadata("design:type", /* property type constructor */)`. The emitted constructors are derived from TypeScript's **cross-file type information**, which means the runtime output depends on type analysis — a deliberate exception to TypeScript's general rule that types do not affect runtime behavior. **Stage 3 decorators do not emit `design:*` metadata**; the TC39 decorator-metadata proposal provides a `context.metadata` bag that carries no type information.

**Beginner-Friendly Explanation.** `emitDecoratorMetadata` is the **switch that tells TypeScript to automatically write down the types** of decorated things. When it is on, TypeScript looks at the types you wrote and emits hidden metadata like “this method takes a `String` and a `Number` and returns a `User`.” Frameworks read this metadata to know what to do. The catch is that **Stage 3 decorators do not get this automatic type recording**, so frameworks that rely on it must stay on legacy decorators.

### Purposes

- **To make erased type information available at runtime**, enabling frameworks to make type-driven decisions.
- **To support dependency injection** by exposing constructor parameter types via `design:paramtypes`.
- **To enable ORM column type inference** through `design:type`.
- **To allow serialization and validation frameworks** to inspect property and parameter types without requiring explicit type annotations.

### Syntax Rules and Structure

**General syntax (compiler configuration).**

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

**Component breakdown.**

- `experimentalDecorators: true` — enables legacy decorators; required for `emitDecoratorMetadata`.
- `emitDecoratorMetadata: true` — instructs the compiler to emit `design:*` metadata for decorated declarations.

**Emitted metadata keys and their values.**

| Key | Applies to | Value |
|---|---|---|
| `design:type` | Properties, methods | The constructor of the property type, or `Function` for methods. |
| `design:paramtypes` | Methods, constructors | An array of parameter type constructors. |
| `design:returntype` | Methods | The return type constructor; `void` becomes `undefined`. |

**Emitted metadata resolution rules.**

| Source type | Emitted value |
|---|---|
| `string`, `number`, `boolean` | `String`, `Number`, `Boolean` |
| Class declaration (not `import type`) | The class constructor itself |
| Class declaration marked `import type` | `Function` (silent behavior change) |
| Enum | `String`, `Number`, or `Object` depending on member types |
| `void` return type | `undefined` |
| Circularly referenced class | May be `undefined` if not yet initialized |

**Syntax rules.**

- `emitDecoratorMetadata` **has no effect** without `experimentalDecorators: true`.
- The metadata is emitted **only for decorated declarations**; undecorated declarations do not receive `design:*` metadata.
- `import type` imports cause the compiler to emit `Function` instead of the class constructor, which can break DI frameworks.
- **Stage 3 decorators do not emit `design:*` metadata**; the `context.metadata` bag is the only Stage 3 metadata mechanism.

**Constraints and limitations.**

- **Build tool support is bimodal**: tools that route through `tsc` or `swc` (ts-node, `@swc-node/register`, NestJS CLI’s swc builder) support metadata; tools that route through `esbuild` (tsx, esbuild-register) **silently fail to emit metadata**, causing runtime DI failures.
- **Circular dependencies** can produce `undefined` entries in `design:paramtypes`.
- **Performance overhead**: emitting metadata requires cross-file type analysis, slowing down builds.
- **Not available in Stage 3**: the TC39 proposal deliberately avoids TypeScript-specific type reflection.

### Annotated Complete Step by Step Code Examples

#### Example 1: Inspecting emitted metadata

```typescript
// example_emit_inspect.ts
// Step 1: Import reflect-metadata.
import "reflect-metadata";

// Step 2: Define a decorator that logs design metadata.
function inspect(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const designType = Reflect.getMetadata("design:type", target, propertyKey);
  const paramTypes = Reflect.getMetadata("design:paramtypes", target, propertyKey);
  const returnType = Reflect.getMetadata("design:returntype", target, propertyKey);

  console.log("design:type:", designType?.name);          // Function
  console.log("design:paramtypes:", paramTypes?.map((t: any) => t?.name));
  console.log("design:returntype:", returnType?.name);
  return descriptor;
}

// Step 3: Apply the decorator.
class Demo {
  @inspect
  process(input: string, count: number): boolean {
    return input.length > count;
  }
}

// Expected:
// design:type: Function
// design:paramtypes: [ 'String', 'Number' ]
// design:returntype: Boolean
```

**Expected Output:**
```
design:type: Function
design:paramtypes: [ 'String', 'Number' ]
design:returntype: Boolean
```

**Why this output occurs.** With `emitDecoratorMetadata: true`, the compiler emits `__metadata("design:type", Function)`, `__metadata("design:paramtypes", [String, Number])`, and `__metadata("design:returntype", Boolean)` for the `process` method. The decorator reads these entries and logs the constructor names.

---

#### Example 2: Dependency injection metadata

```typescript
// example_emit_di.ts
// Step 1: Import reflect-metadata.
import "reflect-metadata";

// Step 2: Define a class to inject.
class Database {
  connect() {
    return "connected";
  }
}

// Step 3: Define a parameter decorator that records the token.
function inject(token: string) {
  return function (
    target: any,
    propertyKey: string | undefined,
    parameterIndex: number
  ) {
    const tokens = Reflect.getMetadata("inject:tokens", target) ?? {};
    tokens[parameterIndex] = token;
    Reflect.defineMetadata("inject:tokens", tokens, target);
  };
}

// Step 4: Apply the decorator to a constructor parameter.
class UserService {
  constructor(@inject("db") private db: Database) {}
}

// Step 5: Read the emitted parameter types and custom tokens.
const paramTypes = Reflect.getMetadata("design:paramtypes", UserService);
const tokens = Reflect.getMetadata("inject:tokens", UserService);

console.log("Param types:", paramTypes?.map((t: any) => t?.name));
// Expected: Param types: [ 'Database' ]
console.log("Tokens:", tokens);
// Expected: Tokens: { '0': 'db' }
```

**Expected Output:**
```
Param types: [ 'Database' ]
Tokens: { '0': 'db' }
```

**Why this output occurs.** The compiler emits `design:paramtypes` with the `Database` constructor because `Database` is a class declaration (not `import type`). The `@inject("db")` parameter decorator separately records the injection token `"db"` for parameter index `0`. A DI container can combine these two pieces of metadata to resolve and inject the correct instance.

---

### Real-World Cases

- **NestJS**: hard-requires `emitDecoratorMetadata`; without it, constructor dependency resolution fails at runtime.
- **TypeORM**: uses `design:type` to infer column types for entity properties.
- **TypeGraphQL**: infers GraphQL field types from `design:type`; explicit `@Field(() => String)` is a fallback when metadata is unavailable.
- **tsyringe**: reads `design:paramtypes` for `@injectable()` constructor injection.

---

## 4. Metadata Best Practices

### Definitions

**Core Definition.** **Metadata best practices** are guidelines for using decorator metadata safely, maintainably, and with appropriate type discipline, while avoiding the known pitfalls of untyped metadata, circular dependencies, and build-tool incompatibilities.

**Technical Definition.** Best practices center on **typed metadata keys** (using `MetadataKey<T>` or equivalent wrappers to restore type safety), **preferring `getOwnMetadata` over `getMetadata`** when per-class metadata is required, **avoiding `import type` for classes referenced in decorator metadata**, **handling circular dependencies with forward references or explicit annotations**, **verifying build-tool support for `emitDecoratorMetadata`**, and **acknowledging the Stage 3 gap** (no `design:*` metadata in Stage 3).

**Beginner-Friendly Explanation.** Best practices mean **being careful with metadata** because TypeScript cannot check it for you. Use **typed keys** instead of raw strings so typos are caught. Use **`getOwnMetadata`** when you want only the current class’s metadata, not the base class’s. **Do not mark classes as `import type`** if they are referenced in decorator metadata, or the metadata will silently change. **Check that your build tool actually emits metadata** — some fast tools like `tsx` silently skip it. And remember that **Stage 3 decorators do not emit type metadata**, so frameworks that need it must stay on legacy decorators.

### Purposes

- **To restore type safety** to metadata operations that are inherently untyped.
- **To prevent prototype-chain bugs** caused by `getMetadata` returning base-class metadata when the current class has none.
- **To avoid silent runtime breakage** from `import type` changing emitted metadata.
- **To ensure build-tool compatibility** so that `emitDecoratorMetadata` actually takes effect.
- **To plan for the Stage 3 transition** by understanding that `design:*` metadata is a legacy-only feature.

### Syntax Rules and Structure

**Typed metadata keys (recommended).**

```
class MetadataKey<T> {
  private readonly _brand!: T;
}

const USER_AGE = new MetadataKey<number>();
Reflector.defineMetadata(USER_AGE, 30, target);   // type-checked
const age = Reflector.getMetadata(USER_AGE, target); // number
```

Libraries such as `@e22m4u/ts-reflector` provide `MetadataKey<T>` and `Reflector` wrappers that enforce type correctness at compile time.

**`getOwnMetadata` for per-class metadata (recommended).**

```
const own = Reflect.getOwnMetadata(key, Derived);   // only Derived's metadata
const inherited = Reflect.getMetadata(key, Derived); // walks prototype chain
```

**Avoid `import type` for metadata-referenced classes.**

```
import { Database } from "./database";   // ✅ emits Database constructor
// import type { Database } from "./database";  // ❌ emits Function
```

**Build-tool compatibility checklist.**

```
Routes through tsc or swc?
  ├── Yes → emitDecoratorMetadata works (ts-node, @swc-node/register, NestJS CLI swc builder)
  └── No  → esbuild-based tools (tsx, esbuild-register) silently drop metadata
```

**Constraints and limitations.**

- **Typed metadata wrappers are user-land solutions**, not built into `reflect-metadata`.
- **`getOwnMetadata` does not walk the prototype chain**; if inheritance-aware metadata is desired, walk the chain manually.
- **Circular dependencies cannot always be resolved by metadata alone**; `forwardRef` or explicit tokens may be required.
- **Stage 3 has no `design:*` equivalent**; migrating to Stage 3 requires replacing type-metadata reads with explicit annotations or framework-provided alternatives.

### Annotated Complete Step by Step Code Examples

#### Example 1: Typed metadata keys with `MetadataKey<T>`

```typescript
// example_best_typed.ts
// Step 1: Import a typed metadata library.
import { Reflector, MetadataKey } from "@e22m4u/ts-reflector";

// Step 2: Define a typed metadata structure.
type UserInfo = {
  name: string;
  age: number;
};

// Step 3: Create a typed metadata key.
const USER_INFO = new MetadataKey<UserInfo>();

// Step 4: Store typed metadata — TypeScript checks the value.
Reflector.defineMetadata(
  USER_INFO,
  { name: "John", age: 30 },
  Object
);

// Step 5: Retrieve typed metadata — the result is UserInfo, not any.
const info = Reflector.getMetadata(USER_INFO, Object);
console.log(info.name.toLowerCase()); // Expected: john
```

**Expected Output:**
```
john
```

**Why this output occurs.** `MetadataKey<UserInfo>` carries the type information that `reflect-metadata`'s string keys lack. `Reflector.defineMetadata` and `Reflector.getMetadata` are generic over the key type, so TypeScript verifies that the stored value matches `UserInfo` and returns `UserInfo` on retrieval. `info.name` is known to be a `string`, so `.toLowerCase()` is type-checked.

---

#### Example 2: `getOwnMetadata` to avoid prototype-chain leakage

```typescript
// example_best_own.ts
// Step 1: Import reflect-metadata.
import "reflect-metadata";

// Step 2: Define metadata on a base class.
class Base {}
Reflect.defineMetadata("fields", ["baseField"], Base);

// Step 3: Define metadata on a derived class.
class Derived extends Base {}
Reflect.defineMetadata("fields", ["derivedField"], Derived);

// Step 4: getMetadata walks the chain and may return base metadata.
console.log(Reflect.getMetadata("fields", Derived));
// Expected: [ 'derivedField' ]

// Step 5: getOwnMetadata returns only the current class's metadata.
console.log(Reflect.getOwnMetadata("fields", Derived));
// Expected: [ 'derivedField' ]

// Step 6: A class without own metadata.
class Plain extends Base {}
console.log(Reflect.getMetadata("fields", Plain));
// Expected: [ 'baseField' ]
console.log(Reflect.getOwnMetadata("fields", Plain));
// Expected: undefined
```

**Expected Output:**
```
[ 'derivedField' ]
[ 'derivedField' ]
[ 'baseField' ]
undefined
```

**Why this output occurs.** `getMetadata` returns `Derived`'s own metadata when it exists, but falls back to `Base`'s metadata for `Plain`, which has none. `getOwnMetadata` returns `Derived`'s metadata but `undefined` for `Plain`. Using `getOwnMetadata` prevents accidental consumption of base-class metadata when per-class configuration is intended.

---

#### Example 3: Avoiding `import type` hazards

```typescript
// example_best_import_type.ts
// Step 1: Import a class normally for metadata emission.
import { Database } from "./database"; // ✅ value import

// Step 2: A class with a decorated constructor.
class Service {
  constructor(private db: Database) {}
}

// Step 3: Read design:paramtypes.
const paramTypes = Reflect.getMetadata("design:paramtypes", Service);
console.log(paramTypes?.[0]?.name); // Expected: Database

// Step 4: If Database were imported with `import type`,
// the emitted metadata would be Function instead of Database.
// import type { Database } from "./database"; // ❌
```

**Expected Output:**
```
Database
```

**Why this output occurs.** A normal value import of `Database` causes the compiler to emit `Database` as the parameter type constructor. If the import were marked `import type`, the compiler would emit `Function` instead, silently changing the metadata and breaking DI frameworks that expect the class constructor.

---

### Real-World Cases

- **NestJS, TypeORM, and class-validator** require `emitDecoratorMetadata`; build pipelines must route through `tsc`, `swc`, or a compatible builder. `tsx` and `esbuild-register` silently fail and are a common source of “Nest can’t resolve dependencies” errors.
- **Large codebases** adopt typed metadata libraries (`@e22m4u/ts-reflector`, `typed-reflector`) to restore compile-time checks and IDE support for metadata keys and values.
- **Frameworks handling circular dependencies** use `forwardRef` or explicit injection tokens when `design:paramtypes` contains `undefined`.
- **Projects migrating to Stage 3** must replace `design:*` metadata reads with explicit type annotations or framework-provided alternatives, because Stage 3 provides only the `context.metadata` bag and `Symbol.metadata`.

---

## References

- TypeScript Metadata — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/typescript/8-typescript-advanced-concepts/6-typescript-metadata.mdx
- TypeScript Decorator Metadata — cool-coding - https://github.com/echo-cool-coding/cool-coding/blob/main/docs/language/typescript/9-typescript-decorator/8-typescript-decorator-metadata.mdx
- Changes to `consistent-type-imports` with Legacy Decorators and Decorator Metadata — typescript-eslint - https://typescript-eslint.io/blog/changes-to-consistent-type-imports-with-decorators
- reflect-metadata API — core-js README - https://raw.githubusercontent.com/zloirock/core-js/6ba79a5aada7286aa44ca9e4029cbb74dd84ffd6/README.md
- Metadata Proposal — ECMAScript (reflect-metadata spec) - https://raw.githubusercontent.com/microsoft/reflect-metadata/main/spec.html
- Improving TypeScript Metadata Type Safety with ts-reflector — tsecurity.de - https://tsecurity.de/de/2498039/IT+Programmierung/Improving+TypeScript+Metadata+Type+Safety+with+ts-reflector/
- emit-decorator-metadata research — nubjs/nub - https://github.com/nubjs/nub/blob/8dd258859f90e1398dd720ecb9a567278d8b559a/wiki/research/emit-decorator-metadata.md
- What’s the difference between Reflect.getMetadata and Reflect.getOwnMetadata? — Stack Overflow - https://stackoverflow.com/questions/39068710/whats-the-difference-between-reflect-getmetadata-and-reflect-getownmetadata
- Reflect metadata design:paramtypes return an array with undefined element — Stack Overflow - https://stackoverflow.com/questions/68576924/reflect-metadata-designparamtypes-return-an-array-with-unfedined-element
- TypeScript decorator metadata performance overhead — esbuild-decorators npm - https://www.npmjs.com/package/esbuild-decorators
- @e22m4u/ts-reflector — npm - https://www.npmjs.com/package/@e22m4u/ts-reflector
- typed-reflector — jsDelivr - https://cdn.jsdelivr.net/npm/typed-reflector
- TypeScript 5.0 Release Notes — Decorators - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html
- TC39 Decorators Proposal (Stage 3) - https://github.com/tc39/proposal-decorators
- TC39 Decorator Metadata Proposal (Stage 3) - https://github.com/tc39/proposal-decorator-metadata