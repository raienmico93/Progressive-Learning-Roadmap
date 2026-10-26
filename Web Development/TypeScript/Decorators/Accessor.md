# Accessor Decorators in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** An **accessor decorator** is a decorator applied to a getter, a setter, or an **auto-accessor** (`accessor` field) within a class. It executes at class definition time and may **observe, modify, or replace** the accessor’s implementation. TypeScript supports two incompatible accessor-decorator systems: **legacy experimental decorators** (applied to traditional `get`/`set` declarations, enabled by `experimentalDecorators: true`) and **Stage 3 decorators** (the TC39-standard system, default in TypeScript 5.0+, which also introduces the `accessor` keyword for auto-accessors).

**Technical Definition.** In the **legacy system**, an accessor decorator is syntactically a **method decorator** applied to a getter or setter declaration. It receives `(target: any, propertyKey: string | symbol, descriptor: PropertyDescriptor)`, where `descriptor.get` and/or `descriptor.set` hold the accessor functions. The decorator may mutate the descriptor and must return it to install the modification; returning `void` leaves the accessor unchanged. In the **Stage 3 system**, accessor decoration splits into three context kinds: `ClassGetterDecoratorContext` (for `get` declarations), `ClassSetterDecoratorContext` (for `set` declarations), and `ClassAccessorDecoratorContext` (for `accessor` auto-accessors). Getter and setter decorators receive the accessor function and a typed context and may return a **replacement function**. Auto-accessor decorators receive a **`{ get, set }` pair** and may return a **replacement `{ get, set }` pair**, enabling read/write interception without changing the class’s runtime shape.

**Beginner-Friendly Explanation.** An accessor decorator is a **function you place above a getter, a setter, or an `accessor` field with `@`**. It runs when the class is defined and can **wrap the reading or writing of a property** — adding logging, validation, reactivity, or caching. In the **old system**, you decorate a `get` or `set` declaration and receive a descriptor you can modify. In the **new standard system**, you can decorate `get`/`set` separately, or use the new `accessor` keyword to create a **getter/setter pair in one declaration** that a decorator can intercept cleanly. Accessor decorators are the foundation of reactive property systems (Lit, MobX) and property-level validation.

---

### Key Characteristics

| Characteristic | Legacy (Experimental) | Stage 3 (Standard) |
|---|---|---|
| **Compiler flag** | `experimentalDecorators: true` required | No flag required in TS 5.0+ with `target: ES2022`+ |
| **Target syntax** | `get` / `set` declarations | `get` / `set` declarations **or** `accessor` keyword |
| **Signature (getter/setter)** | `(target, propertyKey, descriptor)` | `(value, context: ClassGetterDecoratorContext \| ClassSetterDecoratorContext)` |
| **Signature (auto-accessor)** | N/A (no `accessor` keyword) | `(value: { get, set }, context: ClassAccessorDecoratorContext)` |
| **How you modify** | Mutate `descriptor.get` / `descriptor.set` and return descriptor | **Return a replacement function** (getter/setter) or **replacement `{ get, set }`** (auto-accessor) |
| **Type safety** | Broad (`any`, `PropertyDescriptor`) | Precise generics (`This`, `Value`) and typed contexts |
| **Read/write interception** | Possible by wrapping `descriptor.get` / `descriptor.set` | Cleaner via `accessor` and `ClassAccessorDecoratorContext` |
| **Primary ecosystem** | Angular, NestJS, TypeORM (legacy accessors) | Lit, MobX (standard mode), Effect, modern reactive libraries |

---

### Prerequisites

- **Getters, setters, and property descriptors**: understanding `get`, `set`, and how `PropertyDescriptor` describes accessors.
- **Class fields and initialization order**: especially for auto-accessors and backing storage.
- **Generics and context types**: `ClassGetterDecoratorContext<This, Value>`, `ClassSetterDecoratorContext<This, Value>`, `ClassAccessorDecoratorContext<This, Value>`.
- **Basic decorator syntax** (`@expression`) and the distinction between decorator **evaluation** and **execution**.
- **A TypeScript environment** (`tsc` 5.0+ or the TypeScript Playground) and familiarity with `tsconfig.json`.

---

### Related Programming Areas

- **Reactive property systems**: Lit and MobX use accessor decorators to intercept reads and writes for reactivity.
- **Property validation and transformation**: accessor decorators enforce rules on assignment (e.g., clamping, type coercion) without manual getter/setter boilerplate.
- **Serialization and marshaling**: accessor decorators can normalize values during reads or writes, supporting JSON serialization and ORM hydration.
- **Caching and lazy evaluation**: getter decorators can memoize computed properties.
- **Type erasure**: accessor decorators are runtime constructs, but their type signatures are erased; only the decorator function survives compilation.

