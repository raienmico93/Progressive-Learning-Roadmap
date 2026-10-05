# TypeScript Inheritance and Realization: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Inheritance and realization are two fundamental mechanisms for building type hierarchies in TypeScript. Inheritance (`extends`) allows a class to derive from another class, inheriting its members and behavior. Realization (`implements`) allows a class to commit to an interface contract, promising to provide all specified members. Together, they enable code reuse, polymorphism, and contract-based design.

**Technical Definition**
In TypeScript, class inheritance uses the `extends` clause to create a subtype relationship between a derived (child) class and a base (parent) class. The child inherits all non-private members (fields, methods, accessors) from the parent and may override methods (with the `override` keyword under `noImplicitOverride`). The `super` keyword provides access to the parent class's constructor (`super()`) and methods (`super.method()`). Abstract classes (declared with `abstract`) cannot be instantiated and may contain abstract members that subclasses must implement. Interface realization uses the `implements` clause to assert that a class satisfies one or more interfaces. Unlike `extends`, `implements` creates no inheritance relationship—it is a contract check only. TypeScript allows a class to extend one class and implement multiple interfaces simultaneously.

**Beginner-Friendly Explanation**
Inheritance lets one class build on another. If you have a `Vehicle` class, you can create a `Car` class that extends it—`Car` gets all of `Vehicle`'s features and can add its own. Realization lets a class promise to follow an interface's contract. If you have a `Serializable` interface, a class can `implement Serializable` to promise it has all the required methods. Abstract classes are like blueprints that can't be used directly—they define common behavior and leave some methods for subclasses to fill in. Together, these features let you organize related classes and enforce contracts.

### Key Characteristics

- **Single class inheritance**: A class can extend only one parent class.
- **Multiple interface realization**: A class can implement multiple interfaces.
- **Method overriding**: Subclasses can replace parent method implementations.
- **`override` keyword**: Enforces that a method actually overrides a parent method.
- **`super` access**: Child classes can call parent constructors and methods.
- **Abstract classes**: Cannot be instantiated; may declare abstract members.
- **Abstract members**: Must be implemented by concrete subclasses.
- **Polymorphism**: Instances of subclasses are assignable to parent types.

### Prerequisites

- Basic knowledge of TypeScript classes
- Familiarity with interfaces and class members
- Understanding of access modifiers (`public`, `private`, `protected`)
- Familiarity with the `this` type and method binding

### Related Programming Areas

- **Object-Oriented Programming**: Inheritance and polymorphism are core OOP concepts
- **Design Patterns**: Template method, strategy, and factory patterns use inheritance
- **SOLID Principles**: Liskov substitution, open/closed, and interface segregation principles
- **Interface-Based Design**: Realization enables contract-based programming
- **Nominal Typing**: `private` and `protected` members introduce nominal behavior

### Core Concepts / Features

1. Class Extension (`extends`)
2. Parent and Child Class Mechanics
3. Method Overriding and the `override` Keyword
4. `Super` Calls (`super()` and `super.method()`)
5. Interface Realization via the `implements` Clause
6. Abstract Classes and Abstract Members (Properties and Methods)


## 1. Class Extension (`extends`)

### Definitions

**Core Definition**
Class extension is the mechanism by which a class (the child or derived class) inherits members from another class (the parent or base class) using the `extends` keyword. The child class gains all accessible members of the parent and can add new members or override inherited ones.

**Technical Definition**
The `extends` clause in a class declaration creates a subtype relationship between the child class and the parent class. The child class's instances are assignable to the parent class's type. The child inherits all `public` and `protected` members (fields, methods, accessors) from the parent but not `private` members (which remain inaccessible). The child's constructor must call `super()` before accessing `this`. TypeScript enforces that method overrides are compatible with the parent's signatures. A class can extend only one class (single inheritance), but the inheritance chain can be arbitrarily deep.

**Beginner-Friendly Explanation**
Class extension lets one class build on another. If you have a `Vehicle` class with a `move()` method, you can create a `Car` class that `extends Vehicle`. The `Car` class automatically has the `move()` method and can add its own methods like `honk()`. The `Car` is a `Vehicle`—you can use a `Car` wherever a `Vehicle` is expected. This is the foundation of inheritance and polymorphism in object-oriented programming.

### Purposes

- To reuse common properties and methods across related classes.
- To create type hierarchies that model "is-a" relationships.
- To enable polymorphism through subtype assignment.
- To share implementation while allowing specialization.
- To organize code into logical, extensible structures.

### Syntax Rules and Structure

**General Syntax: Class Extension**

```typescript
class ParentClass {
  parentProperty: Type;
  parentMethod(): ReturnType { }
}

class ChildClass extends ParentClass {
  childProperty: Type;
  childMethod(): ReturnType { }
}
```

**Component Breakdown**
- `extends ParentClass`: The clause indicating the parent class.
- The child inherits all accessible parent members.
- The child can add new members and override inherited methods.

**General Syntax: Multi-Level Inheritance**

```typescript
class GrandParent { }
class Parent extends GrandParent { }
class Child extends Parent { }
```

**Component Breakdown**
- Inheritance chains can be arbitrarily deep.

**Syntax Rules**

- A class can extend only one parent class.
- The child inherits all `public` and `protected` members.
- `private` members are not inherited (they remain inaccessible to the child).
- The child must call `super()` in its constructor if the parent has a constructor with required parameters.
- The child's type is assignable to the parent's type (polymorphism).
- The parent's type is not assignable to the child's type.
- The child can override parent methods with compatible signatures.
- The child can add new members not present in the parent.

**Constraints and Limitations**

- Single inheritance: a class cannot extend multiple classes.
- `private` members are not accessible in child classes.
- Overriding methods must have compatible signatures (return type covariance, parameter contravariance under `strictFunctionTypes` for function properties).
- The parent class must be defined before the child class is referenced (or imported).
- Deep inheritance chains can become difficult to maintain (favor composition in such cases).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Class Extension

