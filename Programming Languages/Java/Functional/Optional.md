# Java Optional: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `Optional<T>` is a container object that may or may not contain a non-null value. It provides a type-level solution for representing optional values instead of using `null` references.

**Technical Definition:** "A container object which may or may not contain a non-null value. If a value is present, `isPresent()` returns `true`. If no value is present, the object is considered empty and `isPresent()` returns `false`. Additional methods that depend on the presence or absence of a contained value are provided, such as `orElse()` (returns a default value if no value is present) and `ifPresent()` (performs an action if a value is present). This is a value-based class; programmers should treat instances that are equal as interchangeable and should not use instances for synchronization, or unpredictable behavior may occur".

**Beginner-Friendly Explanation:** Think of `Optional` as a gift box. The box may contain a gift (a value), or it may be empty. Instead of checking "is this gift null?", you check "does this box have a gift inside?" The box itself is never null — it's either a box with something inside or an empty box. This forces you to explicitly handle both cases.

### Key Characteristics

- **Value-Based:** Instances of `Optional` are value-based; identity-sensitive operations (reference equality, identity hash code, synchronization) should be avoided.
- **Immutable:** `Optional` instances cannot be modified after creation.
- **Single-Use Container:** An `Optional` either contains a value or is empty; it cannot be reused for different values.
- **Return Type Focus:** "Optional is primarily intended for use as a method return type where there is a clear need to represent 'no result,' and where using null is likely to cause errors".
- **Non-Null Guarantee:** "A variable whose type is `Optional` should never itself be `null`; it should always point to an `Optional` instance".
- **Functional API:** Provides methods like `map`, `flatMap`, `filter`, and `ifPresent` for declarative value handling.

### Prerequisites

- Basic understanding of Java generics
- Familiarity with lambda expressions and functional interfaces
- Knowledge of the `java.util.function` package
- Understanding of null handling in Java

### Related Programming Areas

- **Null Safety:** `Optional` is Java's primary tool for avoiding `NullPointerException`.
- **Functional Programming:** `Optional` provides a monadic-style API for chaining operations.
- **Stream API:** `Optional.stream()` (Java 9+) bridges `Optional` and streams.
- **API Design:** `Optional` communicates to callers that a method may return no result.
- **Primitive Streams:** `OptionalInt`, `OptionalLong`, and `OptionalDouble` specialise `Optional` for primitive types.

### Core Concepts / Features

---

## 1. Optional

### Definitions

**Core Definition:** `Optional` is a final class in `java.util` that wraps a value that may or may not be present.

**Technical Definition:** "A container object which may or may not contain a non-null value". The class is `final` and provides static factory methods for creation and instance methods for value retrieval and transformation.

**Beginner-Friendly Explanation:** `Optional` is a wrapper that tells you "there might be a value here, or there might not be." Instead of returning `null` from a method (which is easy to forget to check), a method returns an `Optional`, forcing the caller to decide what to do if the value is absent.

### Purposes

- To provide a type-level solution for representing optional values instead of null references
- To reduce the occurrence of `NullPointerException` by making the absence of a value explicit
- To document in a method's signature that it may return no result
- To enable functional-style chaining of operations on possibly-absent values
- To avoid returning `null` from methods where the result may be absent

### Syntax Rules and Structure

**Complete General Syntax:**

```java
Optional<T> optional = Optional.of(value);          // Non-null value
Optional<T> optional = Optional.ofNullable(value);  // May be null
Optional<T> optional = Optional.empty();            // No value
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Optional<T>` | The generic container type. |
| `T` | The type of the contained value. |
| `Optional.of(value)` | Creates an Optional containing a non-null value. |
| `Optional.ofNullable(value)` | Creates an Optional that may be empty. |
| `Optional.empty()` | Creates an empty Optional. |

**Syntax Rules:**

- `Optional` is a `final` class and cannot be subclassed.
- `Optional` is value-based; identity operations should be avoided.
- An `Optional` variable should never be assigned `null`; use `Optional.empty()` instead.

**Constraints and Limitations:**

- `Optional` does not implement `Serializable`, making it unsuitable for fields in serializable classes.
- `Optional` adds heap overhead (approximately 16 bytes per instance).
- `Optional` is not intended for use as a method parameter or class field.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Creating and Using Optional**

