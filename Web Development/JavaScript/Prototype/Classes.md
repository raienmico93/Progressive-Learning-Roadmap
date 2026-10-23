# JavaScript Classes — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A class is a template for creating objects that encapsulates data (fields) and the code that operates on that data (methods), supporting inheritance and other object-oriented patterns.

**Technical Definition:** Per ECMA-262, a class is a special kind of function. Class declarations and expressions create a constructor function whose `prototype` property becomes the `[[Prototype]]` of instances created with `new`. The class body is evaluated in strict mode by default, even without a `"use strict"` directive. Class elements are characterised along three axes: kind (getter, setter, method, or field), location (static or instance), and visibility (public or private), yielding sixteen possible combinations. Class declarations are hoisted but remain in the Temporal Dead Zone (TDZ) until evaluated, behaving like `let` and `const` rather than `var` or function declarations.

**Beginner-Friendly Explanation:** A class is like a blueprint for making objects. If you want to create many "user" objects, you write a `User` class once, and then use it to make as many users as you need. Each user shares the same methods but has its own data. Classes are built on top of JavaScript's prototype system, but they give you a much cleaner, more familiar syntax.

### Key Characteristics

- **"Special functions":** Classes are functions under the hood; `typeof MyClass` is `"function"`.
- **Strict mode by default:** The entire class body executes in strict mode, regardless of the surrounding code.
- **TDZ and hoisting:** Class declarations are hoisted but cannot be accessed before evaluation (type 3 hoisting).
- **No `new` = error:** Calling a class without `new` throws a `TypeError` (unlike ordinary constructor functions).
- **Prototype-based inheritance:** Methods live on the prototype and are shared by all instances; fields live on instances.
- **Rich member syntax:** Supports public/private fields and methods, static members, getters/setters, and static initialization blocks.

### Prerequisites

- Understanding of functions, `this`, and the prototype chain.
- Familiarity with object literals and the `new` operator.
- Basic knowledge of scope and the Temporal Dead Zone.

### Related Programming Areas

- **Prototypal inheritance:** Classes are syntactic sugar over JavaScript's existing prototype-based system.
- **Object-oriented programming:** Encapsulation, inheritance, polymorphism.
- **Module design:** Classes as exported building blocks.
- **Framework components:** React class components (historical), Angular services, etc.

---

## Class Syntax

### Definitions

**Core Definition:** Class syntax is the `class` keyword construct used to define a class, comprising a name, an optional `extends` clause, and a body containing members.

**Technical Definition:** A class definition is a `ClassDeclaration` or `ClassExpression` whose body is a `ClassBody` containing `ClassElement`s (methods, fields, static blocks). The body is enclosed in braces and evaluated in strict mode.

**Beginner-Friendly Explanation:** Class syntax is the way you write a class: the `class` keyword, the class name, and a pair of curly braces containing the class's contents.

### Purposes

- To define reusable object templates with clear, declarative syntax.
- To group data and behaviour in a single construct.
- To enable inheritance and polymorphism with dedicated keywords.

### Syntax Rules and Structure

```js
class ClassName extends ParentClass {
  // class body
}
```

**Component Breakdown:**
- `class`: The declaration keyword.
- `ClassName`: The identifier for the class (required for declarations).
- `extends ParentClass`: Optional inheritance clause.
- `{ ... }`: The class body containing members.

**Constraints and Limitations:**
- A class declaration cannot be anonymous; a class expression can be.
- The class body cannot contain statements, only member declarations (though static blocks allow statements).

### Annotated Code Examples

#### Example 1: Minimal Class

```js
// Step 1: Declare a class with a constructor and a method.
class Greeter {
  constructor(name) {
    this.name = name;
  }
  greet() {
    return `Hello, ${this.name}!`;
  }
}

// Step 2: Instantiate and use.
const g = new Greeter("Alice");
console.log(g.greet()); // "Hello, Alice!"
console.log(typeof Greeter); // "function" (classes are special functions)
```

