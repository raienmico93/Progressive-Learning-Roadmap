# Abstract Classes in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** An **abstract class** is a class declared with the `abstract` modifier that **cannot be instantiated directly** and serves as a **partial blueprint** for derived classes. It may contain **abstract members**—methods and properties declared without an implementation—that concrete subclasses are **required to implement**, as well as **concrete members** with full implementations that subclasses inherit. Abstract classes occupy a middle ground between interfaces (pure contracts) and concrete classes (full implementations): they can define **state**, **constructor logic**, and **shared behavior** while still enforcing a contract on their subclasses.

**Technical Definition.** The `abstract` modifier is a TypeScript-specific compile-time construct that is **erased** during compilation. An abstract class establishes a base class whose constructor has an **abstract constructor type** that cannot be invoked directly; the compiler rejects `new AbstractClass()` with error TS2511 (“Cannot create an instance of an abstract class”). Abstract methods and properties are declared without bodies and must be implemented by every **non-abstract** subclass; if a subclass fails to do so, it must itself be declared `abstract`. Abstract classes may have constructors, which are invoked via `super()` in derived classes. They support access modifiers (`public`, `protected`, `private`) and can be generic. At runtime, the emitted JavaScript is an ordinary class whose abstract members have been removed, and no runtime enforcement of abstractness exists.

**Beginner-Friendly Explanation.** An abstract class is **an incomplete class that is not meant to be used on its own**. It is like a partially filled-in form: some fields are already filled in (the concrete methods and properties), and some are left blank (the abstract methods and properties). You cannot submit the form as-is; you must create a **subclass** that fills in the blanks. The abstract class says: “Every subclass must provide these missing pieces, but I will give you some shared code to get started.” It is TypeScript’s way of saying “this class is a base class only—do not create objects from it directly.”

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Non-instantiable** | `new AbstractClass()` is a compile-time error; only concrete subclasses can be instantiated. |
| **Partial implementation** | May contain both **abstract members** (no implementation) and **concrete members** (full implementation). |
| **Enforced contract** | Every concrete subclass must implement all inherited abstract members, or be declared `abstract` itself. |
| **State and constructors** | Can declare properties, initialize them in constructors, and share initialization logic with subclasses via `super()`. |
| **Single inheritance** | A class may extend **only one** abstract class, unlike interfaces which support multiple implementation. |
| **Runtime presence** | Unlike interfaces, abstract classes exist at runtime as JavaScript classes; abstract members are erased, but the class constructor remains. |
| **Access modifiers** | Abstract members may be `public` or `protected`; `private abstract` is illegal. |

---

### Prerequisites

- **JavaScript classes**: `class`, constructors, `extends`, `super`, and instantiation.
- **TypeScript class fundamentals**: fields, methods, constructors, and access modifiers.
- **Inheritance concepts**: base classes, derived classes, method overriding.
- **The distinction between `extends` and `implements`** : abstract classes are extended, not implemented.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support) to observe compiler behavior.

---

### Related Programming Areas

- **Object-oriented design**: polymorphism, the Template Method pattern, and the Liskov Substitution Principle.
- **Interface-driven design**: contrasting abstract classes with pure interfaces.
- **Composition over inheritance**: when abstract classes are appropriate versus when composition is safer.
- **Design patterns**: Template Method, Strategy, Factory, and Repository patterns often use abstract classes.
- **Generic programming**: abstract classes may be parameterized with type parameters (e.g., `abstract class Repository<T>`).

---

### Core Concepts / Features

The following sections cover each requested concept. Each follows the uniform structure.

---

## 1. Abstract Classes

### Definitions

**Core Definition.** An **abstract class** is a class that **cannot be instantiated** and is designed exclusively to be **extended by subclasses**. It defines a common structure—shared state, shared behavior, and required members—that all concrete subclasses must follow.

