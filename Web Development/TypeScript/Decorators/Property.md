# Property Decorators in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **property decorator** is a decorator applied to a class field declaration. It executes when the class is defined and may **observe or influence** the field’s initialization. TypeScript supports two incompatible property-decorator systems: **legacy experimental decorators** (based on an early Stage 2 proposal, enabled by `experimentalDecorators: true`) and **Stage 3 decorators** (the TC39-standard system, default in TypeScript 5.0+).

**Technical Definition.** In the **legacy system**, a property decorator has the signature `(target: any, propertyKey: string | symbol) => void`. It receives the prototype (or constructor for static members) and the property name, but **no `PropertyDescriptor`**. Because there is no mechanism to describe an instance property while defining prototype members, the legacy property decorator can only **observe that a property has been declared**; it cannot directly wrap or replace the field’s initializer. Its return value is ignored. In the **Stage 3 system**, a field decorator has the signature `(value: undefined, context: ClassFieldDecoratorContext<This, Value>) => ((initialValue: Value) => Value) | void`. It receives `undefined` as the value (fields have no function to receive) and a typed context exposing `kind: "field"`, `name`, `static`, `private`, and `accessor`. It may return an **initializer function** that runs when the field is assigned, receiving the initial value and returning a replacement. The `context.addInitializer` method is available for setup logic that runs during class construction.

**Beginner-Friendly Explanation.** A property decorator is **a function you place above a class field with `@`** that runs when the class is defined. In the **old system**, you cannot wrap the field’s initializer directly — you can only **record that the field exists** (for validation, serialization, or metadata). In the **new standard system**, you can **return a function that transforms the initial value** when an instance is created, and you can use `addInitializer` to run per-instance setup. Property decorators are the workhorses behind ORM column mapping, validation rules, and reactive-property systems.

---

### Key Characteristics

| Characteristic | Legacy (Experimental) | Stage 3 (Standard) |
|---|---|---|
| **Compiler flag** | `experimentalDecorators: true` required | No flag required in TS 5.0+ with `target: ES2022`+ |
| **Signature** | `(target, propertyKey) => void` | `(value: undefined, context: ClassFieldDecoratorContext) => ((v) => v) \| void` |
| **What you receive** | The prototype and the field name | `undefined` and a typed context object |
| **Can observe declaration** | Yes | Yes |
| **Can transform initial value** | **No** (return value ignored) | **Yes** (return an initializer function) |
| **`addInitializer`** | Not available | Available on `context` |
| **Metadata reflection** | Via `reflect-metadata` + `emitDecoratorMetadata` | Not supported |
| **Primary ecosystem** | TypeORM, class-validator, NestJS, class-transformer | Lit, MobX (standard mode), modern validation libraries |

---

### Prerequisites

- **Class fields and initialization order**: understanding that fields are initialized in declaration order when an instance is constructed.
- **Property descriptors**: `writable`, `enumerable`, `configurable`, `get`/`set`.
- **Generics and context types**: `ClassFieldDecoratorContext<This, Value>`.
- **Basic decorator syntax** (`@expression`) and the distinction between decorator **evaluation** and **execution**.
- **A TypeScript environment** (`tsc` 5.0+ or the TypeScript Playground) and familiarity with `tsconfig.json`.

---

### Related Programming Areas

- **Object-relational mapping (ORM)**: ORMs such as TypeORM and MikroORM use property decorators to map fields to database columns.
- **Runtime validation**: Libraries such as class-validator attach validation rules to fields via property decorators.
- **Reactive property systems**: Lit and MobX use property decorators to define reactive fields and accessors.
- **Metadata-driven serialization**: class-transformer uses property decorators to control JSON serialization and deserialization.
- **Type erasure**: Property decorators are runtime constructs, but their type signatures are erased; only the decorator function survives compilation.

---

### Core Concepts / Features

