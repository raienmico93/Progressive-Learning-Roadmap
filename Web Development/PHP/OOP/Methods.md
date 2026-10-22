Here is a comprehensive research cheat sheet on **PHP Methods**, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Methods

### Definitions

**Core Definition**
A **method** is a function defined inside a class that describes a behavior of objects created from that class. Methods operate on the object’s properties and can access the special variable `$this` (in non-static contexts).

**Technical Definition**
In PHP, a method is a function member of a class, trait, interface, or enum. It is declared with optional modifiers (`abstract`, `final`, visibility, `static`), the `function` keyword, a name, a parameter list, an optional return type, and a body (unless abstract). Methods have visibility (`public`, `protected`, `private`), defaulting to `public` if omitted. Non-static methods are bound to an object instance via `$this`; static methods are bound to the class itself and have no `$this`. Since PHP 8.0, methods support union types, `mixed`, and `static` return types; since PHP 8.1, `never` and intersection types. Method declarations follow PSR-12 ordering: `abstract`/`final` before visibility, `static` after visibility.

**Beginner-Friendly Explanation**
A method is an “action” that an object can perform. If a `User` class has properties like `$name` and `$email`, it might have methods like `changeEmail()` or `getFullName()`. Methods are the verbs of object-oriented programming—they define what objects can do.

---

### Key Characteristics

- **Behavior Encapsulation**: Methods bundle logic with the data (properties) it operates on.
- **Visibility Control**: `public`, `protected`, and `private` determine from where a method can be called.
- **`$this` Binding**: Non-static methods automatically receive `$this`, the current object instance.
- **Static Methods**: Belong to the class, not instances; no `$this`; use late static binding (`static::`).
- **Abstract Methods**: Declare a signature without a body; concrete subclasses must implement them.
- **Final Methods**: Cannot be overridden by subclasses.
- **Type Safety**: Parameters and return values can be typed, enforced at runtime.
- **PSR-12 Declaration Order**: `abstract`/`final` precede visibility; `static` follows visibility.

---

### Prerequisites

- **PHP 7.4+** for typed properties and arrow functions; **PHP 8.0+** for union types, `mixed`, `static` return type, and named arguments; **PHP 8.1+** for `never`, `readonly`, and intersection types.
- **Basic Class and Object Syntax**: Understanding of `class`, `new`, `$this`, and object instantiation.
- **`declare(strict_types=1)` Awareness**: Strict types affect parameter and return type enforcement.

---

### Related Programming Areas

- **Object-Oriented Programming (OOP)**: Encapsulation, inheritance, polymorphism, and abstraction.
- **SOLID Principles**: Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion.
- **Design Patterns**: Factory methods, template methods, strategy, and decorator patterns rely on method design.
- **Static Analysis**: PHPStan and Psalm use method signatures to detect type errors.

---

## Core Concepts / Features

---

### 1. Methods (General)

#### Definitions

**Core Definition**
Methods are the function members of a class that define object behavior.

**Technical Definition**
A method is declared inside a class body with the `function` keyword. It may have modifiers (`public`, `protected`, `private`, `static`, `abstract`, `final`), a name, parameters, a return type, and a body. Non-static methods are invoked on an instance (`$object->method()`); static methods are invoked on the class (`ClassName::method()` or `self::method()`). Methods can access instance properties via `$this` and static members via `self::` or `static::`.

**Beginner-Friendly Explanation**
Methods are the actions objects can take. They are functions that live inside a class and usually work with the object’s data.

#### Purposes

- To encapsulate behavior related to an object’s data.
- To provide a controlled interface for interacting with objects.
- To enable polymorphism through method overriding.
- To enforce invariants by validating changes before they occur.

#### Syntax Rules and Structure

