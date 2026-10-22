# Inheritance in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Inheritance** is an object-oriented mechanism in which a **derived class** acquires the properties and methods of a **base class**, establishing an **“is-a” subtype relationship** between the two. In TypeScript, inheritance is expressed with the `extends` keyword and is statically checked: the compiler enforces that a derived class remains a **subtype** of its base class, respecting the base class’s contract.

**Technical Definition.** Inheritance in TypeScript is a **compile-time and runtime** mechanism. At the type level, the derived class type is a subtype of the base class type, meaning a derived instance is assignable wherever a base instance is expected. At the value level, the derived class’s prototype chain includes the base class’s prototype, so inherited members are available through prototype lookup. TypeScript adds **method overriding checks**, the **`super` keyword** for base-class access, and **abstract classes** that cannot be instantiated and may declare abstract members that derived classes must implement. All TypeScript-specific inheritance annotations are erased during compilation, but the underlying JavaScript prototype chain remains.

**Beginner-Friendly Explanation.** Inheritance is a way to **build a new class from an existing one**. The existing class is the **base class** (or parent); the new class is the **derived class** (or child). The child automatically gets everything the parent has, and can add its own features or **override** the parent’s behavior. TypeScript checks that the child respects the parent’s rules — for example, it cannot make a method stricter than the parent’s method — so you can safely use a child wherever a parent is expected.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **“Is-a” relationship** | Inheritance models an “is-a” relationship: a `Dog` is an `Animal`. |
| **Single inheritance** | A class may extend only **one** base class (unlike interfaces, which support multiple implementation). |
| **Subtype requirement** | TypeScript enforces that a derived class is always a **subtype** of its base class. |
| **Prototype-based runtime** | At runtime, inheritance is implemented via the JavaScript prototype chain. |
| **`super` access** | Derived classes access base-class constructors and methods via `super`. |
| **Overriding with checks** | Method overrides must remain compatible with the base signature; `override` and `noImplicitOverride` improve safety. |
| **Abstract base classes** | Abstract classes cannot be instantiated and may declare abstract members that derived classes must implement. |
| **Erasure of TS-only features** | `abstract`, `override`, and access modifiers are compile-time only; the emitted JavaScript uses standard `extends` and `super`. |

---

### Prerequisites

- **JavaScript classes**: `class`, `constructor`, `new`, `this`, `extends`, `super`.
- **TypeScript classes**: fields, methods, constructors, access modifiers.
- **Object-oriented concepts**: encapsulation, polymorphism, subtype relationships.
- **TypeScript’s structural typing**: classes are compared structurally unless `private` or `protected` members introduce nominal behavior.

---

### Related Programming Areas

- **Object-oriented design**: polymorphism, the Liskov Substitution Principle.
- **Design patterns**: Template Method, Strategy, Factory, Decorator.
- **Composition over inheritance**: an alternative to deep hierarchies.
- **Mixins**: a pattern for composing behavior without multiple inheritance.
- **Interface implementation**: `implements` as a contract-checking alternative to `extends`.

---

### Core Concepts / Features

The following sections cover the requested concepts. Each follows the uniform structure.

---

## 1. Inheritance

### Definitions

**Core Definition.** **Inheritance** is the mechanism by which a class (the **derived class**) acquires the members of another class (the **base class**), forming a hierarchical “is-a” relationship and enabling code reuse and polymorphism.

**Technical Definition.** In TypeScript, inheritance is established with the `extends` clause. The derived class inherits the base class’s instance members (fields, methods, accessors) and static members. The derived class type is a **subtype** of the base class type, so instances of the derived class are assignable to variables, parameters, and return types declared as the base class. TypeScript checks that overriding members remain compatible with the base contract. The runtime implements inheritance through the prototype chain: `Derived.prototype` inherits from `Base.prototype`.

**Beginner-Friendly Explanation.** Inheritance is **creating a new class based on an existing one**. The new class gets everything the old class has, and you can add or change things. It is like saying “a `Dog` is a kind of `Animal`,” so a `Dog` can do everything an `Animal` can do, plus dog-specific things.

### Purposes

- **To reuse implementation** across related classes without duplication.
- **To model “is-a” relationships** in the type system, enabling polymorphic substitution.
- **To establish a subtype hierarchy** where derived instances can be used wherever base instances are expected.
- **To share common behavior** through a base class while allowing specialized behavior in derived classes.

### Syntax Rules and Structure

**General syntax (inheritance declaration):**

```
class Derived extends Base {
  // additional or overridden members
}
```

**Component breakdown:**

- `class` — declaration keyword.
- `Derived` — the derived class name.
- `extends` — the inheritance keyword.
- `Base` — the base class (a value expression that evaluates to a constructor).
- `{ ... }` — the derived class body.

**Syntax rules.**

- A class may `extends` **exactly one** base class (single inheritance).
- The base class must be a **constructor value**; it cannot be an interface (use `implements` for interfaces).
- Derived classes inherit instance members and static members.
- A derived class may add new members and override inherited members.
- If the derived class has a constructor, it **must call `super()`** before accessing `this`.

**Constraints and limitations.**

