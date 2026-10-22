Here is a comprehensive research cheat sheet on **PHP Properties**, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Properties

### Definitions

**Core Definition**
A **property** is a variable member of a class that holds data (state) for an object or for the class itself. Properties define what an object “knows” or “contains,” distinguishing them from methods, which define what an object “does.”

**Technical Definition**
In PHP, a property is a class variable declared with at least one modifier (visibility, `static`, or `readonly`), an optional type declaration (PHP 7.4+), and a variable name. The declaration may include an initializer, but the initial value must be a **constant expression**—something evaluable at compile time, such as a literal, a constant, or a simple arithmetic expression of literals. Properties declared without an explicit visibility keyword default to `public`. Instance properties are accessed via `$this->property`; static properties via `self::$property` or `ClassName::$property`. Since PHP 7.4, properties may be typed; since PHP 8.1, they may be `readonly`; and since PHP 8.4, they may use **property hooks** and **asymmetric visibility** to control read and write behaviour separately.

**Beginner-Friendly Explanation**
A property is a “field” or “attribute” that belongs to a class. If a `User` class represents a user, its properties might be `$name`, `$email`, and `$age`. Each object created from the class has its own values for these properties. Think of a property as a labelled box attached to an object: you can put data in it, read it, and sometimes restrict who is allowed to look inside or change its contents.

---

### Key Characteristics

- **State Container**: Properties hold the data that defines an object’s state.
- **Visibility Control**: `public`, `protected`, and `private` determine from where a property can be read or written.
- **Type Safety (PHP 7.4+)**: Typed properties enforce that only values of the declared type can be assigned.
- **Immutability (PHP 8.1+)**: `readonly` properties can be initialized exactly once and never modified thereafter.
- **Class-Level Storage**: `static` properties belong to the class itself, not to individual instances.
- **Constant-Expression Initializers**: Default values must be evaluable at compile time; runtime function calls are not permitted.
- **Modern Access Control (PHP 8.4)**: Property hooks allow custom `get`/`set` logic; asymmetric visibility allows different scopes for reading and writing.

---

### Prerequisites

- **PHP 7.4+** for typed properties; **PHP 8.1+** for `readonly`; **PHP 8.4+** for property hooks and asymmetric visibility.
- **Basic Class Syntax**: Understanding of `class`, `$this`, and object instantiation with `new`.
- **`declare(strict_types=1)` Awareness**: Strict types affect how property type declarations are enforced during assignment.

---

### Related Programming Areas

- **Object-Oriented Programming (OOP)**: Encapsulation, information hiding, and state management.
- **Type Systems**: Typed properties, union types, nullable types, and `readonly`.
- **Design Patterns**: Value Objects, Data Transfer Objects (DTOs), and Entities rely heavily on well-designed properties.
- **Static Analysis**: PHPStan and Psalm infer property types and flag violations, especially with typed and `readonly` properties.
- **Property Hooks (PHP 8.4)**: A modern alternative to getter/setter boilerplate.

---

## Core Concepts / Features

---

### 1. Properties (General)

#### Definitions

**Core Definition**
Properties are the variable members of a class that store an object’s data.

**Technical Definition**
A property declaration consists of optional modifiers (`public`, `protected`, `private`, `static`, `readonly`), an optional type declaration, and a variable name, optionally followed by an initializer that must be a constant expression. Properties are accessed with the object operator (`->`) for instance properties and the scope resolution operator (`::`) for static properties.

**Beginner-Friendly Explanation**
Properties are the “attributes” of an object. They are the pieces of data that make each object unique—like a person’s name, age, and email address.

#### Purposes

- To store the state of an object or class.
- To encapsulate data behind visibility modifiers, controlling how it is accessed and modified.
- To provide a clear, typed contract for what data an object holds.

#### Syntax Rules and Structure

