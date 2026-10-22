This cheat sheet provides a research-backed overview of PHP's `readonly` feature, covering its mechanics and best practices for creating immutable objects.

---

## Topic Overview

### Definitions

**Core Definition**
`readonly` is a PHP language modifier that enforces write-once semantics for object properties. Once a readonly property is initialized, any attempt to modify it results in a runtime error, turning immutability from a convention into a language-enforced guarantee.

**Technical Definition**
The `readonly` modifier, introduced for properties in PHP 8.1.0 and extended to entire classes in PHP 8.2.0, prevents modification of a property after its initial assignment. A readonly property may be initialized only once and only from the scope in which it is declared—typically the constructor. Any subsequent assignment, increment, array append, or reference binding throws an `Error`. Readonly properties must have an explicit type declaration; untyped readonly properties are forbidden because the implicit `null` default would count as an initialization. Before PHP 8.4.0, readonly properties were implicitly "private-set," meaning writes were only permitted from within the declaring class.

**Beginner-Friendly Explanation**
Think of a `readonly` property as a value written in permanent ink. You can write it once—usually when the object is created—but after that, the ink dries and no one, not even the object itself, can change it. If you try to overwrite it, PHP throws an error. This is useful for value objects like money amounts, coordinates, or user IDs, where accidental modification could cause bugs.

---

### Key Characteristics

- **Write-Once Semantics**: A readonly property accepts exactly one initialization; every subsequent write fails.
- **Scope-Restricted Initialization**: The first write must occur from within the scope of the class that declares the property (usually the constructor).
- **Explicit Type Required**: Readonly properties must be typed; untyped readonly properties are a fatal error.
- **No Default Values**: Readonly properties cannot have default values, as the default would count as an initialization.
- **Shallow Immutability**: Readonly prevents reassignment of the property itself, but objects stored in readonly properties may still be mutated internally (interior mutability).
- **Class-Level Shorthand (PHP 8.2+)**: Declaring a class as `readonly` implicitly marks every instance property as readonly and forbids dynamic properties.
- **Static Properties Forbidden**: Readonly classes and individual readonly properties cannot be static.

---

### Prerequisites

- **PHP 8.1.0 or later** for readonly properties; **PHP 8.2.0 or later** for readonly classes.
- **Basic Object-Oriented PHP**: Understanding of classes, constructors, and property declaration.
- **Constructor Property Promotion (Recommended)**: Readonly works seamlessly with PHP 8.0's promoted constructor parameters for concise value objects.

---

### Related Programming Areas

- **Value Objects and DTOs**: Readonly is the natural enforcer of immutability for objects whose identity is defined by their values.
- **Domain-Driven Design (DDD)**: Readonly properties model immutable domain concepts (Money, Email, DateRange).
- **Functional Programming**: Immutable objects align with functional principles of avoiding side effects.
- **Security**: Readonly properties lock validated values (e.g., user IDs, signed tokens) so they cannot be silently overwritten after a security check.

---

## Core Concepts / Features

---

### 1. `readonly` Modifier

#### Definitions

**Core Definition**
The `readonly` modifier is a keyword placed before a property declaration (or before a class declaration in PHP 8.2+) that enforces write-once immutability.

**Technical Definition**
`readonly` is a property modifier that restricts writes to the declaring class's scope and permits only one initializing assignment. For classes, `readonly class` is shorthand that applies the `readonly` modifier to every declared instance property and prohibits dynamic properties.

**Beginner-Friendly Explanation**
The `readonly` keyword is like a "do not edit" stamp you place on a property. Once the value is written, the stamp prevents anyone—including the object's own methods—from changing it.

#### Purposes

- To enforce immutability at the language level, preventing accidental reassignment deep in a call stack.
- To eliminate boilerplate getters and manual guard clauses previously needed to simulate immutability.
- To make value objects tamper-proof and safe to share, cache, and pass between layers.
- To lock validated security-sensitive values (e.g., a resolved user ID or file path) after verification.

#### Syntax Rules and Structure

**General Syntax (Property)**
```php
class ClassName {
    public readonly Type $property;
}
```

**General Syntax (Promoted Constructor Property)**
```php
class ClassName {
    public function __construct(
        public readonly Type $property
    ) {}
}
```

