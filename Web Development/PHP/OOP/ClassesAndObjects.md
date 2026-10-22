Here is a comprehensive research cheat sheet on **PHP Classes and Objects**, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Classes and Objects

### Definitions

**Core Definition**
A **class** is a blueprint or template that defines a set of properties (data) and methods (behavior) that objects created from it will have. An **object** is a concrete instance of a class, containing its own copy of the properties defined by the class and able to invoke the class's methods. Classes and objects are the foundation of object-oriented programming (OOP) in PHP.

**Technical Definition**
In PHP, a class is a user-defined type declared with the `class` keyword, followed by a name and a body enclosed in curly braces. A class may contain **constants**, **properties** (variables), and **methods** (functions). A class is a type; an object is a value of that type. PHP's object model treats objects as references (handles): assigning an object to another variable copies the handle, not the entire object, so both variables refer to the same instance. Objects are created with the `new` keyword, which allocates memory, initializes properties, and invokes the constructor (`__construct()`) if one is defined. Inside methods, the pseudo-variable `$this` refers to the current object instance, while `self`, `parent`, and `static` provide scope-resolution access to class members.

**Beginner-Friendly Explanation**
A class is like a cookie cutter, and objects are the cookies. The class defines the shape (properties) and the recipe (methods), but it is not itself a cookie. You create cookies (objects) by pressing the cutter into dough (`new`). Each cookie can have its own decorations (property values) but shares the same overall shape and behavior defined by the class. `$this` is how a cookie refers to itself, `self` refers to the cookie cutter it came from, `parent` refers to a larger cookie cutter it was based on, and `static` refers to the specific cookie cutter in use when the method was called.

---

### Key Characteristics

- **Blueprint vs. Instance**: The class is the definition; objects are the runtime instances.
- **Reference Semantics**: PHP objects are passed and assigned by handle (reference). Two variables can point to the same object.
- **Encapsulation**: Visibility modifiers (`public`, `protected`, `private`) control access to properties, methods, and constants.
- **Member Types**: Classes contain constants, properties, and methods.
- **Constructor Initialization**: The `__construct()` method runs automatically on `new` and is used to initialize the object.
- **Late Static Binding**: `static` refers to the runtime class, while `self` refers to the compile-time (declaring) class.
- **PHP 8.4 Additions**: Property hooks (executed on get/set) and asymmetric visibility (different access for reading and writing) modernize property handling.

---

### Prerequisites

- **PHP 7.4+ Recommended** (typed properties, arrow functions); **PHP 8.1+** for `readonly` properties and enums; **PHP 8.4** for property hooks and asymmetric visibility.
- **Basic Understanding of Variables and Functions**: Classes are composed of variables (properties) and functions (methods).
- **Awareness of `declare(strict_types=1)`**: Strict types affect how property and parameter types are enforced.

---

### Related Programming Areas

- **Object-Oriented Design (SOLID)**: Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion.
- **Inheritance and Polymorphism**: `extends`, `parent`, abstract classes, interfaces, and traits.
- **Design Patterns**: Factory, Singleton, Repository, Decorator, and many others rely on classes and objects.
- **Dependency Injection**: Objects are wired together via constructors or setters rather than created internally.
- **Static Analysis**: PHPStan and Psalm rely on class declarations, property types, and return types to detect bugs.

---

## Core Concepts / Features

---

### 1. Classes

#### Definitions

**Core Definition**
A class is a user-defined data type that encapsulates data (properties) and behavior (methods) into a single unit. It serves as a blueprint for creating objects.

**Technical Definition**
A class is declared with the `class` keyword, followed by a valid label name and a body enclosed in curly braces. The body may contain **class constants**, **properties** (data members), and **methods** (function members). The class name must match `^[a-zA-Z_\x7f-\xff][a-zA-Z0-9_\x7f-\xff]*$` and must not be a reserved word. Classes may be `final` (cannot be extended), `abstract` (cannot be instantiated), or may extend a parent class and implement interfaces.

**Beginner-Friendly Explanation**
A class is a recipe or blueprint. It describes what data an object will hold and what actions it can perform, but it is not the object itself. You write the class once and use it to create as many objects as you need.

#### Purposes

- To encapsulate related data and behavior into a reusable unit.
- To model real-world entities (User, Order, Product) or abstract concepts (Repository, Logger, Validator).
- To enable code organization, reuse, and maintainability through inheritance and polymorphism.
- To provide a contract for objects via interfaces and abstract classes.

#### Syntax Rules and Structure

**General Syntax**
```php
class ClassName
{
    // constants
    // properties
    // methods
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `class` | The keyword that begins a class declaration. |
| `ClassName` | A valid PHP label; typically `StudlyCaps` per PSR-12. |
| `{ ... }` | The class body containing members. |
| `final` | Optional modifier preventing extension. |
| `abstract` | Optional modifier preventing instantiation. |
| `extends` | Optional parent class. |
| `implements` | Optional interface list. |

**Syntax Rules**

- A class name must start with a letter or underscore, followed by letters, numbers, or underscores.
- PSR-12 recommends `StudlyCaps` for class names (e.g., `UserRepository`, not `user_repository`).
- A class may contain zero or more members (constants, properties, methods).
- Classes are declared at the top level of a file or inside a namespace.

**Constraints and Limitations**

- **No multiple inheritance**: PHP supports single inheritance only (one parent per class).
- **Class name case-insensitivity**: Like function names, class names are case-insensitive for ASCII, but always use the declared casing.
- **Final classes cannot be extended**; abstract classes cannot be instantiated.

#### Annotated Code Examples

**Example 1: A Simple Class with Properties and Methods**

```php
<?php
declare(strict_types=1);

