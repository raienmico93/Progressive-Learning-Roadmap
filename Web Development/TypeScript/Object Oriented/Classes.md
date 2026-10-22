# Classes in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **class** in TypeScript is a **runtime construct** that serves a dual role: it is both a **value** (a JavaScript constructor function) and a **type** (the type of instances created by that constructor). TypeScript extends JavaScript's ES2015 `class` syntax with **type annotations**, **access modifiers**, **readonly properties**, **parameter properties**, and **compile-time member checking**, while preserving the runtime semantics of the underlying JavaScript class.

**Technical Definition.** A class declaration introduces a **named type** and a **constructor value** into the lexical scope. The type describes the **instance shape** — the properties and methods available on objects created via `new`. The value is the constructor function itself, which may have **static members**. TypeScript adds a **type annotation layer** over JavaScript classes: fields, methods, and accessors can be annotated; visibility modifiers (`public`, `private`, `protected`) restrict access at compile time; `readonly` restricts assignment after initialization; and **parameter properties** merge constructor parameters with field declarations. All TypeScript-specific class features are **erased** during compilation, except for runtime constructs such as `enum` and decorator metadata.

**Beginner-Friendly Explanation.** A class is **a blueprint for creating objects**. It describes what data an object holds (its **properties**) and what it can do (its **methods**). In TypeScript, a class is also a **type**, so the compiler knows exactly what properties and methods every instance will have. TypeScript adds **labels** to the blueprint — `private` means “only this class can touch it,” `readonly` means “set it once and never change it” — but those labels disappear when the code runs. They exist to help you and your editor catch mistakes before the program executes.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Dual value/type role** | A class is both a constructor function (value) and a type describing its instances. |
| **Type annotations on members** | Fields, methods, accessors, and parameters can carry type annotations. |
| **Compile-time visibility** | `public`, `private`, and `protected` are enforced **only at compile time**; they have no runtime effect unless ECMAScript `#` private fields are used. |
| **Readonly enforcement** | `readonly` fields can be assigned only at declaration or in the constructor. |
| **Parameter properties** | Constructor parameters prefixed with a visibility modifier or `readonly` automatically become class fields. |
| **Structural compatibility** | Classes are compared structurally unless `private` or `protected` members introduce nominal-like behavior. |
| **Erasure of TS-only features** | Annotations, modifiers, and parameter properties are erased; only standard JavaScript class syntax remains. |

---

### Prerequisites

- **JavaScript ES2015 classes**: `class`, `constructor`, `new`, `this`, `extends`, `super`.
- **TypeScript basics**: type annotations, interfaces, and type aliases.
- **Object-oriented concepts**: properties, methods, inheritance, encapsulation.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Object-oriented programming (OOP)** : encapsulation, inheritance, polymorphism.
- **Design patterns**: Factory, Builder, Singleton, Strategy — often implemented with classes.
- **Dependency injection** and **composition over inheritance**.
- **Structural vs. nominal typing** in TypeScript.
- **Runtime privacy** via ECMAScript `#` private fields.

---

### Core Concepts / Features

The following sections cover the requested core concepts. Each follows the uniform structure.

---

## 1. Classes

### Definitions

**Core Definition.** A **class** is a reusable code template that defines the **state** (properties) and **behavior** (methods) of objects created from it. In TypeScript, a class is simultaneously a **constructor value** and a **type** describing its instances.

**Technical Definition.** A class declaration `class C { ... }` introduces a **type** `C` (the instance type) and a **value** `C` (the constructor function). The instance type includes all non-static members declared in the class body and inherited members. The constructor value includes the constructor signature and all static members. TypeScript checks that instances satisfy the declared shape and that member accesses respect visibility and readonly constraints.

**Beginner-Friendly Explanation.** A class is **a cookie cutter**. The class itself is not a cookie; it is the tool that makes cookies. Each cookie (instance) has the same shape but its own data. TypeScript adds a **label** to the cutter saying what kind of cookie it produces, so the compiler knows what to expect.

### Purposes

- **To bundle related data and behavior** into a single, cohesive unit.
- **To define a reusable type** that can be used in annotations, parameters, and return types.
- **To enforce initialization** of instance state through constructors.
- **To support inheritance and polymorphism** with compile-time checking.

### Syntax Rules and Structure

**General syntax (basic class declaration):**

```
class ClassName {
  // fields, constructor, methods
}
```

**Component breakdown:**

- `class` — the declaration keyword.
- `ClassName` — the identifier; by convention PascalCase.
- `{ ... }` — the **class body**, containing members.

**Syntax rules.**

- A class body may contain fields, constructors, methods, accessors (`get`/`set`), and static members.
- Class declarations are **not hoisted** in the same way as function declarations; they must be declared before use.
- A class may `extends` at most one base class and may `implements` multiple interfaces.
- TypeScript classes support generic type parameters: `class Box<T> { ... }`.

**Constraints and limitations.**

- TypeScript-specific class features (`private`, `readonly`, parameter properties) are **compile-time only** and are erased.
- Classes are compared **structurally** unless `private` or `protected` members are present.
- `this` binding must be handled with arrow functions or the `this` parameter when methods are detached.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic class with fields, constructor, and method

```typescript
// example_class_basic.ts
// Step 1: Declare a class.
class Point {
  // Step 2: Declare typed fields.
  x: number;
  y: number;

  // Step 3: Define a constructor that initializes fields.
  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }

  // Step 4: Define a method.
  distanceTo(other: Point): number {
    return Math.hypot(this.x - other.x, this.y - other.y);
  }
}

// Step 5: Create instances.
const p1 = new Point(0, 0);
const p2 = new Point(3, 4);

console.log(p1.distanceTo(p2)); // Expected: 5
```

**Setup.** Compile with `npx tsc example_class_basic.ts --strict` and run `node example_class_basic.js`.

**Expected Output:**
```
5
```

**Why this output occurs.** `Point` declares two `number` fields and a constructor that assigns them. The `distanceTo` method computes the Euclidean distance using `Math.hypot(3, 4)`, which is `5`. The compiler verifies that `p1` and `p2` are `Point` instances and that `distanceTo` receives a `Point`.

---

#### Example 2: Class used as a type in annotations

```typescript
// example_class_type.ts
// Step 1: Define a class.
class User {
  constructor(public id: string, public name: string) {}
}

// Step 2: Use the class as a type annotation.
function greet(user: User): string {
  return `Hello, ${user.name}`;
}

// Step 3: Call with an instance.
const ada = new User("u1", "Ada");
console.log(greet(ada)); // Expected: Hello, Ada

// Step 4: Structurally compatible object literal is accepted.
console.log(greet({ id: "u2", name: "Grace" })); // Expected: Hello, Grace
```

**Expected Output:**
```
Hello, Ada
Hello, Grace
```

