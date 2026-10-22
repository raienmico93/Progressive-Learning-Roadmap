# Class Expressions in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **class expression** is a syntactic form that defines a class **within an expression context**, producing a constructor value that can be assigned to a variable, passed as an argument, or returned from a function. Unlike a class declaration, which introduces a binding into the enclosing scope, a class expression's optional name is scoped **only to the class body itself**, and the class value is used wherever the expression appears. **Anonymous class expressions** omit the name entirely, leaving only the variable or property to which the expression is assigned as the reference.

**Technical Definition.** In TypeScript and ECMAScript, a class expression has the form `class [BindingIdentifier] [extends Expression] { ClassBody }`. It is evaluated as an expression, producing a constructor function with the class's prototype and static members. The optional binding identifier is **not** introduced into the enclosing lexical scope; it is visible only within the class body, enabling self-reference. When a class expression is used in a statement position, the statement **must be terminated with a semicolon** to disambiguate it from a class declaration. TypeScript adds type annotations and generic parameters to class expressions just as it does to declarations, but the **instance type of an anonymous class expression cannot be referenced by name outside the expression**; it can only be matched **structurally** or inferred from the variable it is assigned to.

**Beginner-Friendly Explanation.** A class expression is **a class written as a value instead of a statement**. You write `const MyClass = class { ... }` instead of `class MyClass { ... }`. The class is created **on the spot** and assigned to a variable—or returned from a function, or passed to another function. An **anonymous class** is simply a class expression without a name: `const MyClass = class { ... }`. You cannot refer to the class's type by name elsewhere, but TypeScript can still check it **structurally** and infer the type from the variable.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Expression context** | Class expressions appear where values are expected: assignments, arguments, return expressions, array elements. |
| **Optional name** | The class name may be omitted (anonymous) or supplied (named expression); a supplied name is scoped only to the class body. |
| **No outer binding** | Unlike class declarations, a named class expression does **not** introduce its name into the enclosing scope. |
| **Redefinable** | Class expressions allow the same variable to be reassigned to new class values; class declarations cannot be redeclared in the same scope. |
| **Semicolon requirement** | A statement containing a class expression **must** end with a semicolon. |
| **Structural typing** | The instance type of an anonymous class expression can be matched structurally; it cannot be named outside the expression. |
| **Mixins foundation** | Class expressions are the recommended pattern for implementing **mixins** in TypeScript. |

---

### Prerequisites

- **JavaScript classes**: `class`, constructors, fields, methods, and `extends`.
- **TypeScript class basics**: type annotations, generics, and `implements` clauses.
- **Expression vs. statement distinction**: understanding that expressions produce values.
- **Function expressions and arrow functions** as a conceptual analogue: class expressions are to class declarations what function expressions are to function declarations.

---

### Related Programming Areas

- **Mixin patterns** in TypeScript, which rely on class expressions to extend a base class dynamically.
- **Factory functions** that return class constructors.
- **Higher-order components and decorators** in framework code.
- **Module patterns** and **IIFE-wrapped classes**.
- **Dynamic class creation** in testing, plugin systems, and metaprogramming.

---

### Core Concepts / Features

The following sections cover the requested concepts. Each follows the uniform structure.

---

## 1. Class Expressions

### Definitions

**Core Definition.** A **class expression** is a class definition used in an **expression position**, producing a constructor value that can be assigned, passed, or returned. It may be **named** or **anonymous**, and its name (if present) is scoped only to the class body.

**Technical Definition.** A class expression is evaluated as a JavaScript expression. It creates a constructor function whose prototype contains the class's instance methods and whose own properties include the class's static members. The optional `BindingIdentifier` is bound **inside the class body only**; it does not leak to the enclosing scope. TypeScript applies the same type-checking rules to class expressions as to declarations, including field initialization checks, method compatibility, and access modifier enforcement. The **instance type** of a class expression is available through the variable to which it is assigned (via inference or explicit annotation) or through structural matching; it cannot be referenced by a class name that is not in scope. When a class expression is the initializer of a `const` or `let` declaration, the statement **must** be terminated with a semicolon.

