# Method Decorators in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **method decorator** is a decorator applied to a method declaration within a class. It is evaluated at class definition time and may **observe, modify, or replace** the method definition. TypeScript supports two incompatible method-decorator systems: **legacy experimental decorators** (based on an early Stage 2 proposal, enabled by `experimentalDecorators: true`) and **Stage 3 decorators** (the TC39-standard system, default in TypeScript 5.0+).

**Technical Definition.** In the **legacy system**, a method decorator has the signature `(target: any, propertyKey: string, descriptor: PropertyDescriptor) => PropertyDescriptor | void`. It receives the class prototype (or constructor for static methods), the method name, and the property descriptor. It may mutate `descriptor.value` to wrap the original method and must return the descriptor to install the modification. In the **Stage 3 system**, a method decorator has the signature `<This, Args extends unknown[], Return>(value: (this: This, ...args: Args) => Return, context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>) => ((this: This, ...args: Args) => Return) | void`. It receives the **method function itself** and a **typed context object** exposing `kind`, `name`, `static`, `private`, and `addInitializer`. A decorator may return a **replacement function** to replace the method, or `void` to leave it unchanged. Method decorators execute **when the class is declared**, not when instances are created.

**Beginner-Friendly Explanation.** A method decorator is a **function you place above a method with `@`** that runs when the class is defined. It receives the method (either indirectly through a descriptor in the old system, or directly as a function in the new system) and can **wrap it with extra behavior** — logging, timing, caching, retrying, access control — without changing the method’s body. TypeScript has an **old style** (experimental, used by NestJS and older frameworks) and a **new standard style** (Stage 3, the default since TypeScript 5.0). They look identical at the call site (`@log`) but receive different arguments, and you cannot mix them in the same project.

---

### Key Characteristics

| Characteristic | Legacy (Experimental) | Stage 3 (Standard) |
|---|---|---|
| **Compiler flag** | `experimentalDecorators: true` required | No flag required in TS 5.0+ with `target: ES2022`+ |
| **Signature** | `(target, propertyKey, descriptor)` | `(originalMethod, context)` |
| **What you receive** | The prototype, the method name, and the descriptor | The method function itself and a typed `ClassMethodDecoratorContext` |
| **How you modify** | Mutate `descriptor.value` and return the descriptor | **Return a replacement function** |
| **Context object** | None (relies on `reflect-metadata`) | `context.name`, `context.static`, `context.private`, `context.addInitializer` |
| **Type safety** | Broad (`any`, `PropertyDescriptor`) | Precise generics (`This`, `Args`, `Return`) |
| **`this` binding** | Manual (`original.apply(this, args)`) | Manual in the wrapper; `addInitializer` enables auto-binding |
| **Primary ecosystem** | NestJS, TypeGraphQL, legacy MobX integrations | New projects, Lit, MobX (standard mode), Effect |

---

### Prerequisites

- **Class methods and prototypes**: understanding that instance methods live on the prototype and `this` is bound at call time.
- **Property descriptors**: `value`, `writable`, `enumerable`, `configurable`.
- **Function types and generics**: `<This, Args extends unknown[], Return>` syntax.
- **Basic decorator syntax** (`@expression`) and the distinction between decorator **evaluation** and **execution**.
- **A TypeScript environment** (`tsc` 5.0+ or the TypeScript Playground) and familiarity with `tsconfig.json`.

---

### Related Programming Areas

- **Aspect-oriented programming (AOP)**: method decorators inject cross-cutting concerns (logging, timing, caching, retry) around method execution.
- **Dependency injection and routing**: legacy method decorators underpin NestJS route handlers (`@Get`, `@Post`) and injection frameworks.
- **Memoization and performance instrumentation**: decorators wrap methods to cache results or measure execution time without modifying method bodies.
- **Type erasure**: method decorators are runtime constructs, but their **type signatures** are erased; only the decorator function survives compilation.
- **The decorator pattern (object-oriented design)**: method decorators are a native syntactic implementation of wrapper-based behavior extension.

---

### Core Concepts / Features

The following four sections cover the requested concepts. Legacy and Stage 3 method decorators are treated together where their behavior aligns, and distinguished explicitly where they differ.

---

## 1. Method Decorators

### Definitions

