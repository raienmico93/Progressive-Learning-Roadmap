# TypeScript Decorators Fundamentals — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **decorator** is a special form of declaration that can be attached to a class declaration or to class members — methods, accessors, properties, and parameters — using the syntax `@expression`. The expression evaluates to a function that is invoked at runtime with information about the decorated declaration, enabling annotation or meta-programmatic modification of that declaration without altering its source text. TypeScript currently supports **two distinct decorator systems**: the legacy **experimental decorators** (aligned with an early Stage 2 proposal) and the **Stage 3 decorators** standardized by TC39 and shipped as the default in TypeScript 5.0+.

**Technical Definition.** A decorator is a function whose call signature depends on the decorator system in use. In the **legacy system**, decorators receive position-dependent arguments — for a class, the constructor; for a method, `(target, propertyKey, descriptor)`; for a property, `(target, propertyKey)`; for a parameter, `(target, propertyKey, parameterIndex)`. In the **Stage 3 system**, a decorator receives `(value, context)`, where `value` is the decorated element (method function, `undefined` for fields, or the class itself) and `context` is a typed metadata object (`ClassMethodDecoratorContext`, `ClassFieldDecoratorContext`, `ClassDecoratorContext`, etc.) describing the element’s `kind`, `name`, `static`, `private`, and providing `addInitializer`. The two systems are **mutually incompatible**; a project must choose one, controlled by the `experimentalDecorators` compiler flag.

**Beginner-Friendly Explanation.** A decorator is like a **sticker with instructions** that you place above a class, a method, or a property using `@`. When the code runs, the sticker’s function is called with information about whatever it is stuck to, and it can **wrap, replace, or annotate** that thing. TypeScript has an **old style** (experimental decorators, still used by Angular, NestJS, TypeORM, and many existing libraries) and a **new standard style** (Stage 3 decorators, the default since TypeScript 5.0). They look similar (`@name`) but behave differently under the hood, and you cannot mix them in the same project.

---

### Key Characteristics

| Characteristic | Experimental (Legacy) | Stage 3 (Standard) |
|---|---|---|
| **Compiler flag** | `experimentalDecorators: true` required | No flag required in TS 5.0+; enabled by default when `target` is `ES2022`+ or `ESNext` |
| **Proposal basis** | Early Stage 2 draft | TC39 Stage 3 (feature-complete) |
| **Decorator arguments** | Position-dependent (`target, key, descriptor`) | Uniform `(value, context)` |
| **Context object** | Not available (relies on `reflect-metadata`) | First-class, typed context per kind |
| **Return values** | Mutate descriptor; may return descriptor | Return a replacement value (method/class) or initializer |
| **Execution order** | Evaluation bottom-to-top; execution top-to-bottom | Well-defined; returned functions execute outermost-to-innermost |
| **Metadata reflection** | Supported via `emitDecoratorMetadata` + `reflect-metadata` | Not supported; must pass types explicitly |
| **Parameter decorators** | Supported | **Not part of the Stage 3 proposal** |
| **Runtime cost** | Metadata reflection adds overhead | No reflection overhead; type-erased |
| **Ecosystem compatibility** | Required by Angular, NestJS, TypeORM, class-validator | Preferred for new projects without legacy dependencies |

---

### Prerequisites

- **Classes and class members** in JavaScript/TypeScript: methods, accessors, fields, static members, private `#` fields.
- **Functions as first-class values** and higher-order functions.
- **Basic understanding of `this` binding** and property descriptors.
- **A TypeScript 5.0+ environment** (`tsc` 5.0 or later) or the TypeScript Playground.
- **Familiarity with `tsconfig.json`** compiler options, especially `target`, `experimentalDecorators`, and `emitDecoratorMetadata`.

---

### Related Programming Areas

- **Aspect-oriented programming (AOP)**: decorators provide native syntax for cross-cutting concerns (logging, caching, validation).
- **Dependency injection**: frameworks like Angular and NestJS use legacy decorators plus `reflect-metadata` for DI containers.
- **Meta-programming**: Stage 3 decorators enable structured runtime class customization with typed contexts.
- **Type erasure**: decorators are runtime constructs, but their **type signatures** are erased; only the decorator function itself survives compilation.
- **TC39 standardization process**: decorators progressed from Stage 2 (legacy) to Stage 3 (standard), a distinction central to TypeScript’s dual support.

---

### Core Concepts / Features

The following five sections cover the requested concepts. Because experimental and Stage 3 decorators are distinct systems, they are treated separately after the general concepts and syntax.

---

## 1. Decorators

### Definitions

**Core Definition.** **Decorators** are a language feature that allows **annotations and meta-programmatic modification** of classes and class members through a declarative `@` syntax, without requiring changes to the decorated code’s body.

**Technical Definition.** In TypeScript, decorators are **functions invoked at class definition time** (when the class is declared, not when instances are created) with arguments that describe the decorated element. Legacy decorators receive implementation-specific positional arguments and may mutate property descriptors or return replacement descriptors. Stage 3 decorators receive `(value, context)` and may return a replacement value or use `context.addInitializer` to inject initialization logic. Decorators compose: multiple decorators on one declaration are evaluated in a defined order, and their results are applied as function composition.

