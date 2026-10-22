Here is a comprehensive research cheat sheet on **PHP Abstract Classes**, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Abstract Classes

### Definitions

**Core Definition**
An **abstract class** is a class that cannot be instantiated directly and is designed to be extended by concrete subclasses. It may declare **abstract methods** (signatures without bodies) that subclasses must implement, and it may also provide **concrete methods** (full implementations) that subclasses inherit and can use as-is or override.

**Technical Definition**
In PHP, an abstract class is declared with the `abstract` keyword before the `class` keyword. Any class that contains at least one abstract method must itself be declared abstract; conversely, an abstract class does not need to contain any abstract methods. Abstract methods declare only the method's signature and visibility (`public` or `protected`), and cannot define an implementation. When a concrete class inherits from an abstract class, it must implement every abstract method declared by the parent, following normal inheritance and signature-compatibility rules. As of PHP 8.4, abstract classes may also declare abstract properties (public or protected) that impose requirements on the child's property implementation.

**Beginner-Friendly Explanation**
An abstract class is like a **partially built house**. It already has some finished rooms (concrete methods) that you can use as-is, but it also has unfinished rooms (abstract methods) that you must complete before anyone can move in. You cannot live in a partially built house (you cannot instantiate an abstract class), but you can use it as the foundation for a finished house (a concrete subclass). Abstract classes are perfect when related classes share some common behavior but also need to enforce that each subclass provides certain specific behavior.

---

### Key Characteristics

- **Cannot Be Instantiated**: `new AbstractClass()` is a fatal error; the class must be extended first.
- **May Contain Abstract and Concrete Members**: Abstract classes can mix abstract method declarations with fully implemented concrete methods.
- **Forces Implementation**: Any concrete subclass must implement all abstract methods inherited from the parent.
- **Single Inheritance**: A class can extend only one abstract class (PHP does not support multiple inheritance), but it can implement multiple interfaces simultaneously.
- **Signature Compatibility**: Implementing methods must follow PHP's usual inheritance and signature compatibility rules (variance).
- **Abstract Properties (PHP 8.4+)**: Abstract classes may declare public or protected abstract properties that impose get/set requirements on subclasses.
- **`abstract` and `final` Are Mutually Exclusive**: A class cannot be both abstract and final; PHP reports a compile-time error.

---

### Prerequisites

- **PHP 5+** for abstract classes and methods; **PHP 8.4+** for abstract properties.
- **Basic Class and Object Syntax**: Understanding of `class`, `extends`, `new`, `$this`, properties, and methods.
- **Awareness of Inheritance and Visibility**: `public` and `protected` visibility determines what subclasses can access and implement.

---

### Related Programming Areas

- **Interfaces**: Pure contracts with no implementation; abstract classes are the “partial implementation” counterpart.
- **Template Method Pattern**: Abstract classes are the natural home for this pattern, where a concrete method orchestrates steps implemented by abstract methods.
- **SOLID Principles**: The Open/Closed Principle (open for extension, closed for modification) and Liskov Substitution Principle guide abstract class design.
- **Design Patterns**: Factory Method, Strategy, and many creational patterns rely on abstract classes or interfaces.

---

## Core Concepts / Features

---

### 1. Abstract Classes

#### Definitions

**Core Definition**
An abstract class is a class that cannot be instantiated and serves as a foundation for concrete subclasses.

**Technical Definition**
An abstract class is declared with the `abstract` keyword. It may contain abstract methods (no body) and concrete methods (with body). It may also contain properties and constants. A concrete subclass that extends an abstract class must implement all inherited abstract methods unless the subclass is itself declared abstract. Abstract classes are often used to share common functionality while enforcing a contract on subclasses.

**Beginner-Friendly Explanation**
An abstract class is a template that is not complete enough to be used on its own. It provides some ready-made behavior and leaves other behavior for subclasses to fill in.

#### Purposes