**Core Definition.** **Method decorators** are decorator functions applied to a class method. They execute when the class is defined and may **observe, modify, or replace** the method’s implementation.

**Technical Definition.** A method decorator is invoked with the method information (legacy: descriptor; Stage 3: the function plus context) and may return a **replacement function** (Stage 3) or a modified **descriptor** (legacy). The replacement wraps or substitutes the original method. Decorators **do not change the static type** of the method unless the return type is explicitly declared in the replacement function. Because method decorators run at **class definition time**, they are well suited to cross-cutting concerns that should apply uniformly across methods. In Stage 3, the `context` object provides the method’s `name`, `static` and `private` status, and `addInitializer`, which registers a callback that runs during class initialization — the standard mechanism for per-instance setup such as `this` binding.

**Beginner-Friendly Explanation.** A method decorator is **a wrapper factory that runs once, when the class is defined**. In the old system, you get a descriptor and swap out `descriptor.value` with a wrapper function. In the new system, you get the method function directly and **return a new function** that calls it. Either way, the goal is the same: **add behavior around the method without touching its body**. Logging, timing, caching, and retry are the classic use cases.

### Purposes

- **To extract cross-cutting concerns** (logging, timing, caching, retry, authorization) from method bodies, keeping business logic focused.
- **To wrap method execution uniformly** across many methods with a single reusable decorator.
- **To instrument method calls** without modifying the method’s implementation or duplicating wrapper code at call sites.
- **To enforce behavioral policies** such as read-only access, retry limits, or memoization boundaries.
- **To bind methods to instances** (Stage 3) through `addInitializer`, preventing `this` loss when methods are passed as callbacks.

### Syntax Rules and Structure

**General syntax (legacy method decorator).**

```
function methodDecorator(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
): PropertyDescriptor | void {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    // before
    const result = original.apply(this, args);
    // after
    return result;
  };
  return descriptor;
}
```

**Component breakdown.**

- `target: any` — the **prototype** for instance methods, or the **constructor** for static methods.
- `propertyKey: string` — the method name.
- `descriptor: PropertyDescriptor` — the property descriptor; `descriptor.value` is the original method function.
- `original.apply(this, args)` — preserves the runtime `this` when invoking the original method.
- Returning `descriptor` — installs the modified method; returning `void` leaves the method unchanged.

**General syntax (Stage 3 method decorator).**

```
function methodDecorator<This, Args extends unknown[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
): (this: This, ...args: Args) => Return {
  return function (this: This, ...args: Args): Return {
    // before
    const result = originalMethod.call(this, ...args);
    // after
    return result;
  };
}
```

**Component breakdown.**

- `originalMethod` — the method function itself.
- `context: ClassMethodDecoratorContext<...>` — typed metadata: `kind: "method"`, `name`, `static`, `private`, `addInitializer`.
- Returning a **function** — replaces the method.
- `originalMethod.call(this, ...args)` — preserves the runtime `this`.

**Syntax rules.**

- The `@` sigil precedes the decorator expression, placed **immediately before** the method declaration.
- Multiple method decorators may be stacked; **evaluation is top-to-bottom**, **execution is bottom-to-top** (the decorator closest to the method applies first).
- Method decorators **cannot** be applied to arrow-function class properties in the legacy system without care; arrow functions do not have their own `this`, so decorating them as methods is problematic.
- In Stage 3, the returned function **must be assignable** to the original method type; the generic signature enforces this.

**Constraints and limitations.**

- **Static type structure cannot be changed**: adding a method at runtime is invisible to the type checker unless declared via interface merging.
- **`this` must be preserved explicitly** in wrappers (`original.apply(this, ...)` or `originalMethod.call(this, ...)`).
- **Legacy and Stage 3 systems are incompatible**; the `experimentalDecorators` flag selects one and disables the other.
- **Parameter decorators are legacy-only**; Stage 3 has no parameter decorator form.
- **Stack traces may show the wrapper**, not the original method name; deliberate effort is needed to preserve names.

### Annotated Complete Step by Step Code Examples

#### Example 1: Legacy logging method decorator

