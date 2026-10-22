Here is a comprehensive research cheat sheet on **PHP Magic Methods**, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Magic Methods

### Definitions

**Core Definition**
**Magic methods** are special methods in PHP that are automatically triggered by specific language operations (such as creating an object, accessing an inaccessible property, or converting an object to a string) rather than being called explicitly by name. They allow developers to intercept and customize the default behavior of objects.

**Technical Definition**
Magic methods are a set of reserved method names that start with the double-underscore prefix (`__`). These names are reserved by PHP; attempting to define a function with one of these names in any class triggers the associated magic behavior. If type declarations are used in the signature of a magic method, they must match the documented signature exactly; otherwise, a fatal `Error` is emitted (since PHP 8.0.0). The complete set of magic methods in modern PHP includes `__construct()`, `__destruct()`, `__call()`, `__callStatic()`, `__get()`, `__set()`, `__isset()`, `__unset()`, `__sleep()`, `__wakeup()`, `__serialize()`, `__unserialize()`, `__toString()`, `__invoke()`, `__set_state()`, `__clone()`, and `__debugInfo()`. As of PHP 8.5, `__sleep()` and `__wakeup()` are deprecated in favor of `__serialize()` and `__unserialize()`. All magic methods must be declared `public` (with the exception of `__construct()` and `__destruct()`, which must not declare a return type).

**Beginner-Friendly Explanation**
Magic methods are like **automatic reflexes** built into your objects. When something specific happens to an object—like someone trying to read a private property, convert the object to a string, or call a method that does not exist—PHP automatically runs a magic method instead of throwing an error or doing nothing. You do not call them yourself; PHP calls them for you at the right moment. They are “magic” because their invocation is invisible in the code—you just define the method, and PHP knows when to use it. The most familiar example is `__construct()`, which PHP runs automatically when you create a new object with `new`.

---

### Key Characteristics

- **Automatic Triggering**: Magic methods are invoked by PHP in response to specific operations, not by explicit method calls in your code.
- **Reserved Names**: All method names beginning with `__` are reserved by PHP and should not be used for any other purpose.
- **Strict Signatures (PHP 8.0+)**: Type declarations on magic methods must match the documented signatures exactly, or a fatal `Error` is thrown.
- **Public Visibility**: Almost all magic methods must be declared `public`; `__construct()` and `__destruct()` must not declare a return type.
- **Overloading Semantics**: PHP’s magic methods `__call()`, `__callStatic()`, `__get()`, `__set()`, `__isset()`, and `__unset()` implement what PHP calls “overloading”—dynamic creation and handling of properties and methods.
- **Static Analysis Trade-off**: Magic methods like `__get()`, `__set()`, `__call()`, and `__callStatic()` make property access and method dispatch invisible to static analysis tools (PHPStan, Psalm), IDEs, and refactoring tools.
- **Serialization Evolution**: `__sleep()` and `__wakeup()` are legacy serialization hooks (deprecated as of PHP 8.5); `__serialize()` and `__unserialize()` (PHP 7.4+) are the modern, recommended replacements.

---

### Prerequisites

- **PHP 8.0+ Recommended**: Strict magic method signature checks and many refinements require PHP 8.0+. `__serialize()` and `__unserialize()` require PHP 7.4+.
- **Object-Oriented Fundamentals**: Understanding of `class`, properties, methods, visibility, and object instantiation.
- **Awareness of Static Analysis**: Tools like PHPStan and Psalm lose type information when magic methods are used heavily; PHPDoc `@method` and `@property` stubs can partially compensate.

---

### Related Programming Areas

- **Object Lifecycle**: `__construct()`, `__destruct()`, and `__clone()` manage creation, destruction, and duplication.
- **Overloading**: `__call()`, `__callStatic()`, `__get()`, `__set()`, `__isset()`, and `__unset()` implement dynamic properties and methods.
- **Serialization**: `__serialize()`, `__unserialize()`, `__sleep()`, and `__wakeup()` control how objects are converted to and from storable formats.
- **String Conversion and Invocation**: `__toString()` and `__invoke()` make objects behave like strings or functions.
- **Debugging and Export**: `__debugInfo()` and `__set_state()` control how objects are inspected and exported.
- **Security**: Deserialization magic methods are a common vector for POP (Property-Oriented Programming) chain attacks; unserializing untrusted data is a critical security risk.