class SimpleClass
{
    // Property declaration with default value
    public string $var = 'a default value';

    // Method declaration
    public function displayVar(): void
    {
        echo $this->var . "\n";
    }
}

$instance = new SimpleClass();
$instance->displayVar();   // Outputs: a default value
?>
```

**Expected Output:**
```
a default value
```

**Why This Output:**  
The class `SimpleClass` declares a public property `$var` with a default value and a method `displayVar()` that echoes it. Creating an instance with `new` and calling `displayVar()` outputs the default value. The `$this` pseudo-variable refers to the current object.

**Step-by-Step Setup:**
1. Save as `simple_class.php`.
2. Run `php simple_class.php`.
3. The output confirms the class was instantiated and the method executed.

---

**Example 2: A Final Class**

```php
<?php
final class Currency
{
    public function __construct(public readonly string $code) {}
}

// class ExtendedCurrency extends Currency {} // Fatal error: cannot extend final class
?>
```

**Expected Output:**
No output (the commented-out line would cause a fatal error if uncommented).

**Why This Output:**  
The `final` keyword prevents `Currency` from being extended. Attempting to extend it results in a fatal error. Final classes are useful for value objects that should not be specialized.

#### Real-World Cases

- **Value Objects**: `Money`, `EmailAddress`, `UserId` — small, immutable classes representing domain concepts.
- **Services**: `UserService`, `PaymentGateway` — classes encapsulating business logic.
- **Repositories**: `UserRepository` — classes abstracting database access.

---

### 2. Objects and Instances

#### Definitions

**Core Definition**
An object is a concrete, runtime instance of a class. It occupies memory, has its own set of property values, and can invoke the methods defined by its class.

**Technical Definition**
An object is created with the `new` keyword followed by the class name and an optional argument list. The result is a value of the class type. PHP objects use **handle-based reference semantics**: assigning an object to another variable copies the handle, so both variables refer to the same underlying object. Objects are destroyed when no references remain (reference counting), and the destructor (`__destruct()`) is called if defined.

**Beginner-Friendly Explanation**
An object is a specific cookie made from the cookie cutter (class). Each cookie can have different icing (property values), but they all share the same shape (methods). If you give a friend a cookie, you are both holding the same cookie (reference semantics)—if your friend takes a bite, your cookie is also bitten.

#### Purposes

- To create concrete entities that hold state and expose behavior.
- To allow multiple independent instances of the same class, each with its own data.
- To enable polymorphism: different objects can respond to the same method call in different ways.

#### Syntax Rules and Structure

**General Syntax**
```php
$object = new ClassName();
$object = new ClassName($arg1, $arg2);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `new` | The keyword that allocates a new object. |
| `ClassName` | The class to instantiate. |
| `()` | Optional argument list passed to the constructor. |

**Syntax Rules**

- `new` returns an object instance; it can be assigned to a variable or used directly (`(new ClassName())->method()`).
- Each `new` call creates a distinct instance (unless the class implements a singleton pattern).
- Object properties are accessed with `->` (e.g., `$object->property`).
- Methods are called with `->` (e.g., `$object->method()`).

**Constraints and Limitations**

- **Abstract classes cannot be instantiated**; only concrete subclasses can.
- **Constructor arguments must match** the constructor signature (or be compatible via default values/named arguments).
- **Objects are not copied on assignment**; only the handle is copied. Use `clone` for a shallow copy.

#### Annotated Code Examples

**Example 1: Creating Multiple Independent Instances**

```php
<?php
class Counter
{
    private int $count = 0;

    public function increment(): void
    {
        $this->count++;
    }

    public function getCount(): int
    {
        return $this->count;
    }
}

$a = new Counter();
$b = new Counter();

$a->increment();
$a->increment();
$b->increment();

echo $a->getCount() . "\n"; // Outputs: 2
echo $b->getCount() . "\n"; // Outputs: 1
?>
```

**Expected Output:**
```
2
1
```

**Why This Output:**  
Each `new Counter()` creates a separate object with its own `$count` property. `$a` is incremented twice and `$b` once, demonstrating independent state.

---

**Example 2: Reference Semantics**

```php
<?php
class Box { public int $value = 0; }

$box1 = new Box();
$box2 = $box1;          // copies the handle, not the object

$box2->value = 42;
echo $box1->value . "\n"; // Outputs: 42
?>
```

**Expected Output:**
```
42
```

**Why This Output:**  
`$box1` and `$box2` both point to the same `Box` object. Modifying the property through `$box2` is visible through `$box1`, demonstrating reference semantics.

#### Real-World Cases

- **Entities**: A `User` object representing a row in the `users` table.
- **Domain Models**: An `Order` object containing line items, totals, and status.
- **Service Objects**: A `Mailer` object configured with SMTP settings and used to send messages.

---

### 3. Class Declaration

#### Definitions

**Core Definition**
Class declaration is the act of defining a class using the `class` keyword, its name, and its body of members.

**Technical Definition**
A class declaration begins with optional modifiers (`final`, `abstract`), the `class` keyword, the class name, optional `extends` and `implements` clauses, and a body. The body is enclosed in curly braces and may contain constants, properties, methods, and traits. Declarations can be conditional (inside `if` statements) or top-level. PSR-4 autoloading maps class names to file paths, so one class per file is the convention.

**Beginner-Friendly Explanation**
Declaring a class is like writing the blueprint. You give it a name, list the data it should hold (properties), and write the actions it can perform (methods). The declaration is not an object—it is the instructions for making objects.

#### Purposes

- To define the structure and behavior of objects.
- To create reusable types that can be instantiated many times.
- To establish inheritance hierarchies and interface contracts.

#### Syntax Rules and Structure