**General Syntax**
```php
[abstract|final] [visibility] [static] function methodName(Type $param): ReturnType
{
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `abstract`/`final` | Precede visibility (PSR-12). |
| `visibility` | `public`, `protected`, or `private`; defaults to `public`. |
| `static` | Follows visibility (PSR-12). |
| `function` | Keyword. |
| `methodName` | `camelCase` per PSR-12. |
| Parameters | Typed, with defaults and variadics as needed. |
| Return type | After `:`, e.g., `: void`, `: ?User`. |

**Constraints and Limitations**

- **`abstract` and `private` cannot be combined**; `abstract` and `final` cannot be combined.
- **`$this` is unavailable in static methods**.
- **Method names are case-insensitive** for ASCII, but always call as declared.

#### Real-World Cases

- **Service Methods**: `UserService::register()` encapsulates registration logic.
- **Domain Behavior**: `Order::calculateTotal()` computes the order total from line items.
- **Getters/Setters**: `getName()` / `setName()` or PHP 8.4 property hooks.

---

### 2. Method Declaration

#### Definitions

**Core Definition**
Method declaration is the act of defining a method inside a class, specifying its modifiers, name, parameters, return type, and body.

**Technical Definition**
A declaration may appear in a class, trait, interface, or enum. It must include the `function` keyword and a name. Visibility must be declared explicitly (PSR-12), and modifier order is `abstract`/`final` → visibility → `static`. Abstract methods omit the body; concrete methods include a body enclosed in curly braces. Constructors (`__construct`) and destructors (`__destruct`) are special methods.

**Beginner-Friendly Explanation**
Declaring a method is like writing a recipe in the class blueprint: you give it a name, list the ingredients (parameters), and write the steps (body).

#### Syntax Rules and Structure

**General Syntax**
```php
class MyClass
{
    public function publicMethod(): void {}
    protected function protectedMethod(): void {}
    private function privateMethod(): void {}
    public static function staticMethod(): void {}
    abstract protected function abstractMethod(): void;
    final public function finalMethod(): void {}
}
```

**Component Breakdown**

| Example | Explanation |
|---------|-------------|
| `public function` | Public instance method. |
| `protected static function` | Protected static method (visibility before `static`). |
| `abstract protected function` | Abstract protected method (modifier before visibility). |
| `final public function` | Final public method (modifier before visibility). |

**Constraints and Limitations**

- **One abstract/final modifier per method**; cannot combine `abstract` and `final`.
- **Constructors cannot be static**.
- **Interface methods are implicitly public and abstract**.

#### Annotated Code Examples

**Example 1: Valid Declaration Order (PSR-12)**

```php
<?php
abstract class Shape
{
    // abstract before visibility
    abstract protected function area(): float;

    // final before visibility
    final public function describe(): string
    {
        return "Area: " . $this->area();
    }

    // visibility before static
    public static function create(): static
    {
        return new static();
    }
}
?>
```

**Expected Output:**
No output (definitions only).

**Why This Output:**  
The declaration order follows PSR-12: `abstract`/`final` precede visibility, and `static` follows visibility. Violating this order may still parse but fails PSR-12 coding standards.

---

#### Real-World Cases

- **Controllers**: `public function store(Request $request): Response` with typed parameters and return types.
- **Repositories**: `public function findById(int $id): ?User`.
- **Domain Services**: `public function transfer(Money $amount, Account $from, Account $to): void`.

---

### 3. Method Visibility

#### Definitions

**Core Definition**
Method visibility determines from where a method can be called: `public` (anywhere), `protected` (the class and its inheritance chain), or `private` (only the declaring class).

**Technical Definition**
Visibility is declared with `public`, `protected`, or `private`. Methods without an explicit visibility default to `public`. `Protected` methods are accessible from the declaring class and by parent/child classes. `Private` methods are accessible only within the class that defines them; they are not inherited. Visibility can be relaxed in subclasses (e.g., `protected` → `public`) but not restricted.

**Beginner-Friendly Explanation**
Visibility is like the lock on a door: `public` doors are open to everyone, `protected` doors are open to family (subclasses), and `private` doors are locked to outsiders.

#### Purposes

- To enforce encapsulation and hide implementation details.
- To provide a clear public API while keeping internal helpers private.
- To allow controlled extension through `protected` methods.

#### Syntax Rules and Structure

**General Syntax**
```php
public function methodName(): void {}
protected function methodName(): void {}
private function methodName(): void {}
```

**Component Breakdown**

| Visibility | Accessible From |
|------------|-----------------|
| `public` | Everywhere. |
| `protected` | Declaring class, parent classes, child classes. |
| `private` | Only the declaring class. |

**Constraints and Limitations**

- **Private methods cannot be overridden** (they are not inherited).
- **`abstract` and `private` cannot be combined**.
- **PSR-12 requires explicit visibility** on every method.

#### Annotated Code Examples

**Example 1: Visibility in Action**

```php
<?php
class Account
{
    public function deposit(float $amount): void
    {
        $this->validateAmount($amount);
        $this->balance += $amount;
    }