```typescript
// Step 1: Define a parent class.
class Vehicle {
  brand: string;
  speed: number = 0;

  constructor(brand: string) {
    this.brand = brand;
    console.log(`Vehicle ${brand} created`);
  }

  move(): void {
    console.log(`${this.brand} is moving at ${this.speed} km/h`);
  }

  accelerate(amount: number): void {
    this.speed += amount;
    console.log(`Speed increased to ${this.speed} km/h`);
  }
}

// Step 2: Define a child class that extends Vehicle.
class Car extends Vehicle {
  doors: number;

  constructor(brand: string, doors: number) {
    super(brand);  // Call parent constructor
    this.doors = doors;
    console.log(`Car with ${doors} doors created`);
  }

  honk(): void {
    console.log(`${this.brand} honks: Beep beep!`);
  }
}

// Step 3: Create a Car instance.
const car = new Car("Toyota", 4);
// Output:
// "Vehicle Toyota created"
// "Car with 4 doors created"

// Step 4: Use inherited and own methods.
car.accelerate(50);  // "Speed increased to 50 km/h"
car.move();          // "Toyota is moving at 50 km/h"
car.honk();          // "Toyota honks: Beep beep!"

// Step 5: Polymorphism — Car is a Vehicle.
const vehicle: Vehicle = car;  // ✅ Allowed
vehicle.move();  // "Toyota is moving at 50 km/h"

// Step 6: Extra properties on child are not accessible via parent type.
// vehicle.doors;  // ❌ Error: Property 'doors' does not exist on type 'Vehicle'.
```

**Expected Output:**
```
Vehicle Toyota created
Car with 4 doors created
Speed increased to 50 km/h
Toyota is moving at 50 km/h
Toyota honks: Beep beep!
Toyota is moving at 50 km/h
```

**Why This Output Occurs:** The `Car` class extends `Vehicle`, inheriting `brand`, `speed`, `move()`, and `accelerate()`. The `Car` constructor calls `super(brand)` to initialize the parent's `brand` property. The `Car` adds a `doors` property and `honk()` method. A `Car` instance is assignable to `Vehicle` because it has all of `Vehicle`'s members.

#### Example 2: Multi-Level Inheritance

```typescript
// Step 1: Define a grandparent class.
class Animal {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  eat(): void {
    console.log(`${this.name} is eating.`);
  }
}

// Step 2: Define a parent class that extends Animal.
class Mammal extends Animal {
  warmBlooded: boolean = true;

  breathe(): void {
    console.log(`${this.name} is breathing.`);
  }
}

// Step 3: Define a child class that extends Mammal.
class Dog extends Mammal {
  breed: string;

  constructor(name: string, breed: string) {
    super(name);  // Calls Mammal's constructor, which calls Animal's
    this.breed = breed;
  }

  bark(): void {
    console.log(`${this.name} (${this.breed}) barks!`);
  }
}

// Step 4: Create a Dog instance.
const rex = new Dog("Rex", "German Shepherd");

// Step 5: Access members from all levels.
rex.eat();      // From Animal: "Rex is eating."
rex.breathe();  // From Mammal: "Rex is breathing."
rex.bark();     // From Dog: "Rex (German Shepherd) barks!"

// Step 6: Polymorphism works through the chain.
const animal: Animal = rex;  // ✅ Dog → Mammal → Animal
animal.eat();  // "Rex is eating."

const mammal: Mammal = rex;  // ✅ Dog → Mammal
mammal.breathe();  // "Rex is breathing."
```

**Expected Output:**
```
Rex is eating.
Rex is breathing.
Rex (German Shepherd) barks!
Rex is eating.
Rex is breathing.
```

**Why This Output Occurs:** The inheritance chain `Animal` → `Mammal` → `Dog` means `Dog` inherits from both `Mammal` and `Animal`. The `super(name)` call in `Dog`'s constructor invokes `Mammal`'s constructor, which invokes `Animal`'s constructor. Polymorphism works at every level: a `Dog` is a `Mammal` and an `Animal`.

### Real-World Cases

**Case 1: UI Component Hierarchies**
UI frameworks use class inheritance for component hierarchies: `Component` → `Button` → `PrimaryButton`, with shared rendering logic in the base class.

**Case 2: Domain Entity Hierarchies**
Domain models use inheritance for entity hierarchies: `Payment` → `CardPayment` → `CreditCardPayment`, sharing common payment logic.

**Case 3: Error Hierarchies**
Custom error classes extend `Error` to add application-specific error information while preserving standard error behavior.

---

## 2. Parent and Child Class Mechanics

### Definitions

**Core Definition**
Parent and child class mechanics describe the runtime and compile-time interactions between a base class and its derived classes. This includes member inheritance, constructor chaining, property shadowing, and the behavior of `this` throughout the inheritance hierarchy.

**Technical Definition**
When a child class extends a parent, the child's prototype chain is linked to the parent's prototype. The child inherits all `public` and `protected` members via the prototype chain. Fields declared in the child shadow same-named fields in the parent. Constructor execution order is: parent field initializers, parent constructor body, child field initializers, child constructor body (after `super()`). The `this` value in a parent method, when called on a child instance, refers to the child instance—enabling polymorphic dispatch. Static members are also inherited via the constructor function's prototype chain.

**Beginner-Friendly Explanation**
When a child class extends a parent, the child gets everything the parent has. But there are some subtle mechanics: when you create a child instance, the parent's constructor runs first (via `super()`), then the child's. Fields declared in the child can shadow same-named fields in the parent. When a parent method runs on a child instance, `this` refers to the child—so the method sees the child's data. Understanding these mechanics helps you avoid subtle bugs in inheritance hierarchies.

### Purposes

- To understand the execution order of constructors and field initializers.
- To predict how `this` behaves in inherited methods.
- To avoid field shadowing bugs between parent and child classes.
- To understand prototype chain behavior for method resolution.
- To correctly design class hierarchies with shared state and behavior.

### Syntax Rules and Structure