**General Syntax (Class)**
```php
readonly class ClassName {
    public function __construct(
        public Type $property
    ) {}
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `readonly` | The modifier; may appear before `class` or before the property declaration. |
| `Type` | An explicit type declaration (required for readonly properties). |
| `$property` | The property variable. |
| Scope of write | The first assignment must occur within the declaring class's scope (typically the constructor). |

**Syntax Rules**

- A readonly property must have an explicit type; `readonly $x` without a type is a fatal error.
- A readonly property must **not** have a default value; the default would count as an initialization.
- The initializing assignment may occur only once and only from the declaring class's scope.
- `readonly` may be combined with visibility modifiers (`public`, `protected`, `private`) and with `static`? **No**—readonly properties cannot be static.

**Constraints and Limitations**

- **No static readonly**: Static properties cannot be readonly.
- **No default values**: Readonly properties cannot declare default values.
- **No untyped readonly**: The type declaration is mandatory.
- **Shallow only**: Readonly prevents reassignment of the property, but the object it points to may still be mutated (interior mutability).
- **No reference binding**: You cannot bind a reference to a readonly property or pass it by reference.

#### Annotated Code Examples

**Example 1: Readonly Property with Constructor Initialization**

```php
<?php
declare(strict_types=1);

class User {
    // Step 1: Declare a typed readonly property
    public readonly string $name;

    public function __construct(string $name) {
        // Step 2: The one and only legal write
        $this->name = $name;
    }
}

$user = new User("Alice");
echo $user->name . "\n"; // Outputs: Alice

