Here is a comprehensive research cheat sheet on **PHP Inheritance and Composition**, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Inheritance

### Definitions

**Core Definition**
**Inheritance** is a mechanism in object-oriented programming where a new class (the **child** or **subclass**) is created based on an existing class (the **parent** or **base class**). The child class automatically acquires the public and protected properties, methods, and constants of the parent, and may add new members or modify (override) inherited behavior.

**Technical Definition**
PHP implements inheritance via the `extends` keyword in a class declaration. When a class extends another, the subclass inherits all `public` and `protected` members (methods, properties, and constants) from the parent class; `private` members are not accessible to the child. PHP supports **single inheritance**—a class may extend only one parent—but **multi-level inheritance** is permitted (a child can itself be extended). The parent class must be defined before the child class unless autoloading is used. Inheritance models an **“is-a”** relationship (e.g., a `Dog` **is an** `Animal`), and PHP enforces signature compatibility rules for overridden methods, including return type covariance and parameter contravariance.

**Beginner-Friendly Explanation**
Inheritance is like a family tree for classes. A child class automatically gets all the “family traits” (public and protected properties and methods) from its parent class. The child can then add its own unique traits or change some of the inherited ones. For example, a `SportsCar` class can inherit from a `Car` class, getting all the basic car features (wheels, engine), and then add a `turboBoost()` method. It saves you from rewriting the same code over and over.

---

### Key Characteristics

- **“Is-a” Relationship**: Inheritance should only be used when the child genuinely **is a** specialised version of the parent (e.g., `Manager` is an `Employee`).
- **Single Inheritance**: PHP does not support multiple inheritance (a class cannot extend two classes), but multi-level inheritance is allowed.
- **Member Inheritance**: Public and protected members are inherited; private members are not accessible to the child but may be redeclared.
- **Override Capability**: Inherited methods and properties can be redeclared in the child class (unless marked `final`).
- **`parent::` Access**: The child can explicitly call the parent’s version of an overridden method.
- **Visibility Relaxation**: Overridden members may widen visibility (e.g., `protected` → `public`) but cannot narrow it.
- **Constructor Exemption**: Constructors are exempt from the usual signature compatibility rules (LSP) when being overridden.

---

### Prerequisites

- **PHP 7.4+ Recommended**; **PHP 8.0+** for typed properties, union types, and constructor promotion.
- **Basic Class and Object Syntax**: Understanding of `class`, `new`, `$this`, properties, and methods.
- **Awareness of Visibility**: `public`, `protected`, and `private` determine what is inherited and accessible.

---

### Related Programming Areas

- **Polymorphism**: Inheritance enables objects of different classes to be treated uniformly through a common parent type.
- **Abstract Classes and Interfaces**: Abstract classes provide partial implementations; interfaces define contracts. Both are closely related to inheritance.
- **Composition**: An alternative to inheritance where objects are composed from other objects rather than extended.
- **SOLID Principles**: The Liskov Substitution Principle (LSP) directly governs correct inheritance design.

---

## Core Concepts / Features

---

### 1. Inheritance (General)

#### Definitions

**Core Definition**
Inheritance is the mechanism by which a child class acquires the members of a parent class and can extend or modify them.

**Technical Definition**
In PHP, inheritance is established with the `extends` keyword. The child class inherits all `public` and `protected` methods, properties, and constants from the parent. The child may override inherited methods (unless `final`) and add new members. `Private` members of the parent are not accessible to the child, and the child may redeclare a private member with the same name without affecting the parent’s private member. Return type declarations on overridden methods must be compatible with the parent (covariant), and omitting a return type where the parent has one triggers a deprecation notice as of PHP 8.1.

**Beginner-Friendly Explanation**
Inheritance lets you build a new class on top of an existing one. The new class gets everything the old class had (as long as it is not private) and can add its own features or change some inherited behaviors. It is a way to reuse code and model real-world relationships.

#### Purposes

- To reuse code by inheriting common functionality from a parent class.
- To model hierarchical “is-a” relationships (e.g., `Dog` → `Animal`).
- To enable polymorphism: a child object can be used wherever a parent type is expected.
- To allow specialized behavior through method overriding while retaining the parent’s interface.

#### Syntax Rules and Structure