    protected function getBalance(): float
    {
        return $this->balance;
    }

    private function validateAmount(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException('Amount must be positive.');
        }
    }

    private float $balance = 0.0;
}

$account = new Account();
$account->deposit(100.0); // public — accessible
// $account->getBalance(); // protected — not accessible from outside
// $account->validateAmount(5); // private — not accessible
?>
```

**Expected Output:**
No output (the commented calls would cause fatal errors).

**Why This Output:**  
`deposit()` is public and callable. `getBalance()` is protected and only accessible within the class hierarchy. `validateAmount()` is private and callable only from within `Account`.

---

#### Real-World Cases

- **Template Method Pattern**: A `public` method calls `protected` hook methods that subclasses can override.
- **Internal Helpers**: `private` methods for validation, formatting, or logging that should not be part of the public API.
- **Framework Base Classes**: `protected` methods designed for subclass customization.

---

### 4. Method Parameters

#### Definitions

**Core Definition**
Method parameters are the variables declared in a method signature that receive values (arguments) when the method is called.

**Technical Definition**
Parameters may be typed (scalar, class, union, intersection, nullable), may have default values (constant expressions), may be passed by reference (`&$param`), may be variadic (`...$args`), and may be called using named arguments (PHP 8.0+). Type declarations are enforced at call time; without `strict_types=1`, scalar coercion may occur. Default values must be constant expressions.

**Beginner-Friendly Explanation**
Parameters are the inputs a method expects. They let the same method work with different data each time it is called.

#### Purposes

- To allow methods to operate on different data.
- To enforce type safety at the method boundary.
- To provide flexible calling conventions via defaults, variadics, and named arguments.

#### Syntax Rules and Structure

**General Syntax**
```php
public function methodName(Type $param1, Type $param2 = default, &$param3, ...$variadic): ReturnType
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Type $param` | Optional type declaration. |
| `$param = default` | Optional default value; must be constant. |
| `&$param` | Passed by reference. |
| `...$variadic` | Captures remaining arguments into an array. |
| Named arguments | `methodName(paramName: $value)` (PHP 8.0+). |

**Constraints and Limitations**

- **Default values must be constant expressions**.
- **By-reference parameters require a variable** at the call site.
- **Variadic must be the last parameter**.

#### Annotated Code Examples

**Example 1: Parameters with Defaults and Types**

```php
<?php
declare(strict_types=1);

class Mailer
{
    public function send(
        string $to,
        string $subject,
        string $body = '',
        bool $html = false
    ): bool {
        // send logic
        return true;
    }
}