**Beginner-Friendly Explanation.** A decorator is a **reusable behavior you attach with `@`** to a class, method, or property. Instead of copying logging code into every method, you write a `@log` decorator once and put `@log` above each method. When the class is defined, the decorator runs and wraps each method with logging. The original method body stays clean.

### Purposes

- **To separate cross-cutting concerns** (logging, caching, validation, authorization) from core business logic, improving readability.
- **To add metadata or behavior to classes and members** without inheritance or manual wrapping.
- **To enable framework-level patterns** such as dependency injection, routing, and ORM entity mapping through declarative annotations.
- **To reuse behavioral modifications** across many declarations with a single, composable expression.
- **To provide a standardized metaprogramming surface** (Stage 3) that integrates with the type system and tooling.

### Syntax Rules and Structure

**General syntax.**

```
Decorator:
  @ DecoratorExpression

DecoratorExpression:
  Identifier
  Identifier . Identifier
  DecoratorFactoryCall
  DecoratorFactoryCall ( . Identifier )?
```

**Component breakdown.**

```
@loggedMethod
```

- `@` — the **decorator sigil**; always precedes the expression.
- `loggedMethod` — an expression evaluating to a decorator function.

```
@logged("LOG:")
```

- `@` — sigil.
- `logged("LOG:")` — a **decorator factory call**; `logged` returns the actual decorator function.
- The `()` invokes the factory at decoration time; the returned function receives the decorated element.

**Syntax rules.**

- A decorator must be placed **immediately before** the declaration it decorates, on the same or preceding line.
- Decorator expressions are restricted to **identifiers, property access with `.` (not `[]`), and calls**; arbitrary expressions are not permitted in Stage 3.
- Multiple decorators may be stacked on separate lines or on one line: `@f @g method()`.
- Decorators may appear on **classes, methods, accessors (get/set), properties, and — in legacy only — parameters**.
- Decorators **cannot** be applied to standalone functions, local variables, or interface members (interfaces have no runtime representation).

**Constraints and limitations.**

- **Experimental vs. Stage 3 incompatibility**: the two systems cannot coexist in one project; the `experimentalDecorators` flag selects legacy mode.
- **Parameter decorators are legacy-only**; Stage 3 has no parameter decorator form.
- **No runtime type information**: decorators cannot inspect generic type arguments or interface shapes at runtime because types are erased.
- **Class decorators run after member decorators** in Stage 3; legacy class decorators run after method/property decorators as well, but the argument shapes differ.

### Annotated Complete Step by Step Code Examples

#### Example 1: A legacy method decorator that logs calls

```typescript
// example_legacy_log.ts
// Step 1: Write a legacy method decorator.
// Legacy method decorators receive (target, propertyKey, descriptor).
function log(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
): PropertyDescriptor {
  const original = descriptor.value;            // the original method
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    const result = original.apply(this, args);  // preserve `this`
    console.log(`Result:`, result);
    return result;
  };
  return descriptor;                            // return the modified descriptor
}

// Step 2: Apply the decorator to a method.
class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

// Step 3: Use the class.
const calc = new Calculator();
calc.add(2, 3);
// Expected:
// Calling add with [ 2, 3 ]
// Result: 5
```

**Setup.** Enable legacy decorators in `tsconfig.json`:
```json
{ "compilerOptions": { "experimentalDecorators": true } }
```
Compile with `npx tsc` and run the output with Node.js.

**Expected Output:**
```
Calling add with [ 2, 3 ]
Result: 5
```

**Why this output occurs.** The `@log` decorator mutates the method’s `descriptor.value`, replacing the original `add` with a wrapper that logs before and after calling the original. Because `descriptor` is returned, the modified descriptor is installed on the prototype. At runtime, calling `calc.add(2, 3)` invokes the wrapper, which logs the arguments, calls the original, logs the result, and returns `5`.

---

#### Example 2: A Stage 3 method decorator with a typed context

```typescript
// example_stage3_log.ts
// Step 1: Write a Stage 3 method decorator.
// It receives (value, context), where value is the method function.
function loggedMethod(
  originalMethod: any,
  context: ClassMethodDecoratorContext
) {
  const methodName = String(context.name);   // metadata from context

  function replacementMethod(this: any, ...args: any[]) {
    console.log(`LOG: Entering '${methodName}'.`);
    const result = originalMethod.call(this, ...args);
    console.log(`LOG: Exiting '${methodName}'.`);
    return result;
  }

  return replacementMethod;                   // return the replacement
}

// Step 2: Apply it to a method (Stage 3 is default in TS 5.0+).
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }

  @loggedMethod
  greet() {
    console.log(`Hello, my name is ${this.name}.`);
  }
}

// Step 3: Use the class.
const p = new Person("Ray");
p.greet();
// Expected:
// LOG: Entering 'greet'.
// Hello, my name is Ray.
// LOG: Exiting 'greet'.
```

**Setup.** No special flag is needed in TypeScript 5.0+ when `target` is `ES2022` or `ESNext`. Compile and run.