**General Syntax: Field Shadowing**

```typescript
class Parent {
  value: number = 10;
}

class Child extends Parent {
  value: number = 20;  // Shadows parent's value
}
```

**Component Breakdown**
- The child's `value` shadows the parent's `value`.
- Accessing `childInstance.value` returns the child's value.

**General Syntax: Constructor Execution Order**

```typescript
class Parent {
  parentField = console.log("Parent field");
  constructor() {
    console.log("Parent constructor");
  }
}

class Child extends Parent {
  childField = console.log("Child field");
  constructor() {
    super();
    console.log("Child constructor");
  }
}
// Output order:
// "Parent field"
// "Parent constructor"
// "Child field"
// "Child constructor"
```

**Component Breakdown**
- Parent field initializers run first, then parent constructor body.
- Then child field initializers, then child constructor body (after `super()`).

**Syntax Rules**

- Parent field initializers run before the parent constructor body.
- Child field initializers run after `super()` returns.
- Field shadowing: child fields with the same name as parent fields shadow the parent's.
- `this` in parent methods refers to the actual instance type (child instance).
- Static members are inherited via the constructor function's prototype chain.
- Method resolution follows the prototype chain (child methods shadow parent methods).

**Constraints and Limitations**

- Field shadowing can cause subtle bugs; use distinct names or `declare` to avoid issues.
- With `useDefineForClassFields: true`, parent fields are redefined on the child instance, which can shadow accessors.
- Parent constructors calling overridden methods can access uninitialized child fields.
- `this` in parent constructors refers to the child instance but child fields are not yet initialized.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Constructor Execution Order

```typescript
// Step 1: Define a parent class with field initializers and constructor logic.
class Parent {
  parentField = "parent-initialized";
  parentField2: string;

  constructor() {
    console.log(`Parent constructor: parentField = ${this.parentField}`);
    this.parentField2 = "parent-constructor-set";
    console.log(`Parent constructor: parentField2 = ${this.parentField2}`);
  }
}

// Step 2: Define a child class.
class Child extends Parent {
  childField = "child-initialized";

  constructor() {
    super();  // Parent field initializers and constructor run first
    console.log(`Child constructor: childField = ${this.childField}`);
    console.log(`Child constructor: parentField = ${this.parentField}`);
  }
}

// Step 3: Create a Child instance and observe the order.
const child = new Child();
// Output:
// "Parent constructor: parentField = parent-initialized"
// "Parent constructor: parentField2 = parent-constructor-set"
// "Child constructor: childField = child-initialized"
// "Child constructor: parentField = parent-initialized"
```

**Expected Output:**
```
Parent constructor: parentField = parent-initialized
Parent constructor: parentField2 = parent-constructor-set
Child constructor: childField = child-initialized
Child constructor: parentField = parent-initialized
```

**Why This Output Occurs:** The parent's field initializers run first (`parentField = "parent-initialized"`), then the parent constructor body (setting `parentField2`). Then the child's field initializers run (`childField`), then the child constructor body. This order is guaranteed by the language specification.

#### Example 2: Field Shadowing and `this` Behavior

```typescript
// Step 1: Define a parent class with a method using `this`.
class Parent {
  value: number = 10;

  getValue(): number {
    return this.value;  // `this` is the actual instance
  }

  describe(): string {
    return `Parent with value ${this.value}`;
  }
}

// Step 2: Define a child that shadows the field.
class Child extends Parent {
  value: number = 20;  // Shadows parent's value

  describe(): string {
    return `Child with value ${this.value}`;
  }
}

// Step 3: Create instances and observe behavior.
const parent = new Parent();
const child = new Child();

console.log(parent.getValue());     // 10 — parent's value
console.log(child.getValue());      // 20 — child's value (via `this`)
console.log(parent.describe());     // "Parent with value 10"
console.log(child.describe());      // "Child with value 20"

// Step 4: `this` in inherited method refers to the child instance.
// The parent's getValue() method, when called on child, sees child.value.
const parentRef: Parent = child;    // Polymorphic assignment
console.log(parentRef.getValue());  // 20 — still the child's value
console.log(parentRef.describe());  // "Child with value 20" — virtual dispatch
```

**Expected Output:**
```
10
20
Parent with value 10
Child with value 20
20
Child with value 20
```

**Why This Output Occurs:** The child's `value` field shadows the parent's. When `getValue()` is called on the child instance, `this.value` resolves to the child's `value` (20). When `describe()` is called on `parentRef` (which holds a `Child` instance), the child's `describe()` method is invoked due to virtual dispatch—even though the reference type is `Parent`.

### Real-World Cases

**Case 1: Framework Base Classes**
Framework base classes (e.g., `React.Component`) use field initialization order carefully to ensure child classes can rely on parent state.

**Case 2: Template Method Pattern**
The template method pattern relies on parent methods calling overridable child methods, requiring careful `this` and initialization order understanding.

**Case 3: Plugin Architectures**
Plugin base classes define extension points via overridable methods, and child plugins override them to customize behavior.

---

## 3. Method Overriding and the `override` Keyword

### Definitions

**Core Definition**
Method overriding is the process by which a child class provides its own implementation of a method that exists in its parent class. The `override` keyword (introduced in TypeScript 4.3) explicitly marks a method as overriding a parent method, enabling compile-time verification that the parent method exists.

**Technical Definition**
Method overriding occurs when a child class declares a method with the same name as a parent method. The child's method must have a compatible signature: the return type must be covariant (child return type assignable to parent return type), and parameters must be compatible (bivariant for method syntax, contravariant for function property syntax under `strictFunctionTypes`). The `override` keyword, when required by `noImplicitOverride`, ensures that the method actually overrides a parent member, catching typos and refactoring errors. The `override` keyword can also be used with the `super` keyword to call the parent implementation.