**General Syntax**
```php
[visibility] [static] [readonly] [type] $propertyName [= constantExpression];
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `visibility` | `public`, `protected`, or `private`; defaults to `public` if omitted. |
| `static` | Property belongs to the class, not instances. |
| `readonly` | PHP 8.1+; initialized once, never modified. |
| `type` | PHP 7.4+; any type except `void` and `callable`. |
| `$name` | The property variable. |
| `= constantExpression` | Optional; must be a compile-time constant. |

**Constraints and Limitations**

- **Initializer must be constant**: Cannot use function calls, `new`, or variables in default values.
- **Typed properties are uninitialized by default**: Accessing a typed property before assignment throws an `Error`.
- **`var` keyword is deprecated**: Use explicit visibility modifiers.

---

### 2. Property Declaration

#### Definitions

**Core Definition**
Property declaration is the act of writing a property inside a class body, specifying its modifiers, type, name, and optional default value.

**Technical Definition**
A declaration may appear directly in a class, in a trait used by the class, in a parent class, or (as of PHP 8.4) in an interface. The declaration may include a constant-expression initializer, but it cannot depend on runtime information.

**Beginner-Friendly Explanation**
Declaring a property is like listing an attribute in a blueprint: you give it a name, decide who can see it, and optionally say what type of data it holds and what value it starts with.

#### Purposes

- To define the data members of a class.
- To enforce type safety and visibility from the point of declaration.
- To provide default values for properties when objects are created.

#### Syntax Rules and Structure

**General Syntax**
```php
class MyClass
{
    public string $name = 'default';
    protected int $count = 0;
    private ?array $items = null;
    public static iterable $notes = [];
}
```

**Component Breakdown**

| Example | Explanation |
|---------|-------------|
| `public string $name = 'default';` | Public typed property with a literal default. |
| `protected int $count = 0;` | Protected integer property. |
| `private ?array $items = null;` | Private nullable array property. |
| `public static iterable $notes = [];` | Static typed property with an array default. |

**Constraints and Limitations**

- **Constant expression only**: `public $x = 1 + 2;` is allowed (scalar expression), but `public $x = self::compute();` is not.
- **One type applies to all properties in a declaration**: `public int $a, $b;` gives both `$a` and `$b` the type `int`.

#### Annotated Code Examples

**Example 1: Valid and Invalid Property Declarations**

```php
<?php
class PropertyDemo
{
    // Valid: constant expression
    public int $a = 1 + 2;

    // Valid: constant
    public string $b = PHP_EOL;

    // Valid: array literal
    public array $c = [true, false];

    // INVALID: function call is not a constant expression
    // public $d = self::compute();

    // INVALID: variable is not a constant expression
    // public $e = $someVar;
}
?>
```

**Expected Output:**
No output (definitions only). The commented lines would cause fatal errors if uncommented.

**Why This Output:**  
PHP evaluates property initializers at compile time. Only expressions that can be resolved without runtime information—literals, constants, and simple arithmetic on them—are permitted.

---

#### Real-World Cases

- **DTOs**: `public readonly string $email;` for immutable data transfer.
- **Entities**: `private int $id = 0;` with a default ID.
- **Configuration**: `protected array $options = [];` for default configuration.

---

### 3. Property Visibility

#### Definitions

**Core Definition**
Property visibility defines from where a property can be accessed: `public` (everywhere), `protected` (the class and its inheritance chain), or `private` (only the declaring class).

**Technical Definition**
Visibility is declared with the keywords `public`, `protected`, or `private`. `Public` members are accessible from any scope. `Protected` members are accessible within the declaring class and by inheriting/parent classes. `Private` members are accessible only by the class that defines them. Properties without an explicit visibility keyword default to `public`.

**Beginner-Friendly Explanation**
Visibility is like the locks on the boxes (properties). `Public` boxes are open to everyone. `Protected` boxes are open to you and your family (subclasses). `Private` boxes are locked in your own room—only you can open them.

#### Purposes

- To enforce encapsulation and prevent external code from corrupting internal state.
- To provide a clear public interface while hiding implementation details.
- To allow controlled extension through inheritance (`protected`).

#### Syntax Rules and Structure

**General Syntax**
```php
public $publicProp;
protected $protectedProp;
private $privateProp;
```

**Component Breakdown**

| Visibility | Accessible From |
|------------|-----------------|
| `public` | Everywhere. |
| `protected` | Declaring class and its inheritance chain (parents and children). |
| `private` | Only the declaring class. |

**Constraints and Limitations**

- **Private properties are not inherited**: A subclass cannot access a parent’s private property directly (though it can redeclare a property with the same name, creating a separate storage location).
- **`protected` is wider than `private`**: Use `private` by default; use `protected` only when subclasses genuinely need access.

#### Annotated Code Examples

**Example 1: Visibility in Action**

```php
<?php
class MyClass
{
    public $public = 'Public';
    protected $protected = 'Protected';
    private $private = 'Private';