**Expected Output:**
```
LOG: Entering 'greet'.
Hello, my name is Ray.
LOG: Exiting 'greet'.
```

**Why this output occurs.** The decorator returns `replacementMethod`, which becomes the new `greet`. The `context` parameter provides `context.name`, which is `"greet"`. Unlike the legacy version, the decorator **returns** a replacement rather than mutating a descriptor. The context object is typed, enabling `context.name` without `any`.

---

### Real-World Cases

- **NestJS and Angular** use legacy decorators (`@Component`, `@Injectable`, `@Controller`) with `reflect-metadata` for dependency injection and metadata-driven configuration.
- **TypeORM and MikroORM** define database entities and columns with legacy decorators (`@Entity`, `@Column`, `@PrimaryKey`); Stage 3 support is being added with explicit type arguments because metadata reflection is unavailable.
- **class-validator** applies legacy property decorators (`@IsEmail`, `@MinLength`) to declare validation rules that are checked at runtime.
- **Stage 3 custom decorators** are used for logging, memoization, and binding in modern codebases that do not depend on legacy frameworks.

---

## 2. Decorator Syntax

### Definitions

**Core Definition.** **Decorator syntax** is the set of grammatical rules governing how decorator expressions are written, where they may appear, and how they compose. The syntax is superficially identical between legacy and Stage 3 systems (`@expression`), but the **semantics and argument shapes** differ.

**Technical Definition.** Syntactically, a decorator is an `@` followed by a **decorator expression**, which is restricted to identifiers, dotted names, and calls. Decorators are attached to declarations and are evaluated in a defined order. In both systems, decorator **expressions** are evaluated **top-to-bottom**, but the resulting **functions** are applied **bottom-to-top** (function composition). Stage 3 imposes additional syntactic restrictions: decorator expressions cannot use computed property access (`[]`), and parameter decorators are not permitted.

**Beginner-Friendly Explanation.** Decorator syntax is simply **`@` plus a name or a function call**, placed directly above the thing you want to decorate. You can stack several decorators. The order rules mean the **closest decorator to the declaration runs first**, even though the expressions are written top to bottom — like peeling an onion from the inside out.

### Purposes

- **To provide a uniform, declarative notation** for attaching behavior across many declarations.
- **To support composition** of multiple decorators without nested manual wrappers.
- **To make decorator intent visually obvious** at the declaration site.
- **To enable factories** that customize decorator behavior through arguments, keeping the `@` expression concise.

### Syntax Rules and Structure

**General syntax (stacking and factories).**

```
@first()
@second("option")
methodName() { ... }
```

**Component breakdown.**

- `@first()` — a **decorator factory call**; `first` is invoked and returns the actual decorator.
- `@second("option")` — a factory with an argument.
- `methodName() { ... }` — the decorated method.
- Evaluation order: `first()` factory evaluated, then `second("option")` factory evaluated, then the returned decorators are applied **bottom-to-top**: `second`’s decorator wraps the original method first, then `first`’s decorator wraps the result.

**Syntax rules.**

- Decorators precede the declaration; they may be on the same line as the declaration or on separate lines.
- A decorator expression must be an **identifier**, a **dotted name**, or a **call**; computed access (`@obj["dec"]`) is not allowed in Stage 3.
- **Factories** are ordinary functions that return decorator functions; the factory is called at decoration time.
- **Multiple decorators** on one declaration are allowed; their evaluation and execution orders are well-defined.
- Decorators are **not hoisted**; they execute when the class is defined, in source order.

**Constraints and limitations.**

- **Stage 3 syntax does not support parameter decorators**; legacy syntax does.
- **Legacy decorator expressions** historically allowed a narrower set than Stage 3 in some cases, but both restrict arbitrary expressions.
- Decorators **cannot** be applied to variable declarations, standalone functions, or statements.
- The `@` sigil is **reserved** for decorators; it is not an operator in ordinary expressions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Stacked decorators and evaluation order

```typescript
// example_stacking.ts
// Step 1: Define two decorator factories.
function first() {
  console.log("first(): factory evaluated");
  return function (target: any, key: string, descriptor: PropertyDescriptor) {
    console.log("first(): decorator called");
  };
}

function second() {
  console.log("second(): factory evaluated");
  return function (target: any, key: string, descriptor: PropertyDescriptor) {
    console.log("second(): decorator called");
  };
}

// Step 2: Stack them on a method.
class Example {
  @first()
  @second()
  method() {}
}

// Expected:
// first(): factory evaluated
// second(): factory evaluated
// second(): decorator called
// first(): decorator called
```

**Expected Output:**
```
first(): factory evaluated
second(): factory evaluated
second(): decorator called
first(): decorator called
```

**Why this output occurs.** The **factories** are evaluated top-to-bottom (`first` then `second`). The **returned decorators** are applied bottom-to-top: `second`’s decorator is applied to the original method first, then `first`’s decorator is applied to the result. This is function composition: `first(second(method))`.

---

#### Example 2: Stage 3 decorator factory with typed context