**Beginner-Friendly Explanation**
Method overriding means a child class can replace a parent's method with its own version. For example, if `Animal` has a `makeSound()` method, `Dog` can override it to return `"Woof"`. The `override` keyword is a safety feature: it tells TypeScript "I intend to override a parent method," and TypeScript checks that the parent method actually exists. If you typo the method name, TypeScript catches it. Without `override`, a typo would silently create a new method instead of overriding. Using `override` is a best practice for maintaining inheritance hierarchies.

### Purposes

- To customize inherited behavior for specific subclasses.
- To enforce that a method actually overrides a parent member (via `override`).
- To catch typos and refactoring errors in inheritance hierarchies.
- To enable polymorphic dispatch where the child's implementation is used.
- To combine parent behavior with child-specific logic via `super`.

### Syntax Rules and Structure

**General Syntax: Method Override**

```typescript
class Parent {
  method(): ReturnType { }
}

class Child extends Parent {
  override method(): ReturnType { }
}
```

**Component Breakdown**
- `override`: The keyword marking the method as an override.
- The child's method replaces the parent's for child instances.

**General Syntax: Override with `super`**

```typescript
class Child extends Parent {
  override method(): ReturnType {
    const parentResult = super.method();
    return transform(parentResult);
  }
}
```

**Component Breakdown**
- `super.method()`: Calls the parent's implementation.
- The child can extend or modify the parent's behavior.

**General Syntax: `noImplicitOverride` Configuration**

```json
{
  "compilerOptions": {
    "noImplicitOverride": true
  }
}
```

**Component Breakdown**
- When enabled, all overriding methods must use the `override` keyword.

**Syntax Rules**

- Overriding methods must have the same name as a parent method.
- The child's method signature must be compatible with the parent's (return type covariant).
- The `override` keyword is optional unless `noImplicitOverride` is enabled.
- `override` can be combined with access modifiers: `public override`, `protected override`.
- `super.method()` calls the parent's implementation.
- The `override` keyword applies to methods, accessors, and properties (TypeScript 4.3+).
- Overriding a method that doesn't exist in the parent is an error when using `override`.

**Constraints and Limitations**

- The `override` keyword requires TypeScript 4.3+.
- Without `noImplicitOverride`, forgetting the `override` keyword is not an error.
- Overriding private methods is not allowed (private members are not inherited).
- Return type covariance is allowed, but parameter types must be compatible.
- Overriding with an incompatible signature is a compile error.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Method Overriding

```typescript
// Step 1: Define a parent class with a method.
class Shape {
  area(): number {
    return 0;
  }

  describe(): string {
    return `Shape with area ${this.area()}`;
  }
}

// Step 2: Define a child class that overrides area().
class Circle extends Shape {
  radius: number;

  constructor(radius: number) {
    super();
    this.radius = radius;
  }

  override area(): number {
    return Math.PI * this.radius ** 2;
  }
}

// Step 3: Define another child that overrides area().
class Square extends Shape {
  side: number;

  constructor(side: number) {
    super();
    this.side = side;
  }

  override area(): number {
    return this.side ** 2;
  }
}

// Step 4: Polymorphic dispatch uses the correct override.
const shapes: Shape[] = [new Shape(), new Circle(5), new Square(4)];
shapes.forEach((shape) => console.log(shape.describe()));
// "Shape with area 0"
// "Shape with area 78.53981633974483"
// "Shape with area 16"
```

**Expected Output:**
```
Shape with area 0
Shape with area 78.53981633974483
Shape with area 16
```

**Why This Output Occurs:** The `describe()` method in `Shape` calls `this.area()`. When `describe()` is invoked on a `Circle` or `Square` instance, virtual dispatch calls the overridden `area()` method. The `override` keyword ensures that `area()` actually overrides a parent method.

#### Example 2: Override with `super` and Typo Detection

```typescript
// Step 1: Define a parent class.
class Logger {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }

  logError(message: string): void {
    console.log(`[ERROR] ${message}`);
  }
}

// Step 2: Define a child that extends the parent's behavior.
class TimestampLogger extends Logger {
  override log(message: string): void {
    const timestamp = new Date().toISOString();
    super.log(`${timestamp} ${message}`);  // Call parent implementation
  }

  // Step 3: Typo detection with override.
  // override logErrorr(message: string): void { }
  // ❌ Error: This member cannot have an 'override' modifier because
  // it is not declared in the base class 'Logger'.

  // Step 4: Without override, a typo silently creates a new method.
  logErrorTypo(message: string): void {
    console.log(`Typo method: ${message}`);
  }
}

// Step 5: Use the child logger.
const logger = new TimestampLogger();
logger.log("Application started");
// Output: "[LOG] 2024-...T...Z Application started"

logger.logError("Something failed");
// Output: "[ERROR] Something failed" (inherited from parent)
```

**Expected Output:**
```
[LOG] 2024-...T...Z Application started
[ERROR] Something failed
```

**Why This Output Occurs:** The `TimestampLogger` overrides `log` to prepend a timestamp, calling `super.log()` to use the parent's formatting. The `override` keyword ensures `log` actually overrides a parent method. If we tried to override `logErrorr` (a typo), TypeScript would produce an error because the parent has no such method.

### Real-World Cases

**Case 1: UI Component Overrides**
React class components override lifecycle methods (`componentDidMount`, `render`) to customize behavior.

**Case 2: Framework Hooks**
Testing frameworks override `setUp` and `tearDown` methods to provide test-specific initialization.

**Case 3: Domain-Specific Behavior**
Domain entities override `toString()`, `equals()`, and `hashCode()` (or their TypeScript equivalents) to provide domain-specific semantics.

---

## 4. `Super` Calls (`super()` and `super.method()`)

### Definitions

**Core Definition**
The `super` keyword provides access to the parent class from within a child class. `super()` calls the parent constructor, and `super.method()` calls a parent method. `super` is essential for constructor chaining and for extending (rather than replacing) parent behavior.