---

## Core Concepts / Features

---

### 1. Object Lifecycle: `__construct()`, `__destruct()`, `__clone()`

#### Definitions

**Core Definition**
- **`__construct()`** is the constructor, called automatically when a new object is created.
- **`__destruct()`** is the destructor, called automatically when an object is destroyed or the script ends.
- **`__clone()`** is called automatically when an object is duplicated using the `clone` operator.

**Technical Definition**
`__construct(mixed ...$values = ""): void` initializes an object on creation. `__destruct(): void` performs cleanup when an object is destroyed. `__clone(): void` is invoked after the shallow copy is made by `clone`, allowing the developer to deep-copy any object properties that should not be shared between the original and the clone. Parent constructors and destructors are not called implicitly; child classes must explicitly call `parent::__construct()` and `parent::__destruct()` if needed.

**Beginner-Friendly Explanation**
- **Constructor** = the object's “birth” routine: it sets up the object when it is created.
- **Destructor** = the object's “cleanup” routine: it releases resources when the object is no longer needed.
- **Clone** = the object's “photocopy” routine: it lets the object adjust itself after being duplicated (for example, deep-copying a nested object so the clone does not share it with the original).

#### Purposes

- To initialize object state and enforce invariants on creation.
- To release resources (files, database connections, locks) on destruction.
- To customize the behavior of `clone` so that duplicated objects are independent.

#### Syntax Rules and Structure

**General Syntax**
```php
public function __construct(Type $param) {}
public function __destruct() {}
public function __clone() {}
```

**Component Breakdown**

| Method | Signature | Triggered By |
|--------|-----------|--------------|
| `__construct` | `public function __construct(mixed ...$values = ""): void` | `new ClassName(...)` |
| `__destruct` | `public function __destruct(): void` | Object destruction or script shutdown |
| `__clone` | `public function __clone(): void` | `clone $object` |

**Constraints and Limitations**

- **No return types**: `__construct()` and `__destruct()` must not declare a return type; doing so is a fatal error.
- **Destructors are not guaranteed**: If the script terminates due to a fatal error, destructors may not run.
- **Shallow copy by default**: `clone` performs a shallow copy; `__clone()` must explicitly deep-copy object properties.

#### Annotated Code Examples

**Example 1: Constructor and Destructor**

```php
<?php
class Student
{
    private string $name;

    public function __construct(string $name)
    {
        $this->name = $name;
        echo "Student {$this->name} created.\n";
    }

    public function __destruct()
    {
        echo "Student {$this->name} destroyed.\n";
    }
}

$student = new Student('Monty'); // Outputs: Student Monty created.
unset($student);                  // Outputs: Student Monty destroyed.
?>
```

**Expected Output:**
```
Student Monty created.
Student Monty destroyed.
```

**Why This Output:**  
The constructor runs on `new`. When `$student` is `unset()`, the reference count drops to zero, and PHP immediately calls the destructor.

---

**Example 2: `__clone()` for Deep Copy**

```php
<?php
class Address
{
    public function __construct(public string $city) {}
}

class User
{
    public Address $address;

    public function __construct(public string $name, Address $address)
    {
        $this->address = $address;
    }

    public function __clone()
    {
        // Deep-copy the Address so the clone has its own copy
        $this->address = clone $this->address;
    }
}

$original = new User('Alice', new Address('Paris'));
$copy = clone $original;
$copy->address->city = 'London';

echo $original->address->city . "\n"; // Outputs: Paris
echo $copy->address->city . "\n";     // Outputs: London
?>
```

**Expected Output:**
```
Paris
London
```

**Why This Output:**  
Without `__clone()`, the `$address` property would be shared between `$original` and `$copy` (shallow copy). The `__clone()` method deep-copies the `Address` object, so the two `User` objects have independent addresses.