**Why this output occurs.** `User` serves as both a constructor and a type. The `greet` function accepts `User`, and `ada` is a `User` instance. The object literal `{ id: "u2", name: "Grace" }` is also accepted because TypeScript’s structural compatibility checks only the **public shape** — and `User` has only public parameter properties. This demonstrates the dual value/type role of classes.

---

### Real-World Cases

- **Domain models**: `class Order { ... }` encapsulates order data and behavior (totals, validation).
- **Service layers**: `class UserService { ... }` groups related operations with dependency-injected clients.
- **Custom errors**: `class ValidationError extends Error { ... }` creates typed, catchable error classes.

---

## 2. `class` Keyword

### Definitions

**Core Definition.** The **`class` keyword** is the syntactic token that introduces a class declaration or class expression in TypeScript and JavaScript (ES2015+). It may be followed by an optional class name, an optional type-parameter list, an optional `extends` clause, an optional `implements` clause, and a class body.

**Technical Definition.** In TypeScript’s grammar, a class declaration is `class BindingIdentifier TypeParameters? ClassHeritage? { ClassBody }`. The `class` token is a **declaration keyword** that creates a lexical binding for the class name and establishes the class’s type and value meanings. Class expressions (`const C = class { ... }`) use the same keyword without a required binding name.

**Beginner-Friendly Explanation.** `class` is **the word you type to start a class**. Everything after it — the name, the body, the `extends` clause — describes the class. It is the TypeScript/JavaScript equivalent of saying “I am defining a blueprint now.”

### Purposes

- **To declare a named class** that introduces both a type and a constructor value.
- **To create class expressions** for anonymous or locally scoped classes.
- **To establish the syntactic foundation** for inheritance (`extends`) and interface implementation (`implements`).
- **To support generic classes** with type parameters.

### Syntax Rules and Structure

**General syntax (declaration with heritage and type parameters):**

```
class ClassName<T> extends BaseClass implements Interface1, Interface2 {
  // body
}
```

**Component breakdown:**

- `class` — declaration keyword.
- `ClassName` — binding identifier (PascalCase convention).
- `<T>` — optional type-parameter list.
- `extends BaseClass` — optional single base class.
- `implements Interface1, Interface2` — optional interface implementation list.
- `{ ... }` — class body.

**Syntax rules.**

- A class declaration is **not hoisted**; it must appear before use in the same scope.
- Class expressions may omit the name: `const C = class { ... }`.
- `extends` accepts a value expression that evaluates to a constructor (or `null`).
- `implements` accepts a type expression (interface, class type, or object type).
- Type parameters on classes are erased and cannot be used at runtime.

**Constraints and limitations.**

- A class may extend **only one** base class (single inheritance).
- A class may implement **multiple** interfaces.
- TypeScript’s `implements` clause is a **compile-time check** only; it is erased.
- `class` declarations are subject to the **temporal dead zone** in block scopes.

### Annotated Complete Step by Step Code Examples

#### Example 1: Class declaration vs. class expression

```typescript
// example_class_keyword.ts
// Step 1: Class declaration with a name.
class Greeter {
  greet(name: string): string {
    return `Hello, ${name}`;
  }
}

// Step 2: Class expression assigned to a const.
const Fareweller = class {
  farewell(name: string): string {
    return `Goodbye, ${name}`;
  }
};

// Step 3: Use both.
console.log(new Greeter().greet("Ada"));       // Expected: Hello, Ada
console.log(new Fareweller().farewell("Ada")); // Expected: Goodbye, Ada
```

**Expected Output:**
```
Hello, Ada
Goodbye, Ada
```

**Why this output occurs.** The `class` keyword introduces both declarations. The declaration form creates a named binding `Greeter`; the expression form creates an anonymous class assigned to `Fareweller`. Both produce constructor values and instance types.

---

#### Example 2: Generic class with `extends` and `implements`

```typescript
// example_class_generic_heritage.ts
// Step 1: Define an interface.
interface Identifiable {
  id: string;
}

// Step 2: Define a base class.
class Entity {
  constructor(public id: string) {}
}

// Step 3: Define a generic subclass implementing the interface.
class Repository<T extends Entity & Identifiable> extends Entity
  implements Identifiable {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  count(): number {
    return this.items.length;
  }
}

// Step 4: Use the class.
const repo = new Repository("users");
repo.add(new Entity("u1") as Entity & Identifiable);
console.log(repo.count()); // Expected: 1
```

**Expected Output:**
```
1
```

**Why this output occurs.** The `class` keyword introduces `Repository` with a type parameter `T` constrained to `Entity & Identifiable`. The `extends Entity` clause makes `Repository` inherit the `id` property, and `implements Identifiable` is statically checked. The generic type parameter is erased at runtime, but the constraint is enforced during compilation.

---

### Real-World Cases

- **Framework base classes**: `class BaseController extends FrameworkController { ... }`.
- **Generic containers**: `class Stack<T> { ... }`, `class Cache<K, V> { ... }`.
- **Anonymous class expressions** in factory functions: `return class { ... }`.

---

## 3. Class Declaration

### Definitions

**Core Definition.** A **class declaration** is the statement form of a class definition that introduces a **named binding** for the class in the enclosing scope. It is the most common way to define classes in TypeScript.

**Technical Definition.** A class declaration `class C { ... }` creates a **lexical binding** `C` in the current scope. The binding has both a **type meaning** (the instance type) and a **value meaning** (the constructor function). Class declarations are **block-scoped** and are subject to the temporal dead zone: they cannot be referenced before the declaration is evaluated.

**Beginner-Friendly Explanation.** A class declaration is **the normal way you write a class** — you give it a name, and that name can be used as both a constructor and a type. It is a statement, like `let` or `function`, and it is scoped to the block it appears in.

### Purposes

- **To introduce a named class type and constructor** into a scope.
- **To support inheritance and interface implementation** with a clear declaration structure.
- **To serve as the canonical form** for defining reusable object blueprints.

### Syntax Rules and Structure

**General syntax:**

```
class ClassName {
  // members
}
```

**Component breakdown:**

- `class` — keyword.
- `ClassName` — identifier bound in the current scope.
- `{ ... }` — body with fields, constructor, methods, accessors, static members.

**Syntax rules.**

- Class declarations are **block-scoped** and **not hoisted** like function declarations.
- The class name is available inside the class body (useful for static methods and recursion).
- A class declaration may be `export`ed: `export class C { ... }`.
- A class declaration may be `default`-exported: `export default class C { ... }`.

**Constraints and limitations.**

- Referencing the class before its declaration in the same scope causes a `ReferenceError` (temporal dead zone).
- Class declarations cannot be conditionally redeclared in the same scope.
- TypeScript-specific members are erased; the emitted JavaScript is a standard class.

### Annotated Complete Step by Step Code Examples

#### Example 1: Named class declaration with export