- TypeScript does **not** support multiple inheritance of classes; use `implements` for multiple interfaces or **mixins** for behavior composition.
- Inheritance creates **tight coupling**: changes to the base class can affect all derived classes.
- Deep inheritance hierarchies are discouraged; prefer composition when flexibility is needed.
- TypeScript-specific inheritance features (`abstract`, `override`) are erased and have no runtime representation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic inheritance with method reuse

```typescript
// example_inheritance.ts
// Step 1: Define a base class.
class Animal {
  move(distanceInMeters: number = 0) {
    console.log(`Animal moved ${distanceInMeters}m.`);
  }
}

// Step 2: Define a derived class using extends.
class Dog extends Animal {
  bark() {
    console.log("Woof! Woof!");
  }
}

// Step 3: Create a Dog and use inherited and own methods.
const dog = new Dog();
dog.bark();       // Dog's own method
dog.move(10);     // Inherited from Animal
dog.bark();
```

**Expected Output:**
```
Woof! Woof!
Animal moved 10m.
Woof! Woof!
```

**Why this output occurs.** `Dog` extends `Animal`, so it inherits the `move` method. `dog.bark()` calls the dog-specific method. `dog.move(10)` calls the inherited method, which logs the distance. This demonstrates the most basic inheritance feature: classes inherit properties and methods from base classes.

---

#### Example 2: Inheritance with constructor and `super()`

```typescript
// example_inheritance_constructor.ts
class Animal {
  constructor(public name: string) {}
}

class Dog extends Animal {
  constructor(name: string, public breed: string) {
    super(name); // Must call super before using this.
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

**Why this output occurs.** `Dog` extends `Animal` and its constructor calls `super(name)` to initialize the inherited `name` property. The `breed` parameter property is initialized after `super()`. The `describe` method accesses both inherited and own properties.

---

### Real-World Cases

- **Framework base classes**: `class BaseController extends FrameworkController { ... }`.
- **Domain hierarchies**: `class SavingsAccount extends BankAccount { ... }`.
- **UI component hierarchies**: `class PrimaryButton extends Button { ... }`.

---

## 2. `extends`

### Definitions

**Core Definition.** The **`extends` keyword** is the syntactic operator that establishes an inheritance relationship between a derived class and a base class. It specifies that the derived class inherits the members of the base class.

**Technical Definition.** In TypeScript’s grammar, `class Derived extends Base { ... }` creates a class whose prototype chain includes `Base.prototype`. The `extends` clause accepts a value expression that evaluates to a constructor (or `null`). TypeScript checks that the derived class is a **subtype** of the base class and that any overridden members are compatible with the base contract. The `extends` keyword is also used in **generic constraints** (`T extends U`), but that is a distinct type-level use.

**Beginner-Friendly Explanation.** `extends` is the word that **connects a child class to its parent**. Writing `class Dog extends Animal` means “Dog is a kind of Animal and inherits everything Animal has.” It is the TypeScript/JavaScript equivalent of saying “this class extends that one.”

### Purposes

- **To declare an inheritance relationship** between two classes.
- **To inherit members** from a base class without re-declaring them.
- **To establish subtyping** so derived instances can be used as base instances.
- **To enable polymorphism** through base-class references to derived instances.

### Syntax Rules and Structure

**General syntax:**

```
class Derived extends Base {
  // body
}
```

**Component breakdown:**

- `extends` — inheritance keyword.
- `Base` — the base class constructor (a value expression).

**Syntax rules.**

- A class may extend **only one** class.
- The base class must be a **constructor value**, not an interface or type alias.
- The derived class inherits both instance and static members.
- If the derived class does not declare a constructor, the base constructor is used implicitly.
- If the derived class declares a constructor, it must call `super()` before accessing `this`.

**Constraints and limitations.**

- `extends` cannot be used with interfaces; use `implements` for interfaces.
- A class cannot extend a **sealed** or **final** class (TypeScript has no `final` keyword, but conventions may discourage extension).
- The base class must be **statically resolvable**; dynamic expressions that produce constructors are allowed but reduce type safety.

### Annotated Complete Step by Step Code Examples

#### Example 1: `extends` with implicit constructor

```typescript
// example_extends_implicit.ts
class Base {
  greet() {
    return "Hello";
  }
}

class Derived extends Base {
  // No constructor — base constructor is used implicitly.
}

const d = new Derived();
console.log(d.greet()); // Expected: Hello
```

**Expected Output:**
```
Hello
```

**Why this output occurs.** `Derived` does not declare a constructor, so it implicitly uses `Base`’s constructor. The `greet` method is inherited via the prototype chain.

---

#### Example 2: `extends` with method addition

```typescript
// example_extends_add.ts
class Shape {
  area(): number {
    return 0;
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }
}

