This cheat sheet provides a research-backed overview of PHP object comparison, covering the mechanics of the `==` and `===` operators, the subtle rules governing object equality and identity, and modern best practices for writing predictable comparison logic.

---

## Topic Overview

### Definitions

**Core Definition**
Object comparison in PHP is the process of evaluating two object variables to determine whether they represent the same value (equality) or the exact same instance in memory (identity), using the `==` and `===` operators respectively.

**Technical Definition**
PHP provides two primary comparison operators for objects. The comparison operator (`==`) evaluates two object instances as equal if they are instances of the same class and all their corresponding attributes and values are equal (with values themselves compared using `==`). The identity operator (`===`) evaluates two object variables as identical if and only if they refer to the same instance of the same class. Object comparison with `==` is recursive: if a property contains an object, that nested object is compared with the corresponding property of the other object using the same rules, which can lead to recursion errors for circular references.

**Beginner-Friendly Explanation**
Think of two identical twins. With `==` (equality), you are asking: "Do these two people look the same, have the same name, and the same birthday?" If everything matches, `==` returns `true`—even though they are two different people. With `===` (identity), you are asking: "Are these two variables pointing to the exact same person?" This is only `true` if both variables refer to the one and only instance of that person in memory.

---

### Key Characteristics

- **Two Distinct Operators**: `==` checks for structural equality (same class and same property values); `===` checks for referential identity (same memory instance).
- **Recursive Equality**: `==` recursively compares nested objects within properties, which can cause fatal errors with circular references.
- **Class Sensitivity**: Both operators require the objects to be instances of the **same class**; instances of different classes are never equal or identical, even if they share identical property names and values.
- **`==` Uses Loose Comparison for Values**: Property values are compared using `==`, meaning type coercion can occur (e.g., `"1"` and `1` may be considered equal inside object properties).
- **No `__equals` Magic Method**: Unlike Java, PHP does not have a built-in `__equals()` magic method for customizing `==` behavior. Custom comparison must be implemented via a userland method.
- **Extensions Can Override `==`**: Internal classes (e.g., `DateTime`, `PDOStatement`) can define their own rules for `==` comparison.

---

### Prerequisites

- **Basic Object-Oriented PHP**: Understanding of classes, instances, properties, and object handles.
- **PHP 7.2+ (Recommended)**: For `spl_object_id()` as a more efficient alternative to `spl_object_hash()`.
- **Awareness of PHP's Type Juggling**: Understanding how loose comparison (`==`) coerces types, as this affects object property comparison.

---

### Related Programming Areas

- **Unit Testing (PHPUnit)**: `assertEquals()` uses loose comparison recursively, while `assertSame()` checks identity; `assertObjectEquals()` allows custom equality logic via a method.
- **Value Objects and Domain-Driven Design (DDD)**: Objects representing values (Money, Email) often need custom equality logic beyond PHP's default recursive `==`.
- **Identity Maps and Caching**: Identity comparison (`===`) and `spl_object_id()` are used to track unique object instances in caches and repositories.
- **Collections and Sets**: Determining whether an object is already in a collection requires choosing between equality (`==`) and identity (`===`) based on the collection's semantics.

---

## Core Concepts / Features

---

### 1. `==` Operator (Equality)

#### Definitions

**Core Definition**
The `==` operator checks whether two object variables are **equal**—meaning they are instances of the same class and have identical property values.

**Technical Definition**
When using `==`, two object instances are considered equal if they have the same attributes and values (values are compared with `==`), and are instances of the same class. The comparison is recursive: nested objects are compared using the same rules, and comparison stops at the first unequal property found. Internal classes may define their own rules for `==` comparison.

**Beginner-Friendly Explanation**
`==` asks: "Do these two objects look exactly the same on the inside?" If two `Person` objects both have `name = "John"` and `age = 30`, `==` returns `true`—even if they are two completely separate objects in memory.

#### Purposes

- To check whether two objects represent the same logical value, regardless of whether they are the same instance.
- To compare objects in contexts where value equality is the meaningful semantic (e.g., comparing two `Money` objects with the same amount and currency).
- To support testing and validation where the structural state of an object is what matters.

#### Syntax Rules and Structure