**General Syntax**
```php
[modifiers] class ClassName [extends ParentClass] [implements Interface1, Interface2]
{
    // members
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `final` | Prevents extension. |
| `abstract` | Prevents instantiation; may contain abstract methods. |
| `extends` | Single parent class. |
| `implements` | One or more interfaces. |
| Body `{ ... }` | Constants, properties, methods, traits. |

**Syntax Rules**

- Only one class per file is strongly recommended (PSR-4).
- The file name should match the class name (e.g., `User.php` for `class User`).
- Class names should be `StudlyCaps` (PascalCase).
- `declare(strict_types=1)` should be the first statement in the file.

**Constraints and Limitations**

- **Single inheritance**: Only one `extends` clause.
- **Abstract methods**: An abstract class may declare abstract methods without a body; concrete subclasses must implement them.
- **Final classes**: Cannot be extended; final methods cannot be overridden.

#### Annotated Code Examples

**Example 1: Declaration with Inheritance**

```php
<?php
declare(strict_types=1);

abstract class Animal
{
    public function __construct(protected string $name) {}

    abstract public function speak(): string;

    public function getName(): string
    {
        return $this->name;
    }
}

class Dog extends Animal
{
    public function speak(): string
    {
        return "Woof!";
    }
}

$dog = new Dog("Rex");
echo $dog->getName() . " says " . $dog->speak() . "\n";
// Outputs: Rex says Woof!
?>
```

**Expected Output:**
```
Rex says Woof!
```

**Why This Output:**  
`Animal` is abstract and declares an abstract `speak()` method. `Dog` extends `Animal`, implements `speak()`, and inherits the constructor and `getName()` method. The object is created as a `Dog` but treated as an `Animal` type.

#### Real-World Cases

- **Controller Classes**: `UserController extends AbstractController`.
- **Model Classes**: `User extends Model`.
- **Exception Hierarchies**: `InsufficientFundsException extends RuntimeException`.

---

### 4. Class Members

#### Definitions

**Core Definition**
Class members are the constituents of a class: **constants**, **properties**, and **methods**. They define the data and behavior of objects created from the class.

**Technical Definition**
Class members are declared inside the class body. **Constants** are immutable values allocated once per class. **Properties** are variables that hold object state; each instance has its own copy (unless declared `static`). **Methods** are functions that operate on the object's state and can access `$this`, `self`, `parent`, and `static`. All members have visibility (`public`, `protected`, or `private`), with `public` as the default for methods and properties, and `public` as the default for constants.

**Beginner-Friendly Explanation**
Class members are the ingredients and instructions of the blueprint. Constants are fixed values (like a maximum size). Properties are the data each object carries (like a name or price). Methods are the actions the object can perform (like calculateTotal or sendEmail).

#### Purposes

- To encapsulate all data and behavior related to a concept within one unit.
- To control access to internal state via visibility modifiers.
- To share immutable values across all instances via constants.

#### Syntax Rules and Structure (Overview)

| Member | Declaration | Access |
|--------|-------------|--------|
| Constant | `const NAME = value;` or `public const NAME = value;` | `ClassName::NAME` or `self::NAME` |
| Property | `visibility [static] Type $name [= default];` | `$this->name` (instance) or `self::$name` (static) |
| Method | `visibility [static] function name(...): ReturnType { ... }` | `$this->name()` or `self::name()` |

**Constraints and Limitations**

- **Property default values** must be constant expressions (evaluated at compile time), not runtime values like function calls.
- **Constants cannot be changed** after declaration; `final` constants cannot be overridden.
- **Static members** belong to the class, not to any instance.

#### Real-World Cases

- **Configuration Constants**: `public const MAX_LOGIN_ATTEMPTS = 5;`
- **Instance Properties**: `private string $email;`
- **Behavioral Methods**: `public function sendWelcomeEmail(): void { ... }`

---

### 5. Properties

#### Definitions

**Core Definition**
Properties are variables declared inside a class that hold the state of an object. Each instance has its own copy of instance properties.

**Technical Definition**
A property declaration may include a visibility modifier (`public`, `protected`, `private`), an optional `static` modifier, an optional type declaration (PHP 7.4+), and an optional default value. The default value must be a constant expression (e.g., a literal, a constant, or an array of constants). Typed properties enforce type safety at assignment; `readonly` properties (PHP 8.1+) can be initialized once and never modified. PHP 8.4 introduces **asymmetric visibility** (`public private(set)`) and **property hooks** (`get`/`set` blocks).

**Beginner-Friendly Explanation**
Properties are the object's “attributes” or “fields.” A `User` object might have `$name`, `$email`, and `$age` properties. Each user object has its own values for these properties.

#### Purposes

- To store object state.
- To enforce type safety via typed properties.
- To control read/write access via visibility modifiers and asymmetric visibility.
- To provide computed or virtual properties via property hooks (PHP 8.4+).

#### Syntax Rules and Structure

**General Syntax**
```php
[visibility] [static] [readonly] [type] $propertyName [= default];
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `visibility` | `public`, `protected`, or `private`. |
| `static` | Optional; property belongs to the class, not instances. |
| `readonly` | PHP 8.1+; initialized once, never modified. |
| `type` | Optional scalar, class, union, or intersection type. |
| `$name` | The property variable name. |
| `= default` | Optional constant expression default. |

**Syntax Rules**

- Typed properties must be initialized before access; accessing an uninitialized typed property throws an `Error`.
- `readonly` properties must be typed and initialized exactly once (in the declaring class scope).
- Asymmetric visibility (PHP 8.4+): `public private(set) int $id;` allows public reads but private writes.
- Property hooks (PHP 8.4+): `public string $name { set => strtoupper($value); }`.

**Constraints and Limitations**