    public function printHello(): void
    {
        echo $this->public . "\n";
        echo $this->protected . "\n";
        echo $this->private . "\n";
    }
}

$obj = new MyClass();
echo $obj->public . "\n";   // Works
// echo $obj->protected;    // Fatal Error
// echo $obj->private;      // Fatal Error
$obj->printHello();         // Outputs all three

class MyClass2 extends MyClass
{
    // Can redeclare public and protected, but not private
    public $public = 'Public2';
    protected $protected = 'Protected2';

    public function printHello(): void
    {
        echo $this->public . "\n";
        echo $this->protected . "\n";
        echo $this->private . "\n"; // Undefined — parent private not accessible
    }
}

$obj2 = new MyClass2();
$obj2->printHello();
?>
```

**Expected Output:**
```
Public
Protected
Private
Public2
Protected2
```
(The last line for `$this->private` in `MyClass2` would trigger an “Undefined property” warning, as the parent’s private property is not inherited.)

**Why This Output:**  
`public` is accessible from outside; `protected` and `private` are not. The subclass can redeclare `public` and `protected` properties but cannot access the parent’s `private` property.

---

**Example 2: Asymmetric Visibility (PHP 8.4+)**

```php
<?php
class Book
{
    public function __construct(
        public private(set) string $title,
        public protected(set) string $author,
        protected private(set) int $pubYear,
    ) {}
}

$b = new Book('How to PHP', 'Peter H. Peterson', 2024);
echo $b->title . "\n";    // Works (public get)
// $b->title = 'New';     // Fatal Error (private set)
echo $b->author . "\n";   // Works (public get)
// $b->author = 'New';    // Fatal Error (protected set)
// echo $b->pubYear;       // Fatal Error (protected get)
?>
```

**Expected Output:**
```
How to PHP
Peter H. Peterson
```

**Why This Output:**  
Asymmetric visibility allows reading a property publicly while restricting writing to a narrower scope. `title` is publicly readable but privately writable; `author` is publicly readable but only protected-writable; `pubYear` is protected for both read and write.

---

#### Real-World Cases

- **Entities with Controlled Mutation**: `public private(set) DateTimeImmutable $createdAt` allows reading the creation date but prevents external modification.
- **Service Configuration**: `protected array $options` is accessible to subclasses but not to external code.
- **Internal State**: `private string $hashedPassword` is hidden from all external code.

---

### 4. Property Types

#### Definitions

**Core Definition**
Property types declare the kind of value a property may hold, enabling PHP to enforce type safety at assignment time.

**Technical Definition**
Since PHP 7.4, property declarations may include a type declaration after the visibility and before the variable name. All types supported by PHP are allowed, with the exceptions of `void` and `callable`. Types apply to both instance and static properties. A typed property without a default value is in an **uninitialized** state; reading it before assignment throws an `Error`.

**Beginner-Friendly Explanation**
A property type is a label that says what kind of data the property can hold: `int`, `string`, `User`, `?array`, and so on. If you try to put the wrong kind of data in, PHP stops you with an error.

#### Purposes

- To catch type-related bugs at the point of assignment, rather than later when the value is used.
- To make class contracts self-documenting.
- To enable static analysis tools to infer property types and detect errors.

#### Syntax Rules and Structure

**General Syntax**
```php
[visibility] [static] Type $propertyName [= constantExpression];
```

**Component Breakdown**

| Type Category | Examples |
|---------------|----------|
| Scalar | `int`, `float`, `string`, `bool` |
| Compound | `array`, `object`, `iterable` |
| Class/Interface | `User`, `Countable` |
| Nullable | `?string` |
| Union (PHP 8.0+) | `int|string` |

**Constraints and Limitations**

- **`void` and `callable` are forbidden** in property type declarations.
- **Uninitialized typed properties**: Accessing before assignment throws `Error: Typed property must not be accessed before initialization`.
- **Strict types**: Without `declare(strict_types=1)`, scalar types may be coerced on assignment.

#### Annotated Code Examples

**Example 1: Typed Properties**

```php
<?php
declare(strict_types=1);

class User
{
    public int $id;
    public ?string $name;
    public static iterable $notes = [];

    public function __construct(int $id, ?string $name)
    {
        $this->id = $id;
        $this->name = $name;
    }
}