---

### Core Concepts / Features

The following four sections cover the requested concepts. Legacy and Stage 3 accessor decorators are treated together where their behavior aligns, and distinguished explicitly where they differ. The **`accessor` auto-accessor** is a Stage 3-only feature and is covered within the Stage 3 discussion.

---

## 1. Accessor Decorators

### Definitions

**Core Definition.** **Accessor decorators** are decorator functions applied to getter, setter, or auto-accessor declarations. They execute at class definition time and may **observe, modify, or replace** the accessor’s implementation.

**Technical Definition.** In the **legacy system**, an accessor decorator is a method decorator applied to a `get` or `set` declaration. It receives `(target, propertyKey, descriptor)` where `descriptor.get` and/or `descriptor.set` are the accessor functions. The decorator may replace `descriptor.get` or `descriptor.set` with a wrapper and return the descriptor. In the **Stage 3 system**, getter decorators receive `(getterFunction, ClassGetterDecoratorContext)` and may return a replacement getter; setter decorators receive `(setterFunction, ClassSetterDecoratorContext)` and may return a replacement setter. **Auto-accessor decorators** receive `{ get, set }` and a `ClassAccessorDecoratorContext` and may return a replacement `{ get, set }` pair. The `accessor` keyword creates a property backed by a private field with an automatically generated getter/setter pair, which a decorator can intercept without changing the class’s runtime shape.

**Beginner-Friendly Explanation.** An accessor decorator lets you **wrap the reading or writing of a property**. In the **old system**, you decorate a `get` or `set` and replace the function in the descriptor. In the **new system**, you can decorate `get` and `set` separately, or use `accessor` to declare both at once and return a new `{ get, set }` pair that adds behavior. This is how you implement **validation on assignment**, **logging on reads**, or **reactive tracking** without writing a manual wrapper every time.

### Purposes

- **To intercept property reads and writes** for logging, validation, transformation, or reactive tracking.
- **To enforce property-level policies** such as read-only access, range constraints, or type coercion on assignment.
- **To implement lazy evaluation or memoization** for computed getters.
- **To support reactive frameworks** (Lit, MobX) that track property access through getter/setter interception.
- **To separate cross-cutting property concerns** (validation, serialization, observation) from the class’s core logic.

### Syntax Rules and Structure

**General syntax (legacy accessor decorator).**

```
function accessorDecorator(
  target: any,
  propertyKey: string | symbol,
  descriptor: PropertyDescriptor
): PropertyDescriptor | void {
  const originalGet = descriptor.get;
  const originalSet = descriptor.set;
  descriptor.get = function () { /* wrap */ return originalGet?.call(this); };
  descriptor.set = function (v) { /* wrap */ originalSet?.call(this, v); };
  return descriptor;
}
```

**Component breakdown.**

- `target: any` — the prototype (or constructor for static accessors).
- `propertyKey` — the accessor name.
- `descriptor: PropertyDescriptor` — holds `get` and/or `set` functions.
- Returning `descriptor` installs the modification; returning `void` leaves the accessor unchanged.

**General syntax (Stage 3 getter/setter decorator).**

```
function getterDecorator<This, Value>(
  value: (this: This) => Value,
  context: ClassGetterDecoratorContext<This, Value>
): ((this: This) => Value) | void {
  return function (this: This): Value {
    // wrap the getter
    return value.call(this);
  };
}
```

**General syntax (Stage 3 auto-accessor decorator).**

```
function accessorDecorator<This, Value>(
  value: { get: (this: This) => Value; set: (this: This, v: Value) => void },
  context: ClassAccessorDecoratorContext<This, Value>
): { get?: (this: This) => Value; set?: (this: This, v: Value) => void } | void {
  return {
    get() { return value.get.call(this); },
    set(v) { value.set.call(this, v); },
  };
}
```

**Syntax rules.**

- Legacy accessor decorators are **syntactically identical to method decorators** but apply to `get`/`set` declarations.
- Stage 3 getter and setter decorators receive the **accessor function** and a typed context; they return a **replacement function**.
- Stage 3 auto-accessor decorators receive a **`{ get, set }` pair** and may return a **partial or full replacement pair**.
- The `accessor` keyword is **Stage 3-only** and cannot be used with `readonly` or `declare` on the same field.
- Multiple accessor decorators may be stacked; **execution is bottom-to-top**.