The following four sections cover the requested concepts. Legacy and Stage 3 property decorators are treated together where their behavior aligns, and distinguished explicitly where they differ.

---

## 1. Property Decorators

### Definitions

**Core Definition.** **Property decorators** are decorator functions applied to class field declarations. They execute at class definition time and may **observe the field’s declaration** (both systems) or **transform the field’s initial value** (Stage 3 only).

**Technical Definition.** In the **legacy system**, a property decorator is invoked with `(target, propertyKey)`. `target` is the prototype for instance fields or the constructor for static fields, and `propertyKey` is the field name. **No `PropertyDescriptor` is provided**, because TypeScript has no mechanism to describe an instance property while defining prototype members, and no way to observe or modify the property initializer. The return value is ignored. Consequently, legacy property decorators are used primarily to **record metadata** about the field. In the **Stage 3 system**, a field decorator is invoked with `(undefined, context)`. The `value` argument is `undefined` because fields have no method function to decorate. The decorator may **return an initializer function** `(initialValue: Value) => Value` that runs when the field is assigned, or `void` to leave the field unchanged. The `context.addInitializer` hook registers additional setup that runs during class construction.

**Beginner-Friendly Explanation.** A property decorator is **a function that runs when a field is declared**. In the **old system**, you get the class prototype and the field name, but you **cannot change how the field is initialized**. You can only **note that the field exists** — for example, to register it for validation or serialization. In the **new system**, you can **return a function that transforms the initial value** each time an instance is created, and you can use `addInitializer` to run setup code per instance. This makes the new system far more powerful for reactive properties and validation.

### Purposes

- **To record metadata about class fields** for frameworks and runtime tooling (ORM column mapping, validation rules, serialization contracts).
- **To transform field initial values** (Stage 3 only) — for example, normalizing strings, coercing types, or applying default values.
- **To register fields in a class-level registry** so that frameworks can discover and process them.
- **To attach cross-cutting initialization logic** (Stage 3) through `addInitializer`, such as binding methods or setting up observers.
- **To enforce field-level policies** (e.g., read-only, non-enumerable) where the runtime semantics of the field can be controlled.

### Syntax Rules and Structure

**General syntax (legacy property decorator).**

```
function propertyDecorator(
  target: any,
  propertyKey: string | symbol
): void {
  // observe that propertyKey exists on target
}
```

**Component breakdown.**

- `target: any` — the prototype for instance fields, or the constructor for static fields.
- `propertyKey: string | symbol` — the name of the decorated field.
- **No `descriptor` parameter** is provided.
- Returning a value has **no effect** (the return value is ignored).

**General syntax (Stage 3 field decorator).**

```
function fieldDecorator<This, Value>(
  value: undefined,
  context: ClassFieldDecoratorContext<This, Value>
): ((initialValue: Value) => Value) | void {
  return function (initialValue: Value): Value {
    // transform and return the initial value
    return initialValue;
  };
}
```

**Component breakdown.**

- `value: undefined` — fields have no method function, so `value` is `undefined`.
- `context: ClassFieldDecoratorContext<This, Value>` — typed metadata: `kind: "field"`, `name`, `static`, `private`, `accessor`, `addInitializer`.
- Returning an **initializer function** — runs when the field is assigned; receives and returns a `Value`.
- Returning `void` — leaves the field unchanged.

**Syntax rules.**

- The `@` sigil precedes the decorator expression, placed **immediately before** the field declaration.
- Multiple property decorators may be stacked; **evaluation is top-to-bottom**, **execution is bottom-to-top**.
- In the **legacy system**, property decorators **cannot modify the field’s initializer**; they can only observe it.
- In the **Stage 3 system**, returning an initializer function is the **only way** to transform the field value; mutating `context` is not the mechanism.

**Constraints and limitations.**