#### Real-World Cases

- **Resource Management**: A `FileWriter` closes its file handle in `__destruct()`.
- **Value Objects**: Immutable value objects use `__clone()` to deep-copy internal mutable structures.
- **Dependency Injection**: Constructors receive dependencies and validate them.

---

### 2. Overloading: `__call()`, `__callStatic()`, `__get()`, `__set()`, `__isset()`, `__unset()`

#### Definitions

**Core Definition**
These magic methods implement dynamic property and method handling. They are triggered when code accesses properties or calls methods that do not exist or are not accessible from the calling scope.

**Technical Definition**
- **`__call(string $name, array $arguments): mixed`** is triggered when an inaccessible or non-existent **instance method** is called.
- **`__callStatic(string $name, array $arguments): mixed`** is triggered when an inaccessible or non-existent **static method** is called.
- **`__get(string $name): mixed`** is triggered when reading an inaccessible or non-existent property.
- **`__set(string $name, mixed $value): void`** is triggered when writing to an inaccessible or non-existent property.
- **`__isset(string $name): bool`** is triggered when calling `isset()` or `empty()` on an inaccessible or non-existent property.
- **`__unset(string $name): void`** is triggered when calling `unset()` on an inaccessible or non-existent property.

**Beginner-Friendly Explanation**
These methods are PHP’s “what do I do if the thing you asked for is not there?” handlers. If you try to read a property that does not exist, `__get()` runs. If you try to call a method that does not exist, `__call()` runs. They let objects pretend to have properties and methods they do not explicitly define.

#### Purposes

- To implement dynamic properties (e.g., ORM entities that map database columns to properties).
- To create proxy objects that forward method calls to other objects.
- To build fluent interfaces and dynamic builders.
- To provide virtual properties computed on the fly.

#### Syntax Rules and Structure

**General Syntax**
```php
public function __call(string $name, array $arguments): mixed {}
public static function __callStatic(string $name, array $arguments): mixed {}
public function __get(string $name): mixed {}
public function __set(string $name, mixed $value): void {}
public function __isset(string $name): bool {}
public function __unset(string $name): void {}
```

**Component Breakdown**

| Method | Trigger |
|--------|---------|
| `__call` | Inaccessible instance method call. |
| `__callStatic` | Inaccessible static method call. |
| `__get` | Read of inaccessible/non-existent property. |
| `__set` | Write to inaccessible/non-existent property. |
| `__isset` | `isset()` or `empty()` on inaccessible/non-existent property. |
| `__unset` | `unset()` on inaccessible/non-existent property. |

**Constraints and Limitations**

- **Static analysis blind spot**: PHPStan cannot infer types for properties or methods handled by these magic methods; it returns `mixed` for every `__get` read and cannot detect typos in `__call` methods.
- **IDE autocompletion failure**: IDEs cannot autocomplete dynamic properties or methods without PHPDoc stubs.
- **Refactoring risk**: Finding all call sites of a dynamically dispatched method becomes guesswork.

#### Annotated Code Examples

**Example 1: `__get()` and `__set()` for Dynamic Properties**

```php
<?php
class DynamicProperties
{
    private array $data = [];

    public function __set(string $name, mixed $value): void
    {
        echo "Setting '$name' to '$value'\n";
        $this->data[$name] = $value;
    }

    public function __get(string $name): mixed
    {
        echo "Getting '$name'\n";
        return $this->data[$name] ?? null;
    }

    public function __isset(string $name): bool
    {
        return isset($this->data[$name]);
    }

    public function __unset(string $name): void
    {
        unset($this->data[$name]);
    }
}

$obj = new DynamicProperties();

$obj->name = 'Alice';           // Outputs: Setting 'name' to 'Alice'
echo $obj->name . "\n";         // Outputs: Getting 'name' \n Alice

var_dump(isset($obj->name));    // Outputs: bool(true)

unset($obj->name);
var_dump(isset($obj->name));    // Outputs: bool(false)
?>
```