**General Syntax**
```php
$object1 == $object2; // returns bool
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `==` | The equality operator. |
| `$object1`, `$object2` | The two object expressions to compare. |
| Result | `true` if both are instances of the same class and all corresponding properties are equal; `false` otherwise. |

**Syntax Rules**

- Both operands must be objects; comparing an object to a non-object returns `false`.
- The objects must be instances of the **same class**; different classes are never equal.
- Property values are compared using `==` (loose comparison), which can lead to type coercion.
- Comparison is recursive for nested objects and stops at the first unequal property.

**Constraints and Limitations**

- **Circular references cause fatal errors**: If object A holds a reference to object B, and B holds a reference back to A, recursive `==` comparison can throw a "Nesting level too deep" fatal error.
- **No custom equality hook**: PHP does not provide a `__equals()` magic method; custom equality logic cannot be injected into the `==` operator.
- **Loose value comparison**: Property values are compared with `==`, which may produce unexpected results (e.g., `"1"` equals `1`).

#### Annotated Code Examples

**Example 1: Equality of Two Separate Instances**

```php
<?php
class Flag {
    public $flag;
    public function __construct($flag = true) {
        $this->flag = $flag;
    }
}

$o = new Flag(); // instance 1
$p = new Flag(); // instance 2 (separate)

// Step 1: Compare two distinct instances with identical properties
var_dump($o == $p); // bool(true)
?>
```

**Expected Output:**
```
bool(true)
```

**Why This Output:**
`$o` and `$p` are two separate instances of the same `Flag` class, but both have `$flag = true`. The `==` operator checks that they are the same class and that all properties are equal. Because both conditions are satisfied, the result is `true`.

**Step-by-Step Setup:**
1.  Save as `equality.php`.
2.  Run `php equality.php`.
3.  The output confirms structural equality.

---

**Example 2: Equality Fails for Different Classes**

```php
<?php
class Flag {
    public $flag;
    public function __construct($flag = true) {
        $this->flag = $flag;
    }
}
class OtherFlag {
    public $flag;
    public function __construct($flag = true) {
        $this->flag = $flag;
    }
}

$o = new Flag();
$r = new OtherFlag();

// Step 1: Objects of different classes are never equal
var_dump($o == $r); // bool(false)
?>
```

**Expected Output:**
```
bool(false)
```

**Why This Output:**
Even though `Flag` and `OtherFlag` have identical property structures and values, they are **different classes**. The `==` operator requires the objects to be instances of the same class, so the comparison returns `false`.

---

**Example 3: Equality with Loose Value Comparison**

```php
<?php
class NumberHolder {
    public $value;
    public function __construct($value) {
        $this->value = $value;
    }
}

$intHolder = new NumberHolder(1);       // int 1
$stringHolder = new NumberHolder("1");  // string "1"

// Step 1: Property values are compared with ==
var_dump($intHolder == $stringHolder); // bool(true)
?>
```

**Expected Output:**
```
bool(true)
```

**Why This Output:**
Both objects are instances of the same class, and their `value` properties are compared using `==`. Because PHP's loose comparison considers `1 == "1"` to be `true`, the objects are deemed equal—even though the property types differ.

#### Real-World Cases

- **Value Object Comparison**: Comparing two `Color` objects to check if they represent the same RGB values, regardless of whether they are the same instance.
- **Form Data Validation**: Comparing a newly submitted `Address` object with a stored `Address` object to detect whether the user has changed their address.
- **Caching**: Checking whether a cached result object is "equal" to a newly computed result object to avoid redundant recomputation.

---

### 2. `===` Operator (Identity)

#### Definitions

**Core Definition**
The `===` operator checks whether two object variables are **identical**—meaning they refer to the exact same instance in memory.

**Technical Definition**
When using the identity operator (`===`), object variables are identical if and only if they refer to the same instance of the same class. Identity comparison is a direct check of the underlying object handle; it does not inspect properties or recurse into nested structures.

**Beginner-Friendly Explanation**
`===` asks: "Are these two variables pointing to the exact same object?" If `$a` and `$b` both hold a handle to the one and only `User` object created at a specific moment, `$a === $b` is `true`. If `$a` and `$b` point to two different `User` objects—even if their properties are identical—`===` is `false`.

#### Purposes

- To determine whether two variables reference the exact same object instance.
- To implement identity-based caching, registries, and identity maps.
- To distinguish between two separate objects that happen to have the same value (e.g., two separate `Order` objects for the same customer).
- To enforce singleton or shared-service semantics.

#### Syntax Rules and Structure

**General Syntax**
```php
$object1 === $object2; // returns bool
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `===` | The identity operator. |
| `$object1`, `$object2` | The two object expressions to compare. |
| Result | `true` if both refer to the exact same instance of the same class; `false` otherwise. |

**Syntax Rules**

- Both operands must be objects; comparing an object to a non-object returns `false`.
- No property comparison occurs; only the object handle is checked.
- There is no recursion or property traversal, so circular references do not cause errors.