**Expected Output:**
```
Hello, Alice!
function
```

**Why This Output:** The class creates a constructor function. `new Greeter("Alice")` creates an instance; `greet()` is found on the prototype.

### Real-World Cases

- **Domain models:** `User`, `Product`, `Order` classes.
- **UI components:** Class-based components in older React code.
- **Service layers:** `ApiClient`, `Logger` classes.

---

## Class Declarations

### Definitions

**Core Definition:** A class declaration is a statement that defines a named class in the current scope.

**Technical Definition:** A `ClassDeclaration` is a hoistable declaration that creates a binding for the class name in the current lexical environment. Like `let` and `const`, the binding is in the TDZ until the declaration is evaluated.

**Beginner-Friendly Explanation:** A class declaration is the standard `class MyClass { ... }` statement. It gives the class a name you can use later.

### Purposes

- To define a named class in a scope.
- To support clear, reusable object templates.
- To enable inheritance with a stable name.

### Syntax Rules and Structure

```js
class ClassName {
  // body
}
```

**Constraints and Limitations:**
- Must have a name.
- Hoisted but in TDZ (cannot be used before declaration).
- Cannot be redeclared in the same scope.

### Annotated Code Examples

#### Example 1: Declaration and Usage

```js
// Step 1: Declare a class.
class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}

// Step 2: Instantiate.
const r = new Rectangle(10, 20);
console.log(r.height * r.width); // 200
```

**Expected Output:**
```
200
```

**Why This Output:** The declaration creates a binding for `Rectangle`; `new` invokes the constructor.

### Real-World Cases

- **Main classes in modules:** Exporting a primary class per file.
- **Application entities:** `class Invoice { ... }`.

---

## Class Expressions

### Definitions

**Core Definition:** A class expression is a class definition used as an expression (e.g., assigned to a variable) rather than as a standalone statement.

**Technical Definition:** A `ClassExpression` is similar to a `ClassDeclaration` but appears in expression position. It may be anonymous or named; a named class expression's name is scoped only to the class body.

**Beginner-Friendly Explanation:** A class expression is a class you assign to a variable or pass to a function — it is not a standalone statement.

### Purposes

- To create classes dynamically or conditionally.
- To assign classes to variables or export them as values.
- To create anonymous classes for one-off use.

### Syntax Rules and Structure

```js
const MyClass = class { /* ... */ };
const MyClass = class NamedClass { /* ... */ };
```

**Constraints and Limitations:**
- The optional name of a named class expression is only visible inside the class body.
- Anonymous class expressions have `name` inferred from the assignment target.

### Annotated Code Examples

#### Example 1: Class Expression

```js
// Step 1: Assign an anonymous class to a variable.
const Point = class {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
};

const p = new Point(3, 4);
console.log(p.x + p.y); // 7
```

**Expected Output:**
```
7
```

**Why This Output:** The class expression produces a constructor assigned to `Point`; `new` works as usual.

### Real-World Cases

- **Conditional class definitions:** Choosing a class implementation based on environment.
- **Higher-order functions:** Returning a class from a factory function.

---

## Class Methods

### Definitions

**Core Definition:** Class methods are functions defined in the class body that are added to the class's prototype and shared by all instances.

**Technical Definition:** Method definitions in a class body create non-enumerable properties on `ClassName.prototype`. They can be ordinary functions, async functions, generator functions, or async generator functions.

**Beginner-Friendly Explanation:** Methods are the actions your objects can perform. They are defined once on the prototype and shared by every instance.

### Purposes

- To define behaviour shared by all instances.
- To encapsulate operations on the object's data.
- To enable polymorphism through overriding in subclasses.

### Syntax Rules and Structure

```js
class MyClass {
  methodName(params) {
    // body
  }
  async asyncMethod() { /* ... */ }
  *generatorMethod() { /* ... */ }
}
```

**Constraints and Limitations:**
- Methods are non-enumerable (unlike methods added via assignment).
- They are called with the instance as `this` when invoked as `instance.method()`.