**Constraints and limitations.**

- **Legacy accessor decorators cannot be applied to `accessor` auto-accessors** (the keyword does not exist in legacy mode).
- **Stage 3 getter and setter decorators are separate**; decorating a `get` does not automatically decorate the matching `set`.
- **Auto-accessor backing storage is private** and not directly reachable; decorators must use the provided `get`/`set` pair.
- **Legacy and Stage 3 systems are incompatible**; the `experimentalDecorators` flag selects one and disables the other.

### Annotated Complete Step by Step Code Examples

#### Example 1: Legacy accessor decorator for read-only enforcement

```typescript
// example_legacy_accessor.ts
// Step 1: Define a legacy accessor decorator factory.
function configurable(value: boolean) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ): PropertyDescriptor {
    descriptor.configurable = value;
    return descriptor;
  };
}

// Step 2: Apply it to getters.
class Point {
  private _x = 0;
  private _y = 0;

  @configurable(false)
  get x() {
    return this._x;
  }

  @configurable(false)
  get y() {
    return this._y;
  }
}

// Step 3: Verify configurability.
const p = new Point();
const desc = Object.getOwnPropertyDescriptor(Point.prototype, "x");
console.log(desc?.configurable); // Expected: false
```

**Expected Output:**
```
false
```

**Why this output occurs.** The legacy accessor decorator `@configurable(false)` receives the `PropertyDescriptor` for the `x` getter and sets `descriptor.configurable = false`. Returning the descriptor installs the change. `Object.getOwnPropertyDescriptor` confirms that the accessor is no longer configurable.

---

#### Example 2: Stage 3 getter decorator with logging

```typescript
// example_stage3_getter.ts
// Step 1: Define a Stage 3 getter decorator.
function logged<This, Value>(
  getter: (this: This) => Value,
  context: ClassGetterDecoratorContext<This, Value>
): (this: This) => Value {
  const name = String(context.name);
  return function (this: This): Value {
    const value = getter.call(this);
    console.log(`[GET] ${name} = ${value}`);
    return value;
  };
}

// Step 2: Apply it to a getter.
class User {
  private _name = "Alice";

  @logged
  get name() {
    return this._name;
  }
}

// Step 3: Access the getter.
const u = new User();
console.log(u.name);
// Expected:
// [GET] name = Alice
// Alice
```

**Expected Output:**
```
[GET] name = Alice
Alice
```

**Why this output occurs.** The decorator returns a replacement getter that logs the accessed value before returning it. `context.name` provides the property name `"name"`. The original getter is called via `getter.call(this)`, preserving `this`.

---

#### Example 3: Stage 3 auto-accessor with validation

```typescript
// example_stage3_auto_accessor.ts
// Step 1: Define a Stage 3 auto-accessor decorator that validates on set.
function clamp(min: number, max: number) {
  return function <This, Value extends number>(
    value: { get: (this: This) => Value; set: (this: This, v: Value) => void },
    _context: ClassAccessorDecoratorContext<This, Value>
  ) {
    return {
      get(this: This): Value {
        return value.get.call(this);
      },
      set(this: This, newValue: Value): void {
        const clamped = Math.min(max, Math.max(min, newValue)) as Value;
        value.set.call(this, clamped);
      },
    };
  };
}

// Step 2: Apply it to an auto-accessor.
class Progress {
  @clamp(0, 100)
  accessor percent = 50;
}

// Step 3: Set values and observe clamping.
const p = new Progress();
p.percent = 150;
console.log(p.percent); // Expected: 100
p.percent = -20;
console.log(p.percent); // Expected: 0
```

**Expected Output:**
```
100
0
```

**Why this output occurs.** The `accessor` keyword creates a getter/setter pair backed by a private field. The decorator receives the original `{ get, set }` pair and returns a new pair whose setter clamps the value to the `[0, 100]` range before delegating to the original setter. The getter passes through unchanged.

---

### Real-World Cases

- **MobX `@observable accessor`**: a Stage 3 auto-accessor decorator that intercepts reads and writes to track reactive dependencies, replacing the legacy `@observable` + `makeObservable` pattern.
- **Lit `@property()`**: defines reactive properties; in standard decorator mode, accessor decoration enables change detection on get/set.
- **class-validator accessor decorators**: enforce validation rules on assignment, rejecting invalid values before they are stored.
- **Legacy Angular `@Input()` / `@Output()` on getters/setters**: legacy accessor decorators record metadata for change detection and template binding.

