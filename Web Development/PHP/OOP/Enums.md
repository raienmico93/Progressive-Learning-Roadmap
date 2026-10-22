This cheat sheet provides a research-backed overview of PHP enums, structured for both academic clarity and beginner accessibility.

---

## Topic Overview

### Definitions

**Core Definition**
An enum (enumeration) in PHP is a special language construct that defines a new type with a fixed, limited set of possible legal values, called cases. Enums are built on top of classes and objects, providing type safety for representing a closed set of alternatives.

**Technical Definition**
Enums, introduced in PHP 8.1.0, are a restrictive layer on top of classes and class constants. An enum defines a new type whose cases are implemented as singleton instances of the enum class. All enums implement the internal `UnitEnum` interface (which provides the `cases()` static method), and backed enums additionally implement the `BackedEnum` interface (which provides `from()` and `tryFrom()`). Enums share the same namespace system as classes and are autoloadable in the same way. Enum case names are case-sensitive, while enum type names are case-insensitive.

**Beginner-Friendly Explanation**
An enum is like a dropdown menu with a fixed list of options. Just as a form might only allow you to choose "Small", "Medium", or "Large" for a T-shirt size, an enum restricts a variable to only the values you explicitly define. You cannot invent a new size—you must pick one of the allowed cases.

---

### Key Characteristics

- **Type Safety**: Enums define a distinct type; passing a raw string or integer where an enum is expected causes a `TypeError`.
- **Singleton Cases**: Each enum case is a singleton object; `Suit::Hearts === Suit::Hearts` is always `true`.
- **No State**: Enum cases cannot have properties (static or instance), constructors, or destructors.
- **No Inheritance**: Enums are implicitly `final` and cannot extend or be extended by other classes or enums (though they can implement interfaces and use traits).
- **Two Flavors**: Pure enums (no scalar backing) and backed enums (each case has a `string` or `int` value).
- **Methods and Interfaces**: Enums can contain methods and implement interfaces.
- **Autoloadable**: Enums participate in the same PSR-4 autoloading system as classes.

---

### Prerequisites

- **PHP 8.1.0 or later**: Enums were introduced in PHP 8.1.0.
- **Basic Object-Oriented PHP**: Understanding of classes, interfaces, and methods.
- **Familiarity with `match` expressions**: Enum methods often use `match` for case-specific logic.
- **A PSR-4 autoloader (recommended)**: Composer for loading enum classes.

---

### Related Programming Areas

- **Domain-Driven Design (DDD)**: Enums model domain concepts like order statuses, user roles, and payment methods.
- **Type Systems**: Enums leverage PHP's type declarations for compile-time and runtime safety.
- **Serialization**: Backed enums are designed for round-tripping to databases and APIs.
- **State Machines**: Enums provide a natural representation of finite states and transitions.
- **PSR Standards**: Enums fall under PSR-4 autoloading and PSR-12 coding style.

---

## Core Concepts / Features

---

### 1. Enums

#### Definitions

**Core Definition**
An enum is a type that defines a closed set of named values called cases, providing a way to represent a fixed number of alternatives with type safety.

**Technical Definition**
An enum is a language construct declared with the `enum` keyword. Each case is a singleton instance of the enum type. Enums are internally represented as classes and implement `UnitEnum` (and `BackedEnum` if backed). They may contain methods, constants, and may implement interfaces, but cannot have state or participate in inheritance.

**Beginner-Friendly Explanation**
An enum is a custom type with a predefined list of allowed values. Instead of using magic strings or numbers, you use descriptive case names that PHP enforces at the language level.

#### Purposes

- To define a closed set of possible values for a type, making invalid states unrepresentable.
- To replace loose "magic" strings and integer constants with type-safe alternatives.
- To improve code readability by giving descriptive names to finite alternatives.
- To enable exhaustive checking with `match` expressions and static analysis tools.

#### Syntax Rules and Structure

