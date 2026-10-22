This cheat sheet provides a research-backed overview of PHP object cloning, covering the mechanics of `clone`, the `__clone()` magic method, the critical distinction between shallow and deep copying, and modern best practices including interaction with `readonly` properties in PHP 8.3+.

---

## Topic Overview

### Definitions

**Core Definition**
Object cloning in PHP is the process of creating a new object that is a copy of an existing object, using the `clone` keyword. The copied object receives all properties of the original, but the depth of that copy depends on the cloning strategy employed.

**Technical Definition**
An object copy is created by using the `clone` keyword, which triggers a shallow copy of all object properties. If the object defines a `__clone()` magic method, it is invoked on the newly created copy after the shallow copy is complete, allowing the class to “mend” any shared references by performing a deep clone. The `__clone()` method cannot be called directly from outside a class; it is invoked by the engine when the `clone` operator is used. Cloning creates a new object without invoking the constructor.

**Beginner-Friendly Explanation**
Cloning is like photocopying a document. The `clone` keyword makes the photocopy. By default, PHP makes a “shallow” photocopy: if the original document had sticky notes attached to pages (nested objects), the photocopy gets sticky notes pointing to the exact same physical notes—not new ones. If you want the photocopy to have its own independent sticky notes, you need to use the `__clone()` method to make new copies of them.

---

### Key Characteristics

- **Constructor Bypass**: Cloning does not call the `__construct()` method; the new object is created in its already-initialized state.
- **Shallow by Default**: Without a `__clone()` method, all properties are copied by value for scalars and by reference for objects.
- **`__clone()` is Not an Override**: The normal cloning process always occurs first (the shallow copy); `__clone()` runs afterward on the new copy and is responsible for “fixing” any unwanted shared references.
- **Visibility-Agnostic Copying**: Private and protected properties are copied during cloning, and `__clone()` runs in the context of the cloned object, allowing it to access private members.
- **PHP 8.3 Readonly Reinitialization**: As of PHP 8.3.0, `readonly` properties can be re-initialized inside `__clone()`, enabling ergonomic “copy-with-changes” patterns for immutable objects.
- **Static Properties Unaffected**: Static properties belong to the class, not the instance, and are not cloned.

---

### Prerequisites

- **PHP 8.1+** is recommended for modern readonly property interaction; **PHP 8.3+** is required for re-initializing readonly properties inside `__clone()`.
- **Object-Oriented PHP Fundamentals**: Classes, properties, constructors, and visibility.
- **Understanding of PHP References**: Knowledge of how objects are passed by reference (handle-based) versus scalars passed by value.

---

### Related Programming Areas

- **Prototype Design Pattern**: The `clone` keyword is PHP’s built-in support for the Prototype creational pattern, allowing new objects to be created by copying an existing prototype.
- **Immutability and Value Objects**: Cloning intersects with `readonly` properties and immutable design patterns, where “changes” are implemented as clones with modified properties.
- **Resource Management**: Objects holding resources (file handles, database connections) require special handling in `__clone()` because resources are shared, not copied.
- **Serialization**: `serialize()`/`unserialize()` is an alternative deep-copy mechanism, with different trade-offs in performance and correctness.

---

## Core Concepts / Features

---

### 1. `clone` Keyword

#### Definitions

**Core Definition**
The `clone` keyword is the language operator that creates a shallow copy of an object.

**Technical Definition**
`clone $object` triggers the engine to allocate a new object, copy all properties from the original (scalars by value, objects by handle/reference), and then invoke the `__clone()` method on the new object if it exists. The original object is not modified.

**Beginner-Friendly Explanation**
`clone` is the photocopy button. It makes a new object, but the “sticky notes” (nested objects) are still shared unless you take extra steps.

#### Purposes

- To create an independent object without re-running the constructor logic.
- To implement the Prototype pattern for creating new objects from a pre-configured prototype.
- To avoid the cost of reconstructing an object when a copy suffices.

#### Syntax Rules and Structure