```typescript
// example_declaration.ts
// Step 1: Declare and export a class.
export class Temperature {
  constructor(private celsius: number) {}

  toFahrenheit(): number {
    return (this.celsius * 9) / 5 + 32;
  }
}

// Step 2: Use the class in the same module.
const temp = new Temperature(100);
console.log(temp.toFahrenheit()); // Expected: 212
```

**Expected Output:**
```
212
```

**Why this output occurs.** The class declaration binds `Temperature` as both a type and a constructor. `new Temperature(100)` creates an instance with `celsius = 100`, and `toFahrenheit` computes `212`. The `export` keyword makes the class available to other modules.

---

#### Example 2: Temporal dead zone behavior

```typescript
// example_tdz.ts
// Step 1: This is valid — class is declared before use.
class A {
  value = 1;
}
console.log(new A().value); // Expected: 1

// Step 2: The following would throw a ReferenceError.
// console.log(new B());
// class B { value = 2; }
// Error: Cannot access 'B' before initialization
```

**Expected Output:**
```
1
```

**Why this output occurs.** `A` is declared before `new A()` is evaluated, so it works. `B` is referenced before its declaration is evaluated, which triggers the temporal dead zone and throws a `ReferenceError` at runtime. This distinguishes class declarations from hoisted function declarations.

---

### Real-World Cases

- **Exported domain classes**: `export class Invoice { ... }` in a module.
- **Default-exported framework classes**: `export default class App extends Component { ... }`.
- **Namespace-scoped classes** in declaration files.

---

## 4. Class Members

### Definitions

**Core Definition.** **Class members** are the constituents of a class body: **fields** (properties), **methods**, **constructors**, **accessors** (`get`/`set`), and **static members**. They define the structure and behavior of instances (and, for static members, of the constructor itself).

**Technical Definition.** A class body contains **member declarations**. Instance members are installed on the prototype (methods, accessors) or on each instance (fields). Static members are installed on the constructor function. TypeScript annotates each member with a type and optionally a visibility modifier, `readonly`, or `static`. Member names may be identifiers, string literals, numeric literals, or computed property names.

**Beginner-Friendly Explanation.** Class members are **the pieces inside a class**. Fields are the data; methods are the actions; the constructor is the setup routine; accessors are controlled getters and setters; static members belong to the class itself rather than to instances.

### Purposes

- **To define the data and behavior** of instances and the class itself.
- **To control access** to data through visibility modifiers.
- **To provide controlled read/write access** through accessors.
- **To attach class-level utilities** through static members.

### Syntax Rules and Structure

**General syntax (member categories):**

```
class C {
  field: Type = value;                  // instance field
  static staticField: Type = value;     // static field
  constructor(params) { ... }           // constructor
  method(params): ReturnType { ... }    // instance method
  static staticMethod(): ReturnType { ... } // static method
  get accessor(): Type { ... }          // getter
  set accessor(value: Type) { ... }     // setter
}
```

**Component breakdown:**

- `field: Type = value` — instance field with annotation and optional initializer.
- `static` — marks a member as belonging to the constructor, not instances.
- `constructor(params)` — initialization routine; no return type annotation.
- `method(params): ReturnType` — instance method; `ReturnType` may be omitted and inferred.
- `get` / `set` — accessor declarations; getters must not have parameters, setters must have exactly one.

**Syntax rules.**

- Fields may be annotated, initialized, or both; unannotated uninitialized fields are implicitly `any` without `strictPropertyInitialization`.
- `static` members are accessed via the class name: `C.staticMember`.
- Accessors cannot have the same name as a field in the same class.
- Computed member names are allowed: `[Symbol.iterator]() { ... }`.

**Constraints and limitations.**

- Instance fields are **not** on the prototype; they are per-instance properties.
- `static` members are inherited by subclasses but are not available on instances.
- TypeScript accessors are compiled to `Object.defineProperty` on the prototype.
- Member names cannot be `constructor`.

### Annotated Complete Step by Step Code Examples

#### Example 1: All member categories in one class

```typescript
// example_members.ts
class Counter {
  // Instance field with initializer.
  count: number = 0;

  // Static field.
  static instances: number = 0;

  // Constructor.
  constructor(public label: string) {
    Counter.instances++;
  }

  // Instance method.
  increment(): void {
    this.count++;
  }

  // Static method.
  static resetInstances(): void {
    Counter.instances = 0;
  }

  // Getter.
  get description(): string {
    return `${this.label}: ${this.count}`;
  }

  // Setter.
  set description(value: string) {
    const [label, count] = value.split(": ");
    this.label = label;
    this.count = Number(count);
  }
}

const c = new Counter("clicks");
c.increment();
c.increment();
console.log(c.description);        // Expected: clicks: 2
console.log(Counter.instances);    // Expected: 1

c.description = "taps: 10";
console.log(c.count);              // Expected: 10
```

**Expected Output:**
```
clicks: 2
1
10
```

**Why this output occurs.** `count` is an instance field initialized to `0`; two `increment` calls make it `2`. `instances` is a static field incremented in the constructor. The getter returns a formatted string; the setter parses the string and updates both `label` and `count`. All member categories are exercised.

---

#### Example 2: Static vs. instance member access

```typescript
// example_static_vs_instance.ts
class MathUtils {
  static readonly PI = 3.14159;

  static circleArea(radius: number): number {
    return MathUtils.PI * radius ** 2;
  }

  description = "utility";
}

console.log(MathUtils.circleArea(2)); // Expected: 12.56636
console.log(new MathUtils().description); // Expected: utility

// MathUtils.description; // Error: Property 'description' does not exist on type 'typeof MathUtils'.
// new MathUtils().circleArea(2); // Error: Property 'circleArea' does not exist on type 'MathUtils'.
```

**Expected Output:**
```
12.56636
utility
```

**Why this output occurs.** `PI` and `circleArea` are static members, accessible only on the class (constructor) itself. `description` is an instance field, accessible only on instances. The compiler enforces this separation, preventing accidental cross-access.

---

### Real-World Cases

- **Entity classes**: fields for data, methods for behavior, static factory methods for creation.
- **Utility classes**: static methods and constants only.
- **Reactive models**: accessors that notify subscribers on `set`.

---

## 5. Properties (Fields)

### Definitions

**Core Definition.** A **property** (also called a **field**) is a class member that holds **data** on an instance or on the class itself. In TypeScript, properties can be annotated with types, initialized with values, and modified with visibility modifiers, `readonly`, or `static`.

**Technical Definition.** An instance field declaration `name: Type` creates a property on each instance created by the constructor. An initializer `name: Type = value` is evaluated during construction (after `super()` in derived classes). TypeScript checks assignments to the field against its annotated type. Under `strictPropertyInitialization`, every field must be initialized either at declaration or in the constructor.

**Beginner-Friendly Explanation.** A property is **a piece of data attached to an object**. `name: string` means every instance has a `name` that must be a string. You can give it a starting value (`= "unknown"`), and TypeScript will make sure you never accidentally assign a number to it.