- **Legacy property decorators cannot wrap or replace the field initializer**; use method or accessor decorators for wrapping behavior.
- **Stage 3 field decorators receive `undefined`**, not a function; do not attempt to call `value`.
- **`addInitializer` on `ClassFieldDecoratorContext`** is available, but its interaction with field initializers is subtle; field decorators can already return an initializer, so `addInitializer` is for additional setup.
- **`useDefineForClassFields`** affects how fields are emitted and can interact with decorator behavior; be aware of the setting in your `tsconfig.json`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Legacy `@format` metadata decorator

```typescript
// example_legacy_property_format.ts
// Step 1: Import reflect-metadata for metadata storage.
import "reflect-metadata";

// Step 2: Define a metadata key.
const formatMetadataKey = Symbol("format");

// Step 3: Define a legacy property decorator factory.
function format(formatString: string) {
  return function (target: any, propertyKey: string): void {
    // Legacy property decorators receive (target, propertyKey) and
    // can only record metadata; they cannot modify the field.
    Reflect.defineMetadata(formatMetadataKey, formatString, target, propertyKey);
  };
}

// Step 4: Define a reader for the metadata.
function getFormat(target: any, propertyKey: string): string {
  return Reflect.getMetadata(formatMetadataKey, target, propertyKey);
}

// Step 5: Apply the decorator to a field.
class Greeter {
  @format("Hello, %s")
  greeting: string;

  constructor(message: string) {
    this.greeting = message;
  }

  greet(): string {
    const fmt = getFormat(this, "greeting");
    return fmt.replace("%s", this.greeting);
  }
}

// Step 6: Use the class.
const g = new Greeter("World");
console.log(g.greet()); // Expected: Hello, World
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
Hello, World
```

**Why this output occurs.** The `@format("Hello, %s")` decorator records the format string as metadata on the `Greeter` prototype under the `greeting` key. It **cannot** change how `greeting` is initialized — the constructor still assigns the raw message. The `greet` method reads the metadata and applies the format at call time. This is the canonical legacy property-decorator pattern: **observe and record, then consult the metadata elsewhere**.

---

#### Example 2: Stage 3 initializer-returning field decorator

```typescript
// example_stage3_property_transform.ts
// Step 1: Define a Stage 3 field decorator that transforms the initial value.
function uppercase<This, Value extends string>(
  value: undefined,
  context: ClassFieldDecoratorContext<This, Value>
): (initialValue: Value) => Value {
  return function (initialValue: Value): Value {
    return initialValue.toUpperCase() as Value;
  };
}

// Step 2: Apply it to a field.
class User {
  @uppercase
  name: string = "alice";

  @uppercase
  role: string = "admin";
}

// Step 3: Instantiate and inspect.
const u = new User();
console.log(u.name); // Expected: ALICE
console.log(u.role); // Expected: ADMIN
```

**Setup.** No special flag is needed in TypeScript 5.0+ with `target: ES2022` or `ESNext`.

**Expected Output:**
```
ALICE
ADMIN
```

**Why this output occurs.** The decorator returns an **initializer function** that receives the field’s initial value (`"alice"` and `"admin"`) and returns the transformed value. When each instance is constructed, the initializer runs and assigns the uppercase value. Unlike the legacy system, the Stage 3 field decorator **directly participates in initialization**.

---

#### Example 3: Stage 3 `addInitializer` for per-instance setup