- **Default values must be constant**: You cannot assign a function call or object instance as a default.
- **Typed properties are uninitialized by default**: Unlike untyped properties (which default to `null`), typed properties without a default are in an “uninitialized” state.
- **Property hooks are incompatible with `readonly`**.

#### Annotated Code Examples

**Example 1: Typed and Readonly Properties**

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public readonly int $id,
        public string $name,
        private ?string $email = null
    ) {}

    public function getEmail(): ?string
    {
        return $this->email;
    }
}

$user = new User(1, "Alice", "alice@example.com");
echo $user->id . "\n";       // Outputs: 1
echo $user->name . "\n";     // Outputs: Alice
echo $user->getEmail() . "\n"; // Outputs: alice@example.com

// $user->id = 2; // Error: cannot modify readonly property
?>
```

**Expected Output:**
```
1
Alice
alice@example.com
```

**Why This Output:**  
The constructor promotes `$id` to a `readonly` property, `$name` to a public string property, and `$email` to a private nullable string property accessed via a getter. Readonly properties can only be initialized once.

---

**Example 2: Asymmetric Visibility (PHP 8.4+)**

```php
<?php
class Product
{
    public private(set) int $stock = 0;

    public function restock(int $quantity): void
    {
        $this->stock += $quantity;
    }
}

$product = new Product();
echo $product->stock . "\n"; // Outputs: 0
$product->restock(10);
echo $product->stock . "\n"; // Outputs: 10
// $product->stock = 5; // Error: cannot modify private(set) property
?>
```

**Expected Output:**
```
0
10
```

**Why This Output:**  
The `public private(set)` visibility allows reading `$stock` from anywhere but only writing it from within the `Product` class. The `restock()` method modifies it internally.

---

**Example 3: Property Hooks (PHP 8.4+)**

```php
<?php
class User
{
    public string $email {
        set {
            if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
                throw new ValueError('Invalid email address');
            }
            $this->email = $value;
        }
    }
}