**General Syntax**
```php
enum EnumName {
    case CaseName1;
    case CaseName2;
    // ...
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `enum` | The keyword that declares an enumeration. |
| `EnumName` | A valid PHP label; follows the same naming rules as classes. |
| `case` | The keyword that declares a case. |
| `CaseName` | A valid PHP label; case names are case-sensitive. |
| `{ ... }` | Curly braces enclosing the list of cases (and optionally methods/constants). |

**Syntax Rules**

- Enum names must be valid PHP labels and are case-insensitive (like class names).
- Case names must be valid PHP labels and are case-sensitive.
- An enum may have zero or more cases; zero-case enums are syntactically valid but rarely useful.
- Enums share the same namespace system as classes and interfaces.

**Constraints and Limitations**

- **No properties**: Enum cases cannot have state; static or instance properties are forbidden.
- **No constructors/destructors**: Enums cannot define `__construct` or `__destruct`.
- **No inheritance**: Enums cannot extend classes or other enums, and cannot be extended.
- **No cloning**: Enum cases are singletons and cannot be cloned.
- **No magic methods (except `__call`, `__callStatic`, `__invoke`)**: `__toString` is not allowed because enums cannot implement `Stringable`.

#### Annotated Code Examples

**Example 1: A Basic Pure Enum**

```php
<?php
// Step 1: Declare an enum named Suit
enum Suit {
    case Hearts;
    case Diamonds;
    case Clubs;
    case Spades;
}

// Step 2: Use a case as a value
$card = Suit::Hearts;

// Step 3: Type-check a function against the enum
function describeSuit(Suit $suit): string {
    return match ($suit) {
        Suit::Hearts, Suit::Diamonds => "Red suit",
        Suit::Clubs, Suit::Spades => "Black suit",
    };
}

echo describeSuit($card) . "\n"; // Outputs: Red suit
?>
```

**Expected Output:**
```
Red suit
```

**Why This Output:**
The `Suit` enum defines four cases. `describeSuit` accepts only a `Suit` instance. The `match` expression maps the case to a string. Passing `Suit::Hearts` matches the first arm, producing `"Red suit"`.

**Step-by-Step Setup:**
1.  Save as `suit.php`.
2.  Run `php suit.php`.
3.  The output confirms type-safe case matching.

---

**Example 2: Enum Cases Are Singleton Objects**

```php
<?php
enum Status {
    case Active;
    case Inactive;
}

$a = Status::Active;
$b = Status::Active;