```typescript
// example_legacy_method_log.ts
// Step 1: Define a legacy method decorator.
function log(
  target: any,                          // prototype (or constructor for static)
  propertyKey: string,                  // method name
  descriptor: PropertyDescriptor        // property descriptor
): PropertyDescriptor {
  const original = descriptor.value;    // the original method function

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    const result = original.apply(this, args);  // preserve `this`
    console.log(`Result:`, result);
    return result;
  };

  return descriptor;                    // install the modified descriptor
}

// Step 2: Apply the decorator.
class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

// Step 3: Use the method.
const calc = new Calculator();
calc.add(2, 3);
// Expected:
// Calling add with [ 2, 3 ]
// Result: 5
```

**Setup.** Enable legacy decorators:
```json
{ "compilerOptions": { "experimentalDecorators": true } }
```
Compile with `npx tsc` and run the output with Node.js.

**Expected Output:**
```
Calling add with [ 2, 3 ]
Result: 5
```

**Why this output occurs.** The decorator replaces `descriptor.value` with a wrapper that logs before and after invoking the original method. Because `original.apply(this, args)` preserves the runtime `this`, the method works correctly when called on an instance. Returning the descriptor installs the wrapper on the prototype.

---

#### Example 2: Stage 3 timing method decorator with typed generics

```typescript
// example_stage3_method_timing.ts
// Step 1: Define a Stage 3 method decorator with precise typing.
function timed<This, Args extends unknown[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
): (this: This, ...args: Args) => Return {
  const methodName = String(context.name);

  return function (this: This, ...args: Args): Return {
    const start = performance.now();
    const result = originalMethod.call(this, ...args);  // preserve `this`
    const duration = performance.now() - start;
    console.log(`${methodName} took ${duration.toFixed(2)}ms`);
    return result;
  };
}

// Step 2: Apply it to a method.
class ReportService {
  @timed
  generate(total: number): string {
    let sum = 0;
    for (let i = 0; i < total; i++) sum += i;
    return `Sum: ${sum}`;
  }
}

// Step 3: Call the method.
const service = new ReportService();
console.log(service.generate(1000));
// Expected:
// generate took <X>ms
// Sum: 499500
```

**Setup.** No special flag is needed in TypeScript 5.0+ with `target: ES2022` or `ESNext`.

**Expected Output (representative):**
```
generate took 0.05ms
Sum: 499500
```

**Why this output occurs.** The decorator returns a replacement function that measures execution time around the original method. The generic signature preserves the original method’s `this`, parameter types, and return type, so `service.generate(1000)` is fully type-checked. The exact timing varies by machine; the sum is deterministic.

---

#### Example 3: Stage 3 `@bound` decorator using `addInitializer`

```typescript
// example_stage3_method_bound.ts
// Step 1: Define a Stage 3 method decorator that binds `this` per instance.
function bound<This, Args extends unknown[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
): (this: This, ...args: Args) => Return {
  context.addInitializer(function (this: This) {
    // Bind the method to the instance during initialization.
    (this as any)[context.name] = originalMethod.bind(this);
  });
  return originalMethod;   // leave the prototype method unchanged
}

// Step 2: Apply it.
class Timer {
  private count = 0;

  @bound
  tick() {
    this.count++;
    console.log(this.count);
  }
}

// Step 3: Extract the method and call it standalone.
const timer = new Timer();
const fn = timer.tick;
fn(); // Expected: 1 (works because `this` was bound)
```

**Expected Output:**
```
1
```

**Why this output occurs.** `context.addInitializer` registers a callback that runs during instance initialization. That callback binds `originalMethod` to the instance and assigns the bound function to `this[context.name]`. When `fn` is extracted and called without a receiver, `this` still refers to the `Timer` instance, so `this.count` increments correctly. Without `@bound`, `fn()` would lose `this` and fail.

---

### Real-World Cases

- **NestJS route handlers**: `@Get(":id")`, `@Post()`, and `@UseGuards(...)` are legacy method decorators that register HTTP routes and middleware at the method level.
- **Automatic memoization**: A `@memoize` method decorator caches results keyed by arguments, using a `WeakMap` attached to the instance so instances remain garbage-collectable.
- **Retry logic**: A `@retry(3)` Stage 3 factory decorator wraps an async method and retries on failure, keeping retry policy out of business logic.
- **Timing and metrics**: A `@timed` decorator instruments method execution for performance monitoring across service classes.

---

## 2. Method Decorator Declaration

### Definitions