**Technical Definition.** An abstract class is declared with the `abstract` modifier before the `class` keyword. Its constructor has an **abstract constructor type**, meaning the class value cannot be invoked with `new`. Abstract classes may contain **abstract methods** (no implementation) and **abstract properties** (no initializer), which must be implemented by concrete subclasses. They may also contain fully implemented methods, properties, and constructors. The TypeScript compiler enforces that no direct instantiation occurs and that all abstract members are implemented in non-abstract subclasses. Abstract classes are **erased** at runtime—the emitted JavaScript is a normal class with abstract members removed.

**Beginner-Friendly Explanation.** An abstract class is **a class that is not finished enough to use by itself**. It is a base class that says: “I will provide some shared code, but you must fill in the missing parts in your own subclass.” You cannot create an object directly from it; you must create a subclass that completes it. It is like a recipe with some steps already written and some steps left for you to fill in.

### Purposes

- **To define a shared foundation** for a group of related classes while forcing subclasses to provide specific implementations.
- **To prevent direct instantiation** of incomplete base classes, ensuring that only fully implemented subclasses are used.
- **To share concrete implementation** (state, helper methods, constructor logic) across subclasses without duplication.
- **To enforce a contract** that guarantees all subclasses implement required methods and properties.
- **To enable polymorphism** by allowing subclasses to be treated as instances of the abstract base type.

### Syntax Rules and Structure

**General syntax:**

```
abstract class AbstractClassName {
  abstract abstractMember: Type;
  abstract abstractMethod(params: Type): ReturnType;

  concreteProperty: Type = value;

  constructor(params) {
    // shared initialization
  }

  concreteMethod(params: Type): ReturnType {
    // shared implementation
  }
}
```

**Component breakdown:**

- `abstract class` — declares the class as abstract and non-instantiable.
- `abstract abstractMember: Type;` — an abstract property; no initializer allowed.
- `abstract abstractMethod(params): ReturnType;` — an abstract method; no body allowed.
- `concreteProperty: Type = value;` — a normal property with implementation.
- `constructor(params) { ... }` — a constructor that may initialize shared state.
- `concreteMethod(params): ReturnType { ... }` — a normal method with implementation.

**Syntax rules.**

- Abstract classes **cannot be instantiated**: `new AbstractClass()` is a compile-time error.
- Every **non-abstract** subclass must implement all inherited abstract members, or the compiler reports an error.
- Abstract methods have **no body** and end with a semicolon.
- Abstract properties have **no initializer** and end with a semicolon.
- Abstract members cannot be `private`; they must be `public` or `protected`.
- Abstract classes may have constructors, which are invoked via `super()` in derived classes.
- Abstract classes may be generic: `abstract class Repository<T> { ... }`.

**Constraints and limitations.**

- A class can extend **only one** abstract class (single inheritance).
- Abstract members are **erased** at runtime; there is no runtime enforcement of abstractness.
- A subclass that does not implement all abstract members must itself be declared `abstract`.
- `abstract` cannot be combined with `static` on the same member (abstract static members are not supported).
- Abstract properties are excluded from `strictPropertyInitialization` checks because they are expected to be initialized by subclasses.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic abstract class with abstract method

```typescript
// example_abstract_basic.ts
// Step 1: Declare an abstract class.
abstract class Animal {
  // Step 2: Abstract method — no implementation.
  abstract makeSound(): void;

  // Step 3: Concrete method — shared implementation.
  move(): void {
    console.log("Moving...");
  }
}

// Step 4: Concrete subclass implements the abstract method.
class Dog extends Animal {
  makeSound(): void {
    console.log("Bark!");
  }
}

// Step 5: Use the subclass.
const dog = new Dog();
dog.makeSound(); // Expected: Bark!
dog.move();      // Expected: Moving...

// Step 6: Direct instantiation is rejected.
// const animal = new Animal();
// Error: Cannot create an instance of the abstract class 'Animal'.
```

**Setup.** Compile with `npx tsc example_abstract_basic.ts --strict`.

**Expected Output:**
```
Bark!
Moving...
```

**Why this output occurs.** `Animal` is abstract, so `new Animal()` is a compile-time error. `Dog` implements `makeSound` and inherits `move`. At runtime, the abstract class exists as a normal class, but the compiler prevents direct instantiation. The shared `move` method is reused without duplication.

---

#### Example 2: Abstract class with constructor and shared state