```java
// File: OptionalBasicDemo.java
import java.util.Optional;

public class OptionalBasicDemo {
    public static void main(String[] args) {
        // Step 1: Create an Optional with a non-null value
        Optional<String> present = Optional.of("Hello");
        System.out.println("Is present: " + present.isPresent());  // true

        // Step 2: Create an Optional that may be empty
        String nullable = null;
        Optional<String> empty = Optional.ofNullable(nullable);
        System.out.println("Is present: " + empty.isPresent());    // false

        // Step 3: Create an explicitly empty Optional
        Optional<String> explicitlyEmpty = Optional.empty();
        System.out.println("Is present: " + explicitlyEmpty.isPresent()); // false

        // Step 4: Retrieve value with default
        String result = empty.orElse("Default Value");
        System.out.println("Result: " + result);  // Default Value

        // Step 5: Perform action if present
        present.ifPresent(value -> System.out.println("Value: " + value));  // Value: Hello
    }
}
```

**Expected Output:**
```
Is present: true
Is present: false
Is present: false
Result: Default Value
Value: Hello
```

**Step-by-Step Explanation:**

1. `Optional.of("Hello")` creates an `Optional` containing the non-null value `"Hello"`.
2. `Optional.ofNullable(nullable)` creates an empty `Optional` because the argument is `null`.
3. `Optional.empty()` creates an empty `Optional` directly.
4. `empty.orElse("Default Value")` returns the default because the `Optional` is empty.
5. `present.ifPresent(...)` executes the lambda because a value is present.

### Real-World Cases

- **Repository Methods:** `Optional<User> findById(Long id)` returns a user or empty if not found.
- **Configuration:** `Optional<String> getConfig(String key)` returns a config value if present.
- **Navigation:** `Optional<Address> getAddress()` on a `User` object that may not have an address.

---

## 2. Optional Creation

### 2.1 `Optional.of()`

#### Definitions

**Core Definition:** `Optional.of(T value)` creates an `Optional` containing the given non-null value.

**Technical Definition:** "Returns an `Optional` with the specified present non-null value". If the argument is `null`, a `NullPointerException` is thrown.

**Beginner-Friendly Explanation:** `Optional.of()` is used when you are certain the value is not null. It's like putting a gift into a box — the gift must exist.

#### Purposes

- To create an `Optional` from a known non-null value
- To enforce non-null contracts at creation time

#### Syntax Rules and Structure

```java
static <T> Optional<T> of(T value)
```

**Constraints and Limitations:**

- Throws `NullPointerException` if `value` is `null`.

#### Annotated Code Example

```java
Optional<String> opt = Optional.of("Hello");
System.out.println(opt.get());  // Hello

// Optional.of(null);  // Throws NullPointerException
```

### 2.2 `Optional.ofNullable()`

#### Definitions

**Core Definition:** `Optional.ofNullable(T value)` creates an `Optional` that is empty if the value is `null`, or contains the value otherwise.

**Technical Definition:** "Returns an `Optional` describing the given value, if non-null, otherwise returns an empty `Optional`".

**Beginner-Friendly Explanation:** `Optional.ofNullable()` is used when the value might be null. It's like wrapping a gift that may or may not be there.

#### Purposes

- To create an `Optional` from a value that may be null
- To safely wrap nullable values without throwing exceptions

#### Syntax Rules and Structure

```java
static <T> Optional<T> ofNullable(T value)
```

#### Annotated Code Example

```java
Optional<String> present = Optional.ofNullable("Hello");
System.out.println(present.isPresent());  // true

Optional<String> empty = Optional.ofNullable(null);
System.out.println(empty.isPresent());    // false
```

### 2.3 `Optional.empty()`

#### Definitions

**Core Definition:** `Optional.empty()` returns an empty `Optional` instance containing no value.

**Technical Definition:** "Returns an empty `Optional` instance".

**Beginner-Friendly Explanation:** `Optional.empty()` is an empty box — it explicitly represents the absence of a value.

#### Purposes

- To create an empty `Optional` explicitly
- To return "no result" from a method without using `null`

#### Syntax Rules and Structure

```java
static <T> Optional<T> empty()
```

#### Annotated Code Example

```java
Optional<String> empty = Optional.empty();
System.out.println(empty.isPresent());  // false
```

### Real-World Cases

- **`of`:** Used when a method has a guaranteed non-null result.
- **`ofNullable`:** Used when wrapping values from external sources that may be null.
- **`empty`:** Used to return "not found" from repository methods.

---

## 3. Optional Methods

### 3.1 `isPresent()`

#### Definitions

**Core Definition:** `isPresent()` returns `true` if a value is present, `false` otherwise.