**General Syntax**
```php
class ParentClass { /* ... */ }

class ChildClass extends ParentClass
{
    // additional members or overrides
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `extends` | Keyword that establishes the inheritance relationship. |
| `ParentClass` | The class being inherited from (must be defined before the child). |
| `ChildClass` | The class that inherits. |
| Members | Inherited public/protected members are automatically available. |

**Syntax Rules**

- Only one parent class may be specified (single inheritance).
- The parent class must be declared before the child class (unless autoloading is used).
- Visibility of overridden members may be widened but not narrowed.
- Return type compatibility must be maintained (covariance); parameters must be contravariant or identical.

**Constraints and Limitations**

- **No multiple inheritance**: PHP does not support extending multiple classes.
- **Private members are not inherited**: The child cannot access them directly, but can redeclare its own private members with the same name.
- **LSP violations are not always caught by PHP**: PHP enforces signature compatibility, but behavioral contracts (preconditions, postconditions, invariants) are the developer’s responsibility.

#### Annotated Code Examples

**Example 1: Basic Inheritance**

```php
<?php
class Animal
{
    public function __construct(protected string $name) {}

    public function eat(): string
    {
        return "{$this->name} is eating.";
    }
}

class Dog extends Animal
{
    public function bark(): string
    {
        return "{$this->name} says Woof!";
    }
}

$dog = new Dog("Rex");
echo $dog->eat() . "\n";  // Outputs: Rex is eating.
echo $dog->bark() . "\n"; // Outputs: Rex says Woof!
?>
```

**Expected Output:**
```
Rex is eating.
Rex says Woof!
```

**Why This Output:**  
`Dog` extends `Animal`, inheriting the constructor and the `eat()` method. The child adds `bark()` and can use the inherited `$name` property (protected) inside its own method.

**Step-by-Step Setup:**
1. Save as `inheritance_basic.php`.
2. Run `php inheritance_basic.php`.
3. The output confirms the child inherited `eat()` and added `bark()`.

---

**Example 2: Multi-Level Inheritance**

```php
<?php
class Vehicle
{
    public function move(): string { return "Moving..."; }
}

class Car extends Vehicle
{
    public function honk(): string { return "Beep!"; }
}

class SportsCar extends Car
{
    public function turbo(): string { return "Turbo boost!"; }
}