const c = new Circle(2);
console.log(c.area().toFixed(2)); // Expected: 12.57
```

**Expected Output:**
```
12.57
```

**Why this output occurs.** `Circle` extends `Shape` and overrides `area` with its own implementation. The `super()` call is required because `Circle` declares a constructor. The overridden `area` returns the circle’s area, demonstrating both inheritance and overriding.

---

### Real-World Cases

- **Error hierarchies**: `class ValidationError extends Error { ... }`.
- **Model hierarchies**: `class Admin extends User { ... }`.
- **Framework extension points**: `class MyComponent extends React.Component { ... }`.

---

## 3. Base Class

### Definitions

**Core Definition.** A **base class** (also called **superclass** or **parent class**) is the class from which other classes inherit. It defines the common members that derived classes share.

**Technical Definition.** The base class is the class named in the `extends` clause. Its prototype becomes the prototype of the derived class. The base class’s instance type is a **supertype** of the derived class’s instance type. Base-class members are accessible to derived classes according to their access modifiers (`private` members are not accessible; `protected` members are accessible within the hierarchy).

**Beginner-Friendly Explanation.** The base class is **the parent**. It contains the shared features that all children inherit. For example, `Animal` might be the base class for `Dog`, `Cat`, and `Bird`.

### Purposes

- **To centralize shared state and behavior** across a family of related classes.
- **To define a common contract** that derived classes extend or specialize.
- **To enable polymorphic substitution** through the base type.
- **To provide a stable extension point** for framework and library design.

### Syntax Rules and Structure

**General syntax (base class declaration):**

```
class Base {
  // common members
}
```

**Component breakdown:**

- `class` — declaration keyword.
- `Base` — the base class name.
- `{ ... }` — common members inherited by derived classes.

**Syntax rules.**

- A base class may itself extend another class, forming a hierarchy.
- Base-class members may be `public`, `protected`, or `private`; `private` members are not inherited in an accessible form.
- Base-class constructors are called via `super()` in derived classes.
- Base classes may be `abstract` (see Section 7).

**Constraints and limitations.**

- Base classes should be **stable**; frequent changes can break derived classes.
- Overly broad base classes can become “god objects.”
- Base classes with `private` members introduce **nominal-like** compatibility, affecting structural typing.

### Annotated Complete Step by Step Code Examples

#### Example 1: Base class with shared and protected members

```typescript
// example_base_class.ts
class Vehicle {
  protected speed = 0;

  accelerate(amount: number): void {
    this.speed += amount;
  }

  describe(): string {
    return `Speed: ${this.speed}`;
  }
}

class Car extends Vehicle {
  honk(): void {
    console.log("Beep!");
  }
}

const car = new Car();
car.accelerate(50);
console.log(car.describe()); // Expected: Speed: 50
car.honk();                  // Expected: Beep!
```

**Expected Output:**
```
Speed: 50
Beep!
```

**Why this output occurs.** `Vehicle` is the base class with a protected `speed` field and a public `accelerate` method. `Car` extends `Vehicle` and adds `honk`. `Car` can access `speed` because it is `protected` and `Car` is a subclass. `accelerate(50)` updates the inherited field.

---

### Real-World Cases

- **`Error` as a base class**: `class AppError extends Error { ... }`.
- **`EventEmitter` as a base class**: custom emitters extend a base emitter.
- **Abstract repository bases**: `class BaseRepository<T> { ... }`.

---

## 4. Derived Class

### Definitions

**Core Definition.** A **derived class** (also called **subclass** or **child class**) is a class that inherits from a base class via `extends`. It acquires the base class’s members and may add or override members.

**Technical Definition.** The derived class’s prototype chain includes the base class’s prototype. Its instance type is a **subtype** of the base type. TypeScript checks that the derived class **does not violate the base contract**: overridden members must be compatible with the base signatures, and the derived class cannot remove members required by the base class. Derived classes may declare constructors that call `super()`.

**Beginner-Friendly Explanation.** The derived class is **the child**. It gets everything from the parent and can add its own stuff or change what the parent does. For example, `Dog` is a derived class of `Animal`; it inherits `move` and adds `bark`.

### Purposes

- **To specialize base-class behavior** for a particular subtype.
- **To add subtype-specific members** while retaining inherited members.
- **To override methods** with compatible implementations.
- **To participate in polymorphism** through the base-class type.

### Syntax Rules and Structure

**General syntax:**

```
class Derived extends Base {
  // additional members
  // overridden members
}
```

**Component breakdown:**

- `Derived` — the derived class name.
- `extends Base` — the inheritance relationship.
- `{ ... }` — the derived class body.

**Syntax rules.**

- A derived class may add **new** fields, methods, and accessors.
- A derived class may **override** inherited methods and fields.
- If the derived class declares a constructor, it must call `super()` before `this`.
- Derived classes inherit static members as well as instance members.
- TypeScript enforces that the derived class is a **subtype** of the base class.

**Constraints and limitations.**

- A derived class cannot **remove** a base-class member.
- Overridden methods must remain **compatible** with the base signature (same or wider parameter types; same or narrower return type under `strictFunctionTypes`).
- Deeply nested derived classes are discouraged.

### Annotated Complete Step by Step Code Examples

#### Example 1: Derived class adding members

```typescript
// example_derived_add.ts
class Animal {
  move() {
    console.log("Moving");
  }
}

class Bird extends Animal {
  fly() {
    console.log("Flying");
  }
}

const bird = new Bird();
bird.move(); // Inherited
bird.fly();  // Own
```

**Expected Output:**
```
Moving
Flying
```

**Why this output occurs.** `Bird` extends `Animal` and adds `fly`. It inherits `move` from `Animal`. Both methods are available on `bird`.

---

#### Example 2: Derived class overriding a method

```typescript
// example_derived_override.ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet(); // Call base implementation.
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}