**Technical Definition:** "If a value is present, returns `true`, otherwise `false`".

#### Syntax Rules and Structure

```java
boolean isPresent()
```

#### Annotated Code Example

```java
Optional<String> opt = Optional.of("Hello");
System.out.println(opt.isPresent());  // true
```

### 3.2 `isEmpty()` (Java 11+)

#### Definitions

**Core Definition:** `isEmpty()` returns `true` if no value is present, `false` otherwise.

**Technical Definition:** "If a value is not present, returns `true`, otherwise `false`".

#### Syntax Rules and Structure

```java
boolean isEmpty()
```

#### Annotated Code Example

```java
Optional<String> opt = Optional.empty();
System.out.println(opt.isEmpty());  // true
```

### 3.3 `get()`

#### Definitions

**Core Definition:** `get()` returns the value if present, otherwise throws `NoSuchElementException`.

**Technical Definition:** "If a value is present, returns the value, otherwise throws `NoSuchElementException`".

**Beginner-Friendly Explanation:** `get()` is dangerous — it should be avoided in favour of safer alternatives.

#### Syntax Rules and Structure

```java
T get()
```

**Constraints and Limitations:**

- Throws `NoSuchElementException` if the `Optional` is empty.
- Prefer `orElseThrow()` over `get()`.

#### Annotated Code Example

```java
Optional<String> opt = Optional.of("Hello");
System.out.println(opt.get());  // Hello

// Optional.empty().get();  // Throws NoSuchElementException
```

### 3.4 `orElse()`

#### Definitions

**Core Definition:** `orElse(T other)` returns the value if present, otherwise returns the default.

**Technical Definition:** "If a value is present, returns the value, otherwise returns `other`".

**Beginner-Friendly Explanation:** `orElse()` provides a fallback value.

#### Syntax Rules and Structure

```java
T orElse(T other)
```

**Constraints and Limitations:**

- `orElse()` always evaluates its argument, even when the `Optional` is present.

#### Annotated Code Example

```java
Optional<String> empty = Optional.empty();
String result = empty.orElse("Default");
System.out.println(result);  // Default
```

### 3.5 `orElseGet()`

#### Definitions

**Core Definition:** `orElseGet(Supplier<? extends T> supplier)` returns the value if present, otherwise invokes the supplier and returns its result.

**Technical Definition:** "If a value is present, returns the value, otherwise returns the result produced by the supplying function".

**Beginner-Friendly Explanation:** `orElseGet()` lazily computes a fallback only when needed.

#### Syntax Rules and Structure

```java
T orElseGet(Supplier<? extends T> supplier)
```

**Constraints and Limitations:**

- The supplier is only invoked when the `Optional` is empty.
- Use `orElseGet()` instead of `orElse()` when the fallback is expensive.

#### Annotated Code Example

```java
Optional<String> empty = Optional.empty();
String result = empty.orElseGet(() -> "Computed Default");
System.out.println(result);  // Computed Default
```

### 3.6 `orElseThrow()`

#### Definitions

**Core Definition:** `orElseThrow()` returns the value if present, otherwise throws `NoSuchElementException`.

**Technical Definition:** "If a value is present, returns the value, otherwise throws `NoSuchElementException`". An overload accepts a supplier for a custom exception.

#### Syntax Rules and Structure

```java
T orElseThrow()
T orElseThrow(Supplier<? extends X> exceptionSupplier)
```

#### Annotated Code Example

```java
Optional<String> empty = Optional.empty();
try {
    empty.orElseThrow(() -> new IllegalStateException("Missing value"));
} catch (IllegalStateException e) {
    System.out.println(e.getMessage());  // Missing value
}
```

### 3.7 `ifPresent()`

#### Definitions

**Core Definition:** `ifPresent(Consumer<? super T> consumer)` performs an action if a value is present.

**Technical Definition:** "If a value is present, performs the given action with the value, otherwise does nothing".

#### Syntax Rules and Structure

```java
void ifPresent(Consumer<? super T> consumer)
```

#### Annotated Code Example

```java
Optional<String> present = Optional.of("Hello");
present.ifPresent(v -> System.out.println("Value: " + v));  // Value: Hello
```

### 3.8 `ifPresentOrElse()` (Java 9+)

#### Definitions

**Core Definition:** `ifPresentOrElse()` performs one action if a value is present and another if absent.

**Technical Definition:** "If a value is present, performs the given action with the value, otherwise performs the given empty-based action".