$sportsCar = new SportsCar();
echo $sportsCar->move() . "\n";  // Outputs: Moving...
echo $sportsCar->honk() . "\n";  // Outputs: Beep!
echo $sportsCar->turbo() . "\n"; // Outputs: Turbo boost!
?>
```

**Expected Output:**
```
Moving...
Beep!
Turbo boost!
```

**Why This Output:**  
`SportsCar` extends `Car`, which extends `Vehicle`. `SportsCar` inherits from both levels: `move()` from `Vehicle` and `honk()` from `Car`, plus its own `turbo()`.

---

#### Real-World Cases

- **Framework Base Controllers**: `UserController extends AbstractController` inherits request handling, templating, and dependency injection infrastructure.
- **Exception Hierarchies**: `InsufficientFundsException extends RuntimeException` inherits the standard exception API and adds domain-specific semantics.
- **UI Widgets**: `Button extends Widget` inherits rendering and event handling, adding button-specific behavior.

---

### 2. The `extends` Keyword

#### Definitions

**Core Definition**
`extends` is the keyword used in a class declaration to specify that the class inherits from another class.

**Technical Definition**
The `extends` keyword appears immediately after the child class name and before the parent class name: `class Child extends Parent`. It establishes a compile-time inheritance relationship. The parent class must exist before the child class is declared (unless an autoloader resolves it). A class declaration may contain at most one `extends` clause.

**Beginner-Friendly Explanation**
`extends` is the word you write to say, “This new class is based on that existing class.” It creates the parent-child link.

#### Purposes

- To declare the parent class of a child class.
- To establish the inheritance hierarchy that enables member reuse and polymorphism.
- To signal an “is-a” relationship in the type system.

#### Syntax Rules and Structure

**General Syntax**
```php
class ChildClass extends ParentClass
{
    // ...
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `class` | Class declaration keyword. |
| `ChildClass` | The inheriting class. |
| `extends` | Inheritance keyword. |
| `ParentClass` | The class being inherited (must be defined first). |

**Constraints and Limitations**

- **One parent only**: Multiple classes cannot follow `extends`.
- **Order matters**: The parent must be declared before the child, unless autoloading is used.
- **Final parents**: A class declared `final` cannot be extended.

#### Annotated Code Examples

**Example 1: `extends` in Action**

```php
<?php
class BaseService
{
    public function log(string $message): void
    {
        echo "[LOG] $message\n";
    }
}

class UserService extends BaseService
{
    public function register(string $email): void
    {
        $this->log("Registering $email"); // inherited method
        // registration logic...
    }
}

$service = new UserService();
$service->register("alice@example.com");
// Outputs: [LOG] Registering alice@example.com
?>
```

**Expected Output:**
```
[LOG] Registering alice@example.com
```

**Why This Output:**  
`UserService extends BaseService` inherits the `log()` method, which is called inside `register()` via `$this`.

---

#### Real-World Cases

- **Domain Layers**: `OrderRepository extends BaseRepository` inherits common CRUD methods.
- **Notification Channels**: `SmsChannel extends NotificationChannel` inherits retry logic and logging.
- **Console Commands**: `SendEmailsCommand extends AbstractCommand` inherits argument parsing and output formatting.

---

### 3. Parent Class and Child Class

#### Definitions

**Core Definition**
The **parent class** (also called base class or superclass) is the class being inherited from. The **child class** (subclass or derived class) is the class that inherits from the parent.

**Technical Definition**
The parent class defines the shared members that the child inherits. The child class may add new members, override inherited methods (unless `final`), and access inherited `public` and `protected` members. The child cannot access the parent’s `private` members directly, but may redeclare members with the same name. In PHP, the parent must be declared before the child (unless autoloading is used).

**Beginner-Friendly Explanation**
The parent is the “general” class, and the child is the “specialized” class. The child gets everything the parent has, plus its own unique features. For example, `Animal` is the parent, and `Dog` is the child.

#### Purposes

- To share common state and behavior through the parent.
- To create specialized variants that add or modify behavior.
- To enable polymorphic substitution: a child object can be used wherever the parent type is expected.

#### Syntax Rules and Structure

**General Syntax**
```php
class ParentClass { /* shared members */ }

class ChildClass extends ParentClass
{
    // specialized members
}
```

**Component Breakdown**

| Class | Role |
|-------|------|
| Parent | Defines shared interface and implementation. |
| Child | Inherits, extends, and/or overrides. |

**Constraints and Limitations**

- **Private members** of the parent are invisible to the child.
- **Final members** cannot be overridden by the child.
- **Visibility cannot be narrowed** in the child (only widened).

#### Annotated Code Examples

**Example 1: Parent and Child Relationship**

```php
<?php
class Employee
{
    public function __construct(
        protected string $name,
        protected float $salary
    ) {}

    public function getAnnualCost(): float
    {
        return $this->salary * 12;
    }
}

class Manager extends Employee
{
    public function __construct(
        string $name,
        float $salary,
        private float $bonus
    ) {
        parent::__construct($name, $salary);
    }

    public function getAnnualCost(): float
    {
        return parent::getAnnualCost() + $this->bonus;
    }
}

$manager = new Manager("Alice", 5000.0, 10000.0);
echo $manager->getAnnualCost() . "\n"; // Outputs: 70000
?>
```

**Expected Output:**
```
70000
```

**Why This Output:**  
`Manager` extends `Employee`. It calls `parent::__construct()` to initialize inherited state and overrides `getAnnualCost()` to add a bonus on top of the inherited calculation.

---

#### Real-World Cases

- **E-commerce**: `Product` (parent) → `DigitalProduct`, `PhysicalProduct` (children).
- **Authentication**: `Authenticator` (parent) → `SessionAuthenticator`, `TokenAuthenticator`.
- **Payment Methods**: `PaymentMethod` (parent) → `CreditCard`, `PayPal`, `BankTransfer`.

---

### 4. Method Overriding

#### Definitions

**Core Definition**
Method overriding is the act of redeclaring a method in a child class with the same name as a method in the parent class, providing a new implementation.

**Technical Definition**
When a child class declares a method with the same name as an inherited method, the child’s version overrides the parent’s version for calls made on child objects. PHP requires the overriding method to be **signature-compatible** with the parent: parameter types may be contravariant (wider or identical), and return types must be covariant (narrower or identical). Constructors are exempt from this compatibility requirement. The parent’s version can still be invoked explicitly using `parent::method()`. Methods declared `final` cannot be overridden.

**Beginner-Friendly Explanation**
Overriding means the child class says, “I want to do this method my own way.” The child provides a new version of a method that already exists in the parent. When you call the method on a child object, the child’s version runs. If you need the parent’s original behavior, you can call `parent::method()`.

#### Purposes

- To customize or specialize inherited behavior for the child’s specific needs.
- To implement abstract methods declared by the parent.
- To provide polymorphic behavior: different child classes can implement the same method differently.

#### Syntax Rules and Structure

**General Syntax**
```php
class ParentClass
{
    public function methodName(Type $param): ReturnType { /* ... */ }
}

class ChildClass extends ParentClass
{
    public function methodName(Type $param): ReturnType
    {
        // new implementation
    }
}
```

**Component Breakdown**

| Rule | Description |
|------|-------------|
| Same name | The method name must match the parent’s. |
| Parameter compatibility | Types must be the same or wider (contravariance). |
| Return type compatibility | Types must be the same or narrower (covariance). |
| `final` protection | Final methods cannot be overridden. |

**Constraints and Limitations**

- **LSP behavioral contract**: Signature compatibility is enforced by PHP, but behavioral compatibility (preconditions, postconditions, invariants) is the developer’s responsibility.
- **Constructors are exempt**: A child constructor may have a completely different signature from the parent constructor.
- **Private methods**: A child can declare a method with the same name as a parent’s private method without it being considered an override (the parent’s private method is invisible).

#### Annotated Code Examples

**Example 1: Overriding with Return Type Covariance**

```php
<?php
class Animal
{
    public function create(): Animal
    {
        return new Animal();
    }
}

class Dog extends Animal
{
    public function create(): Dog   // narrower return type — allowed
    {
        return new Dog();
    }
}
?>
```

**Expected Output:**
No output (definitions only). The covariant return type `Dog` is compatible with the parent’s `Animal` return type.

**Why This Output:**  
PHP 7.4+ supports covariant return types. The child’s `create()` returns a more specific type (`Dog`) than the parent’s (`Animal`), which is allowed.

---

**Example 2: LSP Violation (Behavioral)**

```php
<?php
class Rectangle
{
    public function __construct(
        protected int $width,
        protected int $height
    ) {}

    public function setWidth(int $width): void { $this->width = $width; }
    public function setHeight(int $height): void { $this->height = $height; }
    public function area(): int { return $this->width * $this->height; }
}

class Square extends Rectangle
{
    public function setWidth(int $width): void
    {
        $this->width = $width;
        $this->height = $width;   // side effect — violates Rectangle contract
    }

    public function setHeight(int $height): void
    {
        $this->width = $height;
        $this->height = $height;
    }
}

function resizeAndMeasure(Rectangle $rect): int
{
    $rect->setWidth(5);
    $rect->setHeight(10);
    return $rect->area();   // expects 50
}

echo resizeAndMeasure(new Rectangle(0, 0)) . "\n"; // Outputs: 50
echo resizeAndMeasure(new Square(0, 0)) . "\n";    // Outputs: 100 (LSP violation)
?>
```

**Expected Output:**
```
50
100
```

**Why This Output:**  
`Square` overrides `setWidth()` and `setHeight()` in a way that changes the behavior expected from `Rectangle`. Substituting a `Square` where a `Rectangle` is expected breaks the caller’s assumptions, violating the Liskov Substitution Principle. The better design uses a shared `Shape` interface with separate `Rectangle` and `Square` implementations.

---

#### Real-World Cases

- **Notification Channels**: `EmailChannel` and `SmsChannel` both override `send()` with channel-specific logic.
- **Payment Gateways**: `StripeGateway` and `PayPalGateway` override `charge()` with provider-specific API calls.
- **File Parsers**: `JsonParser` and `XmlParser` override `parse()` with format-specific parsing.

---

### 5. `parent::` Keyword

#### Definitions

**Core Definition**
`parent::` is a keyword used inside a child class to access members (methods, properties, constants) of the parent class.

**Technical Definition**
`parent::` uses the scope resolution operator (`::`) to reference the immediate parent class of the class in which the code is written. It is most commonly used to call the parent’s version of an overridden method (`parent::method()`) or to invoke the parent constructor (`parent::__construct()`). The parent constructor is not called implicitly; a child must call `parent::__construct()` explicitly if the parent’s initialization is required.

**Beginner-Friendly Explanation**
`parent::` means “do it the way my parent class would do it.” It lets a child class reuse the parent’s implementation while adding its own logic.

#### Purposes

- To call the parent’s constructor from the child constructor.
- To invoke an overridden parent method while extending its behavior.
- To access parent constants or static members that the child has not overridden.

#### Syntax Rules and Structure

**General Syntax**
```php
parent::methodName($args);
parent::__construct($args);
parent::CONSTANT_NAME;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `parent` | The immediate parent class. |
| `::` | Scope resolution operator. |
| Member | Method, property, or constant to access. |

**Constraints and Limitations**

- **Immediate parent only**: `parent::` refers only to the direct parent; there is no `grandparent::` keyword.
- **Parent constructors are not automatic**: `parent::__construct()` must be called explicitly if needed.
- **Parent destructors**: Similarly, `parent::__destruct()` should be called explicitly, typically as the last statement in the child destructor.

#### Annotated Code Examples

**Example 1: Calling `parent::__construct()`**

```php
<?php
class Animal
{
    public function __construct(protected string $name) {}
}

class Dog extends Animal
{
    public function __construct(string $name, private string $breed)
    {
        parent::__construct($name); // required to initialize $name
    }