**Beginner-Friendly Explanation.** A class expression is **a class that you create inside an assignment or a function call**. Instead of declaring a class as a standalone statement, you write `const C = class { ... }`. The class exists as a value—you can store it, pass it around, or return it. If you give it a name (`class Named { ... }`), that name only works **inside the class itself**; outside, you must use the variable. If you omit the name, it is anonymous, and the variable is your only handle. Remember to put a **semicolon** after the statement when the class expression is used in statement position.

### Purposes

- **To create classes as values** that can be assigned, passed, or returned dynamically.
- **To enable mixin patterns** by returning class expressions that extend a base class.
- **To support factory functions** that produce class constructors with captured configuration.
- **To allow conditional or dynamic class definition** based on runtime conditions.
- **To provide scoped class names** that do not pollute the enclosing namespace.

### Syntax Rules and Structure

**General syntax (anonymous class expression):**

```
const Variable = class {
  // members
};
```

**General syntax (named class expression):**

```ts
const Variable = class InternalName {
  // members
};
```

**General syntax (with heritage):**

```ts
const Variable = class extends Base implements Interface {
  // members
};
```

**Component breakdown:**

- `const Variable` — the binding that receives the class value.
- `= class` — introduces a class expression.
- `InternalName` (optional) — a name scoped **only to the class body**.
- `extends Base` — optional base class.
- `implements Interface` — optional interface implementation.
- `{ ... }` — class body with fields, methods, constructors, and accessors.
- `;` — **required** when the class expression is used in a statement position.

**Syntax rules.**

- A class expression may omit its name (anonymous) or include one (named).
- A named class expression's name is **not** available in the enclosing scope.
- Class expressions support `extends`, `implements`, generics, and all class members.
- When a statement begins with `const`, `let`, or `var` and contains a class expression, the statement **must** end with a semicolon.
- A class expression cannot begin an **expression statement** directly; it must be parenthesized or used in a context that expects an expression.

**Constraints and limitations.**

- The **instance type** of a class expression cannot be referenced by name outside the expression; use type inference or structural matching.
- Anonymous class expressions have **no name** in stack traces; named class expressions improve debugging output.
- Class expressions are **not hoisted**; they must be evaluated before use.
- Mixin patterns using class expressions create **singletons** and cannot be mapped to different variable types at the type-system level.

### Annotated Complete Step by Step Code Examples

#### Example 1: Anonymous class expression

```typescript
// example_class_expr_anonymous.ts
// Step 1: Assign an anonymous class expression to a const.
const Point = class {
  constructor(public x: number, public y: number) {}

  length(): number {
    return Math.sqrt(this.x ** 2 + this.y ** 2);
  }
};

// Step 2: Create an instance using the variable.
const p = new Point(3, 4);
console.log(p.length()); // Expected: 5
```

**Setup.** Compile with `npx tsc example_class_expr_anonymous.ts --strict`.

**Expected Output:**
```
5
```

**Why this output occurs.** The anonymous class expression produces a constructor assigned to `Point`. TypeScript infers the instance type from the variable, so `p.x`, `p.y`, and `p.length()` are all statically typed. The class itself has no name outside the expression, but the variable serves as the reference. The statement ends with a semicolon, as required.

---

#### Example 2: Named class expression with internal self-reference

```typescript
// example_class_expr_named.ts
// Step 1: Named class expression — name is scoped to the class body.
const Counter = class InternalCounter {
  private count = 0;

  increment(): this {
    this.count++;
    return this;
  }

  // The internal name is available for self-reference.
  static create(): InternalCounter {
    return new InternalCounter();
  }

  getValue(): number {
    return this.count;
  }
};

// Step 2: Use the variable externally.
const counter = Counter.create();
counter.increment().increment();
console.log(counter.getValue()); // Expected: 2

// InternalCounter; // Error: Cannot find name 'InternalCounter'.
```