---

## 2. Accessor Decorator Declaration

### Definitions

**Core Definition.** **Accessor decorator declaration** is the act of writing the decorator function — defining its parameters, return type, and behavior — before it is applied to an accessor with `@`.

**Technical Definition.** In the **legacy system**, an accessor decorator declaration is a function `(target: any, propertyKey: string | symbol, descriptor: PropertyDescriptor) => PropertyDescriptor | void`. In the **Stage 3 system**, a getter decorator declaration is `<This, Value>(value: (this: This) => Value, context: ClassGetterDecoratorContext<This, Value>) => ((this: This) => Value) | void`; a setter decorator declaration is `<This, Value>(value: (this: This, v: Value) => void, context: ClassSetterDecoratorContext<This, Value>) => ((this: This, v: Value) => void) | void`; and an auto-accessor decorator declaration is `<This, Value>(value: { get, set }, context: ClassAccessorDecoratorContext<This, Value>) => ({ get?, set? }) | void`. The declaration may be a **direct decorator** or a **decorator factory** (a function that returns a decorator function, allowing configuration).

**Beginner-Friendly Explanation.** Declaring an accessor decorator means **writing the function that runs when the accessor is decorated**. In the **old system**, you write a function that receives the prototype, the property name, and the descriptor, and you modify the descriptor. In the **new system**, you write a generic function that receives the accessor function (or `{ get, set }` pair) and a typed context, and you **return a replacement**. If you want options (like a min/max range), you write a **factory**.

### Purposes

- **To define reusable accessor-level behavior** that can be applied to many getters, setters, or auto-accessors.
- **To encapsulate configuration** through decorator factories, keeping call sites concise.
- **To type the decorator precisely** (especially in Stage 3) so the compiler verifies that the replacement is compatible with the accessor.
- **To separate read/write interception logic** from the classes that declare the accessors.

### Syntax Rules and Structure

**Legacy declaration (direct decorator).**

```
function decoratorName(
  target: any,
  propertyKey: string | symbol,
  descriptor: PropertyDescriptor
): PropertyDescriptor | void {
  // wrap descriptor.get and/or descriptor.set
}
```

**Legacy declaration (factory).**

```
function decoratorFactory(config: ConfigType) {
  return function (
    target: any,
    propertyKey: string | symbol,
    descriptor: PropertyDescriptor
  ): PropertyDescriptor | void {
    // use config
  };
}
```

**Stage 3 getter declaration (direct).**

```
function getterDecorator<This, Value>(
  getter: (this: This) => Value,
  context: ClassGetterDecoratorContext<This, Value>
): ((this: This) => Value) | void {
  return function (this: This): Value { /* wrap */ };
}
```

**Stage 3 auto-accessor declaration (factory).**

```
function accessorFactory(config: Config) {
  return function <This, Value>(
    value: { get: (this: This) => Value; set: (this: This, v: Value) => void },
    context: ClassAccessorDecoratorContext<This, Value>
  ): { get?: (this: This) => Value; set?: (this: This, v: Value) => void } | void {
    // use config and return a replacement pair
  };
}
```

**Syntax rules.**

- The **generic signature** `<This, Value>` preserves the instance type and the accessor’s value type.
- `context: ClassGetterDecoratorContext<...>` or `ClassSetterDecoratorContext<...>` or `ClassAccessorDecoratorContext<...>` is the typed context for the specific accessor kind.
- Legacy declarations receive a `PropertyDescriptor`; returning it installs the modification.
- Stage 3 declarations return a **replacement function** (getter/setter) or a **replacement `{ get, set }` pair** (auto-accessor).
- Factories return the decorator function; the factory itself is **not** the decorator.

**Constraints and limitations.**

- **Legacy declarations cannot distinguish getter from setter** at the signature level; both receive the same `PropertyDescriptor`.
- **Stage 3 getter and setter declarations are separate**; a single decorator cannot cleanly cover both unless it checks the context kind.
- **Auto-accessor declarations must handle a `{ get, set }` pair**; they cannot assume only a getter or only a setter exists.
- **Do not mix systems**; the `experimentalDecorators` flag selects one and disables the other.

### Annotated Complete Step by Step Code Examples

#### Example 1: Legacy declaration — direct and factory