$mailer = new Mailer();
$mailer->send('alice@example.com', 'Hello'); // $body and $html use defaults
$mailer->send('bob@example.com', 'Hi', '<p>Hello</p>', html: true);
?>
```

**Expected Output:**
No output (method calls succeed).

**Why This Output:**  
The first call omits `$body` and `$html`, so their defaults are used. The second call uses a named argument for `$html`, skipping `$body`’s default.

---

#### Real-World Cases

- **Repository Methods**: `find(int $id, array $options = [])`.
- **Service Methods**: `transfer(Money $amount, Account $from, Account $to): void`.
- **Utility Methods**: `format(string $value, ?string $prefix = null)`.

---

### 5. Method Return Types

#### Definitions

**Core Definition**
A method return type declares the type of value a method will return, enabling PHP to enforce type safety at the method boundary.

**Technical Definition**
Return type declarations appear after the parameter list preceded by a colon (`:`). PHP supports scalar types, class/interface types, nullable types (`?Type`), union types (`int|string`), intersection types (`A&B`), and the special return-only types `void`, `never`, `static`, `self`, and `parent`. `void` indicates no return value; `never` indicates the method never returns normally (throws or exits); `static` returns an instance of the called class (late static binding); `self` returns an instance of the declaring class; `parent` returns an instance of the parent class.

**Beginner-Friendly Explanation**
A return type is a promise: “This method will give you back a `User`” or “This method returns nothing (`void`).” PHP checks the promise and throws a `TypeError` if it is broken.

#### Purposes

- To make method contracts explicit.
- To catch type errors at the return boundary.
- To enable static analysis and IDE inference.
- To support covariance in inheritance.

#### Syntax Rules and Structure

**General Syntax**
```php
public function methodName(): ReturnType
{
    // ...
}
```

**Component Breakdown**

| Return Type | Meaning | Availability |
|-------------|---------|--------------|
| `void` | No return value. | PHP 7.1+ |
| `never` | Never returns normally. | PHP 8.1+ |
| `mixed` | Any type. | PHP 8.0+ |
| `static` | Instance of called class. | PHP 8.0+ |
| `self` | Instance of declaring class. | PHP 7.0+ |
| `parent` | Instance of parent class. | PHP 7.0+ |
| `?Type` | Type or `null`. | PHP 7.1+ |
| `A|B` | Union type. | PHP 8.0+ |
| `A&B` | Intersection type. | PHP 8.1+ |

**Constraints and Limitations**

- **`void` functions cannot return a value**; a bare `return;` is allowed.
- **`never` functions cannot contain `return`**.
- **`static` and `never` are return-only types**; they cannot be used for parameters.
- **Covariance**: Child methods may narrow return types but cannot widen them.

#### Annotated Code Examples

**Example 1: Return Types in Practice**

```php
<?php
declare(strict_types=1);

class UserRepository
{
    public function find(int $id): ?User
    {
        // returns User or null
    }

    public function findOrFail(int $id): User
    {
        $user = $this->find($id);
        if ($user === null) {
            throw new UserNotFoundException($id);
        }
        return $user;
    }

    public function save(User $user): void
    {
        // side effect only
    }