```typescript
// example_abstract_constructor.ts
abstract class Employee {
  constructor(public name: string) {}

  abstract getRole(): string;

  describe(): string {
    return `${this.name} is a ${this.getRole()}`;
  }
}

class Manager extends Employee {
  getRole(): string {
    return "Manager";
  }
}

class Developer extends Employee {
  getRole(): string {
    return "Developer";
  }
}

const manager = new Manager("Alice");
const developer = new Developer("Bob");

console.log(manager.describe());   // Expected: Alice is a Manager
console.log(developer.describe()); // Expected: Bob is a Developer
```

**Expected Output:**
```
Alice is a Manager
Bob is a Developer
```

**Why this output occurs.** The abstract class `Employee` initializes shared state (`name`) in its constructor. Each subclass implements `getRole`. The inherited `describe` method calls `this.getRole()`, which resolves polymorphically to each subclass’s implementation. This demonstrates how abstract classes combine **shared state** with **required specialization**.

---

### Real-World Cases

- **Payment processing**: `abstract class PaymentProcessor` with abstract `processPayment()` and concrete `logTransaction()`.
- **Data repositories**: `abstract class DataRepository<T>` with abstract `fetchById()` and `save()`, plus concrete caching logic.
- **UI components**: `abstract class Component` with abstract `mount()` and `unmount()`, and shared lifecycle logic.
- **HTTP clients**: `abstract class HttpClient` with abstract `authenticate()` and concrete `request()`.

---

## 2. `abstract` Keyword

### Definitions

**Core Definition.** The **`abstract` keyword** is a TypeScript modifier applied to a **class** or to a **member** (method or property) to declare that the entity is **incomplete** and must be completed by a subclass. On a class, it prevents direct instantiation; on a member, it declares a requirement without providing an implementation.

**Technical Definition.** `abstract` is a **compile-time-only** modifier that is erased during compilation. On a class, it assigns an **abstract constructor type** that cannot be invoked with `new`. On a method, it introduces a signature without a body; on a property, it introduces a type annotation without an initializer. Abstract members must be implemented in every **concrete** (non-abstract) subclass. The modifier cannot be applied to `static` members, and abstract members cannot be `private`. The compiler enforces all of these constraints and reports errors for violations.

**Beginner-Friendly Explanation.** The `abstract` keyword is **the marker that says “this part is not finished”** . Putting it before `class` means “do not create objects directly from this class.” Putting it before a method means “every subclass must provide this method.” It is TypeScript’s way of enforcing completeness: the compiler will not let you forget to fill in the missing pieces.

### Purposes

- **To mark a class as non-instantiable**, forcing subclassing.
- **To declare required members** that subclasses must implement.
- **To distinguish incomplete contracts** from concrete implementations in the type system.
- **To enable compile-time verification** that all abstract members are eventually implemented.

### Syntax Rules and Structure

**General syntax (abstract class):**

```
abstract class ClassName { ... }
```

**General syntax (abstract member):**

```
abstract memberName: Type;
abstract methodName(params: Type): ReturnType;
```

**Component breakdown:**

- `abstract` — the modifier.
- `class` — indicates an abstract class.
- `memberName` / `methodName` — the required member.
- `: Type` / `: ReturnType` — the member’s type.
- No initializer or body is allowed.

**Syntax rules.**

- `abstract` cannot be used on `static` members.
- `abstract` members cannot be `private`; use `public` or `protected`.
- Abstract methods end with a semicolon, not a body.
- Abstract properties end with a semicolon, not an initializer.
- A class containing abstract members must itself be declared `abstract`.

**Constraints and limitations.**

- `abstract` is erased at runtime; there is no runtime abstractness.
- A non-abstract subclass **must** implement all abstract members, or the compiler rejects it.
- Abstract properties are excluded from `strictPropertyInitialization` checks because initialization is deferred to subclasses.

### Annotated Complete Step by Step Code Examples

#### Example 1: Abstract class and abstract method