**Core Definition.** **Method decorator declaration** is the act of writing the decorator function — defining its parameters, return type, and behavior — before it is applied to a method with `@`.

**Technical Definition.** In the **legacy system**, a method decorator declaration is a function `(target: any, propertyKey: string, descriptor: PropertyDescriptor) => PropertyDescriptor | void`. In the **Stage 3 system**, it is a generic function `<This, Args extends unknown[], Return>(originalMethod: (this: This, ...args: Args) => Return, context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>) => ((this: This, ...args: Args) => Return) | void`. The declaration may be a **direct decorator** (a function used as-is) or a **decorator factory** (a function that returns a decorator function, allowing configuration through arguments).

**Beginner-Friendly Explanation.** Declaring a method decorator means **writing the function that will run when the class is defined**. In the old system, you write a function that receives the prototype, the method name, and the descriptor, and you modify the descriptor. In the new system, you write a generic function that receives the method and a context object, and you **return a replacement function**. If you want options (like a retry count or a log prefix), you write a **factory** — a function that takes the options and returns the decorator.

### Purposes

- **To define reusable method-level behavior** that can be applied to many methods across classes.
- **To encapsulate configuration** through decorator factories, keeping call sites concise and readable.
- **To type the decorator precisely** (especially in Stage 3) so the compiler verifies that the replacement is compatible with the original method.
- **To separate cross-cutting logic** from the methods it decorates, preserving single responsibility.

### Syntax Rules and Structure

**Legacy declaration (direct decorator).**

```
function decoratorName(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
): PropertyDescriptor | void {
  // observe or replace descriptor.value
}
```

**Legacy declaration (factory).**

```
function decoratorFactory(config: ConfigType) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ): PropertyDescriptor | void {
    // use config
  };
}
```

**Stage 3 declaration (direct decorator).**

```
function decoratorName<This, Args extends unknown[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
): (this: This, ...args: Args) => Return {
  // return a replacement function
}
```

**Stage 3 declaration (factory).**

```
function decoratorFactory(config: ConfigType) {
  return function <This, Args extends unknown[], Return>(
    originalMethod: (this: This, ...args: Args) => Return,
    context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
  ): (this: This, ...args: Args) => Return {
    // use config
  };
}
```

**Syntax rules.**

- The **generic signature** `<This, Args extends unknown[], Return>` preserves the original method’s `this` type, parameter tuple, and return type.
- `context: ClassMethodDecoratorContext<...>` is the typed context for method decorators; other kinds use `ClassGetterDecoratorContext`, `ClassFieldDecoratorContext`, etc..
- The `context` parameter may be named `_context` if unused.
- Factory declarations return a decorator function; the factory itself is **not** the decorator.
- The returned function must be assignable to the original method type; TypeScript enforces this in Stage 3.

**Constraints and limitations.**

- **Legacy declarations lack type precision**: `any`, `string`, and `PropertyDescriptor` do not encode the method’s argument or return types.
- **Stage 3 declarations must return a function** or `void`; returning an unrelated value is a type error.
- **Factories should be pure** where possible; avoid shared mutable state across decorations.
- **`context.addInitializer` is the correct place for setup logic** tied to a decorated member, not the decorator body itself.

### Annotated Complete Step by Step Code Examples

#### Example 1: Legacy declaration — direct and factory

```typescript
// example_legacy_declaration.ts
// Step 1: Direct legacy method decorator.
function log(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
): PropertyDescriptor {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`[LOG] ${propertyKey}`);
    return original.apply(this, args);
  };
  return descriptor;
}

// Step 2: Legacy decorator factory with configuration.
function prefix(label: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ): PropertyDescriptor {
    const original = descriptor.value;
    descriptor.value = function (...args: any[]) {
      console.log(`${label} ${propertyKey}`);
      return original.apply(this, args);
    };
    return descriptor;
  };
}

// Step 3: Apply both.
class Service {
  @prefix("DEBUG:")
  @log
  run() {
    console.log("running");
  }
}

new Service().run();
// Expected:
// DEBUG: run
// [LOG] run
// running
```

**Expected Output:**
```
DEBUG: run
[LOG] run
running
```

**Why this output occurs.** `@log` is a direct decorator; `@prefix("DEBUG:")` is a factory. Execution is **bottom-to-top**: `log` runs first, wrapping the original `run`, then `prefix` wraps the logged version. When `run()` is called, the outermost wrapper (`prefix`) logs `DEBUG: run`, then calls the inner wrapper (`log`), which logs `[LOG] run`, then calls the original method, which logs `running`.