    public function describe(): string
    {
        return "$this->name is a $this->breed";
    }
}

$dog = new Dog("Rex", "Labrador");
echo $dog->describe() . "\n"; // Outputs: Rex is a Labrador
?>
```

**Expected Output:**
```
Rex is a Labrador
```

**Why This Output:**  
Without `parent::__construct($name)`, the inherited `$name` property would remain uninitialized. The explicit call ensures the parent’s initialization logic runs.

---

**Example 2: Extending an Overridden Method**

```php
<?php
class Logger
{
    public function log(string $message): void
    {
        echo "[LOG] $message\n";
    }
}

class TimestampLogger extends Logger
{
    public function log(string $message): void
    {
        parent::log(date('Y-m-d H:i:s') . " - $message");
    }
}

$logger = new TimestampLogger();
$logger->log("User logged in");
// Outputs: [LOG] 2026-10-22 12:00:00 - User logged in
?>
```

**Expected Output:**
```
[LOG] 2026-10-22 12:00:00 - User logged in
```

**Why This Output:**  
`TimestampLogger` overrides `log()` but calls `parent::log()` to reuse the parent’s formatting, adding a timestamp prefix.

---

#### Real-World Cases

- **Framework Controllers**: `parent::__construct()` initializes request and response objects.
- **Decorator Pattern**: A decorator calls `parent::method()` and adds behavior before or after.
- **Template Method Pattern**: A subclass override calls `parent::method()` for shared steps and adds its own.

---

### 6. `final` Keyword

#### Definitions

**Core Definition**
The `final` keyword prevents a class from being extended or a method (or, since PHP 8.1, a constant) from being overridden.

**Technical Definition**
When applied to a **class**, `final` prevents any other class from extending it. When applied to a **method**, `final` prevents child classes from overriding it. As of PHP 8.1.0, `final` can also be applied to **class constants**. Properties cannot be declared `final`. As of PHP 8.0.0, `private` methods may not be declared `final` except for constructors.

**Beginner-Friendly Explanation**
`final` means “this is the last word.” A final class cannot be a parent; a final method cannot be changed by a child class. It is used to lock down critical behavior and prevent unintended modification.

#### Purposes

- To protect critical classes or methods from being altered by subclasses.
- To enforce a specific implementation that must remain consistent across all usage.
- To signal design intent: “this class is complete and not designed for extension.”

#### Syntax Rules and Structure

**General Syntax**
```php
final class ClassName { /* ... */ }

class ParentClass
{
    final public function methodName(): ReturnType { /* ... */ }
}
```

**Component Breakdown**

| Application | Effect |
|-------------|--------|
| `final class` | Class cannot be extended. |
| `final public function` | Method cannot be overridden. |
| `final public const` | Constant cannot be overridden (PHP 8.1+). |

**Constraints and Limitations**

- **Cannot be applied to properties**: Only classes, methods, and constants can be `final`.
- **`final` and `abstract` are mutually exclusive**: A method cannot be both.
- **Private `final` methods are not allowed** (except constructors) as of PHP 8.0.

#### Annotated Code Examples

**Example 1: Final Class**

```php
<?php
final class Money
{
    public function __construct(
        public readonly int $amount,
        public readonly string $currency
    ) {}
}

// class ExtendedMoney extends Money {} // Fatal error: cannot extend final class
?>
```

**Expected Output:**
No output (the commented line would cause a fatal error if uncommented).

**Why This Output:**  
`Money` is a value object designed to be complete and not extended. The `final` keyword enforces this design decision at the language level.

---

**Example 2: Final Method**

```php
<?php
class Base
{
    final public function criticalOperation(): string
    {
        return "This must not change.";
    }
}

class Child extends Base
{
    // public function criticalOperation(): string {} // Fatal error
}
?>
```

**Expected Output:**
No output (the commented method would cause a fatal error).

**Why This Output:**  
The `final` method `criticalOperation()` cannot be overridden by `Child`. The parent’s implementation is guaranteed to be used.

---

#### Real-World Cases

- **Security-Critical Methods**: Password hashing, token generation, and encryption methods are often `final` to prevent weakening in subclasses.
- **Value Objects**: `Money`, `Email`, `UserId` are often `final` to preserve immutability and invariants.
- **Framework Internals**: Methods that enforce framework contracts are marked `final` to protect the framework’s behavior.

---

### 7. `abstract` Keyword

#### Definitions

**Core Definition**
The `abstract` keyword declares a class that cannot be instantiated or a method that has no body and must be implemented by concrete subclasses.

**Technical Definition**
An **abstract class** may not be instantiated, and any class containing at least one abstract method must itself be declared abstract. Abstract methods declare only a signature (name, parameters, and optional return type) without an implementation; concrete subclasses must implement them. When inheriting from an abstract class, the child must define all abstract methods with the same or less restricted visibility (e.g., a `protected` abstract method can be implemented as `public`, but not as `private`). Abstract classes may also contain concrete methods with full implementations.

**Beginner-Friendly Explanation**
An abstract class is an incomplete blueprint. It defines what methods must exist, but not how they work. You cannot create an object directly from an abstract class; you must first create a concrete child class that fills in the missing methods. It is like a job description that lists required skills (abstract methods) but does not provide the actual work.

#### Purposes

- To define a template or contract that concrete subclasses must fulfill.
- To share common concrete behavior while forcing specific implementations.
- To prevent instantiation of a class that is not complete enough to be used directly.

#### Syntax Rules and Structure

**General Syntax**
```php
abstract class AbstractClass
{
    abstract public function methodName(Type $param): ReturnType;

