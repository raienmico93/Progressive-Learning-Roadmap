Here is a comprehensive research cheat sheet on **PHP Constructors and Destructors**, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Constructors and Destructors

### Definitions

**Core Definition**
A **constructor** (`__construct()`) is a special method that is automatically invoked when a new object is created. It is used to initialize the object's state before the object is used. A **destructor** (`__destruct()`) is a special method that is automatically invoked when an object is destroyed (when no references to it remain or at script shutdown). It is used to perform cleanup tasks before the object is removed from memory.

**Technical Definition**
In PHP, `__construct()` and `__destruct()` are **magic methods**. The constructor is called on each newly-created object and is suitable for any initialization the object may need before it is used. The destructor is called as soon as all references to a particular object are removed, or in any order during the shutdown sequence. Unlike other methods, `__construct()` is exempt from the usual signature compatibility rules (Liskov Substitution Principle) when being extended, meaning a child class may define a constructor with a completely different signature than its parent. Parent constructors and destructors are **not called implicitly**; a child class must explicitly call `parent::__construct()` and `parent::__destruct()` if the parent's implementations are needed. Since PHP 8.0.0, constructor parameters may be **promoted** to class properties, combining property declaration and constructor assignment into a single concise syntax.

**Beginner-Friendly Explanation**
A constructor is the “setup routine” for an object. When you create a new object with `new`, the constructor runs automatically to put the object into a valid starting state—like filling out a form before you can submit it. A destructor is the “cleanup routine.” When an object is no longer needed, the destructor runs to close files, release database connections, or tidy up other resources. PHP calls them automatically at the right times, so you do not have to remember to do it yourself.

---

### Key Characteristics

- **Automatic Invocation**: Constructors run on `new`; destructors run on object destruction or script shutdown.
- **Magic Method Names**: `__construct()` and `__destruct()` are reserved magic method names.
- **No Implicit Parent Calls**: Child classes must explicitly call `parent::__construct()` and `parent::__destruct()`.
- **Constructor Exemption from LSP**: Constructors are exempt from signature compatibility rules in inheritance.
- **Promotion (PHP 8.0+)**: Constructor parameters with a visibility modifier are automatically promoted to properties.
- **Destructors Take No Parameters and Return `void`**.
- **Destructors Are Not Guaranteed**: If the script terminates due to a fatal error or an uncaught exception, destructors may not be called.
- **Throwing from Destructors Is Dangerous**: Throwing an exception from a destructor during script shutdown causes a fatal error.

---

### Prerequisites

- **PHP 8.0+ Recommended**: Constructor property promotion requires PHP 8.0.0. `readonly` promotion requires PHP 8.1.
- **Basic Class and Object Syntax**: Understanding of `class`, `new`, `$this`, and method declarations.
- **Awareness of Inheritance**: Constructors and destructors interact with parent and child classes in specific ways.

---

### Related Programming Areas

- **Object Lifecycle Management**: Constructors and destructors bracket an object's lifetime.
- **Dependency Injection**: Constructors are the primary mechanism for injecting dependencies.
- **Value Objects and DTOs**: Promotion and `readonly` make constructors concise for immutable data.
- **Resource Management**: Destructors are used for cleanup of files, sockets, and database connections.
- **Magic Methods**: `__construct` and `__destruct` are part of PHP's magic method family.

---

## Core Concepts / Features

---

### 1. `__construct()` (Constructor)

#### Definitions

**Core Definition**
`__construct()` is the magic method that PHP calls automatically when a new object is created. It initializes the object's properties and validates invariants.

**Technical Definition**
A constructor is declared as `public function __construct(mixed ...$values = ""): void`. Classes that have a constructor method call this method on each newly-created object. Constructors may define an arbitrary number of arguments, which may be required, typed, and have default values. Constructor arguments are passed by placing them in parentheses after the class name (`new ClassName($arg1, $arg2)`). If a child class defines a constructor, the parent constructor is **not** called implicitly; an explicit call to `parent::__construct()` is required. If the child does not define a constructor, the parent's constructor is inherited (unless it was declared `private`). Prior to PHP 8.0.0, a method named the same as the class acted as an “old-style” constructor in the global namespace; this is deprecated and should never be used in new code.

**Beginner-Friendly Explanation**
The constructor is the object's “birth” method. It runs the moment you create an object, making sure the object has everything it needs to start working correctly. For example, a `User` object's constructor might require an email address and ensure it is valid before the object is allowed to exist.

#### Purposes