### Annotated Code Examples

#### Example 1: Instance Method

```js
class Calculator {
  constructor(value) {
    this.value = value;
  }
  add(n) {
    this.value += n;
    return this;
  }
  getResult() {
    return this.value;
  }
}

const calc = new Calculator(10);
console.log(calc.add(5).getResult()); // 15
```

**Expected Output:**
```
15
```

**Why This Output:** Methods are on the prototype; chaining works because `add` returns `this`.

### Real-World Cases

- **Entity behaviour:** `user.getFullName()`, `order.calculateTotal()`.
- **Utility methods:** `array.sum()`, `string.reverse()`.

---

## Class Fields

### Definitions

**Core Definition:** Class fields are instance properties defined directly in the class body using field declaration syntax, independent of the constructor.

**Technical Definition:** Public instance fields are added to each instance at construction time — before the constructor body runs in a base class, or immediately after `super()` returns in a subclass. Fields without initialisers are set to `undefined`. They are added using `[[DefineOwnProperty]]` semantics (like `Object.defineProperty`), so they do not trigger setters on the prototype.

**Beginner-Friendly Explanation:** Class fields let you declare instance variables directly in the class body, without writing them inside the constructor. It is a cleaner way to set up default values.

### Purposes

- To declare instance properties explicitly in the class body.
- To provide default values without constructor logic.
- To improve readability by separating field declarations from constructor logic.

### Syntax Rules and Structure

```js
class MyClass {
  fieldName = initialValue; // public instance field
  anotherField;             // initialised to undefined
}
```

**Constraints and Limitations:**
- Field declarations do not use `const` or `let`.
- Field initialisers are evaluated at instance creation time.
- Derived class fields are added after `super()`.

### Annotated Code Examples

#### Example 1: Fields with Defaults

```js
class Counter {
  count = 0; // field with default
  label;     // field without initialiser

  constructor(label) {
    this.label = label;
  }

  increment() {
    this.count++;
  }
}

const c = new Counter("clicks");
console.log(c.count);  // 0
c.increment();
console.log(c.count);  // 1
console.log(c.label);  // "clicks"
```

**Expected Output:**
```
0
1
clicks
```

**Why This Output:** The field `count` is initialised to 0 on each instance. `label` is `undefined` until the constructor assigns it.

### Real-World Cases

- **Default configuration:** `retries = 3`, `timeout = 5000`.
- **UI state:** `isOpen = false`, `items = []`.

---

## Static Methods

### Definitions

**Core Definition:** Static methods are functions defined on the class itself (not on instances) using the `static` keyword.

**Technical Definition:** Static methods are added to the class constructor as own properties at class evaluation time. They are accessed via `ClassName.method()`, not via instances.

**Beginner-Friendly Explanation:** Static methods belong to the class, not to individual objects. You call them on the class itself — like `Math.max()`.

### Purposes

- To provide utility functions related to the class.
- To create factory methods (e.g., `ClassName.fromJSON()`).
- To implement caches or configuration at the class level.

### Syntax Rules and Structure

```js
class MyClass {
  static methodName(params) {
    // body
  }
}
```

**Constraints and Limitations:**
- Static methods are not available on instances.
- The name `prototype` is forbidden for static members.
- Inside a static method, `this` refers to the class itself.

### Annotated Code Examples

#### Example 1: Factory Method

```js
class User {
  constructor(name) {
    this.name = name;
  }
  static createGuest() {
    return new User("Guest");
  }
}

const guest = User.createGuest();
console.log(guest.name); // "Guest"
```

**Expected Output:**
```
Guest
```

**Why This Output:** `createGuest` is called on the class, not an instance, and returns a new `User`.

### Real-World Cases

- **Factory methods:** `Date.now()`, `Array.from()`.
- **Utility functions:** `ClassName.isValid(data)`.
- **Configuration:** `ClassName.defaultOptions`.

---

## Static Fields

### Definitions