```typescript
// example_legacy_declaration.ts
// Step 1: Direct legacy accessor decorator.
function enumerable(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
): PropertyDescriptor {
  descriptor.enumerable = true;
  return descriptor;
}

// Step 2: Legacy decorator factory with configuration.
function logged(prefix: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ): PropertyDescriptor {
    const original = descriptor.get;
    descriptor.get = function () {
      const v = original?.call(this);
      console.log(`${prefix} ${propertyKey} = ${v}`);
      return v;
    };
    return descriptor;
  };
}

// Step 3: Apply both.
class Person {
  private _name = "Alice";

  @logged("GET:")
  @enumerable
  get name() {
    return this._name;
  }
}

const p = new Person();
console.log(p.name);
// Expected:
// GET: name = Alice
// Alice
```

**Expected Output:**
```
GET: name = Alice
Alice
```

**Why this output occurs.** `@enumerable` is a direct decorator that sets `descriptor.enumerable = true`. `@logged("GET:")` is a factory that wraps the original getter with logging. Execution is bottom-to-top: `enumerable` runs first, then `logged` wraps the getter. Accessing `p.name` triggers the logged getter.

---

#### Example 2: Stage 3 declaration — getter and auto-accessor factories

```typescript
// example_stage3_declaration.ts
// Step 1: Stage 3 getter factory that transforms the value.
function formatted(format: (v: string) => string) {
  return function <This, Value extends string>(
    getter: (this: This) => Value,
    _context: ClassGetterDecoratorContext<This, Value>
  ): (this: This) => Value {
    return function (this: This): Value {
      return format(getter.call(this)) as Value;
    };
  };
}

// Step 2: Stage 3 auto-accessor factory that logs writes.
function loggedWrites<This, Value>(
  value: { get: (this: This) => Value; set: (this: This, v: Value) => void },
  context: ClassAccessorDecoratorContext<This, Value>
) {
  return {
    get(this: This): Value {
      return value.get.call(this);
    },
    set(this: This, v: Value): void {
      console.log(`[SET] ${String(context.name)} = ${v}`);
      value.set.call(this, v);
    },
  };
}

// Step 3: Apply both.
class User {
  private _name = "alice";

  @formatted((v) => v.toUpperCase())
  get name() {
    return this._name;
  }

  @loggedWrites
  accessor age = 30;
}

const u = new User();
console.log(u.name); // Expected: ALICE
u.age = 31;          // Expected: [SET] age = 31
```

**Expected Output:**
```
ALICE
[SET] age = 31
```

**Why this output occurs.** The getter decorator returns a replacement getter that uppercases the value. The auto-accessor decorator returns a replacement `{ get, set }` pair whose setter logs the write before delegating to the original setter. Both preserve `this` via `call(this)`.

---

### Real-World Cases

- **MobX `@observable` factory**: in Stage 3 mode, applied to `accessor` fields to create observable properties with change tracking.
- **Validation factories**: `@min(0)`, `@max(100)`, `@pattern(/.../)` applied to auto-accessors to enforce constraints on assignment.
- **Serialization factories**: `@serialize({ name: "custom_name" })` applied to getters to control JSON output.

---

## 3. Accessor Decorator Usage

### Definitions

**Core Definition.** **Accessor decorator usage** is the application of a declared accessor decorator to a getter, setter, or auto-accessor via `@decoratorName` or `@decoratorFactory(args)`, triggering the decorator’s execution at class definition time.

**Technical Definition.** Usage consists of placing the `@` expression **immediately before** the accessor declaration. The decorator expression is evaluated (including any factory call), and the resulting decorator function is invoked with the accessor information (legacy: `(target, propertyKey, descriptor)`; Stage 3 getter: `(getter, ClassGetterDecoratorContext)`; Stage 3 setter: `(setter, ClassSetterDecoratorContext)`; Stage 3 auto-accessor: `({ get, set }, ClassAccessorDecoratorContext)`). Multiple decorators may be stacked; **evaluation is top-to-bottom**, **execution is bottom-to-top**.

**Beginner-Friendly Explanation.** Using an accessor decorator means **putting `@name` right above the getter, setter, or `accessor` field**. When the class is defined, the decorator runs automatically. If it is a factory, you call the factory with parentheses (`@clamp(0, 100)`), and it returns the actual decorator. When you stack decorators, the one **closest to the accessor runs first**.

### Purposes

- **To apply reusable read/write interception** to accessors without modifying their bodies.
- **To compose multiple accessor decorators** on one accessor, each handling a distinct concern (validation, logging, caching).
- **To configure decorators** through factory arguments at the usage site.
- **To declaratively attach property-level policies** adjacent to the accessor declaration.