```typescript
// example_stage3_addInitializer.ts
// Step 1: Define a field decorator that uses addInitializer.
function tracked<This, Value>(
  value: undefined,
  context: ClassFieldDecoratorContext<This, Value>
) {
  context.addInitializer(function (this: This) {
    // Runs during construction, after the field is initialized.
    (this as any)[`__${String(context.name)}_tracked`] = true;
  });
}

// Step 2: Apply it.
class Config {
  @tracked
  apiKey = "secret";
}

// Step 3: Inspect the instance.
const c = new Config();
console.log((c as any).__apiKey_tracked); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** `context.addInitializer` registers a callback that runs during instance initialization. The callback adds a companion property indicating that `apiKey` is tracked. This pattern is useful for registration, logging, or setting up observers that must run **after** the field is initialized.

---

### Real-World Cases

- **TypeORM `@Column()`**: a legacy property decorator that records column metadata (type, length, nullability) for database schema generation; it cannot change the field’s initializer.
- **class-validator `@IsEmail()`**: a legacy property decorator that attaches validation metadata to a field, which a separate validation pass consults.
- **Lit `@property()`**: a property decorator that defines a reactive property; in standard decorator mode, it may return an initializer or use `addInitializer` for reactive tracking.
- **Stage 3 `@clamp(min, max)`**: a field decorator that returns an initializer clamping numeric values to a range at construction time.

---

## 2. Property Decorator Declaration

### Definitions

**Core Definition.** **Property decorator declaration** is the act of writing the decorator function — defining its parameters, return type, and behavior — before it is applied to a field with `@`.

**Technical Definition.** In the **legacy system**, a property decorator declaration is a function `(target: any, propertyKey: string | symbol) => void`. In the **Stage 3 system**, it is a generic function `<This, Value>(value: undefined, context: ClassFieldDecoratorContext<This, Value>) => ((initialValue: Value) => Value) | void`. The declaration may be a **direct decorator** (a function used as-is) or a **decorator factory** (a function that returns a decorator function, allowing configuration through arguments).

**Beginner-Friendly Explanation.** Declaring a property decorator means **writing the function that will run when the field is declared**. In the **old system**, you write a function that receives the prototype and the field name, and you record metadata. In the **new system**, you write a generic function that receives `undefined` and a context object, and you **return an initializer function** if you want to transform the value. If you want options (like a min/max range), you write a **factory** — a function that takes the options and returns the decorator.

### Purposes

- **To define reusable field-level behavior** that can be applied to many fields across classes.
- **To encapsulate configuration** through decorator factories, keeping call sites concise.
- **To type the decorator precisely** (especially in Stage 3) so the compiler verifies that the initializer is compatible with the field’s type.
- **To separate field metadata or transformation logic** from the classes that declare the fields.

### Syntax Rules and Structure

**Legacy declaration (direct decorator).**

```
function decoratorName(
  target: any,
  propertyKey: string | symbol
): void {
  // record metadata about propertyKey on target
}
```

**Legacy declaration (factory).**

```
function decoratorFactory(config: ConfigType) {
  return function (
    target: any,
    propertyKey: string | symbol
  ): void {
    // use config and propertyKey
  };
}
```

**Stage 3 declaration (direct decorator).**

```
function decoratorName<This, Value>(
  value: undefined,
  context: ClassFieldDecoratorContext<This, Value>
): ((initialValue: Value) => Value) | void {
  return function (initialValue: Value): Value {
    return initialValue;
  };
}
```

**Stage 3 declaration (factory).**

```
function decoratorFactory(config: ConfigType) {
  return function <This, Value>(
    value: undefined,
    context: ClassFieldDecoratorContext<This, Value>
  ): ((initialValue: Value) => Value) | void {
    return function (initialValue: Value): Value {
      // use config to transform initialValue
      return initialValue;
    };
  };
}
```

**Syntax rules.**

- The **generic signature** `<This, Value>` preserves the instance type and the field’s declared type.
- `context: ClassFieldDecoratorContext<This, Value>` is the typed context for field decorators.
- `value: undefined` is required by the signature; fields do not provide a function value.
- Returning an initializer function is optional; returning `void` leaves the field unchanged.
- The `context` parameter may be named `_context` if unused.

**Constraints and limitations.**

- **Legacy declarations cannot return a useful value**; the return value is ignored, so do not attempt to return a `PropertyDescriptor`.
- **Stage 3 declarations must return a function or `void`**; returning an unrelated value is a type error.
- **Factories should be pure** where possible; avoid shared mutable state across decorations.
- **`addInitializer` should be used for setup**, not for transforming the field value; the returned initializer is the transformation mechanism.

### Annotated Complete Step by Step Code Examples

#### Example 1: Legacy declaration — direct and factory

```typescript
// example_legacy_declaration.ts
// Step 1: Direct legacy property decorator.
function readonly(target: any, propertyKey: string): void {
  console.log(`Field ${propertyKey} is read-only`);
}