**Expected Output:**
```
2
```

**Why this output occurs.** The class expression is named `InternalCounter`, but that name is **only in scope inside the class body**. The static `create` method uses the internal name to construct an instance. Outside, only the variable `Counter` is available. Attempting to reference `InternalCounter` externally is a compile-time error. This pattern is useful for self-referential static methods and recursion.

---

#### Example 3: Class expression with `extends` and `implements`

```typescript
// example_class_expr_heritage.ts
interface Identifiable {
  id: string;
}

class Entity {
  constructor(public id: string) {}
}

// Class expression extending a base and implementing an interface.
const User = class extends Entity implements Identifiable {
  constructor(id: string, public name: string) {
    super(id);
  }

  describe(): string {
    return `${this.name} (${this.id})`;
  }
};

const user = new User("u1", "Ada");
console.log(user.describe()); // Expected: Ada (u1)
```

**Expected Output:**
```
Ada (u1)
```

**Why this output occurs.** The class expression extends `Entity` and implements `Identifiable`. The compiler verifies the `implements` contract and the `super()` call. The instance type is inferred from `User`, enabling access to `name`, `id`, and `describe`.

---

#### Example 4: Factory function returning a class expression (mixin foundation)

```typescript
// example_class_expr_factory.ts
// Step 1: Define a constructor type.
type Constructor = new (...args: any[]) => {};

// Step 2: Factory returns a class expression extending a base.
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = new Date();

    getTimestamp(): string {
      return this.timestamp.toISOString();
    }
  };
}

// Step 3: Apply the mixin to a base class.
class User {
  constructor(public name: string) {}
}

const TimestampedUser = Timestamped(User);
const user = new TimestampedUser("Ada");

console.log(user.name);              // Expected: Ada
console.log(typeof user.getTimestamp()); // Expected: string
```

**Expected Output:**
```
Ada
string
```

**Why this output occurs.** `Timestamped` returns an **anonymous class expression** that extends the provided base class. This is the recommended **mixin pattern** in TypeScript. The returned class combines the base class's members (`name`) with the mixin's members (`timestamp`, `getTimestamp`). The class expression is used as a value, demonstrating its role in dynamic composition.

---

### Real-World Cases

- **Mixins**: TypeScript’s official mixin documentation uses class expressions as the core pattern for composing behavior.
- **Factory-generated classes**: ORM and validation libraries return class expressions configured with schema metadata.
- **Dynamic class creation in tests**: test helpers return class expressions with stubbed methods.
- **Conditional implementations**: choosing between two class expressions based on an environment flag.

---

## 2. Anonymous Classes

### Definitions

**Core Definition.** An **anonymous class** is a **class expression without a name**. It is created with `class { ... }` and is referenced exclusively through the variable, property, or expression to which it is assigned.

**Technical Definition.** An anonymous class expression omits the optional `BindingIdentifier`. The resulting constructor has **no internal name** for self-reference and appears as `(anonymous)` in stack traces. Its instance type is available through **inference** from the variable it is assigned to or through **structural matching**; it cannot be named elsewhere. Anonymous classes are the most common form of class expression in factory functions and mixins, where the class is created dynamically and does not need a stable name.

**Beginner-Friendly Explanation.** An anonymous class is **a class with no name**. You create it with `const MyClass = class { ... }` and use the variable to refer to it. It is useful when you are creating a class on the fly—for example, inside a function that returns a specialized class—and you do not need a separate name for it.

### Purposes

- **To create classes dynamically** without polluting the namespace with a name.
- **To return classes from factory functions** where the class is configured by the factory’s arguments.
- **To implement mixins** and other composition patterns that produce classes on demand.
- **To reduce naming overhead** when the class is used only through a variable.

### Syntax Rules and Structure

**General syntax:**

```ts
const Variable = class {
  // members
};
```

**Component breakdown:**

- `class` — introduces the anonymous class expression.
- `{ ... }` — the class body.
- `;` — required in statement position.

**Syntax rules.**