### Purposes

- **To store per-instance or per-class state**.
- **To enforce type safety on data** through annotations.
- **To ensure initialization** of instance state before use.
- **To control mutability** through `readonly` and visibility modifiers.

### Syntax Rules and Structure

**General syntax:**

```
[modifiers] fieldName: Type [= initializer];
```

**Component breakdown:**

- `modifiers` — optional `public` / `private` / `protected` / `readonly` / `static` combination.
- `fieldName` — identifier, string literal, numeric literal, or computed property.
- `: Type` — optional annotation; if omitted with an initializer, the type is inferred.
- `= initializer` — optional value; evaluated during construction.

**Syntax rules.**

- `strictPropertyInitialization` requires initialization at declaration or in the constructor.
- Definite assignment assertion `!` can suppress the check: `name!: string;`.
- `readonly` fields may be assigned only at declaration or in the constructor.
- `static` fields belong to the constructor, not instances.
- Computed property names are allowed: `[Symbol.toStringTag]: string`.

**Constraints and limitations.**

- Uninitialized fields without `strictPropertyInitialization` are implicitly `any` and may be `undefined` at runtime.
- Fields are **per-instance**; changing a field on one instance does not affect others (unless the field is `static`).
- TypeScript field declarations may emit runtime initialization code, affecting class semantics compared to JavaScript prototype-based fields.

### Annotated Complete Step by Step Code Examples

#### Example 1: Typed fields with initialization

```typescript
// example_fields.ts
class User {
  // Annotated and initialized.
  name: string = "Anonymous";

  // Annotated, initialized in constructor.
  email: string;

  // Inferred from initializer.
  active = true;

  // Readonly field.
  readonly createdAt: Date = new Date();

  constructor(email: string) {
    this.email = email;
  }
}

const user = new User("ada@example.com");
console.log(user.name);       // Expected: Anonymous
console.log(user.email);      // Expected: ada@example.com
console.log(user.active);     // Expected: true

// user.createdAt = new Date(); // Error: Cannot assign to 'createdAt' because it is a read-only property.
```

**Expected Output:**
```
Anonymous
ada@example.com
true
```

**Why this output occurs.** `name` has a default initializer, so every instance starts with `"Anonymous"`. `email` is assigned in the constructor. `active` is inferred as `boolean` from `true`. `createdAt` is `readonly`, so reassignment is a compile-time error. `strictPropertyInitialization` is satisfied because all fields are initialized.

---

#### Example 2: Definite assignment assertion and optional fields

```typescript
// example_fields_assertion.ts
class Config {
  // Definite assignment assertion — tells the compiler it will be set.
  host!: string;

  // Optional field — may be undefined.
  port?: number;

  initialize() {
    this.host = "localhost";
  }
}

const config = new Config();
config.initialize();
console.log(config.host); // Expected: localhost
console.log(config.port); // Expected: undefined
```

**Expected Output:**
```
localhost
undefined
```

**Why this output occurs.** `host!` suppresses `strictPropertyInitialization`; the field is declared but not initialized at declaration or in the constructor, and the `!` asserts that it will be assigned before use. `port?` makes the field optional, so its type is `number | undefined`. At runtime, `port` has not been assigned, so it is `undefined`.

---

### Real-World Cases

- **Configuration objects**: `host!: string` with a separate `initialize` method.
- **Entity IDs**: `readonly id: string` assigned once in the constructor.
- **Optional metadata**: `tags?: string[]` for optional fields.

---

## 6. Methods

### Definitions

**Core Definition.** A **method** is a class member that defines **behavior** — a function associated with instances (or with the class itself, if `static`). Methods can accept parameters, return values, and access instance state via `this`.

**Technical Definition.** An instance method declaration installs a function on the class’s prototype. A static method installs a function on the constructor. TypeScript annotates parameter types and return types, and checks calls against the method signature. The `this` type can be explicitly annotated as the first parameter to control the method’s binding context.

**Beginner-Friendly Explanation.** A method is **an action an object can perform**. `deposit(amount: number)` on a `BankAccount` class means every account can deposit money. TypeScript checks that you pass a number and that the method’s return value (if any) is used correctly.

### Purposes

- **To encapsulate behavior** alongside the data it operates on.
- **To provide a typed interface** for object operations.
- **To support static utility functions** that do not require an instance.
- **To control `this` binding** through the `this` parameter.

### Syntax Rules and Structure

**General syntax:**

```
[modifiers] methodName(params): ReturnType {
  // body
}
```

**Component breakdown:**

- `modifiers` — optional `public` / `private` / `protected` / `static`.
- `methodName` — identifier, string/numeric literal, or computed name.
- `params` — parameter list with optional annotations, defaults, and rest parameters.
- `ReturnType` — optional annotation; inferred from the body if omitted.

**Syntax rules.**

- Methods are installed on the prototype; they are shared across instances.
- `this` inside a method refers to the instance (unless the method is detached).
- The `this` parameter can be declared as the first parameter: `method(this: C, ...) { ... }`.
- Arrow function fields (`method = () => { ... }`) bind `this` lexically and are per-instance.

**Constraints and limitations.**

- Detached methods lose their `this` binding unless bound or defined as arrow function fields.
- Overloading is supported via multiple method signatures followed by an implementation signature.
- `static` methods cannot access instance members via `this`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Instance and static methods

```typescript
// example_methods.ts
class Calculator {
  // Instance method.
  add(a: number, b: number): number {
    return a + b;
  }

  // Static method.
  static multiply(a: number, b: number): number {
    return a * b;
  }

  // Arrow function field — lexically bound this.
  log = (message: string): void => {
    console.log(message);
  };
}

const calc = new Calculator();
console.log(calc.add(2, 3));           // Expected: 5
console.log(Calculator.multiply(2, 3)); // Expected: 6
calc.log("done");                      // Expected: done

const detachedAdd = calc.add;
// console.log(detachedAdd(2, 3)); // Error at runtime: this is undefined.
```

**Expected Output:**
```
5
6
done
```

**Why this output occurs.** `add` is a prototype method; `multiply` is a static method on the constructor. `log` is an arrow function field, so `this` is permanently bound to the instance. Detaching `add` loses its `this` binding, which is why the commented call would fail at runtime.

---

#### Example 2: Method with `this` parameter

```typescript
// example_this_param.ts
class Counter {
  count = 0;

  increment(this: Counter, by: number = 1): void {
    this.count += by;
  }
}

const counter = new Counter();
const increment = counter.increment;

// The this parameter ensures the method is called on a Counter.
increment.call(counter, 5);
console.log(counter.count); // Expected: 5

// increment(5); // Error: The 'this' context of type 'void' is not assignable to method's 'this' of type 'Counter'.
```

**Expected Output:**
```
5
```

