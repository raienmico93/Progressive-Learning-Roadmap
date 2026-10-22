Here is a comprehensive research cheat sheet on **PHP Static Members**, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Static Members

### Definitions

**Core Definition**
**Static members** are properties and methods that belong to the **class itself** rather than to any individual object instance. They are shared by all instances of the class and can be accessed without creating an object.

**Technical Definition**
In PHP, static members are declared with the `static` keyword. Static properties are stored once per class (not once per object) and are accessed using the scope resolution operator (`::`) — either via `ClassName::$property`, `self::$property`, or `static::$property`. Static methods are functions bound to the class, not to an object; they have no `$this` and cannot access non-static members directly. Since PHP 5.3.0, the `static::` keyword enables **late static binding**, which resolves to the class that was actually called at runtime rather than the class where the method is defined. Since PHP 8.0.0, `static` can also be used as a return type, requiring the returned value to be an instance of the called class.

**Beginner-Friendly Explanation**
Static members are like **tools attached to the blueprint (class)** rather than to each individual house (object) built from it. Every house shares the same blueprint tools. You can use a static method without building a house first — just call it on the class name. Static properties are like a **shared whiteboard** in the blueprint room: everyone who looks at the blueprint sees the same whiteboard. Because they are shared, you must be careful — changing a static property affects all objects of that class.

---

### Key Characteristics

- **Class-Level Ownership**: Static members belong to the class, not to instances. They exist once per class (per inherited class in the hierarchy).
- **No Instantiation Required**: Static methods and properties can be accessed directly via `ClassName::member` without `new`.
- **No `$this` in Static Methods**: Static methods cannot use `$this` and cannot directly access non-static properties or methods.
- **Shared State**: Static properties are shared by all instances; modifying one affects the class as a whole (and all its objects).
- **Late Static Binding (PHP 5.3+)**: `static::` resolves to the runtime called class, enabling inheritance-aware static behaviour.
- **Static Return Type (PHP 8.0+)**: Methods can declare `: static`, requiring the return value to be an instance of the called class.
- **PHP 8.5 Refinement**: In `final` classes, overriding a `static` return type with `self` or the class’s own type is now permitted because they are equivalent in a non-extendable class.

---

### Prerequisites

- **PHP 5.3+** for late static binding; **PHP 8.0+** for the `static` return type; **PHP 8.5** for the final-class return type refinement.
- **Basic Class and Object Syntax**: Understanding of `class`, `new`, `$this`, visibility, and inheritance.
- **Awareness of Scope**: Static members are accessed with `::`, not `->`.

---

### Related Programming Areas

- **Inheritance**: Late static binding is essential for static methods in class hierarchies.
- **Design Patterns**: Singleton, Factory Method, and utility classes often use static members.
- **Dependency Injection**: Static methods hinder testability because dependencies cannot be injected.
- **Static Analysis**: PHPStan and Psalm track static property types and `static` return types.

---

## Core Concepts / Features

---

### 1. Static Properties

#### Definitions

**Core Definition**
A static property is a variable that belongs to the class itself and is shared by all instances of that class.

**Technical Definition**
Static properties are declared with the `static` keyword: `public static int $count = 0;`. They are stored once per class (and separately in each subclass that redeclares them). Access is via `ClassName::$property`, `self::$property`, or `static::$property`. Static properties may be typed (PHP 7.4+) and, since PHP 8.3, may be `readonly`.

**Beginner-Friendly Explanation**
A static property is a **shared storage box** attached to the blueprint. Every object made from the blueprint sees the same box. If one object changes what is in the box, all other objects see the change.

#### Purposes

- To share data across all instances of a class (e.g., instance counters, global configuration).
- To implement class-level caches or registries.
- To store singleton instances.

#### Syntax Rules and Structure

**General Syntax**
```php
[visibility] static [type] $propertyName [= constantExpression];
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `visibility` | `public`, `protected`, or `private` (defaults to `public`). |
| `static` | Makes the property class-level. |
| `type` | Optional type declaration (PHP 7.4+). |
| `$name` | Property name. |
| `= value` | Optional constant-expression default. |

**Constraints and Limitations**

- **Not accessed via `->`**: `$object->staticProperty` triggers a notice and does not access the static value.
- **Shared mutable state**: Modifying a static property affects all code that reads it.
- **Readonly static (PHP 8.3+)**: Static properties may be `readonly`, preventing modification after initialization.

#### Annotated Code Examples

**Example 1: Instance Counter**

```php
<?php
class Counter
{
    public static int $count = 0;