---

#### Example 2: Stage 3 declaration with generic typing and a factory

```typescript
// example_stage3_declaration.ts
// Step 1: Stage 3 factory that preserves the method signature.
function retry(maxAttempts: number) {
  return function <This, Args extends unknown[], Return>(
    originalMethod: (this: This, ...args: Args) => Promise<Return>,
    context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Promise<Return>>
  ): (this: This, ...args: Args) => Promise<Return> {
    const methodName = String(context.name);

    return async function (this: This, ...args: Args): Promise<Return> {
      let lastError: unknown;
      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
          return await originalMethod.call(this, ...args);
        } catch (err) {
          lastError = err;
          console.log(`${methodName} attempt ${attempt} failed, retrying...`);
        }
      }
      throw lastError;
    };
  };
}

// Step 2: Apply the factory.
class ApiClient {
  @retry(3)
  async fetchUser(id: string): Promise<string> {
    if (Math.random() < 0.5) {
      throw new Error("network error");
    }
    return `User:${id}`;
  }
}

// Step 3: Call the method.
const client = new ApiClient();
client.fetchUser("42").then(console.log).catch(console.error);
```

**Expected Output (representative, since retries are probabilistic):**
```
User:42
```

**Why this output occurs.** The factory captures `maxAttempts = 3` and returns a Stage 3 decorator. The decorator returns an async wrapper that retries the original method up to three times, logging each failed attempt. If all attempts fail, the last error is thrown. The generic signature preserves the `Promise<string>` return type, so `.then(console.log)` is type-checked.

---

### Real-World Cases

- **NestJS method decorators**: `@Get()`, `@Post()`, and `@HttpCode(201)` are legacy factory declarations that configure route metadata at the method level.
- **Stage 3 `@memoize` factory**: a factory that takes a cache key resolver and returns a method decorator that caches results per instance.
- **Stage 3 `@authorize(role)` factory**: a factory that returns a decorator checking the current user’s role before invoking the method.

---

## 3. Method Decorator Usage

### Definitions

**Core Definition.** **Method decorator usage** is the application of a declared method decorator to a class method via `@decoratorName` or `@decoratorFactory(args)`, triggering the decorator’s execution at class definition time.

**Technical Definition.** Usage consists of placing the `@` expression **immediately before** a method declaration. The decorator expression is evaluated (including any factory call), and the resulting decorator function is invoked with the method information (legacy: descriptor; Stage 3: the function and context). Multiple decorators may be stacked; their **evaluation order** is top-to-bottom, but their **execution order** is bottom-to-top: `@A @B method` applies `B` first, then `A` to the result, equivalent to `A(B(method))`.

**Beginner-Friendly Explanation.** Using a method decorator means **putting `@name` right above the method**. When the class is defined, the decorator runs automatically. You do not call it yourself. If the decorator is a factory, you call the factory with parentheses (`@retry(3)`), and it returns the actual decorator that runs. When you stack decorators, the one **closest to the method runs first**, even though the expressions are written top to bottom — like an onion, the innermost layer is applied first.

### Purposes

- **To apply reusable behavior** to methods without modifying their bodies.
- **To compose multiple decorators** on one method, each handling a distinct concern.
- **To configure decorators** through factory arguments at the usage site.
- **To declaratively attach routing, validation, caching, or logging policies** adjacent to the method implementation, keeping the contract visible at the definition site.

### Syntax Rules and Structure

**General usage syntax.**

```
@decoratorName
methodName() { ... }

@decoratorFactory(options)
methodName() { ... }

@first
@second("option")
methodName() { ... }
```

**Component breakdown.**

- `@decoratorName` — direct decorator usage; the function is the decorator.
- `@decoratorFactory(options)` — factory usage; the factory is called, and its return value is the decorator.
- Stacking — multiple decorators on separate lines; execution is bottom-to-top.

**Syntax rules.**