- To initialize an object's properties with the values provided at creation time.
- To validate invariants and reject invalid objects before they are used.
- To inject dependencies (services, repositories, configuration) into the object.
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
| `__construct` | The reserved magic method name. |
| Parameters | Typed, with optional defaults and variadics. |
| Body | Initialization logic; may call `parent::__construct()`. |
| Visibility | May be `public`, `protected`, or `private`. |

**Syntax Rules**

- A class may have at most one constructor.
- Constructor arguments are passed via `new ClassName(...)`.
- If a child defines a constructor, the parent constructor is not called automatically; use `parent::__construct()`.
- Constructors are exempt from LSP signature compatibility rules in inheritance.
- If the class has no constructor or the constructor has no required arguments, parentheses after `new ClassName` may be omitted.

**Constraints and Limitations**

- **Constructor exceptions**: If the constructor throws, the object is not fully created.
- **`private` constructors**: Prevent direct instantiation from outside the class (used in singletons and factories).
- **Old-style constructors are deprecated**: Always use `__construct()` in new code.

#### Annotated Code Examples

**Example 1: Basic Constructor with Validation**

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

**Step-by-Step Setup:**
1. Save as `constructor_validation.php`.
2. Run `php constructor_validation.php`.
3. The output confirms the valid email was accepted.

---

**Example 2: Inheritance and `parent::__construct()`**

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
        parent::__construct($name); // required
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
`Dog`'s constructor calls `parent::__construct($name)` to initialize the inherited `$name` property. Without this call, `$name` would remain uninitialized.

---

#### Real-World Cases

- **Dependency Injection**: `public function __construct(private UserRepository $users, private LoggerInterface $logger) {}`.
- **Value Objects**: `new Money(1000, 'USD')` with validation in the constructor.
- **DTOs**: `new CreateUserCommand($email, $name)` with promoted readonly properties.

---

### 2. `__destruct()` (Destructor)

#### Definitions

**Core Definition**
`__destruct()` is the magic method that PHP calls automatically when an object is no longer referenced or when the script is shutting down. It performs cleanup tasks.

**Technical Definition**
A destructor is declared as `public function __destruct(): void`. It takes no parameters and returns `void`. PHP calls destructors as soon as objects are no longer available (e.g., when a variable is `unset()`, set to `null`, goes out of scope, or at script shutdown). Destructors are called even if the script is terminated with `exit()`. However, if the script terminates due to a fatal error or an uncaught exception, destructors are **not** called. Destructors are not called implicitly in parent classes; a child class that defines its own destructor should explicitly call `parent::__destruct()`, typically as the last statement, to ensure top-down destruction. Throwing an exception from a destructor during script shutdown causes a fatal error and is strongly discouraged.

**Beginner-Friendly Explanation**
The destructor is the object's “death” method. When an object is about to be removed from memory, PHP calls this method so the object can clean up after itself—close a file, release a lock, or disconnect from a database. You do not call it yourself; PHP does it automatically at the right time.

#### Purposes

- To release resources such as file handles, database connections, sockets, and locks.
- To perform cleanup tasks that must happen when an object is no longer needed.
- To deregister the object from registries or listener lists to prevent memory leaks.
- To explicitly `unset()` large object properties to reduce memory usage in long-running scripts.

#### Syntax Rules and Structure

**General Syntax**
```php
public function __destruct(): void
{
    // cleanup
    parent::__destruct(); // if needed
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `__destruct` | The reserved magic method name. |
| No parameters | Destructors take no arguments. |
| `: void` | Destructors return nothing. |
| `parent::__destruct()` | Optional; call last to ensure top-down cleanup. |

**Syntax Rules**

- A class may have at most one destructor.
- Destructors take no parameters.
- Parent destructors are not called implicitly; use `parent::__destruct()`.
- Call `parent::__destruct()` **after** local cleanup code so that parent resources are available while the child cleans up.

**Constraints and Limitations**

- **Not guaranteed on fatal errors**: If the script terminates due to an uncaught exception or fatal error, destructors may not run.
- **Throwing exceptions is fatal**: Attempting to throw an exception from a destructor during script shutdown causes a fatal error.
- **No return value**: Destructors cannot return a value.

#### Annotated Code Examples

**Example 1: Basic Destructor**

```php
<?php
class MyClass
{
    public function __construct()
    {
        echo "Object created.\n";
    }

    public function __destruct()
    {
        echo "Object destroyed.\n";
    }
}

