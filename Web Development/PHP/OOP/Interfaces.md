Here is a comprehensive research cheat sheet on **PHP Interfaces**, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Interfaces

### Definitions

**Core Definition**
An **interface** is a formal contract in PHP that specifies which methods (and, since PHP 8.4, properties) a class must implement, without defining how those methods work. Any class that implements an interface guarantees that it provides the specified behavior, allowing code to depend on the interface rather than on concrete implementations.

**Technical Definition**
In PHP, an interface is declared with the `interface` keyword, followed by a name and a body containing method signatures (without bodies) and optionally constants. All interface methods must be declared `public`; they are implicitly abstract. A class implements an interface using the `implements` operator and must provide a compatible implementation for every method declared in the interface, or a fatal error occurs. Interfaces share the same namespace as classes, traits, and enums, so names must be unique. A class may implement multiple interfaces by listing them comma-separated after `implements`. Interfaces themselves may extend one or more other interfaces using `extends`. Since PHP 8.4, interfaces may also declare property requirements using `get`/`set` hooks, specifying only public read/write access.

**Beginner-Friendly Explanation**
An interface is like a job description. It lists the skills (methods) a class must have, but it does not tell the class how to perform them. Any class that “applies for the job” (implements the interface) promises to provide all the listed skills. This lets you write code that says, “I don’t care what specific object you are, as long as you can `render()`” or “as long as you can `save()`.” It is a way to make different classes interchangeable by agreeing on a common set of behaviors.

---

### Key Characteristics

- **Pure Contract**: Interfaces contain no implementation (no method bodies), only signatures.
- **All Methods Are Public and Abstract**: Interface methods are implicitly `public` and have no body; they cannot be `private` or `protected`.
- **Multiple Implementation**: A class may implement any number of interfaces, separated by commas.
- **Interface Inheritance**: Interfaces can extend one or more other interfaces using `extends`.
- **Constants**: Interfaces may define constants (implicitly `public`); since PHP 8.1, implementing classes may override them.
- **No Constructors (Recommended)**: Although PHP allows constructors in interfaces, the manual strongly discourages them because they reduce flexibility and are not enforced by inheritance rules.
- **Property Requirements (PHP 8.4+)**: Interfaces may declare public read/write property requirements using hooks.
- **Signature Compatibility**: Implementing methods must be compatible with the interface signature, following covariance and contravariance rules.

---

### Prerequisites

- **PHP 7.4+ Recommended**; **PHP 8.0+** for named arguments and union types; **PHP 8.1+** for overridable interface constants; **PHP 8.4** for interface property requirements.
- **Basic Class and Object Syntax**: Understanding of `class`, `implements`, `new`, and method declarations.
- **Awareness of Visibility and Type Declarations**: Interface methods are always public, and parameter/return types must be compatible.

---

### Related Programming Areas

- **SOLID Principles**: The **Interface Segregation Principle (ISP)** and **Dependency Inversion Principle (DIP)** are directly about interface design.
- **Polymorphism**: Interfaces enable objects of different classes to be used interchangeably through a common contract.
- **Dependency Injection**: Dependencies are typically typed against interfaces, allowing implementations to be swapped.
- **Design Patterns**: Strategy, Adapter, Repository, and many other patterns rely on interfaces.
- **Abstract Classes vs. Interfaces**: Abstract classes provide partial implementations; interfaces define pure contracts.

---

## Core Concepts / Features

---

### 1. Interfaces (General)

#### Definitions

**Core Definition**
An interface is a named contract that specifies method signatures (and, since PHP 8.4, property requirements) without providing implementations.

**Technical Definition**
An interface is declared with the `interface` keyword. It may contain method signatures (without bodies) and constants. All methods are implicitly `public` and abstract. A class that implements an interface must define every method declared in the interface with a compatible signature. Interfaces serve two complementary purposes: allowing objects of different classes to be used interchangeably because they share the same interface, and allowing functions or methods to accept a parameter that conforms to an interface without caring about the concrete class.

**Beginner-Friendly Explanation**
An interface is a promise. If a class says it implements an interface, it promises to provide all the methods listed in that interface. This lets you write code that depends on the promise rather than on a specific class.