// Step 2: Legacy decorator factory with configuration.
function minLength(length: number) {
  return function (target: any, propertyKey: string): void {
    Reflect.defineMetadata(`minLength:${propertyKey}`, length, target);
  };
}

// Step 3: Apply both.
class User {
  @readonly
  id: number = 1;

  @minLength(3)
  username: string = "ab";
}

// Expected:
// Field id is read-only
```

**Expected Output:**
```
Field id is read-only
```

**Why this output occurs.** `@readonly` is a direct decorator that logs the field name. `@minLength(3)` is a factory that records the minimum length as metadata. Neither decorator can change the field’s initializer; they only observe and record. The actual enforcement of `minLength` would require a separate validation pass.

---

#### Example 2: Stage 3 declaration with generic typing and a factory

```typescript
// example_stage3_declaration.ts
// Step 1: Stage 3 factory that clamps numeric field values.
function clamp(min: number, max: number) {
  return function <This, Value extends number>(
    value: undefined,
    context: ClassFieldDecoratorContext<This, Value>
  ): (initialValue: Value) => Value {
    return function (initialValue: Value): Value {
      return Math.min(max, Math.max(min, initialValue)) as Value;
    };
  };
}

// Step 2: Apply the factory.
class Progress {
  @clamp(0, 100)
  percent: number = 150;

  @clamp(0, 100)
  safePercent: number = 50;
}

// Step 3: Instantiate and inspect.
const p = new Progress();
console.log(p.percent);     // Expected: 100
console.log(p.safePercent); // Expected: 50
```

**Expected Output:**
```
100
50
```

**Why this output occurs.** The factory captures `min = 0` and `max = 100` and returns a Stage 3 field decorator. The decorator returns an initializer that clamps the field’s initial value. When `percent` is initialized with `150`, the initializer returns `100`. When `safePercent` is initialized with `50`, it remains `50`. The generic constraint `Value extends number` ensures the initializer only applies to numeric fields.

---

### Real-World Cases

- **TypeORM `@Column("varchar", { length: 255 })`**: a legacy factory that records column type and length metadata for schema generation.
- **class-validator `@Min(0)` / `@Max(100)`**: a legacy factory that attaches numeric range validation metadata to a field.
- **Stage 3 `@clamp(min, max)`**: a factory that returns an initializer clamping numeric values at construction.
- **Stage 3 `@defaultTo(value)`**: a factory that returns an initializer substituting a default when the field is `undefined`.

---

## 3. Property Decorator Usage

### Definitions

**Core Definition.** **Property decorator usage** is the application of a declared property decorator to a class field via `@decoratorName` or `@decoratorFactory(args)`, triggering the decorator’s execution at class definition time.

**Technical Definition.** Usage consists of placing the `@` expression **immediately before** a field declaration. The decorator expression is evaluated (including any factory call), and the resulting decorator function is invoked with `(target, propertyKey)` in the legacy system or `(undefined, context)` in the Stage 3 system. Multiple decorators may be stacked; their **evaluation order** is top-to-bottom, but their **execution order** is bottom-to-top: `@A @B field` applies `B` first, then `A` to the result.

**Beginner-Friendly Explanation.** Using a property decorator means **putting `@name` right above the field**. When the class is defined, the decorator runs automatically. You do not call it yourself. If the decorator is a factory, you call the factory with parentheses (`@min(0)`), and it returns the actual decorator. When you stack decorators, the one **closest to the field runs first**, even though the expressions are written top to bottom.

### Purposes

- **To apply reusable field behavior** to fields without modifying their declarations.
- **To compose multiple decorators** on one field, each handling a distinct concern (validation, serialization, reactivity).
- **To configure decorators** through factory arguments at the usage site.
- **To declaratively attach field-level policies** adjacent to the field declaration, keeping the contract visible.

### Syntax Rules and Structure

**General usage syntax.**

```
@decoratorName
fieldName: Type = initialValue;