const d = new Derived();
d.greet();        // Expected: Hello, world!
d.greet("reader"); // Expected: Hello, READER
```

**Expected Output:**
```
Hello, world!
Hello, READER
```

**Why this output occurs.** `Derived` overrides `greet`. When called without an argument, it delegates to `super.greet()`; with an argument, it uses its own implementation. The override is compatible with the base signature because the parameter is optional.

---

### Real-World Cases

- **Specialized services**: `class PremiumUserService extends UserService { ... }`.
- **Custom errors**: `class NotFoundError extends HttpError { ... }`.
- **UI variants**: `class IconButton extends Button { ... }`.

---

## 5. Method Overriding

### Definitions

**Core Definition.** **Method overriding** is the act of a derived class providing its own implementation of a method that is already defined in its base class, replacing the inherited behavior for instances of the derived class.

**Technical Definition.** In TypeScript, an overriding method must have a signature **compatible** with the base method: it may accept the same or **wider** parameter types (contravariance for function parameters under `strictFunctionTypes`) and must return the same or a **narrower** return type (covariance). The `override` keyword (introduced in TypeScript 4.3) explicitly marks a method as an override, and `noImplicitOverride` can require it to catch accidental “out-of-sync” overrides when base methods are renamed. The `super` keyword allows the overriding method to call the base implementation.

**Beginner-Friendly Explanation.** Method overriding is **replacing a parent’s method with a child’s version**. The child says “I want to do this differently.” But TypeScript makes sure the child’s version still fits the parent’s rules — you cannot make the method require more arguments or return a completely different type. You can optionally call the parent’s version with `super.method()`.

### Purposes

- **To customize inherited behavior** for a specific subtype.
- **To implement abstract methods** declared in an abstract base class.
- **To extend base behavior** by calling `super` and adding logic.
- **To participate in polymorphism** with subtype-specific implementations.

### Syntax Rules and Structure

**General syntax:**

```
class Derived extends Base {
  [override] methodName(params): ReturnType {
    // implementation
    // super.methodName(...) optional
  }
}
```

**Component breakdown:**

- `override` — optional keyword (required under `noImplicitOverride`) indicating the method overrides a base method.
- `methodName` — the name of the overridden method.
- `params` — parameter list; must be compatible with the base signature.
- `ReturnType` — return type; must be compatible with the base return type.
- `super.methodName(...)` — optional call to the base implementation.

**Syntax rules.**

- The overriding method must be **compatible** with the base method.
- The `override` keyword is optional by default but **required** under `noImplicitOverride`.
- `super` can be used to call the base method implementation.
- Overriding a field with a method or vice versa is generally discouraged; TypeScript may allow it but it can be confusing.

**Constraints and limitations.**

- Overriding methods must not make parameters **more restrictive** (e.g., a required parameter cannot replace an optional one).
- Overriding methods must not return a **wider** type than the base method.
- Without `noImplicitOverride`, renaming a base method can silently break overrides because the derived method becomes a **new method** rather than an override.
- `super` cannot be used in static methods to access instance methods.

### Annotated Complete Step by Step Code Examples

#### Example 1: Compatible override with `super`

```typescript
// example_override_super.ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  override greet(name?: string) {
    if (name === undefined) {
      super.greet(); // Call base implementation.
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}

const d = new Derived();
d.greet();        // Expected: Hello, world!
d.greet("reader"); // Expected: Hello, READER
```

**Expected Output:**
```
Hello, world!
Hello, READER
```

**Why this output occurs.** The `override` keyword marks `greet` as an override. The optional parameter `name` makes the signature compatible with the base `greet()` (a method with fewer parameters is assignable to one with more). `super.greet()` calls the base implementation when no name is provided.

---

#### Example 2: Incompatible override rejected

```typescript
// example_override_incompatible.ts
class Base {
  greet() {
    console.log("Hello");
  }
}

class Derived extends Base {
  // ❌ This override is incompatible: it requires a parameter.
  // greet(name: string) {
  //   console.log(`Hello, ${name}`);
  // }
  // Error: Property 'greet' in type 'Derived' is not assignable to the same property in base type 'Base'.
}

console.log("Incompatible override would be rejected");
```

**Expected Output:**
```
Incompatible override would be rejected
```

**Why this output occurs.** The base `greet()` accepts zero arguments, but the attempted override requires one argument. TypeScript rejects this because a `Derived` instance could be used as a `Base`, and calling `greet()` on it would fail at runtime.

---

#### Example 3: `noImplicitOverride` catching a renamed base method

```typescript
// example_no_implicit_override.ts
// With noImplicitOverride enabled:
class Album {
  download() {
    console.log("Downloading album");
  }
}

class SharedAlbum extends Album {
  // ❌ Without override, this is a new method, not an override.
  // download() { ... }
  // Error: This member must have an 'override' modifier because it overrides a member in the base class 'Album'.

  override download() {
    console.log("Downloading shared album");
  }
}