- To provide a reusable foundation that shares concrete behavior among related classes.
- To enforce a contract on subclasses by declaring abstract methods they must implement.
- To model a general concept (e.g., `Shape`, `Animal`, `PaymentMethod`) that is not concrete enough to exist on its own.
- To support the Template Method Pattern, where a concrete method defines an algorithm’s skeleton and defers some steps to abstract methods.

#### Syntax Rules and Structure

**General Syntax**
```php
abstract class AbstractClassName
{
    // abstract method (no body)
    abstract public function abstractMethod(Type $param): ReturnType;

    // concrete method (with body)
    public function concreteMethod(): ReturnType
    {
        // implementation
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `abstract class` | Declares the class as abstract (cannot be instantiated). |
| `abstract function` | Declares a method signature without a body. |
| Concrete method | A normal method with a body that subclasses inherit. |
| Abstract property (PHP 8.4+) | `abstract public string $prop { get; }` — imposes a property requirement. |

**Syntax Rules**

- Any class containing at least one abstract method must be declared abstract.
- An abstract class may contain zero abstract methods (it can be abstract purely to prevent instantiation).
- Abstract methods cannot be `private`; they must be `public` or `protected`.
- A subclass must implement all abstract methods with compatible signatures (variance rules apply).
- `abstract` and `final` cannot be combined on the same class or method.

**Constraints and Limitations**

- **Cannot be instantiated**: `new AbstractClass()` is a fatal error.
- **Single inheritance**: A class can extend only one abstract class.
- **Visibility can be relaxed**: A `protected` abstract method can be implemented as `public`, but not as `private`.
- **Abstract properties require hooks**: Abstract property declarations use `get`/`set` hook syntax (PHP 8.4+) and must be satisfied by compatible property declarations or hooks.

#### Annotated Code Examples

**Example 1: Basic Abstract Class with Abstract and Concrete Methods**

```php
<?php
// Step 1: Declare the abstract class
abstract class AbstractClass
{
    // Step 2: Abstract method — subclass MUST implement
    abstract protected function getValue(): string;

    // Step 3: Abstract method with parameters
    abstract protected function prefixValue(string $prefix): string;

    // Step 4: Concrete method — subclasses inherit this
    public function printOut(): void
    {
        print $this->getValue() . "\n";
    }
}

// Step 5: First concrete subclass
class ConcreteClass1 extends AbstractClass
{
    protected function getValue(): string
    {
        return "ConcreteClass1";
    }

    public function prefixValue(string $prefix): string
    {
        return "{$prefix}ConcreteClass1";
    }
}

// Step 6: Second concrete subclass
class ConcreteClass2 extends AbstractClass
{
    public function getValue(): string
    {
        return "ConcreteClass2";
    }

    public function prefixValue(string $prefix): string
    {
        return "{$prefix}ConcreteClass2";
    }
}

// Step 7: Use the concrete subclasses
$class1 = new ConcreteClass1();
$class1->printOut();                        // Outputs: ConcreteClass1
echo $class1->prefixValue('FOO_') . "\n";  // Outputs: FOO_ConcreteClass1

$class2 = new ConcreteClass2();
$class2->printOut();                        // Outputs: ConcreteClass2
echo $class2->prefixValue('FOO_') . "\n";  // Outputs: FOO_ConcreteClass2
?>
```

**Expected Output:**
```
ConcreteClass1
FOO_ConcreteClass1
ConcreteClass2
FOO_ConcreteClass2
```

**Why This Output:**  
`AbstractClass` declares two abstract methods that both subclasses must implement. It also provides a concrete `printOut()` method that calls `$this->getValue()`. Because PHP uses late static binding for method calls, `printOut()` invokes the subclass’s `getValue()` implementation at runtime. This is the Template Method Pattern in action.

**Step-by-Step Setup:**
1. Save as `abstract_basic.php`.
2. Run `php abstract_basic.php`.
3. The output confirms both subclasses implemented the abstract methods and inherited the concrete one.

---

**Example 2: Abstract Class with No Abstract Methods**

```php
<?php
abstract class BaseService
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
        $this->log("Registering $email");
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
`BaseService` is abstract but contains only a concrete method. It is abstract to prevent direct instantiation (a `BaseService` alone is not useful), but it provides shared logging behavior to subclasses. This is a valid and common use of abstract classes.