    public function create(): static
    {
        return new static();
    }
}
?>
```

**Expected Output:**
No output (definitions only).

**Why This Output:**  
`find()` returns a nullable `User`; `findOrFail()` guarantees a `User` or throws; `save()` returns `void`; `create()` returns `static` for late static binding.

---

#### Real-World Cases

- **Repository Pattern**: `find(): ?User`, `findAll(): array`.
- **Factory Methods**: `createFromArray(array $data): static`.
- **Validation Methods**: `validate(): void` (side effect: throws on failure).

---

### 6. Static Methods

#### Definitions

**Core Definition**
A static method belongs to the class itself, not to any instance. It can be called without creating an object and has no access to `$this`.

**Technical Definition**
Static methods are declared with the `static` keyword after visibility. They are invoked via `ClassName::method()`, `self::method()`, or `static::method()`. `self::` resolves to the declaring class; `static::` uses late static binding to resolve to the called class. Static methods can access static properties and constants but cannot access instance properties or call non-static methods directly. As of PHP 8.0, static methods can be abstract and can have `static` return types.

**Beginner-Friendly Explanation**
A static method is a method that belongs to the blueprint, not to any cookie made from it. You can call it without creating an object. It is useful for utility functions or factory methods that do not need object state.

#### Purposes

- To provide utility functions that do not depend on object state.
- To implement named constructors and factory methods.
- To support late static binding for inheritance-friendly instantiation.
- To group related functions under a class namespace.

#### Syntax Rules and Structure

**General Syntax**
```php
[visibility] static function methodName(): ReturnType
{
    // no $this
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `static` | Follows visibility (PSR-12). |
| `$this` | Not available. |
| `self::` | Declaring class. |
| `static::` | Called class (late static binding). |

**Constraints and Limitations**

- **No `$this`**: Static methods cannot access instance properties or call non-static methods.
- **Abstract static methods are allowed** (with a strict warning in some contexts).
- **Static methods are not truly inherited**; they are copied into subclasses if not overridden.

#### Annotated Code Examples

**Example 1: Static Factory Method with Late Static Binding**

```php
<?php
class Model
{
    public static function create(): static
    {
        return new static();
    }
}

class User extends Model {}

var_dump(User::create()); // object(User)
?>
```

**Expected Output:**
```
object(User)#1 (0) {}
```

**Why This Output:**  
`create()` uses `new static()`, which resolves to the class on which the method was called (`User`), thanks to late static binding.

---

**Example 2: Static Utility Method**

```php
<?php
class MathHelper
{
    public static function double(int $n): int
    {
        return $n * 2;
    }
}

echo MathHelper::double(5) . "\n"; // Outputs: 10
?>
```

**Expected Output:**
```
10
```

**Why This Output:**  
`double()` does not depend on object state; it is a pure utility function declared static and called directly on the class.

---

#### Real-World Cases

- **Named Constructors**: `User::fromArray($data): static`.
- **Utility Classes**: `Str::slug($string)`, `Arr::get($array, $key)`.
- **Singletons**: `Logger::getInstance(): self`.

---

### 7. Abstract Methods

#### Definitions

**Core Definition**
An abstract method declares a method signature without providing an implementation. Concrete subclasses must implement the method.

**Technical Definition**
Abstract methods are declared with the `abstract` keyword and have no body. The containing class must also be declared `abstract` and cannot be instantiated. When a subclass inherits an abstract method, it must implement it with a compatible signature (same or weaker visibility, same or compatible parameters, same or covariant return type). Abstract methods can be `public` or `protected` but not `private`. As of PHP 8.0, `abstract` and `private` cannot be combined.

**Beginner-Friendly Explanation**
An abstract method is a requirement: “Any class that extends me must provide this method.” It defines what must be done, but not how.

#### Purposes

- To enforce a contract on subclasses.
- To define a template method pattern where the base class orchestrates and subclasses fill in details.
- To share common behavior while forcing specific implementations.

#### Syntax Rules and Structure

**General Syntax**
```php
abstract class AbstractClass
{
    abstract public function methodName(Type $param): ReturnType;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `abstract` | Precedes visibility. |
| No body | Ends with a semicolon. |
| Subclass | Must implement all abstract methods. |

**Constraints and Limitations**

- **Class containing an abstract method must be abstract**.
- **Abstract methods cannot be `private`**.
- **Abstract methods cannot be `final`**.
- **Visibility can be relaxed** (e.g., `protected` abstract → `public` implementation).

#### Annotated Code Examples

**Example 1: Abstract Method Implemented by Subclass**

```php
<?php
abstract class Animal
{
    abstract public function speak(): string;

    public function describe(): string
    {
        return "This animal says: " . $this->speak();
    }
}

class Dog extends Animal
{
    public function speak(): string
    {
        return "Woof!";
    }
}

$dog = new Dog();
echo $dog->describe() . "\n"; // Outputs: This animal says: Woof!
?>
```

**Expected Output:**
```
This animal says: Woof!
```

**Why This Output:**  
`Animal` declares an abstract `speak()` method. `Dog` implements it. The concrete `describe()` method in `Animal` calls `$this->speak()`, which resolves to `Dog::speak()` at runtime.

---

#### Real-World Cases

- **Template Method Pattern**: `AbstractController` with `abstract protected function handle(): Response`.
- **Payment Gateways**: `AbstractGateway` with `abstract public function charge(Money $amount): Receipt`.
- **Validation Rules**: `AbstractRule` with `abstract public function validate(mixed $value): bool`.

---

### 8. Final Methods

#### Definitions

**Core Definition**
A final method cannot be overridden by subclasses. It locks the method’s implementation.

**Technical Definition**
The `final` keyword, placed before visibility, prevents child classes from overriding the method. If a class itself is declared `final`, it cannot be extended. As of PHP 8.0, `private` methods cannot be declared `final` except for the constructor (because private methods are not inherited and therefore cannot be overridden anyway). A method cannot be both `abstract` and `final`.

**Beginner-Friendly Explanation**
A final method is “locked down.” Subclasses can use it but cannot change it. This is useful when a method’s behavior must remain identical in all subclasses for correctness or security reasons.

#### Purposes

- To preserve critical behavior across an inheritance hierarchy.
- To prevent accidental or malicious overriding.
- To enforce a contract that must not be altered.

#### Syntax Rules and Structure

**General Syntax**
```php
final public function methodName(): ReturnType
{
    // implementation
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `final` | Precedes visibility. |
| `function` | Keyword. |
| Body | Must be provided (unlike abstract). |

**Constraints and Limitations**

- **Cannot be overridden**; attempting to do so is a fatal error.
- **Cannot be combined with `abstract`**.
- **Private methods cannot be final** (except constructors as of PHP 8.0).

#### Annotated Code Examples

**Example 1: Final Method Cannot Be Overridden**

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

$child = new Child();
echo $child->criticalOperation() . "\n"; // Outputs: This must not change.
?>
```

**Expected Output:**
```
This must not change.
```

**Why This Output:**  
`criticalOperation()` is final and cannot be overridden. The commented attempt in `Child` would cause a fatal error. The inherited method is used instead.

---

#### Real-World Cases

- **Security-Critical Methods**: Password hashing, token generation, or encryption methods that must not be altered in subclasses.
- **Framework Base Classes**: Methods that enforce a contract (e.g., `AbstractController::render()`).
- **Value Objects**: Methods that compute immutable values and must not be specialized.

---

### 9. Method Best Practices

#### Definitions

**Core Definition**
Method best practices are guidelines for writing methods that are cohesive, testable, readable, and maintainable, following SOLID principles and PSR standards.

**Technical Definition**
Best practices include: **single responsibility** (a method does one thing); **high cohesion** (all statements contribute to one purpose); **explicit visibility** (PSR-12); **full type declarations** on parameters and return values; **`declare(strict_types=1)`**; **short methods** (aim for 20–30 lines); **pure methods where possible**; **dependency injection** rather than global state; **PSR-12 naming** (`camelCase`); and **static analysis** (PHPStan/Psalm). Methods should also be **testable** in isolation and should **avoid side effects** unless explicitly intended.

**Beginner-Friendly Explanation**
Best practices for methods are the habits of good craftsmanship: keep methods small and focused, name them clearly, use types, avoid hidden dependencies, and let tools check your work.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Single Responsibility** | Each method does one thing and does it well. |
| **High Cohesion** | All statements in the method contribute to its single purpose. |
| **Explicit Visibility** | Declare `public`, `protected`, or `private` on every method (PSR-12). |
| **Full Type Declarations** | Type every parameter and every return value. |
| **`declare(strict_types=1)`** | Enforce exact type matching. |
| **Short Methods** | Aim for 20–30 lines; extract sub-methods when longer. |
| **Pure Methods Where Possible** | Prefer deterministic methods without side effects. |
| **Dependency Injection** | Pass dependencies as parameters; avoid `global`. |
| **PSR-12 Naming** | `camelCase` method names; no leading underscores for visibility. |
| **Static Analysis** | Run PHPStan or Psalm to catch type and design issues. |

**Constraints and Limitations**

- **SOLID is a guide, not a law**: Over-application can lead to excessive abstraction.
- **Strict types are per-file**: Coercion behavior depends on the strictness of the file where the call occurs.
- **Static methods hinder testability**: Prefer instance methods with injected dependencies for testable code.

#### Annotated Code Examples

**Example 1: A Well-Designed Method**

```php
<?php
declare(strict_types=1);

final class PriceCalculator
{
    public function calculateTotal(array $items, float $taxRate): float
    {
        if ($taxRate < 0) {
            throw new InvalidArgumentException('Tax rate cannot be negative.');
        }

        $subtotal = array_sum(
            array_map(
                static fn(array $item): float => $item['price'] * $item['quantity'],
                $items
            )
        );

        return $subtotal * (1 + $taxRate);
    }
}

$calculator = new PriceCalculator();
$items = [
    ['price' => 10.0, 'quantity' => 2],
    ['price' => 5.0, 'quantity' => 3],
];

echo $calculator->calculateTotal($items, 0.2) . "\n"; // Outputs: 42
?>
```

**Expected Output:**
```
42
```

**Why This Output:**  
This method exemplifies best practices: it has `strict_types=1`, full type declarations, a clear return type (`float`), a single responsibility (calculating a total), high cohesion, a short body, a pure computation (no side effects), and a typed exception for invalid input. The calculation is `(10*2 + 5*3) * 1.2 = 35 * 1.2 = 42`.

---

**Example 2: Before and After Refactoring for SRP**

```php
<?php
// BEFORE: Method doing validation, calculation, and formatting
function processOrderOld(array $order): string
{
    if (empty($order['items'])) {
        return "Error: No items";
    }
    $total = 0;
    foreach ($order['items'] as $item) {
        if ($item['quantity'] <= 0) {
            return "Error: Invalid quantity";
        }
        $total += $item['price'] * $item['quantity'];
    }
    return "Total: $" . number_format($total, 2);
}

// AFTER: Each method has a single responsibility
function validateOrder(array $order): ?string
{
    if (empty($order['items'])) return "No items";
    foreach ($order['items'] as $item) {
        if ($item['quantity'] <= 0) return "Invalid quantity";
    }
    return null;
}

function calculateOrderTotal(array $order): float
{
    $total = 0.0;
    foreach ($order['items'] as $item) {
        $total += $item['price'] * $item['quantity'];
    }
    return $total;
}

function formatTotal(float $total): string
{
    return "Total: $" . number_format($total, 2);
}

function processOrderNew(array $order): string
{
    $error = validateOrder($order);
    if ($error !== null) return "Error: $error";
    return formatTotal(calculateOrderTotal($order));
}
?>
```

**Expected Output:**
No output (definitions only). The refactored version splits one long method into four focused methods, each with a single responsibility.

**Why This Output:**  
The original method violated SRP by mixing validation, calculation, and formatting. The refactored methods are shorter, more cohesive, easier to test, and easier to reuse.

---

#### Real-World Cases

- **Laravel/Symfony Codebases**: Methods follow PSR-12, use full type declarations, and are kept small for testability.
- **Static Analysis in CI**: PHPStan at level 6+ and PHP CS Fixer enforce return types, visibility, and naming conventions.
- **Open-Source Libraries**: Well-maintained packages follow these practices consistently, making them reliable and easy to integrate.

---

## References

**PHP Manual – Visibility** - https://static.php.net/manual/en/language.oop5.visibility.php

**PHP Manual – Static Keyword** - https://static.php.net/manual/en/language.oop5.static.php

**PHP Manual – Abstract Classes and Methods** - https://github.com/php/doc-en/blob/master/language/oop5/abstract.xml

**PHP Manual – Final Keyword** - https://static.php.net/manual/en/language.oop5.final.php

**PHP Manual – Late Static Bindings** - https://static.php.net/manual/en/language.oop5.late-static-bindings.php

**PHP Manual – Function Parameters and Arguments** - https://static.php.net/manual/en/functions.arguments.php

**PHP Manual – Return Type Declarations** - https://static.php.net/manual/en/language.types.declarations.php

**PHP Best Practices – Return Type Declarations (Agent Skills)** - https://github.com/voku/agent-skills/blob/main/skills/php-best-practices/rules/type-return-types.md

**PSR-12: Extended Coding Style** - https://raw.githubusercontent.com/armourinfosec/Secure-PHP-Development/refs/heads/main/PSR-Standards/PSR-12-Extended-Coding-Style.md

**PHP Naming Conventions (PSR-1 / PSR-12)** - https://github.com/turkeryildirim/ai-skills/blob/main/code-standards/references/naming-php.md

**IxDF's SOLID Principles for PHP – Single Responsibility** - https://handbook.ixdf.org/library/backend/SOLID.html

**PHP Method Declaration Standard (PHP_CodeSniffer)** - https://github.com/squizlabs/PHP_CodeSniffer/blob/master/src/Standards/PSR2/Docs/Methods/MethodDeclarationStandard.xml