new SharedAlbum().download(); // Expected: Downloading shared album
```

**Expected Output:**
```
Downloading shared album
```

**Why this output occurs.** Under `noImplicitOverride`, any method that overrides a base method must include the `override` keyword. This prevents the “out-of-sync” problem where a base method is renamed and the derived method silently becomes a new method rather than an override.

---

### Real-World Cases

- **Custom error messages**: `class ValidationError extends Error { override message = "..."; }`.
- **Framework lifecycle hooks**: overriding `componentDidMount` in React class components.
- **Strategy implementations**: overriding a base `execute` method with a specific algorithm.

---

## 6. `super`

### Definitions

**Core Definition.** The **`super` keyword** provides access to the **base class** from within a derived class. It is used to call the base-class constructor (`super(...)`) and to invoke base-class methods (`super.method(...)`).

**Technical Definition.** In a derived-class constructor, `super(...)` invokes the base-class constructor and initializes the `this` binding. In a derived-class method, `super.method(...)` performs a **dynamic lookup** on the base class’s prototype, allowing the derived method to call the inherited implementation. TypeScript enforces that `super()` is called **before any reference to `this`** in a derived constructor (with limited leniency introduced in TypeScript 4.6). `super` is not a value; it is a keyword with special semantics.

**Beginner-Friendly Explanation.** `super` means **“the parent.”** In a child’s constructor, `super(args)` calls the parent’s constructor to set things up. In a child’s method, `super.method()` calls the parent’s version of that method. It lets the child reuse the parent’s behavior instead of replacing it completely.

### Purposes

- **To initialize inherited state** by calling the base-class constructor.
- **To invoke base-class methods** from an overriding method, extending rather than replacing behavior.
- **To satisfy TypeScript’s requirement** that `super()` precedes `this` access in derived constructors.

### Syntax Rules and Structure

**General syntax:**

```
super(arguments);          // Call base constructor
super.methodName(arguments); // Call base method
```

**Component breakdown:**

- `super` — keyword referring to the base class.
- `(...)` — constructor call; must occur before `this` access.
- `.methodName(...)` — base method call.

**Syntax rules.**

- `super()` must be called in a derived constructor **before** any reference to `this`.
- `super.method()` can be used in derived methods to call the base implementation.
- `super` cannot be used in **static methods** to access instance methods; it can access static methods via `super.staticMethod()`.
- `super` cannot be used outside a class body.

**Constraints and limitations.**

- Calling `super()` more than once is an error.
- `super()` cannot be called after `this` has been accessed.
- `super` property access (`super.field`) is allowed for `protected` fields but not for `private` fields.
- TypeScript 4.6+ permits some code before `super()` but still requires `super()` to occur at the top level before any `this` reference.

### Annotated Complete Step by Step Code Examples

#### Example 1: `super()` in a derived constructor

```typescript
// example_super_constructor.ts
class Base {
  constructor(public name: string) {}
}

class Derived extends Base {
  constructor(name: string, public age: number) {
    super(name); // Initialize base state first.
    // this.age is initialized by the parameter property after super().
  }
}

const d = new Derived("Ada", 36);
console.log(d.name); // Expected: Ada
console.log(d.age);  // Expected: 36
```

**Expected Output:**
```
Ada
36
```

**Why this output occurs.** `Derived` calls `super(name)` to invoke `Base`’s constructor, which initializes the inherited `name` field. The `age` parameter property is initialized after `super()`. Accessing `this.age` before `super()` would be a compile-time error.

---

#### Example 2: `super.method()` in an overriding method

```typescript
// example_super_method.ts
class Logger {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }
}

class TimestampedLogger extends Logger {
  override log(message: string): void {
    super.log(`${new Date().toISOString()}: ${message}`);
  }
}

const logger = new TimestampedLogger();
logger.log("System started");
// Expected: [LOG] 2025-...: System started
```

**Expected Output (approximate):**
```
[LOG] 2025-01-15T10:30:00.000Z: System started
```

**Why this output occurs.** `TimestampedLogger` overrides `log` and calls `super.log(...)` to reuse the base logging behavior, prepending a timestamp. This demonstrates **extending** rather than replacing base behavior.

---

### Real-World Cases

- **Custom error classes**: `super(message)` to initialize `Error.message`.
- **Lifecycle hooks**: `super.componentDidMount()` in React class components.
- **Template methods**: calling `super.execute()` before adding subclass-specific logic.

---

## 7. Abstract Classes

### Definitions

**Core Definition.** An **abstract class** is a class that **cannot be instantiated directly** and may contain **abstract members** — members declared without an implementation that derived classes must implement. Abstract classes serve as base classes that define a partial implementation and a contract for subclasses.

**Technical Definition.** The `abstract` modifier on a class prevents `new AbstractClass()` from compiling. Abstract classes may contain concrete members (with implementations) and abstract members (without implementations). Abstract methods must be implemented in every concrete derived class. Abstract properties must be initialized in derived classes. Under `strictPropertyInitialization`, abstract properties are excluded from the initialization check because they are expected to be initialized by subclasses.

**Beginner-Friendly Explanation.** An abstract class is **a blueprint that is not complete enough to use on its own**. It says “every subclass must provide these methods, but I will provide some shared code too.” You cannot create an object directly from an abstract class; you must create a concrete subclass that fills in the missing pieces.

### Purposes

- **To define a contract** that multiple derived classes must fulfill.
- **To share common implementation** while forcing subclasses to implement specific members.
- **To prevent direct instantiation** of incomplete base classes.
- **To model template-method patterns** where the base class defines the algorithm skeleton and subclasses fill in steps.

### Syntax Rules and Structure

**General syntax:**

```
abstract class AbstractBase {
  abstract abstractMethod(): ReturnType;
  abstract abstractProperty: Type;