#### Purposes

- To define a contract that multiple classes can fulfill, enabling polymorphism.
- To decouple high-level code from concrete implementations (Dependency Inversion).
- To allow objects of different classes to be used interchangeably when they share the same interface.
- To enforce a consistent API across unrelated classes.

#### Syntax Rules and Structure

**General Syntax**
```php
interface InterfaceName
{
    public function methodName(Type $param): ReturnType;
    // constants and property requirements (PHP 8.4+) may also appear
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `interface` | Keyword that begins the interface declaration. |
| `InterfaceName` | Valid PHP label (StudlyCaps per PSR-12). |
| Method signatures | `public function name(...): Type;` — no body. |
| Constants | `const NAME = value;` — implicitly public. |
| Property requirements | PHP 8.4+; `public Type $prop { get; set; }`. |

**Syntax Rules**

- All interface methods must be `public`.
- Methods have no body; they end with a semicolon.
- Interfaces cannot be instantiated directly.
- A class may implement multiple interfaces by listing them after `implements`, comma-separated.
- An interface may extend multiple other interfaces using `extends`.

**Constraints and Limitations**

- **No implementation**: Interfaces cannot contain method bodies (except for abstract classes that partially implement them).
- **No properties (before PHP 8.4)**: Traditional interfaces cannot declare properties; only methods and constants.
- **Constructors are discouraged**: The manual strongly advises against constructors in interfaces because they reduce flexibility and are not enforced by inheritance rules.

#### Annotated Code Examples

**Example 1: A Basic Interface and Implementation**

```php
<?php
// Step 1: Declare the interface
interface Template
{
    public function setVariable($name, $var);
    public function getHtml($template);
}

// Step 2: Implement the interface
class WorkingTemplate implements Template
{
    private array $vars = [];

    public function setVariable($name, $var): void
    {
        $this->vars[$name] = $var;
    }

    public function getHtml($template): string
    {
        foreach ($this->vars as $name => $value) {
            $template = str_replace('{' . $name . '}', $value, $template);
        }
        return $template;
    }
}

// Step 3: Use the object
$template = new WorkingTemplate();
$template->setVariable('name', 'Alice');
echo $template->getHtml('Hello, {name}!') . "\n";
// Outputs: Hello, Alice!
?>
```

**Expected Output:**
```
Hello, Alice!
```

**Why This Output:**  
`WorkingTemplate` implements `Template` and provides both required methods. The object can be used wherever a `Template` is expected. If `getHtml()` were omitted, PHP would throw a fatal error.

**Step-by-Step Setup:**
1. Save as `interface_basic.php`.
2. Run `php interface_basic.php`.
3. The output confirms the interface contract was fulfilled.

---

**Example 2: Incomplete Implementation Causes Fatal Error**

```php
<?php
interface Template
{
    public function setVariable($name, $var);
    public function getHtml($template);
}

// This will cause a fatal error
class BadTemplate implements Template
{
    private array $vars = [];