- The class name is **omitted** entirely.
- The class is referenced only through the variable or expression to which it is assigned.
- Anonymous classes may extend a base class and implement interfaces.
- Anonymous classes may be used in any expression context: arguments, return values, array elements.

**Constraints and limitations.**

- **No self-reference by name** inside the class body; use the variable (which may not be initialized yet) or restructure.
- **Stack traces** show `(anonymous)` instead of a meaningful class name, complicating debugging.
- The instance type cannot be named outside the expression; rely on inference or structural typing.
- Anonymous classes are **not hoisted**.

### Annotated Complete Step by Step Code Examples

#### Example 1: Anonymous class assigned to a variable

```typescript
// example_anonymous_basic.ts
const Greeter = class {
  greet(name: string): string {
    return `Hello, ${name}`;
  }
};

const g = new Greeter();
console.log(g.greet("Ada")); // Expected: Hello, Ada
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The class is anonymous but is assigned to `Greeter`. The variable serves as the constructor reference. TypeScript infers the instance type from the variable, so `greet` is available and typed.

---

#### Example 2: Anonymous class returned from a factory

```typescript
// example_anonymous_factory.ts
function createValidator(pattern: RegExp) {
  return class {
    validate(value: string): boolean {
      return pattern.test(value);
    }
  };
}

const EmailValidator = createValidator(/^[^@]+@[^@]+\.[^@]+$/);
const validator = new EmailValidator();

console.log(validator.validate("ada@example.com")); // Expected: true
console.log(validator.validate("not-an-email"));    // Expected: false
```

**Expected Output:**
```
true
false
```

**Why this output occurs.** The factory returns an **anonymous class expression** that captures the `pattern` argument in its closure. Each call to `createValidator` produces a distinct class. The class has no name, but the returned value is assigned to `EmailValidator` and used to create instances.

---

#### Example 3: Debugging trade-off — anonymous vs. named

```typescript
// example_anonymous_debug.ts
const Anonymous = class {
  boom(): never {
    throw new Error("failure");
  }
};

const Named = class NamedInternal {
  boom(): never {
    throw new Error("failure");
  }
};

try {
  new Anonymous().boom();
} catch (e) {
  console.log((e as Error).stack?.split("\n")[0]);
  // Expected: Error: failure
}