  concreteMethod() {
    // shared implementation
  }
}
```

**Component breakdown:**

- `abstract class` — declares an abstract class.
- `abstract methodName(params): ReturnType;` — an abstract method declaration (no body).
- `abstract propertyName: Type;` — an abstract property declaration (no initializer).
- `concreteMethod()` — a regular method with implementation, inherited by subclasses.

**Syntax rules.**

- An abstract class **cannot be instantiated** directly.
- Abstract methods have **no body** and must be implemented in concrete derived classes.
- Abstract properties must be initialized in derived classes.
- Abstract classes may have constructors, which are called via `super()` in derived constructors.
- Abstract members may have `protected` or `public` visibility; `private abstract` is not allowed.

**Constraints and limitations.**

- Abstract classes cannot be `final` (TypeScript has no `final` keyword).
- A derived class that does not implement all abstract members must itself be declared `abstract`.
- Abstract members are **erased** at runtime; they exist only for type checking.
- Abstract classes contribute to the **nominal-like** behavior of `private` and `protected` members.

### Annotated Complete Step by Step Code Examples

#### Example 1: Abstract class with abstract method

```typescript
// example_abstract.ts
abstract class Shape {
  abstract area(): number; // Must be implemented by subclasses.

  describe(): string {
    return `Area: ${this.area()}`;
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }
}

const circle = new Circle(2);
console.log(circle.describe()); // Expected: Area: 12.566370614359172

// const shape = new Shape(); // Error: Cannot create an instance of an abstract class.
```

**Expected Output:**
```
Area: 12.566370614359172
```

**Why this output occurs.** `Shape` is abstract and declares `area` as abstract. `Circle` implements `area`. `describe` is a concrete method inherited by `Circle`; it calls `this.area()`, which resolves to `Circle`’s implementation. Direct instantiation of `Shape` is rejected at compile time.

---

#### Example 2: Abstract class with abstract property

```typescript
// example_abstract_property.ts
abstract class Vehicle {
  abstract wheels: number;

  describe(): string {
    return `This vehicle has ${this.wheels} wheels.`;
  }
}

class Car extends Vehicle {
  wheels = 4;
}

class Bike extends Vehicle {
  wheels = 2;
}

console.log(new Car().describe());  // Expected: This vehicle has 4 wheels.
console.log(new Bike().describe()); // Expected: This vehicle has 2 wheels.
```

**Expected Output:**
```
This vehicle has 4 wheels.
This vehicle has 2 wheels.
```

**Why this output occurs.** `Vehicle` declares `wheels` as an abstract property. `Car` and `Bike` initialize it with concrete values. The inherited `describe` method accesses `this.wheels`, which is provided by the subclass. Abstract properties are excluded from `strictPropertyInitialization` checks because they are expected to be initialized in subclasses.

---

### Real-World Cases

- **Template method pattern**: `abstract class DataProcessor { abstract parse(); process() { ... } }`.
- **Framework base classes**: `abstract class BaseController { abstract handleRequest(); }`.
- **Domain contracts**: `abstract class PaymentMethod { abstract charge(amount: number): void; }`.

---

## 8. `abstract`

### Definitions

**Core Definition.** The **`abstract` keyword** is a modifier applied to a **class** or a **member** (method or property) to indicate that it is incomplete and must be completed by a derived class. An abstract class cannot be instantiated; an abstract member has no implementation.

**Technical Definition.** `abstract` is a TypeScript-only modifier that affects **static checking** and is **erased** during compilation. On a class, it prevents direct instantiation. On a method, it declares a signature without a body; derived concrete classes must provide an implementation. On a property, it declares the property type without an initializer; derived classes must initialize it. Abstract members may have access modifiers (`public`, `protected`) but not `private`.

**Beginner-Friendly Explanation.** `abstract` means **“not finished yet.”** An abstract class is a class with missing pieces; you cannot use it directly. An abstract method is a method with no body; the child class must fill in the body. It is TypeScript’s way of saying “subclasses must implement this.”

### Purposes

- **To mark a class as non-instantiable** while still allowing inheritance.
- **To declare members that subclasses must implement**, enforcing a contract.
- **To combine shared implementation with required subclass customization**.
- **To support the template-method pattern** with abstract steps.

### Syntax Rules and Structure

**General syntax:**

```
abstract class AbstractClass {
  abstract abstractMethod(params): ReturnType;
  abstract abstractProperty: Type;
}
```

**Component breakdown:**

- `abstract class` — non-instantiable class.
- `abstract method` — method declaration without a body.
- `abstract property` — property declaration without an initializer.

**Syntax rules.**

- `abstract` classes cannot be instantiated.
- `abstract` methods have no body and must be implemented by concrete subclasses.
- `abstract` properties must be initialized by concrete subclasses.
- `abstract` cannot be used on **static** members.
- `abstract` members may have `public` or `protected` visibility; `private abstract` is not allowed.

**Constraints and limitations.**

- `abstract` is erased at runtime; it has no runtime effect.
- A subclass that does not implement all abstract members must also be `abstract`.
- `abstract` cannot be combined with `final` (TypeScript has no `final`).
- Abstract properties are excluded from `strictPropertyInitialization` checks.

### Annotated Complete Step by Step Code Examples

#### Example 1: Abstract class with abstract and concrete members

```typescript
// example_abstract_members.ts
abstract class Repository<T> {
  abstract findById(id: string): T | undefined;