**Constraints and Limitations**

- **Cannot check value equality**: `===` tells you nothing about whether two distinct objects have the same properties.
- **Not suitable for value-object comparison**: Two separate `Money` objects with identical amounts are `===`-different, even though they are logically equal.

#### Annotated Code Examples

**Example 1: Identity of the Same Instance**

```php
<?php
class Flag {
    public $flag;
    public function __construct($flag = true) {
        $this->flag = $flag;
    }
}

$o = new Flag();
$q = $o; // $q references the same instance as $o

// Step 1: Compare a variable to another handle to the same instance
var_dump($o === $q); // bool(true)
?>
```

**Expected Output:**
```
bool(true)
```

**Why This Output:**
`$q` was assigned the value of `$o`, which is an object handle pointing to the same instance. The `===` operator checks whether both operands refer to the same instance and returns `true`.

---

**Example 2: Identity Fails for Separate Instances**

```php
<?php
class Flag {
    public $flag;
    public function __construct($flag = true) {
        $this->flag = $flag;
    }
}

$o = new Flag();
$p = new Flag(); // separate instance

// Step 1: Compare two distinct instances
var_dump($o === $p); // bool(false)
?>
```

**Expected Output:**
```
bool(false)
```

**Why This Output:**
`$o` and `$p` are two separate instances of `Flag`, even though they have identical properties. The `===` operator checks referential identity, not structural equality, so the result is `false`.

#### Real-World Cases

- **Dependency Injection Containers**: Checking whether a service retrieved from the container is the exact same singleton instance that was registered.
- **Entity Identity**: In Doctrine, two entity objects loaded from the database are considered identical (same entity) based on their identity map, not their property values; `===` checks are used to verify that the same managed instance is returned.
- **Object Caches**: An in-memory cache uses `===` to verify that the cached object is the exact instance that was stored, preventing stale or duplicated objects from being served.

---

### 3. Object Comparison (Comprehensive)

#### Definitions

**Core Definition**
Object comparison encompasses all techniques for determining relationships between objects, including equality (`==`), identity (`===`), custom comparison methods, and identity-hash functions.

**Technical Definition**
PHP's native comparison operators handle the most common cases, but comprehensive object comparison also involves identity-hash functions (`spl_object_hash()` and `spl_object_id()`) for explicit identity checks, custom userland methods (often named `equals()`) for domain-specific equality logic, and manual property-by-property comparison when full control is needed. The `spl_object_hash()` function returns a string hash, while `spl_object_id()` (PHP 7.2+) returns an integer; both uniquely identify an object instance during its lifetime.

**Beginner-Friendly Explanation**
Beyond `==` and `===`, PHP gives you other tools. `spl_object_hash()` and `spl_object_id()` are like fingerprint scanners for objects—they give you a unique identifier for each object instance. If you need custom rules (e.g., "two products are equal if their SKUs match, even if their names differ"), you write your own `equals()` method.

#### Purposes

- To provide explicit, readable identity checks using hash/id functions when `===` is not sufficiently expressive.
- To implement domain-specific equality logic that cannot be expressed by PHP's default recursive `==`.
- To compare objects safely in the presence of circular references (identity-based approaches avoid recursion).
- To integrate with testing frameworks that distinguish between value equality and identity.

#### Syntax Rules and Structure

**General Syntax (Identity Hash)**
```php
spl_object_hash($object): string
spl_object_id($object): int
```

**General Syntax (Custom Method)**
```php
public function equals(self $other): bool {
    return $this->property === $other->property;
}
```

**Component Breakdown**

| Technique | Description |
|-----------|-------------|
| `==` | Structural equality; recursive; can fail on circular references. |
| `===` | Referential identity; no recursion. |
| `spl_object_hash()` | Returns a unique string hash for an object instance. |
| `spl_object_id()` | Returns a unique integer ID for an object instance (PHP 7.2+). |
| Custom `equals()` | Userland method for domain-specific equality logic. |

**Syntax Rules**

- Identity hashes must be compared with `===` (not `==`), because the hash may be a numeric string that could be coerced under loose comparison.
- `spl_object_id()` is preferred over `spl_object_hash()` for performance and type safety (integer comparison).
- Custom `equals()` methods should be type-safe, symmetric, reflexive, and transparent about float comparison behavior.

**Constraints and Limitations**

- **Hash reuse**: Object hashes/IDs may be reused after an object is destroyed; they are only guaranteed unique among live objects.
- **No native `__equals`**: PHP does not provide a magic method for customizing `==`; custom equality must be invoked explicitly.
- **Recursion safety**: Custom `equals()` methods must guard against infinite recursion when objects contain circular references.