    public function setVariable($name, $var): void
    {
        $this->vars[$name] = $var;
    }
    // getHtml() is missing
}
?>
```

**Expected Output:**
```
Fatal error: Class BadTemplate contains 1 abstract method and must therefore be declared abstract or implement the remaining methods (Template::getHtml)
```

**Why This Output:**  
A concrete class must implement every method declared in the interface. `BadTemplate` omits `getHtml()`, so PHP treats it as abstract and refuses to instantiate it.

---

#### Real-World Cases

- **Caching Strategies**: `Cacheable` interface with `get()`, `set()`, `delete()`; implemented by `RedisCache`, `FileCache`, `MemoryCache`.
- **Payment Gateways**: `PaymentGateway` interface with `charge()`; implemented by `StripeGateway`, `PayPalGateway`.
- **Renderable Objects**: `Renderable` interface with `render(): string`; implemented by `HtmlView`, `JsonView`, `XmlView`.

---

### 2. The `interface` Keyword

#### Definitions

**Core Definition**
`interface` is the keyword used to declare an interface in PHP.

**Technical Definition**
The `interface` keyword appears at the start of the declaration, followed by the interface name and a body. It signals to the parser that a contract definition follows, not a class implementation. Interface names occupy the same namespace as classes, so they cannot collide with class, trait, or enum names.

**Beginner-Friendly Explanation**
`interface` is the word you write to say, “I am defining a contract now.” It is the counterpart to `class`.

#### Purposes

- To declare a contract that classes can implement.
- To distinguish interfaces from classes at the syntax level.
- To create a named type that can be used in type declarations.

#### Syntax Rules and Structure

**General Syntax**
```php
interface InterfaceName
{
    // method signatures, constants, property requirements
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `interface` | Declaration keyword. |
| `InterfaceName` | Valid PHP label (StudlyCaps). |
| Body `{ ... }` | Contains method signatures and constants. |

**Constraints and Limitations**

- **Cannot be instantiated**: `new InterfaceName()` is a fatal error.
- **Cannot contain properties (before PHP 8.4)**.
- **Cannot contain method bodies**.

#### Annotated Code Examples

**Example 1: Declaring and Using an Interface Type**

```php
<?php
interface Logger
{
    public function log(string $message): void;
}

class FileLogger implements Logger
{
    public function log(string $message): void
    {
        file_put_contents('/tmp/app.log', $message . "\n", FILE_APPEND);
    }
}

function process(Logger $logger): void
{
    $logger->log('Processing started');
}

process(new FileLogger());
?>
```

**Expected Output:**
No output (the log is written to a file).

**Why This Output:**  
`process()` accepts any `Logger`, not a specific implementation. This is the essence of programming to an interface.

---

#### Real-World Cases

- **PSR-3 LoggerInterface**: The PHP-FIG standard defines `LoggerInterface`, implemented by Monolog, Laravel’s logger, and many others.
- **PSR-7 HTTP Message Interfaces**: `RequestInterface`, `ResponseInterface`, `StreamInterface` are implemented by Guzzle, Slim, and Symfony.

---

### 3. The `implements` Keyword

#### Definitions

**Core Definition**
`implements` is the keyword used by a class to declare that it fulfills the contract of one or more interfaces.

**Technical Definition**
The `implements` keyword appears after the class name (and after any `extends` clause) in a class declaration. It is followed by a comma-separated list of interface names. The class must then provide a compatible implementation for every method declared in every listed interface. If it does not, a fatal error occurs (unless the class is declared `abstract`).

**Beginner-Friendly Explanation**
`implements` is the word a class uses to say, “I promise to follow this contract.” It is how a class signs the agreement defined by an interface.

#### Purposes

- To bind a class to one or more interface contracts.
- To enable the class to be used wherever the interface type is expected.
- To enforce that the class provides the required methods.

#### Syntax Rules and Structure

**General Syntax**
```php
class ClassName implements Interface1, Interface2
{
    // implementations of all interface methods
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `class` | Class declaration keyword. |
| `ClassName` | The implementing class. |
| `implements` | Binding keyword. |
| `Interface1, Interface2` | Comma-separated list of interfaces. |

**Syntax Rules**

- `implements` comes after `extends` if both are present: `class C extends P implements I1, I2`.
- All interface methods must be implemented with compatible signatures.
- A class may implement multiple interfaces, but can extend only one parent class.
- An abstract class may implement only a portion of an interface; concrete subclasses must implement the rest.

**Constraints and Limitations**

- **Fatal error on missing methods**: A concrete class that does not implement all interface methods is itself abstract and cannot be instantiated.
- **Parameter name recommendation**: Since PHP 8.0, callers may rely on parameter names in named arguments, so it is strongly recommended to use the same parameter names as the interface.

#### Annotated Code Examples

**Example 1: Implementing an Interface**

```php
<?php
interface Serializable
{
    public function serialize(): string;
}

class User implements Serializable
{
    public function __construct(private string $name) {}