    public function __construct()
    {
        self::$count++;
    }
}

new Counter();
new Counter();
new Counter();

echo Counter::$count . "\n"; // Outputs: 3
?>
```

**Expected Output:**
```
3
```

**Why This Output:**  
`$count` is static and shared. Each `new Counter()` increments the same static property via `self::$count`. The final count is `3`.

**Step-by-Step Setup:**
1. Save as `static_property.php`.
2. Run `php static_property.php`.
3. The output confirms the shared counter.

---

**Example 2: Static Properties in Inheritance (Distinct per Class)**

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
When a subclass redeclares a static property, it creates its own separate storage. `ParentClass::$name` and `ChildClass::$name` are independent.

#### Real-World Cases

- **Instance Counters**: Tracking how many objects of a class have been created.
- **Global Configuration**: `public static array $config = [];`
- **Singleton Storage**: `private static ?self $instance = null;`

---

### 2. Static Methods

#### Definitions

**Core Definition**
A static method is a function that belongs to the class and can be called without creating an object. It has no `$this` and operates only on static members or its parameters.

**Technical Definition**
Static methods are declared with `static` after visibility: `public static function methodName(): ReturnType`. They are invoked via `ClassName::method()`, `self::method()`, or `static::method()`. They cannot access non-static properties or call non-static methods directly.

**Beginner-Friendly Explanation**
A static method is an **action the blueprint itself can perform**. You do not need a house (object) to use it — you just ask the blueprint.

#### Purposes

- To provide utility functions that do not depend on object state.
- To implement named constructors and factory methods.
- To create singleton accessors.

#### Syntax Rules and Structure

**General Syntax**
```php
[visibility] static function methodName(Type $param): ReturnType
{
    // no $this
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `visibility` | `public`, `protected`, or `private`. |
| `static` | Makes the method class-level. |
| `function` | Keyword. |
| `$this` | Not available. |

**Constraints and Limitations**

- **No `$this`**: Cannot access instance properties or call instance methods.
- **Static methods are not polymorphic in the same way**: `self::` resolves to the defining class; `static::` enables late static binding.

#### Annotated Code Examples

**Example 1: Utility Method**

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

---

**Example 2: Static Factory Method**

```php
<?php
class User
{
    private function __construct(private string $name) {}

    public static function create(string $name): self
    {
        return new self($name);
    }

    public function getName(): string
    {
        return $this->name;
    }
}

$user = User::create('Alice');
echo $user->getName() . "\n"; // Outputs: Alice
?>
```

**Expected Output:**
```
Alice
```

**Why This Output:**  
The constructor is private, forcing creation through the static factory `create()`. The factory uses `new self()` to instantiate the class.

#### Real-World Cases

- **Named Constructors**: `User::fromArray($data): static`.
- **Utility Classes**: `Str::slug()`, `Arr::get()`.
- **Singletons**: `Logger::getInstance(): self`.

---

### 3. `static::` and Late Static Binding

#### Definitions

**Core Definition**
`static::` is a keyword that resolves to the class that was **actually called at runtime**, enabling inherited static methods to operate on the subclass rather than the parent class.

**Technical Definition**
Late static binding (PHP 5.3.0+) works by storing the class named in the last “non-forwarding call” — for static calls, this is the class explicitly named (usually to the left of `::`). `static::` is resolved using runtime information, not the class where the method is defined. `self::` and `__CLASS__` are resolved using the defining class.

**Beginner-Friendly Explanation**
`static::` means “**the class that was actually used to make this call**.” If a child class calls a parent’s static method, `static::` makes the method use the child class’s overridden methods and properties. `self::` would stick to the parent class instead.

#### Purposes

- To make static methods inheritance-aware so subclasses can override behaviour.
- To implement factory methods and named constructors that return the called class.
- To share static behaviour across a class hierarchy without hard-coding the parent class.

#### Syntax Rules and Structure

**General Syntax**
```php
static::method();
static::$property;
static::CONSTANT;
```

**Component Breakdown**

| Keyword | Resolves To |
|---------|-------------|
| `self::` | The class where the method is defined (compile-time). |
| `static::` | The class that was called at runtime (late static binding). |
| `parent::` | The immediate parent class. |

**Constraints and Limitations**

- **`static::` can only refer to static members** in static contexts.
- **Resolution stops at a fully resolved static call**: Unlike `self::` and `parent::`, `static::` does not forward calling information further up the hierarchy.
- **In non-static contexts**, the called class is the class of the object instance.

#### Annotated Code Examples

**Example 1: `self::` vs. `static::`**

```php
<?php
class A
{
    public static function who(): void
    {
        echo __CLASS__ . "\n";
    }

    public static function testSelf(): void
    {
        self::who();       // always A
    }

    public static function testStatic(): void
    {
        static::who();     // late static binding
    }
}

class B extends A
{
    public static function who(): void
    {
        echo __CLASS__ . "\n";
    }
}

B::testSelf();   // Outputs: A
B::testStatic(); // Outputs: B
?>
```

**Expected Output:**
```
A
B
```

**Why This Output:**  
`testSelf()` uses `self::who()`, which resolves to `A::who()` regardless of the calling class. `testStatic()` uses `static::who()`, which resolves to the called class (`B`), so `B::who()` runs.

---

**Example 2: Late Static Binding with Static Properties**

```php
<?php
class ParentClass
{
    protected static string $name = 'Parent';

    public static function getName(): string
    {
        return static::$name;
    }
}

class ChildClass extends ParentClass
{
    protected static string $name = 'Child';
}

echo ParentClass::getName() . "\n"; // Outputs: Parent
echo ChildClass::getName() . "\n";  // Outputs: Child
?>
```

**Expected Output:**
```
Parent
Child
```

**Why This Output:**  
`getName()` uses `static::$name`, which resolves to the called class’s static property. `ParentClass::getName()` returns `'Parent'`; `ChildClass::getName()` returns `'Child'`.

#### Real-World Cases

- **Active Record / ORM**: `Model::find($id)` uses `static` so it returns an instance of the called subclass.
- **Named Constructors**: `static::fromArray($data)` preserves the called class.
- **Counters in Hierarchies**: Each subclass can have its own static counter.

---

### 4. Static Return Type (PHP 8.0+) and Covariance

#### Definitions

**Core Definition**
The `static` return type requires a method to return an instance of the class on which the method was called (the runtime class), enabling inheritance-friendly factories and fluent interfaces.

**Technical Definition**
Introduced in PHP 8.0.0, `: static` is a return-only type. It is a subtype of `self` for variance purposes. In a `final` class (PHP 8.5+), an overriding method may declare `self` or the class’s own type instead of `static`, because they are equivalent in a non-extendable class.

**Beginner-Friendly Explanation**
`: static` means “**return an object of the class that was actually used to call this method**.” If a child class calls a parent’s method declared `: static`, it must get back a child object, not a parent object. This is essential for fluent APIs and named constructors.

#### Purposes

- To guarantee that factory methods and fluent methods return the called class.
- To support immutable “wither” methods that return a new instance of the same class.
- To express late static binding precisely in the type system.

#### Syntax Rules and Structure

**General Syntax**
```php
public function methodName(): static
{
    return new static();
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `: static` | Return-only type; value must be an instance of the called class. |
| `new static()` | Instantiates the called class (late static binding). |
| `return $this;` | Common in fluent methods. |

**Constraints and Limitations**

- **`static` cannot be used for parameters or properties** (return-only).
- **Covariance**: An overriding method may narrow `self` to `static`, but not the reverse.
- **PHP 8.5 refinement**: In a `final` class, `self` or the class’s own type may replace `static` as the return type because they are equivalent.

#### Annotated Code Examples

**Example 1: Fluent Interface with `static` Return Type**

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
Each fluent method returns `$this` with the `: static` return type, guaranteeing that chaining works correctly even in subclasses.

---

**Example 2: Named Constructor with `static` Return Type**

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
`create()` uses `new static()`, which resolves to the called class (`User`). The `: static` return type enforces that `User::create()` returns a `User`.

#### Real-World Cases

- **Immutable Value Objects**: `withName(string $name): static` clones the object and returns the new instance.
- **Fluent Builders**: Query builders, HTTP clients, and form builders.
- **Named Constructors**: `fromArray(array $data): static`.

---

### 5. Static Best Practices

#### Definitions

**Core Definition**
Best practices for static members are guidelines for using them judiciously — where they add value and where they introduce hidden coupling, testing difficulties, and global mutable state.

**Technical Definition**
The consensus is to **minimize static members** in application code. Use static methods only for pure utility functions that do not depend on object state, named constructors, or singletons (sparingly). Use static properties only for truly class-level immutable data (e.g., constants) or carefully controlled shared state. Prefer `static::` over `self::` for static member references when inheritance is expected, because `self::` breaks polymorphism by always resolving to the defining class.

**Beginner-Friendly Explanation**
Use static members like **power tools**: they are useful for specific jobs, but dangerous if overused. Do not use static properties as global variables. Do not use static methods just to avoid creating an object. If you need shared behaviour across subclasses, use `static::` so the right class is used.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Prefer `static::` for late binding** | When referencing static members that subclasses may override, use `static::` instead of `self::` to respect inheritance. |
| **Use `self::` for private members and constants** | `self::` is safe for private static members and constants because they cannot be overridden. |
| **Avoid mutable static properties** | Shared mutable state is essentially global state; it hides dependencies and complicates testing. |
| **Use static methods only for stateless utility** | If a method needs object state, it should not be static. |
| **Do not use `$this->staticProperty`** | Access static members with `::`, never with `->`. |
| **Prefer dependency injection over static factories** | Static factories and singletons make testing harder; inject dependencies when possible. |
| **Use `: static` for factories and fluent methods** | Ensures correct subclass return types. |

#### Annotated Code Examples

**Example 1: `self::` vs. `static::` in Inheritance**

```php
<?php
class Toy
{
    public static function status(): void
    {
        static::getStatus(); // late static binding
    }

    protected static function getStatus(): void
    {
        echo "Sticks are fun!\n";
    }
}

class Ball extends Toy
{
    protected static function getStatus(): void
    {
        echo "Balls are fun!\n";
    }
}

Ball::status(); // Outputs: Balls are fun!
?>
```

**Expected Output:**
```
Balls are fun!
```

**Why This Output:**  
`status()` uses `static::getStatus()`. When called as `Ball::status()`, `static` resolves to `Ball`, so `Ball::getStatus()` runs. If `self::getStatus()` were used, `Toy::getStatus()` would run and output “Sticks are fun!”.

---

**Example 2: Avoid Mutable Static State**

```php
<?php
// POOR: mutable static state (global)
class Counter
{
    public static int $count = 0;

    public static function increment(): void
    {
        self::$count++;
    }
}

Counter::increment();
Counter::increment();
echo Counter::$count . "\n"; // Outputs: 2

// GOOD: inject a stateful object
class CounterService
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

$counter = new CounterService();
$counter->increment();
$counter->increment();
echo $counter->getCount() . "\n"; // Outputs: 2
?>
```

**Expected Output:**
```
2
2
```

**Why This Output:**  
Both produce `2`, but the static version is global mutable state that any code can modify. The injected version is testable, isolated, and its dependencies are explicit.

#### Real-World Cases

- **Utility Classes**: `MathHelper::square()` — stateless, pure.
- **Singletons**: `Logger::getInstance()` — use sparingly and inject the instance where possible.
- **Framework Facades**: Laravel facades use static proxies to resolve services from the container; they are a deliberate framework pattern, not a general best practice.

---

## References

**PHP Manual – Static Keyword** - https://static.php.net/manual/en/language.oop5.static.php

**PHP Manual – Late Static Bindings** - https://static.php.net/manual/it/language.oop5.late-static-bindings.php

**SonarSource – Static members should be referenced with “static::”** - https://rules.sonarsource.com/php/tag/pitfall/rspec-2037/

**ZetCode – PHP static Keyword** - https://zetcode.cn/php/static-keyword/

**PHP Manual – Covariance and Contravariance** - https://static.php.net/manual/it/language.oop5.variance.php

**PHP Manual – Relative Class Types (static return type)** - https://static.php.net/manual/en/language.types.relative-class-types.php

**Stack Overflow – Late Static Bindings in PHP** - https://stackoverflow.com/revisions/fd4f62ab-fe31-4d84-ae62-52e5405ffa5f/view-source

**Stack Overflow – Best Practices for Static Members** - https://cloud.tencent.com.cn/developer/ask/sof/111023099

**TYPO3 – Static Methods, Static Classes, Utility Classes** - https://docs.typo3.org

**Stack Overflow – Should You Use Static Methods for Helpers?** - https://stackoverflow.com

**Psalm EO Rules – Forbids Static** - https://github.com/haspadar/psalm-eo-rules

**PHP 8.5 – Static Return Type Refinement in Final Classes** - https://static.php.net/manual/it/migration85.incompatible.php

**PHP RFC – Static Return Type** - https://wiki.php.net/rfc/static_return_type