$user = new User();
$user->email = 'alice@example.com'; // OK
// $user->email = 'not-an-email';   // Throws ValueError
?>
```

**Expected Output:**
No output (the invalid assignment would throw an exception).

**Why This Output:**  
The `set` hook validates the email address before assignment. If the value is not a valid email, a `ValueError` is thrown, enforcing the invariant directly on the property.

#### Real-World Cases

- **Domain Entities**: `User` with `readonly int $id` and mutable `string $name`.
- **Configuration Objects**: `private array $options` with public getters.
- **Data Transfer Objects (DTOs)**: `public readonly string $email` for immutable data.

---

### 6. Methods

#### Definitions

**Core Definition**
Methods are functions declared inside a class that define the behavior of objects. They can access and modify the object's properties via `$this`.

**Technical Definition**
A method is declared with a visibility modifier, optional `static` modifier, the `function` keyword, a name, a parameter list, an optional return type, and a body. Non-static methods receive `$this` automatically. Static methods do not have `$this` and are called via `ClassName::method()` or `self::method()`. Methods can be `final` (cannot be overridden) or `abstract` (no body, must be implemented by subclasses). Constructors (`__construct`) and destructors (`__destruct`) are special methods.

**Beginner-Friendly Explanation**
Methods are the actions an object can perform. A `User` object might have a `changeEmail()` method or a `getFullName()` method. Methods can read and change the object's properties.

#### Purposes

- To encapsulate behavior related to the object's data.
- To enforce invariants by validating changes before modifying properties.
- To provide a public interface for interacting with the object.
- To implement polymorphism via method overriding.

#### Syntax Rules and Structure

**General Syntax**
```php
[visibility] [static] [final|abstract] function methodName(Type $param): ReturnType
{
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `visibility` | `public`, `protected`, or `private`. |
| `static` | Method belongs to the class, not instances. |
| `final` | Prevents overriding in subclasses. |
| `abstract` | No body; subclass must implement. |
| `function` | Keyword. |
| `methodName` | `camelCase` per PSR-12. |
| `$this` | Available in non-static methods. |

**Syntax Rules**

- Method names are `camelCase` (PSR-12).
- Non-static methods are called on an instance (`$object->method()`).
- Static methods are called on the class (`ClassName::method()` or `self::method()`).
- Constructors are named `__construct`; destructors are `__destruct`.

**Constraints and Limitations**

- **`$this` is not available in static methods**; attempting to use it results in an `Error`.
- **Abstract methods** must be implemented by concrete subclasses.
- **Final methods** cannot be overridden.

#### Annotated Code Examples

**Example 1: Instance Methods and `$this`**

```php
<?php
class BankAccount
{
    private float $balance = 0.0;

    public function deposit(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException('Deposit must be positive.');
        }
        $this->balance += $amount;
    }

    public function getBalance(): float
    {
        return $this->balance;
    }
}

$account = new BankAccount();
$account->deposit(100.0);
$account->deposit(50.0);
echo $account->getBalance() . "\n"; // Outputs: 150
?>
```

**Expected Output:**
```
150
```

**Why This Output:**  
The `deposit()` method modifies the object's `$balance` property via `$this`. Each call accumulates the balance.

---

**Example 2: Static Methods**

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
`double()` is static and does not depend on object state. It is called directly on the class.

#### Real-World Cases

- **Service Methods**: `sendEmail()`, `processPayment()`.
- **Getters and Setters**: `getName()`, `setName()`, or property hooks in PHP 8.4+.
- **Factory Methods**: `static createFromArray(array $data): static`.

---

### 7. Constants

#### Definitions

**Core Definition**
Class constants are immutable values defined on a per-class basis. They are allocated once per class and shared by all instances.

**Technical Definition**
Class constants are declared with the `const` keyword (or `public const`, `protected const`, `private const` as of PHP 7.1). They must be initialized with a constant expression. As of PHP 8.3, class constants can have scalar type declarations (`bool`, `int`, `float`, `string`, `array`). Constants are accessed with `ClassName::CONSTANT` or `self::CONSTANT`. Final constants (PHP 8.1+) cannot be overridden by child classes.

**Beginner-Friendly Explanation**
A class constant is a fixed value that belongs to the class itself, not to any object. It is like a rule written on the blueprint: every object made from the blueprint follows the same rule, and the rule cannot be changed. For example, a `Circle` class might have a constant `PI = 3.14159`.

#### Purposes

- To define fixed values that are shared across all instances.
- To avoid magic numbers and strings by giving them meaningful names.
- To enforce consistency when the same value is used in multiple places.

#### Syntax Rules and Structure

**General Syntax**
```php
[visibility] [final] const [type] NAME = value;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `visibility` | `public` (default), `protected`, or `private`. |
| `final` | PHP 8.1+; prevents overriding. |
| `type` | PHP 8.3+; scalar or array type. |
| `NAME` | Constant name; `UPPER_SNAKE_CASE` per convention. |
| `value` | Constant expression. |

**Syntax Rules**

- Constants do not use the `$` sigil.
- Access with `ClassName::NAME` or `self::NAME`.
- Interface constants cannot be overridden by implementing classes (since PHP 8.1, unless not final).

**Constraints and Limitations**

- **Value must be constant**: Cannot depend on runtime information.
- **Cannot be modified**: Attempting to assign to a constant causes an error.
- **`final` constants cannot be overridden** by child classes.

#### Annotated Code Examples

**Example 1: Class Constants with Visibility**

```php
<?php
class Circle
{
    public const float PI = 3.14159;   // typed constant (PHP 8.3+)
    private const int PRECISION = 2;

    public static function area(float $radius): float
    {
        return round(self::PI * $radius ** 2, self::PRECISION);
    }
}

echo Circle::area(5.0) . "\n"; // Outputs: 78.54
?>
```

**Expected Output:**
```
78.54
```

**Why This Output:**  
`PI` is a public constant accessible from outside the class. `PRECISION` is private and used internally by the static method `area()`.

#### Real-World Cases

- **Mathematical Constants**: `Circle::PI`.
- **Configuration Limits**: `User::MAX_LOGIN_ATTEMPTS`.
- **Status Codes**: `Order::STATUS_PENDING`, `Order::STATUS_SHIPPED`.

---

### 8. `new` (Object Creation)

#### Definitions

**Core Definition**
The `new` keyword is used to create a new object instance from a class.

**Technical Definition**
`new ClassName($args)` allocates memory for a new object, initializes its properties, and invokes the constructor (`__construct`) if one is defined. The result is an object handle that can be assigned to a variable or used immediately. PHP 8.4 allows `new ClassName` without parentheses when there are no constructor arguments.

**Beginner-Friendly Explanation**
`new` is the command that actually makes an object out of a class. It is like pressing the cookie cutter into the dough—you get a new cookie (object).

#### Purposes

- To create concrete instances from a class.
- To invoke the constructor with initialization arguments.
- To allocate memory for an object's state.

#### Syntax Rules and Structure

**General Syntax**
```php
$object = new ClassName();
$object = new ClassName($arg1, $arg2);
$object = new ClassName;        // PHP 8.4+, no parentheses if no args
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `new` | Allocation keyword. |
| `ClassName` | The class to instantiate. |
| `()` | Argument list for the constructor (optional). |

**Syntax Rules**

- The class must exist and be instantiable (not abstract).
- Constructor arguments must match the constructor signature.
- `new` can be used in expressions: `$obj = new Foo();` or `(new Foo())->bar()`.

**Constraints and Limitations**

- **Abstract classes cannot be instantiated**.
- **Constructor may throw exceptions**; these propagate to the caller.

#### Annotated Code Examples

**Example 1: Creating an Object with Constructor Arguments**

```php
<?php
class Point
{
    public function __construct(
        public readonly float $x,
        public readonly float $y
    ) {}
}

$point = new Point(3.0, 4.0);
echo "({$point->x}, {$point->y})\n"; // Outputs: (3, 4)
?>
```

**Expected Output:**
```
(3, 4)
```

**Why This Output:**  
`new Point(3.0, 4.0)` invokes the constructor, which promotes the arguments to readonly properties. The object is then used to access `$x` and `$y`.

#### Real-World Cases

- **Dependency Injection**: `new UserService(new UserRepository($pdo))`.
- **Factory Methods**: `User::createFromArray($data)` internally calls `new self(...)`.
- **Value Objects**: `new Money(100, 'USD')`.

---

### 9. Object Initialization (Constructor)

#### Definitions

**Core Definition**
Object initialization is the process of setting up an object's initial state, typically performed by the constructor method `__construct()`.

**Technical Definition**
The constructor is a special method named `__construct()` that is automatically invoked when an object is created with `new`. It receives the arguments passed to `new` and is used to initialize properties, validate invariants, and perform setup tasks. If a child class defines a constructor, it must explicitly call `parent::__construct()` if the parent constructor is needed. PHP allows at most one constructor per class. Constructor property promotion (PHP 8.0+) allows declaring and initializing properties directly in the constructor signature.

**Beginner-Friendly Explanation**
The constructor is the object's “setup routine.” It runs automatically the moment an object is born, making sure all its properties are set to sensible starting values.

#### Purposes

- To initialize an object's properties with provided values.
- To validate invariants before the object is used.
- To establish dependencies (e.g., database connections, services).
- To perform any setup tasks required for the object to function.

#### Syntax Rules and Structure

**General Syntax**
```php
public function __construct(Type $param1, Type $param2 = default)
{
    // initialization
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `__construct` | The special method name. |
| Parameters | Values passed from `new`. |
| Body | Initialization logic. |
| `parent::__construct()` | Explicit call to parent constructor if needed. |

**Syntax Rules**

- A class may have at most one constructor.
- Constructor arguments are passed via `new ClassName(...)`.
- If a child defines a constructor, the parent constructor is **not** called automatically; use `parent::__construct()`.
- Constructor property promotion (PHP 8.0+): `public function __construct(private string $name) {}`.

**Constraints and Limitations**

- **Constructor exceptions**: If the constructor throws, the object is not fully created.
- **Promoted properties cannot be duplicated** in the class body.
- **Promoted properties cannot be `callable`** (use `Closure`).

#### Annotated Code Examples

**Example 1: Constructor with Validation**

```php
<?php
declare(strict_types=1);

class Email
{
    private string $address;

    public function __construct(string $address)
    {
        if (!filter_var($address, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException('Invalid email address.');
        }
        $this->address = $address;
    }

    public function getAddress(): string
    {
        return $this->address;
    }
}

$email = new Email('alice@example.com');
echo $email->getAddress() . "\n"; // Outputs: alice@example.com
?>
```

**Expected Output:**
```
alice@example.com
```

**Why This Output:**  
The constructor validates the email before assigning it to the property. If the email is invalid, an exception is thrown, preventing an invalid object from being created.

---

**Example 2: Constructor Property Promotion (PHP 8.0+)**

```php
<?php
class User
{
    public function __construct(
        public readonly int $id,
        public string $name,
        private ?string $email = null
    ) {}
}

$user = new User(1, "Alice");
echo $user->id . "\n";   // Outputs: 1
echo $user->name . "\n"; // Outputs: Alice
?>
```

**Expected Output:**
```
1
Alice
```

**Why This Output:**  
The constructor promotes `$id`, `$name`, and `$email` to properties with the specified visibility and types, eliminating boilerplate property declarations.

#### Real-World Cases

- **Entity Constructors**: `new User($id, $email)` with validation.
- **Service Constructors**: `new Mailer($smtpHost, $smtpPort)`.
- **Value Object Constructors**: `new Money($amount, $currency)`.

---

### 10. `$this`

#### Definitions

**Core Definition**
`$this` is a pseudo-variable available inside non-static methods that refers to the current object instance.

**Technical Definition**
`$this` is automatically bound to the object on which a non-static method is called. It allows the method to access the object's properties (`$this->property`) and other methods (`$this->method()`). `$this` is not available in static methods. When a method is called on a child object, `$this` refers to the child instance, even if the method is defined in the parent class.

**Beginner-Friendly Explanation**
`$this` is how an object refers to itself inside its own methods. When you tell a `User` object to `changeEmail()`, `$this` is that specific user object.

#### Purposes

- To access and modify the current object's properties.
- To call other methods on the same object.
- To enable fluent interfaces by returning `$this`.

#### Syntax Rules and Structure

**General Syntax**
```php
$this->property;
$this->method();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$this` | The current object instance. |
| `->` | Object operator. |
| `property` / `method` | Member name. |

**Syntax Rules**

- `$this` is only available in non-static methods.
- `$this` can be returned for fluent chaining: `return $this;`.
- `$this` cannot be reassigned.

**Constraints and Limitations**

- **Not available in static methods**: Using `$this` in a static method causes an `Error`.
- **`$this` is always the calling object**: In inheritance, `$this` refers to the actual runtime object, not the declaring class.

#### Annotated Code Examples

**Example 1: Fluent Interface with `$this`**

```php
<?php
class QueryBuilder
{
    private array $parts = [];

    public function select(string $columns): static
    {
        $this->parts[] = "SELECT $columns";
        return $this;
    }

    public function from(string $table): static
    {
        $this->parts[] = "FROM $table";
        return $this;
    }

    public function getQuery(): string
    {
        return implode(' ', $this->parts);
    }
}

$query = (new QueryBuilder())
    ->select('*')
    ->from('users')
    ->getQuery();

echo $query . "\n"; // Outputs: SELECT * FROM users
?>
```

**Expected Output:**
```
SELECT * FROM users
```

**Why This Output:**  
Each fluent method returns `$this`, allowing method chaining. The final `getQuery()` returns the composed SQL string.

#### Real-World Cases

- **Fluent Builders**: Query builders, HTTP clients, form builders.
- **Domain Methods**: `$this->validate()` called from within a service method.
- **Event Dispatching**: `$this->dispatch(new Event())`.

---

### 11. `self`

#### Definitions

**Core Definition**
`self` is a keyword that refers to the class in which it is written (the declaring class). It is resolved at compile time.

**Technical Definition**
`self` is used with the scope resolution operator (`::`) to access static members, constants, and static methods of the current class. Unlike `static`, `self` does not use late static binding; it always refers to the class where the code is defined, even when inherited. `self` is commonly used for constants and static methods that should not be overridden.

**Beginner-Friendly Explanation**
`self` means “this exact class, the one where the code is written.” It does not change when inherited. If a child class calls a parent method that uses `self`, `self` still refers to the parent class.

#### Purposes

- To access the declaring class's constants and static members.
- To instantiate the declaring class (e.g., `new self()`).
- To avoid late static binding when the exact class is required.

#### Syntax Rules and Structure

**General Syntax**
```php
self::CONSTANT;
self::$staticProperty;
self::staticMethod();
new self();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `self` | The declaring class. |
| `::` | Scope resolution operator. |

**Syntax Rules**

- `self` is available in both static and non-static methods.
- `self` cannot be used to access instance properties (`$this->property` is required for those).

**Constraints and Limitations**

- **No late static binding**: `self` does not resolve to subclasses.
- **Cannot access non-static members** with `self::`.

#### Annotated Code Examples

**Example 1: `self` vs. `static`**

```php
<?php
class A
{
    public static function createSelf(): self
    {
        return new self();
    }

    public static function createStatic(): static
    {
        return new static();
    }
}

class B extends A {}

var_dump(A::createSelf());   // object(A)
var_dump(B::createSelf());   // object(A) — still A!
var_dump(B::createStatic()); // object(B) — late static binding
?>
```

**Expected Output:**
```
object(A)#1 (0) {}
object(A)#1 (0) {}
object(B)#1 (0) {}
```

**Why This Output:**  
`self` always resolves to the declaring class `A`, even when called as `B::createSelf()`. `static` uses late static binding and resolves to the called class `B`.

#### Real-World Cases

- **Constants**: `self::MAX_ATTEMPTS`.
- **Static Factory Methods**: `new self()` for guaranteed declaring-class instantiation.
- **Singleton Patterns**: `self::$instance ??= new self();`.

---

### 12. `parent`

#### Definitions

**Core Definition**
`parent` is a keyword that refers to the immediate parent class of the class in which it is used.

**Technical Definition**
`parent::` is used to access static members, constants, and methods of the parent class. It is commonly used to call the parent constructor (`parent::__construct()`) or to invoke a parent method that has been overridden in the child class. `parent` is resolved at compile time to the parent of the declaring class.

**Beginner-Friendly Explanation**
`parent` means “my parent class.” It lets a child class reuse or extend the behavior of its parent.

#### Purposes

- To call the parent constructor when the child defines its own.
- To invoke a parent method that has been overridden.
- To access parent constants and static members.

#### Syntax Rules and Structure

**General Syntax**
```php
parent::__construct();
parent::method();
parent::CONSTANT;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `parent` | The immediate parent class. |
| `::` | Scope resolution operator. |

**Syntax Rules**

- `parent` is available only inside a class that has a parent.
- `parent::__construct()` is required if the parent constructor must run.

**Constraints and Limitations**

- **No grandparent access**: `parent` refers only to the immediate parent; there is no `grandparent` keyword (use the parent's name explicitly if needed).

#### Annotated Code Examples

**Example 1: Calling the Parent Constructor**

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
        parent::__construct($name);   // required to initialize parent state
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
The child constructor calls `parent::__construct($name)` to initialize the inherited `$name` property. Without this call, the parent constructor would not run, and `$name` would be uninitialized.

#### Real-World Cases

- **Controller Inheritance**: `parent::__construct()` to initialize a base controller.
- **Exception Hierarchies**: `parent::__construct($message, $code, $previous)`.
- **Template Method Pattern**: Overriding a method but calling `parent::method()` for shared behavior.

---

### 13. `static`

#### Definitions

**Core Definition**
`static` is a keyword with two related meanings in PHP classes: (1) it declares static properties and methods that belong to the class rather than to instances; and (2) as a return type or scope-resolution keyword, it refers to the runtime class using **late static binding**.

**Technical Definition**
As a **member modifier**, `static` makes a property or method belong to the class itself, accessible via `ClassName::$property` or `ClassName::method()` without an instance. As a **scope-resolution keyword** (`static::`), it uses late static binding to refer to the class that was actually called at runtime, even if the method is defined in a parent class. As a **return type**, `static` (PHP 8.0+) requires the returned value to be an instance of the called class.

**Beginner-Friendly Explanation**
`static` means “belongs to the class, not to any particular object.” A static property is shared by all objects of the class. When used as `static::`, it means “the class that was actually used to call this method,” which is useful when a child class inherits a method but wants it to return the child type.

#### Purposes

- To share data across all instances of a class (static properties).
- To provide utility methods that do not depend on object state (static methods).
- To enable late static binding for inheritance-friendly factories and fluent methods.
- To declare return types that preserve the called class.

#### Syntax Rules and Structure

**General Syntax**
```php
// Static property
visibility static Type $property = default;

// Static method
visibility static function method(): ReturnType { ... }

// Late static binding
static::method();
static::CONSTANT;
static::$property;

// Static return type
function create(): static { return new static(); }
```

**Component Breakdown**

| Usage | Description |
|-------|-------------|
| Static property | Belongs to the class, shared by all instances. |
| Static method | No `$this`; called on the class. |
| `static::` | Late static binding; resolves to the called class. |
| `: static` | Return type requiring an instance of the called class. |

**Syntax Rules**

- Static members are accessed with `::`.
- Inside static methods, `$this` is not available; use `self::` or `static::`.
- `static::` can access static properties, constants, and methods.

**Constraints and Limitations**

- **Static methods cannot use `$this`**.
- **Static properties are shared**: Modifying a static property in one instance affects all instances (unless the property is redeclared in a subclass).
- **Late static binding only works with `static::`**, not `self::`.

#### Annotated Code Examples

**Example 1: Static Property Shared Across Instances**

```php
<?php
class Counter
{
    private static int $count = 0;

    public function __construct()
    {
        self::$count++;
    }

    public static function getCount(): int
    {
        return self::$count;
    }
}

new Counter();
new Counter();
new Counter();

echo Counter::getCount() . "\n"; // Outputs: 3
?>
```

**Expected Output:**
```
3
```

**Why This Output:**  
`$count` is static and shared by all instances. Each `new Counter()` increments the same static property. `getCount()` is static and returns the shared value.

---

**Example 2: Late Static Binding with `static`**

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
`static` in `create()` uses late static binding. When called as `User::create()`, `static` resolves to `User`, so `new static()` creates a `User` instance, not a `Model` instance.

#### Real-World Cases

- **Active Record / ORM**: `Model::find($id)` returns an instance of the called subclass.
- **Named Constructors**: `static::fromArray($data)` preserves the called class.
- **Counters and Registries**: Static properties for global counters or singleton instances.

---

### 14. Class Best Practices

#### Definitions

**Core Definition**
Class best practices are guidelines for designing classes that are cohesive, loosely coupled, testable, and maintainable, following community standards and object-oriented principles.

**Technical Definition**
Best practices for PHP classes include: **PSR-12** coding style (StudlyCaps class names, one class per file, 4-space indentation); **`declare(strict_types=1)`** in every file; **SOLID principles** (Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion); **favor composition over inheritance**; **use dependency injection** rather than instantiating dependencies internally; **keep classes small and cohesive**; **use `readonly` properties** for immutable value objects; and **prefer private/protected visibility** with public getters or property hooks.

**Beginner-Friendly Explanation**
Best practices are the “good habits” for writing classes: give them clear names, keep them focused on one job, don't let them depend on too many other things, and use modern PHP features (types, readonly, strict types) to catch mistakes early.

#### Purposes

- To improve readability, consistency, and maintainability across a codebase.
- To reduce coupling and increase testability through dependency injection.
- To leverage PHP's type system for safety and static analysis.
- To align with community standards (PSR) for interoperability.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Follow PSR-12** | StudlyCaps class names, one class per file, 4-space indentation, opening brace on its own line. |
| **Enable strict types** | `declare(strict_types=1);` as the first statement. |
| **Apply SOLID principles** | Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion. |
| **Favor composition over inheritance** | Inject collaborators rather than extending deep hierarchies. |
| **Use dependency injection** | Pass dependencies via the constructor; do not instantiate them internally. |
| **Keep classes small and focused** | A class should have one reason to change. |
| **Use `readonly` for immutable value objects** | Prevents accidental mutation. |
| **Prefer private/protected with accessors** | Encapsulate state; expose only what is necessary. |
| **Document with PHPDoc** | `@param`, `@return`, `@throws`, and generic array shapes for static analysis. |

**Constraints and Limitations**

- **SOLID is a guide, not a law**: Over-application can lead to excessive abstraction and complexity.
- **PSR-12 does not cover everything**: Architectural decisions (e.g., how to structure services) are project-specific.

#### Annotated Code Examples

**Example 1: A Well-Designed Class Following Best Practices**

```php
<?php
declare(strict_types=1);

namespace App\Domain;

use InvalidArgumentException;

final class Money
{
    public function __construct(
        private readonly int $amountInCents,
        private readonly string $currency
    ) {
        if ($amountInCents < 0) {
            throw new InvalidArgumentException('Amount cannot be negative.');
        }
        if (strlen($currency) !== 3) {
            throw new InvalidArgumentException('Currency must be a 3-letter code.');
        }
    }

    public function add(self $other): self
    {
        $this->assertSameCurrency($other);
        return new self($this->amountInCents + $other->amountInCents, $this->currency);
    }

    public function getAmount(): string
    {
        return number_format($this->amountInCents / 100, 2) . ' ' . $this->currency;
    }

    private function assertSameCurrency(self $other): void
    {
        if ($this->currency !== $other->currency) {
            throw new InvalidArgumentException('Cannot add different currencies.');
        }
    }
}

$price = new Money(1999, 'USD');
$tax = new Money(500, 'USD');
echo $price->add($tax)->getAmount() . "\n"; // Outputs: 24.99 USD
?>
```

**Expected Output:**
```
24.99 USD
```

**Why This Output:**  
This class follows best practices: `final`, `strict_types=1`, `readonly` properties, constructor validation, immutable operations (returning a new instance), explicit return types, private helper methods, and a clear domain name (`Money`). It has a single responsibility: representing a monetary amount.

#### Real-World Cases

- **Laravel/Symfony Projects**: Both frameworks enforce PSR-12, dependency injection, and SOLID principles in their coding standards.
- **Value Objects**: `Money`, `Email`, `UserId` — immutable, self-validating, final classes.
- **Service Classes**: Small, focused classes with dependencies injected via the constructor.

---

## References

**PHP Manual – Classes and Objects** - https://static.php.net/manual/en/language.oop5.php

**PHP Manual – The Basics (class keyword, $this)** - https://svn.php.net/viewvc/phpdoc/en/trunk/language/oop5/basic.xml?revision=337758&view=markup&pathrev=338185

**PHP Manual – Visibility** - https://static.php.net/manual/en/language.oop5.visibility.php

**PHP Manual – Class Constants** - https://static.php.net/manual/en/language.oop5.constants.php

**PHP Manual – Static Keyword** - https://static.php.net/manual/en/language.oop5.static.php

**PHP Manual – Constructors and Destructors** - https://static.php.net/manual/en/language.oop5.decon.php

**PHP Manual – Relative class types (self, parent, static)** - https://www.php.net/manual/en/language.types.relative-class-types.php

**PHP 8.4: Property Hooks** - https://www.php.net/manual/en/language.oop5.property-hooks.php

**PHP 8.4: Asymmetric Visibility** - https://www.php.net/manual/en/language.oop5.visibility.php#language.oop5.visibility-members-aviz

**PHP Foundation – PHP 8.4 Released (Property Hooks, Asymmetric Visibility)** - https://thephp.foundation/blog/2024/11/21/php-core-roundup-20-php84/

**PSR-12: Extended Coding Style** - https://www.php-fig.org/psr/psr-12/

**PHP Best Practices – SOLID and PSR-12 (rtcamp)** - https://rtcamp.com/blog/php-best-practices/