```typescript
// example_factory_stage3.ts
// Step 1: A Stage 3 decorator factory that customizes the log prefix.
function logged(prefix: string) {
  return function (
    originalMethod: any,
    context: ClassMethodDecoratorContext
  ) {
    const name = String(context.name);
    return function (this: any, ...args: any[]) {
      console.log(`${prefix} Entering '${name}'.`);
      return originalMethod.call(this, ...args);
    };
  };
}

// Step 2: Apply the factory with an argument.
class Service {
  @logged("DEBUG:")
  run() {
    console.log("running");
  }
}

new Service().run();
// Expected:
// DEBUG: Entering 'run'.
// running
```

**Expected Output:**
```
DEBUG: Entering 'run'.
running
```

**Why this output occurs.** `logged("DEBUG:")` is a **factory**; it captures `prefix` and returns the actual decorator. The decorator receives `context` with `name = "run"` and returns a replacement method that logs the custom prefix before calling the original. This pattern is the Stage 3 equivalent of a legacy decorator factory.

---

### Real-World Cases

- **`@Component({ selector: "app-root" })`** in Angular: a legacy decorator factory that configures component metadata.
- **`@Column({ type: "varchar", length: 255 })`** in TypeORM: a legacy property decorator factory that configures database column options.
- **`@memoize()`** as a Stage 3 method decorator factory that caches results based on arguments, returning a wrapped method.

---

## 3. The `@` Sigil

### Definitions

**Core Definition.** The **`@` sigil** is the syntactic marker that introduces a decorator expression. It is the character that visually and grammatically distinguishes a decorator from ordinary code.

**Technical Definition.** In the grammar, `@` is a **punctuator** that precedes a `DecoratorExpression`. It is not an operator and has no standalone meaning; it is only valid in decorator position. The parser recognizes `@` followed by an expression as a decorator and attaches the resulting decorator function to the following declaration.

**Beginner-Friendly Explanation.** The `@` symbol is simply the **“decorator starts here” marker**. Just as `#` starts a private field or `$` is often used in variable names, `@` is the special character that tells TypeScript “the following expression is a decorator.”

### Purposes

- **To unambiguously distinguish decorators** from ordinary identifiers, function calls, and annotations.
- **To provide a concise, visually scannable notation** that highlights decorated declarations.
- **To anchor the grammar** so the parser knows when a decorator expression begins and ends.

### Syntax Rules and Structure

**General syntax.**

```
@ DecoratorExpression
```

**Component breakdown.**

```
@sealed
```

- `@` — the **sigil**; required.
- `sealed` — the decorator expression (an identifier referring to a function).

```
@decorators.log
```

- `@` — sigil.
- `decorators.log` — dotted access; allowed in Stage 3.

```
@decoratorFactory(arg)
```

- `@` — sigil.
- `decoratorFactory(arg)` — a call expression; the factory is invoked.

**Syntax rules.**

- `@` must be **immediately followed** by a valid decorator expression; whitespace between `@` and the expression is allowed but conventionally omitted.
- `@` cannot be used as a standalone token in expressions; it is only valid in decorator position.
- In **Stage 3**, decorator expressions may use `.` but **not** computed `[]` access.
- The sigil is **not** used in the emitted JavaScript; it is erased during compilation.

**Constraints and limitations.**

- `@` does **not** appear at runtime; it is purely syntactic sugar.
- The sigil is **reserved** and cannot be repurposed for other language features.
- Decorator expressions are evaluated **at decoration time**, not at call time; the `@` marks a position whose expression executes when the class is defined.

### Annotated Complete Step by Step Code Examples

#### Example 1: `@` with a simple identifier

```typescript
// example_sigil_identifier.ts
// Step 1: Define a class decorator function.
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

// Step 2: Apply it with @.
@sealed
class BugReport {
  type = "report";
  title: string;
  constructor(title: string) {
    this.title = title;
  }
}

// Step 3: Verify sealing.
console.log(Object.isSealed(BugReport)); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** `@sealed` is the sigil plus the identifier `sealed`. The `sealed` function is invoked with the `BugReport` constructor, and it seals both the constructor and its prototype. `Object.isSealed(BugReport)` returns `true` because the decorator ran when the class was defined.

---

#### Example 2: `@` with a dotted name and a factory call

```typescript
// example_sigil_dotted.ts
// Step 1: Define a namespace-like object holding decorators.
const decorators = {
  log(target: any, key: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    descriptor.value = function (...args: any[]) {
      console.log(`Calling ${key}`);
      return original.apply(this, args);
    };
    return descriptor;
  },
};

// Step 2: Apply with a dotted expression.
class Task {
  @decorators.log
  execute() {
    console.log("executing");
  }
}