### Syntax Rules and Structure

**General usage syntax.**

```
@decoratorName
get propertyName() { ... }

@decoratorFactory(options)
accessor propertyName = initialValue;

@first
@second("option")
get propertyName() { ... }
```

**Component breakdown.**

- `@decoratorName` — direct decorator usage.
- `@decoratorFactory(options)` — factory usage; the factory is called, and its return value is the decorator.
- Stacking — multiple decorators on separate lines; execution is bottom-to-top.

**Syntax rules.**

- The decorator must be **immediately above** the accessor declaration.
- Decorator expressions are restricted to **identifiers, dotted names, and calls**.
- **Evaluation order**: top-to-bottom. **Execution order**: bottom-to-top.
- An accessor may have **multiple decorators**; each receives the accessor (or the result of the decorator below it).

**Constraints and limitations.**

- **Decorator factories must be called** with parentheses; `@clamp` without `()` passes the factory itself, which is usually incorrect.
- **Execution order matters**: if a validation decorator and a logging decorator are stacked, the order determines whether invalid values are logged before being rejected.
- **Legacy and Stage 3 usage is syntactically identical** but semantically incompatible; the same `@` syntax selects different behavior based on the compiler flag.
- **Stage 3 getter and setter decoration are independent**; decorating a `get` does not decorate the corresponding `set`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Stacked legacy accessor decorators

```typescript
// example_usage_stacked_legacy.ts
// Step 1: Define two legacy accessor decorators.
function first(target: any, key: string, desc: PropertyDescriptor): PropertyDescriptor {
  console.log("first applied");
  return desc;
}

function second(target: any, key: string, desc: PropertyDescriptor): PropertyDescriptor {
  console.log("second applied");
  return desc;
}

// Step 2: Stack them on a getter.
class Example {
  @first
  @second
  get value() {
    return 42;
  }
}

// Expected:
// second applied
// first applied
```

**Expected Output:**
```
second applied
first applied
```

**Why this output occurs.** `@second` is closest to the getter, so it executes first. Then `@first` executes. Both receive the same descriptor and return it unchanged. The order demonstrates bottom-to-top execution.

---

#### Example 2: Stacked Stage 3 auto-accessor decorators

```typescript
// example_usage_stacked_stage3.ts
// Step 1: Define two Stage 3 auto-accessor decorators.
function trim<This, Value extends string>(
  value: { get: (this: This) => Value; set: (this: This, v: Value) => void },
  _context: ClassAccessorDecoratorContext<This, Value>
) {
  return {
    get(this: This): Value {
      return value.get.call(this);
    },
    set(this: This, v: Value): void {
      value.set.call(this, v.trim() as Value);
    },
  };
}

function lowercase<This, Value extends string>(
  value: { get: (this: This) => Value; set: (this: This, v: Value) => void },
  _context: ClassAccessorDecoratorContext<This, Value>
) {
  return {
    get(this: This): Value {
      return value.get.call(this);
    },
    set(this: This, v: Value): void {
      value.set.call(this, v.toLowerCase() as Value);
    },
  };
}

// Step 2: Stack them.
class User {
  @trim
  @lowercase
  accessor username = "  ALICE  ";
}

// Step 3: Set a new value and observe transformation order.
const u = new User();
u.username = "  BOB  ";
console.log(u.username); // Expected: bob
```

**Expected Output:**
```
bob
```

**Why this output occurs.** `@lowercase` executes first (closest to the accessor), lowercasing `"  BOB  "` to `"  bob  "`. Then `@trim` executes, trimming to `"bob"`. The setters compose: `trim(lowercase(value))`. Order matters for transformations where trimming before or after lowercasing affects the result.

---

### Real-World Cases

- **MobX `@observable accessor`**: applied to auto-accessors to track reads and writes; the decorator returns a replacement `{ get, set }` pair that integrates with MobX’s reactivity system.
- **Lit `@property({ type: Number })`**: applied to auto-accessors to define reactive numeric properties with type coercion on set.
- **Validation stacks**: `@required @minLength(3) accessor name = ""` — multiple decorators compose to enforce multiple constraints on assignment.

---

## 4. Accessor Decorator Best Practices

### Definitions

**Core Definition.** **Accessor decorator best practices** are guidelines for declaring and using accessor decorators in ways that preserve type safety, maintainability, composability, and compatibility with the chosen decorator system.