**Technical Definition**
In a derived class constructor, `super()` must be called before accessing `this`. The `super()` call invokes the parent constructor with the provided arguments, initializing the parent portion of the instance. In instance methods, `super.method(args)` invokes the parent's implementation of `method`, bypassing the child's override. `super` is not a value—it cannot be assigned to a variable or passed as an argument. In static methods, `super.method()` calls the parent class's static method. The `super` keyword is only valid inside classes with an `extends` clause.

**Beginner-Friendly Explanation**
The `super` keyword lets a child class use its parent's code. `super()` calls the parent constructor—you must do this before using `this` in a child constructor. `super.method()` calls the parent's version of a method, which is useful when you want to extend the parent's behavior rather than completely replace it. For example, a child `log()` method might call `super.log()` to use the parent's formatting, then add its own timestamp. `super` is how child classes build on parent behavior.

### Purposes

- To initialize the parent portion of an instance via `super()`.
- To call a parent method from an overriding child method.
- To extend parent behavior rather than replacing it entirely.
- To access parent static methods from child static methods.
- To enforce constructor initialization order in derived classes.

### Syntax Rules and Structure

**General Syntax: `super()` in Constructor**

```typescript
class Child extends Parent {
  constructor(args: ArgTypes) {
    super(parentArgs);  // Must be called before `this` access
    // child initialization
  }
}
```

**Component Breakdown**
- `super(parentArgs)`: Invokes the parent constructor.
- Must be the first statement (or before any `this` access) in the child constructor.

**General Syntax: `super.method()` in Methods**

```typescript
class Child extends Parent {
  method(args: ArgTypes): ReturnType {
    const parentResult = super.method(args);
    return transform(parentResult);
  }
}
```

**Component Breakdown**
- `super.method(args)`: Calls the parent's implementation of `method`.

**General Syntax: `super.method()` in Static Methods**

```typescript
class Child extends Parent {
  static staticMethod(args: ArgTypes): ReturnType {
    return super.staticMethod(args);
  }
}
```

**Component Breakdown**
- `super.staticMethod(args)`: Calls the parent's static method.

**Syntax Rules**

- `super()` must be called before `this` access in derived class constructors.
- `super()` can only be called in a constructor of a derived class.
- `super.method()` can be called in any instance method of a derived class.
- `super.staticMethod()` can be called in any static method of a derived class.
- `super` cannot be used as a value (no `const s = super`).
- `super` is only valid inside classes with an `extends` clause.
- If the child constructor does not explicitly call `super()`, TypeScript inserts an implicit `super()` call (if the parent has a no-arg constructor).

**Constraints and Limitations**

- `super()` must be called before any `this` access in the child constructor.
- `super` cannot access parent `private` members.
- `super.method()` bypasses the child's own override (calls parent version).
- `super` in static methods refers to the parent class, not an instance.
- Arrow functions inside methods capture `super` lexically.
- Calling `super()` twice in the same constructor is an error.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `super()` and `super.method()`

```typescript
// Step 1: Define a parent class.
class Employee {
  name: string;
  salary: number;

  constructor(name: string, salary: number) {
    this.name = name;
    this.salary = salary;
    console.log(`Employee ${name} created with salary ${salary}`);
  }

  getDetails(): string {
    return `${this.name} earns $${this.salary}`;
  }
}

// Step 2: Define a child class that uses super.
class Manager extends Employee {
  reports: string[];

  constructor(name: string, salary: number, reports: string[] = []) {
    super(name, salary);  // Call parent constructor
    this.reports = reports;
    console.log(`Manager ${name} manages ${reports.length} reports`);
  }

  override getDetails(): string {
    const baseDetails = super.getDetails();  // Call parent method
    return `${baseDetails} and manages ${this.reports.length} people`;
  }
}

// Step 3: Create a Manager instance.
const manager = new Manager("Alice", 100000, ["Bob", "Charlie"]);
// Output:
// "Employee Alice created with salary 100000"
// "Manager Alice manages 2 reports"

// Step 4: Use the overridden method.
console.log(manager.getDetails());
// "Alice earns $100000 and manages 2 people"

// Step 5: The parent method is still available via super internally.
const employee: Employee = manager;
console.log(employee.getDetails());  // Uses Manager's override (virtual dispatch)
// "Alice earns $100000 and manages 2 people"
```

**Expected Output:**
```
Employee Alice created with salary 100000
Manager Alice manages 2 reports
Alice earns $100000 and manages 2 people
Alice earns $100000 and manages 2 people
```

**Why This Output Occurs:** The `Manager` constructor calls `super(name, salary)`, which invokes the `Employee` constructor. The `getDetails` override calls `super.getDetails()` to get the parent's formatted string, then appends the reports count. Virtual dispatch ensures the child's version is called even through a `Employee` reference.

#### Example 2: `super` in Static Methods

```typescript
// Step 1: Define a parent class with static members.
class BaseModel {
  static tableName: string = "base";

  static find(id: number): string {
    return `SELECT * FROM ${this.tableName} WHERE id = ${id}`;
  }

  static createTable(): string {
    return `CREATE TABLE ${this.tableName}`;
  }
}

// Step 2: Define a child class that overrides static behavior.
class UserModel extends BaseModel {
  static tableName: string = "users";  // Shadows parent's static

  static find(id: number): string {
    const query = super.find(id);  // Calls parent's static find
    console.log(`Executing: ${query}`);
    return query;
  }

  static createTable(): string {
    return super.createTable() + " (with user-specific columns)";
  }
}

// Step 3: Use static methods.
console.log(BaseModel.find(1));
// "SELECT * FROM base WHERE id = 1"

console.log(UserModel.find(2));
// "Executing: SELECT * FROM users WHERE id = 2"
// "SELECT * FROM users WHERE id = 2"

console.log(UserModel.createTable());
// "CREATE TABLE users (with user-specific columns)"

// Step 4: `this` in static methods refers to the class.
console.log(BaseModel.find(3));  // Uses BaseModel.tableName ("base")
```

**Expected Output:**
```
SELECT * FROM base WHERE id = 1
Executing: SELECT * FROM users WHERE id = 2
SELECT * FROM users WHERE id = 2
CREATE TABLE users (with user-specific columns)
SELECT * FROM base WHERE id = 3
```