**Core Definition:** Static fields are properties defined on the class itself (not on instances) using the `static` keyword.

**Technical Definition:** Static fields are added to the class constructor at evaluation time. They are useful for data that should exist once per class, not per instance — such as caches, configuration, or counters.

**Beginner-Friendly Explanation:** Static fields are like class-level variables. There is only one copy, shared by the class, not one per object.

### Purposes

- To store class-level configuration or constants.
- To implement instance counters.
- To cache shared data.

### Syntax Rules and Structure

```js
class MyClass {
  static fieldName = value;
}
```

**Constraints and Limitations:**
- Static fields are not reinitialised on subclasses but are accessible via the prototype chain.
- The name `prototype` is forbidden.

### Annotated Code Examples

#### Example 1: Instance Counter

```js
class Widget {
  static count = 0;

  constructor() {
    Widget.count++;
  }
}

new Widget();
new Widget();
console.log(Widget.count); // 2
```

**Expected Output:**
```
2
```

**Why This Output:** `Widget.count` is a single static field shared by all instances; the constructor increments it.

### Real-World Cases

- **Instance counting:** Tracking how many objects have been created.
- **Class-level caches:** `static cache = new Map()`.
- **Configuration:** `static API_URL = "..."`.

---

## Private Fields

### Definitions

**Core Definition:** Private fields are class fields accessible only from within the class body, declared with a `#` prefix.

**Technical Definition:** Private fields are not ordinary properties; they are "private elements" whose privacy is enforced by the language. They cannot be accessed, enumerated, deleted, or proxied from outside the class. All private names declared in a class must be unique (shared namespace between static and instance elements).

**Beginner-Friendly Explanation:** Private fields are hidden from the outside world. Only the class's own methods can read or write them. You declare them with `#name`.

### Purposes

- To enforce encapsulation at the language level.
- To hide implementation details.
- To prevent external code from accidentally modifying internal state.

### Syntax Rules and Structure

```js
class MyClass {
  #privateField = value;
  #privateMethod() { /* ... */ }
}
```

**Constraints and Limitations:**
- Cannot be accessed outside the class (SyntaxError).
- Not inherited by subclasses.
- Not cloned by `structuredClone()`, not affected by `Object.freeze()`.
- The name `#constructor` is forbidden.

### Annotated Code Examples

#### Example 1: Private Field with Public Accessor

```js
class BankAccount {
  #balance = 0;

  deposit(amount) {
    this.#balance += amount;
  }

  get balance() {
    return this.#balance;
  }
}

const acc = new BankAccount();
acc.deposit(100);
console.log(acc.balance); // 100
console.log(acc.#balance); // SyntaxError (outside class)
```

**Expected Output:**
```
100
SyntaxError
```

**Why This Output:** `#balance` is accessible only inside the class; external access is a syntax error.

### Real-World Cases

- **Financial systems:** Hiding balance, PIN, or token fields.
- **Library internals:** Preventing users from mutating internal state.
- **Secure APIs:** Enforcing invariants that public properties would violate.

---

## Private Methods

### Definitions

**Core Definition:** Private methods are class methods accessible only from within the class body, declared with a `#` prefix.

**Technical Definition:** Private methods are private elements like private fields. They are non-enumerable and cannot be invoked from outside the class. They can be instance or static, and they participate in the shared private name namespace.

**Beginner-Friendly Explanation:** Private methods are helper functions that only the class itself can call. They keep internal logic hidden.

### Purposes

- To hide helper logic from external callers.
- To encapsulate complex internal operations.
- To prevent external code from relying on implementation details.

### Syntax Rules and Structure

```js
class MyClass {
  #privateMethod(params) { /* ... */ }
}
```

**Constraints and Limitations:** Same restrictions as private fields.

### Annotated Code Examples

#### Example 1: Private Helper Method

```js
class Temperature {
  #celsius;

  constructor(celsius) {
    this.#celsius = celsius;
  }

  #toFahrenheit() {
    return (this.#celsius * 9) / 5 + 32;
  }

  get fahrenheit() {
    return this.#toFahrenheit();
  }
}

const t = new Temperature(100);
console.log(t.fahrenheit); // 212
```