@decoratorFactory(options)
fieldName: Type = initialValue;

@first
@second("option")
fieldName: Type = initialValue;
```

**Component breakdown.**

- `@decoratorName` — direct decorator usage; the function is the decorator.
- `@decoratorFactory(options)` — factory usage; the factory is called, and its return value is the decorator.
- Stacking — multiple decorators on separate lines; execution is bottom-to-top.

**Syntax rules.**

- The decorator must be **immediately above** the field declaration (or on the same line, before the field name).
- Decorator expressions are restricted to **identifiers, dotted names, and calls**.
- **Evaluation order**: top-to-bottom. **Execution order**: bottom-to-top.
- A field may have **multiple decorators**; each receives the field information (or the result of the decorator below it).

**Constraints and limitations.**

- **Decorator factories must be called** with parentheses; `@min` without `()` passes the factory itself, which is usually incorrect.
- **Execution order matters**: if one decorator depends on metadata set by another, order the decorators accordingly.
- **Legacy and Stage 3 usage is syntactically identical** but semantically incompatible; the same `@` syntax selects different behavior based on the compiler flag.
- **Field decorators cannot be applied to `declare` fields** in ambient contexts where no runtime field exists.

### Annotated Complete Step by Step Code Examples

#### Example 1: Stacked legacy property decorators

```typescript
// example_usage_stacked_legacy.ts
// Step 1: Define two legacy property decorators.
function first(target: any, propertyKey: string): void {
  console.log(`first: ${propertyKey}`);
}

function second(target: any, propertyKey: string): void {
  console.log(`second: ${propertyKey}`);
}

// Step 2: Stack them.
class Example {
  @first
  @second
  value: string = "test";
}

// Expected:
// second: value
// first: value
```

**Expected Output:**
```
second: value
first: value
```

**Why this output occurs.** `@second` is closest to the field, so it executes first, logging `second: value`. Then `@first` executes, logging `first: value`. Both receive the same `(target, propertyKey)` arguments; neither can modify the field. The order demonstrates bottom-to-top execution.

---

#### Example 2: Stacked Stage 3 field decorators with initializer composition

```typescript
// example_usage_stacked_stage3.ts
// Step 1: Define two Stage 3 field decorators.
function trim<This, Value extends string>(
  value: undefined,
  _context: ClassFieldDecoratorContext<This, Value>
) {
  return function (initialValue: Value): Value {
    return initialValue.trim() as Value;
  };
}

function lowercase<This, Value extends string>(
  value: undefined,
  _context: ClassFieldDecoratorContext<This, Value>
) {
  return function (initialValue: Value): Value {
    return initialValue.toLowerCase() as Value;
  };
}

// Step 2: Stack them.
class User {
  @trim
  @lowercase
  username: string = "  ALICE  ";
}