**Why this output occurs.** The `this: Counter` parameter tells the compiler that `increment` must be called with a `Counter` as `this`. Using `.call(counter, 5)` satisfies this requirement and increments the count. Calling `increment(5)` without a proper `this` is rejected at compile time.

---

### Real-World Cases

- **Service methods**: `class UserService { async findById(id: string): Promise<User> { ... } }`.
- **Static helpers**: `class StringUtils { static capitalize(s: string): string { ... } }`.
- **Event handlers**: arrow function fields for stable `this` in callbacks.

---

## 7. Constructors

### Definitions

**Core Definition.** A **constructor** is a special class member that is automatically invoked when an instance is created with `new`. It initializes instance state, validates input, and may call the parent constructor via `super()`.

**Technical Definition.** The `constructor` member defines the **construction signature** of the class. It has no return type annotation because it always returns the instance. In a derived class, the constructor must call `super(...)` before accessing `this`. TypeScript checks constructor arguments against the declared parameter types and enforces `strictPropertyInitialization` where applicable. Constructors may be overloaded with multiple signatures.

**Beginner-Friendly Explanation.** A constructor is **the setup routine for a new object**. When you write `new User("Ada")`, the constructor runs with `"Ada"` as its argument. It puts the object into a valid starting state. TypeScript checks that you provide the right arguments and that all required fields are initialized.

### Purposes

- **To initialize instance fields** to valid starting values.
- **To validate constructor arguments** before the object is used.
- **To enforce required initialization** under `strictPropertyInitialization`.
- **To support inheritance** by calling `super()` and passing required arguments.

### Syntax Rules and Structure

**General syntax:**

```
constructor(params) {
  // initialization
}
```

**Component breakdown:**

- `constructor` — the keyword; cannot be renamed.
- `params` — parameter list with types, defaults, rest parameters, and optional parameter properties.
- `{ ... }` — body; must call `super(...)` first in derived classes.

**Syntax rules.**

- Constructors have **no return type annotation**.
- In a derived class, `super(...)` must be called before `this` is accessed.
- Parameter properties (`public`, `private`, `protected`, `readonly`) create and initialize fields automatically.
- Constructor overloads are declared as multiple signatures followed by one implementation.
- `private` constructors prevent direct instantiation from outside the class (useful for singletons).

**Constraints and limitations.**

- `strictPropertyInitialization` requires all fields to be initialized at declaration or in the constructor.
- Forgetting `super()` in a derived class is a compile-time error.
- Constructor logic cannot be asynchronous.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic constructor with validation

```typescript
// example_constructor.ts
class BankAccount {
  balance: number;

  constructor(public owner: string, initialBalance: number) {
    if (initialBalance < 0) {
      throw new Error("Initial balance cannot be negative");
    }
    this.balance = initialBalance;
  }
}

const account = new BankAccount("Ada", 1000);
console.log(account.owner);   // Expected: Ada
console.log(account.balance); // Expected: 1000

try {
  new BankAccount("Grace", -1);
} catch (e) {
  console.log((e as Error).message); // Expected: Initial balance cannot be negative
}
```

**Expected Output:**
```
Ada
1000
Initial balance cannot be negative
```

**Why this output occurs.** The constructor validates `initialBalance` and throws if it is negative. `owner` is a parameter property, so it is automatically declared and initialized. `balance` is assigned explicitly after validation. The `strictPropertyInitialization` check is satisfied.

---

#### Example 2: Inheritance and `super()`

```typescript
// example_constructor_inheritance.ts
class Animal {
  constructor(public name: string) {}
}

class Dog extends Animal {
  constructor(name: string, public breed: string) {
    super(name); // Must call super before accessing this.
  }

  describe(): string {
    return `${this.name} (${this.breed})`;
  }
}

const dog = new Dog("Rex", "Labrador");
console.log(dog.describe()); // Expected: Rex (Labrador)
```

**Expected Output:**
```
Rex (Labrador)
```

**Why this output occurs.** `Dog` extends `Animal`. Its constructor calls `super(name)` to initialize the inherited `name` property before initializing `breed`. The parameter property `public breed: string` creates and assigns `breed` automatically. The `describe` method accesses both inherited and own properties.

---

### Real-World Cases

- **Entity construction**: validating IDs and required fields in domain entities.
- **Singleton pattern**: `private constructor()` with a static `getInstance()`.
- **Dependency injection**: constructors accept service interfaces, enabling test doubles.

---

## 8. Access Modifiers — `public`

### Definitions

**Core Definition.** **`public`** is the default access modifier in TypeScript. A `public` member is accessible from **anywhere**: inside the class, from subclasses, and from external code.

**Technical Definition.** The `public` modifier imposes **no access restriction** during type checking. Because it is the default, explicitly writing `public` is redundant except in **parameter properties**, where it is required to create the field. Google’s TypeScript style guide recommends **not** writing `public` except for non-readonly parameter properties.

**Beginner-Friendly Explanation.** `public` means **“anyone can use this.”** It is the default, so you rarely need to write it. If a member has no modifier, it is public.

### Purposes

- **To explicitly document** that a member is part of the public API.
- **To create public parameter properties** in constructors.
- **To serve as the default visibility** for class members.

### Syntax Rules and Structure

**General syntax:**

```
public memberName: Type;
```

**Component breakdown:**

- `public` — visibility modifier.
- `memberName` — identifier.
- `: Type` — annotation.

**Syntax rules.**

- `public` is the default; omitting it has the same effect.
- `public` can be combined with `readonly` and `static`.
- In parameter properties, `public` creates a field and assigns the parameter.

**Constraints and limitations.**

- Explicit `public` is often considered redundant style; follow project conventions.
- Public members are part of the structural type and affect compatibility.

### Annotated Complete Step by Step Code Examples

#### Example 1: Public vs. default visibility

```typescript
// example_public.ts
class User {
  public name: string;      // Explicit public (redundant but legal).
  email: string;            // Implicit public (default).

  constructor(name: string, email: string) {
    this.name = name;
    this.email = email;
  }
}

const user = new User("Ada", "ada@example.com");
console.log(user.name);  // Expected: Ada
console.log(user.email); // Expected: ada@example.com
```

**Expected Output:**
```
Ada
ada@example.com
```

**Why this output occurs.** Both `name` and `email` are public (one explicit, one implicit) and are accessible from outside the class. The explicit `public` documents intent but does not change behavior.

---

### Real-World Cases

- **Public API surfaces**: explicitly marking stable public members in libraries.
- **Parameter properties**: `constructor(public readonly id: string) {}`.

---

## 9. Access Modifiers — `private`

### Definitions

**Core Definition.** **`private`** restricts member access to **within the declaring class only**. Subclasses and external code cannot access `private` members.