**Expected Output:**
```
212
```

**Why This Output:** `#toFahrenheit` is callable only inside the class; the public getter exposes the result.

### Real-World Cases

- **Validation logic:** Private `#validate()` methods.
- **Caching:** Private `#compute()` helpers.
- **State management:** Private `#reset()` methods.

---

## Getters and Setters

### Definitions

**Core Definition:** Getters (`get`) and setters (`set`) are special methods that are invoked when a property is read or written, respectively, allowing computed or validated property access.

**Technical Definition:** Getter and setter functions are defined in the class body with `get` and `set` prefixes. They create accessor properties on the prototype (for instance accessors) or on the constructor (for static accessors). A getter without a setter makes the property effectively read-only.

**Beginner-Friendly Explanation:** Getters and setters let you control how a property is read or written, like a smart property that runs code when you access it.

### Purposes

- To compute property values on the fly.
- To validate or transform values on assignment.
- To expose private data safely.
- To create read-only properties (getter only).

### Syntax Rules and Structure

```js
class MyClass {
  get propName() { return value; }
  set propName(value) { /* ... */ }
}
```

**Constraints and Limitations:**
- A getter and setter pair can share the same name.
- Attempting to set a getter-only property throws a TypeError in strict mode.
- Accessors are non-enumerable.

### Annotated Code Examples

#### Example 1: Getter and Setter with Validation

```js
class Person {
  #age = 0;

  get age() {
    return this.#age;
  }

  set age(value) {
    if (value < 0) throw new RangeError("Age cannot be negative");
    this.#age = value;
  }
}

const p = new Person();
p.age = 25;
console.log(p.age); // 25
try { p.age = -5; } catch (e) { console.log(e.message); } // "Age cannot be negative"
```

**Expected Output:**
```
25
Age cannot be negative
```

**Why This Output:** The setter validates before assigning to the private field; the getter returns the stored value.

### Real-World Cases

- **Form models:** Validating input on assignment.
- **Computed properties:** `fullName` derived from `firstName` and `lastName`.
- **Immutable properties:** Getter-only `id` fields.

---

## Constructors

### Definitions

**Core Definition:** The constructor is a special method in a class that runs when an instance is created with `new`, responsible for initialising the object.

**Technical Definition:** The `constructor` method is invoked during `[[Construct]]`. A class can have at most one constructor; if omitted, a default constructor is used (empty for base classes, `super(...args)` for derived classes). Constructors cannot be async, generators, or accessors.

**Beginner-Friendly Explanation:** The constructor is the "setup" function that runs when you create a new object. It receives the arguments you pass to `new` and prepares the object.

### Purposes

- To initialise instance fields.
- To validate constructor arguments.
- To call the parent constructor in derived classes.

### Syntax Rules and Structure

```js
class MyClass {
  constructor(params) {
    // initialisation
  }
}
```

**Constraints and Limitations:**
- Only one constructor per class.
- In a derived class, `super()` must be called before accessing `this`.
- Cannot be named `constructor` as a getter/setter or async method.

### Annotated Code Examples

#### Example 1: Constructor with Validation

```js
class Circle {
  constructor(radius) {
    if (radius <= 0) throw new RangeError("Radius must be positive");
    this.radius = radius;
  }
}

const c = new Circle(5);
console.log(c.radius); // 5
try { new Circle(-1); } catch (e) { console.log(e.name); } // "RangeError"
```

**Expected Output:**
```
5
RangeError
```

**Why This Output:** The constructor validates the argument and throws for invalid input.

### Real-World Cases

- **Entity creation:** `new User(name, email)`.
- **Resource acquisition:** Opening connections in the constructor.
- **Configuration objects:** Normalising options.

---

## `super`

### Definitions

**Core Definition:** `super` is a keyword used in derived classes to access the parent class's constructor or methods.