- The decorator must be **immediately above** the method declaration (or on the same line, before the method name).
- Decorator expressions are restricted to **identifiers, dotted names, and calls**.
- **Evaluation order**: top-to-bottom. **Execution order**: bottom-to-top (the decorator closest to the method runs first).
- A method may have **multiple decorators**; each receives the method (or the result of the decorator below it).
- Decorators **cannot** be applied to a method expression used as a value; they apply to method **declarations**.

**Constraints and limitations.**

- **Decorator factories must be called** with parentheses; `@retry` without `()` passes the factory itself as the decorator, which is usually incorrect.
- **Execution order matters**: a decorator that depends on the method’s shape should account for wrappers introduced by decorators below it.
- **Legacy and Stage 3 usage is syntactically identical** but semantically incompatible; the same `@` syntax selects different behavior based on the compiler flag.
- **Arrow-function class properties cannot be reliably decorated as methods** in the legacy system because arrow functions capture `this` lexically and do not have their own `this`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Stacked decorators and execution order (Stage 3)

```typescript
// example_usage_stacked.ts
// Step 1: Define two Stage 3 method decorators.
function first<This, Args extends unknown[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  _context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
) {
  return function (this: This, ...args: Args): Return {
    console.log("first: before");
    const result = originalMethod.call(this, ...args);
    console.log("first: after");
    return result;
  };
}

function second<This, Args extends unknown[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  _context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
) {
  return function (this: This, ...args: Args): Return {
    console.log("second: before");
    const result = originalMethod.call(this, ...args);
    console.log("second: after");
    return result;
  };
}

// Step 2: Stack them (execution bottom-to-top).
class Example {
  @first
  @second
  run() {
    console.log("original");
  }
}

// Step 3: Call the method.
new Example().run();
// Expected:
// first: before
// second: before
// original
// second: after
// first: after
```

**Expected Output:**
```
first: before
second: before
original
second: after
first: after
```

**Why this output occurs.** `@second` is closest to the method, so it wraps the original `run` first. `@first` then wraps the result of `second`. At call time, the outermost wrapper (`first`) runs first, logging `first: before`, then calls `second`, which logs `second: before`, then calls the original, which logs `original`. After the original returns, `second` logs `second: after`, then `first` logs `first: after`. This is function composition: `first(second(run))`.

---

#### Example 2: Factory usage with configuration (Stage 3)

```typescript
// example_usage_factory.ts
// Step 1: Define a Stage 3 factory decorator.
function labeled(label: string) {
  return function <This, Args extends unknown[], Return>(
    originalMethod: (this: This, ...args: Args) => Return,
    context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
  ) {
    const methodName = String(context.name);
    return function (this: This, ...args: Args): Return {
      console.log(`[${label}] entering ${methodName}`);
      return originalMethod.call(this, ...args);
    };
  };
}

// Step 2: Use the factory with different arguments.
class Service {
  @labeled("AUTH")
  login(user: string) {
    console.log(`logging in ${user}`);
  }

  @labeled("DATA")
  fetchData() {
    console.log("fetching data");
  }
}

// Step 3: Call the methods.
const s = new Service();
s.login("alice");
// Expected: [AUTH] entering login
//           logging in alice
s.fetchData();
// Expected: [DATA] entering fetchData
//           fetching data
```

**Expected Output:**
```
[AUTH] entering login
logging in alice
[DATA] entering fetchData
fetching data
```

**Why this output occurs.** `@labeled("AUTH")` and `@labeled("DATA")` each call the factory with different arguments, producing **independent decorators** with captured configuration. Each decorator returns a wrapper that logs its own label before invoking the original method. The factory pattern keeps the usage site concise while allowing per-method customization.

---

### Real-World Cases

- **NestJS routing**: `@Get(":id")` and `@Post()` decorate controller methods, making the HTTP route contract visible next to the handler implementation.
- **NestJS guards and interceptors**: `@UseGuards(AuthGuard)` and `@UseInterceptors(LoggingInterceptor)` are factory usages applied above handler methods.
- **Stage 3 `@retry(3)`**: a factory usage that wraps an async method with configurable retry behavior.
- **Stage 3 `@memoize()`**: a factory usage that applies caching to a method, with the cache scoped to the instance.

---

## 4. Method Decorator Best Practices

### Definitions

**Core Definition.** **Method decorator best practices** are guidelines for declaring and using method decorators in ways that preserve type safety, maintainability, composability, and compatibility with the chosen decorator system.