**Technical Definition.** The `private` modifier is enforced **at compile time** by TypeScript. It does not emit runtime access checks; the emitted JavaScript still allows access to the property. For **true runtime privacy**, ECMAScript `#` private fields must be used. Two classes with `private` members are **not structurally compatible** unless the private member originates from the same declaration, introducing nominal-like behavior.

**Beginner-Friendly Explanation.** `private` means **“only this class can touch it.”** Other code — even subclasses — cannot read or write it. TypeScript will give you an error if you try. But remember: this is a **compile-time** rule, not a runtime lock.

### Purposes

- **To encapsulate internal state** and prevent external mutation.
- **To hide implementation details** from the public API.
- **To introduce nominal identity** between classes with private members.
- **To enforce invariants** by controlling access to sensitive fields.

### Syntax Rules and Structure

**General syntax:**

```
private memberName: Type;
```

**Component breakdown:**

- `private` — visibility modifier.
- `memberName` — identifier.

**Syntax rules.**

- `private` members are accessible only within the class body.
- `private` can be combined with `readonly` and `static`.
- Private parameter properties create private fields.
- Private members affect structural compatibility: classes with private members are compatible only if the private member comes from the same declaration.

**Constraints and limitations.**

- `private` is **compile-time only**; it can be bypassed at runtime via bracket notation or `as any`.
- `private` does not prevent subclassing, but subclasses cannot access the private member.
- For true runtime privacy, use `#privateField` syntax.

### Annotated Complete Step by Step Code Examples

#### Example 1: Private field and method

```typescript
// example_private.ts
class BankAccount {
  private balance: number = 0;

  deposit(amount: number): void {
    if (amount <= 0) throw new Error("Invalid amount");
    this.balance += amount;
  }

  getBalance(): number {
    return this.balance;
  }
}

const account = new BankAccount();
account.deposit(100);
console.log(account.getBalance()); // Expected: 100

// account.balance; // Error: Property 'balance' is private.
// account.deposit(-5); // Throws at runtime.
```

**Expected Output:**
```
100
```

**Why this output occurs.** `balance` is private, so external code cannot read or write it directly. `deposit` controls mutation and validates the amount. `getBalance` provides controlled read access. The compiler enforces the private boundary.

---

#### Example 2: Private members and nominal compatibility

```typescript
// example_private_nominal.ts
class A {
  private secret = 1;
  value = 2;
}

class B {
  private secret = 1;
  value = 2;
}

const a = new A();
// const b: B = a;
// Error: Property 'secret' is private in type 'A' but not in type 'B'.
```

**Expected Output:**
```
(no output; compile-time error if uncommented)
```

**Why this output occurs.** Although `A` and `B` have identical public shapes, their `private` members originate from different declarations. TypeScript therefore considers them incompatible, exhibiting **nominal-like** behavior. Without the private members, they would be structurally compatible.

---

### Real-World Cases

- **Encapsulated domain state**: `private balance` with controlled mutation.
- **Internal helper methods**: `private validate()` hidden from the public API.
- **Nominal branding**: private members to distinguish semantically different types.

---

## 10. Access Modifiers — `protected`

### Definitions

**Core Definition.** **`protected`** allows access **within the declaring class and all subclasses**, but not from external code.

**Technical Definition.** The `protected` modifier is checked at compile time. It permits access from derived classes via `this` and `super`, but not from unrelated code. Like `private`, `protected` members introduce **nominal-like** compatibility: two classes with `protected` members are compatible only if the member comes from the same declaration.

**Beginner-Friendly Explanation.** `protected` means **“this class and its children can use it, but outsiders cannot.”** It is useful when you want subclasses to customize behavior without exposing the internals to the world.

### Purposes

- **To share implementation details with subclasses** while hiding them from external code.
- **To enable template-method patterns** where subclasses override protected hooks.
- **To maintain encapsulation** across an inheritance hierarchy.

### Syntax Rules and Structure

**General syntax:**

```
protected memberName: Type;
```

**Component breakdown:**

- `protected` — visibility modifier.
- `memberName` — identifier.

**Syntax rules.**

- `protected` members are accessible within the class and subclasses.
- A subclass can access a protected member via `this` or `super`.
- Protected members affect structural compatibility nominally.
- `protected` can be combined with `readonly` and `static`.

**Constraints and limitations.**

- Protected members are **compile-time only**.
- External code cannot access protected members, even via subclass instances.
- Protected constructors prevent direct instantiation but allow subclassing.

### Annotated Complete Step by Step Code Examples

#### Example 1: Protected member in an inheritance hierarchy

```typescript
// example_protected.ts
class Animal {
  protected energy = 100;

  protected consume(amount: number): void {
    this.energy -= amount;
  }
}

class Dog extends Animal {
  bark(): void {
    this.consume(10); // Allowed: protected access from subclass.
  }

  status(): string {
    return `Energy: ${this.energy}`;
  }
}

const dog = new Dog();
dog.bark();
console.log(dog.status()); // Expected: Energy: 90

// dog.energy;   // Error: Property 'energy' is protected.
// dog.consume(5); // Error: Property 'consume' is protected.
```

**Expected Output:**
```
Energy: 90
```

**Why this output occurs.** `energy` and `consume` are protected, so `Dog` (a subclass) can access them inside its methods. External code cannot access them directly. `bark` calls `consume(10)`, reducing energy to `90`.

---

### Real-World Cases

- **Template method pattern**: protected hooks for subclasses to override.
- **Base service classes**: protected helper methods shared by subclasses.
- **Framework base controllers**: protected lifecycle methods.

---

## 11. Access Modifiers — `readonly`

### Definitions

**Core Definition.** **`readonly`** is a modifier that prevents reassignment of a property after its initial assignment. It can be applied to fields and parameter properties.

**Technical Definition.** A `readonly` property may be assigned only in its declaration or in the constructor of the declaring class. TypeScript checks assignments at compile time and rejects reassignment. `readonly` does not perform a deep freeze; it prevents reassignment of the property binding, not mutation of the referenced object. `readonly` can be combined with `public`, `private`, `protected`, and `static`.

**Beginner-Friendly Explanation.** `readonly` means **“set it once, then leave it alone.”** You can assign it when you declare it or in the constructor, but after that, TypeScript will not let you change it. It does not make the object inside immutable; it only prevents you from pointing the property at a different value.

### Purposes

- **To enforce immutability of identity** (IDs, creation timestamps).
- **To prevent accidental reassignment** of configuration values.
- **To document intent** that a property should not change after construction.

### Syntax Rules and Structure

**General syntax:**

```
readonly memberName: Type = value;
```

**Component breakdown:**

- `readonly` — modifier.
- `memberName` — identifier.
- `: Type` — annotation.
- `= value` — optional initializer; assignment here is allowed.

**Syntax rules.**

- `readonly` fields can be assigned at declaration or in the constructor.
- `readonly` parameter properties create readonly fields.
- `readonly` can be combined with visibility modifiers and `static`.
- `readonly` does not prevent mutation of object contents (`readonly arr: number[]` still allows `arr.push(1)`).