```typescript
// example_abstract_keyword.ts
// Step 1: Declare the abstract class.
abstract class Shape {
  // Step 2: Declare an abstract method.
  abstract getArea(): number;

  // Step 3: Concrete method.
  printArea(): void {
    console.log(`Area: ${this.getArea()}`);
  }
}

// Step 4: Concrete subclass must implement getArea.
class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  getArea(): number {
    return Math.PI * this.radius ** 2;
  }
}

// Step 5: Use the subclass.
const circle = new Circle(2);
circle.printArea(); // Expected: Area: 12.566370614359172
```

**Expected Output:**
```
Area: 12.566370614359172
```

**Why this output occurs.** `Shape` is abstract and declares `getArea` as abstract. `Circle` implements `getArea`. The concrete `printArea` method calls `this.getArea()`, which resolves to `Circle`’s implementation. The abstract keyword enforces that no subclass can omit `getArea`.

---

#### Example 2: Subclass that fails to implement an abstract member is rejected

```typescript
// example_abstract_missing.ts
abstract class Shape {
  abstract getArea(): number;
}

// ❌ Subclass does not implement getArea.
// class BrokenShape extends Shape {}
// Error: Non-abstract class 'BrokenShape' does not implement
//        inherited abstract member 'getArea' from class 'Shape'.

// ✅ Correct: either implement the member or declare the subclass abstract.
abstract class StillAbstract extends Shape {
  // No implementation — allowed because StillAbstract is abstract.
}

console.log("Missing implementation is rejected");
```

**Expected Output:**
```
Missing implementation is rejected
```

**Why this output occurs.** The compiler requires every non-abstract subclass to implement all inherited abstract members. `BrokenShape` omits `getArea`, so compilation fails. `StillAbstract` is itself abstract, so it may defer implementation to its own subclasses.

---

### Real-World Cases

- **Plugin architectures**: `abstract class Plugin` with abstract `activate()` and `deactivate()`.
- **Domain contracts**: `abstract class PaymentMethod` with abstract `charge()`.
- **Framework base classes**: `abstract class BaseController` with abstract `handleRequest()`.

---

## 3. Abstract Methods

### Definitions

**Core Definition.** An **abstract method** is a method declared in an abstract class **without a body** (implementation). It specifies the method’s name, parameters, and return type, and **every concrete subclass must provide an implementation**.

**Technical Definition.** An abstract method is declared with the `abstract` modifier and ends with a semicolon instead of a body. It contributes to the abstract class’s contract. The TypeScript compiler verifies that each concrete subclass implements the method with a **compatible signature**. Abstract methods may be `public` or `protected`; they cannot be `private`. They are erased at runtime. Abstract methods enable **polymorphic behavior**: the abstract class can call the method (e.g., in a concrete template method), and the subclass’s implementation is dispatched dynamically.

**Beginner-Friendly Explanation.** An abstract method is **a method that the base class says must exist, but does not write**. It is like a blank line in a form: “You must fill in how this works.” Every subclass must write the method’s body. The base class can still call the method—it trusts that the subclass will provide it.

### Purposes

- **To enforce that subclasses provide specific behavior** while allowing the base class to define the surrounding algorithm.
- **To enable the Template Method pattern**, where the base class defines the algorithm skeleton and subclasses fill in abstract steps.
- **To define a polymorphic contract** that the base class can rely on when calling the method.
- **To prevent incomplete subclasses** from being instantiated without the required behavior.

### Syntax Rules and Structure

**General syntax:**

```
abstract methodName(params: Type): ReturnType;
```

**Component breakdown:**

- `abstract` — modifier.
- `methodName` — identifier.
- `params` — parameter list with types.
- `ReturnType` — return type.
- `;` — terminates the declaration (no body).

**Syntax rules.**

- Abstract methods have **no body**; they end with a semicolon.
- They must be implemented in every concrete subclass.
- They may be `public` or `protected`; `private abstract` is illegal.
- They may be generic: `abstract map<T>(fn: (x: T) => T): T;`.
- They may be overloaded with multiple abstract signatures followed by an implementation in subclasses.

**Constraints and limitations.**

- Abstract methods cannot have an implementation in the abstract class.
- A subclass’s implementation must be signature-compatible with the abstract declaration.
- Abstract methods are erased at runtime.