**Technical Definition.** Best practices center on **precise typing** (using generic `<This, Args, Return>` signatures in Stage 3 rather than broad `any`), **`this` preservation** (`originalMethod.call(this, ...)` or `apply`), **use of `addInitializer` for per-instance setup** rather than side effects in the decorator body, **single responsibility** (one concern per decorator), **order awareness** (decorators apply bottom-to-top), **system selection** (legacy for NestJS/TypeGraphQL dependencies, Stage 3 for new projects), and **stack-trace preservation** where debugging matters.

**Beginner-Friendly Explanation.** Best practices mean **writing method decorators that are safe, predictable, and easy to debug**. Use the **new Stage 3 system** for new projects because it is type-safe and standard. Preserve `this` in every wrapper, or your methods will break when called as callbacks. Use `addInitializer` for binding and setup, not the decorator body. Keep each decorator focused on one job. Remember that decorators run bottom-to-top, so order matters. And choose legacy decorators only if your framework requires them.

### Purposes

- **To preserve type safety** by using precise generic signatures instead of `any`.
- **To prevent `this`-loss bugs** by consistently forwarding the runtime receiver to the original method.
- **To keep decorators composable and order-independent** where possible, documenting order dependence when it cannot be avoided.
- **To choose a future-proof decorator system** aligned with the project’s ecosystem dependencies.
- **To facilitate debugging** by preserving method names and avoiding opaque wrapper layers where possible.

### Syntax Rules and Structure

**Precise Stage 3 typing (recommended).**

```
function decorator<This, Args extends unknown[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
): (this: This, ...args: Args) => Return {
  return function (this: This, ...args: Args): Return {
    return originalMethod.call(this, ...args);   // preserve `this`
  };
}
```

**`this` preservation (mandatory in wrappers).**

```
const result = original.apply(this, args);              // legacy
const result = originalMethod.call(this, ...args);      // Stage 3
```

**`addInitializer` for per-instance setup (Stage 3).**

```
context.addInitializer(function (this: This) {
  (this as any)[context.name] = originalMethod.bind(this);
});
```

**System selection checklist.**

```
Depends on NestJS, TypeGraphQL, or legacy MobX integrations?
  ├── Yes → experimentalDecorators: true (legacy system)
  └── No  → Stage 3 decorators (default in TS 5.0+)
```

**Constraints and limitations.**

- **Do not mix systems**; the `experimentalDecorators` flag selects one and disables the other.
- **Do not use arrow functions as decorated methods**; they capture `this` lexically and cannot be reliably wrapped with runtime `this`.
- **Do not rely on decorator-added members without interface merging**; the type checker will not see runtime additions.
- **Do not place setup logic in the decorator body** when it should run per instance; use `addInitializer`.
- **Stack traces may show wrapper names**; set the wrapper’s `name` property if debuggability is critical.

### Annotated Complete Step by Step Code Examples

#### Example 1: Precise typing and `this` preservation (Stage 3)

```typescript
// example_best_typed_this.ts
// ✅ Stage 3 decorator with generic typing and `this` preservation.
function logged<This, Args extends unknown[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
): (this: This, ...args: Args) => Return {
  const name = String(context.name);
  return function (this: This, ...args: Args): Return {
    console.log(`[${name}] called with`, args);
    return originalMethod.call(this, ...args);   // preserve `this`
  };
}

class UserService {
  private users = ["alice", "bob"];

  @logged
  getUser(index: number): string {
    return this.users[index];   // relies on `this`
  }
}

const svc = new UserService();
console.log(svc.getUser(0));
// Expected:
// [getUser] called with [ 0 ]
// alice
```

**Expected Output:**
```
[getUser] called with [ 0 ]
alice
```

**Why this output occurs.** The generic signature preserves the method’s parameter and return types. The wrapper uses `originalMethod.call(this, ...args)` so that `this.users` resolves correctly. Without `this` preservation, calling `originalMethod(...args)` would lose the receiver and `this.users` would be `undefined`.

---

#### Example 2: `@bound` with `addInitializer` (Stage 3)