**Why This Output Occurs:** The `UserModel` overrides the static `tableName` property and the `find` method. `super.find(id)` calls the parent's static method, which uses `this.tableName`—and `this` refers to `UserModel` when called on `UserModel`. Static `super` calls access the parent class's static members.

### Real-World Cases

**Case 1: UI Component Lifecycle**
React class components use `super(props)` in constructors and `super.componentDidMount()` to extend lifecycle behavior.

**Case 2: Domain Entity Initialization**
Domain entity subclasses call `super()` to initialize base entity fields (ID, timestamps) before adding subclass-specific state.

**Case 3: Framework Base Classes**
Framework base classes define template methods that call `super` implementations, allowing subclasses to extend rather than replace behavior.

---

## 5. Interface Realization via the `implements` Clause

### Definitions

**Core Definition**
Interface realization is the mechanism by which a class commits to satisfying one or more interfaces using the `implements` clause. The class must provide implementations for all interface members. Unlike `extends`, `implements` creates no inheritance relationship—it is a contract check performed by the compiler.

**Technical Definition**
The `implements` clause in a class declaration asserts that the class satisfies the specified interfaces. TypeScript checks that the class has all required members with compatible types. A class can implement multiple interfaces simultaneously. The class's instances are assignable to the interface types. Unlike `extends`, `implements` does not inherit any implementation—the class must provide its own. Interfaces can be implemented by classes without affecting the class's inheritance chain. A class can extend a parent class and implement interfaces simultaneously.

**Beginner-Friendly Explanation**
Realization (`implements`) is a promise: a class says "I will provide all the members this interface requires." TypeScript checks that the class keeps this promise. Unlike inheritance, where the child gets the parent's code, `implements` gives you no code—you must write everything yourself. But `implements` lets a class satisfy multiple interfaces at once (unlike `extends`, which allows only one parent class). It's the foundation of interface-based design: define contracts as interfaces, and let classes implement them however they want.

### Purposes

- To enforce that a class satisfies one or more interface contracts.
- To enable interface-based design where classes are programmed against interfaces, not implementations.
- To allow a class to fulfill multiple contracts simultaneously.
- To decouple class consumers from specific class implementations.
- To enable polymorphism through interface types.

### Syntax Rules and Structure

**General Syntax: Single Interface Implementation**

```typescript
interface InterfaceName {
  property: Type;
  method(): ReturnType;
}

class ClassName implements InterfaceName {
  property: Type = value;
  method(): ReturnType { }
}
```

**Component Breakdown**
- `implements InterfaceName`: The clause asserting the class satisfies the interface.
- The class must provide all interface members.

**General Syntax: Multiple Interface Implementation**

```typescript
class ClassName implements Interface1, Interface2, Interface3 {
  // Must satisfy all three interfaces
}
```

**Component Breakdown**
- Multiple interfaces are separated by commas.

**General Syntax: Extends and Implements Together**

```typescript
class Child extends ParentClass implements Interface1, Interface2 {
  // Inherits from ParentClass and satisfies Interface1 and Interface2
}
```

**Component Breakdown**
- `extends` comes first, then `implements`.

**Syntax Rules**

- A class can implement multiple interfaces.
- The class must provide implementations for all interface members.
- The class's members must be compatible with the interface's member types.
- Interfaces can be implemented by classes that also extend a parent class.
- `implements` does not inherit any implementation—the class provides its own.
- The class's instances are assignable to the interface types.
- Optional interface members do not need to be implemented.
- Readonly interface members can be implemented as readonly or mutable (TypeScript does not enforce readonly on implementation).

**Constraints and Limitations**

- The class must implement all required interface members (compile error otherwise).
- TypeScript does not enforce `readonly` on interface members when implemented by a class.
- Implementing an interface does not inherit any code—full implementation is required.
- A class cannot implement an interface that requires a constructor signature (interfaces describe instances, not constructors).
- Interface members with `private` or `protected` (not allowed in interfaces) cannot be implemented.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Single Interface Implementation

```typescript
// Step 1: Define an interface.
interface Serializable {
  serialize(): string;
  deserialize(data: string): void;
}

// Step 2: Implement the interface in a class.
class User implements Serializable {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  serialize(): string {
    return JSON.stringify({ name: this.name, age: this.age });
  }

  deserialize(data: string): void {
    const parsed = JSON.parse(data);
    this.name = parsed.name;
    this.age = parsed.age;
  }
}

// Step 3: Use the class through the interface type.
const user: Serializable = new User("Alice", 30);
console.log(user.serialize());  // '{"name":"Alice","age":30}'

// Step 4: Missing methods cause errors.
// class Invalid implements Serializable {
//   serialize(): string { return ""; }
//   // ❌ Error: Class 'Invalid' incorrectly implements interface 'Serializable'.
//   // Property 'deserialize' is missing.
// }
```

**Expected Output:**
```
{"name":"Alice","age":30}
```

**Why This Output Occurs:** The `User` class implements `Serializable`, providing both `serialize` and `deserialize` methods. The `user` variable is typed as `Serializable`, so only interface members are accessible.

#### Example 2: Multiple Interfaces and Extends Together

