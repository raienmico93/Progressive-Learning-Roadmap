# TypeScript Polymorphism and Structural Behavior: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Polymorphism is the ability of a value to take on multiple forms—specifically, the ability to use a value of one type wherever a related type is expected. In TypeScript, polymorphism emerges from two complementary mechanisms: structural subtyping (type compatibility based on shape) and interface-driven programming (type compatibility based on declared contracts). These enable subtype polymorphism and generic polymorphism without requiring nominal inheritance.

**Technical Definition**
TypeScript's type system is structurally typed: two types are compatible if their members are compatible, regardless of declaration site or name. This means any object with the same shape as a class instance is interchangeable with it, enabling "duck typing" at the type level. Subtype polymorphism in TypeScript occurs when a value of a subtype (a type with more members) is used where a supertype (a type with fewer members) is expected. Interface-driven polymorphism generalizes this by letting unrelated classes satisfy the same interface contract. Generic polymorphism extends polymorphism to type parameters, enabling algorithms and data structures that work uniformly across many types while preserving type information.

**Beginner-Friendly Explanation**
Polymorphism means "many forms." In TypeScript, it lets you write code that works with multiple types through a common interface. There are two main flavors. Structural subtyping means TypeScript only cares about what an object *has*, not what it's called—if it walks like a duck and quacks like a duck, it's a duck. Subtype and interface polymorphism let you write a function that accepts a `Shape` and call it with a `Circle` or `Square`. Generic polymorphism lets you write a function like `identity<T>(x: T): T` that works with any type while still remembering what type you passed in. Together, these make TypeScript code flexible without sacrificing type safety.

### Key Characteristics

- **Structural typing**: Type compatibility is determined by members, not names.
- **Subtype polymorphism**: A subtype can be used wherever a supertype is expected.
- **Interface-driven polymorphism**: Unrelated classes can satisfy the same interface.
- **Generic polymorphism**: Type parameters enable algorithms that work across many types.
- **Duck typing at compile time**: Objects are compatible if they have the required shape.
- **Nominal exceptions**: `private`/`protected` members and `#private` fields introduce nominal behavior.
- **Compile-time only**: All polymorphism is erased at runtime.

### Prerequisites

- Basic knowledge of TypeScript object types and interfaces
- Familiarity with classes and inheritance
- Understanding of union types and type narrowing
- Basic familiarity with generics (type parameters)

### Related Programming Areas

- **Type Theory**: Structural subtyping, parametric polymorphism, and bounded quantification
- **Object-Oriented Programming**: Subtype polymorphism and the Liskov substitution principle
- **Functional Programming**: Parametric polymorphism and generics
- **Design Patterns**: Strategy, factory, and dependency injection patterns
- **Duck Typing**: The dynamic-language equivalent of structural subtyping

### Core Concepts / Features

1. Structural Subtyping (Why an Object Looks Like a Class Instance)
2. Subtype Polymorphism via Base-Type References
3. Interface-Driven Polymorphism
4. Introduction to Generic Polymorphism in OOP (Generic Classes and Methods)


## 1. Structural Subtyping (Why an Object Looks Like a Class Instance)

### Definitions

**Core Definition**
Structural subtyping is TypeScript's rule that two types are compatible if their members are compatible, regardless of their names or declaration sites. Any object that has the same shape as a class instance is assignable to that class's type, even if it was never created from that class.

**Technical Definition**
TypeScript's type system implements structural subtyping: a type `S` is a subtype of type `T` if every member of `T` is present in `S` with a compatible type. This is checked at compile time by comparing members (properties, methods, call signatures). The relationship is not declared—it is inferred from shape. Classes with only `public` members are compared structurally, so a plain object literal can be assigned to a class-typed variable. Classes with `private` or `protected` members introduce nominal behavior: only classes that share the same declaration of those members are compatible. JavaScript `#private` fields are also nominal and cannot be satisfied by plain objects.

**Beginner-Friendly Explanation**
Structural subtyping means TypeScript looks at the *shape* of a value, not its *name*. If you have a class called `User` with a `name` and `email`, and you create a plain object `{ name: "Alice", email: "alice@example.com" }`, TypeScript treats that plain object as a valid `User`. It doesn't matter that the object wasn't created with `new User()`—it has the same shape, so it fits. This is why TypeScript is often described as "duck typed": if it has the properties and methods a class has, TypeScript accepts it as that class type.