**Expected Output:**
```
Setting 'name' to 'Alice'
Getting 'name'
Alice
bool(true)
bool(false)
```

**Why This Output:**  
When `$obj->name` is assigned, `__set()` intercepts the assignment and stores the value in the private `$data` array. When `$obj->name` is read, `__get()` retrieves it from the array. `__isset()` and `__unset()` operate on the same backing array.

---

**Example 2: `__call()` for Dynamic Method Dispatch**

```php
<?php
class User
{
    private array $attributes = ['email' => 'alice@example.com', 'name' => 'Alice'];

    public function __call(string $name, array $arguments): mixed
    {
        if (str_starts_with($name, 'get')) {
            $property = lcfirst(substr($name, 3));
            return $this->attributes[$property] ?? null;
        }
        throw new BadMethodCallException("Method {$name} not found");
    }
}

$user = new User();
echo $user->getEmail() . "\n"; // Outputs: alice@example.com
echo $user->getName() . "\n";  // Outputs: Alice
// $user->getNmae(); // Returns null (typo not caught)
?>
```

**Expected Output:**
```
alice@example.com
Alice
```

**Why This Output:**  
`__call()` intercepts calls to `getEmail()` and `getName()`, which do not explicitly exist. It derives the property name from the method name and returns the corresponding value from the `$attributes` array. A typo like `getNmae()` would silently return `null`, which is exactly the kind of bug static analysis cannot catch.

#### Real-World Cases

- **Eloquent ORM (Laravel)**: Models use `__get()` and `__set()` to access database columns as properties and `__call()` for query builder methods.
- **Doctrine Proxies**: Proxy objects use `__call()` to lazily load related entities.
- **Fluent Builders**: `__call()` can map method names to configuration options (e.g., `$builder->setName('x')->setEmail('y')`).

---

### 3. Serialization: `__serialize()`, `__unserialize()`, `__sleep()`, `__wakeup()`

#### Definitions

**Core Definition**
These magic methods control how objects are converted to and from storable formats (e.g., sessions, caches).

**Technical Definition**
- **`__serialize(): array`** (PHP 7.4+) is called by `serialize()`. It must return an associative array representing the object’s serialized form. If both `__serialize()` and `__sleep()` are defined, `__serialize()` takes precedence.
- **`__unserialize(array $data): void`** (PHP 7.4+) is called by `unserialize()` with the array returned by `__serialize()`.
- **`__sleep(): array`** (legacy, deprecated in PHP 8.5) returns an array of property names to serialize.
- **`__wakeup(): void`** (legacy, deprecated in PHP 8.5) re-establishes resources after deserialization.

**Beginner-Friendly Explanation**
- **`__serialize()`** is like packing a suitcase: you decide exactly which items to pack and how to label them.
- **`__unserialize()`** is like unpacking: you take the items out and put them back where they belong.
- **`__sleep()`/`__wakeup()`** were the old way: you listed which items to pack (`__sleep`) and then fixed things up after unpacking (`__wakeup`). They are being phased out in favor of the new methods.

#### Purposes

- To control which object state is persisted and which is excluded.
- To re-establish external resources (database connections, file handles) after deserialization.
- To avoid serializing sensitive or non-serializable data.

#### Syntax Rules and Structure

**General Syntax**
```php
public function __serialize(): array { /* return data */ }
public function __unserialize(array $data): void { /* restore */ }
```

**Component Breakdown**

| Method | Signature | Status |
|--------|-----------|--------|
| `__serialize` | `public function __serialize(): array` | Modern (PHP 7.4+) |
| `__unserialize` | `public function __unserialize(array $data): void` | Modern (PHP 7.4+) |
| `__sleep` | `public function __sleep(): array` | Legacy (deprecated PHP 8.5) |
| `__wakeup` | `public function __wakeup(): void` | Legacy (deprecated PHP 8.5) |

**Constraints and Limitations**

- **`__serialize()` must return an array**: Returning a non-array throws a `TypeError`.
- **Precedence**: If `__serialize()` exists, `__sleep()` is ignored; if the `Serializable` interface is implemented, its `serialize()` method is ignored in favor of `__serialize()`.
- **`__serialize()` does not control `json_encode()`**: Implementing `__serialize()` has no effect on `json_encode()`; use `JsonSerializable` for that.