### Annotated Complete Step by Step Code Examples

#### Example 1: Abstract method implemented in multiple subclasses

```typescript
// example_abstract_method.ts
abstract class PaymentProcessor {
  // Abstract method — each payment type must implement this.
  abstract processPayment(amount: number): Promise<boolean>;

  // Concrete method — shared by all payment types.
  logTransaction(amount: number): void {
    console.log(`Processing payment of $${amount}`);
  }
}

class CreditCardProcessor extends PaymentProcessor {
  async processPayment(amount: number): Promise<boolean> {
    this.logTransaction(amount);
    return true;
  }
}

class PayPalProcessor extends PaymentProcessor {
  async processPayment(amount: number): Promise<boolean> {
    this.logTransaction(amount);
    return true;
  }
}

(async () => {
  await new CreditCardProcessor().processPayment(100);
  // Expected: Processing payment of $100
  await new PayPalProcessor().processPayment(50);
  // Expected: Processing payment of $50
})();
```

**Expected Output:**
```
Processing payment of $100
Processing payment of $50
```

**Why this output occurs.** Both `CreditCardProcessor` and `PayPalProcessor` implement the abstract `processPayment` method. The shared `logTransaction` method is inherited and reused. The abstract method enforces that no payment processor can omit its processing logic.

---

#### Example 2: Abstract method in a Template Method pattern

```typescript
// example_abstract_template.ts
abstract class DataProcessor<T> {
  // Template method — defines the algorithm structure.
  async process(rawData: string): Promise<T[]> {
    if (!this.validate(rawData)) {
      throw new Error("Invalid data format");
    }
    const parsed = await this.parse(rawData);   // Abstract step
    const transformed = await this.transform(parsed); // Abstract step
    return transformed.filter((item) => this.shouldInclude(item));
  }

  protected validate(data: string): boolean {
    return data.length > 0;
  }

  // Abstract steps — subclasses must implement.
  protected abstract parse(data: string): Promise<T[]>;
  protected abstract transform(data: T[]): Promise<T[]>;

  // Hook method — subclasses may override.
  protected shouldInclude(item: T): boolean {
    return true;
  }
}

class CsvUserProcessor extends DataProcessor<{ id: string; name: string }> {
  protected async parse(data: string) {
    return data.split("\n").map((line) => {
      const [id, name] = line.split(",");
      return { id, name };
    });
  }

  protected async transform(users: { id: string; name: string }[]) {
    return users.map((u) => ({ ...u, name: u.name.toUpperCase() }));
  }
}

(async () => {
  const processor = new CsvUserProcessor();
  const result = await processor.process("1,alice\n2,bob");
  console.log(result); // Expected: [ { id: '1', name: 'ALICE' }, { id: '2', name: 'BOB' } ]
})();
```

**Expected Output:**
```
[ { id: '1', name: 'ALICE' }, { id: '2', name: 'BOB' } ]
```

**Why this output occurs.** The abstract class defines the algorithm’s skeleton in `process`. The abstract methods `parse` and `transform` are implemented by `CsvUserProcessor`. The Template Method pattern ensures that all processors follow the same validation, parsing, transformation, and filtering flow while allowing each to customize the abstract steps.

---

### Real-World Cases

- **Payment processing**: abstract `processPayment(amount)` implemented by credit card, PayPal, and bank transfer processors.
- **Data processing pipelines**: abstract `parse()` and `transform()` with a shared `process()` template method.
- **Authentication strategies**: abstract `authenticate()` implemented by API key and OAuth clients.

---

## 4. Abstract Properties

### Definitions

**Core Definition.** An **abstract property** is a property declared in an abstract class **without an initializer**. It specifies the property’s name and type, and **every concrete subclass must provide an implementation** (usually by declaring the property with a value or a getter).

**Technical Definition.** An abstract property is declared with the `abstract` modifier followed by the property name and type, ending with a semicolon (no `= initializer`). It may be a data property or a getter/setter pair. Concrete subclasses must provide a compatible property declaration or accessor. Abstract properties are **excluded** from `strictPropertyInitialization` checks because they are expected to be initialized by subclasses. They may be `public` or `protected`; `private abstract` is illegal. Like abstract methods, abstract properties are erased at runtime.