**General Syntax**
```php
$copy = clone $original;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `clone` | The operator keyword. |
| `$original` | The expression evaluating to the object to be copied. |
| Result | A new object with properties shallow-copied. |

**Syntax Rules**

- `clone` can be used in any expression context, including `(clone $obj)->method()`.
- The expression after `clone` must evaluate to an object; cloning a non-object is a fatal error.

**Constraints and Limitations**

- **No constructor**: The `__construct()` method is not called on the clone.
- **Shallow by default**: Nested objects and resources are shared unless `__clone()` intervenes.
- **Cloning `readonly` classes**: Prior to PHP 8.3, re-initializing readonly properties inside `__clone()` was forbidden; PHP 8.3+ allows it.

#### Annotated Code Examples

**Example 1: Basic Clone and Shallow Copy Behavior**

```php
<?php
class Address {
    public function __construct(public string $city) {}
}

class User {
    public function __construct(
        public string $name,
        public Address $address
    ) {}
}

$original = new User("Alice", new Address("Paris"));
$copy = clone $original;

// Step 1: Scalar property is independent
$copy->name = "Bob";

// Step 2: Nested object is SHARED — modifying it affects both!
$copy->address->city = "London";

echo "Original: {$original->name}, {$original->address->city}\n";
echo "Copy:     {$copy->name}, {$copy->address->city}\n";
?>
```

**Expected Output:**
```
Original: Alice, London
Copy:     Bob, London
```

**Why This Output:**
The scalar `$name` was shallow-copied by value, so changing the copy’s name did not affect the original. The `$address` property holds an object handle; the shallow copy copied the handle, so both `$original->address` and `$copy->address` point to the **same** `Address` instance. Changing `city` through the copy is visible through the original.

**Step-by-Step Setup:**
1.  Save as `shallow_clone.php`.
2.  Run `php shallow_clone.php`.
3.  The output demonstrates the shared nested object.

---

**Example 2: Accessing a Cloned Object’s Member Directly**

```php
<?php
$dateTime = new DateTime();
echo (clone $dateTime)->format('Y') . "\n";
?>
```

**Expected Output:** *(Outputs the current year, e.g., `2026`)*

**Why This Output:**
The `clone` operator returns a new object, and the `->format()` method can be called directly on the result of the clone expression without assigning it to an intermediate variable.

#### Real-World Cases

- **Prototype Pattern**: A pre-configured `EmailTemplate` object is cloned for each recipient, then personalized, avoiding repeated template-loading logic.
- **Configuration Snapshots**: A `Config` object is cloned to create a snapshot before applying temporary overrides, preserving the original settings.

---

### 2. `__clone()` Magic Method

#### Definitions

**Core Definition**
`__clone()` is a magic method that PHP automatically invokes on a newly cloned object, allowing the class to customize the cloning process—most commonly to perform a deep copy of nested objects.

**Technical Definition**
`__clone(): void` is an instance method that runs on the clone **after** the engine has performed the shallow copy. It cannot be called directly from outside the class; it may be invoked via `self::__clone()` within a derived class. Its return value is ignored when invoked via the `clone` operator. It runs in the context of the cloned object, so it has access to private and protected properties.

**Beginner-Friendly Explanation**
`__clone()` is a “post-photocopy touch-up” step. After the photocopier has made the copy, `__clone()` runs on the copy and lets you replace any shared sticky notes with fresh, independent ones.

#### Purposes

- To convert a shallow copy into a deep copy by explicitly cloning nested objects.
- To re-establish resources (e.g., file handles, database connections) that cannot be meaningfully shared.
- To reset or regenerate transient state (e.g., timestamps, unique IDs) on the clone.
- To prevent cloning entirely by throwing an exception inside `__clone()`.

#### Syntax Rules and Structure

**General Syntax**
```php
public function __clone(): void {
    $this->nestedObject = clone $this->nestedObject;
    // additional re-initialization logic
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `public function __clone()` | The magic method signature. |
| `: void` | Recommended return type; return values are ignored by the `clone` operator. |
| `$this` | Refers to the newly cloned object. |

**Syntax Rules**

- The method must be declared as `public` (or omitted visibility, which defaults to public) to be effective for standard cloning.
- It runs **after** the shallow copy; the object already has all properties copied at that point.
- A subclass that overrides `__clone()` should call `parent::__clone()` to ensure parent-level deep-copy logic runs.

**Constraints and Limitations**

- **Cannot be called directly**: `$obj->__clone()` from outside the class is forbidden.
- **Readonly properties**: Prior to PHP 8.3, writing to a `readonly` property inside `__clone()` threw an `Error`; PHP 8.3+ permits re-initialization.
- **Not a constructor replacement**: `__clone()` runs on an already-initialized copy; it should not be treated as a constructor.

#### Annotated Code Examples

**Example 1: Deep Clone of a Nested Object**

```php
<?php
class LineItem {
    public function __construct(
        public string $product,
        public int $quantity
    ) {}
}

class Order {
    public array $items = [];

    public function __construct() {
        $this->items[] = new LineItem("Widget", 2);
    }

    public function __clone(): void {
        // Step 1: Manually clone each nested object
        $this->items = array_map(
            fn(LineItem $item) => clone $item,
            $this->items
        );
    }
}

$original = new Order();
$copy = clone $original;

// Step 2: Modify the nested object through the copy
$copy->items[0]->quantity = 99;

echo "Original quantity: {$original->items[0]->quantity}\n";
echo "Copy quantity:     {$copy->items[0]->quantity}\n";
?>
```

**Expected Output:**
```
Original quantity: 2
Copy quantity:     99
```

**Why This Output:**
Without `__clone()`, the `items` array would contain the **same** `LineItem` objects in both the original and the copy. The `__clone()` method iterates over the array and replaces each element with a `clone` of itself, so the copy has its own independent `LineItem` instances. Modifying the copy’s item no longer affects the original.

---

**Example 2: Re-establishing a Resource in `__clone()`**

```php
<?php
class FileReader {
    private $handle;

    public function __construct(private string $path) {
        $this->handle = fopen($this->path, 'r');
    }

    public function __clone(): void {
        // Step 1: Re-open the file for the clone
        $this->handle = fopen($this->path, 'r');
    }

    public function readLine(): string {
        return fgets($this->handle);
    }

    public function __destruct() {
        if (is_resource($this->handle)) {
            fclose($this->handle);
        }
    }
}

$file = tempnam(sys_get_temp_dir(), 'clone_demo_');
file_put_contents($file, "First line\nSecond line\n");

$original = new FileReader($file);
$copy = clone $original;

echo $original->readLine(); // Outputs: First line
echo $copy->readLine();     // Outputs: First line (independent cursor)
?>
```

**Expected Output:**
```
First line
First line
```

**Why This Output:**
File handles are resources and are shared by default when an object is cloned. The `__clone()` method explicitly re-opens the file for the clone, giving each object its own independent file pointer. Both objects can read from the beginning independently.

#### Real-World Cases

- **Order/LineItem Graphs**: An `Order` containing `LineItem` objects uses `__clone()` to deep-clone each item, preventing “remove item from copy” operations from mutating the original order.
- **Database Connection Wrappers**: A repository object holding a `PDO` connection implements `__clone()` to open a fresh connection for the clone, avoiding shared transaction state.
- **Preventing Cloning**: Singleton classes implement `__clone()` with `throw new LogicException("Cannot clone singleton");` to enforce the singleton invariant.

---

### 3. Shallow Cloning

#### Definitions

**Core Definition**
Shallow cloning is PHP’s default cloning behavior, where scalar properties are copied by value but object properties are copied by handle, causing the original and the clone to share nested objects.

**Technical Definition**
When `clone` is used without an intervening `__clone()` method, PHP copies all properties of the object. Scalar values (integers, strings, booleans, floats) and arrays (which have copy-on-write semantics) are effectively independent. Object properties are reference-like handles; the clone receives a copy of the handle, pointing to the same underlying object instance.

**Beginner-Friendly Explanation**
A shallow copy is like photocopying a document that has sticky notes. The paper is new, but the sticky notes are the same physical notes stuck to both pages. If you write on a sticky note through the copy, the original sees the change.

#### Purposes

- To provide a fast, cheap copy when nested objects are immutable or sharing is acceptable.
- To serve as the baseline behavior before `__clone()` applies deep-copy corrections.
- To clone value objects that contain only scalars or other immutable objects.

#### Syntax Rules and Structure

**General Syntax**
```php
$copy = clone $original; // shallow copy by default
```

**Component Breakdown**

| Property Type | Shallow Copy Behavior |
|---------------|----------------------|
| Scalars (int, float, string, bool) | Independent copies (by value). |
| Arrays | Copy-on-write; effectively independent until modified. |
| Objects | Shared handles; both point to the same instance. |
| Resources | Shared resource handles. |
| `null` | Independent. |

**Syntax Rules**

- Shallow cloning is the default; no special syntax is required.
- Adding a `__clone()` method changes the behavior from shallow to whatever the method implements (typically deep).

**Constraints and Limitations**

- **Shared mutable state**: Mutating a shared nested object through one clone is visible through all clones and the original.
- **Not suitable for mutable object graphs**: Orders, carts, and documents with nested mutable structures require deep cloning.
- **Resources remain shared**: File handles, sockets, and database connections are not duplicated.

#### Annotated Code Examples

**Example 1: Demonstrating Shared State in Shallow Clones**

```php
<?php
class Counter {
    public int $value = 0;
}

class Holder {
    public Counter $counter;
    public function __construct() {
        $this->counter = new Counter();
    }
}

$original = new Holder();
$original->counter->value = 10;

$copy = clone $original; // shallow copy

// Modify the nested object through the copy
$copy->counter->value = 99;

echo "Original counter: {$original->counter->value}\n";
echo "Copy counter:     {$copy->counter->value}\n";
?>
```

**Expected Output:**
```
Original counter: 99
Copy counter:     99
```

**Why This Output:**
The `Holder` class has no `__clone()` method, so cloning is shallow. The `$counter` property holds an object handle. The clone’s `$counter` points to the same `Counter` instance as the original’s. Writing `99` through the copy mutates that shared instance, so both see `99`.

#### Real-World Cases

- **Immutable Value Objects**: Cloning a `Money` object that holds only an `int` amount and a `string` currency is safe as a shallow copy because there are no mutable nested objects.
- **Snapshotting Scalars**: Cloning a simple `Pagination` object with `int $page` and `int $perPage` is a valid shallow clone.
- **Prototype with Immutable Dependencies**: Cloning a `ReportGenerator` that holds an immutable `Logger` dependency is acceptable if the logger’s state is not mutated.

---

### 4. Deep Cloning

#### Definitions

**Core Definition**
Deep cloning is a cloning strategy where every nested object in the object graph is also cloned, resulting in a fully independent copy with no shared mutable state.

**Technical Definition**
Deep cloning is not automatic in PHP. It is achieved either by implementing `__clone()` at every level of the object graph (manually cloning each nested object), or by using `serialize()`/`unserialize()` to traverse and recreate the entire graph. The `__clone()` approach respects class-specific cloning logic but requires coordination at every level. The serialization approach is automatic but can fail for resources, closures, and certain object types.

**Beginner-Friendly Explanation**
A deep clone is a photocopy where every sticky note is also photocopied onto fresh paper. The copy shares nothing mutable with the original—you can write on any sticky note without affecting the original.

#### Purposes

- To create truly independent copies of complex object graphs where nested objects must not be shared.
- To prevent subtle bugs where modifying a clone unexpectedly mutates the original.
- To support undo/redo, snapshotting, and speculative execution in applications.

#### Syntax Rules and Structure

**General Syntax (Manual `__clone` Approach)**
```php
public function __clone(): void {
    $this->child = clone $this->child;
    $this->collection = array_map(fn($item) => clone $item, $this->collection);
}
```

**General Syntax (Serialization Approach)**
```php
$deepCopy = unserialize(serialize($original));
```

**Component Breakdown**

| Approach | Mechanism | Trade-offs |
|----------|-----------|------------|
| `__clone()` chain | Each class clones its nested objects. | Respects custom logic; requires implementation at every level. |
| `serialize`/`unserialize` | Serializes the graph to a string, then reconstructs it. | Automatic; slower (≈4× per operation in benchmarks); fails on resources and closures. |

**Syntax Rules**

- **Manual approach**: Every class in the graph that holds mutable objects must implement `__clone()` and clone its children. Subclasses must call `parent::__clone()`.
- **Serialization approach**: Only works for serializable objects; `__wakeup()` runs on the reconstructed objects.

**Constraints and Limitations**

- **Manual approach**: Easy to forget a nested object when the class evolves (e.g., adding a new object property without updating `__clone()`).
- **Serialization approach**: Cannot handle resources (file handles, database connections); can be significantly slower; may trigger `__wakeup()` unexpectedly.
- **Circular references**: Manual cloning requires care to avoid infinite recursion; serialization handles circular references naturally.

#### Annotated Code Examples

**Example 1: Manual Deep Clone with `__clone()`**

```php
<?php
class Engine {
    public function __construct(public int $horsepower) {}
}

class Car {
    public Engine $engine;
    public function __construct(int $hp) {
        $this->engine = new Engine($hp);
    }

    public function __clone(): void {
        // Step 1: Clone the nested object
        $this->engine = clone $this->engine;
    }
}

$original = new Car(300);
$copy = clone $original;

$copy->engine->horsepower = 500;

echo "Original HP: {$original->engine->horsepower}\n";
echo "Copy HP:     {$copy->engine->horsepower}\n";
?>
```

**Expected Output:**
```
Original HP: 300
Copy HP:     500
```

**Why This Output:**
The `__clone()` method in `Car` replaces the shared `Engine` handle with a cloned `Engine`. The copy now has its own independent engine. Changing horsepower through the copy does not affect the original.

---

**Example 2: Deep Clone via `serialize`/`unserialize`**

```php
<?php
class Engine {
    public function __construct(public int $horsepower) {}
}

class Car {
    public Engine $engine;
    public function __construct(int $hp) {
        $this->engine = new Engine($hp);
    }
}

$original = new Car(300);
$copy = unserialize(serialize($original));

$copy->engine->horsepower = 500;

echo "Original HP: {$original->engine->horsepower}\n";
echo "Copy HP:     {$copy->engine->horsepower}\n";
?>
```

**Expected Output:**
```
Original HP: 300
Copy HP:     500
```

**Why This Output:**
Serialization traverses the entire object graph, converting it to a string; unserialization reconstructs an entirely new graph. The copy’s `Engine` is a different instance from the original’s, so mutations are isolated. Note that this bypasses any `__clone()` logic and invokes `__wakeup()` if defined.

#### Real-World Cases

- **E-Commerce Carts**: A `Cart` containing mutable `CartItem` objects must be deep-cloned when creating a saved-for-later copy; otherwise, editing the saved copy would mutate the active cart.
- **Document Editing**: An undo stack stores deep clones of a `Document` object graph so that reverting does not suffer from shared-state corruption.
- **Game State Snapshots**: A game engine deep-clones the entire `GameState` graph at checkpoints, ensuring that subsequent gameplay does not retroactively alter the snapshot.

---

### 5. Clone Best Practices

#### Definitions

**Core Definition**
Clone best practices are guidelines for when to use shallow cloning, when to implement deep cloning via `__clone()`, and how to integrate cloning with modern PHP features like `readonly` properties and immutable value objects.

**Technical Definition**
Best practices recommend preferring immutable value objects (e.g., `final readonly` classes) over mutable objects that require cloning, using shallow clones for objects containing only scalars or immutable dependencies, implementing `__clone()` at every level of a mutable object graph when deep cloning is necessary, leveraging PHP 8.3+ readonly reinitialization for “copy-with-changes,” and explicitly forbidding cloning of singletons and non-cloneable services.

**Beginner-Friendly Explanation**
The best practice is: **avoid needing clones by making objects immutable**. If you must clone, know that `clone` alone only makes a shallow copy—if your object holds other objects, you must handle them in `__clone()`.

#### Purposes

- To avoid shared-state bugs caused by accidental shallow cloning of mutable object graphs.
- To leverage immutability (readonly classes) where cloning would otherwise be error-prone.
- To make cloning behavior explicit, tested, and maintainable as classes evolve.
- To protect singletons and services from meaningless or harmful cloning.

#### Guidelines Summary

| Practice | Description |
|----------|-------------|
| **Prefer immutable value objects** | Use `final readonly class` for value objects (Money, Email, Coordinates); no cloning needed because “changes” return new instances. |
| **Assume `clone` is shallow** | Always verify whether nested objects are shared; implement `__clone()` if they must be independent. |
| **Deep-clone at every level** | If a parent class clones nested objects but the nested class does not clone *its* nested objects, sharing persists at deeper levels. |
| **Use PHP 8.3+ readonly reinitialization** | Inside `__clone()`, you can now re-assign `readonly` properties for ergonomic “copy-with-changes.” |
| **Forbid cloning of singletons/services** | Throw `LogicException` in `__clone()` for objects that should never be cloned. |
| **Re-establish resources** | File handles, database connections, and sockets must be re-opened in `__clone()` because they are shared by default. |
| **Prefer manual `__clone()` over `serialize` in hot paths** | Benchmarks show `__clone()` is significantly faster; use `serialize` for complex graphs where per-class implementation is impractical. |
| **Test clone independence** | Unit tests should explicitly verify that mutating the clone does not affect the original. |

**Syntax Rules**

- `__clone()` should be `public` and ideally declared with `: void`.
- Subclasses must call `parent::__clone()` when overriding.
- For singletons: `public function __clone() { throw new LogicException("Cannot clone singleton"); }`.

**Constraints and Limitations**

- **Readonly before PHP 8.3**: Cannot re-initialize readonly properties inside `__clone()`.
- **Serialization limits**: Cannot clone resources or closures reliably; may trigger `__wakeup()` with side effects.
- **Performance**: Deep cloning large graphs is inherently more expensive than shallow copying; design to minimize the need for it.

#### Annotated Code Examples

**Example 1: Best-Practice “Copy-with-Changes” Using PHP 8.3 Readonly Reinitialization**

```php
<?php
declare(strict_types=1);

class LogEntry {
    public function __construct(
        public readonly string $message,
        public readonly DateTimeImmutable $timestamp,
    ) {}

    public function __clone(): void {
        // PHP 8.3+: re-initialize a readonly property on the clone
        $this->timestamp = new DateTimeImmutable();
    }
}

$original = new LogEntry(
    "PHP Sussex is great",
    new DateTimeImmutable("2024-01-01 10:00:00")
);

$copy = clone $original;

echo "Original: {$original->timestamp->format('Y-m-d H:i:s')}\n";
echo "Copy:     {$copy->timestamp->format('Y-m-d H:i:s')}\n";
?>
```

**Expected Output:** *(Illustrative; copy timestamp reflects the time of cloning)*
```
Original: 2024-01-01 10:00:00
Copy:     2026-10-22 12:34:56
```

**Why This Output:**
Prior to PHP 8.3, writing `$this->timestamp = ...` inside `__clone()` on a `readonly` property threw an `Error`. As of PHP 8.3.0, the engine permits re-initialization of readonly properties during cloning. The original retains its original timestamp; the clone receives a fresh `DateTimeImmutable`, demonstrating the “copy-with-changes” pattern.

---

**Example 2: Forbidding Cloning of a Singleton**

```php
<?php
declare(strict_types=1);

final class DatabaseConnection {
    private static ?self $instance = null;

    private function __construct() {}

    public static function getInstance(): self {
        return self::$instance ??= new self();
    }

    public function __clone(): void {
        throw new LogicException("Cannot clone a singleton");
    }
}

$conn = DatabaseConnection::getInstance();

try {
    $copy = clone $conn;
} catch (LogicException $e) {
    echo "Cloning blocked: " . $e->getMessage() . "\n";
}
?>
```

**Expected Output:**
```
Cloning blocked: Cannot clone a singleton
```

**Why This Output:**
The `__clone()` method throws a `LogicException`, preventing any attempt to clone the singleton. This enforces the singleton invariant at the language level.

#### Real-World Cases

- **Immutable DTOs in Laravel/Symfony**: Modern frameworks encourage `final readonly class` for request/response DTOs, eliminating the need for cloning and its associated pitfalls.
- **Doctrine Entities**: Entities are mutable and typically should not be cloned without careful `__clone()` implementation to handle collections and associations; many teams forbid cloning entities altogether.
- **Configuration Objects**: Symfony’s `ParameterBag` implements `__clone()` to deep-clone nested arrays, ensuring that cloned configurations do not share mutable state.

---

## References

**PHP Manual – Object Cloning** - https://www.php.net/manual/en/language.oop5.cloning.php

**PHP Language Specification – Classes (`__clone`)** - https://github.com/php/php-langspec/blob/master/spec/14-classes.md

**PHP Manual – Magic Methods (`__clone`)** - https://www.php.net/manual/en/language.oop5.magic.php#object.clone

**Exakat – Should Deep Clone** - https://exakat.readthedocs.io/en/latest/Reference/Rules/Classes/ShouldDeepClone.html

**PHP RFC: Readonly Amendments (PHP 8.3 clone reinitialization)** - https://wiki.php.net/rfc/readonly_amendments

**Derick Rethans – What’s New in PHP 8.[1234]? (Readonly Amendments and Clone)** - https://derickrethans.nl/talks/php-phpday24.pdf

**Symfony DeepCloner (performance comparison with serialize/unserialize)** - https://symfony.com/blog/new-in-symfony-8-1-deep-cloner

**Yiisoft – Immutability (when shallow cloning suffices)** - https://yiisoft.github.io/docs/guide/immutability/

**PHP Bug #66116 – Cloning object leaving resource reference** - https://bugs.php.net/bug.php?id=66116