**Technical Definition.** Best practices center on **system selection** (legacy for frameworks requiring `emitDecoratorMetadata` or traditional `get`/`set` decorators, Stage 3 for new projects and reactive libraries), **precise typing** (using `<This, Value>` generics and typed contexts in Stage 3), **`this` preservation** (`getter.call(this)` / `setter.call(this, v)`), **correct handling of the `{ get, set }` pair** in auto-accessor decorators, **single responsibility** (one concern per decorator), and **order awareness** (decorators apply bottom-to-top). When intercepting reads and writes, prefer the **`accessor` keyword** in Stage 3 over decorating manual getters and setters separately, because it provides a clean `{ get, set }` interface and compiler-generated backing storage.

**Beginner-Friendly Explanation.** Best practices mean **choosing the right system** for your project, **preserving `this` in every wrapper**, and **using the `accessor` keyword** when you need to intercept both reads and writes in Stage 3. Keep each decorator focused on one job (validation, logging, caching). Remember that order matters: the decorator closest to the accessor runs first. If you depend on a framework that uses `emitDecoratorMetadata` or decorates traditional `get`/`set` declarations, stay on legacy decorators.

### Purposes

- **To prevent `this`-loss bugs** by consistently forwarding the runtime receiver to the original accessor.
- **To preserve type safety** by using precise generic signatures and typed contexts in Stage 3.
- **To choose a future-proof decorator system** aligned with the project’s ecosystem dependencies.
- **To keep accessor decorators composable and predictable** by following execution-order rules and single responsibility.
- **To leverage the `accessor` keyword correctly** for read/write interception, avoiding the limitations of separate getter/setter decoration.

### Syntax Rules and Structure

**System selection checklist.**

```
Depends on Angular, NestJS, TypeORM, or frameworks using emitDecoratorMetadata?
  ├── Yes → experimentalDecorators: true (legacy system)
  │         Decorate traditional get/set declarations.
  │         Use PropertyDescriptor mutation.
  └── No  → Stage 3 decorators (default in TS 5.0+)
            Use the `accessor` keyword for read/write interception.
            Return a replacement { get, set } pair.
```

**Precise Stage 3 auto-accessor typing (recommended).**

```
function decorator<This, Value>(
  value: { get: (this: This) => Value; set: (this: This, v: Value) => void },
  context: ClassAccessorDecoratorContext<This, Value>
): { get?: (this: This) => Value; set?: (this: This, v: Value) => void } | void
```

**`this` preservation in wrappers (mandatory).**

```
return value.get.call(this);          // getter
value.set.call(this, newValue);       // setter
```

**Constraints and limitations.**

- **Do not mix systems**; the `experimentalDecorators` flag selects one and disables the other.
- **Do not decorate `get` and `set` separately** in Stage 3 when you need unified read/write interception; use `accessor` instead.
- **Do not use `addInitializer` to transform values** in auto-accessor decorators; return a replacement `{ get, set }` pair.
- **Do not forget that `accessor` requires `target: ES2015` or later** (or downleveling by TypeScript).

### Annotated Complete Step by Step Code Examples

#### Example 1: Correct legacy accessor decoration with `this` preservation

```typescript
// example_best_legacy.ts
// ✅ Legacy accessor decorator with `this` preservation and factory config.
function logged(prefix: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ): PropertyDescriptor {
    const originalGet = descriptor.get;
    if (originalGet) {
      descriptor.get = function () {
        const v = originalGet.call(this);   // preserve `this`
        console.log(`${prefix} ${propertyKey} = ${v}`);
        return v;
      };
    }
    return descriptor;
  };
}

class Person {
  private _name = "Alice";

  @logged("GET:")
  get name() {
    return this._name;
  }
}

const p = new Person();
console.log(p.name);
// Expected:
// GET: name = Alice
// Alice
```

**Expected Output:**
```
GET: name = Alice
Alice
```

**Why this output occurs.** The decorator checks whether a getter exists and wraps it with a function that logs the value. `originalGet.call(this)` preserves the runtime receiver, so `this._name` resolves correctly. The descriptor is returned to install the wrapper.

---

#### Example 2: Correct Stage 3 auto-accessor with validation and `this` preservation