new Task().execute();
// Expected:
// Calling execute
// executing
```

**Expected Output:**
```
Calling execute
executing
```

**Why this output occurs.** `@decorators.log` is the sigil followed by a **dotted name**. The decorator is resolved from the `decorators` object and applied to `execute`. The wrapper logs the method name before calling the original.

---

### Real-World Cases

- **Angular templates and components** are marked with `@Component`, `@Directive`, and `@Input` — all using the `@` sigil to distinguish metadata decorators.
- **NestJS controllers** use `@Controller("users")` and `@Get(":id")`; the sigil makes routing metadata visually distinct from method bodies.
- **Stage 3 custom decorators** such as `@bound`, `@memo`, and `@measure` use the same sigil, ensuring consistency across the ecosystem.

---

## 4. Experimental Decorators

### Definitions

**Core Definition.** **Experimental decorators** (also called **legacy decorators**) are TypeScript’s original decorator implementation, based on an **early Stage 2 TC39 proposal**. They are enabled by the `experimentalDecorators` compiler flag and remain widely used by established frameworks and libraries.

**Technical Definition.** Legacy decorators use **position-dependent call signatures**: class decorators receive the constructor; method and accessor decorators receive `(target, propertyKey, descriptor)`; property decorators receive `(target, propertyKey)`; parameter decorators receive `(target, propertyKey, parameterIndex)`. They may mutate the descriptor and return it to replace the member. When combined with `emitDecoratorMetadata: true` and the `reflect-metadata` package, the compiler emits runtime type metadata for decorated declarations, enabling dependency injection and serialization frameworks.

**Beginner-Friendly Explanation.** Experimental decorators are the **old decorator system** that TypeScript has had for years. They are called “experimental” because they were based on a draft standard that later changed. They still work, and many popular libraries depend on them. They are more **hands-on**: the decorator receives the target, the property name, and the property descriptor, and it can modify the descriptor directly. They also support **parameter decorators**, which the new standard does not.

### Purposes

- **To support existing frameworks and libraries** (Angular, NestJS, TypeORM, class-validator) that are built on the legacy decorator model.
- **To enable metadata reflection** through `emitDecoratorMetadata` and `reflect-metadata`, which many dependency-injection systems require.
- **To provide parameter decorators** for constructor and method parameter injection.
- **To maintain backward compatibility** in codebases that predate TypeScript 5.0.

### Syntax Rules and Structure

**General syntax (legacy method decorator).**

```
function DecoratorName(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
): PropertyDescriptor | void
```

**Component breakdown.**

- `target` — the prototype for instance members, or the constructor for static members.
- `propertyKey` — the name of the decorated member.
- `descriptor` — the property descriptor; may be mutated and returned.
- Returning the descriptor installs the modified member; returning `void` leaves it unchanged.

**Syntax rules.**

- Enable with `experimentalDecorators: true` in `tsconfig.json` or `--experimentalDecorators` on the command line.
- `emitDecoratorMetadata: true` (optional) emits `design:type`, `design:paramtypes`, and `design:returntype` metadata; requires `reflect-metadata` at runtime.
- Parameter decorators use the signature `(target, propertyKey, parameterIndex)`.
- Legacy decorators **mutate** descriptors; they do not return replacement functions in the Stage 3 sense (though returning a descriptor is supported).

**Constraints and limitations.**

- **Incompatible with Stage 3 decorators**; the flag selects one system or the other.
- **No typed context object**; decorators rely on positional arguments and optional metadata reflection.
- **Metadata reflection requires an extra dependency** (`reflect-metadata`) and adds runtime overhead.
- **Parameter decorators are legacy-only**; they have no Stage 3 equivalent.
- **Execution order** can be less intuitive: evaluation is bottom-to-top, execution top-to-bottom.

### Annotated Complete Step by Step Code Examples

#### Example 1: Legacy parameter decorator

```typescript
// example_legacy_parameter.ts
// Step 1: Define a legacy parameter decorator.
function required(target: any, propertyKey: string, parameterIndex: number) {
  console.log(
    `Parameter ${parameterIndex} of ${propertyKey} is required`
  );
}

// Step 2: Apply it to a constructor parameter.
class UserService {
  constructor(@required private name: string) {}
}

// Expected: Parameter 0 of undefined is required
```

**Expected Output:**
```
Parameter 0 of undefined is required
```

**Why this output occurs.** The parameter decorator is invoked at class definition time with `target` (the class constructor), `propertyKey` (`undefined` for constructor parameters), and `parameterIndex` (`0`). This form exists **only in legacy decorators**; Stage 3 has no parameter decorator syntax.

---

#### Example 2: Legacy metadata reflection for dependency injection

```typescript
// example_legacy_metadata.ts
// Step 1: Import reflect-metadata (required for metadata reflection).
import "reflect-metadata";

// Step 2: Define a class decorator that reads emitted metadata.
function logType(target: any, key: string) {
  const type = Reflect.getMetadata("design:type", target, key);
  console.log(`${key} type: ${type.name}`);
}

// Step 3: Apply it to a property.
class Demo {
  @logType
  public attr: string = "test";
}

// Expected: attr type: String
```

**Setup.** Enable both flags:
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
attr type: String
```

**Why this output occurs.** With `emitDecoratorMetadata`, the compiler emits `design:type` metadata for the property. The `logType` decorator reads that metadata via `Reflect.getMetadata` and logs the type name. This mechanism is the basis of legacy dependency-injection frameworks.

---

### Real-World Cases