#### Annotated Code Examples

**Example 1: Identity Hash Comparison**

```php
<?php
$person1 = new stdClass();
$person2 = new stdClass();

// Step 1: Compare identity using spl_object_id (PHP 7.2+)
if (spl_object_id($person1) === spl_object_id($person2)) {
    echo "Same instance\n";
} else {
    echo "Different instances\n"; // Outputs: Different instances
}

// Step 2: Compare with spl_object_hash (string-based)
if (spl_object_hash($person1) === spl_object_hash($person2)) {
    echo "Same instance (hash)\n";
} else {
    echo "Different instances (hash)\n"; // Outputs: Different instances (hash)
}
?>
```

**Expected Output:**
```
Different instances
Different instances (hash)
```

**Why This Output:**
`spl_object_id()` and `spl_object_hash()` return unique identifiers for each object instance. Since `$person1` and `$person2` are separate instances, their IDs and hashes differ, and the strict `===` comparison returns `false`.

---

**Example 2: Custom Equality Method**

```php
<?php
class Product {
    public function __construct(
        public string $sku,
        public string $name,
        public float $price
    ) {}

    // Step 1: Define domain-specific equality based on SKU only
    public function equals(self $other): bool {
        return $this->sku === $other->sku;
    }
}

$productA = new Product("SKU-001", "Widget", 9.99);
$productB = new Product("SKU-001", "Gadget", 19.99);
$productC = new Product("SKU-002", "Widget", 9.99);

// Step 2: Compare using the custom method
var_dump($productA->equals($productB)); // bool(true) — same SKU
var_dump($productA->equals($productC)); // bool(false) — different SKU

// Step 3: Compare with native == (all properties must match)
var_dump($productA == $productB); // bool(false) — names and prices differ
?>
```

**Expected Output:**
```
bool(true)
bool(false)
bool(false)
```

**Why This Output:**
The `equals()` method defines equality solely based on the `sku` property, so `$productA` and `$productB` are equal under this custom logic. The native `==` operator, however, compares all properties and finds mismatches in `name` and `price`, returning `false`.

#### Real-World Cases

- **Entity Identity Maps**: ORMs like Doctrine use identity maps keyed by `spl_object_id()` (or internal handles) to ensure that the same database row is always represented by the same object instance within a request.
- **Domain-Specific Equality**: An `EmailAddress` value object implements `equals()` to compare the normalized form (e.g., case-insensitive domain) rather than raw string equality.
- **Testing**: PHPUnit's `assertSame()` uses `===`, `assertEquals()` uses recursive `==`, and `assertObjectEquals()` invokes a custom `equals()` method, giving testers full control over comparison semantics.

---

### 4. Object Comparison Best Practices

#### Definitions

**Core Definition**
Object comparison best practices are guidelines for choosing the right comparison operator or technique based on whether value equality or referential identity is the intended semantic, and for implementing custom comparison logic safely and predictably.

**Technical Definition**
Best practices recommend using `===` for identity checks (preferring it over `==` when identity is intended), using `==` only for value objects with well-understood property semantics, implementing explicit `equals()` methods for domain-specific equality that cannot be expressed by native comparison, using `spl_object_id()` for identity-map keys, avoiding `==` on objects with circular references, and aligning comparison semantics with testing framework expectations (`assertSame` vs. `assertEquals` vs. `assertObjectEquals`).

**Beginner-Friendly Explanation**
The golden rule is: **if you want to know whether two variables point to the same object, use `===`. If you want to know whether two objects represent the same value, use `==`—but be careful, because `==` can recurse and might not match your domain logic.** When in doubt, write an explicit `equals()` method so everyone knows exactly what "equal" means for your class.

#### Purposes

- To prevent subtle bugs caused by using `==` when identity was intended (or vice versa).
- To make comparison semantics explicit and testable, especially for value objects.
- To avoid fatal errors from recursive `==` comparison of circular object graphs.
- To ensure consistency between production code and test assertions.

#### Guidelines Summary