#### Annotated Code Examples

**Example 1: Modern Serialization**

```php
<?php
class Connection
{
    private string $dsn;
    private string $username;
    private string $password;
    private ?PDO $link = null;

    public function __construct(string $dsn, string $username, string $password)
    {
        $this->dsn = $dsn;
        $this->username = $username;
        $this->password = $password;
        $this->connect();
    }

    private function connect(): void
    {
        $this->link = new PDO($this->dsn, $this->username, $this->password);
    }

    public function __serialize(): array
    {
        return [
            'dsn' => $this->dsn,
            'username' => $this->username,
            'password' => $this->password,
        ];
    }

    public function __unserialize(array $data): void
    {
        $this->dsn = $data['dsn'];
        $this->username = $data['username'];
        $this->password = $data['password'];
        $this->connect(); // re-establish the connection
    }
}
?>
```

**Expected Output:**
No direct output (the object is serialized and unserialized with the connection re-established).

**Why This Output:**  
`__serialize()` returns only the data needed to reconstruct the object, omitting the live `PDO` connection (which cannot be serialized). `__unserialize()` restores the data and calls `connect()` to re-establish the connection. This is the recommended modern pattern.

---

**Example 2: Legacy `__sleep()` and `__wakeup()`**

```php
<?php
class LegacyConnection
{
    protected ?PDO $link = null;
    private string $dsn;
    private string $username;
    private string $password;

    public function __construct(string $dsn, string $username, string $password)
    {
        $this->dsn = $dsn;
        $this->username = $username;
        $this->password = $password;
        $this->connect();
    }

    private function connect(): void
    {
        $this->link = new PDO($this->dsn, $this->username, $this->password);
    }

    public function __sleep(): array
    {
        return ['dsn', 'username', 'password'];
    }

    public function __wakeup(): void
    {
        $this->connect();
    }
}
?>
```

**Expected Output:**
No direct output. This code still works but is deprecated in PHP 8.5.

**Why This Output:**  
`__sleep()` returns the names of the properties to serialize; `__wakeup()` re-establishes the connection. This legacy pattern is being replaced by `__serialize()`/`__unserialize()` because the modern methods offer clearer data flow and avoid side effects.

#### Real-World Cases

- **Session Objects**: Objects stored in `$_SESSION` use serialization; magic methods control which state is persisted.
- **Cache Serialization**: Caching libraries serialize complex objects; `__serialize()` controls the serialized representation.
- **Legacy Migration**: Projects migrating to PHP 8.5+ must replace `__sleep()`/`__wakeup()` with `__serialize()`/`__unserialize()`.

---

### 4. String Conversion and Invocation: `__toString()`, `__invoke()`

#### Definitions

**Core Definition**
- **`__toString()`** defines how an object behaves when it is treated as a string (e.g., `echo $obj`).
- **`__invoke()`** defines how an object behaves when it is called like a function (e.g., `$obj()`).

**Technical Definition**
`__toString(): string` is called when an object is used in a string context. As of PHP 8.0.0, the return value follows standard type semantics (coerced to `string` if strict typing is disabled), and any class with `__toString()` implicitly implements the `Stringable` interface. `__invoke(mixed ...$args): mixed` is called when an object is invoked as a function, enabling objects to be used as callables (e.g., in `usort()`, `array_map()`, or any callback parameter).

**Beginner-Friendly Explanation**
- **`__toString()`** tells PHP what to print when someone tries to use your object as a string. Without it, PHP would throw an error.
- **`__invoke()`** makes your object callable—it lets you write `$object()` as if the object were a function.

#### Purposes

- To provide a human-readable string representation of an object.
- To make objects usable as callables without needing a separate closure.
- To implement value objects that can be converted to strings (e.g., `Money`, `Email`).

#### Syntax Rules and Structure

**General Syntax**
```php
public function __toString(): string {}
public function __invoke(mixed ...$args): mixed {}
```