#### Syntax Rules and Structure

```java
void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)
```

#### Annotated Code Example

```java
Optional<String> present = Optional.of("Hello");
present.ifPresentOrElse(
    v -> System.out.println("Present: " + v),
    () -> System.out.println("Empty")
);
// Output: Present: Hello

Optional<String> empty = Optional.empty();
empty.ifPresentOrElse(
    v -> System.out.println("Present: " + v),
    () -> System.out.println("Empty")
);
// Output: Empty
```

### 3.9 `map()`

#### Definitions

**Core Definition:** `map(Function<? super T, ? extends U> mapper)` applies a transformation to the value if present.

**Technical Definition:** "If a value is present, returns an `Optional` describing the result of applying the given mapping function to the value, otherwise returns an empty `Optional`".

#### Syntax Rules and Structure

```java
<U> Optional<U> map(Function<? super T, ? extends U> mapper)
```

#### Annotated Code Example

```java
Optional<String> name = Optional.of("Alice");
Optional<Integer> length = name.map(String::length);
System.out.println(length.get());  // 5
```

### 3.10 `flatMap()`

#### Definitions

**Core Definition:** `flatMap(Function<? super T, ? extends Optional<? extends U>> mapper)` applies a mapping function that itself returns an `Optional`, and flattens the result.

**Technical Definition:** "If a value is present, returns the result of applying the given `Optional`-bearing mapping function to the value, otherwise returns an empty `Optional`".

#### Syntax Rules and Structure

```java
<U> Optional<U> flatMap(Function<? super T, ? extends Optional<? extends U>> mapper)
```

#### Annotated Code Example

```java
Optional<String> name = Optional.of("Alice");
Optional<String> upper = name.flatMap(n -> Optional.of(n.toUpperCase()));
System.out.println(upper.get());  // ALICE
```

### 3.11 `filter()`

#### Definitions

**Core Definition:** `filter(Predicate<? super T> predicate)` returns the `Optional` if the value matches the predicate, otherwise returns an empty `Optional`.

**Technical Definition:** "If a value is present, and the value matches the given predicate, returns an `Optional` describing the value, otherwise returns an empty `Optional`".

#### Syntax Rules and Structure

```java
Optional<T> filter(Predicate<? super T> predicate)
```

#### Annotated Code Example

```java
Optional<Integer> number = Optional.of(10);
Optional<Integer> filtered = number.filter(n -> n > 5);
System.out.println(filtered.isPresent());  // true

Optional<Integer> notFiltered = number.filter(n -> n > 20);
System.out.println(notFiltered.isPresent());  // false
```

### 3.12 `or()` (Java 9+)

#### Definitions

**Core Definition:** `or(Supplier<? extends Optional<? extends T>> supplier)` returns the current `Optional` if a value is present, otherwise returns the `Optional` produced by the supplier.

**Technical Definition:** "If a value is present, returns an `Optional` describing the value, otherwise returns an `Optional` produced by the supplying function".

#### Syntax Rules and Structure

```java
Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)
```

#### Annotated Code Example

```java
Optional<String> empty = Optional.empty();
Optional<String> fallback = empty.or(() -> Optional.of("Fallback"));
System.out.println(fallback.get());  // Fallback
```

### 3.13 `stream()` (Java 9+)

#### Definitions

**Core Definition:** `stream()` converts the `Optional` into a `Stream` containing either the value or nothing.

**Technical Definition:** "If a value is present, returns a sequential `Stream` containing only that value, otherwise returns an empty `Stream`".

#### Syntax Rules and Structure

```java
Stream<T> stream()
```

#### Annotated Code Example

```java
List<Optional<String>> optionals = List.of(
    Optional.of("A"), Optional.empty(), Optional.of("B")
);
List<String> values = optionals.stream()
    .flatMap(Optional::stream)
    .collect(Collectors.toList());
System.out.println(values);  // [A, B]
```

### Real-World Cases

- **`orElse`/`orElseGet`:** Providing default configuration values.
- **`map`/`flatMap`:** Chaining transformations on possibly-absent values.
- **`filter`:** Validating a value before proceeding.
- **`ifPresent`/`ifPresentOrElse`:** Executing side effects conditionally.
- **`stream`:** Integrating `Optional` with stream pipelines.

---

## 4. Optional Best Practices

### Definitions

**Core Definition:** Optional best practices are guidelines for using `Optional` effectively without introducing unnecessary complexity or performance overhead.