- **Angular** uses legacy decorators (`@Component`, `@Injectable`, `@Input`, `@Output`) plus metadata reflection for its dependency-injection system.
- **NestJS** builds its entire module/controller/provider architecture on legacy decorators and `reflect-metadata`.
- **TypeORM** entities and columns are defined with legacy decorators; Stage 3 support requires explicit type arguments because metadata reflection is unavailable.
- **class-validator** uses legacy property decorators to register validation rules that are enforced at runtime.

---

## 5. Stage 3 Decorators

### Definitions

**Core Definition.** **Stage 3 decorators** are the standardized JavaScript decorator system defined by the **TC39 Stage 3 proposal**. They are the **default decorator implementation in TypeScript 5.0+** and are enabled automatically when the compilation target is `ES2022` or later (or `ESNext`), without the `experimentalDecorators` flag.

**Technical Definition.** Stage 3 decorators use a **uniform `(value, context)` signature**. The `value` is the decorated element — the method function for methods, `undefined` for fields, the class constructor for classes, or `{ get, set }` for auto-accessors. The `context` is a typed object (`ClassMethodDecoratorContext`, `ClassFieldDecoratorContext`, `ClassDecoratorContext`, etc.) that describes the element’s `kind`, `name`, `static`, `private`, and provides `addInitializer`. Decorators **return a replacement value** (a new method, a new class, or an initializer function) rather than mutating descriptors. A new `accessor` keyword enables **auto-accessors** that can be decorated. **Parameter decorators are not part of the Stage 3 proposal**.

**Beginner-Friendly Explanation.** Stage 3 decorators are the **new, official decorator system**. They are the default in TypeScript 5.0 and later — you do not need a special flag. Instead of receiving a target and a descriptor, every decorator gets **the thing it decorates** and a **context object** that tells it what kind of thing it is (method, field, class), its name, and whether it is static or private. You **return a replacement** instead of mutating an object. This makes decorators more predictable, more type-safe, and easier to reason about. The trade-off is that parameter decorators and metadata reflection are not supported.

### Purposes

- **To provide a standardized, future-proof decorator system** aligned with ECMAScript, ensuring long-term compatibility.
- **To improve type safety** through typed context objects (`ClassMethodDecoratorContext`, etc.) that describe the decorated element.
- **To make decorator behavior more predictable** with well-defined return-value semantics and execution order.
- **To eliminate the need for `reflect-metadata`** in new code, reducing dependencies and runtime overhead.
- **To support auto-accessors** (`accessor`), enabling decorators on getter/setter pairs with a single declaration.

### Syntax Rules and Structure

**General syntax (Stage 3 method decorator).**

```
function DecoratorName(
  value: MethodFunction,
  context: ClassMethodDecoratorContext
): ReplacementMethod | void
```

**Component breakdown.**

- `value` — the original method function.
- `context: ClassMethodDecoratorContext` — typed metadata: `kind: "method"`, `name`, `static`, `private`, `addInitializer`.
- Return a **replacement function** to replace the method, or `void` to leave it unchanged.
- Use `context.addInitializer(fn)` to inject initialization logic into the constructor.

**Syntax rules.**

- No `experimentalDecorators` flag is needed; Stage 3 is the default in TS 5.0+ when `target` is `ES2022`+.
- Decorators **return** replacements; they do not mutate descriptors.
- **Parameter decorators are unavailable** in Stage 3.
- The `accessor` keyword declares an **auto-accessor** that can be decorated:
  ```typescript
  class C {
    @dec accessor x = 1;
  }
  ```
- **Metadata reflection is not supported**; type information must be provided explicitly.

**Constraints and limitations.**

- **No parameter decorators**; dependency-injection patterns that rely on constructor parameter decorators cannot use Stage 3 directly.
- **No `emitDecoratorMetadata` equivalent**; frameworks requiring runtime type reflection must stay on legacy decorators.
- **Incompatible with `experimentalDecorators: true`**; a project must choose one system.
- **Execution order** is well-defined but may differ from legacy expectations; returned functions execute outermost-to-innermost.

### Annotated Complete Step by Step Code Examples

#### Example 1: Stage 3 `@bound` decorator with `addInitializer`

```typescript
// example_stage3_bound.ts
// Step 1: Define a Stage 3 method decorator that binds `this`.
function bound(
  originalMethod: any,
  context: ClassMethodDecoratorContext
) {
  const methodName = context.name;
  if (context.private) {
    throw new Error(`'bound' cannot decorate private properties.`);
  }
  context.addInitializer(function (this: any) {
    this[methodName] = this[methodName].bind(this);
  });
  // Return nothing: the method is left unchanged; addInitializer handles binding.
}

// Step 2: Apply it.
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }

  @bound
  greet() {
    console.log(`Hello, my name is ${this.name}.`);
  }
}

// Step 3: Extract the method and call it standalone.
const p = new Person("Ray");
const greet = p.greet;
greet(); // Expected: Hello, my name is Ray.
```

**Expected Output:**
```
Hello, my name is Ray.
```

**Why this output occurs.** `context.addInitializer` registers a callback that runs at the beginning of the constructor (or class initialization for statics). The callback binds `this[methodName]` to the instance, so when `greet` is called as a standalone function, `this` still refers to `p`. The decorator returns nothing, so the original method remains, but the initializer ensures correct binding.