| Practice | Description |
|----------|-------------|
| **Use `===` for identity** | When you need to know if two variables point to the same object, always use `===` (or `spl_object_id()` for explicit ID checks). |
| **Prefer `spl_object_id()` over `spl_object_hash()`** | `spl_object_id()` returns an integer, is more efficient, and avoids string-comparison pitfalls. |
| **Use `==` cautiously for value objects** | `==` is only safe when all properties are scalars or immutable objects and there are no circular references. |
| **Implement `equals()` for custom logic** | When domain equality differs from structural equality, provide an explicit `equals(self $other): bool` method. |
| **Guard against circular references** | Identity-based comparison (`===`, `spl_object_id()`) is immune to recursion; `==` is not. |
| **Align with PHPUnit semantics** | Use `assertSame()` for identity, `assertEquals()` for loose value equality, and `assertObjectEquals()` for custom `equals()` logic. |
| **Avoid `==` for mixed-type properties** | Because `==` uses loose value comparison for properties, type-coercion surprises can occur (e.g., `1 == "1"`). |

**Syntax Rules**

- When comparing identity hashes, always use `===` (or `!==`), never `==`, because the hash may be a numeric string.
- Custom `equals()` methods should be `public`, return `bool`, and type-hint the parameter as `self` (or the relevant interface).
- If a class contains float properties, document the comparison precision or use a delta-based comparison in `equals()`.

**Constraints and Limitations**

- **Hash reuse**: Identity hashes are only unique among live objects; a destroyed object's hash may be reused.
- **No operator overloading**: PHP does not allow overloading `==`; custom equality must be invoked explicitly.
- **PHPUnit's `assertEquals()` is recursive**: It uses `==` internally and can fail on circular references, just like the native operator.

#### Annotated Code Examples

**Example 1: Choosing the Right Comparison Technique**

```php
<?php
class Money {
    public function __construct(
        public int $amount,
        public string $currency
    ) {}

    public function equals(self $other): bool {
        return $this->amount === $other->amount
            && $this->currency === $other->currency;
    }
}

$price = new Money(1000, "USD");
$samePrice = new Money(1000, "USD");
$alias = $price;

// Step 1: Identity — is $alias the same instance as $price?
var_dump($price === $alias); // bool(true)
var_dump($price === $samePrice); // bool(false)

// Step 2: Native equality — same class, same property values
var_dump($price == $samePrice); // bool(true)

// Step 3: Custom equality — explicit, domain-aware
var_dump($price->equals($samePrice)); // bool(true)

// Step 4: Identity hash comparison
var_dump(spl_object_id($price) === spl_object_id($samePrice)); // bool(false)
?>
```

**Expected Output:**
```
bool(true)
bool(false)
bool(true)
bool(true)
bool(false)
```

**Why This Output:**
- `$price === $alias` is `true` because both refer to the same instance.
- `$price === $samePrice` is `false` because they are separate instances.
- `$price == $samePrice` is `true` because the same class and identical property values satisfy PHP's recursive equality.
- `$price->equals($samePrice)` is `true` because the custom method explicitly checks amount and currency.
- `spl_object_id()` comparison is `false` because the IDs of separate instances differ.

#### Real-World Cases

- **Laravel Collections**: `contains()` uses loose `==` by default, which can lead to surprising results for value objects; best practice is to use `containsStrict()` (which uses `===`) or provide a custom callback for domain equality.
- **Doctrine Identity Map**: Uses `spl_object_id()` to track managed entities, ensuring that querying the same row twice returns the same object instance (`===`-identical) within a unit of work.
- **PHPUnit Test Suites**: A test that expects two `DateTimeImmutable` objects representing the same moment should use `assertEquals()` (which compares values via `==`), while a test that expects the exact same instance should use `assertSame()`.

---

## References

**PHP Manual – Comparing Objects** - https://www.php.net/manual/en/language.oop5.object-comparison.php

**PHP Manual – Object Comparison (Alternative View)** - https://static.php.net/manual/it/language.oop5.object-comparison.php

**PHP Manual – `spl_object_hash`** - https://www.php.net/manual/en/function.spl-object-hash.php

**PHP Manual – `spl_object_id`** - https://www.php.net/manual/en/function.spl-object-id.php

**PHP RFC – `spl_object_id()`** - https://wiki.php.net/rfc/spl_object_id

**PHPUnit – Assertions Reference (`assertEquals`, `assertSame`, `assertObjectEquals`)** - https://docs.phpunit.de/en/10.5/assertions.html

**PHP-Core – Equatable Trait Best Practices** - https://github.com/mossy2100/PHP-Core/blob/1aa7a80ea9b9b07a7d7c3e4944f08f1ebfe0b041/docs/Traits/Comparison/Equatable.md

**Packagist – Parity (Deep Comparison Library)** - https://packagist.org/packages/schmittjoh/php-collection

**PHP.Watch – `spl_object_id()` (PHP 7.2)** - https://php.watch/versions/7.2/spl_object_id