// Step 3: Any reassignment throws an Error
try {
    $user->name = "Bob";
} catch (Error $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
// Outputs: Error: Cannot modify readonly property User::$name
?>
```

**Expected Output:**
```
Alice
Error: Cannot modify readonly property User::$name
```

**Why This Output:**
The property `$name` is readonly and typed as `string`. It is initialized once in the constructor. The subsequent assignment outside the class throws an `Error` because readonly properties cannot be modified after initialization.

**Step-by-Step Setup:**
1.  Save as `readonly_user.php`.
2.  Run `php readonly_user.php`.
3.  The output confirms the successful read and the failed write.

---

**Example 2: Promoted Readonly Property**

```php
<?php
declare(strict_types=1);

class Point {
    // Step 1: Promote and mark readonly in the constructor signature
    public function __construct(
        public readonly float $x,
        public readonly float $y,
    ) {}
}

$p = new Point(3.5, 2.0);
echo "X: {$p->x}, Y: {$p->y}\n"; // Outputs: X: 3.5, Y: 2.0
?>
```

**Expected Output:**
```
X: 3.5, Y: 2.0
```

**Why This Output:**
Constructor property promotion declares and initializes the properties in one step. The `readonly` modifier is applied to each promoted property, making them immutable after construction.

#### Real-World Cases

- **Value Objects**: `readonly class Money { public function __construct(public int $amount, public string $currency) {} }` ensures a monetary value cannot be silently altered after creation.
- **DTOs (Data Transfer Objects)**: Request/response objects carrying user input are made readonly so downstream code cannot mutate validated data.
- **Security Tokens**: A readonly `Token` object locks a validated token string after signature verification, preventing later code from overwriting it with an unsigned value.

---

### 2. Readonly Properties

#### Definitions

**Core Definition**
A readonly property is an instance property that accepts exactly one initialization and cannot be modified thereafter.

**Technical Definition**
A readonly property is declared with the `readonly` modifier, must have an explicit type, and may be initialized only once from within the scope of the declaring class. The initializing assignment is typically performed in the constructor. Any subsequent write—including increment, decrement, array append, or reference binding—throws an `Error`. Readonly properties are not implicitly `final`; a child class may declare its own readonly properties, but it cannot redeclare a parent's readonly property.

**Beginner-Friendly Explanation**
A readonly property is like a sealed envelope: once you put the letter inside and seal it, you can read the letter but you cannot replace it. The sealing happens in the constructor, and after that the value is fixed.

#### Purposes

- To create immutable properties without hand-written getters and guard clauses.
- To expose public properties safely without fear of external mutation breaking class invariants.
- To lock security-critical values after validation.
- To signal to readers and static analysis tools that a property is write-once.

#### Syntax Rules and Structure

**General Syntax**
```php
class ClassName {
    public readonly Type $property;

    public function __construct(Type $value) {
        $this->property = $value; // legal initialization
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `readonly` | The modifier. |
| `Type` | Required explicit type. |
| `$property` | The property name. |
| Initialization | Must occur inside the declaring class, typically in the constructor. |

**Syntax Rules**

- The initializing assignment must be the **first** write to the property.
- The write must occur from within the declaring class's scope; external assignment fails.
- The property cannot have a default value.
- The property cannot be static.

**Constraints and Limitations**

- **No default values**: Defaults are treated as initializations and are therefore forbidden.
- **No untyped properties**: An explicit type is mandatory.
- **No reference binding**: `$ref =& $this->readonlyProperty` is an error.
- **No modification via foreach**: `foreach ($obj as &$prop)` fails if `$prop` is readonly.
- **Interior mutability allowed**: Objects inside readonly properties can still be mutated internally.

#### Annotated Code Examples

**Example 1: Readonly Property with Validation**

```php
<?php
declare(strict_types=1);

final class Email {
    public readonly string $address;

    public function __construct(string $address) {
        // Step 1: Validate before the one legal write
        if (filter_var($address, FILTER_VALIDATE_EMAIL) === false) {
            throw new InvalidArgumentException("Invalid email: $address");
        }
        // Step 2: The single initialization
        $this->address = $address;
    }
}

$email = new Email("alice@example.com");
echo $email->address . "\n"; // Outputs: alice@example.com

// Step 3: Any write attempt fails
try {
    $email->address = "attacker@evil.com";
} catch (Error $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
// Outputs: Error: Cannot modify readonly property Email::$address
?>
```

**Expected Output:**
```
alice@example.com
Error: Cannot modify readonly property Email::$address
```

**Why This Output:**
Validation occurs before the initialization. Once the valid address is assigned, the readonly property is frozen. The attempted reassignment throws an `Error`, demonstrating that the validated value cannot be overwritten.

---

**Example 2: Interior Mutability Despite Readonly**

```php
<?php
declare(strict_types=1);

class Container {
    public function __construct(
        public readonly stdClass $data
    ) {}
}

$container = new Container(new stdClass());
$container->data->foo = "bar"; // Legal: interior mutation
echo $container->data->foo . "\n"; // Outputs: bar

// $container->data = new stdClass(); // Illegal reassignment
?>
```

**Expected Output:**
```
bar
```

**Why This Output:**
The readonly property `$data` holds a reference to a mutable `stdClass` object. Readonly prevents reassigning `$data` to a different object, but the object it points to can still be modified internally. This is "shallow immutability".

#### Real-World Cases

- **User IDs in Security Contexts**: After authenticating a user, the resolved `userId` is stored in a readonly property so subsequent code cannot swap it for another ID.
- **Configuration Values**: A configuration object with readonly properties (e.g., `databaseHost`, `apiKey`) ensures settings are not accidentally altered at runtime.
- **Date Ranges**: A `DateRange` value object with readonly `start` and `end` properties guarantees the range cannot be narrowed or expanded after construction.

---

### 3. Readonly Classes

#### Definitions

**Core Definition**
A readonly class (PHP 8.2+) is a class declared with the `readonly` modifier, which implicitly makes every instance property readonly and forbids dynamic properties.

**Technical Definition**
`readonly class ClassName` applies the `readonly` modifier to every declared instance property, whether declared explicitly or via constructor promotion. The engine enforces that all properties are typed, that no static properties exist, and that no dynamic properties can be created. A readonly class may be extended only by another readonly class; a non-readonly class cannot extend a readonly one. As of PHP 8.3, a readonly property may be reinitialized during `__clone()`, enabling ergonomic "copy-with-changes" patterns.

**Beginner-Friendly Explanation**
A readonly class is like a sealed box: everything inside is automatically locked. Instead of putting a "do not edit" stamp on each item individually, you seal the entire box with one keyword. This is less error-prone because you cannot forget to mark a new property.

#### Purposes

- To make immutability structural rather than a per-property discipline, eliminating the risk of forgetting `readonly` on a newly added property.
- To produce concise, self-documenting value objects and DTOs with minimal boilerplate.
- To forbid dynamic properties, ensuring the object's shape is fixed at compile time.
- To support "wither" methods that return new instances with one field changed, using the clone-reinitialization capability of PHP 8.3.

#### Syntax Rules and Structure

**General Syntax**
```php
readonly class ClassName {
    public function __construct(
        public Type $property,
    ) {}
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `readonly` | The modifier placed before `class`. |
| Class body | Every instance property is implicitly readonly; individual `readonly` modifiers are redundant (and may trigger static-analysis warnings). |
| Properties | Must be typed; static properties are forbidden. |

**Syntax Rules**

- All instance properties in a readonly class are implicitly readonly; you cannot mark one as writable.
- All properties must be typed; an untyped property is a fatal error.
- Static properties are not allowed in a readonly class.
- Dynamic properties are forbidden; the class behaves as if `#[\AllowDynamicProperties]` is permanently disabled.
- A readonly class may be extended only by another readonly class (prior to PHP 8.3; see the readonly amendments RFC for proposed relaxation).

**Constraints and Limitations**

- **No static properties**: Readonly classes cannot contain static properties.
- **No untyped properties**: Every property must have an explicit type.
- **No dynamic properties**: Dynamic property creation is forbidden.
- **Inheritance restriction**: A non-readonly class cannot extend a readonly class (this restriction was relaxed in PHP 8.3 per the readonly amendments RFC).
- **ORMs may conflict**: Doctrine and Eloquent entities typically require mutable properties and are incompatible with readonly classes.

#### Annotated Code Examples

**Example 1: Readonly Class for a Money Value Object**

```php
<?php
declare(strict_types=1);

readonly class Money {
    public function __construct(
        public int $amountCents,
        public string $currency,
    ) {}
}

$price = new Money(1999, "USD");
echo "{$price->amountCents} {$price->currency}\n"; // Outputs: 1999 USD

// Any property write fails
try {
    $price->amountCents = 2999;
} catch (Error $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
?>
```

**Expected Output:**
```
1999 USD
Error: Cannot modify readonly property Money::$amountCents
```

**Why This Output:**
The `readonly` modifier on the class makes both promoted properties implicitly readonly. The initialization succeeds, but the subsequent write throws an `Error` because the property is frozen after construction.

---

**Example 2: "Wither" Method with Readonly Class**

```php
<?php
declare(strict_types=1);

readonly class Colour {
    public function __construct(
        public int $red,
        public int $green,
        public int $blue,
    ) {}

    // Step 1: Return a new instance with one field changed
    public function withBlue(int $blue): self {
        return new self($this->red, $this->green, $blue);
    }
}

$red = new Colour(255, 0, 0);
$blue = $red->withBlue(255);

echo "Red: ({$red->red}, {$red->green}, {$red->blue})\n";
// Outputs: Red: (255, 0, 0)
echo "Blue: ({$blue->red}, {$blue->green}, {$blue->blue})\n";
// Outputs: Blue: (255, 0, 255)
?>
```

**Expected Output:**
```
Red: (255, 0, 0)
Blue: (255, 0, 255)
```

**Why This Output:**
The `withBlue()` method returns a **new** `Colour` instance with the blue channel changed. The original `$red` object remains unchanged because it is immutable. This "copy-with-changes" pattern is the idiomatic way to "modify" readonly objects.

#### Real-World Cases

- **DTOs (Data Transfer Objects)**: `readonly class CreateUserRequest { public function __construct(public string $email, public string $name) {} }` ensures request data cannot be tampered with after validation.
- **Domain Events**: `readonly class UserCreated { public function __construct(public int $userId, public \DateTimeImmutable $occurredAt) {} }` guarantees that recorded events are immutable facts.
- **Geometry and Coordinates**: `readonly class Coordinate { public function __construct(public float $lat, public float $lng) {} }` prevents accidental mutation of geographic positions.

---

### 4. Readonly Best Practices

#### Definitions

**Core Definition**
Readonly best practices are community-established guidelines for when and how to use `readonly` properties and classes to maximize safety, clarity, and maintainability.

**Technical Definition**
Best practices recommend using `readonly` for pure value objects, DTOs, events, and commands; preferring `readonly class` over per-property `readonly` when the entire object is meant to be immutable; performing all validation in the constructor; and avoiding `readonly` for ORM entities or classes that require post-construction mutation. They also include combining `readonly` with `final` for value objects and using "wither" methods (returning new instances) instead of mutating existing ones.

**Beginner-Friendly Explanation**
Best practices answer the question "When should I use `readonly`?" The short answer is: use it for objects that represent a fixed value—like money, an email address, or a set of coordinates. Do not use it for objects that need to change over time, like a shopping cart or a user profile being edited.

#### Purposes

- To ensure immutability is applied consistently and structurally, not ad hoc.
- To prevent silent bugs caused by forgetting `readonly` on a newly added property.
- To make value objects safe to share, cache, and pass across layers without defensive copying.
- To align with static analysis tools (PHPStan, Psalm) that treat readonly class properties as write-once and flag violations.

#### Guidelines Summary

| Practice | Description |
|----------|-------------|
| **Prefer `readonly class` for pure value objects** | If every property is meant to be immutable, use the class-level modifier to avoid forgetting `readonly` on new properties. |
| **Combine with `final` for value objects** | `final readonly class` prevents extension and communicates that the value type is complete. |
| **Validate in the constructor** | Because properties are frozen after construction, all validation must occur before or during initialization. |
| **Use constructor promotion** | `public function __construct(public readonly int $x)` keeps value objects concise and self-documenting. |
| **Use "wither" methods for changes** | Return a new instance with the changed field instead of attempting mutation. |
| **Do not use readonly for ORM entities** | Doctrine and Eloquent entities require mutable properties for change tracking and hydration. |
| **Do not use readonly on abstract classes meant for extension** | Abstract classes intended for developer extension should generally not be readonly, as this restricts child classes. |

**Syntax Rules**

- Place the `readonly` modifier before `class` for class-level immutability.
- Do not redundantly add `readonly` to individual properties inside a readonly class; it is implicit and may trigger static-analysis warnings.
- Ensure all properties in a readonly class are typed.

**Constraints and Limitations**

- **ORMs**: Doctrine and Eloquent rely on mutable properties; readonly classes are incompatible.
- **Inheritance**: A readonly class can only be extended by another readonly class (restriction relaxed in PHP 8.3 per the readonly amendments RFC).
- **Shallow immutability**: Readonly does not make nested objects immutable; interior mutability remains possible.

#### Annotated Code Examples

**Example 1: Best-Practice Value Object**

```php
<?php
declare(strict_types=1);

final readonly class EmailAddress {
    public function __construct(
        public string $local,
        public string $domain,
    ) {
        // Step 1: Validate in the constructor
        if (filter_var("{$local}@{$domain}", FILTER_VALIDATE_EMAIL) === false) {
            throw new InvalidArgumentException("Invalid email: {$local}@{$domain}");
        }
    }

    // Step 2: Derived accessor (no mutation)
    public function full(): string {
        return "{$this->local}@{$this->domain}";
    }

    // Step 3: Wither method for "changes"
    public function withDomain(string $domain): self {
        return new self($this->local, $domain);
    }
}

$email = new EmailAddress("alice", "example.com");
echo $email->full() . "\n"; // Outputs: alice@example.com

$newEmail = $email->withDomain("example.org");
echo $newEmail->full() . "\n"; // Outputs: alice@example.org
echo $email->full() . "\n";   // Outputs: alice@example.com (unchanged)
?>
```

**Expected Output:**
```
alice@example.com
alice@example.org
alice@example.com
```

**Why This Output:**
The class is `final readonly`, so both promoted properties are immutable. Validation occurs in the constructor. The `withDomain()` method returns a **new** `EmailAddress` instance, leaving the original unchanged. This follows best practices for immutable value objects.

#### Real-World Cases

- **TYPO3 Core**: The TYPO3 core team recommends declaring services and their properties as readonly where possible, while noting that abstract classes intended for extension should not be readonly.
- **Laravel/Symfony DTOs**: Framework applications increasingly use `final readonly class` for request/response DTOs, form data objects, and command messages.
- **Static Analysis Integration**: PHPStan and Psalm recognize readonly classes and flag any post-construction write as an error, making readonly a first-class citizen in static analysis workflows.

---

## References

**PHP RFC: Readonly properties 2.0** - https://wiki.php.net/rfc/readonly_properties_v2

**PHP RFC: Readonly classes** - https://wiki.php.net/rfc/readonly_classes

**PHP RFC: Readonly amendments** - https://wiki.php.net/rfc/readonly_amendments

**PHP Manual – Properties (readonly modifier)** - https://www.php.net/manual/en/language.oop5.properties.php

**PHP 8.1.0 Release Announcement** - https://www.php.net/releases/8.1/en.php

**PHP 8.2.0 Release Announcement** - https://www.php.net/releases/8.2/en.php

**PHP 8.3.0 Release Announcement (clone reinitialization)** - https://www.php.net/releases/8.3/en.php

**Readonly Properties (Secure PHP Development)** - https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Modern-PHP/Readonly-Properties.md

**Readonly Classes (Secure PHP Development)** - https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Modern-PHP/Readonly-Classes.md

**Modern Readonly Classes (Best Practices)** - https://github.com/voku/agent-skills/blob/main/skills/php-best-practices/rules/modern-readonly-classes.md