```typescript
// Step 1: Define multiple interfaces.
interface Identifiable {
  id: number;
}

interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}

interface Comparable<T> {
  compareTo(other: T): number;
}

// Step 2: Define a base class.
class Entity {
  protected version: number = 1;
}

// Step 3: Implement multiple interfaces and extend a class.
class User extends Entity implements Identifiable, Timestamped, Comparable<User> {
  id: number;
  createdAt: Date;
  updatedAt: Date;
  name: string;

  constructor(id: number, name: string) {
    super();
    this.id = id;
    this.name = name;
    this.createdAt = new Date();
    this.updatedAt = new Date();
  }

  compareTo(other: User): number {
    return this.name.localeCompare(other.name);
  }

  touch(): void {
    this.updatedAt = new Date();
    this.version += 1;
  }
}

// Step 4: Use the class through different interface types.
const alice = new User(1, "Alice");
const bob = new User(2, "Bob");

const identifiable: Identifiable = alice;
console.log(`ID: ${identifiable.id}`);  // 1

const timestamped: Timestamped = alice;
console.log(`Created: ${timestamped.createdAt.toISOString()}`);

const comparable: Comparable<User> = alice;
console.log(`Compare: ${comparable.compareTo(bob)}`);  // Negative (Alice < Bob)

// Step 5: All members are accessible on the class instance.
console.log(`Name: ${alice.name}`);  // "Alice"
alice.touch();  // Uses protected `version` from Entity
```

**Expected Output:**
```
ID: 1
Created: 2024-...T...Z
Compare: -1
Name: Alice
```

**Why This Output Occurs:** The `User` class extends `Entity` (gaining `version`) and implements three interfaces (`Identifiable`, `Timestamped`, `Comparable<User>`), providing all required members. The class is assignable to each interface type, enabling interface-based polymorphism.

### Real-World Cases

**Case 1: Repository Pattern**
Repository interfaces (`UserRepository`, `OrderRepository`) define contracts that concrete implementations (`PostgresUserRepository`, `MongoUserRepository`) implement.

**Case 2: Plugin Systems**
Plugin interfaces define contracts that plugin classes implement, enabling dynamic loading and polymorphism.

**Case 3: Domain-Driven Design**
Domain interfaces (`AggregateRoot`, `ValueObject`, `Entity`) are implemented by domain classes, enforcing architectural constraints.

---

## 6. Abstract Classes and Abstract Members

### Definitions

**Core Definition**
An abstract class is a class that cannot be instantiated directly and may contain abstract members (methods and properties without implementation) that subclasses must implement. Abstract classes provide a way to define partial implementations and enforce that subclasses complete them.

**Technical Definition**
Abstract classes are declared with the `abstract` keyword. They cannot be instantiated with `new`. Abstract members (declared with `abstract`) have no implementation in the abstract class and must be implemented by concrete subclasses. Abstract classes can also have concrete (implemented) members that subclasses inherit. A subclass of an abstract class must implement all abstract members unless it is also declared abstract. Abstract classes can be extended but not implemented (interfaces cannot extend abstract classes, but classes can extend abstract classes and implement interfaces). Abstract methods cannot be `private` (they must be accessible to subclasses).

**Beginner-Friendly Explanation**
An abstract class is a class that's meant to be a blueprint, not something you create directly. You can't do `new AbstractClass()`—you must create a subclass. Abstract classes can have two kinds of members: regular members with implementation (which subclasses inherit) and abstract members without implementation (which subclasses must provide). This is useful when you have common behavior that all subclasses share, plus some behavior that each subclass must define for itself. For example, an abstract `Shape` class might have a concrete `describe()` method but an abstract `area()` method that each shape must implement.

### Purposes

- To define partial implementations that subclasses complete.
- To enforce that subclasses implement specific members.
- To share common behavior across related classes while requiring specialization.
- To model abstract concepts that have no direct instances.
- To enable template method patterns where the abstract class defines the algorithm skeleton.

### Syntax Rules and Structure

**General Syntax: Abstract Class**

```typescript
abstract class AbstractClassName {
  concreteProperty: Type = value;

  abstract abstractProperty: Type;

  concreteMethod(): ReturnType {
    // implementation
  }

  abstract abstractMethod(params: ParamTypes): ReturnType;
}
```

**Component Breakdown**
- `abstract class`: The keyword combination for abstract classes.
- `abstract abstractProperty`: An abstract property (no initializer).
- `abstract abstractMethod(...)`: An abstract method (no body).

**General Syntax: Concrete Subclass**

```typescript
class ConcreteClass extends AbstractClassName {
  abstractProperty: Type = value;

  abstractMethod(params: ParamTypes): ReturnType {
    // implementation
  }
}
```

**Component Breakdown**
- The concrete subclass must implement all abstract members.

**General Syntax: Abstract Subclass**

```typescript
abstract class AbstractSubclass extends AbstractClassName {
  // May leave some abstract members unimplemented
}
```

**Component Breakdown**
- Abstract subclasses can defer implementation to further subclasses.

**Syntax Rules**

- Abstract classes are declared with `abstract class`.
- Abstract classes cannot be instantiated with `new`.
- Abstract members are declared with `abstract` and have no implementation.
- Abstract properties cannot have initializers.
- Abstract methods cannot have bodies.
- Concrete subclasses must implement all abstract members.
- Abstract subclasses can defer implementation.
- Abstract members cannot be `private` (must be accessible to subclasses).
- Abstract classes can have constructors (called by subclasses via `super()`).
- Abstract classes can implement interfaces.
- Abstract classes can have static members.

**Constraints and Limitations**

- Abstract classes cannot be instantiated.
- Abstract methods cannot be `private`.
- Abstract properties cannot have initializers.
- Abstract members must be implemented by concrete subclasses.
- TypeScript's `abstract` keyword has no runtime representation (it's a compile-time constraint).
- Abstract classes are erased to regular classes at runtime.
- A class cannot extend multiple abstract classes (single inheritance).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Abstract Class