**Component Breakdown**

| Method | Trigger | Return Type |
|--------|---------|-------------|
| `__toString` | String context (`echo`, concatenation, `strlen()`) | `string` |
| `__invoke` | `$obj(...)` | `mixed` |

**Constraints and Limitations**

- **`__toString()` must return a string**: Returning a non-string (or throwing) causes a fatal error.
- **`__toString()` and strict types**: Under `strict_types=1`, a `Stringable` object is not automatically accepted where a `string` is required; use a union type like `string|Stringable` if needed.

#### Annotated Code Examples

**Example 1: `__toString()` for a Value Object**

```php
<?php
class Money
{
    public function __construct(
        private int $amount,
        private string $currency
    ) {}

    public function __toString(): string
    {
        return number_format($this->amount / 100, 2) . ' ' . $this->currency;
    }
}

$price = new Money(1999, 'USD');
echo $price . "\n"; // Outputs: 19.99 USD
?>
```

**Expected Output:**
```
19.99 USD
```

**Why This Output:**  
When `$price` is concatenated with `"\n"`, PHP calls `__toString()`, which returns the formatted monetary string.

---

**Example 2: `__invoke()` as a Comparator**

```php
<?php
class SortByKey
{
    public function __construct(private string $key) {}

    public function __invoke(array $a, array $b): int
    {
        return $a[$this->key] <=> $b[$this->key];
    }
}

$customers = [
    ['id' => 1, 'first_name' => 'John'],
    ['id' => 3, 'first_name' => 'Alice'],
    ['id' => 2, 'first_name' => 'Bob'],
];

usort($customers, new SortByKey('first_name'));
print_r($customers);
?>
```

**Expected Output:**
```
Array
(
    [0] => Array ( [id] => 3 [first_name] => Alice )
    [1] => Array ( [id] => 2 [first_name] => Bob )
    [2] => Array ( [id] => 1 [first_name] => John )
)
```

**Why This Output:**  
`usort()` expects a callable. The `SortByKey` object implements `__invoke()`, so it can be used as a callable. `usort()` invokes the object like a function, passing two array elements to compare.

#### Real-World Cases

- **Value Objects**: `Email`, `Money`, `UserId` implement `__toString()` for easy display.
- **Comparators**: Sorting strategies implemented as invokable objects.
- **Middleware**: Middleware classes implement `__invoke()` to be called by frameworks.

---

### 5. Debugging and Export: `__debugInfo()`, `__set_state()`

#### Definitions

**Core Definition**
- **`__debugInfo()`** customizes the output of `var_dump()` and `print_r()` on an object.
- **`__set_state()`** is a static method used by `var_export()` to reconstruct an object from its exported representation.

**Technical Definition**
`__debugInfo(): ?array` is called by `var_dump()` and `print_r()` to determine which properties (and what values) are displayed. If it returns `null`, PHP falls back to showing all properties. `__set_state(array $properties): object` is a static method called by `var_export()` when the exported code is executed; it must return an instance of the class. As of PHP 8.0, `__set_state()` must follow a strict signature.

**Beginner-Friendly Explanation**
- **`__debugInfo()`** controls what developers see when they inspect an object with `var_dump()`. It lets you hide internal details and show only meaningful data.
- **`__set_state()`** is the “reconstruct from export” method. When `var_export()` generates PHP code to recreate an object, that code calls `__set_state()` to build the new object.

#### Purposes

- To provide a clean, meaningful debugging representation of an object.
- To support round-tripping objects through `var_export()` and `eval()`.
- To hide sensitive or noisy properties from debug output.

#### Syntax Rules and Structure

**General Syntax**
```php
public function __debugInfo(): ?array {}
public static function __set_state(array $properties): object {}
```

**Component Breakdown**

| Method | Trigger | Return Type |
|--------|---------|-------------|
| `__debugInfo` | `var_dump()`, `print_r()` | `?array` |
| `__set_state` | `var_export()` output execution | `object` |

**Constraints and Limitations**