try {
  new Named().boom();
} catch (e) {
  console.log((e as Error).stack?.split("\n")[0]);
  // Expected: Error: failure
}
```

**Expected Output (approximate):**
```
Error: failure
Error: failure
```

**Why this output occurs.** Both throw the same error. However, in a full stack trace, the **named class expression** would show `NamedInternal` in the trace, while the **anonymous class** would show `(anonymous)` or only the call site. Named class expressions improve debuggability at the cost of a slightly longer declaration.

---

### Real-World Cases

- **Mixin factories**: `function Serializable<T extends Constructor>(Base: T) { return class extends Base { ... } }`.
- **ORM model generation**: `const UserModel = defineModel({ table: "users" })` where `defineModel` returns an anonymous class.
- **Testing utilities**: `const MockService = createMock({ method: () => 42 })`.
- **Conditional polyfills**: choosing between anonymous class implementations based on feature detection.

---

## 3. Class Expression Best Practices

### Definitions

**Core Definition.** **Class expression best practices** are guidelines for using class expressions and anonymous classes in TypeScript in ways that maximize **readability, debuggability, type safety, and maintainability**, while avoiding pitfalls such as missing semicolons, unnamed stack traces, and type-erasure surprises.

**Technical Definition.** Best practices center on **preferring class declarations for named, stable classes** and reserving class expressions for **dynamic composition, mixins, and factory patterns**. When class expressions are used, they should be **named** for debugging when practical, **terminated with semicolons** in statement position, **typed explicitly** when inference is insufficient, and **scoped narrowly** to avoid unnecessary exposure. The Google TypeScript Style Guide explicitly requires semicolons after statements containing class expressions and prohibits semicolons after class declarations. ESLint’s `no-class-assign` rule discourages reassigning class declarations, reinforcing the distinction between declarations (stable bindings) and expressions (dynamic values).

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep class expressions from causing confusion. The big rules are: use a **class declaration** when you have a normal, named class; use a **class expression** when you need to create a class dynamically (factory, mixin); give the class expression a **name** if you want better error messages; always put a **semicolon** after a statement that contains a class expression; and be aware that anonymous classes have **no name** in stack traces.

### Purposes

- **To choose the right form** (declaration vs. expression) for each class.
- **To improve debuggability** by naming class expressions where practical.
- **To avoid syntax errors** by correctly terminating class-expression statements with semicolons.
- **To keep types precise** by annotating class-expression variables when inference is insufficient.
- **To prevent accidental reassignment** of class declarations (use expressions when reassignment is intentional).

### Syntax Rules and Structure

Best practices are conventions that interact with class expression syntax. The core rules are:

1. **Prefer class declarations for stable, named classes.** Use expressions only for dynamic creation, mixins, factories, or scoped values.
2. **Use named class expressions for self-reference and debugging.** The name is scoped to the class body and improves stack traces.
3. **Terminate class-expression statements with semicolons.** A statement like `const C = class { ... }` must end with `;`.
4. **Do not terminate class declarations with semicolons.** `class C { ... }` must **not** end with `;`.
5. **Annotate the variable when inference is insufficient.** Use `const C: SomeConstructor = class { ... }` or a type alias to name the instance type.
6. **Avoid reassigning class declarations.** Use `no-class-assign` (ESLint) to enforce stability; use class expressions when reassignment is intended.
7. **Keep class expressions scoped narrowly.** Assign them to `const` variables inside the smallest scope that needs them.
8. **Use the mixin pattern with class expressions** for behavior composition rather than deep inheritance.

**Component breakdown (declaration vs. expression decision tree):**

```
Does the class need a stable, named binding in the enclosing scope?
  ├── Yes → Use a class declaration.
  └── No → Is the class created dynamically (factory, mixin, conditional)?
           ├── Yes → Use a class expression.
           │        ├── Does it need self-reference or better stack traces?
           │        │   ├── Yes → Use a named class expression.
           │        │   └── No  → Use an anonymous class expression.
           │        └── End the statement with a semicolon.
           └── No → Reconsider; a class declaration is likely clearer.
```

**Constraints and limitations.**

- Named class expressions improve debugging but do not introduce the name into the outer scope.
- Semicolon rules differ between declarations and expressions; mixing them causes lint errors or syntax issues.
- The instance type of an anonymous class cannot be named elsewhere; structural typing or explicit type aliases may be required.
- Mixin patterns using class expressions create **singletons** and cannot be mapped to different variable types at the type-system level.

### Annotated Complete Step by Step Code Examples

#### Example 1: Declaration vs. expression — correct semicolon usage

```typescript
// example_best_semicolons.ts
// ✅ Class declaration: NO semicolon.
class Foo {
  method(): number {
    return 1;
  }
}

// ✅ Class expression in a statement: MUST have a semicolon.
export const Baz = class extends Foo {
  method(): number {
    return 2;
  }
}; // Semicolon required here.

console.log(new Foo().method()); // Expected: 1
console.log(new Baz().method()); // Expected: 2
```

**Expected Output:**
```
1
2
```

**Why this output occurs.** The class declaration `class Foo { ... }` correctly omits the semicolon. The statement `export const Baz = class extends Foo { ... };` correctly includes the semicolon because it contains a class expression. The Google TypeScript Style Guide explicitly distinguishes these cases: declarations must **not** end with semicolons; statements containing class expressions **must** end with semicolons.

---

#### Example 2: Naming class expressions for debugging

```typescript
// example_best_named.ts
// ✅ Named class expression: self-reference and better stack traces.
const Task = class TaskImpl {
  run(): void {
    throw new Error("Task failed");
  }

  static create(): TaskImpl {
    return new TaskImpl();
  }
};