**Technical Definition:** Best practices derive from the original design intent of `Optional`, as articulated by Brian Goetz (Java's language architect): "Optional is intended to provide a limited mechanism for library method return types where there needed to be a clear way to represent 'no result,' and using null for such was overwhelmingly likely to cause errors".

**Beginner-Friendly Explanation:** `Optional` is a tool with a specific purpose. Using it correctly means using it for method return types, not for fields, parameters, or collections. It's like using a screwdriver for screws, not for hammering nails.

### Purposes

- To communicate clearly that a method may return no result
- To force callers to handle the absent case explicitly
- To avoid `NullPointerException` in method return values
- To enable functional chaining of operations on possibly-absent values

### Key Best Practices

| Practice | Description |
|----------|-------------|
| Use as Return Type | Return `Optional` from methods where the result may be absent. |
| Never Use for Fields | `Optional` is not `Serializable` and breaks JPA, Jackson, and other frameworks. |
| Never Use for Parameters | Method signatures become complex; callers can still pass `Optional.empty()`. |
| Never Use for Collections | Return an empty collection instead of `Optional<List<T>>`. |
| Prefer `orElseGet` over `orElse` | `orElse` eagerly evaluates; `orElseGet` lazily evaluates. |
| Prefer `orElseThrow` over `get` | `get()` throws `NoSuchElementException`; `orElseThrow` provides a custom exception. |
| Use `map`/`flatMap`/`filter` | Chain operations declaratively instead of nested null checks. |
| Never Assign `null` to Optional | Use `Optional.empty()` instead. |

### Annotated Code Example

```java
// GOOD: Optional as return type
public Optional<User> findUserById(Long id) {
    User user = repository.find(id);
    return Optional.ofNullable(user);
}

// GOOD: Using orElseGet for expensive fallback
Optional<Config> config = loadConfig();
Config result = config.orElseGet(() -> loadDefaultConfig());  // Lazy

// GOOD: Using orElseThrow for required value
Optional<User> user = findUserById(1L);
User required = user.orElseThrow(() -> new UserNotFoundException("User not found"));

// BAD: Optional as field
// private Optional<String> name;  // Avoid — breaks serialization

// BAD: Optional as parameter
// public void save(Optional<User> user) { ... }  // Avoid — use overloading

// BAD: Optional as collection element
// Optional<List<String>>  // Avoid — return empty list instead
```

### Real-World Cases

- **Repository Methods:** `Optional<T> findById(ID id)` is the canonical use case.
- **Service Methods:** Returning `Optional` from lookup methods communicates absence.
- **Configuration:** `Optional<String> getProperty(String key)` returns empty if not set.

---

## 5. Optional Pitfalls

### Common Pitfalls Table

| Pitfall | Description | Solution |
|---------|-------------|----------|
| Calling `get()` Without Check | Throws `NoSuchElementException` if empty. | Use `orElse`, `orElseGet`, or `orElseThrow`. |
| `orElse` with Expensive Default | `orElse` always evaluates its argument. | Use `orElseGet` for lazy evaluation. |
| Using Optional as Field | Breaks serialization (Jackson, JPA). | Use plain fields with `null` or domain-specific types. |
| Using Optional as Parameter | Complicates method signatures. | Use method overloading or builder pattern. |
| Optional of Collection | Returning `Optional<List<T>>` instead of empty list. | Return empty collection instead. |
| Assigning `null` to Optional | Violates the contract; leads to `NullPointerException`. | Use `Optional.empty()`. |
| Identity Operations on Optional | `==`, `synchronized`, identity hash code are unreliable. | Use `.equals()` for comparison. |
| Nesting Optional | `Optional<Optional<T>>` is cumbersome. | Use `flatMap` instead. |

### Annotated Code Example

```java
// PITFALL: get() without check
Optional<String> empty = Optional.empty();
// String value = empty.get();  // NoSuchElementException

// SOLUTION: orElseThrow
String value = empty.orElseThrow(() -> new IllegalStateException("Empty"));

// PITFALL: orElse with expensive computation
Optional<String> opt = Optional.of("Present");
String result = opt.orElse(expensiveComputation());  // expensiveComputation() IS called

// SOLUTION: orElseGet
String result2 = opt.orElseGet(() -> expensiveComputation());  // NOT called
```

---

## 6. OptionalInt, OptionalLong, OptionalDouble

### Definitions

**Core Definition:** `OptionalInt`, `OptionalLong`, and `OptionalDouble` are primitive specialisations of `Optional` for `int`, `long`, and `double` respectively.

**Technical Definition:** "A container object which may or may not contain an `int` value. If a value is present, `isPresent()` returns `true` and `getAsInt()` returns the value". These classes exist to avoid the boxing/unboxing overhead of `Optional<Integer>`, `Optional<Long>`, and `Optional<Double>`.

**Beginner-Friendly Explanation:** These are like `Optional` but for primitive numbers. Instead of wrapping an `int` in an `Integer` object (which is less efficient), they work directly with the primitive value.

### Purposes

- To represent optional primitive values without boxing overhead
- To provide primitive-specific methods like `getAsInt()`, `getAsLong()`, `getAsDouble()`
- To integrate with primitive streams (`IntStream`, `LongStream`, `DoubleStream`)

### Syntax Rules and Structure

**Creation:**

```java
OptionalInt optionalInt = OptionalInt.of(42);
OptionalInt emptyInt = OptionalInt.empty();
OptionalLong optionalLong = OptionalLong.of(42L);
OptionalDouble optionalDouble = OptionalDouble.of(3.14);
```

**Retrieval:**

```java
int value = optionalInt.getAsInt();      // If present
long value = optionalLong.getAsLong();
double value = optionalDouble.getAsDouble();
```

**Presence Check:**

```java
boolean present = optionalInt.isPresent();
```

**Fallback:**

```java
int value = optionalInt.orElse(0);
```

### Annotated Code Example

```java
// File: PrimitiveOptionalDemo.java
import java.util.OptionalInt;
import java.util.OptionalLong;
import java.util.OptionalDouble;

public class PrimitiveOptionalDemo {
    public static void main(String[] args) {
        // OptionalInt
        OptionalInt optionalInt = OptionalInt.of(42);
        System.out.println("Int present: " + optionalInt.isPresent());  // true
        System.out.println("Int value: " + optionalInt.getAsInt());    // 42

        OptionalInt emptyInt = OptionalInt.empty();
        System.out.println("Empty int: " + emptyInt.orElse(0));        // 0

        // OptionalLong
        OptionalLong optionalLong = OptionalLong.of(100L);
        System.out.println("Long value: " + optionalLong.getAsLong()); // 100

        // OptionalDouble
        OptionalDouble optionalDouble = OptionalDouble.of(3.14);
        System.out.println("Double value: " + optionalDouble.getAsDouble()); // 3.14
    }
}
```

**Expected Output:**
```
Int present: true
Int value: 42
Empty int: 0
Long value: 100
Double value: 3.14
```

**Step-by-Step Explanation:**

1. `OptionalInt.of(42)` creates an `OptionalInt` containing `42`.
2. `getAsInt()` retrieves the primitive `int` value.
3. `OptionalInt.empty()` creates an empty `OptionalInt`; `orElse(0)` returns `0`.
4. `OptionalLong.of(100L)` and `OptionalDouble.of(3.14)` demonstrate the other primitive variants.

### Real-World Cases

- **Stream Reductions:** `IntStream.max()` returns `OptionalInt`; `DoubleStream.average()` returns `OptionalDouble`.
- **Array Searches:** `Arrays.stream(intArray).findFirst()` returns `OptionalInt`.
- **Numerical Computations:** Avoiding boxing in performance-critical calculations.

---

## References Links

- Class Optional<T> — Java® Platform, Standard Edition & Java Development Kit Version 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html
- Class OptionalInt — Java® Platform, Standard Edition & Java Development Kit Version 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/OptionalInt.html
- Class OptionalLong — Java® Platform, Standard Edition & Java Development Kit Version 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/OptionalLong.html
- Class OptionalDouble — Java® Platform, Standard Edition & Java Development Kit Version 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/OptionalDouble.html
- Optional (Java Platform SE 8) — Java™ Platform, Standard Edition 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html
- Guide To Java Optional — Baeldung - https://www.baeldung.com/java-optional
- Java Optional as Return Type — Baeldung - https://www.baeldung.com/java-optional-return
- Using Optionals — Dev.java - https://dev.java/learn/api/streams/optionals/
- Brian Goetz on Optional Design — OpenJDK Mailing List - https://mail.openjdk.org/pipermail/lambda-libs-spec-experts/2013-October/000077.html
- Java Optional (SE 8): Best Practices, Pitfalls, and Practical Examples — Cleverence - https://www.cleverence.com/articles/oracle-documentation/optional-java-platform-se-8-4821/