**Constraints and limitations.**

- `readonly` is **compile-time only**; it does not enforce runtime immutability.
- `readonly` on arrays and objects is shallow; use `ReadonlyArray<T>` or `Readonly<T>` for deeper immutability.
- `readonly` cannot be applied to methods.

### Annotated Complete Step by Step Code Examples

#### Example 1: Readonly field assignment rules

```typescript
// example_readonly.ts
class Config {
  readonly host: string = "localhost";
  readonly port: number;

  constructor(port: number) {
    this.port = port; // Allowed: assignment in constructor.
  }

  // changeHost() {
  //   this.host = "example.com"; // Error: Cannot assign to 'host' because it is a read-only property.
  // }
}

const config = new Config(3000);
console.log(config.host); // Expected: localhost
console.log(config.port); // Expected: 3000
```

**Expected Output:**
```
localhost
3000
```

**Why this output occurs.** `host` is initialized at declaration and cannot be reassigned afterward. `port` is readonly but uninitialized at declaration; the constructor assigns it once. The compiler rejects any assignment outside the constructor.

---

#### Example 2: Readonly parameter properties

```typescript
// example_readonly_param.ts
class Point {
  constructor(public readonly x: number, public readonly y: number) {}
}

const p = new Point(1, 2);
console.log(p.x); // Expected: 1

// p.x = 10; // Error: Cannot assign to 'x' because it is a read-only property.
```

**Expected Output:**
```
1
```

**Why this output occurs.** The parameter properties `public readonly x` and `public readonly y` create public readonly fields and assign the constructor arguments. After construction, the fields cannot be reassigned.

---

### Real-World Cases

- **Entity IDs**: `readonly id: string` assigned once.
- **Configuration values**: `readonly apiUrl: string`.
- **Immutable value objects**: `readonly x`, `readonly y` in `Point`.

---

## 12. Parameter Properties

### Definitions

**Core Definition.** **Parameter properties** are a TypeScript-specific shorthand that allows constructor parameters prefixed with a visibility modifier (`public`, `private`, `protected`) or `readonly` to be **automatically declared and initialized as class fields** with the same name.

**Technical Definition.** The syntax `constructor(private name: string)` declares a private field `name` and assigns the constructor argument to it. The modifier before the parameter serves a dual role: it is both a **parameter declaration** and a **field declaration**. Parameter properties support all access modifiers and `readonly`. They reduce boilerplate by merging declaration, parameter, and assignment into one statement. The emitted JavaScript contains the field assignment, but the modifier itself is erased.

**Beginner-Friendly Explanation.** Parameter properties are **a shortcut**. Instead of writing a field, a constructor parameter, and an assignment, you write `constructor(private name: string)` — and TypeScript creates the private field and assigns the value automatically. It saves typing and keeps the constructor clean.

### Purposes

- **To eliminate boilerplate** of declaring fields, parameters, and assignments separately.
- **To keep constructors concise** and focused on initialization logic.
- **To make field visibility explicit** directly in the constructor signature.
- **To support readonly fields** with minimal syntax.

### Syntax Rules and Structure

**General syntax:**

```
constructor([modifier] parameterName: Type, ...) { ... }
```

**Component breakdown:**

- `modifier` — `public`, `private`, `protected`, `readonly`, or a combination (e.g., `private readonly`).
- `parameterName` — the name of both the parameter and the created field.
- `: Type` — the type annotation.

**Syntax rules.**

- Parameter properties must be declared in the **constructor parameter list**.
- Supported modifiers: `public`, `private`, `protected`, `readonly`.
- A parameter property may combine `readonly` with a visibility modifier.
- The created field is assigned the parameter value at the start of the constructor body.
- Parameter properties cannot be used in **ambient declarations** or **overload signatures**.

**Constraints and limitations.**

- Some style guides and ESLint configurations (`no-parameter-properties`) discourage parameter properties, requiring explicit field declarations instead.
- Parameter properties are erased as syntax, but the field assignment remains in the emitted JavaScript.
- Debugging can be harder because the field does not appear in the class body.

### Annotated Complete Step by Step Code Examples

#### Example 1: Parameter properties vs. explicit declarations

```typescript
// example_param_props.ts
// ❌ Verbose: explicit field + parameter + assignment.
class VerboseUser {
  private name: string;
  private age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

// ✅ Concise: parameter properties.
class ConciseUser {
  constructor(private name: string, private age: number) {}
}

const user = new ConciseUser("Ada", 36);
console.log(user); // Expected: ConciseUser { name: 'Ada', age: 36 }
```

**Expected Output:**
```
ConciseUser { name: 'Ada', age: 36 }
```

**Why this output occurs.** The `ConciseUser` constructor declares two private parameter properties, which the compiler expands to field declarations and assignments. At runtime, the instance has `name` and `age` properties set to the constructor arguments. The output is the same as the verbose version, but the source is significantly shorter.

---

#### Example 2: Readonly parameter properties with inheritance

```typescript
// example_param_props_inheritance.ts
class Entity {
  constructor(public readonly id: string) {}
}

class User extends Entity {
  constructor(id: string, public readonly email: string) {
    super(id); // Must call super before using this.
  }
}

const user = new User("u1", "ada@example.com");
console.log(user.id);    // Expected: u1
console.log(user.email); // Expected: ada@example.com
```

**Expected Output:**
```
u1
ada@example.com
```

**Why this output occurs.** `Entity` declares a public readonly `id` parameter property. `User` extends `Entity` and declares a public readonly `email` parameter property. The `super(id)` call initializes the inherited `id` field. Both fields are readonly and public, so they are accessible on the instance.

---

### Real-World Cases

- **DTOs and entities**: `constructor(public readonly id: string, public name: string) {}`.
- **Service classes**: `constructor(private readonly http: HttpClient) {}` for dependency injection.
- **Value objects**: `constructor(public readonly x: number, public readonly y: number) {}`.

---

## 13. Class Best Practices

### Definitions

**Core Definition.** **Class best practices** are guidelines for designing and using TypeScript classes that maximize **type safety, encapsulation, maintainability, and testability**, while avoiding common pitfalls such as unsafe `any` leakage, fragile inheritance hierarchies, and redundant boilerplate.

**Technical Definition.** Best practices encompass **explicit visibility modifiers**, **preference for composition over inheritance**, **dependency injection via constructor parameters**, **readonly for immutable identity**, **avoidance of `public` where redundant**, **precise type annotations on public APIs**, and **runtime privacy with `#` when true encapsulation is required**. Google’s style guide, for example, recommends never using `public` except for non-readonly parameter properties, and preferring `readonly` for properties that should not be reassigned.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep classes clean and safe. They include: always write `private` or `protected` for internal members; prefer building classes from smaller pieces (composition) rather than deep inheritance; inject dependencies through the constructor so classes are easy to test; use `readonly` for things that should never change; and avoid writing `public` because it is already the default.