const task = Task.create();

try {
  task.run();
} catch (e) {
  console.log((e as Error).stack?.includes("TaskImpl"));
  // Expected: true (the class name appears in the stack trace).
}
```

**Expected Output:**
```
true
```

**Why this output occurs.** The class expression is named `TaskImpl`. Although `TaskImpl` is not available in the outer scope, it **is** used in the generated stack trace, making debugging easier. The static `create` method also uses the internal name for self-reference.

---

#### Example 3: Explicit typing for anonymous class expressions

```typescript
// example_best_explicit_type.ts
// Type alias naming the instance shape.
interface Greeter {
  greet(name: string): string;
}

// Explicit constructor type for the variable.
type GreeterConstructor = new () => Greeter;

const GreeterClass: GreeterConstructor = class {
  greet(name: string): string {
    return `Hello, ${name}`;
  }
};

const g = new GreeterClass();
console.log(g.greet("Ada")); // Expected: Hello, Ada
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The anonymous class expression is assigned to a variable with the explicit `GreeterConstructor` type. This names the instance type (`Greeter`) externally, compensating for the anonymous class’s inability to be referenced by name. The compiler verifies that the class satisfies the constructor type.

---

#### Example 4: Mixin pattern with class expressions (recommended)

```typescript
// example_best_mixin.ts
type Constructor = new (...args: any[]) => {};

// Mixin factory returns a class expression extending the base.
function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActive = false;

    activate(): void {
      this.isActive = true;
    }

    deactivate(): void {
      this.isActive = false;
    }
  };
}

class User {
  constructor(public name: string) {}
}

const ActiveUser = Activatable(User);
const user = new ActiveUser("Ada");

user.activate();
console.log(user.isActive); // Expected: true
console.log(user.name);     // Expected: Ada
```

**Expected Output:**
```
true
Ada
```

**Why this output occurs.** The `Activatable` mixin returns an **anonymous class expression** that extends the provided base class. This is the pattern recommended by TypeScript’s official mixin documentation. The resulting class combines the base’s `name` with the mixin’s `isActive`, `activate`, and `deactivate`. Class expressions are the correct tool for this dynamic composition.

---

### Real-World Cases

- **Mixin libraries**: TypeScript’s official mixin documentation uses class expressions as the primary pattern.
- **ORM and schema libraries**: `defineModel()` returns a class expression configured with schema metadata.
- **Dependency injection containers**: factories return class expressions with injected dependencies captured in closures.
- **Testing mocks**: `createMock()` returns an anonymous class expression that implements a given interface with stubbed behavior.
- **Style guides and linting**: Google’s TypeScript Style Guide enforces semicolon rules for class expressions; ESLint’s `no-class-assign` prevents reassigning class declarations.

---

## References

- TypeScript Handbook — Mixins (Class Expression Pattern) - https://www.typescriptlang.org/docs/handbook/mixins.html
- TypeScript 1.6 Release Notes — Class Expressions - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-6.html#class-expressions
- TypeScript Handbook — Classes (Current) - https://www.typescriptlang.org/docs/handbook/2/classes.html
- MDN Web Docs — Class Expression - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/class
- MDN Web Docs — Classes (Reference) - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- Google TypeScript Style Guide — Class Declarations and Class Expressions (Semicolon Rules) - https://google.github.io/styleguide/tsguide.html
- ESLint — `no-class-assign` (Disallow Reassigning Class Declarations) - https://eslint.org/docs/latest/rules/no-class-assign
- TypeScript ESLint — `member-ordering` (Ordering for Classes and Class Expressions) - https://typescript-eslint.io/rules/member-ordering/
- Microsoft/TypeScript Issue #497 — Class Expressions (Design Discussion) - https://github.com/microsoft/TypeScript/issues/497
- Microsoft/TypeScript Wiki — What’s New in TypeScript (Class Expressions and Extending Expressions) - https://github.com/microsoft/TypeScript/wiki/What's-new-in-TypeScript