---

#### Real-World Cases

- **Template Method Pattern**: An `AbstractReportGenerator` defines the skeleton of report generation (`generate()` calls `fetchData()`, `formatData()`, and `output()`), with abstract methods for the steps that vary by report type.
- **Framework Base Controllers**: `AbstractController` provides request handling, templating, and dependency injection infrastructure, with abstract methods for the specific action logic.
- **Payment Gateways**: `AbstractGateway` provides common retry logic, logging, and HTTP client setup, with abstract methods `charge()` and `refund()` that concrete gateways (Stripe, PayPal) must implement.

---

### 2. The `abstract` Keyword

#### Definitions

**Core Definition**
`abstract` is the keyword used to declare abstract classes and abstract methods in PHP.

**Technical Definition**
The `abstract` keyword can be applied to classes (preventing instantiation) and methods (declaring a signature without a body). When applied to a method, the method must reside in an abstract class. A class containing at least one abstract method must itself be abstract. As of PHP 8.4, `abstract` can also be applied to properties.

**Beginner-Friendly Explanation**
`abstract` is the word that tells PHP, “This thing is not finished yet—subclasses must complete it.”

#### Purposes

- To declare that a class cannot be instantiated and must be extended.
- To declare that a method has no implementation and must be implemented by subclasses.
- To signal design intent: “this is a foundation, not a finished product.”

#### Syntax Rules and Structure

**General Syntax**
```php
abstract class ClassName
{
    abstract public function methodName(Type $param): ReturnType;
}
```

**Component Breakdown**

| Application | Effect |
|-------------|--------|
| `abstract class` | Class cannot be instantiated. |
| `abstract function` | Method has no body; subclass must implement. |
| `abstract public string $prop { get; }` | PHP 8.4+; imposes a property requirement. |

**Constraints and Limitations**

- **`abstract` and `final` are mutually exclusive** on classes and methods.
- **Abstract methods cannot be `private`**.
- **Abstract classes cannot be instantiated**, but can contain static methods.

#### Annotated Code Examples

**Example 1: Abstract Keyword in Class and Method**

```php
<?php
abstract class Shape
{
    abstract public function area(): float;

    public function describe(): string
    {
        return "Area: " . $this->area();
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
echo $circle->describe() . "\n"; // Outputs: Area: 78.53981633974483
?>
```

**Expected Output:**
```
Area: 78.53981633974483
```

**Why This Output:**  
`Shape` is abstract and declares `area()` abstract. `Circle` implements it. The concrete `describe()` method in `Shape` calls `$this->area()`, which resolves to `Circle::area()` at runtime.

---

#### Real-World Cases

- **Domain Models**: `abstract class Entity` with an abstract `getId(): int` method that all entities must implement.
- **Value Objects**: `abstract class ValueObject` with concrete equality logic and abstract `equals()` methods.
- **Console Commands**: `abstract class AbstractCommand` with abstract `execute()` and concrete `configure()`.

---

### 3. Abstract Methods

#### Definitions

**Core Definition**
An abstract method is a method declaration without a body. It specifies the method’s name, parameters, and return type, but leaves the implementation to subclasses.

**Technical Definition**
Abstract methods are declared with the `abstract` keyword and end with a semicolon instead of a body. They must be `public` or `protected` (not `private`). The containing class must be abstract. When a concrete subclass inherits an abstract method, it must provide an implementation with a compatible signature: parameter types may be contravariant (wider or identical), and return types must be covariant (narrower or identical). Abstract methods cannot be `final`.

**Beginner-Friendly Explanation**
An abstract method is a **requirement** written into the blueprint: “Any class that extends this class **must** provide this method.” It tells subclasses what they must be able to do, but not how to do it.