### Purposes

- **To maximize encapsulation** by defaulting to the narrowest visibility.
- **To improve testability** through constructor injection and composition.
- **To prevent accidental mutation** with `readonly`.
- **To keep public APIs precise and documented** with explicit return types.
- **To avoid the pitfalls of deep inheritance** by favoring composition.

### Syntax Rules and Structure

Best practices are conventions that interact with class syntax. The core rules are:

1. **Explicit visibility.** Write `private` or `protected` for non-public members; omit `public` where it is redundant.
2. **Prefer composition over inheritance.** Build behavior by combining small objects rather than extending deep hierarchies.
3. **Constructor injection.** Accept dependencies as constructor parameters (often `private readonly`) rather than instantiating them internally.
4. **Use `readonly` for immutable identity.** Mark fields that should not change after construction as `readonly`.
5. **Type public APIs explicitly.** Annotate return types of public methods for clarity and stability.
6. **Use `#` for runtime privacy** when true encapsulation is required, rather than relying solely on TypeScript’s compile-time `private`.
7. **Avoid `any`.** Use `unknown` and narrow, or precise types.
8. **Prefer named exports over default exports** for better refactoring and auto-imports.

**Component breakdown (visibility decision tree):**

```
Should this member be accessible outside the class?
  ├── No → Should subclasses access it?
  │        ├── Yes → protected
  │        └── No  → private (or #private for runtime privacy)
  └── Yes → Is it part of the stable public API?
           ├── Yes → public (omit modifier if non-parameter property)
           └── No  → Consider making the class internal or the member private
```

**Constraints and limitations.**

- Overusing inheritance can lead to fragile base classes; composition is often safer.
- `private` is compile-time only; do not rely on it for security.
- Constructor injection increases constructor parameter counts; consider a parameter object for many dependencies.
- Explicit `public` is sometimes required by team style guides despite being redundant.

### Annotated Complete Step by Step Code Examples

#### Example 1: Composition over inheritance and constructor injection

```typescript
// example_best_composition.ts
// ✅ Composition: small, focused classes combined.
class Logger {
  log(message: string): void {
    console.log(message);
  }
}

class Database {
  query(sql: string): string[] {
    return [`result for: ${sql}`];
  }
}

class UserRepository {
  // Constructor injection with private readonly.
  constructor(
    private readonly db: Database,
    private readonly logger: Logger
  ) {}

  findAll(): string[] {
    this.logger.log("Fetching users");
    return this.db.query("SELECT * FROM users");
  }
}

// ✅ Easy to test with mocks.
const repo = new UserRepository(new Database(), new Logger());
console.log(repo.findAll()); // Expected: [ 'result for: SELECT * FROM users' ]
```

**Expected Output:**
```
Fetching users
[ 'result for: SELECT * FROM users' ]
```

**Why this output occurs.** `UserRepository` composes `Database` and `Logger` through constructor injection rather than inheriting from them. The dependencies are `private readonly`, so they cannot be reassigned. This design is testable: `Database` and `Logger` can be replaced with mocks. It avoids the fragility of a deep inheritance hierarchy.

---

#### Example 2: Runtime privacy with `#` vs. compile-time `private`

```typescript
// example_best_runtime_privacy.ts
class SecureAccount {
  #pin: number;          // True runtime privacy.
  private balance: number; // Compile-time privacy only.

  constructor(pin: number, balance: number) {
    this.#pin = pin;
    this.balance = balance;
  }

  validate(input: number): boolean {
    return this.#pin === input;
  }
}

const acc = new SecureAccount(1234, 500);
console.log(acc.validate(1234)); // Expected: true

// acc.#pin;      // SyntaxError at runtime — truly private.
// (acc as any).balance; // Allowed at runtime — TypeScript private is compile-time only.
```

**Expected Output:**
```
true
```

**Why this output occurs.** `#pin` is an ECMAScript private field, enforced at runtime. `balance` is a TypeScript `private` field, erased to a normal property at runtime. If true encapsulation is required, `#` is the correct choice; TypeScript `private` is for compile-time checking only.

---

#### Example 3: Explicit return types and readonly identity

```typescript
// example_best_return_readonly.ts
class Invoice {
  constructor(
    public readonly id: string,
    private readonly items: readonly string[]
  ) {}

  // Explicit return type on public API.
  totalItems(): number {
    return this.items.length;
  }
}

const inv = new Invoice("i1", ["a", "b"]);
console.log(inv.totalItems()); // Expected: 2

// inv.id = "i2"; // Error: Cannot assign to 'id' because it is a read-only property.
// inv.items.push("c"); // Error: Property 'push' does not exist on type 'readonly string[]'.
```

**Expected Output:**
```
2
```

**Why this output occurs.** `id` is a public readonly parameter property, so it cannot be reassigned. `items` is a private readonly `readonly string[]`, preventing both reassignment and mutation. `totalItems` has an explicit `number` return type, documenting the public API. These practices enforce immutability and clarity.

---

### Real-World Cases

- **Dependency injection in services**: `constructor(private readonly http: HttpClient, private readonly config: Config) {}`.
- **Domain entities with immutable identity**: `readonly id` and `readonly createdAt`.
- **Testable architecture**: composition and constructor injection enable mock-based testing.
- **Security-sensitive classes**: `#private` fields for PINs, passwords, and secrets.

---

## References

- TypeScript Handbook — Classes - https://www.typescriptlang.org/docs/handbook/2/classes.html
- TypeScript Handbook — Classes (Legacy, Deprecated) - https://www.typescriptlang.org/docs/handbook/classes.html
- TypeScript Documentation — TypeScript for Java/C# Programmers (Classes) - https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-oop.html
- TypeScript Documentation — TypeScript for JavaScript Programmers - https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html
- Microsoft Learn — Declare Classes and Create Instances in TypeScript - https://learn.microsoft.com/en-us/training/modules/typescript-declare-instantiate-classes/
- W3Schools — TypeScript Classes - https://www.w3schools.com/typescript/typescript_classes.php
- Refine — Essentials of TypeScript Classes - https://refine.dev/blog/typescript-classes/
- Convex — TypeScript Constructors - https://www.convex.dev/typescript/core-concepts/object-oriented-programming/typescript-constructor
- Google TypeScript Style Guide — Class Members and Visibility - https://google.github.io/styleguide/tsguide.html
- TypeScript ESLint — `no-parameter-properties` Rule - https://typescript-eslint.io/rules/parameter-properties/
- MDN — Classes (JavaScript) - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- MDN — Private Class Features - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields
- Microsoft/TypeScript Wiki — TypeScript Design Goals - https://github.com/microsoft/TypeScript/wiki/TypeScript-Design-Goals