```typescript
// example_best_bound.ts
// ✅ Use addInitializer for per-instance binding.
function bound<This, Args extends unknown[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
): (this: This, ...args: Args) => Return {
  context.addInitializer(function (this: This) {
    (this as any)[context.name] = originalMethod.bind(this);
  });
  return originalMethod;
}

class Counter {
  private count = 0;

  @bound
  increment() {
    this.count++;
    console.log(this.count);
  }
}

const c = new Counter();
const inc = c.increment;
inc(); // Expected: 1
inc(); // Expected: 2
```

**Expected Output:**
```
1
2
```

**Why this output occurs.** `addInitializer` registers a callback that runs when the instance is initialized. The callback binds `increment` to the instance and assigns the bound function to `this.increment`. Extracting `inc` and calling it standalone still uses the correct `this`, so `count` increments from 0 to 1 to 2.

---

#### Example 3: Single-responsibility decorators and order awareness

```typescript
// example_best_single_responsibility.ts
// ✅ Each decorator does ONE thing.
function log<This, Args extends unknown[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  _context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
) {
  return function (this: This, ...args: Args): Return {
    console.log("log: before");
    const r = originalMethod.call(this, ...args);
    console.log("log: after");
    return r;
  };
}

function cache<This, Args extends unknown[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  _context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
) {
  const store = new Map<string, Return>();
  return function (this: This, ...args: Args): Return {
    const key = JSON.stringify(args);
    if (store.has(key)) return store.get(key)!;
    const r = originalMethod.call(this, ...args);
    store.set(key, r);
    return r;
  };
}

class MathService {
  @log
  @cache
  square(n: number): number {
    console.log("computing");
    return n * n;
  }
}

const m = new MathService();
console.log(m.square(4)); // Expected:
// log: before
// computing
// log: after
// 16
console.log(m.square(4)); // Expected:
// log: before
// log: after
// 16   (cached; "computing" is skipped)
```

**Expected Output:**
```
log: before
computing
log: after
16
log: before
log: after
16
```

**Why this output occurs.** `@cache` is applied first (closest to the method), then `@log` wraps the cached method. On the first call, `log` logs `before`, `cache` misses, the original computes and logs `computing`, `cache` stores the result, then `log` logs `after`. On the second call, `log` logs `before`, `cache` returns the stored result without calling the original, and `log` logs `after` — `computing` is skipped, demonstrating that each decorator has a single responsibility and that order matters.

---

### Real-World Cases

- **NestJS and TypeGraphQL** require legacy decorators; projects using them should keep `experimentalDecorators: true` and not mix Stage 3 decorators in the same codebase.
- **New projects** should omit `experimentalDecorators` and use Stage 3 decorators with precise generic signatures and `addInitializer` for binding.
- **Library authoring**: libraries targeting both ecosystems may publish separate entry points or use conditional checks, but should document which decorator system each entry point supports.
- **Performance-sensitive code**: `@memoize` and `@timed` decorators should be designed with order independence in mind, or their ordering requirements should be documented explicitly.

---

## References

- Decorators — TypeScript (Advanced) - https://mintlify.wiki/Microsoft/typescript/advanced/decorators
- Use Stage 3 Decorators with Decorator Context for Metaprogramming - https://raw.githubusercontent.com/pproenca/dot-skills/cf93c57cac89d6fc3e4194686000411567f5caf3/skills/.experimental/typescript-advanced-patterns/references/mod-stage-3-decorators.md
- TypeScript 5.x: Features That Change How You Write Code — The Gaulton Lab - https://www.gaultonlab.org/posts/typescript-5-new-features/
- TypeScript Decorators: The Complete Guide to Stage-3 and Beyond — DEV Community - https://dev.to/abhaysinghr1/typescript-decorators-the-complete-guide-to-stage-3-and-beyond-40ok
- Guided: Apply Decorators to Enhance Class Behavior in TypeScript — Pluralsight - https://www.pluralsight.com/labs/codeLabs/guided-apply-decorators-to-enhance-class-behavior-in-typescript
- TypeScript Decorators — W3Schools - https://www.w3schools.com/typescript/typescript_decorators.php
- How to Build Custom Decorators in TypeScript — OneUptime Blog - https://raw.githubusercontent.com/OneUptime/blog/master/posts/2026-01-30-typescript-custom-decorators.md
- How to Use a TypeScript Method Decorator and Retain Normal `this` Scope — Stack Overflow - https://stackoverflow.com/questions/56189631/how-to-use-a-typescript-method-decorator-and-retain-normal-this-scope