**Technical Definition:** In a derived class constructor, `super(...args)` invokes the parent constructor and must be called before using `this`. In methods, `super.method()` calls the corresponding method on the parent prototype. `super` is not a function object; it is a syntactic form bound to the current method's home object.

**Beginner-Friendly Explanation:** `super` lets you call the parent class's constructor or methods from a child class.

### Purposes

- To initialise inherited state in the parent constructor.
- To call parent methods while extending their behaviour.
- To access overridden functionality.

### Syntax Rules and Structure

```js
super(args);          // call parent constructor
super.methodName();   // call parent method
```

**Constraints and Limitations:**
- `super()` is only valid in a derived class constructor.
- Must be called before accessing `this`.
- `super` cannot be used in static methods to access instance fields.

### Annotated Code Examples

#### Example 1: `super` in Constructor and Method

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} makes a sound.`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // call parent constructor
    this.breed = breed;
  }
  speak() {
    return `${super.speak()} ${this.name} barks.`;
  }
}

const d = new Dog("Rex", "Lab");
console.log(d.speak()); // "Rex makes a sound. Rex barks."
```

**Expected Output:**
```
Rex makes a sound. Rex barks.
```

**Why This Output:** `super(name)` initialises the parent part; `super.speak()` calls the parent method and extends its result.

### Real-World Cases

- **Framework lifecycle:** Calling parent `componentDidMount()`.
- **Error hierarchies:** `super(message)` in custom error classes.
- **UI components:** Extending base component behaviour.

---

## Inheritance

### Definitions

**Core Definition:** Inheritance is the mechanism by which a class (subclass) acquires the properties and methods of another class (superclass).

**Technical Definition:** JavaScript uses prototypal inheritance. `class Child extends Parent` sets `Child.prototype`'s `[[Prototype]]` to `Parent.prototype`, and `Child`'s `[[Prototype]]` to `Parent`. This creates a prototype chain: instance → `Child.prototype` → `Parent.prototype` → `Object.prototype`.

**Beginner-Friendly Explanation:** Inheritance lets a child class reuse and extend the behaviour of a parent class. The child inherits the parent's methods and can add or override them.

### Purposes

- To reuse code across related classes.
- To model "is-a" relationships (a `Dog` is an `Animal`).
- To enable polymorphism through method overriding.

### Syntax Rules and Structure

```js
class Parent { /* ... */ }
class Child extends Parent { /* ... */ }
```

**Constraints and Limitations:**
- A class can extend only one parent (single inheritance).
- Private elements are not inherited.
- The prototype chain determines method lookup.

### Annotated Code Examples

#### Example 1: Multi-level Inheritance

```js
class Vehicle {
  constructor(wheels) {
    this.wheels = wheels;
  }
  describe() {
    return `Has ${this.wheels} wheels.`;
  }
}

class Car extends Vehicle {
  constructor(brand) {
    super(4);
    this.brand = brand;
  }
  describe() {
    return `${this.brand}: ${super.describe()}`;
  }
}

const car = new Car("Toyota");
console.log(car.describe()); // "Toyota: Has 4 wheels."
```

**Expected Output:**
```
Toyota: Has 4 wheels.
```

**Why This Output:** `Car` inherits from `Vehicle`, calls `super(4)`, and overrides `describe` while using `super.describe()`.

### Real-World Cases

- **Error hierarchies:** `class ValidationError extends Error`.
- **UI frameworks:** `class MyComponent extends React.Component`.
- **Domain models:** `PremiumUser extends User`.

---

## `extends`

### Definitions

**Core Definition:** The `extends` keyword is used in a class declaration or expression to create a class as a child of another class or constructor.

**Technical Definition:** `class Child extends Parent` sets up the prototype chain so that `Child` inherits from `Parent`. The `extends` clause can be any constructor expression, including `null` (which creates a class with no prototype).

**Beginner-Friendly Explanation:** `extends` is the keyword that says "this class inherits from that one."

### Purposes