### Purposes

- To enable flexible interoperability between classes, interfaces, and plain objects.
- To allow plain objects to satisfy class-typed parameters without explicit conversion.
- To reduce boilerplate by avoiding unnecessary class definitions for simple data.
- To enable gradual typing of JavaScript code that creates objects anonymously.
- To support testing by allowing mock objects that match class shapes.

### Syntax Rules and Structure

**General Syntax: Structural Compatibility**

```typescript
class ClassName {
  property: Type;
  method(): ReturnType { }
}

const objectLiteral: ClassName = {
  property: value,
  method() { },
};
```

**Component Breakdown**
- `objectLiteral`: A plain object with the same shape as `ClassName`.
- The assignment is allowed because the object satisfies `ClassName`'s public members.

**General Syntax: Structural Type with Interfaces**

```typescript
interface Contract {
  method(): void;
}

class Implementation {
  method(): void { }
}

const contract: Contract = new Implementation();  // ✅ Structural
```

**Component Breakdown**
- Classes automatically satisfy interfaces they structurally match.

**Syntax Rules**

- Structural compatibility compares all public members of the target type.
- Extra properties on the source are allowed (for non-literal objects).
- Object literals are subject to excess property checking.
- `private` and `protected` members make classes nominal (not structural).
- `#private` fields are also nominal.
- Callable objects (functions) are compared structurally by signature.
- Generic classes are compared structurally after type arguments are applied.

**Constraints and Limitations**