$user = new User(1234, null);
var_dump($user->id);    // int(1234)
var_dump($user->name);  // NULL
?>
```

**Expected Output:**
```
int(1234)
NULL
```

**Why This Output:**  
`$id` is typed `int` and receives `1234`; `$name` is nullable (`?string`) and receives `null`. The static property `$notes` is typed `iterable` and defaults to an empty array.

---

**Example 2: Type Error on Invalid Assignment**

```php
<?php
declare(strict_types=1);

class Shape
{
    public int $numberOfSides;
}

$shape = new Shape();
$shape->numberOfSides = "four"; // TypeError
?>
```

**Expected Output:**
```
Fatal error: Uncaught TypeError: Cannot assign string to property Shape::$numberOfSides of type int
```

**Why This Output:**  
With strict types enabled, assigning a `string` to an `int`-typed property throws a `TypeError` at the assignment boundary.

---

#### Real-World Cases

- **Domain Models**: `public readonly Money $price;` ensures the price is always a `Money` object.
- **Collections**: `private array $items = [];` guarantees the property is always an array.
- **Nullable Relationships**: `private ?User $owner = null;` allows an optional owner.

---

### 5. Typed Properties

#### Definitions

**Core Definition**
Typed properties are properties with an explicit type declaration, enforced at runtime when values are assigned.

**Technical Definition**
Introduced in PHP 7.4, typed properties support all PHP types except `void` and `callable`. They may have default values, provided the default matches the declared type (or is `null` for nullable types). A typed property without a default is uninitialized; reading it before assignment throws an `Error`. Types apply to static properties as well.

**Beginner-Friendly Explanation**
A typed property is a property that says “I only accept integers” or “I only accept User objects.” If you try to put something else in, PHP will refuse.

#### Purposes

- To enforce data integrity at the property level.
- To eliminate ambiguity about what type of value a property holds.
- To improve IDE autocompletion and static analysis.

#### Syntax Rules and Structure

**General Syntax**
```php
public int $age;
protected ?User $owner = null;
public static iterable $tastingNotes;
```

**Component Breakdown**

| Rule | Description |
|------|-------------|
| Type before variable | `int $age`, not `$age int`. |
| Default must match | `public int $rating = 92;` is valid; `public int $rating = null;` is not (unless nullable). |
| Static typed | `public static iterable $notes;` is allowed. |
| Uninitialized state | Reading before assignment throws `Error`. |

**Constraints and Limitations**

- **No `callable` type**: Use `Closure` if you need a typed callable property.
- **No implicit null**: `public int $x;` cannot be set to `null`; use `?int` for nullable.
- **Coercion without strict types**: In non-strict mode, scalar types may be coerced (e.g., `"5"` to `5`).

#### Annotated Code Examples

**Example 1: Typed Property with Default Value**

```php
<?php
class Whisky
{
    public int $rating = 92;
    public ?string $tastingNotes = null;
}

$w = new Whisky();
echo $w->rating . "\n";       // Outputs: 92
var_dump($w->tastingNotes);   // Outputs: NULL
?>
```

**Expected Output:**
```
92
NULL
```

**Why This Output:**  
The typed property `$rating` has a default of `92`; `$tastingNotes` is nullable and defaults to `null`. Both are valid because the defaults match the declared types.

---

#### Real-World Cases

- **Configuration Objects**: `public int $timeout = 30;`
- **Value Objects**: `public readonly string $currency;`
- **Counters**: `public int $retryCount = 0;`

---

### 6. Readonly Properties (PHP 8.1+)

#### Definitions

**Core Definition**
A `readonly` property is a typed property that can be initialized exactly once and cannot be modified thereafter.

**Technical Definition**
Introduced in PHP 8.1, `readonly` properties must have an explicit type and must not have a default value. They can be written only once, typically from within the declaring class scope (most commonly the constructor). Before PHP 8.4, `readonly` properties were implicitly `private(set)`; from PHP 8.4 onward, they can be combined with asymmetric visibility. A `readonly` property cannot be `static`. A `readonly` class (PHP 8.2+) applies the constraint to every instance property.

**Beginner-Friendly Explanation**
A `readonly` property is like a value written in permanent ink: once you write it, you cannot erase or change it. This is perfect for values that should never change after an object is created, such as a user’s ID or a creation date.

#### Purposes

- To enforce immutability of object state after construction.
- To make value objects safe and predictable.
- To eliminate accidental reassignment bugs.

#### Syntax Rules and Structure

**General Syntax**
```php
public readonly Type $propertyName;
```

**Component Breakdown**

| Rule | Description |
|------|-------------|
| Must be typed | `readonly` requires an explicit type. |
| No default value | `readonly` properties cannot have a default initializer. |
| Write once | Initialization must occur inside the declaring class (typically the constructor). |
| Not static | `readonly` cannot be applied to static properties. |

**Constraints and Limitations**

- **Cannot be modified after initialization**: Any second assignment throws an `Error`.
- **Inheritance**: A `readonly` property cannot be redeclared in a subclass if it has already been initialized in the parent.
- **Property hooks incompatible**: `readonly` and property hooks cannot be combined on the same property.

#### Annotated Code Examples

**Example 1: Readonly Property Initialized in Constructor**

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public readonly int $id,
        public readonly string $email
    ) {}
}

$user = new User(1, 'alice@example.com');
echo $user->id . "\n";    // Outputs: 1
echo $user->email . "\n"; // Outputs: alice@example.com

// $user->id = 2; // Error: Cannot modify readonly property
?>
```