$obj = new MyClass();   // Outputs: Object created.
unset($obj);            // Outputs: Object destroyed.
?>
```

**Expected Output:**
```
Object created.
Object destroyed.
```

**Why This Output:**  
The constructor runs on `new`. When `$obj` is `unset()`, the reference count drops to zero, and PHP immediately calls the destructor.

---

**Example 2: Destructor with Resource Cleanup**

```php
<?php
class FileWriter
{
    private $handle;

    public function __construct(string $path)
    {
        $this->handle = fopen($path, 'w');
        if ($this->handle === false) {
            throw new RuntimeException("Cannot open file: $path");
        }
    }

    public function write(string $content): void
    {
        fwrite($this->handle, $content);
    }

    public function __destruct()
    {
        if (is_resource($this->handle)) {
            fclose($this->handle);
        }
    }
}

$writer = new FileWriter('/tmp/example.txt');
$writer->write("Hello, world!");
// Destructor runs at script end or when $writer is unset, closing the file
?>
```

**Expected Output:**
No visible output; the file is written and closed automatically.

**Why This Output:**  
The destructor checks whether the file handle is still a resource and closes it. This ensures the file is properly flushed and released even if the developer forgets to call `fclose()`.

---

**Example 3: Calling `parent::__destruct()`**

```php
<?php
class ParentClass
{
    public function __destruct()
    {
        echo "Parent destructor.\n";
    }
}

class ChildClass extends ParentClass
{
    public function __destruct()
    {
        echo "Child cleanup.\n";
        parent::__destruct(); // called last
    }
}

$child = new ChildClass();
unset($child);
?>
```

**Expected Output:**
```
Child cleanup.
Parent destructor.
```

**Why This Output:**  
The child destructor performs its local cleanup first, then calls `parent::__destruct()` as the last statement, ensuring the parent's cleanup runs after the child's and that parent resources are not destroyed before the child finishes using them.

---

#### Real-World Cases

- **Database Connections**: A `DatabaseConnection` class closes the connection in its destructor.
- **File Locks**: A `LockFile` class releases the lock in its destructor.
- **Temporary Files**: A `TempFile` class deletes the temporary file in its destructor.
- **Listener Deregistration**: An event listener unregisters itself from the dispatcher in its destructor to prevent memory leaks.

---

### 3. Constructor Promotion (PHP 8.0+)

#### Definitions

**Core Definition**
Constructor property promotion is a PHP 8.0 feature that allows constructor parameters to be declared with a visibility modifier (`public`, `protected`, `private`, or `readonly`), automatically creating a property with the same name and assigning the argument to it.

**Technical Definition**
When a method parameter is prefixed with one of the visibility keywords, it is considered “promoted.” For each promoted parameter, PHP adds a property with the same name and includes a forwarding assignment to that property in the constructor body. The promoted parameter replaces the need for a separate property declaration, a constructor parameter, and a manual assignment. The syntax is strictly equivalent to writing the property, parameter, and assignment separately, but more concise. Promotion may be combined with `readonly` (PHP 8.1+) for immutable properties. Constraints include: promotion is only allowed in non-abstract constructors; the `var` keyword is not supported; variadic parameters cannot be promoted; `callable` is not allowed as a promoted property type; and nullability must be explicit (`?Type`), not inferred from a `null` default.

**Beginner-Friendly Explanation**
Constructor promotion is a shortcut. Instead of writing the same name three times (property, parameter, assignment), you write it once in the constructor signature. PHP does the rest: it creates the property and assigns the value automatically.

#### Purposes

- To reduce boilerplate for simple value objects and dependency injection.
- To eliminate the risk of mismatches between property declarations and constructor assignments.
- To make constructors more concise and readable.

#### Syntax Rules and Structure

**General Syntax**
```php
public function __construct(
    public Type $property1,
    protected Type $property2,
    private Type $property3 = default
) {}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `public` / `protected` / `private` | Visibility modifier; makes the parameter a promoted property. |
| `readonly` | PHP 8.1+; immutable promoted property. |
| `Type $name` | The property type and name. |
| `= default` | Optional default value. |

**Syntax Rules**

- Only non-abstract constructors may have promoted parameters.
- The `var` keyword is not allowed.
- A property cannot be declared both normally and via promotion (no redeclaration).
- `callable` is not a valid promoted property type.
- Nullability must be explicit: use `?Type $prop = null`, not `Type $prop = null`.
- Variadic parameters cannot be promoted.
- Promotion may be combined with `readonly` for immutable properties.

**Constraints and Limitations**