var_dump($a === $b);        // bool(true)
var_dump($a instanceof Status); // bool(true)
var_dump($a == Status::Inactive); // bool(false)
?>
```

**Expected Output:**
```
bool(true)
bool(true)
bool(false)
```

**Why This Output:**
Enum cases are singleton instances; the same case always yields the same object. Identity (`===`) is `true`. An enum case is an instance of its enum type. Comparison (`==`) between different cases is `false` because they are distinct objects.

#### Real-World Cases

- **Order Statuses**: `enum OrderStatus { case Pending; case Paid; case Shipped; case Delivered; }` replaces integer constants and prevents invalid status values.
- **User Roles**: `enum UserRole { case Admin; case Editor; case Viewer; }` used in authorization checks, ensuring only defined roles are accepted.
- **HTTP Methods**: `enum HttpMethod { case GET; case POST; case PUT; case DELETE; }` for routing and API client logic.

---

### 2. `enum` Keyword

#### Definitions

**Core Definition**
The `enum` keyword is the language construct that declares a new enumeration type.

**Technical Definition**
`enum` is a reserved keyword introduced in PHP 8.1.0 that begins an enumeration declaration. It is followed by the enum name, an optional backing type (for backed enums), an optional `implements` clause, and a body containing cases (and optionally methods and constants).

**Beginner-Friendly Explanation**
The `enum` keyword is like the word "recipe" at the top of a recipe card—it tells PHP, "I am about to define a new list of allowed values."

#### Purposes

- To signal the start of an enumeration definition to the PHP parser.
- To distinguish enums from classes and other constructs.
- To introduce a new type into the type system.

#### Syntax Rules and Structure

**General Syntax**
```php
enum EnumName {
    case Case1;
    case Case2;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `enum` | The required keyword. |
| `EnumName` | The name of the new type. |
| `{ ... }` | The body containing cases (and optionally methods/constants). |

**Syntax Rules**

- The `enum` keyword must be followed by a valid name.
- The body uses curly braces, like a class body.
- For backed enums, a colon and type (`: string` or `: int`) follow the name (see Section 4).

**Constraints and Limitations**

- `enum` is a reserved keyword and cannot be used as a function or class name.
- Enums cannot be declared conditionally in the same way as functions; they follow class-like declaration rules.

#### Annotated Code Examples

**Example 1: Declaring an Enum**

```php
<?php
// The "enum" keyword begins the declaration
enum Direction {
    case North;
    case South;
    case East;
    case West;
}

$d = Direction::North;
echo $d->name . "\n"; // Outputs: North
?>
```

**Expected Output:**
```
North
```

**Why This Output:**
The `enum` keyword creates a new type `Direction` with four cases. The read-only `name` property returns the case name as a string.

#### Real-World Cases

- **Configuration Options**: `enum Environment { case Development; case Staging; case Production; }` for application environment settings.
- **Days of the Week**: `enum DayOfWeek { case Monday; case Tuesday; /* ... */ }` for scheduling logic.

---

### 3. Pure Enums

#### Definitions

**Core Definition**
A pure enum is an enumeration where cases have no scalar backing value; each case is simply a named singleton object.

**Technical Definition**
A pure enum contains only "Pure Cases"—cases with no associated scalar equivalent. Each pure case is backed by a singleton object of the enum type. Pure enums implement the `UnitEnum` interface but not `BackedEnum`.

**Beginner-Friendly Explanation**
A pure enum is a list of names without any hidden numbers or strings attached. `Suit::Hearts` is just `Suit::Hearts`—it is not secretly equal to `0` or `"H"`.

#### Purposes

- To represent a closed set of alternatives where the names themselves carry all necessary meaning.
- To avoid coupling to database values or external representations when none are needed.
- To provide a lightweight, type-safe alternative to class constants for in-memory logic.

#### Syntax Rules and Structure

**General Syntax**
```php
enum PureEnumName {
    case Case1;
    case Case2;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| No colon or type | Pure enums have no backing type declaration. |
| `case Name;` | Cases have no `= value` assignment. |

**Syntax Rules**

- A pure enum may contain only pure cases; it cannot mix pure and backed cases.
- Pure cases have no `value` property.
- Pure enums still have the `name` property and the `cases()` static method.

**Constraints and Limitations**

- **No `from()` or `tryFrom()`**: These methods are exclusive to backed enums.
- **No scalar serialization**: Pure cases cannot be directly round-tripped to a scalar without custom logic.

#### Annotated Code Examples

**Example 1: Pure Enum in Practice**

```php
<?php
enum Season {
    case Spring;
    case Summer;
    case Autumn;
    case Winter;
}

$current = Season::Summer;

// Access the case name
echo $current->name . "\n"; // Outputs: Summer

// Iterate over all cases
foreach (Season::cases() as $season) {
    echo $season->name . "\n";
}
?>
```

**Expected Output:**
```
Summer
Spring
Summer
Autumn
Winter
```

**Why This Output:**
The pure enum `Season` has four cases. `$current->name` returns `"Summer"`. `Season::cases()` returns an array of all four singleton case objects, which are iterated and printed.

#### Real-World Cases

- **Internal State Flags**: A pure enum `enum Visibility { case Public; case Private; case Draft; }` used only within application logic, where the case names are self-explanatory and no database value is needed.
- **UI Options**: A pure enum `enum SortDirection { case Ascending; case Descending; }` for in-memory sorting logic.

---

### 4. Backed Enums

#### Definitions

**Core Definition**
A backed enum is an enumeration where every case has an associated scalar value (`string` or `int`), allowing cases to round-trip to databases and serialized formats.

**Technical Definition**
A backed enum is declared with a backing type (`: string` or `: int`), and every case must explicitly assign a unique scalar equivalent. Backed cases have a read-only `value` property. Backed enums implement the `BackedEnum` interface, which provides `from()` and `tryFrom()` static methods for constructing cases from scalars.

**Beginner-Friendly Explanation**
A backed enum is like a pure enum but each case has a "secret code" (a string or number). This code is useful when you need to save the enum to a database or send it over an API, because databases and APIs understand strings and numbers—not PHP objects.

#### Purposes

- To enable serialization and persistence of enum cases to databases or external systems.
- To provide a safe, typed way to convert raw input (e.g., from HTTP requests) into enum cases using `from()` or `tryFrom()`.
- To support APIs and data formats that require scalar representations.

#### Syntax Rules and Structure

**General Syntax**
```php
enum BackedEnumName: string|int {
    case Case1 = scalarValue;
    case Case2 = scalarValue;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `: string` or `: int` | The backing type. Only one type is allowed per enum (no union). |
| `= value` | Each case must explicitly assign a unique scalar value. No auto-generation. |
| `->value` | Read-only property returning the scalar value. |

**Syntax Rules**

- A backed enum may contain only backed cases; a pure enum may contain only pure cases.
- Backed values must be unique within the enum.
- Prior to PHP 8.2.0, backed values had to be literals or literal expressions; from PHP 8.2.0, constant scalar expressions are allowed.
- `from(int|string): self` returns the matching case or throws `ValueError`.
- `tryFrom(int|string): ?self` returns the matching case or `null`.

**Constraints and Limitations**

- **Single backing type**: You cannot mix `int` and `string` backing in one enum.
- **No auto-increment**: Values must be explicitly assigned.
- **`from()` throws on unknown values**: Use `tryFrom()` when the input is untrusted.
- **`value` is read-only**: You cannot assign a reference to `->value`.

#### Annotated Code Examples

**Example 1: String-Backed Enum**

```php
<?php
enum Suit: string {
    case Hearts = 'H';
    case Diamonds = 'D';
    case Clubs = 'C';
    case Spades = 'S';
}

// Access the backing value
echo Suit::Hearts->value . "\n"; // Outputs: H

// Convert from a scalar
$suit = Suit::from('D');
echo $suit->name . "\n"; // Outputs: Diamonds
?>
```

**Expected Output:**
```
H
Diamonds
```

**Why This Output:**
`Suit::Hearts->value` returns the string `'H'`. `Suit::from('D')` searches for a case with value `'D'` and returns `Suit::Diamonds`, whose `name` is `"Diamonds"`.

---

**Example 2: `from()` vs. `tryFrom()`**

```php
<?php
enum Status: int {
    case Pending = 0;
    case Active = 1;
    case Archived = 2;
}

// from() throws on unknown values
try {
    Status::from(99);
} catch (ValueError $e) {
    echo "ValueError: " . $e->getMessage() . "\n";
}

// tryFrom() returns null on unknown values
$result = Status::tryFrom(99);
var_dump($result); // NULL
?>
```

**Expected Output:**
```
ValueError: 99 is not a valid backing value for enum Status
NULL
```

**Why This Output:**
`from()` is strict and throws `ValueError` for unknown scalars. `tryFrom()` returns `null` instead, allowing the caller to handle the absence of a match.

#### Real-World Cases

- **Database Status Columns**: `enum OrderStatus: string { case Pending = 'pending'; case Paid = 'paid'; }` stored directly in a `VARCHAR` column, with `from()` used when hydrating models.
- **API Response Codes**: `enum HttpStatus: int { case OK = 200; case NotFound = 404; }` for type-safe HTTP response handling.
- **Payment Methods**: `enum PaymentMethod: string { case CreditCard = 'cc'; case PayPal = 'paypal'; }` parsed from request input via `tryFrom()`.

---

### 5. Enum Methods

#### Definitions

**Core Definition**
Enum methods are functions defined within an enum body that operate on case instances (instance methods) or on the enum type itself (static methods).

**Technical Definition**
Both pure and backed enums may contain methods. Instance methods have access to `$this`, which refers to the current case instance. Static methods are primarily used as alternative constructors. Methods may be public, private, or protected (though private and protected are equivalent because enums cannot be inherited).

**Beginner-Friendly Explanation**
Enum methods are like tools attached to each case. For example, a `Suit` case could have a `color()` method that returns `"Red"` for Hearts and Diamonds, and `"Black"` for Clubs and Spades.

#### Purposes

- To attach behavior intrinsic to the enum's state, keeping logic close to the data.
- To provide human-readable labels, colors, or descriptions for cases.
- To implement alternative constructors via static methods (e.g., `Size::fromLength()`）.
- To fulfill interface contracts when an enum implements an interface.

#### Syntax Rules and Structure

**General Syntax**
```php
enum EnumName {
    case Case1;
    // ...
    public function instanceMethod(): ReturnType { /* ... */ }
    public static function staticMethod(/* ... */): ReturnType { /* ... */ }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `public function method()` | Instance method; `$this` refers to the case instance. |
| `public static function method()` | Static method; called on the enum type itself. |
| `match ($this)` | Common pattern for returning different values per case. |

**Syntax Rules**

- Methods may be public, private, or protected; inheritance is not allowed, so private and protected are effectively equivalent.
- A static method named `cases()` cannot be defined; it is built-in and final.
- Instance methods have access to `$this->name` and, for backed enums, `$this->value`.

**Constraints and Limitations**

- **No state modification**: Methods cannot add or modify properties because enums have no state.
- **No `__toString`**: Enums cannot implement `Stringable` or define `__toString`.

#### Annotated Code Examples

**Example 1: Instance Method with `match`**

```php
<?php
enum Suit {
    case Hearts;
    case Diamonds;
    case Clubs;
    case Spades;

    public function color(): string {
        return match($this) {
            Suit::Hearts, Suit::Diamonds => 'Red',
            Suit::Clubs, Suit::Spades => 'Black',
        };
    }
}

echo Suit::Hearts->color() . "\n"; // Outputs: Red
echo Suit::Spades->color() . "\n"; // Outputs: Black
?>
```

**Expected Output:**
```
Red
Black
```

**Why This Output:**
The `color()` method uses `match ($this)` to return `'Red'` for Hearts and Diamonds, and `'Black'` for Clubs and Spades. `$this` refers to the specific case instance on which the method is called.

---

**Example 2: Static Method as Alternative Constructor**

```php
<?php
enum Size {
    case Small;
    case Medium;
    case Large;

    public static function fromLength(int $cm): self {
        return match(true) {
            $cm < 50 => self::Small,
            $cm < 100 => self::Medium,
            default => self::Large,
        };
    }
}

$size = Size::fromLength(75);
echo $size->name . "\n"; // Outputs: Medium
?>
```

**Expected Output:**
```
Medium
```

**Why This Output:**
`Size::fromLength(75)` is a static method that returns the appropriate case based on the input length. `75` is less than `100` but not less than `50`, so it matches `self::Medium`.

#### Real-World Cases

- **UI Labels**: A `Status` enum's `label()` method returns human-readable text like `"Pending approval"` for display in views.
- **Color Coding**: A `Priority` enum's `color()` method returns `"red"`, `"yellow"`, or `"green"` for use in dashboards.
- **Factory Methods**: A `Currency` enum's `fromCode(string $code): self` static method validates and constructs a case from an ISO code.

---

### 6. Enum Interfaces

#### Definitions

**Core Definition**
An enum interface is a standard PHP interface that an enum can implement, allowing enum cases to be used polymorphically wherever that interface is type-hinted.

**Technical Definition**
Both pure and backed enums may implement interfaces. If an enum implements an interface, any type check for that interface also accepts all cases of that enum. For backed enums, the `implements` clause follows the backing type declaration: `enum Suit: string implements Colorful`. All enums implicitly implement `UnitEnum`, and backed enums implicitly implement `BackedEnum`; user-defined classes cannot implement `UnitEnum` or `BackedEnum` directly, but user-defined interfaces may extend them.

**Beginner-Friendly Explanation**
An interface is a contract. If an enum implements an interface, it promises to provide the methods that the interface declares. This means you can write code that works with any object implementing that interface—including enum cases.

#### Purposes

- To allow enum cases to be used polymorphically with other classes that share the same interface.
- To enforce that certain methods are implemented by the enum.
- To enable dependency injection and type-hinting against abstractions rather than concrete enums.
- To extend internal enum interfaces (`UnitEnum`, `BackedEnum`) in userland for specialized type checks.

#### Syntax Rules and Structure

**General Syntax**
```php
interface SomeInterface {
    public function method(): ReturnType;
}

enum EnumName implements SomeInterface {
    case Case1;
    // ...
    public function method(): ReturnType { /* ... */ }
}
```

For backed enums:
```php
enum BackedEnumName: string implements SomeInterface {
    case Case1 = 'value';
    // ...
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `implements` | The keyword introducing the interface list. |
| `SomeInterface` | The interface to implement. Multiple interfaces may be listed, separated by commas. |
| Placement | For backed enums, `implements` comes after the backing type declaration. |

**Syntax Rules**

- All methods declared in the interface must be implemented by the enum.
- An enum may implement multiple interfaces.
- User-defined interfaces may extend `UnitEnum` or `BackedEnum`; however, user-defined classes cannot implement these internal interfaces directly.

**Constraints and Limitations**

- **No interface properties**: Interfaces with properties cannot be implemented by enums because enums cannot have state.
- **No `Stringable`**: Enums cannot implement `Stringable` because `__toString` is disallowed.

#### Annotated Code Examples

**Example 1: Enum Implementing a Custom Interface**

```php
<?php
interface HasLabel {
    public function label(): string;
}

enum Priority implements HasLabel {
    case Low;
    case Medium;
    case High;

    public function label(): string {
        return match($this) {
            Priority::Low => 'Low priority',
            Priority::Medium => 'Medium priority',
            Priority::High => 'High priority',
        };
    }
}

function printLabel(HasLabel $item): void {
    echo $item->label() . "\n";
}

printLabel(Priority::High); // Outputs: High priority
?>
```

**Expected Output:**
```
High priority
```

**Why This Output:**
`Priority` implements `HasLabel`, so `Priority::High` is accepted by `printLabel(HasLabel $item)`. The `label()` method fulfills the interface contract and returns the case-specific string.

---

**Example 2: Backed Enum with Interface (Correct Ordering)**

```php
<?php
interface HasCode {
    public function code(): int;
}

enum HttpStatus: int implements HasCode {
    case OK = 200;
    case NotFound = 404;

    public function code(): int {
        return $this->value;
    }
}

echo HttpStatus::OK->code() . "\n"; // Outputs: 200
?>
```

**Expected Output:**
```
200
```

**Why This Output:**
The `implements HasCode` clause correctly follows the `: int` backing type declaration. The `code()` method returns the backing `value` property, which is `200` for `OK`.

#### Real-World Cases

- **Comparable Enums**: An enum implementing `JsonSerializable` so cases can be safely encoded in API responses.
- **Domain Contracts**: An `OrderStatus` enum implementing a `HasLabel` interface so it can be rendered consistently alongside other labeled domain objects.
- **PSR Interfaces**: While enums cannot directly implement most PSR interfaces that require state, they can implement interfaces like `Stringable` (if not for the `__toString` restriction) or custom contracts.

---

### 7. Enum Best Practices

#### Definitions

**Core Definition**
Enum best practices are community-established conventions for declaring, naming, and using enums to maximize type safety, readability, and interoperability.

**Technical Definition**
Best practices encompass using backed enums for any finite set of domain states, preferring enums over class constants, naming cases in PascalCase, placing enums under a dedicated `Enums` namespace, using `from()`/`tryFrom()` at input boundaries, and leveraging `match` expressions for exhaustive case handling. They also include adhering to PSR-12 coding style and PSR-4 autoloading.

**Beginner-Friendly Explanation**
Best practices are the "rules of the road" for enums: use them instead of magic strings, give cases clear names, keep them organized, and always validate raw input before turning it into an enum.

#### Purposes

- To replace fragile magic strings and integer constants with type-safe enums.
- To ensure exhaustive handling of all cases when new cases are added.
- To maintain consistent naming and file organization across a codebase.
- To enable static analysis tools (PHPStan, Psalm) to verify correctness.

#### Guidelines Summary

| Practice | Description |
|----------|-------------|
| **Prefer Backed Enums for Persistence** | Use backed enums (`: string` or `: int`) when values must be stored in databases or APIs. |
| **Use Enums Over Class Constants** | Replace finite sets of class constants with enums for type safety. |
| **PascalCase for Cases** | Enum case names should use PascalCase (e.g., `Status::Active`）. |
| **Dedicated `Enums` Namespace** | Place enum classes under a dedicated `Enums` namespace/folder (e.g., `App\Enums`）. |
| **Use `from()`/`tryFrom()` at Boundaries** | Parse raw input with `tryFrom()` and use the enum type everywhere else. |
| **Use `match` for Case Logic** | Prefer `match ($this)` in enum methods for exhaustive, readable case handling. |
| **Avoid `__toString`** | Enums cannot implement `Stringable`; use a method like `label()` instead. |
| **Follow PSR-12** | Enums follow the same coding style as classes under PSR-12. |

**Syntax Rules**

- Enum case names must be PascalCase per community standards.
- Enums should be autoloaded via PSR-4 like classes.
- Backed enum values should be explicit and unique.

**Constraints and Limitations**

- **No state**: Best practices cannot override the language restriction that enums have no properties.
- **No inheritance**: Enums cannot be extended, so shared behavior should be provided via traits or interfaces.
- **No magic `__toString`**: Use explicit methods for string representations.

#### Annotated Code Examples

**Example 1: Backed Enum with Label Method (Best Practice Pattern)**

```php
<?php
declare(strict_types=1);

namespace App\Enums;

enum OrderStatus: string {
    case Pending = 'pending';
    case Paid = 'paid';
    case Shipped = 'shipped';
    case Delivered = 'delivered';

    public function label(): string {
        return match($this) {
            self::Pending => 'Pending payment',
            self::Paid => 'Payment received',
            self::Shipped => 'Shipped',
            self::Delivered => 'Delivered',
        };
    }
}

// Boundary: parse untrusted input safely
$input = 'paid';
$status = OrderStatus::tryFrom($input);

if ($status !== null) {
    echo $status->label() . "\n"; // Outputs: Payment received
} else {
    echo "Invalid status\n";
}
?>
```

**Expected Output:**
```
Payment received
```

**Why This Output:**
`OrderStatus::tryFrom('paid')` returns `OrderStatus::Paid`. The `label()` method uses `match` to return `"Payment received"`. This pattern follows best practices: backed enum, strict types, `tryFrom()` for input, and `match` for case logic.

---

**Example 2: Enum in a Dedicated Namespace with Interface**

```php
<?php
declare(strict_types=1);

namespace App\Enums;

interface HasLabel {
    public function label(): string;
}

enum UserRole: string implements HasLabel {
    case Admin = 'admin';
    case Editor = 'editor';
    case Viewer = 'viewer';

    public function label(): string {
        return match($this) {
            self::Admin => 'Administrator',
            self::Editor => 'Content Editor',
            self::Viewer => 'Read-only Viewer',
        };
    }
}
?>
```

**Expected Output:**
No output (class/enum definitions). `UserRole::Admin->label()` returns `"Administrator"`.

**Why This Output:**
The enum is placed in the `App\Enums` namespace, uses PascalCase cases, implements a labeled interface, and uses `match` for exhaustive labeling—all following best practices.

#### Real-World Cases

- **Laravel Applications**: Enums are placed under `app/Enums` and used in models, form requests, and policies. `tryFrom()` parses request input, and `label()` methods feed Blade templates.
- **Symfony Applications**: Enums implement `TranslatableInterface` or custom label interfaces, and are used in Doctrine entities with `enumType` mappings.
- **API Development**: Backed enums serialize to scalars in JSON responses, while `tryFrom()` safely handles deserialization from request payloads.

---

## References

**PHP Manual – Enumerations (Overview)** - https://www.php.net/manual/en/language.enumerations.php

**PHP Manual – Basic Enumerations** - https://www.php.net/manual/en/language.enumerations.basics.php

**PHP Manual – Backed Enumerations** - https://www.php.net/manual/en/language.enumerations.backed.php

**PHP Manual – Enumeration Methods** - https://www.php.net/manual/en/language.enumerations.methods.php

**PHP Manual – Enumeration Static Methods** - https://www.php.net/manual/en/language.enumerations.static-methods.php

**PHP Manual – Enumeration Constants** - https://www.php.net/manual/en/language.enumerations.constants.php

**PHP Manual – Differences from Objects** - https://www.php.net/manual/en/language.enumerations.object-differences.php

**PHP RFC: Enumerations** - https://wiki.php.net/rfc/enumerations

**PHP 8.1.0 Release Announcement (Enums)** - https://www.php.net/releases/8.1/en.php

**PSR-12: Extended Coding Style** - https://www.php-fig.org/psr/psr-12/

**PSR-4: Autoloading Standard** - https://www.php-fig.org/psr/psr-4/