```typescript
// example_best_stage3.ts
// ✅ Stage 3 auto-accessor decorator with precise typing and `this` preservation.
function range(min: number, max: number) {
  return function <This, Value extends number>(
    value: { get: (this: This) => Value; set: (this: This, v: Value) => void },
    _context: ClassAccessorDecoratorContext<This, Value>
  ) {
    return {
      get(this: This): Value {
        return value.get.call(this);
      },
      set(this: This, v: Value): void {
        if (v < min || v > max) {
          throw new RangeError(`Value must be between ${min} and ${max}`);
        }
        value.set.call(this, v);
      },
    };
  };
}

class Progress {
  @range(0, 100)
  accessor percent = 50;
}

const p = new Progress();
p.percent = 75;
console.log(p.percent); // Expected: 75
try {
  p.percent = 150;
} catch (e) {
  console.log((e as Error).message); // Expected: Value must be between 0 and 100
}
```

**Expected Output:**
```
75
Value must be between 0 and 100
```

**Why this output occurs.** The decorator returns a replacement `{ get, set }` pair. The setter validates the value and throws if it is out of range. Valid values are delegated to the original setter via `value.set.call(this, v)`. The getter passes through unchanged. This pattern keeps validation logic separate from the class and preserves `this`.

---

#### Example 3: Single-responsibility decorators and order awareness

```typescript
// example_best_single.ts
// ✅ Each decorator does ONE thing.
function trim<This, Value extends string>(
  value: { get: (this: This) => Value; set: (this: This, v: Value) => void },
  _context: ClassAccessorDecoratorContext<This, Value>
) {
  return {
    get(this: This): Value {
      return value.get.call(this);
    },
    set(this: This, v: Value): void {
      value.set.call(this, v.trim() as Value);
    },
  };
}

function uppercase<This, Value extends string>(
  value: { get: (this: This) => Value; set: (this: This, v: Value) => void },
  _context: ClassAccessorDecoratorContext<This, Value>
) {
  return {
    get(this: This): Value {
      return value.get.call(this);
    },
    set(this: This, v: Value): void {
      value.set.call(this, v.toUpperCase() as Value);
    },
  };
}

class Code {
  @uppercase
  @trim
  accessor value = "  abc  ";
}

const c = new Code();
console.log(c.value); // Expected: ABC
```

**Expected Output:**
```
ABC
```

**Why this output occurs.** `@trim` executes first (closest to the accessor), trimming `"  abc  "` to `"abc"`. Then `@uppercase` executes, producing `"ABC"`. Each decorator has a single responsibility, and the composition order determines the final result. If the order were reversed, the result would be `"ABC"` as well in this case, but for transformations where order affects the outcome (e.g., validation before transformation), explicit ordering is essential.

---

### Real-World Cases

- **MobX standard decorators** require `@observable accessor` rather than `@observable field` because the accessor provides the get/set pair that the decorator wraps; using the wrong form is a documented anti-pattern.
- **Lit reactive properties** use auto-accessor decorators to intercept reads and writes for change detection; decorating plain fields does not provide the same interception capability.
- **Legacy frameworks** (Angular, NestJS, TypeORM) that depend on `emitDecoratorMetadata` or traditional getter/setter decoration should remain on legacy decorators until the frameworks support Stage 3.
- **Validation libraries** prefer single-responsibility auto-accessor decorators (`@min`, `@max`, `@pattern`) that compose predictably, rather than monolithic decorators that mix multiple concerns.

---

## References

- Decorators — TypeScript (Advanced) - https://mintlify.wiki/Microsoft/typescript/advanced/decorators
- Use Stage 3 Decorators with Decorator Context for Metaprogramming - https://raw.githubusercontent.com/pproenca/dot-skills/cf93c57cac89d6fc3e4194686000411567f5caf3/skills/.experimental/typescript-advanced-patterns/references/mod-stage-3-decorators.md
- Support for auto-accessor fields from the Stage 3 Decorators proposal (microsoft/TypeScript PR #49705) - https://github.com/microsoft/TypeScript/pull/49705
- TypeScript 78 Decorators — Stage 3 Standard (KandZ) - https://tuts.kandz.me/typescript-78-🔷-decorators-stage-3-standard/
- LegacyAccessorDecoratorFunction — @decorators/types (JSR) - https://jsr.io/@decorators/types/doc/legacy/accessor/~/LegacyAccessorDecoratorFunction
- ClassAccessorDecoratorFunction — @decorators/types (JSR) - https://jsr.io/@decorators/types/doc/accessor/~/ClassAccessorDecoratorFunction
- ClassGetterDecoratorContext / ClassSetterDecoratorContext — TypeScript lib definitions (Oracle Docs mirror) - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html (reference to `ClassMemberDecoratorContext` union)
- TypeScript Decorators in Brief — Refine - https://refine.dev/blog/typescript-decorators/