- **Significant constructor logic**: If the constructor body contains substantial validation or transformation, promotion can bury the logic among the parameter list; traditional declarations may be clearer.
- **Computed properties**: Properties calculated in the constructor body cannot be promoted and must be declared separately.

#### Annotated Code Examples

**Example 1: Before and After Promotion**

```php
<?php
// BEFORE: Repetitive boilerplate
class PointBefore
{
    public float $x;
    public float $y;

    public function __construct(float $x = 0.0, float $y = 0.0)
    {
        $this->x = $x;
        $this->y = $y;
    }
}

// AFTER: Constructor promotion
class PointAfter
{
    public function __construct(
        public float $x = 0.0,
        public float $y = 0.0,
    ) {}
}

$p = new PointAfter(3.0, 4.0);
echo "({$p->x}, {$p->y})\n"; // Outputs: (3, 4)
?>
```

**Expected Output:**
```
(3, 4)
```

**Why This Output:**  
The promoted constructor creates `$x` and `$y` properties and assigns the arguments to them. The result is identical to the manual version, with far less code.

---

**Example 2: Promotion with `readonly` and Validation**

```php
<?php
declare(strict_types=1);

final class Money
{
    public function __construct(
        public readonly int $amountCents,
        public readonly string $currency = 'USD',
    ) {
        if ($amountCents < 0) {
            throw new InvalidArgumentException('Amount cannot be negative.');
        }
        if (strlen($currency) !== 3) {
            throw new InvalidArgumentException('Currency must be a 3-letter code.');
        }
    }
}

$price = new Money(1999, 'USD');
echo $price->amountCents . ' ' . $price->currency . "\n"; // Outputs: 1999 USD
?>
```

**Expected Output:**
```
1999 USD
```

**Why This Output:**  
The promoted `readonly` properties are initialized once with the constructor arguments. The validation logic in the constructor body ensures the object is never created in an invalid state, while promotion eliminates property declaration boilerplate.

---

**Example 3: Mixing Promoted and Non-Promoted Properties**

```php
<?php
declare(strict_types=1);

final class Order
{
    private string $orderNumber;

    public function __construct(
        public readonly OrderId $id,
        public readonly array $lines,
        public readonly \DateTimeImmutable $createdAt,
    ) {
        // Computed property — cannot be promoted
        $this->orderNumber = sprintf(
            'ORD-%s-%s',
            $createdAt->format('Ymd'),
            strtoupper(substr($id->value, 0, 8)),
        );
    }

    public function getOrderNumber(): string
    {
        return $this->orderNumber;
    }
}
?>
```

**Expected Output:**
No output (definitions only). The computed `$orderNumber` is declared separately and assigned in the constructor body.

**Why This Output:**  
Properties that are computed from other values cannot be promoted because promotion only performs a direct assignment. Mixing promoted and non-promoted properties is allowed and appropriate for such cases.

---

#### Real-World Cases

- **Dependency Injection**: `public function __construct(private UserRepository $users, private LoggerInterface $logger) {}` reduces service class boilerplate.
- **DTOs and Commands**: `public readonly string $email`, `public readonly string $name` for immutable data transfer.
- **Value Objects**: `Money`, `Email`, `UserId` with promotion and validation.

---

### 4. Constructor and Destructor Best Practices

#### Definitions

**Core Definition**
Best practices for constructors and destructors are guidelines for using these magic methods safely, predictably, and in a way that supports testability and maintainability.

**Technical Definition**
Best practices include: keeping constructors **simple and focused on initialization**; **validating invariants** in the constructor; **injecting dependencies** via the constructor rather than instantiating them internally; using **constructor promotion** for concise value objects and dependency injection; **avoiding significant logic** in constructors that would obscure promoted properties; **calling `parent::__construct()`** when extending a class with a constructor; **calling `parent::__destruct()` last** when extending a class with a destructor; **never throwing exceptions from destructors**; and using destructors only for **resource cleanup**, not for business logic.