```typescript
// Step 1: Define an abstract class.
abstract class Shape {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  // Concrete method — shared by all shapes
  describe(): string {
    return `${this.name} with area ${this.area().toFixed(2)}`;
  }

  // Abstract method — must be implemented by subclasses
  abstract area(): number;

  // Abstract property — must be implemented by subclasses
  abstract readonly color: string;
}

// Step 2: Attempt to instantiate the abstract class (error).
// const shape = new Shape("Generic");
// ❌ Error: Cannot create an instance of an abstract class.

// Step 3: Define a concrete subclass.
class Circle extends Shape {
  readonly color: string;

  constructor(radius: number, color: string) {
    super("Circle");
    this.radius = radius;
    this.color = color;
  }

  radius: number;

  area(): number {
    return Math.PI * this.radius ** 2;
  }
}

// Step 4: Define another concrete subclass.
class Square extends Shape {
  readonly color: string;

  constructor(side: number, color: string) {
    super("Square");
    this.side = side;
    this.color = color;
  }

  side: number;

  area(): number {
    return this.side ** 2;
  }
}

// Step 5: Create instances of concrete subclasses.
const circle = new Circle(5, "red");
const square = new Square(4, "blue");

console.log(circle.describe());  // "Circle with area 78.54"
console.log(square.describe());  // "Square with area 16.00"

// Step 6: Polymorphism through the abstract class type.
const shapes: Shape[] = [circle, square];
shapes.forEach((s) => console.log(`${s.color}: ${s.describe()}`));
// "red: Circle with area 78.54"
// "blue: Square with area 16.00"
```

**Expected Output:**
```
Circle with area 78.54
Square with area 16.00
red: Circle with area 78.54
blue: Square with area 16.00
```

**Why This Output Occurs:** The abstract `Shape` class defines a concrete `describe()` method that uses the abstract `area()` method. Subclasses `Circle` and `Square` implement `area()` and the abstract `color` property. Instances of subclasses are assignable to `Shape`, enabling polymorphic dispatch.

#### Example 2: Template Method Pattern with Abstract Class

```typescript
// Step 1: Define an abstract class implementing a template method.
abstract class DataProcessor {
  // Template method — defines the algorithm skeleton
  process(data: string[]): string[] {
    const filtered = this.filter(data);
    const transformed = this.transform(filtered);
    const sorted = this.sort(transformed);
    return sorted;
  }

  // Concrete helper method
  protected log(message: string): void {
    console.log(`[${this.constructor.name}] ${message}`);
  }

  // Abstract methods — subclasses provide specific behavior
  protected abstract filter(data: string[]): string[];
  protected abstract transform(data: string[]): string[];
  protected abstract sort(data: string[]): string[];
}

// Step 2: Concrete implementation for numeric data.
class NumberProcessor extends DataProcessor {
  protected filter(data: string[]): string[] {
    this.log("Filtering numbers");
    return data.filter((s) => !isNaN(Number(s)));
  }

  protected transform(data: string[]): string[] {
    this.log("Doubling numbers");
    return data.map((s) => String(Number(s) * 2));
  }

  protected sort(data: string[]): string[] {
    this.log("Sorting numerically");
    return data.sort((a, b) => Number(a) - Number(b));
  }
}

// Step 3: Concrete implementation for text data.
class TextProcessor extends DataProcessor {
  protected filter(data: string[]): string[] {
    this.log("Filtering non-empty strings");
    return data.filter((s) => s.trim().length > 0);
  }

  protected transform(data: string[]): string[] {
    this.log("Uppercasing text");
    return data.map((s) => s.toUpperCase());
  }

  protected sort(data: string[]): string[] {
    this.log("Sorting alphabetically");
    return data.sort();
  }
}

// Step 4: Use the processors.
const numberProcessor = new NumberProcessor();
console.log(numberProcessor.process(["5", "abc", "2", "10", "1"]));
// Output: [LOG lines] then ["2", "4", "10", "20"]

const textProcessor = new TextProcessor();
console.log(textProcessor.process(["banana", "", "apple", "cherry"]));
// Output: [LOG lines] then ["APPLE", "BANANA", "CHERRY"]
```

**Expected Output:**
```
[NumberProcessor] Filtering numbers
[NumberProcessor] Doubling numbers
[NumberProcessor] Sorting numerically
[ '2', '4', '10', '20' ]
[TextProcessor] Filtering non-empty strings
[TextProcessor] Uppercasing text
[TextProcessor] Sorting alphabetically
[ 'APPLE', 'BANANA', 'CHERRY' ]
```

**Why This Output Occurs:** The abstract `DataProcessor` defines the `process` template method, which calls the abstract `filter`, `transform`, and `sort` methods. Subclasses implement these methods with domain-specific logic. The `log` method uses `this.constructor.name` to identify the concrete class, demonstrating polymorphic behavior.

### Real-World Cases

**Case 1: Framework Base Classes**
Frameworks like Angular and NestJS use abstract classes for base components, services, and controllers, defining lifecycle hooks that subclasses implement.

**Case 2: Template Method Pattern**
Data processing pipelines, report generators, and ETL workflows use abstract classes to define algorithm skeletons with customizable steps.

**Case 3: Domain Modeling**
Domain models use abstract classes for base entities and value objects, enforcing that subclasses provide domain-specific behavior.

---

## References

- TypeScript Handbook: Classes (Inheritance) — https://www.typescriptlang.org/docs/handbook/2/classes.html#extends-clause
- TypeScript Handbook: Classes (Implements Clauses) — https://www.typescriptlang.org/docs/handbook/2/classes.html#implements-clauses
- TypeScript Handbook: Classes (Abstract Classes and Members) — https://www.typescriptlang.org/docs/handbook/2/classes.html#abstract-classes-and-members
- TypeScript 4.3 Release Notes (override Keyword) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-3.html
- TypeScript `noImplicitOverride` Documentation — https://www.typescriptlang.org/tsconfig#noImplicitOverride
- TypeScript `strictFunctionTypes` Documentation — https://www.typescriptlang.org/tsconfig#strictFunctionTypes
- MDN: extends — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends
- MDN: super — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super
- TypeScript Playground: Classes — https://www.typescriptlang.org/play/typescript/classes.ts.html
- Effective TypeScript: Item 28 — Prefer Classes to Namespaces
- Total TypeScript: Classes — https://www.totaltypescript.com/tutorials/beginners-typescript/12-classes
- TypeScript ESLint: explicit-member-accessibility — https://typescript-eslint.io/rules/explicit-member-accessibility/
- TypeScript ESLint: no-extraneous-class — https://typescript-eslint.io/rules/no-extraneous-class/