**Beginner-Friendly Explanation.** An abstract property is **a property that the base class says must exist, but does not give a value for**. It is like a blank field in a form: “Every subclass must fill in this piece of data.” The base class knows the property will exist and can use it in its methods, but the actual value comes from the subclass.

### Purposes

- **To require subclasses to provide specific data** without dictating how it is stored or computed.
- **To allow the abstract class to reference properties** that it knows subclasses will supply.
- **To support both data properties and accessors** in subclasses, providing flexibility.
- **To enforce completeness** of subclass state alongside abstract methods.

### Syntax Rules and Structure

**General syntax:**

```
abstract propertyName: Type;
```

**Component breakdown:**

- `abstract` — modifier.
- `propertyName` — identifier.
- `: Type` — the property’s type.
- `;` — terminates the declaration (no initializer).

**Syntax rules.**

- Abstract properties have **no initializer**.
- Concrete subclasses must provide a property declaration, a getter, or a getter/setter pair.
- Abstract properties may be `public` or `protected`; `private abstract` is illegal.
- Abstract properties are excluded from `strictPropertyInitialization`.
- They may be optional: `abstract propertyName?: Type;` (subclasses may omit).

**Constraints and limitations.**

- Abstract properties cannot have an initializer in the abstract class.
- A subclass must provide a compatible type; the property type must be assignable.
- Abstract properties are erased at runtime.

### Annotated Complete Step by Step Code Examples

#### Example 1: Abstract property implemented in subclass

```typescript
// example_abstract_property.ts
abstract class Person {
  // Abstract property — no value in the base class.
  abstract name: string;

  // Concrete method uses the abstract property.
  display(): void {
    console.log(this.name);
  }
}

class Employee extends Person {
  // Subclass provides the property value.
  name: string;
  empCode: number;

  constructor(name: string, code: number) {
    super();
    this.name = name;
    this.empCode = code;
  }
}

const emp = new Employee("James", 100);
emp.display(); // Expected: James
```

**Expected Output:**
```
James
```

**Why this output occurs.** `Person` declares `name` as an abstract property. `Employee` provides a concrete `name` field and assigns it in the constructor. The inherited `display` method accesses `this.name`, which is valid because the abstract class guarantees the property will exist.

---

#### Example 2: Abstract property with getter in subclass

```typescript
// example_abstract_property_getter.ts
abstract class Config {
  abstract get apiUrl(): string;

  describe(): string {
    return `API URL: ${this.apiUrl}`;
  }
}

class EnvConfig extends Config {
  get apiUrl(): string {
    return process.env.API_URL ?? "https://default.example.com";
  }
}

const config = new EnvConfig();
console.log(config.describe());
// Expected: API URL: https://default.example.com (or the env value)
```

**Expected Output (assuming no env variable):**
```
API URL: https://default.example.com
```

**Why this output occurs.** `Config` declares `apiUrl` as an abstract getter. `EnvConfig` implements it as a getter that reads an environment variable or falls back to a default. The abstract class’s concrete `describe` method can safely access `this.apiUrl` because every subclass must provide it.

---

### Real-World Cases

- **Configuration classes**: abstract `apiUrl` or `timeout` properties implemented by environment-specific subclasses.
- **Domain entities**: abstract `id` or `createdAt` properties implemented by concrete entity classes.
- **UI components**: abstract `label` or `disabled` properties implemented by button and input components.

---

## 5. Abstract Class Best Practices

### Definitions

**Core Definition.** **Abstract class best practices** are guidelines for using abstract classes in TypeScript in ways that maximize **maintainability, type safety, and architectural clarity**, while avoiding the pitfalls of deep inheritance hierarchies, leaky abstractions, and inappropriate use where interfaces or composition would be safer.