**Beginner-Friendly Explanation**
Best practices for constructors and destructors are the rules of good object hygiene: constructors should set up the object cleanly and validate it, destructors should clean up resources without causing trouble, and both should be predictable and simple.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Keep constructors simple** | A constructor's role is to initialize an object, not to perform complex business logic. |
| **Validate invariants in the constructor** | Reject invalid objects before they can be used. |
| **Inject dependencies via the constructor** | Pass required collaborators as parameters; do not instantiate them internally. |
| **Use constructor promotion for DTOs and DI** | Collapse property declaration, parameter, and assignment into one concise syntax. |
| **Call `parent::__construct()` when extending** | Parent constructors are not called implicitly. |
| **Call `parent::__destruct()` last** | Ensure top-down destruction and that parent resources are available during child cleanup. |
| **Never throw exceptions from destructors** | Throwing during shutdown causes a fatal error. |
| **Use destructors only for resource cleanup** | Close files, connections, locks, and sockets; do not perform business logic. |
| **Explicitly `unset()` large properties in destructors** | Helps reduce memory usage in long-running scripts. |
| **Prefer `readonly` for immutable promoted properties** | Prevents accidental mutation after construction. |

**Constraints and Limitations**

- **Destructors are not guaranteed** on fatal errors or uncaught exceptions; do not rely on them for critical cleanup that must always happen (use `finally` or `register_shutdown_function()` instead).
- **Promotion can hurt readability** when the constructor body has significant logic; use traditional declarations in those cases.
- **`private` constructors** are useful for singletons and factories but prevent direct instantiation.

#### Annotated Code Examples

**Example 1: A Well-Designed Class with Constructor Promotion and Validation**

```php
<?php
declare(strict_types=1);

final class Email
{
    public function __construct(
        public readonly string $value,
    ) {
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email: {$value}");
        }
    }
}

$email = new Email('alice@example.com');
echo $email->value . "\n"; // Outputs: alice@example.com
?>
```

**Expected Output:**
```
alice@example.com
```

**Why This Output:**  
This class follows best practices: `final`, `strict_types=1`, a `readonly` promoted property for immutability, validation in the constructor to enforce the invariant, and a clear, focused constructor that does nothing beyond initialization and validation.

---

**Example 2: Destructor for Resource Cleanup (Without Throwing)**

```php
<?php
class DatabaseConnection
{
    private ?PDO $pdo = null;

    public function __construct(string $dsn, string $user, string $pass)
    {
        $this->pdo = new PDO($dsn, $user, $pass);
    }

    public function query(string $sql): array
    {
        return $this->pdo->query($sql)->fetchAll();
    }

    public function __destruct()
    {
        // Cleanup: close the connection (no exceptions thrown)
        $this->pdo = null;
    }
}

$db = new DatabaseConnection('sqlite::memory:', '', '');
// Destructor runs at script end, releasing the connection
?>
```

**Expected Output:**
No visible output; the database connection is released automatically.

**Why This Output:**  
The destructor sets the PDO property to `null`, releasing the connection. It does not throw exceptions or perform complex logic, following the best practice of using destructors only for safe resource cleanup.

---

#### Real-World Cases

- **Laravel/Symfony Services**: Constructors use promotion and dependency injection to receive repositories, loggers, and configuration.
- **Value Objects**: `Email`, `Money`, `UserId` use promoted `readonly` properties with constructor validation.
- **Resource Wrappers**: `FileWriter`, `SocketClient`, `LockFile` use destructors for automatic cleanup.
- **Long-Running CLI Scripts**: Destructors explicitly `unset()` large properties to keep memory usage low.

---

## References

**PHP Manual – Constructors and Destructors** - https://static.php.net/manual/en/language.oop5.decon.php

**PHP RFC: Constructor Property Promotion** - https://wiki.php.net/rfc/constructor_promotion

**PHP Manual – Constructor Promotion (section)** - https://static.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion

**Exakat – Should Have Destructor (resource cleanup best practice)** - https://exakat.readthedocs.io/en/latest/Reference/Rules/Classes/ShouldHaveDestructor.html

**Exakat – Throw In Destruct (avoid exceptions in destructors)** - https://exakat.readthedocs.io/en/latest/Reference/Rules/Classes/ThrowInDestruct.html

**Compile-N-Run – PHP Destructors (lifecycle and cleanup)** - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/php/10-php-oop/3-php-destructors.mdx

**PHP Language Specification – Destructors (parent::__destruct() ordering)** - https://github.com/php/php-langspec/blob/master/spec/14-classes.md

**PHP Agent Skills – Modern Constructor Promotion (best practices)** - https://github.com/voku/agent-skills/blob/main/skills/php-best-practices/rules/modern-constructor-promotion.md

**Secure PHP Development – Constructor Property Promotion (readonly examples)** - https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Modern-PHP/Constructor-Property-Promotion.md

**PHP Bug #47143 – Throwing an exception in a destructor causes a fatal error** - https://bugs.php.net/bug.php?id=47143