- Excess property checking applies to direct object literals but not variables.
- Classes with `private`/`protected` members cannot be satisfied by plain objects.
- Structural typing can allow unintended compatibility between similar shapes.
- Runtime behavior may differ even when types are compatible (e.g., a plain object lacks the class's prototype methods not declared as members).
- Structural typing does not check the constructor—only the instance type.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Plain Object Satisfying a Class Type

```typescript
// Step 1: Define a class.
class Point {
  constructor(public x: number, public y: number) {}

  distanceFromOrigin(): number {
    return Math.sqrt(this.x ** 2 + this.y ** 2);
  }
}

// Step 2: Assign a plain object to a Point-typed variable.
const plainPoint: Point = {
  x: 3,
  y: 4,
  distanceFromOrigin() {
    return Math.sqrt(this.x ** 2 + this.y ** 2);
  },
};

// Step 3: Use the plain object as a Point.
console.log(plainPoint.distanceFromOrigin());  // 5

// Step 4: A Point instance and a plain object are interchangeable.
const realPoint = new Point(6, 8);
console.log(realPoint.distanceFromOrigin());  // 10

// Step 5: Structural compatibility requires all public members.
// const incomplete: Point = { x: 1 };
// ❌ Error: Property 'y' is missing in type '{ x: number; }' but required in type 'Point'.

// Step 6: Extra properties on variables are allowed.
const extraPoint = { x: 1, y: 2, z: 3, distanceFromOrigin: () => 0 };
const accepted: Point = extraPoint;  // ✅ Allowed — extra properties ignored
```

**Expected Output:**
```
5
10
```

**Why This Output Occurs:** The plain object `plainPoint` has all of `Point`'s public members (`x`, `y`, `distanceFromOrigin`), so it is assignable to `Point`. TypeScript does not require the object to be created via `new Point()`. The `extraPoint` variable has extra properties, but because it's a variable (not a fresh literal), excess property checking does not apply.

#### Example 2: Nominal Behavior with `private` Members

```typescript
// Step 1: Define a class with a private member.
class SecretHolder {
  private secret: string = "hidden";

  reveal(): string {
    return this.secret;
  }
}

// Step 2: A plain object cannot satisfy the class type.
// const fake: SecretHolder = {
//   secret: "exposed",
//   reveal() { return "exposed"; },
// };
// ❌ Error: Property 'secret' is private in type 'SecretHolder'
// but not in type '{ secret: string; reveal(): string; }'.

// Step 3: Another class with the same private member declaration is also incompatible.
class OtherSecretHolder {
  private secret: string = "other";
  reveal(): string { return this.secret; }
}

// const other: SecretHolder = new OtherSecretHolder();
// ❌ Error: Types have separate declarations of a private property 'secret'.

// Step 4: Only SecretHolder itself is assignable to SecretHolder.
const real: SecretHolder = new SecretHolder();
console.log(real.reveal());  // "hidden"
```

**Expected Output:**
```
hidden
```

**Why This Output Occurs:** The `private secret` field makes `SecretHolder` nominally typed. Plain objects and classes with separately declared private members are not structurally compatible. This is a deliberate design: private members represent implementation details that should not be interchangeable.

### Real-World Cases

**Case 1: Test Mocks**
Tests provide plain objects with the same shape as production classes, avoiding the overhead of instantiating real classes.

**Case 2: API Data**
API responses (plain JSON objects) can be typed as interface-shaped types without requiring class instantiation.

**Case 3: Configuration Objects**
Configuration objects use plain object literals that structurally satisfy configuration class types.

---

## 2. Subtype Polymorphism via Base-Type References

### Definitions

**Core Definition**
Subtype polymorphism is the ability to use a value of a subtype wherever a supertype is expected. In TypeScript, this is enabled by structural subtyping: a class or interface with more members is a subtype of one with fewer members, so instances of the subtype can be assigned to variables of the supertype.

**Technical Definition**
Subtype polymorphism in TypeScript follows the Liskov substitution principle: if `S` is a subtype of `T`, then values of type `S` can be used wherever values of type `T` are expected, without altering the correctness of the program. TypeScript implements subtype polymorphism structurally: `S` is a subtype of `T` if `S` has all of `T`'s members with compatible types. Method calls on supertype references dispatch to the subtype's implementations (virtual dispatch). Parameter types are checked bivariantly for methods and contravariantly for function properties under `strictFunctionTypes`.

**Beginner-Friendly Explanation**
Subtype polymorphism means you can treat a more specific type as a more general one. If `Dog` is a subtype of `Animal` (because `Dog` has everything `Animal` has, plus more), you can put a `Dog` where an `Animal` is expected. A function that takes an `Animal` works with a `Dog` because the `Dog` is an `Animal`. When you call a method on the `Animal` reference, TypeScript (and JavaScript) uses the actual `Dog` implementation. This lets you write code that works with `Animal` in general, without knowing about specific animals.

### Purposes

- To write functions and data structures that work with a general type while accepting specific subtypes.
- To enable virtual dispatch, where method calls resolve to the actual runtime type's implementation.
- To decouple consumers from specific implementations.
- To support extensibility: new subtypes can be added without modifying consumers.
- To implement the Liskov substitution principle in code.

### Syntax Rules and Structure

**General Syntax: Subtype Assignment**

```typescript
class Supertype { }
class Subtype extends Supertype { }

const sub: Subtype = new Subtype();
const superRef: Supertype = sub;  // ✅ Allowed — Subtype is a subtype of Supertype
```

**Component Breakdown**
- `sub`: A value of the subtype.
- `superRef`: A reference of the supertype, assigned from the subtype.
- The assignment is allowed because `Subtype` has all of `Supertype`'s members.

**General Syntax: Structural Subtyping (No Inheritance)**

```typescript
interface Animal { name: string; }
interface Dog { name: string; breed: string; }

const dog: Dog = { name: "Rex", breed: "Lab" };
const animal: Animal = dog;  // ✅ Allowed — Dog has all Animal members
```

**Component Breakdown**
- `Dog` is structurally a subtype of `Animal` even without an `extends` clause.

**Syntax Rules**

- A subtype must have all members of the supertype with compatible types.
- Extra members on the subtype are allowed.
- Method return types must be covariant (subtype's return type assignable to supertype's).
- Method parameters are bivariant; function property parameters are contravariant under `strictFunctionTypes`.
- Virtual dispatch ensures the actual type's method implementation is called.
- Subtype relationships are transitive: if `A` is a subtype of `B` and `B` is a subtype of `C`, `A` is a subtype of `C`.

**Constraints and Limitations**

- The supertype reference can only access members declared on the supertype (not subtype-specific members).
- Type narrowing is required to access subtype-specific members.
- Virtual dispatch applies to methods, not to properties (which are resolved at the reference's static type).
- Overriding methods must have compatible signatures.
- Structural subtyping can allow unintended compatibility.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Subtype Polymorphism with Classes

```typescript
// Step 1: Define a base class.
class Animal {
  constructor(public name: string) {}

  speak(): string {
    return "...";
  }

  describe(): string {
    return `${this.name} says: ${this.speak()}`;
  }
}

// Step 2: Define subtypes.
class Dog extends Animal {
  speak(): string {
    return "Woof!";
  }

  fetch(): string {
    return `${this.name} fetches the ball.`;
  }
}

class Cat extends Animal {
  speak(): string {
    return "Meow!";
  }

  purr(): string {
    return `${this.name} purrs.`;
  }
}

// Step 3: Use subtype polymorphism.
const animals: Animal[] = [
  new Dog("Rex"),
  new Cat("Whiskers"),
  new Animal("Generic"),
];

// Step 4: Virtual dispatch calls the actual subtype's speak().
animals.forEach((animal) => console.log(animal.describe()));
// "Rex says: Woof!"
// "Whiskers says: Meow!"
// "Generic says: ..."

// Step 5: Narrowing is required to access subtype-specific members.
animals.forEach((animal) => {
  if (animal instanceof Dog) {
    console.log(animal.fetch());  // ✅ Narrowed to Dog
  } else if (animal instanceof Cat) {
    console.log(animal.purr());   // ✅ Narrowed to Cat
  }
});
// "Rex fetches the ball."
// "Whiskers purrs."
```

**Expected Output:**
```
Rex says: Woof!
Whiskers says: Meow!
Generic says: ...
Rex fetches the ball.
Whiskers purrs.
```

**Why This Output Occurs:** The `animals` array is typed as `Animal[]`, but it holds `Dog`, `Cat`, and `Animal` instances. When `describe()` is called, the virtual dispatch mechanism calls the actual subtype's `speak()` method. To access `fetch()` or `purr()`, type narrowing via `instanceof` is required.

#### Example 2: Structural Subtype Polymorphism (No Inheritance)

```typescript
// Step 1: Define interfaces (no classes).
interface Named {
  name: string;
}

interface Aged {
  age: number;
}

interface Person {
  name: string;
  age: number;
  email: string;
}

// Step 2: Person is structurally a subtype of both Named and Aged.
const person: Person = { name: "Alice", age: 30, email: "alice@example.com" };

const named: Named = person;  // ✅ Person has name
const aged: Aged = person;    // ✅ Person has age

console.log(named.name);  // "Alice"
console.log(aged.age);    // 30

// Step 3: A function accepting the supertype works with any subtype.
function greet(entity: Named): string {
  return `Hello, ${entity.name}!`;
}

console.log(greet(person));                          // "Hello, Alice!"
console.log(greet({ name: "Bob" }));                 // "Hello, Bob!"
console.log(greet({ name: "Charlie", extra: true })); // "Hello, Charlie!"

// Step 4: Arrays of supertypes accept subtype values.
const namedThings: Named[] = [
  person,
  { name: "Book" },
  { name: "City", population: 100000 },
];
namedThings.forEach((thing) => console.log(thing.name));
```

**Expected Output:**
```
Alice
30
Hello, Alice!
Hello, Bob!
Hello, Charlie!
Alice
Book
City
```

**Why This Output Occurs:** `Person` structurally satisfies `Named` and `Aged` because it has the required members. The `greet` function accepts any `Named`, and objects with extra properties (variables, not literals) are allowed. This demonstrates structural subtyping without inheritance.

### Real-World Cases

**Case 1: Shape Hierarchies**
Graphics libraries use subtype polymorphism where `Shape` is the supertype and `Circle`, `Square`, and `Triangle` are subtypes with different area calculations.

**Case 2: Notification Systems**
Notification systems define a `Notifier` interface and implement `EmailNotifier`, `SmsNotifier`, and `PushNotifier` as subtypes, enabling polymorphic dispatch.

**Case 3: Payment Processing**
Payment processors use `PaymentMethod` as a supertype with `CreditCard`, `PayPal`, and `BankTransfer` as subtypes, allowing uniform processing.

---

## 3. Interface-Driven Polymorphism

### Definitions

**Core Definition**
Interface-driven polymorphism is the practice of programming against interfaces (contracts) rather than concrete classes. Any class or object that satisfies an interface's shape can be used polymorphically through that interface, regardless of its actual type or inheritance hierarchy.

**Technical Definition**
Interface-driven polymorphism decouples consumers from implementations: a function that accepts an interface type can work with any class or object that satisfies the interface, even if the implementing classes are unrelated. This is a form of structural subtyping applied at the interface level. TypeScript checks interface satisfaction structurally, so a class does not need to explicitly declare `implements InterfaceName` for its instances to satisfy the interface—though explicit `implements` provides better documentation and error detection. Interface-driven polymorphism enables dependency injection, strategy patterns, and testability through mocks.

**Beginner-Friendly Explanation**
Interface-driven polymorphism means you write code that depends on *what something can do* rather than *what it is*. If you define a `Logger` interface with a `log` method, any class that has a `log` method can be used as a `Logger`—whether it's `ConsoleLogger`, `FileLogger`, or a test mock. Your code doesn't care about the specific class; it just calls `log`. This makes your code flexible and testable. You can swap implementations without changing the code that uses them. It's the foundation of dependency injection and the strategy pattern.

### Purposes

- To decouple consumers from concrete implementations.
- To enable dependency injection and testability (using mocks).
- To support the strategy pattern and other behavioral patterns.
- To allow unrelated classes to be used interchangeably through a shared contract.
- To enforce the interface-segregation principle (small, focused interfaces).

### Syntax Rules and Structure

**General Syntax: Interface-Driven Polymorphism**

```typescript
interface Contract {
  method(): ReturnType;
}

class ImplementationA implements Contract {
  method(): ReturnType { }
}

class ImplementationB implements Contract {
  method(): ReturnType { }
}

function useContract(contract: Contract): void {
  contract.method();
}

useContract(new ImplementationA());
useContract(new ImplementationB());
```

**Component Breakdown**
- `Contract`: The interface defining the behavior.
- `ImplementationA`/`ImplementationB`: Classes satisfying the contract.
- `useContract`: A function accepting any implementation via the interface.

**General Syntax: Structural Interface Satisfaction (No `implements`)**

```typescript
interface Contract {
  method(): void;
}

class ImplicitImplementation {
  method(): void { }
}

const contract: Contract = new ImplicitImplementation();  // ✅ Structural
```

**Component Breakdown**
- Classes satisfy interfaces structurally, even without `implements`.

**Syntax Rules**

- Interfaces define contracts that multiple classes can satisfy.
- `implements` explicitly declares satisfaction; structural matching allows implicit satisfaction.
- Functions accepting interfaces work with any satisfying class.
- Interfaces can be extended and combined.
- Optional and readonly members are part of the contract.
- Interface-driven polymorphism works with plain objects, not just classes.
- Dependencies are typically injected via constructor parameters or function arguments.

**Constraints and Limitations**

- Implicit interface satisfaction is less explicit and may reduce clarity (prefer `implements`).
- Interfaces cannot enforce runtime behavior—only compile-time shape.
- Interface changes break all implementations (use default implementations or abstract classes for partial contracts).
- Overly broad interfaces reduce flexibility (prefer interface segregation).
- Interfaces cannot have implementations (unlike abstract classes).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Dependency Injection via Interfaces

```typescript
// Step 1: Define a service interface.
interface Logger {
  log(message: string): void;
}

// Step 2: Implement the interface with different classes.
class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`[CONSOLE] ${message}`);
  }
}

class FileLogger implements Logger {
  constructor(private filename: string) {}

  log(message: string): void {
    console.log(`[FILE:${this.filename}] ${message}`);
  }
}

// Step 3: A consumer depends on the interface, not the implementation.
class UserService {
  constructor(private logger: Logger) {}

  createUser(name: string): void {
    this.logger.log(`Creating user: ${name}`);
    // ... user creation logic
  }
}

// Step 4: Inject different implementations.
const consoleService = new UserService(new ConsoleLogger());
consoleService.createUser("Alice");  // "[CONSOLE] Creating user: Alice"

const fileService = new UserService(new FileLogger("app.log"));
fileService.createUser("Bob");  // "[FILE:app.log] Creating user: Bob"

// Step 5: Mock for testing (structural satisfaction).
const mockLogger: Logger = {
  log: (message: string) => {
    // Test: capture the message without printing
  },
};
const testService = new UserService(mockLogger);
testService.createUser("Charlie");  // No output — mocked
```

**Expected Output:**
```
[CONSOLE] Creating user: Alice
[FILE:app.log] Creating user: Bob
```

**Why This Output Occurs:** The `UserService` class depends on the `Logger` interface, not on a specific logger class. Injecting `ConsoleLogger`, `FileLogger`, or a mock object all work because they all satisfy the `Logger` contract. This is interface-driven polymorphism, and it enables both flexibility and testability.

#### Example 2: Strategy Pattern with Interfaces

```typescript
// Step 1: Define a strategy interface.
interface SortStrategy<T> {
  sort(data: T[]): T[];
}

// Step 2: Implement multiple strategies.
class AscendingSort implements SortStrategy<number> {
  sort(data: number[]): number[] {
    return [...data].sort((a, b) => a - b);
  }
}

class DescendingSort implements SortStrategy<number> {
  sort(data: number[]): number[] {
    return [...data].sort((a, b) => b - a);
  }
}

// Step 3: Compose the strategy into a data processor.
class DataProcessor {
  constructor(private strategy: SortStrategy<number>) {}

  setStrategy(strategy: SortStrategy<number>): void {
    this.strategy = strategy;
  }

  process(data: number[]): number[] {
    return this.strategy.sort(data);
  }
}

// Step 4: Swap strategies at runtime.
const processor = new DataProcessor(new AscendingSort());
console.log(processor.process([3, 1, 4, 1, 5, 9, 2, 6]));
// [1, 1, 2, 3, 4, 5, 6, 9]

processor.setStrategy(new DescendingSort());
console.log(processor.process([3, 1, 4, 1, 5, 9, 2, 6]));
// [9, 6, 5, 4, 3, 2, 1, 1]
```

**Expected Output:**
```
[1, 1, 2, 3, 4, 5, 6, 9]
[9, 6, 5, 4, 3, 2, 1, 1]
```

**Why This Output Occurs:** The `SortStrategy<number>` interface defines a contract for sorting. `AscendingSort` and `DescendingSort` implement this contract. The `DataProcessor` composes a strategy and delegates sorting to it. Swapping strategies at runtime changes the behavior without changing the processor.

### Real-World Cases

**Case 1: Repository Pattern**
Repositories define interfaces (`UserRepository`, `OrderRepository`), and concrete implementations (SQL, NoSQL, in-memory) satisfy them, enabling database-agnostic code.

**Case 2: Payment Gateways**
Payment interfaces (`PaymentProcessor`) with implementations for Stripe, PayPal, and Square allow applications to swap payment providers without changing business logic.

**Case 3: Testing with Mocks**
Tests provide mock implementations of interfaces (loggers, HTTP clients, databases), enabling fast, isolated unit tests.

---

## 4. Introduction to Generic Polymorphism in OOP (Generic Classes and Methods)

### Definitions

**Core Definition**
Generic polymorphism (also called parametric polymorphism) is the ability to write code that works uniformly across many types by parameterizing it over those types. In TypeScript, generics use type parameters (`<T>`) to create reusable classes, interfaces, functions, and methods that preserve type information.

**Technical Definition**
Generic polymorphism in TypeScript is implemented through type parameters that act as placeholders for concrete types supplied at use sites. A generic class declares type parameters in angle brackets after the class name (`class Box<T> { }`), and each instance specifies the type argument (`new Box<number>()`). Generic methods declare their own type parameters (`method<U>(value: U): U`). TypeScript infers type arguments when possible, and constrains them with `extends`. Generics preserve type relationships: `Box<number>` and `Box<string>` are distinct types, even though they share the same class definition. This enables type-safe containers, algorithms, and abstractions without sacrificing type information.

**Beginner-Friendly Explanation**
Generics let you write code that works with any type while keeping the type information. Instead of writing separate classes for `NumberBox` and `StringBox`, you write one `Box<T>` class where `T` is a placeholder. When you write `new Box<number>()`, `T` becomes `number`, and TypeScript tracks that this box holds numbers. Generic methods work the same way: a function `identity<T>(x: T): T` returns exactly the type it receives. This is "generic polymorphism"—the same code works with many types, but TypeScript still knows which type you're using.

### Purposes

- To write reusable code that works with any type while preserving type information.
- To create type-safe containers, collections, and data structures.
- To implement algorithms that are independent of the specific element type.
- To enforce type relationships between parameters and return types.
- To reduce code duplication without losing type safety (unlike `any`).

### Syntax Rules and Structure

**General Syntax: Generic Class**

```typescript
class ClassName<T> {
  constructor(public value: T) {}

  getValue(): T {
    return this.value;
  }
}

const instance = new ClassName<number>(42);
```

**Component Breakdown**
- `<T>`: The type parameter declared after the class name.
- `value: T`: A member using the type parameter.
- `new ClassName<number>(42)`: The type argument `number` substituted for `T`.

**General Syntax: Generic Method**

```typescript
class ClassName {
  method<U>(value: U): U {
    return value;
  }
}
```

**Component Breakdown**
- `<U>`: The method's own type parameter.
- The method works with any type `U`.

**General Syntax: Generic Constraint**

```typescript
class Container<T extends { id: number }> {
  constructor(public items: T[]) {}
}
```

**Component Breakdown**
- `T extends { id: number }`: The type parameter must satisfy the constraint.

**General Syntax: Generic Interface**

```typescript
interface Repository<T> {
  findById(id: number): T | undefined;
  save(entity: T): void;
}

class UserRepository implements Repository<User> {
  findById(id: number): User | undefined { }
  save(user: User): void { }
}
```

**Component Breakdown**
- `Repository<T>`: A generic interface parameterized over the entity type.

**Syntax Rules**

- Type parameters are declared in angle brackets after the class, interface, or method name.
- Type arguments are supplied at use sites and can often be inferred.
- Type parameters can have constraints (`extends`) and defaults (`= DefaultType`).
- Generic classes create distinct types for each type argument: `Box<number>` ≠ `Box<string>`.
- Generic methods can have their own type parameters independent of the class.
- Type parameters are erased at runtime (no runtime type information).
- Multiple type parameters are separated by commas: `<T, U, V>`.

**Constraints and Limitations**

- Type parameters are erased at runtime; you cannot use `instanceof T` or `new T()`.
- Generic types are invariant by default (no automatic covariance/contravariance).
- Overly complex generic signatures can be hard to read and maintain.
- Generic constraints must be declared; unconstrained `T` allows any type.
- Generic classes cannot be used without type arguments unless defaults are provided.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Generic Class

```typescript
// Step 1: Define a generic class.
class Box<T> {
  private _value: T;

  constructor(value: T) {
    this._value = value;
  }

  get value(): T {
    return this._value;
  }

  map<U>(fn: (value: T) => U): Box<U> {
    return new Box(fn(this._value));
  }

  toString(): string {
    return `Box(${JSON.stringify(this._value)})`;
  }
}

// Step 2: Create boxes with different types.
const numberBox = new Box<number>(42);
const stringBox = new Box<string>("hello");
const objectBox = new Box({ name: "Alice", age: 30 });

// Step 3: Type information is preserved.
const n: number = numberBox.value;       // ✅
const s: string = stringBox.value;       // ✅
// const wrong: string = numberBox.value; // ❌ Error: number is not assignable to string.

// Step 4: Use generic methods (map).
const doubled = numberBox.map((n) => n * 2);
console.log(doubled.toString());  // "Box(84)"

const upper = stringBox.map((s) => s.toUpperCase());
console.log(upper.toString());    // "Box(\"HELLO\")"

const nameOnly = objectBox.map((o) => o.name);
console.log(nameOnly.toString()); // "Box(\"Alice\")"

// Step 5: Type inference works without explicit arguments.
const inferred = new Box(100);  // Box<number> inferred
console.log(inferred.toString());  // "Box(100)"
```

**Expected Output:**
```
Box(84)
Box("HELLO")
Box("Alice")
Box(100)
```

**Why This Output Occurs:** The `Box<T>` class is generic over its element type. Each instance (`Box<number>`, `Box<string>`, `Box<{name, age}>`) preserves its type information. The `map<U>` method transforms the box's content type from `T` to `U`, producing a new `Box<U>`. Type inference determines `T` from the constructor argument when not explicitly provided.

#### Example 2: Generic Method and Constraints

```typescript
// Step 1: Define an interface for the constraint.
interface Identifiable {
  id: number;
}

// Step 2: Define a generic repository class with a constraint.
class Repository<T extends Identifiable> {
  private items: Map<number, T> = new Map();

  save(entity: T): void {
    this.items.set(entity.id, entity);
  }

  findById(id: number): T | undefined {
    return this.items.get(id);
  }

  findAll(): T[] {
    return Array.from(this.items.values());
  }

  // Generic method with its own type parameter.
  mapAll<U>(fn: (entity: T) => U): U[] {
    return this.findAll().map(fn);
  }
}

// Step 3: Define entities.
interface User extends Identifiable {
  id: number;
  name: string;
  email: string;
}

interface Product extends Identifiable {
  id: number;
  name: string;
  price: number;
}

// Step 4: Use the repository with different entity types.
const userRepo = new Repository<User>();
userRepo.save({ id: 1, name: "Alice", email: "alice@example.com" });
userRepo.save({ id: 2, name: "Bob", email: "bob@example.com" });

const alice = userRepo.findById(1);
console.log(alice?.name);  // "Alice"

const names = userRepo.mapAll((user) => user.name);
console.log(names);  // ["Alice", "Bob"]

// Step 5: Constraints prevent using types without an id.
// const invalidRepo = new Repository<string>();
// ❌ Error: Type 'string' does not satisfy the constraint 'Identifiable'.

// Step 6: The same Repository works for Products.
const productRepo = new Repository<Product>();
productRepo.save({ id: 101, name: "Laptop", price: 999 });
const products = productRepo.mapAll((p) => `${p.name}: $${p.price}`);
console.log(products);  // ["Laptop: $999"]
```

**Expected Output:**
```
Alice
[ 'Alice', 'Bob' ]
[ 'Laptop: $999' ]
```

**Why This Output Occurs:** The `Repository<T extends Identifiable>` class constrains `T` to types with an `id` number. Both `User` and `Product` satisfy the constraint. The generic `mapAll<U>` method transforms entities to any type `U`, preserving the transformation's type. The constraint ensures that `entity.id` is always available for the internal `Map` key.

### Real-World Cases

**Case 1: Type-Safe Collections**
Custom collections (`List<T>`, `Stack<T>`, `Queue<T>`) use generics to provide type-safe storage and retrieval.

**Case 2: API Response Wrappers**
`ApiResponse<T>` generics wrap API responses with metadata while preserving the payload type.

**Case 3: Repository Pattern**
Generic repositories (`Repository<T>`) provide CRUD operations for any entity type, reducing boilerplate.

**Case 4: React Hooks**
React hooks like `useState<T>` and `useRef<T>` use generics to preserve the state/ref type across renders.

---

## References

- TypeScript Handbook: Generics — https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook: Type Compatibility — https://www.typescriptlang.org/docs/handbook/type-compatibility.html
- TypeScript Handbook: Classes (Implements Clauses, Extends Clauses) — https://www.typescriptlang.org/docs/handbook/2/classes.html
- TypeScript Handbook: Object Types (Interfaces) — https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript FAQ: Why are function parameters bivariant? — https://github.com/microsoft/TypeScript/wiki/FAQ
- TypeScript `strictFunctionTypes` Documentation — https://www.typescriptlang.org/tsconfig#strictFunctionTypes
- Effective TypeScript: Item 13 — Know the Differences Between type and interface
- Effective TypeScript: Item 34 — Prefer Composition to Inheritance
- Liskov Substitution Principle (Barbara Liskov, 1987) — https://en.wikipedia.org/wiki/Liskov_substitution_principle
- Design Patterns: Elements of Reusable Object-Oriented Software (Gang of Four)
- MDN: Classes — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- TypeScript Playground: Generics — https://www.typescriptlang.org/play/typescript/generics.ts.html
- Total TypeScript: Generics — https://www.totaltypescript.com/tutorials/beginners-typescript/09-generics
- TypeScript ESLint: no-explicit-any — https://typescript-eslint.io/rules/no-explicit-any/