**Technical Definition.** Best practices encompass **preferring interfaces for pure contracts** and using abstract classes only when **shared implementation or state** is genuinely needed. They include **keeping hierarchies shallow**, **avoiding deep inheritance chains**, **using the Template Method pattern appropriately**, **combining interfaces with abstract classes** for maximum flexibility (e.g., an interface defines the contract, an abstract class provides shared implementation), **using `protected` for members intended for subclass use**, and **preferring composition over inheritance when the relationship is not a true “is-a”** . Abstract classes should be **stable and focused**, and abstract members should represent **genuine specialization points**, not arbitrary hooks.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep abstract classes from becoming a mess. The big rules are: use interfaces for pure contracts; use abstract classes only when you have shared code to reuse; do not build deep chains of abstract classes; keep each abstract class focused on one job; and consider composition when the relationship is “has-a” rather than “is-a”.

### Purposes

- **To choose the right abstraction tool** between interfaces, abstract classes, and composition.
- **To avoid fragile base classes** and deep inheritance hierarchies that are hard to maintain.
- **To maximize flexibility** by combining interfaces (contracts) with abstract classes (shared implementation).
- **To ensure abstract classes remain stable and focused** so subclasses are not tightly coupled to changing base classes.
- **To improve testability** by keeping abstract classes small and injectable.

### Syntax Rules and Structure

Best practices are conventions that interact with abstract class syntax. The core rules are:

1. **Prefer interfaces for pure contracts.** If there is no shared state or implementation, use an interface instead of an abstract class.
2. **Combine interfaces with abstract classes.** Define the contract with an interface, then provide shared implementation with an abstract class that implements the interface.
3. **Keep hierarchies shallow.** Avoid deep chains of abstract classes; prefer composition for behavior sharing.
4. **Use the Template Method pattern deliberately.** Abstract methods should represent genuine steps that subclasses must customize, not arbitrary extension points.
5. **Use `protected` for members intended for subclasses.** Shared helpers and state should be `protected` so they are accessible to subclasses but hidden from external code.
6. **Keep abstract classes stable.** Changes to an abstract class ripple through all subclasses; design them carefully.
7. **Prefer composition when the relationship is “has-a.”** Use abstract classes only for genuine “is-a” hierarchies where shared state or behavior justifies inheritance.

**Component breakdown (abstraction tool decision tree):**

```
Is there shared state or concrete implementation?
  ├── No → Use an interface (pure contract).
  └── Yes → Is the relationship a genuine “is-a”?
            ├── No → Use composition.
            └── Yes → Is the hierarchy shallow and focused?
                      ├── No → Refactor or use composition.
                      └── Yes → Use an abstract class.
                               └── Optionally: interface for the contract +
                                   abstract class for shared implementation.
```

**Constraints and limitations.**

- Abstract classes introduce single-inheritance coupling; interfaces offer multiple implementation.
- Deep hierarchies are difficult to refactor and test.
- Abstract classes exist at runtime, adding a small runtime footprint compared to interfaces.
- Overusing abstract classes for simple code sharing can lead to an over-engineered architecture.

### Annotated Complete Step by Step Code Examples

#### Example 1: Combining interface and abstract class

```typescript
// example_best_interface_abstract.ts
// Step 1: Define a pure contract with an interface.
interface Logger {
  log(message: string, level: string): void;
}

// Step 2: Abstract class provides shared implementation.
abstract class BaseLogger implements Logger {
  protected enabled: boolean = true;

  log(message: string, level: string): void {
    if (!this.enabled) return;
    const timestamp = new Date().toISOString();
    const formatted = this.formatMessage(message, level, timestamp);
    this.write(formatted);
  }

  // Abstract method — each logger decides where to write.
  protected abstract write(message: string): void;

  private formatMessage(
    message: string,
    level: string,
    timestamp: string
  ): string {
    return `[${timestamp}] ${level.toUpperCase()}: ${message}`;
  }
}

// Step 3: Concrete implementations only implement `write`.
class ConsoleLogger extends BaseLogger {
  protected write(message: string): void {
    console.log(message);
  }
}

class FileLogger extends BaseLogger {
  constructor(private filepath: string) {
    super();
  }

  protected write(message: string): void {
    // Write to file (simulated).
    console.log(`Writing to ${this.filepath}: ${message}`);
  }
}

const logger: Logger = new ConsoleLogger();
logger.log("System started", "info");
// Expected: [timestamp] INFO: System started
```