**Expected Output:**
```
1
alice@example.com
```

**Why This Output:**  
The constructor promotes `$id` and `$email` to `readonly` properties, initializing them exactly once. Attempting to modify them afterward throws an `Error`.

---

**Example 2: Readonly Class (PHP 8.2+)**

```php
<?php
readonly class Money
{
    public function __construct(
        public int $amount,
        public string $currency
    ) {}
}

$price = new Money(1000, 'USD');
echo $price->amount . ' ' . $price->currency . "\n"; // Outputs: 1000 USD
// $price->amount = 2000; // Error
?>
```

**Expected Output:**
```
1000 USD
```

**Why This Output:**  
The `readonly` class modifier makes every declared property `readonly` automatically, eliminating the need to repeat the keyword on each property.

---

#### Real-World Cases

- **Value Objects**: `Money`, `Email`, `UserId`—immutable by design.
- **DTOs**: Request/response data objects that should not change after construction.
- **Configuration Snapshots**: A configuration object whose values are fixed once loaded.

---

### 7. Static Properties

#### Definitions

**Core Definition**
A `static` property belongs to the class itself rather than to any individual instance. It is shared by all objects of the class.

**Technical Definition**
Static properties are declared with the `static` keyword and are accessed using the scope resolution operator (`::`), not the object operator (`->`). They can be typed (PHP 7.4+) and can have constant-expression defaults. Unlike instance properties, static properties are initialized once per class, not once per object. In inheritance, static properties declared in both parent and child classes hold **distinct values** for each class.

**Beginner-Friendly Explanation**
A static property is a value stored on the blueprint (class) itself, not on each object made from the blueprint. All objects share the same static property. It is useful for counters, registries, or configuration that applies to the whole class.

#### Purposes

- To share data across all instances of a class.
- To implement counters, singletons, or class-level registries.
- To avoid duplicating data that is identical for every object.

#### Syntax Rules and Structure

**General Syntax**
```php
[visibility] static [type] $propertyName [= constantExpression];
```

**Component Breakdown**

| Rule | Description |
|------|-------------|
| `static` keyword | Makes the property belong to the class. |
| Access | `self::$property` or `ClassName::$property`; **not** `$object->property`. |
| Initialization | Must be a constant expression (literal or constant). |
| Inheritance | Parent and child classes have separate storage for same-named static properties. |

**Constraints and Limitations**

- **Cannot be accessed via `->`**: Using `$object->staticProperty` triggers a notice/warning and does not access the static value.
- **Shared mutable state**: Modifying a static property in one part of the code affects all code that reads it.
- **Static readonly (PHP 8.3+)**: Static properties can be `readonly` as of PHP 8.3.

#### Annotated Code Examples

**Example 1: Static Counter**

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
`$count` is static and shared by all instances. Each `new Counter()` increments the same static property, so the final count is `3`.

---

**Example 2: Parent and Child Static Properties Are Distinct**

```php
<?php
class ParentClass
{
    public static string $name = 'Parent';
}

class ChildClass extends ParentClass
{
    public static string $name = 'Child';
}

echo ParentClass::$name . "\n"; // Outputs: Parent
echo ChildClass::$name . "\n";  // Outputs: Child
?>
```

**Expected Output:**
```
Parent
Child
```

**Why This Output:**  
Static properties declared in both parent and child classes hold separate values. Redeclaring a static property in a subclass creates a distinct storage location for that subclass.