    public function concreteMethod(): void { /* ... */ }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `abstract class` | Class cannot be instantiated. |
| `abstract function` | Method has no body; child must implement. |
| Visibility | Abstract methods can be `public` or `protected`, not `private`. |

**Constraints and Limitations**

- **Cannot instantiate an abstract class**: `new AbstractClass()` is a fatal error.
- **All abstract methods must be implemented** by the first concrete subclass.
- **Visibility cannot be restricted**: A `protected` abstract method cannot be implemented as `private` in the child.

#### Annotated Code Examples

**Example 1: Abstract Class with Abstract and Concrete Methods**

```php
<?php
abstract class Shape
{
    abstract public function area(): float;

    public function describe(): string
    {
        return "This shape has an area of " . $this->area();
    }
}

class Circle extends Shape
{
    public function __construct(private float $radius) {}

    public function area(): float
    {
        return M_PI * $this->radius ** 2;
    }
}

$circle = new Circle(5.0);
echo $circle->describe() . "\n"; // Outputs: This shape has an area of 78.53981633974483
?>
```

**Expected Output:**
```
This shape has an area of 78.53981633974483
```

**Why This Output:**  
`Shape` is abstract and declares `area()` abstract. `Circle` implements `area()`. The concrete `describe()` method in `Shape` calls `$this->area()`, which resolves to `Circle::area()` at runtime.

---

#### Real-World Cases

- **Template Method Pattern**: `AbstractController` defines the skeleton of a request-handling algorithm, with abstract steps for authentication and action execution.
- **Payment Gateways**: `AbstractGateway` declares `abstract public function charge(Money $amount): Receipt`.
- **Validation Rules**: `AbstractRule` declares `abstract public function validate(mixed $value): bool`.

---

### 8. Inheritance Best Practices

#### Definitions

**Core Definition**
Inheritance best practices are guidelines for using inheritance correctly, safely, and maintainably, ensuring that hierarchies model true “is-a” relationships and uphold behavioral contracts.

**Technical Definition**
Best practices center on the **Liskov Substitution Principle (LSP)**: a subtype must be substitutable for its parent without changing the correctness of the program. This means preserving preconditions, postconditions, and invariants. Other practices include: **favoring composition over inheritance** when the relationship is not a true “is-a”; **keeping hierarchies shallow**; **programming to interfaces**; **avoiding fragile base classes**; and using `final` to protect critical behavior.

**Beginner-Friendly Explanation**
Best practices for inheritance are the rules of good family design: only use inheritance when the child really is a specialized version of the parent, don’t break the promises the parent makes, and don’t build deep, complicated family trees.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Model “is-a”, not “has-a”** | A `Car` **is a** `Vehicle`; a `Car` **has an** `Engine` (use composition for the latter). |
| **Uphold the Liskov Substitution Principle** | A child must be usable anywhere the parent is used without surprises. |
| **Keep hierarchies shallow** | Deep hierarchies are hard to understand and maintain. |
| **Program to interfaces, not implementations** | Depend on abstractions (interfaces/abstract classes) rather than concrete classes. |
| **Use `final` to protect critical behavior** | Prevent overriding of methods that must remain stable. |
| **Avoid overriding concrete methods unnecessarily** | If a method should not be changed, make it `final`. |
| **Prefer composition when inheritance is not a true “is-a”** | Composition is more flexible and avoids fragile base class problems. |

**Constraints and Limitations**

- **PHP enforces signature compatibility, not behavioral LSP**: Preconditions and postconditions must be validated by the developer.
- **Inheritance creates tight coupling**: Changes to the parent can break children; use `final` and careful design to mitigate this.

#### Annotated Code Examples

**Example 1: LSP-Compliant Design**

```php
<?php
interface Shape
{
    public function area(): int;
}

final class Rectangle implements Shape
{
    public function __construct(
        private int $width,
        private int $height
    ) {}

    public function area(): int
    {
        return $this->width * $this->height;
    }
}

final class Square implements Shape
{
    public function __construct(private int $side) {}

    public function area(): int
    {
        return $this->side ** 2;
    }
}

function totalArea(Shape ...$shapes): int
{
    return array_sum(array_map(fn(Shape $s) => $s->area(), $shapes));
}

echo totalArea(new Rectangle(2, 3), new Square(4)) . "\n"; // Outputs: 22
?>
```

**Expected Output:**
```
22
```

**Why This Output:**  
Both `Rectangle` and `Square` implement the `Shape` interface and honor its contract (`area()` returns the correct area). The `totalArea()` function treats them uniformly without any behavioral surprises, satisfying LSP.

---

#### Real-World Cases

- **Laravel/Symfony**: Framework base classes are carefully designed with `final` methods and interfaces to protect contracts.
- **Domain-Driven Design**: Entities and value objects follow LSP-compliant inheritance hierarchies.
- **Static Analysis**: PHPStan and Psalm can detect signature-level LSP violations, but behavioral violations require careful testing.

---

### 9. Composition over Inheritance

#### Definitions

**Core Definition**
**Composition** is a design technique where a class achieves functionality by containing instances of other classes (its **components**) rather than inheriting from them. “Composition over inheritance” is the principle that composition should be preferred when the relationship is not a true “is-a”.

**Technical Definition**
With composition, a class holds references to other objects (via properties, typically injected through the constructor) and delegates work to them. This models a **“has-a”** relationship: a `Car` **has an** `Engine`; a `User` **has a** `UserDetails` object. Composition avoids the tight coupling and fragile base class problems of inheritance, allows behavior to be changed at runtime by swapping components, and makes testing easier because components can be mocked or stubbed.

**Beginner-Friendly Explanation**
Composition means building a class out of other objects, like building a computer from parts (CPU, RAM, hard drive) rather than inheriting from a “Computer” family tree. If you need a different behavior, you can swap the part instead of rewriting the whole class.

#### Purposes

- To avoid the tight coupling and fragile base class problem of deep inheritance hierarchies.
- To allow behavior to be composed and changed dynamically at runtime.
- To make classes easier to test by injecting mock components.
- To model “has-a” relationships correctly.

#### Syntax Rules and Structure

**General Syntax**
```php
class ComponentA { /* ... */ }
class ComponentB { /* ... */ }

class CompositeClass
{
    public function __construct(
        private ComponentA $a,
        private ComponentB $b
    ) {}

    public function doWork(): void
    {
        $this->a->method();
        $this->b->method();
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Component classes | Independent classes providing behavior. |
| Composite class | Holds component instances as properties. |
| Constructor injection | Components are passed in, not instantiated internally. |
| Delegation | The composite calls methods on its components. |

**Constraints and Limitations**

- **More boilerplate**: Composition requires explicit delegation methods, though this is often outweighed by flexibility.
- **Interface-based design**: Components should be typed as interfaces to allow swapping implementations.

#### Annotated Code Examples

**Example 1: Inheritance vs. Composition**

```php
<?php
// INHERITANCE: "ShoLink is a Link" — tight coupling
class Link
{
    public function create(string $name, string $type): void { /* ... */ }
    public function show(): string { return "Link: $name"; }
}

class ShoLink extends Link { /* inherits everything */ }

// COMPOSITION: "User has a Link" — flexible, testable
class User
{
    public function __construct(private Link $link) {}