**Expected Output (approximate):**
```
[2025-01-15T10:30:00.000Z] INFO: System started
```

**Why this output occurs.** The `Logger` interface defines the contract. `BaseLogger` implements the interface and provides shared formatting and filtering logic. Subclasses only implement `write`, the genuine specialization point. This combination gives **type safety (interface)** , **code reuse (abstract class)** , and **flexibility (subclasses)** .

---

#### Example 2: Keeping hierarchies shallow and preferring composition

```typescript
// example_best_shallow.ts
// ❌ Deep hierarchy is hard to maintain.
// abstract class A { ... }
// abstract class B extends A { ... }
// abstract class C extends B { ... }
// class D extends C { ... }

// ✅ Shallow hierarchy: one abstract base, concrete leaves.
abstract class NotificationChannel {
  abstract send(message: string): void;

  log(message: string): void {
    console.log(`[Notification] ${message}`);
  }
}

class EmailChannel extends NotificationChannel {
  send(message: string): void {
    this.log(`Email: ${message}`);
  }
}

class SmsChannel extends NotificationChannel {
  send(message: string): void {
    this.log(`SMS: ${message}`);
  }
}

// ✅ Composition for shared behavior where inheritance is not needed.
class NotificationService {
  constructor(private channel: NotificationChannel) {}

  notify(message: string): void {
    this.channel.send(message);
  }
}

const service = new NotificationService(new EmailChannel());
service.notify("Hello"); // Expected: [Notification] Email: Hello
```

**Expected Output:**
```
[Notification] Email: Hello
```

**Why this output occurs.** The abstract hierarchy is **shallow** (one abstract base, two concrete leaves). Shared logging logic lives in the abstract class. The `NotificationService` uses **composition** (it “has-a” channel) rather than inheriting from a channel, avoiding unnecessary coupling. This follows the best practice of preferring composition when the relationship is not a true “is-a”.

---

### Real-World Cases

- **Logging frameworks**: interface `Logger` + abstract `BaseLogger` + concrete `ConsoleLogger` and `FileLogger`.
- **Data access layers**: abstract `DataRepository<T>` with shared caching and abstract `fetchById`/`save` methods.
- **UI component libraries**: abstract `Component` with shared lifecycle and abstract `mount`/`unmount`.
- **Payment systems**: abstract `PaymentProcessor` with shared logging and abstract `processPayment`.

---

## References

- TypeScript Handbook — Classes: Abstract Classes and Members - https://www.typescriptlang.org/docs/handbook/2/classes.html#abstract-classes-and-members
- Convex — Abstract Classes in TypeScript: A Simple Guide - https://www.convex.dev/typescript/core-concepts/object-oriented-programming/typescript-abstract-class
- GeeksforGeeks — What are Abstract Classes in TypeScript? - https://origin.geeksforgeeks.org/what-are-abstract-classes-in-typescript/
- Mimo — TypeScript Abstract Class: Syntax, Usage, and Examples - https://mimo.org/glossary/typescript/abstract-class
- Developer Roadmap — Abstract Classes - https://raw.githubusercontent.com/nilbuild/developer-roadmap/refs/heads/master/roadmaps/typescript/content/abstract-classes%40tZFWeWHdOUJcCEtHfXH9p.md
- Manning — Programming with Types: Elements of Object-Oriented Programming (Chapter 8) - https://www.manning.com/preview/programming-with-types/chapter-8
- TypeScript 1.6 Release Notes — Abstract Classes and Methods - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-6.html
- TypeScript Handbook — Classes (Legacy) - https://www.typescriptlang.org/docs/handbook/classes.html
- Stack Overflow — TypeScript abstract property syntax - https://stackoverflow.com/questions/33387338/typescript-abstract-property
- Stack Overflow — Abstract class vs interface in TypeScript - https://stackoverflow.com/questions/50110844/what-is-the-difference-between-interface-and-abstract-class-in-typescript