    public function serialize(): string
    {
        return json_encode(['name' => $this->name]);
    }
}

$user = new User('Alice');
echo $user->serialize() . "\n"; // Outputs: {"name":"Alice"}
?>
```

**Expected Output:**
```
{"name":"Alice"}
```

**Why This Output:**  
`User` implements `Serializable` and provides the required `serialize()` method.

---

#### Real-World Cases

- **Laravel**: Controllers implement `ShouldQueue` to indicate queued execution; jobs implement `ShouldBeUnique`.
- **Symfony**: Services implement `EventSubscriberInterface` to register event listeners.
- **Domain-Driven Design**: Repositories implement domain-specific interfaces like `UserRepositoryInterface`.

---

### 4. Interface Methods

#### Definitions

**Core Definition**
Interface methods are the method signatures declared in an interface. They define what a class must be able to do, not how.

**Technical Definition**
All methods declared in an interface are implicitly `public` and abstract (they have no body). An implementing class must provide a method with the same name and a compatible signature. Signature compatibility follows PHP’s variance rules: parameter types may be contravariant (wider or identical), and return types must be covariant (narrower or identical). Constructors may technically be declared in interfaces, but the manual strongly discourages this because constructors are not subject to inheritance signature rules and reduce flexibility.

**Beginner-Friendly Explanation**
Interface methods are the “must-have skills” listed in the contract. The interface says “you must be able to `render()`” or “you must be able to `save()`,” and the implementing class writes the actual code.

#### Purposes

- To specify the behavior that implementing classes must provide.
- To define a uniform API across different classes.
- To enable type-safe polymorphism through method calls on interface-typed variables.

#### Syntax Rules and Structure

**General Syntax**
```php
interface InterfaceName
{
    public function methodName(Type $param): ReturnType;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `public` | Required (explicit or implicit). |
| `function` | Keyword. |
| `methodName` | `camelCase` per PSR-12. |
| Parameters | Typed, with optional defaults (constant expressions). |
| Return type | After `:`. |
| Semicolon | No body. |

**Constraints and Limitations**

- **No bodies**: Interface methods cannot contain implementation code.
- **Must be public**: `protected` and `private` methods are not allowed.
- **Constructors are discouraged**: The manual warns that constructors in interfaces reduce flexibility and are not enforced by inheritance rules.
- **Parameter name compatibility**: Use the same parameter names as the interface to support named arguments.

#### Annotated Code Examples

**Example 1: Interface Methods with Type Declarations**

```php
<?php
declare(strict_types=1);

interface Shape
{
    public function area(): float;
    public function perimeter(): float;
}

final class Circle implements Shape
{
    public function __construct(private float $radius) {}

    public function area(): float
    {
        return M_PI * $this->radius ** 2;
    }

    public function perimeter(): float
    {
        return 2 * M_PI * $this->radius;
    }
}

$circle = new Circle(5.0);
echo $circle->area() . "\n";       // Outputs: 78.53981633974483
echo $circle->perimeter() . "\n";  // Outputs: 31.41592653589793
?>
```

**Expected Output:**
```
78.53981633974483
31.41592653589793
```

**Why This Output:**  
`Circle` implements both methods required by `Shape` with the correct return types (`float`). The interface guarantees that any `Shape` can be asked for its area and perimeter.

---

#### Real-World Cases

- **Repository Interfaces**: `find(int $id): ?Model`, `save(Model $model): void`, `delete(int $id): void`.
- **Notification Channels**: `send(string $recipient, string $message): bool`.
- **File Storage**: `put(string $path, string $contents): void`, `get(string $path): string`.

---

### 5. Multiple Interfaces

#### Definitions

**Core Definition**
Multiple interface implementation allows a single class to fulfill the contracts of more than one interface simultaneously.

**Technical Definition**
A class may implement any number of interfaces by listing them after the `implements` keyword, separated by commas. The class must then provide implementations for every method declared in every listed interface. If two interfaces declare a method with the same name, the implementing method must satisfy the signature compatibility rules for both; covariance and contravariance can be applied. PHP’s manual notes that the order of interfaces can matter in edge cases involving signature resolution.

**Beginner-Friendly Explanation**
A class can make multiple promises at once. For example, a `Page` class might promise to be both `Renderable` (it can be rendered) and `Cacheable` (it can be cached). It must keep both promises by implementing all the required methods.

#### Purposes

- To allow a class to fulfill multiple contracts without multiple inheritance.
- To compose capabilities: a class can be renderable, cacheable, serializable, and more.
- To support the Interface Segregation Principle by combining small, focused interfaces as needed.

#### Syntax Rules and Structure

**General Syntax**
```php
class ClassName implements Interface1, Interface2, Interface3
{
    // implement all methods from all interfaces
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `implements` | Keyword. |
| `Interface1, Interface2` | Comma-separated list. |
| Methods | Must implement all methods from all interfaces. |

**Syntax Rules**

- Interfaces are separated by commas.
- If a class also extends a parent, `extends` must come first: `class C extends P implements I1, I2`.
- If multiple interfaces declare the same method name, the implementation must satisfy all signatures.

**Constraints and Limitations**

- **No ambiguity allowed**: If two interfaces declare a method with incompatible signatures, the class cannot implement both without a fatal error.
- **Parameter name consistency**: Use the same parameter names as the interfaces to avoid issues with named arguments.

#### Annotated Code Examples

**Example 1: Implementing Multiple Interfaces**

```php
<?php
interface Renderable
{
    public function render(): string;
}

interface Cacheable
{
    public function getCacheKey(): string;
}

class Page implements Renderable, Cacheable
{
    public function __construct(private string $title) {}