#### Purposes

- To force subclasses to provide specific behavior.
- To define a contract that cannot be implemented in the abstract class because it depends on subclass-specific details.
- To enable the Template Method Pattern by leaving “hook” methods for subclasses.

#### Syntax Rules and Structure

**General Syntax**
```php
abstract public function methodName(Type $param): ReturnType;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `abstract` | Required keyword. |
| `public` or `protected` | Visibility (not `private`). |
| Parameters | Typed, with optional defaults. |
| Return type | After `:`. |
| Semicolon | No body. |

**Constraints and Limitations**

- **Must be in an abstract class**.
- **Cannot be `private`**.
- **Cannot be `final`**.
- **Parameter names should match the interface/abstract declaration** for named-argument compatibility.

#### Annotated Code Examples

**Example 1: Abstract Method with Optional Parameters in Subclass**

```php
<?php
abstract class AbstractClass
{
    // Abstract method: only the required arguments are declared
    abstract protected function prefixName(string $name): string;
}

class ConcreteClass extends AbstractClass
{
    // Child may add optional parameters not present in the parent signature
    public function prefixName(string $name, string $separator = "."): string
    {
        if ($name === "Pacman") {
            $prefix = "Mr";
        } elseif ($name === "Pacwoman") {
            $prefix = "Mrs";
        } else {
            $prefix = "";
        }
        return "{$prefix}{$separator} {$name}";
    }
}