- To create subclasses with inherited behaviour.
- To extend built-ins (`Error`, `Array`, `Map`).
- To establish "is-a" relationships.

### Syntax Rules and Structure

```js
class Child extends Parent { /* ... */ }
const Child = class extends Parent { /* ... */ };
```

**Constraints and Limitations:**
- Only one `extends` clause per class.
- `extends null` creates a class whose instances have no prototype.

### Annotated Code Examples

#### Example 1: Extending a Built-in

```js
class MyError extends Error {
  constructor(message, code) {
    super(message);
    this.name = "MyError";
    this.code = code;
  }
}

try {
  throw new MyError("Something broke", 500);
} catch (e) {
  console.log(e.name, e.code, e.message); // "MyError 500 Something broke"
}
```

**Expected Output:**
```
MyError 500 Something broke
```

**Why This Output:** `MyError` extends `Error`, inheriting its stack and message behaviour, and adds a `code` property.

### Real-World Cases

- **Custom errors:** `class ApiError extends Error`.
- **Extended collections:** `class Stack extends Array`.
- **Framework base classes:** `class Admin extends User`.

---

## Abstract Classes

### Definitions

**Core Definition:** An abstract class is a class that is designed to be inherited from but not instantiated directly, often defining a partial implementation or a contract for subclasses.

**Technical Definition:** JavaScript does not have a native `abstract` keyword or built-in enforcement of abstract classes. Abstract behaviour is simulated by conventions: throwing an error in the constructor if `new.target === TheClass`, or by defining methods that throw "not implemented" errors.

**Beginner-Friendly Explanation:** An abstract class is a "template for templates." It defines common code but is not meant to be used directly — you must create a subclass to use it. JavaScript does not enforce this, so you enforce it yourself.

### Purposes

- To define a common interface for a family of classes.
- To share partial implementation while requiring subclasses to fill in the rest.
- To prevent direct instantiation of a base class.

### Syntax Rules and Structure

There is no dedicated syntax. Common simulation patterns:

```js
class AbstractBase {
  constructor() {
    if (new.target === AbstractBase) {
      throw new TypeError("Cannot instantiate abstract class");
    }
  }
  mustOverride() {
    throw new Error("Method must be implemented");
  }
}
```

**Constraints and Limitations:**
- Not enforced by the language; relies on runtime checks.
- TypeScript provides `abstract` as a compile-time feature, but it disappears at runtime.

### Annotated Code Examples

#### Example 1: Simulated Abstract Class

```js
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new TypeError("Shape is abstract");
    }
  }
  area() {
    throw new Error("area() must be implemented");
  }
}

class Square extends Shape {
  constructor(side) {
    super();
    this.side = side;
  }
  area() {
    return this.side ** 2;
  }
}

try { new Shape(); } catch (e) { console.log(e.message); } // "Shape is abstract"
const sq = new Square(4);
console.log(sq.area()); // 16
```

**Expected Output:**
```
Shape is abstract
16
```

**Why This Output:** The constructor check prevents direct instantiation; the subclass implements the required method.

### Real-World Cases

- **Plugin architectures:** Base plugin class with required hooks.
- **Data adapters:** Abstract `DataSource` with concrete implementations.
- **Game entities:** Abstract `Character` with concrete `Player` and `Enemy`.

---

## Class Hoisting

### Definitions

**Core Definition:** Class hoisting is the behaviour whereby class declarations are hoisted to the top of their scope but remain in the Temporal Dead Zone until evaluated.

**Technical Definition:** Class declarations are hoisted with type 3 behaviour (like `let` and `const`): the binding is created at the top of the scope, but it is uninitialised until the class declaration is evaluated. Accessing the class before its declaration throws a `ReferenceError`.

**Beginner-Friendly Explanation:** Classes are "hoisted" in the sense that the name exists throughout the scope, but you cannot use the class before you declare it — doing so throws an error.

### Purposes

- To ensure that classes are not used before they are fully defined.
- To maintain consistency with `let` and `const` semantics.
- To prevent subtle bugs caused by partial class definitions.