---

#### Example 2: Stage 3 auto-accessor decorator

```typescript
// example_stage3_accessor.ts
// Step 1: Define a decorator for auto-accessors.
function logged(
  value: { get: () => any; set: (v: any) => void },
  context: ClassAccessorDecoratorContext
) {
  return {
    get() {
      const v = value.get.call(this);
      console.log(`Getting ${String(context.name)}: ${v}`);
      return v;
    },
    set(newValue: any) {
      console.log(`Setting ${String(context.name)}: ${newValue}`);
      value.set.call(this, newValue);
    },
  };
}

// Step 2: Use it on an auto-accessor.
class Temperature {
  @logged
  accessor celsius = 20;
}

const t = new Temperature();
t.celsius = 25;          // Expected: Setting celsius: 25
console.log(t.celsius);  // Expected: Getting celsius: 25
                         //           25
```

**Expected Output:**
```
Setting celsius: 25
Getting celsius: 25
25
```

**Why this output occurs.** The `accessor` keyword creates an auto-accessor backed by a private field with a getter/setter pair. The decorator receives `{ get, set }` and returns a replacement `{ get, set }` that logs before delegating to the original accessor. This is a Stage 3-only feature; legacy decorators cannot decorate auto-accessors because the `accessor` keyword does not exist in legacy mode.

---

### Real-World Cases

- **Modern logging and memoization**: Stage 3 decorators with typed context are used for cross-cutting concerns in codebases without legacy framework dependencies.
- **Binding**: The `@bound` pattern using `addInitializer` replaces manual `bind` calls in constructors.
- **Auto-accessor validation**: Libraries can decorate `accessor` fields to intercept reads and writes, enabling reactive properties or validation.
- **Migration targets**: New projects starting on TypeScript 5.0+ without Angular/NestJS/TypeORM dependencies should use Stage 3 decorators for future compatibility.

---

## 6. Decorator Best Practices

### Definitions

**Core Definition.** **Decorator best practices** are guidelines for choosing between the legacy and Stage 3 systems, structuring decorator functions for type safety and maintainability, and avoiding patterns that lead to fragile or unmaintainable metaprogramming.

**Technical Definition.** Best practices encompass **system selection** (legacy vs. Stage 3 based on ecosystem dependencies), **signature correctness** (using typed context objects or correct positional arguments), **composability** (designing decorators that stack predictably), **single responsibility** (one concern per decorator), and **type-safety preservation** (avoiding `any` where `unknown` or typed contexts suffice). Migration guidance emphasizes that mixing systems is unsupported and that legacy dependencies (Angular, NestJS, TypeORM) constrain the choice.

**Beginner-Friendly Explanation.** Best practices mean **choosing the right decorator system for your project**, writing decorators that are **clear and safe**, and not overusing `any` just because decorators often involve dynamic behavior. If you depend on Angular or NestJS, you must use legacy decorators. If you are starting fresh, use Stage 3 decorators. Keep each decorator focused on one job, and use the typed context object in Stage 3 instead of casting everything to `any`.

### Purposes

- **To prevent incompatible-system errors** by selecting one decorator system and applying it consistently.
- **To maximize type safety** by using Stage 3’s typed contexts and preferring `unknown` over `any` where types are genuinely unknown.
- **To keep decorators composable and predictable** by following well-defined evaluation and execution order.
- **To facilitate future migration** by preferring Stage 3 for new code unless legacy ecosystem constraints require otherwise.
- **To avoid hidden coupling** between decorators and the declarations they decorate, preserving readability.

### Syntax Rules and Structure

Best practices are not a single syntax; they are **conventions** that interact with both decorator systems.

**System selection checklist.**

```
Does the project depend on Angular, NestJS, TypeORM, or class-validator?
  ├── Yes → Use experimentalDecorators: true (legacy system).
  │         Do NOT enable Stage 3; the systems are incompatible.
  └── No  → Use Stage 3 decorators (TS 5.0+ default).
            Do NOT set experimentalDecorators: true.
```

**Stage 3 signature best practice.**

```
function goodDecorator(
  value: any,                            // or a precise function type
  context: ClassMethodDecoratorContext   // typed context
) {
  // use context.name, context.static, context.private, context.addInitializer
}
```

- Use the **typed context** instead of separate positional arguments.
- Return a **replacement** rather than mutating; do not rely on hidden side effects.
- Use `unknown` and narrow with runtime checks rather than `any` for external data.

**Legacy signature best practice.**

```
function legacyDecorator(
  target: any,
  key: string,
  descriptor: PropertyDescriptor
): PropertyDescriptor {
  // mutate and return the descriptor deliberately
}
```

- Return the descriptor to make the mutation explicit.
- Use `emitDecoratorMetadata` only when required; document the `reflect-metadata` dependency.

**Constraints and limitations.**

- **Never enable both systems**; the flag selects one and the other is unavailable.
- **Stage 3 lacks parameter decorators**; if parameter injection is required, legacy is the only option.
- **Stage 3 lacks metadata reflection**; do not attempt to use `reflect-metadata` patterns with Stage 3.
- **Decorator factories should be pure** where possible; avoid relying on external mutable state across decorations.