---

#### Real-World Cases

- **Instance Counters**: Tracking how many objects of a class have been created.
- **Global Configuration**: `public static array $config = [];`
- **Singleton Instances**: `private static ?self $instance = null;`

---

### 8. Property Hooks (PHP 8.4+)

#### Definitions

**Core Definition**
Property hooks (also called property accessors) are a PHP 8.4 feature that allows custom logic to be executed when a property is read (`get`) or written (`set`), directly at the property declaration.

**Technical Definition**
Property hooks intercept and override the read and write behaviour of a property. Two hooks are available: `get` and `set`. A property may be **backed** (stores a value) or **virtual** (computed, no stored value). The `set` hook may specify the type and name of the incoming value. Hooks are available for typed and untyped properties, but **cannot be combined with `readonly`**. Property hooks were introduced in PHP 8.4.

**Beginner-Friendly Explanation**
Property hooks let you attach a small piece of logic directly to a property. For example, you can automatically lowercase an email address whenever it is set, or compute a full name from first and last names whenever it is read—without writing separate getter and setter methods.

#### Purposes

- To eliminate boilerplate getter/setter methods.
- To co-locate access logic with the property declaration.
- To support computed (virtual) properties without stored backing data.

#### Syntax Rules and Structure

**General Syntax**
```php
public string $property {
    get { return $this->property; }
    set(string $value) { $this->property = strtolower($value); }
}
```

**Component Breakdown**

| Hook | Description |
|------|-------------|
| `get` | Executed when the property is read. May be a single expression (`get => expr;`). |
| `set` | Executed when the property is written; receives `$value`. |
| Virtual property | Hooks do not reference the property itself; no storage is allocated. |

**Constraints and Limitations**

- **Cannot be used with `readonly` properties**.
- **No side effects**: Hooks should be pure; do not perform I/O, database queries, or logging inside hooks.
- **Type compatibility**: The `set` hook parameter type must be the same as or wider (contravariant) than the property type.

#### Annotated Code Examples

**Example 1: Normalising a Property with a `set` Hook**

```php
<?php
declare(strict_types=1);

final class User
{
    public string $email {
        set(string $value) {
            $this->email = strtolower(trim($value));
        }
    }

    public function __construct(
        public readonly string $firstName,
        public readonly string $lastName,
    ) {}
}

$user = new User('Alice', 'Smith');
$user->email = ' ALICE@EXAMPLE.COM ';
echo $user->email . "\n"; // Outputs: alice@example.com
?>
```

**Expected Output:**
```
alice@example.com
```

**Why This Output:**  
The `set` hook normalises the email by trimming whitespace and converting it to lowercase. The `get` hook is implicit and returns the stored value.

---

**Example 2: Virtual Computed Property**

```php
<?php
declare(strict_types=1);

final class User
{
    public string $fullName {
        get => "{$this->firstName} {$this->lastName}";
    }

    public function __construct(
        public readonly string $firstName,
        public readonly string $lastName,
    ) {}
}

$user = new User('Alice', 'Smith');
echo $user->fullName . "\n"; // Outputs: Alice Smith
?>
```

**Expected Output:**
```
Alice Smith
```

**Why This Output:**  
`$fullName` is a virtual property: its `get` hook computes the value from `$firstName` and `$lastName` without storing anything. Accessing it triggers the computation.

---

#### Real-World Cases

- **Email Normalisation**: Automatic lowercasing and trimming on assignment.
- **Computed Properties**: `fullName`, `area`, `total`—derived values without stored duplication.
- **Validation on Write**: Rejecting invalid values directly in the `set` hook.

---

### 9. Property Best Practices

#### Definitions

**Core Definition**
Property best practices are guidelines for declaring and using properties in a way that maximizes safety, clarity, and maintainability.

**Technical Definition**
Best practices include: **defaulting to `private`** for properties and exposing controlled access via getters, setters, or property hooks; **using typed properties** everywhere; **preferring `readonly`** for value objects and DTOs; **declaring constant-expression defaults** only; **using asymmetric visibility (PHP 8.4+)** when read and write scopes should differ; **keeping property hooks pure** (no side effects); and **documenting property types** with PHPDoc for generic shapes.