### Syntax Rules and Structure

```js
// console.log(MyClass); // ReferenceError (TDZ)
class MyClass { /* ... */ }
```

**Constraints and Limitations:**
- Unlike `var`, which is hoisted with `undefined`, classes are not initialised until evaluation.
- Unlike function declarations, classes cannot be called before their definition.

### Annotated Code Examples

#### Example 1: TDZ with Classes

```js
try {
  const obj = new MyClass(); // ReferenceError
} catch (e) {
  console.log(e.name); // "ReferenceError"
}

class MyClass {
  constructor() {
    this.x = 1;
  }
}
```

**Expected Output:**
```
ReferenceError
```

**Why This Output:** The class binding is in the TDZ before the declaration is evaluated; `new` throws a `ReferenceError`.

### Real-World Cases

- **Module ordering:** Classes must be defined before use within a module.
- **Circular dependencies:** Understanding TDZ helps diagnose circular import errors.
- **Debugging:** TDZ errors are clearer than the silent `undefined` from `var` hoisting.

---

## Class vs. Constructor Functions

### Definitions

**Core Definition:** Class syntax is a modern (ES2015) declarative form for creating constructor functions and prototypes; constructor functions are the older, function-based equivalent.

**Technical Definition:** Classes are "special functions" — under the hood they create a constructor function and populate its `prototype`. However, classes enforce stricter semantics: they must be called with `new`, run in strict mode, are not hoisted in the same way, and support `super`, private elements, and static blocks natively.

**Beginner-Friendly Explanation:** Before ES2015, you wrote "classes" using functions and prototypes. The `class` keyword does the same thing but with cleaner syntax and helpful safety checks (like requiring `new`).

### Purposes

- To choose the appropriate style for a codebase.
- To understand legacy code that uses constructor functions.
- To appreciate the safety and clarity benefits of classes.

### Key Differences

| Aspect | Class | Constructor Function |
|---|---|---|
| **Call without `new`** | TypeError | Works (may pollute globals) |
| **Hoisting** | TDZ (type 3) | Fully hoisted (type 1) |
| **Strict mode** | Always | Only if declared |
| **`super`** | Native | Not available |
| **Private elements** | Native (`#`) | Simulated (closures/WeakMap) |
| **Methods enumerable** | No | Yes (if assigned to `prototype`) |
| **`extends`** | Native | Manual prototype chain |

### Annotated Code Examples

#### Example 1: Equivalent Implementations

```js
// Constructor function style
function PersonFn(name) {
  this.name = name;
}
PersonFn.prototype.greet = function () {
  return `Hi, ${this.name}`;
};

// Class style
class PersonClass {
  constructor(name) {
    this.name = name;
  }
  greet() {
    return `Hi, ${this.name}`;
  }
}

console.log(new PersonFn("A").greet()); // "Hi, A"
console.log(new PersonClass("B").greet()); // "Hi, B"
```

**Expected Output:**
```
Hi, A
Hi, B
```

**Why This Output:** Both create objects with a `name` property and a shared `greet` method. The class version is stricter and more concise.

### Real-World Cases

- **Legacy codebases:** Understanding pre-ES2015 patterns.
- **Transpilation:** Babel converts classes to constructor functions for older targets.
- **Style guides:** Modern projects overwhelmingly prefer class syntax.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Classes — https://tc39.es/ecma262/
- MDN Web Docs – Classes — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- MDN Web Docs – Private elements — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_elements
- MDN Web Docs – `static` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static
- MDN Web Docs – Public class fields — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Public_class_fields
- MDN Web Docs – `constructor` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor
- MDN Web Docs – `super` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super
- MDN Web Docs – `extends` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends
- MDN Web Docs – Hoisting — https://developer.mozilla.org/en-US/docs/Glossary/Hoisting
- MDN Web Docs – Using classes — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_classes
- MDN Web Docs – Getters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get
- MDN Web Docs – Setters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set