  findMany(ids: string[]): (T | undefined)[] {
    return ids.map((id) => this.findById(id));
  }
}

class UserRepository extends Repository<{ id: string; name: string }> {
  private users: { id: string; name: string }[] = [
    { id: "u1", name: "Ada" },
    { id: "u2", name: "Grace" },
  ];

  findById(id: string) {
    return this.users.find((u) => u.id === id);
  }
}

const repo = new UserRepository();
console.log(repo.findMany(["u1", "u3"]).map((u) => u?.name));
// Expected: [ 'Ada', undefined ]
```

**Expected Output:**
```
[ 'Ada', undefined ]
```

**Why this output occurs.** `Repository` declares `findById` as abstract and provides a concrete `findMany` that calls it. `UserRepository` implements `findById`. The inherited `findMany` uses the concrete implementation. This demonstrates the **template-method pattern** with an abstract step.

---

#### Example 2: Abstract class cannot be instantiated

```typescript
// example_abstract_instantiation.ts
abstract class Shape {
  abstract area(): number;
}

// const s = new Shape();
// Error: Cannot create an instance of an abstract class.

class Square extends Shape {
  constructor(private side: number) {
    super();
  }

  area(): number {
    return this.side ** 2;
  }
}

console.log(new Square(3).area()); // Expected: 9
```

**Expected Output:**
```
9
```

**Why this output occurs.** Direct instantiation of `Shape` is a compile-time error because it is abstract. `Square` implements `area` and can be instantiated. The abstract class serves purely as a contract and partial implementation.

---

### Real-World Cases

- **Plugin architectures**: `abstract class Plugin { abstract activate(): void; }`.
- **Data access layers**: `abstract class BaseRepository<T> { abstract find(id: string): T; }`.
- **Game entities**: `abstract class Entity { abstract update(dt: number): void; }`.

---

## 9. Inheritance Best Practices

### Definitions

**Core Definition.** **Inheritance best practices** are guidelines for using class inheritance in TypeScript in ways that maximize **maintainability, type safety, and flexibility**, while avoiding the pitfalls of tight coupling, deep hierarchies, and fragile base classes.

**Technical Definition.** Best practices encompass **favoring composition over inheritance** when the relationship is not a genuine “is-a,” **keeping base classes stable and narrowly focused**, **avoiding deep inheritance chains**, **using `abstract` to define contracts**, **using `override` and `noImplicitOverride` to keep overrides synchronized**, **preferring `implements` for interface contracts**, and **respecting the Liskov Substitution Principle** so that derived instances can always substitute for base instances without breaking behavior.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep inheritance from becoming a mess. The biggest rule is: **favor composition over inheritance**. Only use inheritance when there is a true “is-a” relationship. Keep parent classes small and stable. Do not build deep chains of classes. Use `override` so you know when you are replacing a parent method. And remember that inheritance creates a tight bond — changing the parent can break all the children.

### Purposes

- **To prevent tight coupling** between base classes and derived classes.
- **To keep hierarchies shallow and understandable**, reducing debugging and maintenance costs.
- **To ensure derived classes remain valid substitutes** for base classes (Liskov Substitution Principle).
- **To catch override mistakes early** with `override` and `noImplicitOverride`.
- **To choose the right tool** — inheritance for “is-a,” composition for “has-a” and flexibility.

### Syntax Rules and Structure

Best practices are conventions that interact with inheritance syntax. The core rules are:

1. **Favor composition over inheritance.** Use inheritance only for genuine “is-a” relationships where the derived class is a true subtype.
2. **Keep base classes stable and narrow.** A base class should have a focused, well-defined contract.
3. **Avoid deep inheritance hierarchies.** Deep chains are hard to understand and maintain.
4. **Use `abstract` for contracts.** Declare abstract classes and members to force subclasses to implement required behavior.
5. **Use `override` and enable `noImplicitOverride`.** Explicitly mark overrides to catch renamed base methods.
6. **Respect the Liskov Substitution Principle.** A derived class must be usable anywhere its base class is expected without breaking behavior.
7. **Prefer `implements` for interface contracts.** Use `implements` when you only need to guarantee a shape, not inherit implementation.
8. **Consider mixins for behavior composition.** When multiple inheritance-like behavior is needed, use TypeScript mixins instead of deep hierarchies.

**Component breakdown (inheritance decision tree):**

```
Is there a genuine “is-a” relationship?
  ├── No → Use composition (has-a) or implements (contract).
  └── Yes → Is the base class stable and narrow?
            ├── No → Refactor the base class or use composition.
            └── Yes → Will the hierarchy remain shallow (2–3 levels)?
                      ├── No → Consider composition or mixins.
                      └── Yes → Inheritance is appropriate.