**Beginner-Friendly Explanation**
Best practices for properties are simple rules: keep your properties private by default, give them types so PHP can catch mistakes, make things `readonly` when they should never change, and don’t hide complex logic or side effects inside property access. Use the right tool for the job—`private` for internal state, `readonly` for immutable values, and property hooks for small, pure transformations.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Default to `private`** | Expose properties only when necessary; use getters/setters or hooks for controlled access. |
| **Always declare types** | Typed properties catch errors early and improve static analysis. |
| **Use `readonly` for value objects** | `Money`, `Email`, DTOs, and other immutable data should be `readonly`. |
| **Use `readonly` classes when all fields are immutable** | Less noisy than repeating `readonly` on every property. |
| **Prefer property hooks over boilerplate** | Use PHP 8.4 hooks for pure normalisation, formatting, and computed values. |
| **Keep hooks pure** | No database queries, HTTP requests, logging, or other side effects inside `get`/`set`. |
| **Use asymmetric visibility (PHP 8.4+) for read-public/write-private** | Avoid unnecessary getters when only write access needs restriction. |
| **Validate in the constructor** | Ensure an object can never exist in an invalid state. |

**Constraints and Limitations**

- **Strict types are per-file**: Type coercion for properties follows the strictness of the file where the assignment occurs.
- **Static properties are global mutable state**: Use them sparingly; they make testing and reasoning harder.
- **`readonly` prevents mutation**: Do not use `readonly` on properties that legitimately need to change after construction (e.g., status fields in entities).

#### Annotated Code Examples

**Example 1: A Well-Designed Class Using Property Best Practices**

```php
<?php
declare(strict_types=1);

final class Money
{
    public function __construct(
        public readonly int $amountInCents,
        public readonly string $currency
    ) {
        if ($amountInCents < 0) {
            throw new InvalidArgumentException('Amount cannot be negative.');
        }
        if (strlen($currency) !== 3) {
            throw new InvalidArgumentException('Currency must be a 3-letter code.');
        }
    }
}

$price = new Money(1999, 'USD');
echo $price->amountInCents . ' ' . $price->currency . "\n"; // Outputs: 1999 USD
?>
```

**Expected Output:**
```
1999 USD
```

**Why This Output:**  
This class follows best practices: `final` (prevents uncontrolled inheritance), `declare(strict_types=1)`, `readonly` properties for immutability, constructor validation to enforce invariants, and clear property names. It is a value object that is safe to pass around and share.

---

**Example 2: Default to `private` with Controlled Access**

```php
<?php
declare(strict_types=1);

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
echo $account->getBalance() . "\n"; // Outputs: 100
?>
```

**Expected Output:**
```
100
```

**Why This Output:**  
`$balance` is `private`, so external code cannot directly modify it. The `deposit()` method validates the amount before changing the balance, enforcing the invariant that the balance never becomes negative.

---

#### Real-World Cases

- **Domain-Driven Design**: Entities use `private` properties with public methods; value objects use `readonly`.
- **Laravel/Symfony Codebases**: Modern PHP frameworks encourage typed properties and `readonly` for DTOs.
- **Static Analysis in CI**: PHPStan rules (e.g., `phpstan-readonly-enforcing`) automatically suggest `readonly` where safe.

---

## References

**PHP Manual – Visibility** - https://static.php.net/manual/en/language.oop5.visibility.php

**PHP Manual – Properties** - https://static.php.net/language.oop5.properties

**PHP Manual – Static Keyword** - https://static.php.net/manual/en/language.oop5.static.php

**PHP Manual – Property Hooks (PHP 8.4)** - https://static.php.net/manual/en/language.oop5.property-hooks.php

**PHP Manual – Asymmetric Visibility (PHP 8.4)** - https://static.php.net/manual/en/language.oop5.visibility.php#language.oop5.visibility-members-aviz

**PHP Manual – Readonly Properties (PHP 8.1)** - https://static.php.net/manual/en/language.oop5.properties.php#language.oop5.properties.readonly-properties

**PHP RFC: Typed Properties 2.0** - https://wiki.php.net/rfc/typed_properties_v2

**PHP RFC: Readonly Properties 2.0** - https://wiki.php.net/rfc/readonly_properties_v2

**PSR-12: Extended Coding Style** - https://www.php-fig.org/psr/psr-12/

**PHP Best Practices – Property Hooks (Agent Skills)** - https://github.com/voku/agent-skills/blob/main/skills/php-best-practices/rules/modern-property-hooks.md

**Secure PHP Development – Readonly Properties** - https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Modern-PHP/Readonly-Properties.md