    public function render(): string
    {
        return "<h1>{$this->title}</h1>";
    }

    public function getCacheKey(): string
    {
        return 'page_' . md5($this->title);
    }
}

$page = new Page('Welcome');
echo $page->render() . "\n";       // Outputs: <h1>Welcome</h1>
echo $page->getCacheKey() . "\n";  // Outputs: page_...
?>
```

**Expected Output:**
```
<h1>Welcome</h1>
page_...
```

**Why This Output:**  
`Page` implements both `Renderable` and `Cacheable`, providing both `render()` and `getCacheKey()`. It can be used wherever either interface is expected.

---

**Example 2: Interface Inheritance Combined with Multiple Implementation**

```php
<?php
interface A { public function foo(): void; }
interface B { public function bar(): void; }
interface C extends A, B { public function baz(): void; }

class D implements C
{
    public function foo(): void { echo "foo\n"; }
    public function bar(): void { echo "bar\n"; }
    public function baz(): void { echo "baz\n"; }
}

$d = new D();
$d->foo(); // Outputs: foo
$d->bar(); // Outputs: bar
$d->baz(); // Outputs: baz
?>
```

**Expected Output:**
```
foo
bar
baz
```

**Why This Output:**  
Interface `C` extends `A` and `B`, so any class implementing `C` must implement `foo()`, `bar()`, and `baz()`. `D` does so.

---

#### Real-World Cases

- **Page Objects**: Implementing `Renderable`, `Cacheable`, and `Serializable`.
- **Domain Repositories**: A full-featured repository implements `FindsUsers`, `SavesUsers`, `PaginatesUsers`, and `SearchesUsers` (small, segregated interfaces).
- **HTTP Clients**: Implementing `ClientInterface`, `DebugInterface`, and `ConfigurableInterface`.

---

### 6. Interface Constants

#### Definitions

**Core Definition**
Interface constants are fixed values declared inside an interface. They are implicitly `public` and are available to any class that implements the interface.

**Technical Definition**
Constants in interfaces are declared with the `const` keyword. They behave like class constants but, prior to PHP 8.1.0, could not be overridden by a class or interface that inherited them. Since PHP 8.1.0, interface constants are overridable by implementing classes (unless declared `final`). Interface constants can be accessed via `InterfaceName::CONSTANT` or, from within an implementing class, via `self::CONSTANT` or `static::CONSTANT`.

**Beginner-Friendly Explanation**
Interface constants are fixed values that come as part of the contract. They are like shared rules that all implementing classes agree on—for example, a maximum size or a status code.

#### Purposes

- To define shared constant values that are part of the interface contract.
- To avoid magic numbers and strings in implementing classes.
- To provide a single source of truth for values that all implementations must respect.

#### Syntax Rules and Structure

**General Syntax**
```php
interface InterfaceName
{
    const CONSTANT_NAME = value;
    // or, since PHP 8.1
    final const CONSTANT_NAME = value;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `const` | Keyword. |
| `CONSTANT_NAME` | `UPPER_SNAKE_CASE` per convention. |
| `= value` | Constant expression. |
| `final` | PHP 8.1+; prevents overriding. |

**Constraints and Limitations**

- **Overriding**: Prior to PHP 8.1.0, implementing classes could not override interface constants. Since PHP 8.1.0, they can, unless the constant is `final`.
- **Constant expression**: The value must be a constant expression.
- **Visibility**: Interface constants are implicitly `public`; no other visibility is allowed.

#### Annotated Code Examples

**Example 1: Interface Constant Access and Overriding**

```php
<?php
interface Status
{
    const PENDING = 'pending';
    const APPROVED = 'approved';
    const REJECTED = 'rejected';
}

class Order implements Status
{
    public string $status = self::PENDING;

    public function approve(): void
    {
        $this->status = self::APPROVED;
    }
}

$order = new Order();
echo $order->status . "\n"; // Outputs: pending
$order->approve();
echo $order->status . "\n"; // Outputs: approved
echo Status::REJECTED . "\n"; // Outputs: rejected
?>
```

**Expected Output:**
```
pending
approved
rejected
```

**Why This Output:**  
`Order` implements `Status` and uses the interface constants `self::PENDING` and `self::APPROVED`. The constants are also accessible directly via `Status::REJECTED`.

---

**Example 2: Overriding an Interface Constant (PHP 8.1+)**

```php
<?php
interface A
{
    const B = 'Interface constant';
}

class B implements A
{
    const B = 'Class constant'; // Allowed since PHP 8.1.0
}

echo A::B . "\n"; // Outputs: Interface constant
echo B::B . "\n"; // Outputs: Class constant
?>
```

**Expected Output:**
```
Interface constant
Class constant
```

**Why This Output:**  
Prior to PHP 8.1.0, this would have caused a fatal error. Since PHP 8.1.0, implementing classes may override interface constants.

---

#### Real-World Cases

- **HTTP Status Codes**: `HttpStatusInterface` with `const OK = 200`, `const NOT_FOUND = 404`.
- **Order States**: `OrderStatusInterface` with `const PENDING`, `const SHIPPED`, `const DELIVERED`.
- **Configuration Limits**: `LimitsInterface` with `const MAX_UPLOAD_SIZE`, `const TIMEOUT_SECONDS`.

---

### 7. Interface Best Practices

#### Definitions

**Core Definition**
Interface best practices are guidelines for designing interfaces that are cohesive, minimal, and correctly used, following the **Interface Segregation Principle (ISP)** and **Dependency Inversion Principle (DIP)** from SOLID.

**Technical Definition**
Best practices for PHP interfaces include: **keeping interfaces small and focused** (ISP)—no “kitchen-sink” contracts; **avoiding constructors** in interfaces; **using consistent parameter names** to support named arguments; **preferring interfaces over abstract classes** when there is no shared implementation; **creating interfaces at architectural boundaries** for dependency inversion; **not creating `FooInterface + Foo` pairs** when only one implementation will ever exist; and **using `final` on interface constants** where overriding is not desired.

**Beginner-Friendly Explanation**
Best practices for interfaces are the rules of good contract design: keep contracts small and specific, don’t force classes to implement things they don’t need, and use interfaces where they genuinely make code more flexible—not just for the sake of having an interface.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Keep interfaces small and focused (ISP)** | If a class is forced to implement methods it does not use (throwing exceptions or leaving them empty), the interface is too wide. Split it. |
| **Avoid constructors in interfaces** | The manual strongly discourages them; they reduce flexibility and are not enforced by inheritance rules. |
| **Use consistent parameter names** | Named arguments (PHP 8.0+) mean callers may rely on interface parameter names; use the same names in implementations. |
| **Create interfaces at architectural boundaries** | Define interfaces between layers (application ↔ infrastructure) and around third-party SDKs to decouple your code. |
| **Avoid ritual interfaces** | Do not create `FooInterface` with a single implementation and no foreseeable need for substitution; this adds noise. |
| **Use `final` on interface constants** | If implementing classes should not override a constant, declare it `final` (PHP 8.1+). |
| **Prefer composition and small interfaces over large ones** | Combine small interfaces as needed rather than creating one large interface with many unrelated methods. |

**Constraints and Limitations**

- **ISP is about client needs**: Split interfaces by cohesive use cases (read, write, search), not by individual methods.
- **Over-segregation is also a smell**: One interface per method is not the goal; group by cohesive capability.

#### Annotated Code Examples

**Example 1: ISP Violation vs. ISP-Compliant Design**

```php
<?php
// BAD: Fat interface — Robot must lie about methods it cannot implement
interface WorkerInterface
{
    public function work(): void;
    public function eat(): void;
    public function sleep(): void;
    public function attendMeeting(): void;
}

class Robot implements WorkerInterface
{
    public function work(): void { /* real logic */ }
    public function eat(): void { throw new LogicException("Robots don't eat"); }
    public function sleep(): void { throw new LogicException("Robots don't sleep"); }
    public function attendMeeting(): void { throw new LogicException("Robots don't attend meetings"); }
}
```
```php
// GOOD: Segregated interfaces
interface Workable
{
    public function work(): void;
}

interface HumanWorker extends Workable
{
    public function eat(): void;
    public function sleep(): void;
    public function attendMeeting(): void;
}

class Robot implements Workable
{
    public function work(): void { /* real logic */ }
}

class Employee implements HumanWorker
{
    public function work(): void { /* work logic */ }
    public function eat(): void { /* lunch break */ }
    public function sleep(): void { /* after hours */ }
    public function attendMeeting(): void { /* meeting */ }
}
?>
```

**Expected Output:**
No output (definitions only). The “good” design avoids forcing `Robot` to implement methods it cannot fulfill.

**Why This Output:**  
The fat `WorkerInterface` violates ISP because `Robot` is forced to implement `eat()`, `sleep()`, and `attendMeeting()`, none of which apply to it. The segregated interfaces allow `Robot` to implement only `Workable`, while `Employee` implements the richer `HumanWorker` contract.

---

#### Real-World Cases

- **Repository Interfaces**: Split into `FindsUsers`, `SavesUsers`, `PaginatesUsers`, `SearchesUsers` rather than one large `UserRepositoryInterface`.
- **Laravel Contracts**: Laravel’s `Illuminate\Contracts` namespace uses small, focused interfaces (e.g., `Cache`, `Queue`, `Mailer`).
- **PSR Standards**: PSR-3 (`LoggerInterface`), PSR-7 (HTTP messages), and PSR-11 (container) are all small, focused contracts.

---

## References

**PHP Manual – Object Interfaces** - https://static.php.net/manual/en/language.oop5.interfaces.php

**PHP Manual – Interfaces (implements, multiple interfaces)** - https://static.php.net/manual/en/language.oop5.interfaces.php#language.oop5.interfaces.implements

**PHP Manual – Interface Constants** - https://static.php.net/manual/en/language.oop5.interfaces.php#language.oop5.interfaces.constants

**PHP Manual – Interface Properties (PHP 8.4)** - https://static.php.net/manual/en/language.oop5.interfaces.php#language.oop5.interfaces.properties

**PHP Manual – Variance (covariance and contravariance)** - https://static.php.net/manual/en/language.oop5.variance.php

**PHP RFC: Interface Default Methods (context)** - https://wiki.php.net/rfc/interface_default_methods

**PHP RFC: Interface Constants Overriding (PHP 8.1)** - https://wiki.php.net/rfc/interface_constants

**PHP Best Practices – Interface Segregation Principle (Agent Skills)** - https://github.com/voku/agent-skills/blob/main/skills/php-best-practices/rules/solid-isp.md

**IxDF – SOLID Principles for PHP** - https://handbook.ixdf.org/library/backend/SOLID.html

**PHP: Object Interfaces (W3Schools overview)** - https://www.w3schools.com/php/php_oop_interfaces.asp

**PHP: Interfaces (GeeksforGeeks)** - https://www.geeksforgeeks.org/php-interfaces/