```

**Constraints and limitations.**

- Composition is not a universal replacement; inheritance is still appropriate for genuine subtype relationships.
- Deep hierarchies are sometimes unavoidable, but should be documented and kept as shallow as possible.
- `noImplicitOverride` may require updating existing codebases with explicit `override` keywords.
- Abstract classes and `implements` serve different purposes: abstract classes share implementation; interfaces define pure contracts.

### Annotated Complete Step by Step Code Examples

#### Example 1: Composition over inheritance for flexible behavior

```typescript
// example_best_composition.ts
// ❌ Inheritance would tightly couple these behaviors.
// class Logger { log() { ... } }
// class DatabaseLogger extends Logger { ... } // Tight coupling.

// ✅ Composition: inject the behavior.
class ConsoleLogger {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }
}

class UserService {
  constructor(private readonly logger: ConsoleLogger) {}

  createUser(name: string): void {
    this.logger.log(`Creating user: ${name}`);
  }
}

const service = new UserService(new ConsoleLogger());
service.createUser("Ada"); // Expected: [LOG] Creating user: Ada
```

**Expected Output:**
```
[LOG] Creating user: Ada
```

**Why this output occurs.** `UserService` **has-a** `ConsoleLogger` rather than **is-a** `ConsoleLogger`. Composition allows the logger to be swapped at runtime (e.g., for a file logger or a mock in tests) without changing the class hierarchy. This is more flexible than inheritance.

---

#### Example 2: Shallow hierarchy with abstract contract and `override`

```typescript
// example_best_hierarchy.ts
abstract class PaymentMethod {
  abstract charge(amount: number): void;

  process(amount: number): void {
    console.log(`Processing payment of $${amount}`);
    this.charge(amount);
  }
}

class CreditCard extends PaymentMethod {
  override charge(amount: number): void {
    console.log(`Charged $${amount} to credit card`);
  }
}

class PayPal extends PaymentMethod {
  override charge(amount: number): void {
    console.log(`Charged $${amount} via PayPal`);
  }
}

const payments: PaymentMethod[] = [new CreditCard(), new PayPal()];
payments.forEach((p) => p.process(100));
// Expected:
// Processing payment of $100
// Charged $100 to credit card
// Processing payment of $100
// Charged $100 via PayPal
```

**Expected Output:**
```
Processing payment of $100
Charged $100 to credit card
Processing payment of $100
Charged $100 via PayPal
```

**Why this output occurs.** The hierarchy is **shallow** (one abstract base, two concrete leaves). The abstract class defines the contract and shared `process` logic. Both concrete classes use `override` to explicitly implement `charge`. This satisfies the Liskov Substitution Principle: both `CreditCard` and `PayPal` can be used as `PaymentMethod` without breaking behavior.

---

### Real-World Cases

- **API clients**: composition of `HttpClient` rather than extending a base client.
- **UI components**: composition of behavior (hooks, render props) over deep component inheritance chains.
- **Domain models**: shallow hierarchies with abstract contracts for payment methods, shipping strategies, or notification channels.
- **Code review checklists**: enforcing “is-a” justifications for new `extends` relationships and preferring `implements` for pure contracts.

---

## References

- TypeScript Handbook — Classes (Inheritance, `extends`, `super`, Overriding) - https://www.typescriptlang.org/docs/handbook/2/classes.html
- TypeScript Handbook — Classes (Legacy, Abstract Classes) - https://www.typescriptlang.org/docs/handbook/classes.html
- TSConfig — `noImplicitOverride` - https://www.typescriptlang.org/tsconfig/noImplicitOverride.html
- TypeScript 4.6 Release Notes — Allowing Code in Constructors Before `super()` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-6.html
- TypeScript for Java/C# Programmers — Inheritance and Subtyping - https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-oop.html
- TypeScript for JavaScript Programmers — Classes and Inheritance - https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html
- Convex — TypeScript Inheritance Guide (Abstract Classes and Composition) - https://www.convex.dev/typescript/core-concepts/object-oriented-programming/typescript-inheritance
- EpicWeb — Composition vs Inheritance (Object-Oriented TypeScript) - https://github.com/epicweb-dev/object-oriented-typescript/blob/main/exercises/04.composition-vs-inheritance/FINISHED.mdx
- CodeSignal — Implementing Inheritance Wisely with TypeScript (Best Practices) - https://codesignal.com/learn/courses/implementing-inheritance-wisely-with-typescript
- Google TypeScript Style Guide — Classes and Inheritance - https://google.github.io/styleguide/tsguide.html
- MDN — Classes (`extends`, `super`) - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- Microsoft/TypeScript Wiki — TypeScript Design Goals (Soundness and Erasure) - https://github.com/microsoft/TypeScript/wiki/TypeScript-Design-Goals
- TypeScript Handbook — Mixins (Alternative to Multiple Inheritance) - https://www.typescriptlang.org/docs/handbook/mixins.html