### Annotated Complete Step by Step Code Examples

#### Example 1: Correct system selection — legacy for a DI-style framework

```typescript
// example_best_legacy.ts
// Step 1: Configure legacy decorators because parameter injection is required.
// tsconfig.json:
// {
//   "compilerOptions": {
//     "experimentalDecorators": true,
//     "emitDecoratorMetadata": true
//   }
// }

// Step 2: Use a legacy parameter decorator (Stage 3 cannot express this).
function inject(token: string) {
  return function (
    target: any,
    propertyKey: string | undefined,
    parameterIndex: number
  ) {
    console.log(`Injecting ${token} at parameter ${parameterIndex}`);
  };
}

class Service {
  constructor(@inject("db") private db: string) {}
}

// Expected: Injecting db at parameter 0
```

**Expected Output:**
```
Injecting db at parameter 0
```

**Why this output occurs.** The project correctly chooses legacy decorators because **parameter decorators are essential** to the injection pattern and are not available in Stage 3. Enabling both systems would be invalid; the checklist selects legacy based on the dependency.

---

#### Example 2: Stage 3 with typed context and `unknown`

```typescript
// example_best_stage3.ts
// Step 1: Use Stage 3 with a typed context and unknown for external data.
function validate(
  originalMethod: any,
  context: ClassMethodDecoratorContext
) {
  const name = String(context.name);
  return function (this: any, ...args: unknown[]) {
    if (args.some((a) => a === undefined || a === null)) {
      throw new Error(`${name} received nullish arguments`);
    }
    return originalMethod.apply(this, args);
  };
}

class Formatter {
  @validate
  format(value: string): string {
    return value.toUpperCase();
  }
}

const f = new Formatter();
console.log(f.format("hello")); // Expected: HELLO
try {
  f.format(null as any);
} catch (e) {
  console.log((e as Error).message);
  // Expected: format received nullish arguments
}
```

**Expected Output:**
```
HELLO
format received nullish arguments
```

**Why this output occurs.** The Stage 3 decorator uses `context.name` for metadata and declares `args: unknown[]`, forcing explicit handling rather than silently accepting `any`. The runtime check validates arguments, demonstrating the recommended combination of typed context and `unknown`-based narrowing.

---

#### Example 3: Single-responsibility decorators

```typescript
// example_best_single.ts
// ✅ Each decorator does ONE thing.
function log(originalMethod: any, context: ClassMethodDecoratorContext) {
  return function (this: any, ...args: any[]) {
    console.log(`Calling ${String(context.name)}`);
    return originalMethod.apply(this, args);
  };
}

function memoize(originalMethod: any, _context: ClassMethodDecoratorContext) {
  const cache = new Map<string, any>();
  return function (this: any, ...args: any[]) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

class MathService {
  @log
  @memoize
  square(n: number): number {
    return n * n;
  }
}

const m = new MathService();
console.log(m.square(4)); // Expected: 16
console.log(m.square(4)); // Expected: 16 (cached; no second log)
```

**Expected Output:**
```
Calling square
16
16
```

**Why this output occurs.** `@memoize` wraps the original method with caching, and `@log` wraps the memoized method with logging. Because decorators apply **bottom-to-top**, `memoize` is applied first, then `log`. The first call logs and computes; the second call returns the cached result and does **not** log again, demonstrating that each decorator has a single, composable responsibility.

---

### Real-World Cases

- **Framework migration**: Projects moving from Angular/NestJS to modern stacks often keep legacy decorators for existing code while writing new modules with Stage 3, but **not in the same compilation unit** because the systems are incompatible.
- **Library design**: Libraries targeting both legacy and Stage 3 ecosystems may publish separate entry points (e.g., `@mikro-orm/decorators/legacy` vs. `@mikro-orm/decorators/es`) rather than trying to unify them.
- **Cross-cutting concerns**: Single-responsibility decorators (`@log`, `@cache`, `@validate`) are preferred over monolithic decorators that mix logging, caching, and validation, because they compose predictably.

---

## References

- TypeScript Handbook — Decorators - https://www.typescriptlang.org/docs/handbook/decorators.html
- TypeScript 5.0 Release Notes — Decorators - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html
- TypeScript Documentation — Decorators (advanced overview, Stage 3 vs. legacy) - https://mintlify.wiki/Microsoft/typescript/advanced/decorators
- TC39 Decorators Proposal (Stage 3) - https://github.com/tc39/proposal-decorators
- MikroORM — Using Decorators (Legacy vs. ES Spec) - https://mikro-orm.io/docs/7.1/using-decorators
- Refine — TypeScript Decorators in Brief (Best Practices) - https://refine.dev/blog/typescript-decorators/
- TypeScript 5.0 Release Announcement (Stage 3 Decorators) - https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/
- TypeScript GitHub — Decorator Context Types (`ClassMethodDecoratorContext`, etc.) - https://github.com/microsoft/TypeScript/blob/main/src/lib/decorators.d.ts