- **`__set_state()` must be static**: It is called statically by `var_export()`.
- **Inheritance caveat**: A static `__set_state()` defined in a parent class is called for child classes, which can lead to creating an object of the wrong type if not careful.

#### Annotated Code Examples

**Example 1: `__debugInfo()` Hiding Internal Data**

```php
<?php
class User
{
    public function __construct(
        private string $name,
        private string $passwordHash,
        private array $internalFlags = []
    ) {}

    public function __debugInfo(): ?array
    {
        return [
            'name' => $this->name,
            'passwordHash' => '***hidden***',
        ];
    }
}

$user = new User('Alice', 'hashed_secret', ['admin' => true]);
var_dump($user);
?>
```

**Expected Output:**
```
object(User)#1 (2) {
  ["name"]=>
  string(5) "Alice"
  ["passwordHash"]=>
  string(12) "***hidden***"
}
```

**Why This Output:**  
`__debugInfo()` returns an array containing only the properties that should be shown. The `passwordHash` is masked, and `internalFlags` is omitted entirely.

---

**Example 2: `__set_state()` for `var_export()` Round-Trip**

```php
<?php
class Point
{
    public function __construct(
        public float $x,
        public float $y
    ) {}

    public static function __set_state(array $properties): static
    {
        return new static($properties['x'], $properties['y']);
    }
}

$point = new Point(3.0, 4.0);
$exported = var_export($point, true);
echo $exported . "\n";

// Reconstruct the object
eval('$restored = ' . $exported . ';');
var_dump($restored);
?>
```

**Expected Output:**
```
\Point::__set_state(array(
   'x' => 3.0,
   'y' => 4.0,
))
object(Point)#2 (2) {
  ["x"]=>
  float(3)
  ["y"]=>
  float(4)
}
```

**Why This Output:**  
`var_export()` generates code that calls `Point::__set_state()` with the exported properties. `__set_state()` creates a new `Point` with those values, allowing the object to be reconstructed.

#### Real-World Cases

- **Debugging Tools**: Frameworks use `__debugInfo()` to present clean object representations in debug toolbars.
- **Configuration Export**: `var_export()` and `__set_state()` are used to export configuration objects to cache files that can be re-included.

---

### 6. Magic Method Best Practices

#### Definitions

**Core Definition**
Best practices for magic methods are guidelines for using them safely, predictably, and only where they add genuine value, avoiding the pitfalls of invisible behavior and lost static analysis.

**Technical Definition**
The overarching recommendation is to **avoid magic-heavy design**. Prefer explicit, typed properties and methods. Use magic methods only in framework infrastructure (ORM base classes, proxies, dynamic builders) where the trade-off is justified, and compensate for the lost static visibility with PHPDoc stubs (`@method`, `@property`) or PHPStan plugins. For serialization, prefer `__serialize()`/`__unserialize()` over the deprecated `__sleep()`/`__wakeup()`. Never trust serialized data from untrusted sources, as magic methods like `__wakeup()` and `__unserialize()` are common vectors for POP chain attacks.

**Beginner-Friendly Explanation**
Magic methods are powerful, but they are also “invisible” to tools and other developers. The best practice is simple: **use them sparingly**. If you can write an explicit, typed method or property, do that instead. Use magic methods only when you are building a framework or a library where dynamic behavior is essential—and always document what they do.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Prefer explicit, typed members** | Use typed properties and named methods instead of `__get`, `__set`, `__call` whenever possible. |
| **Use magic only in infrastructure** | Reserve magic methods for ORM base classes, proxies, dynamic builders, and similar framework-level code. |
| **Compensate with PHPDoc** | Use `@method` and `@property` stubs or PHPStan plugins to restore static analysis coverage for magic-heavy classes. |
| **Migrate to `__serialize()`/`__unserialize()`** | Replace deprecated `__sleep()`/`__wakeup()` with the modern serialization API. |
| **Never unserialize untrusted data** | Deserialization triggers `__wakeup()`/`__unserialize()`, which attackers can exploit; validate and sanitize first. |
| **Keep `__toString()` side-effect free** | `__toString()` should return a string representation, not perform I/O or mutate state. |
| **Use `__invoke()` for single-purpose callables** | When an object’s sole purpose is to be called, `__invoke()` is appropriate; otherwise, use a named method. |
| **Protect `__clone()` for deep copy** | If the object contains nested objects that should not be shared, implement `__clone()` to deep-copy them. |