    public function getLinkSummary(): string
    {
        return $this->link->show();
    }
}

$link = new Link();
$user = new User($link);
echo $user->getLinkSummary() . "\n"; // Outputs: Link: ...
?>
```

**Expected Output:**
```
Link: ...
```

**Why This Output:**  
The `User` class does not inherit from `Link`; it receives a `Link` instance via constructor injection and delegates to it. This models “has-a” and allows the `Link` implementation to be swapped without changing `User`.

---

**Example 2: Composition with Interfaces for Flexibility**

```php
<?php
interface Logger
{
    public function log(string $message): void;
}

class FileLogger implements Logger
{
    public function log(string $message): void { /* write to file */ }
}

class DatabaseLogger implements Logger
{
    public function log(string $message): void { /* write to database */ }
}

class UserService
{
    public function __construct(private Logger $logger) {}

    public function register(string $email): void
    {
        $this->logger->log("Registering $email");
        // registration logic...
    }
}

// Swap implementations without changing UserService
$service = new UserService(new FileLogger());
$service->register("alice@example.com");
?>
```

**Expected Output:**
No output (logging is an internal side effect).

**Why This Output:**  
`UserService` depends on the `Logger` interface, not a concrete implementation. Any `Logger` can be injected, making the service flexible and testable.

---

#### Real-World Cases

- **Strategy Pattern**: `PaymentProcessor` composes a `PaymentGateway` and can swap between `StripeGateway` and `PayPalGateway`.
- **Dependency Injection**: Services receive repositories, loggers, and mailers via the constructor rather than extending base classes.
- **Decorator Pattern**: A `TimestampLogger` decorates a `Logger` and adds a timestamp, instead of extending a logger class.

---

## References

**PHP Manual – Object Inheritance** - https://www.php.net/manual/en/language.oop5.inheritance.php

**PHP Manual – The Basics (class, extends)** - https://www.php.net/manual/en/language.oop5.basic.php

**PHP Manual – Final Keyword** - https://static.php.net/manual/en/language.oop5.final.php

**PHP Manual – Class Abstraction** - https://svn.php.net/viewvc/phpdoc/en/trunk/language/oop5/abstract.xml?annotate=332727&pathrev=336596

**PHP Manual – Overriding Methods (parent::)** - https://svn.php.net/viewvc/phpdoc/en/branches/function_definitions/language/oop5/basic.xml?sortdir=down&r1=219625&r2=234020&pathrev=331342

**Agent Skills – Liskov Substitution Principle (LSP) for PHP** - https://github.com/voku/agent-skills/blob/main/skills/php-best-practices/rules/solid-lsp.md

**Tsecurity – PHP OOP Part 7: Composition vs Inheritance** - https://tsecurity.de/de/2527896/IT+Programmierung/PHP+OOP+Part-7:+Composition+vs+Inheritance+and+Dependency+Injection/

**Clean Code PHP – Composition over Inheritance** - https://github.com/mts7/clean-code-php

**PHP Internals – Composition over Inheritance (delegating methods discussion)** - https://news-web.php.net/php.internals/101931