// Step 3: Instantiate.
const u = new User();
console.log(u.username); // Expected: alice
```

**Expected Output:**
```
alice
```

**Why this output occurs.** `@lowercase` executes first (closest to the field), transforming `"  ALICE  "` to `"  alice  "`. Then `@trim` executes, transforming the result to `"alice"`. The initializers compose: `trim(lowercase(initialValue))`. Order matters: if the decorators were reversed, the result would be `"alice"` as well in this case, but for other transformations the order would produce different results.

---

### Real-World Cases

- **TypeORM `@Column()` + `@Index()`**: two legacy property decorators stacked on a field, one recording column metadata and the other recording index metadata.
- **class-validator `@IsString()` + `@MinLength(3)`**: stacked legacy decorators attaching multiple validation rules to one field.
- **Lit `@property({ type: String })` + `@state()`**: stacked Stage 3 decorators configuring reactive property behavior and internal state.

---

## 4. Property Decorator Best Practices

### Definitions

**Core Definition.** **Property decorator best practices** are guidelines for declaring and using property decorators in ways that preserve type safety, maintainability, and compatibility with the chosen decorator system.

**Technical Definition.** Best practices center on **system selection** (legacy for TypeORM/class-validator/NestJS dependencies, Stage 3 for new projects), **understanding the legacy limitation** (property decorators cannot modify initializers; use accessor decorators or Stage 3 initializers for transformation), **precise Stage 3 typing** (using `<This, Value>` generics and typed context), **use of `addInitializer` for per-instance setup**, **single responsibility** (one concern per decorator), and **order awareness** (decorators apply bottom-to-top).

**Beginner-Friendly Explanation.** Best practices mean **choosing the right system** for your project, **not expecting legacy property decorators to do things they cannot do** (like change initial values), and **writing Stage 3 decorators with precise types**. If you need to transform a field value in the legacy system, use an **accessor decorator** or a manual getter/setter instead. In Stage 3, return an initializer for transformation and use `addInitializer` for setup. Keep each decorator focused on one job, and remember that order matters.

### Purposes

- **To avoid frustrated debugging** caused by expecting legacy property decorators to modify field values when they cannot.
- **To preserve type safety** by using precise generic signatures and typed contexts in Stage 3.
- **To choose a future-proof decorator system** aligned with the project’s ecosystem dependencies.
- **To keep decorators composable and predictable** by following execution-order rules and single responsibility.

### Syntax Rules and Structure

**System selection checklist.**

```
Depends on TypeORM, class-validator, NestJS, or class-transformer?
  ├── Yes → experimentalDecorators: true (legacy system)
  │         Use property decorators for METADATA ONLY.
  │         Use accessor decorators for transformation.
  └── No  → Stage 3 decorators (default in TS 5.0+)
            Use returned initializers for TRANSFORMATION.
            Use context.addInitializer for SETUP.
```

**Stage 3 precise typing (recommended).**

```
function decorator<This, Value>(
  value: undefined,
  context: ClassFieldDecoratorContext<This, Value>
): ((initialValue: Value) => Value) | void
```

**`addInitializer` for setup (Stage 3).**

```
context.addInitializer(function (this: This) {
  // per-instance setup after field initialization
});
```

**Constraints and limitations.**

- **Do not mix systems**; the `experimentalDecorators` flag selects one and disables the other.
- **Do not attempt to return a `PropertyDescriptor` from a legacy property decorator**; the return value is ignored.
- **Do not use `addInitializer` to transform the field value** in Stage 3; return an initializer function instead.
- **Be aware of `useDefineForClassFields`**; it affects field emission and can change decorator behavior in edge cases.

### Annotated Complete Step by Step Code Examples

#### Example 1: Correct legacy usage — metadata only, transformation via accessor

```typescript
// example_best_legacy_metadata.ts
// ✅ Legacy property decorators record metadata; transformation uses accessors.
function column(target: any, propertyKey: string): void {
  const columns = Reflect.getMetadata("columns", target) ?? [];
  columns.push(propertyKey);
  Reflect.defineMetadata("columns", columns, target);
}

class Entity {
  @column
  id: number = 1;

  @column
  name: string = "test";
}