**Constraints and Limitations**

- **Static analysis limitations are real**: PHPStan returns `mixed` for `__get` reads and cannot catch typos in `__call`-dispatched methods.
- **IDE support degrades**: Autocompletion and refactoring tools cannot see dynamic properties and methods.
- **Security risk**: Serialization magic methods are a well-known attack surface; do not unserialize user-supplied data.

#### Annotated Code Examples

**Example 1: Bad vs. Good Design**

```php
<?php
// BAD: Magic-heavy design — everything hidden from static analysis
class UserBad
{
    private array $attributes = [];

    public function __get(string $name): mixed
    {
        return $this->attributes[$name] ?? null;
    }

    public function __set(string $name, mixed $value): void
    {
        $this->attributes[$name] = $value; // no type check
    }
}

$bad = new UserBad();
$bad->email = 'alice@example.com'; // __set — no type check
$bad->name = 42;                   // accepted silently — wrong type

// GOOD: Explicit, typed properties — static analysis works
final class UserGood
{
    public function __construct(
        public readonly string $id,
        public string $email,
        public string $name,
    ) {}
}

$good = new UserGood('1', 'alice@example.com', 'Alice');
echo $good->email . "\n"; // Outputs: alice@example.com
?>
```

**Expected Output:**
```
alice@example.com
```

**Why This Output:**  
The “bad” design silently accepts a wrong type (`$bad->name = 42`) because `__set()` has no type enforcement. The “good” design uses explicit typed properties, so IDEs and PHPStan can see every property and detect type errors at analysis time.

---

#### Real-World Cases

- **Laravel Eloquent**: Uses `__get()`/`__set()` for attribute access but provides PHPDoc stubs and IDE helper packages to compensate.
- **Doctrine Proxies**: Use `__call()` for lazy loading but are generated code, not hand-written application logic.
- **Static Analysis in CI**: Projects use PHPStan rules like `phpstan-rules-no-magic` to forbid magic methods in application code and restrict them to infrastructure layers.

---

## References

**PHP Manual – Magic Methods** - https://static.php.net/manual/en/language.oop5.magic.php

**PHP Manual – Overloading (Magic Methods)** - https://www.php.net/manual/en/language.oop5.overloading.php

**PHP Manual – Object Cloning (`__clone`)** - https://www.php.net/manual/en/language.oop5.cloning.php

**PHP Manual – Constructors and Destructors** - https://www.php.net/manual/en/language.oop5.decon.php

**PHP Manual – `__serialize()` and `__unserialize()`** - https://static.php.net/manual/en/language.oop5.magic.php#object.serialize

**PHP Manual – `__sleep()` and `__wakeup()`** - https://static.php.net/manual/en/language.oop5.magic.php#object.sleep

**Drupal – PHP `__sleep()`/`__wakeup()` Replaced with Modern Serialization** - https://www.drupal.org/node/3578423

**Symfony – Rule `modern_serialization_methods`** - https://cs.symfony.com/doc/rules/php/php_unit_construct.html

**Agent Skills – Avoid Magic-Heavy Design** - https://github.com/voku/agent-skills/blob/main/skills/php-best-practices/rules/design-no-magic.md

**Envato Tuts+ – PHP Magic Methods Cheatsheet** - https://code.tutsplus.com/php-magic-methods-cheat-sheet--cms-34681t

**PHP Architect – PHP's Magic Methods** - https://www.phparch.com/2024/05/phps-magic-methods/

**PHPStan Rules – No Magic** - https://packagist.org/packages/frankvanhest/phpstan-rules-no-magic

**Grupo Oesía – Deserialization in PHP (POP Chains and Magic Methods)** - https://grupooesia.com/2023/09/18/deserializacion-en-php/