$class = new ConcreteClass();
echo $class->prefixName("Pacman") . "\n";   // Outputs: Mr. Pacman
echo $class->prefixName("Pacwoman") . "\n"; // Outputs: Mrs. Pacwoman
?>
```

**Expected Output:**
```
Mr. Pacman
Mrs. Pacwoman
```

**Why This Output:**  
The abstract method `prefixName(string $name)` declares only one required parameter. The subclass is allowed to add an **optional** parameter (`$separator = "."`) because it does not violate signature compatibility—callers can still call the method with a single argument.

---

#### Real-World Cases

- **Payment Gateways**: `abstract public function charge(Money $amount, PaymentMethod $method): Receipt;` — each gateway implements the actual API call.
- **File Parsers**: `abstract public function parse(string $contents): array;` — JSON, XML, and CSV parsers implement format-specific logic.
- **Notification Channels**: `abstract public function send(string $recipient, string $message): bool;` — email, SMS, and push channels implement delivery.

---

### 4. Concrete Methods

#### Definitions

**Core Definition**
A concrete method is a method with a full implementation (body) inside an abstract or concrete class. Subclasses inherit concrete methods and can use them directly or override them (unless marked `final`).

**Technical Definition**
Concrete methods are declared without the `abstract` keyword and contain a body enclosed in curly braces. In an abstract class, concrete methods provide shared behavior that all subclasses inherit. A concrete method can call abstract methods declared in the same class (the Template Method Pattern), because the abstract methods will be implemented by the concrete subclass at runtime. Concrete methods may be `final` to prevent overriding.

**Beginner-Friendly Explanation**
Concrete methods are the **finished rooms** in the partially built house. They are ready to use as-is. Subclasses can move in and use them without any extra work.

#### Purposes

- To share common behavior across all subclasses without duplicating code.
- To define an algorithm’s skeleton (Template Method) that calls abstract steps implemented by subclasses.
- To provide default implementations that subclasses can override if needed.

#### Syntax Rules and Structure

**General Syntax**
```php
public function methodName(Type $param): ReturnType
{
    // implementation
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `public` / `protected` | Visibility. |
| `function` | Keyword. |
| Body `{ ... }` | Implementation code. |
| `final` | Optional; prevents overriding. |

**Constraints and Limitations**

- **Concrete methods can call abstract methods**: The abstract methods will be resolved to the subclass’s implementation at runtime.
- **`final` concrete methods cannot be overridden** by subclasses.
- **Concrete methods are inherited** by subclasses and can be overridden unless `final`.

#### Annotated Code Examples

**Example 1: Template Method Pattern with Concrete and Abstract Methods**

```php
<?php
abstract class ReportGenerator
{
    // Template method — concrete, defines the algorithm skeleton
    final public function generate(): string
    {
        $data = $this->fetchData();       // abstract step
        $formatted = $this->formatData($data); // abstract step
        return $this->output($formatted); // abstract step
    }

    abstract protected function fetchData(): array;
    abstract protected function formatData(array $data): string;
    abstract protected function output(string $formatted): string;
}

class CsvReport extends ReportGenerator
{
    protected function fetchData(): array
    {
        return [['name' => 'Alice', 'age' => 30], ['name' => 'Bob', 'age' => 25]];
    }

    protected function formatData(array $data): string
    {
        $lines = ["name,age"];
        foreach ($data as $row) {
            $lines[] = "{$row['name']},{$row['age']}";
        }
        return implode("\n", $lines);
    }

    protected function output(string $formatted): string
    {
        return $formatted . "\n";
    }
}

$report = new CsvReport();
echo $report->generate();
// Outputs:
// name,age
// Alice,30
// Bob,25
?>
```

**Expected Output:**
```
name,age
Alice,30
Bob,25
```

**Why This Output:**  
`ReportGenerator::generate()` is a **final concrete method** that defines the algorithm skeleton (fetch → format → output) and calls three abstract methods. `CsvReport` implements those abstract methods. The `final` keyword on `generate()` ensures subclasses cannot change the algorithm structure, only the steps. This is the Template Method Pattern.

---

#### Real-World Cases

- **Framework Base Classes**: `AbstractController::render()` is a concrete method that uses abstract `getViewData()` and `getTemplate()` methods implemented by subclasses.
- **HTTP Clients**: `AbstractHttpClient::request()` handles retries, logging, and error mapping, while abstract `doRequest()` is implemented by Guzzle, cURL, or stream wrappers.
- **Validation**: `AbstractRule::validate()` provides common null-checking and type-checking, while abstract `passes()` implements the rule-specific logic.

---

### 5. Abstract Class vs. Interface

#### Definitions

**Core Definition**
An **abstract class** is a partially implemented class that can contain both abstract methods (no body) and concrete methods (with body). An **interface** is a pure contract that contains only method signatures (no bodies) and constants. A class can extend only one abstract class but can implement many interfaces.

**Technical Definition**
Abstract classes support single inheritance and can provide shared implementation, state (properties), and concrete methods. Interfaces define a contract without implementation (though they may contain constants) and support multiple implementation. Since PHP 8.4, interfaces can declare property requirements (get/set hooks), narrowing the gap slightly, but interfaces still cannot provide implementation for methods. Abstract classes are used when subclasses share common code; interfaces are used when unrelated classes must adhere to a common contract.

**Beginner-Friendly Explanation**
- **Abstract class** = a **partially built house** (some rooms finished, some unfinished).
- **Interface** = a **job description** (lists required skills, but provides no training).
- A class can **extend one** abstract class but **implement many** interfaces.

#### Purposes

- To choose the right tool: abstract classes for shared implementation + contract; interfaces for pure contracts across unrelated types.
- To combine both: an abstract class can implement an interface and provide a partial implementation, letting subclasses fill in the rest.

#### Syntax Rules and Structure (Comparison)

| Aspect | Abstract Class | Interface |
|--------|---------------|-----------|
| **Instantiation** | Cannot be instantiated | Cannot be instantiated |
| **Method bodies** | May contain concrete methods (with body) | No method bodies (signatures only) |
| **Properties** | May contain properties | Cannot contain properties (before PHP 8.4; 8.4+ property requirements) |
| **Inheritance** | Single (`extends` one class) | Multiple (`implements` many interfaces) |
| **Constructor** | May contain a constructor | Discouraged (not enforced) |
| **Use case** | Shared implementation + contract | Pure contract across unrelated classes |
| **Naming convention** | Often `Abstract` prefix | Often `Interface` suffix |

**Constraints and Limitations**

- **Abstract classes are more coupled**: Subclasses inherit all concrete methods and properties, which can make refactoring harder.
- **Interfaces are more flexible**: A class can implement any number of interfaces, enabling type composition across unrelated hierarchies.

#### Annotated Code Examples

**Example 1: Abstract Class and Interface Combined**

```php
<?php
interface Renderable
{
    public function render(): string;
}

abstract class AbstractView implements Renderable
{
    public function __construct(protected array $data = []) {}

    // Concrete helper method shared by all views
    protected function escape(string $value): string
    {
        return htmlspecialchars($value, ENT_QUOTES, 'UTF-8');
    }

    // render() remains abstract — subclasses must implement
}

class HtmlView extends AbstractView
{
    public function render(): string
    {
        $name = $this->escape($this->data['name'] ?? 'Guest');
        return "<h1>Hello, $name</h1>";
    }
}

$view = new HtmlView(['name' => '<script>alert(1)</script>']);
echo $view->render() . "\n";
// Outputs: <h1>Hello, &lt;script&gt;alert(1)&lt;/script&gt;</h1>
?>
```

**Expected Output:**
```
<h1>Hello, &lt;script&gt;alert(1)&lt;/script&gt;</h1>
```

**Why This Output:**  
`AbstractView` implements the `Renderable` interface but does not implement `render()`. It provides a concrete `escape()` helper method that subclasses can use. `HtmlView` extends `AbstractView` and implements `render()`, leveraging the inherited `escape()` method. This design combines the contract enforcement of an interface with the shared implementation of an abstract class.

---

#### Real-World Cases

- **Framework Architecture**: `AbstractController implements ControllerInterface` — the interface defines the contract, and the abstract class provides shared request/response handling.
- **Repository Pattern**: `AbstractRepository implements RepositoryInterface` — the abstract class provides common CRUD operations, and concrete repositories add entity-specific queries.
- **Payment Processing**: `AbstractGateway implements PaymentGatewayInterface` — the abstract class handles retries and logging, and concrete gateways implement the actual API calls.

---

### 6. Abstract Class Best Practices

#### Definitions

**Core Definition**
Best practices for abstract classes are guidelines for designing abstract classes that are cohesive, correctly used, and maintainable, following SOLID principles and established PHP conventions.

**Technical Definition**
Best practices include: **naming abstract classes with an `Abstract` prefix** (e.g., `AbstractRepository`) per PHP-FIG naming conventions; **keeping abstract classes focused** on a single responsibility; **using abstract methods only when subclass-specific behavior is required**; **providing concrete methods for shared behavior** (Template Method Pattern); **marking public methods of abstract classes as `final`** to enforce API encapsulation; **preferring composition over inheritance** when the “is-a” relationship is not genuine; and **avoiding deep abstract hierarchies** that become hard to understand.

**Beginner-Friendly Explanation**
Best practices for abstract classes are the rules of good foundation design: name them clearly, keep them focused, provide concrete help where you can, and don’t force subclasses to inherit things they don’t need.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Prefix abstract classes with `Abstract`** | PHP-FIG naming convention: `AbstractRepository`, `AbstractController`. |
| **Keep abstract classes focused** | One reason to change; do not create “kitchen-sink” abstract classes. |
| **Use abstract methods only for genuinely variable behavior** | If a method can be implemented in the abstract class without knowing subclass details, make it concrete. |
| **Provide concrete shared behavior** | Use concrete methods for logic that is identical across all subclasses (Template Method Pattern). |
| **Mark public methods of abstract classes `final`** | Prevents subclasses from overriding the orchestration logic, protecting the algorithm’s structure. |
| **Prefer composition when “is-a” is not genuine** | If the relationship is “has-a” or “uses-a”, use composition instead of inheritance. |
| **Keep hierarchies shallow** | Deep abstract hierarchies are hard to understand and maintain. |
| **Document the contract** | Use PHPDoc to explain what abstract methods must do and what invariants subclasses must uphold. |

**Constraints and Limitations**

- **`final` public methods in abstract classes** enforce encapsulation but may reduce flexibility; use judiciously for orchestration methods that must not change.
- **Abstract classes are coupled**: Subclasses inherit all concrete methods and properties; changes to the abstract class affect all subclasses.
- **Not a substitute for interfaces**: If no shared implementation is needed, an interface is the better choice.

#### Annotated Code Examples

**Example 1: Best-Practice Abstract Class with `final` Template Method**

```php
<?php
declare(strict_types=1);

abstract class AbstractExporter
{
    // Final concrete method — subclasses cannot change the algorithm structure
    final public function export(array $data): string
    {
        $prepared = $this->prepare($data);
        $formatted = $this->format($prepared);
        return $this->finalize($formatted);
    }

    // Abstract steps — subclass-specific behavior
    abstract protected function prepare(array $data): array;
    abstract protected function format(array $prepared): string;

    // Concrete helper method — shared behavior
    protected function finalize(string $formatted): string
    {
        return $formatted . "\n";
    }
}

final class JsonExporter extends AbstractExporter
{
    protected function prepare(array $data): array
    {
        return array_map(fn($row) => array_change_key_case($row, CASE_LOWER), $data);
    }

    protected function format(array $prepared): string
    {
        return json_encode($prepared, JSON_PRETTY_PRINT);
    }
}

$exporter = new JsonExporter();
echo $exporter->export([['Name' => 'Alice', 'Age' => 30]]);
// Outputs:
// [
//     {
//         "name": "Alice",
//         "age": 30
//     }
// ]
?>
```

**Expected Output:**
```
[
    {
        "name": "Alice",
        "age": 30
    }
]
```

**Why This Output:**  
This class follows best practices: the `Abstract` prefix is used; the `export()` method is `final` to protect the algorithm’s structure; abstract methods (`prepare()`, `format()`) are used only for genuinely variable behavior; a concrete helper (`finalize()`) provides shared behavior; and the class is focused on a single responsibility (exporting data). The `JsonExporter` subclass implements only the variable steps.

---

#### Real-World Cases

- **Laravel/Symfony Codebases**: Abstract classes are prefixed with `Abstract` and keep public orchestration methods `final` to protect framework contracts.
- **Doctrine ORM**: `AbstractEntityRepository` provides shared repository logic, while concrete repositories add entity-specific queries.
- **PHPUnit**: `AbstractTestCase` provides shared assertion and setup logic; test classes extend it and implement test methods.

---

## References

**PHP Manual – Class Abstraction** - https://www.php.net/manual/en/language.oop5.abstract.php

**PHP Manual – Abstract Methods (examples)** - https://www.php.net/manual/en/language.oop5.abstract.php#language.oop5.abstract.example-1

**PHP Manual – Abstract Properties (PHP 8.4)** - https://www.php.net/manual/en/language.oop5.abstract.php#language.oop5.abstract.properties

**Exakat – Final, Abstract and Other Family Classes** - https://www.exakat.io/final-abstract-and-other-family-classes/

**Stack Overflow – Abstract Class vs Interface (when to use)** - https://stackoverflow.com/revisions/dee5c2ad-fc0d-45a0-bb41-1b155295cb8a/view-source

**PHP-FIG – PSR Naming Conventions (Abstract prefix)** - https://www.php-fig.org/bylaws/psr-naming-conventions/

**PHP Best Practices – Abstract Public Methods Should Be Final (Template Method Pattern)** - https://raw.githubusercontent.com/voku/agent-skills/main/skills/php-best-practices/rules/abstract-public-methods-final.md

**PHP Manual – Visibility** - https://static.php.net/manual/en/language.oop5.visibility.php

**PHP Manual – Object Inheritance** - https://www.php.net/manual/en/language.oop5.inheritance.php