console.log(Reflect.getMetadata("columns", Entity.prototype));
// Expected: [ 'id', 'name' ]
```

**Expected Output:**
```
[ 'id', 'name' ]
```

**Why this output occurs.** The `@column` decorator records the field names in metadata. It does **not** attempt to change the field values. A separate framework pass would read this metadata to generate SQL or validation rules. This is the correct legacy pattern: **observe and record, do not attempt to transform**.

---

#### Example 2: Correct Stage 3 usage — initializer for transformation, `addInitializer` for setup

```typescript
// example_best_stage3_transform_setup.ts
// ✅ Stage 3: return an initializer for transformation, use addInitializer for setup.
function normalized<This, Value extends string>(
  value: undefined,
  context: ClassFieldDecoratorContext<This, Value>
) {
  context.addInitializer(function (this: This) {
    console.log(`[init] ${String(context.name)} initialized`);
  });
  return function (initialValue: Value): Value {
    return initialValue.trim().toLowerCase() as Value;
  };
}

class User {
  @normalized
  email: string = "  ALICE@EXAMPLE.COM  ";
}

const u = new User();
// Expected: [init] email initialized
console.log(u.email); // Expected: alice@example.com
```

**Expected Output:**
```
[init] email initialized
alice@example.com
```

**Why this output occurs.** The decorator returns an initializer that trims and lowercases the email. `context.addInitializer` registers a separate callback that logs during construction. The initializer handles **value transformation**, while `addInitializer` handles **setup logic**. This separation of concerns is the recommended Stage 3 pattern.

---

#### Example 3: Single-responsibility decorators and order awareness

```typescript
// example_best_single_responsibility.ts
// ✅ Each decorator does ONE thing.
function trim<This, Value extends string>(
  value: undefined,
  _context: ClassFieldDecoratorContext<This, Value>
) {
  return function (initialValue: Value): Value {
    return initialValue.trim() as Value;
  };
}

function uppercase<This, Value extends string>(
  value: undefined,
  _context: ClassFieldDecoratorContext<This, Value>
) {
  return function (initialValue: Value): Value {
    return initialValue.toUpperCase() as Value;
  };
}

class Code {
  @uppercase
  @trim
  value: string = "  abc  ";
}

const c = new Code();
console.log(c.value); // Expected: ABC
```

**Expected Output:**
```
ABC
```

**Why this output occurs.** `@trim` executes first (closest to the field), producing `"abc"`. Then `@uppercase` executes, producing `"ABC"`. If the order were reversed (`@trim` above `@uppercase`), the result would be `"ABC"` as well in this simple case, but for transformations where order affects the result (e.g., `trim` before `uppercase` vs. `uppercase` before `trim`), explicit ordering is essential. Each decorator has a single responsibility, making composition predictable.

---

### Real-World Cases

- **TypeORM projects** keep legacy property decorators for column mapping and use accessor decorators or manual getters for value transformation.
- **New projects** use Stage 3 field decorators with returned initializers for normalization and `addInitializer` for registration or logging.
- **Lit and MobX** in standard decorator mode use field decorators that return initializers to define reactive properties, replacing legacy `@property()` patterns that relied on metadata reflection.

---

## References

- Decorators — TypeScript (Advanced) - https://mintlify.wiki/Microsoft/typescript/advanced/decorators
- TypeScript Handbook — Decorators (legacy) - https://www.typescriptlang.org/docs/handbook/decorators.html
- TypeScript-Handbook/Decorators.md — Property Decorator Note - https://github.com/Microsoft/TypeScript-Handbook/blame/master/pages/Decorators.md
- Use Stage 3 Decorators with Decorator Context for Metaprogramming - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-advanced-patterns/references/mod-stage-3-decorators.md
- TypeScript Decorators: The Complete Guide to Stage-3 and Beyond — DEV Community - https://dev.to/abhaysinghr1/typescript-decorators-the-complete-guide-to-stage-3-and-beyond-40ok
- ClassFieldDecoratorContext — JetBrains API Reference - https://jetbrains.github.io/kotlin-wrappers/kotlin-js/js.decorators/-class-field-decorator-context/
- TC39 Decorators Proposal (Stage 3) - https://github.com/tc39/proposal-decorators
- TypeScript 5.0 Release Notes — Decorators - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html