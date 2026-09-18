# Java Optional APIs: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** `Optional<T>` is a container object that may or may not contain a non-null value, introduced in Java 8 to provide a type-level solution for representing optional values instead of using `null` references.

**Technical Definition:** `public final class Optional<T> extends Object` is a value-based class in the `java.util` package that wraps a single value of type `T`. If a value is present, `isPresent()` returns `true`; if no value is present, the object is considered empty and `isPresent()` returns `false`. Because `Optional` is value-based, use of identity-sensitive operations (including reference equality `==`, identity hash code, or synchronization) on instances of `Optional` may have unpredictable results and should be avoided.

**Beginner-Friendly Explanation:** `Optional` is like a gift box that may or may not contain a gift. Instead of returning `null` (which often causes `NullPointerException`), a method returns an `Optional` box. The caller can then check if the box contains something, or use functional methods to safely extract or transform the gift if it is there.

### Key Characteristics

`Optional` is primarily intended for use as a method return type where there is a clear need to represent "no result," and where using `null` is likely to cause errors. A variable whose type is `Optional` should never itself be `null`; it should always point to an `Optional` instance. `Optional` provides methods for initialization (`empty`, `of`, `ofNullable`), transformation (`map`, `flatMap`, `filter`), pipeline integration (`stream`), conditional branching (`orElse`, `orElseGet`, `orElseThrow`, `ifPresent`, `ifPresentOrElse`, `or`), and direct inspection (`isPresent`, `isEmpty`, `get`).

### Prerequisites

Readers should be familiar with Java generics, lambda expressions, and method references. A basic understanding of the `java.util.function` package (particularly `Function`, `Predicate`, `Consumer`, `Supplier`, and `Runnable`) is assumed. For the `stream()` method, knowledge of the Stream API is helpful.

### Related Programming Areas with Explanation

`Optional` is related to **functional programming** (through its use of higher-order functions for transformation and branching), **null safety** (providing a type-safe alternative to null references), **API design** (communicating the possibility of absent values in method signatures), and **Stream API integration** (through the `stream()` method, enabling the filtering of empty optionals in stream pipelines).


## Core Concepts and Key Features

### 1. Initialization Methods

#### 1.1 empty()

**Definitions**

- **Core Definition:** Returns an empty `Optional` instance.
- **Technical Definition:** `public static <T> Optional<T> empty()` returns an `Optional` with no value present. Since `Optional` is immutable, this method returns the same cached empty instance on every call.
- **Beginner-Friendly Explanation:** `empty()` creates an empty gift box—a box that contains nothing.

**Purposes**

- To create an `Optional` that explicitly represents "no result."
- To serve as a return value from methods that have no value to return.
- To serve as a fallback in `or()` and `orElse()` chains.

**Syntax Structures and Rules**

Complete general syntax:

```java
Optional<T> empty()
```

Component breakdown:

- Return value: an empty `Optional<T>` instance.
- No parameters.

Syntax rules: The returned instance is a singleton; multiple calls return the same reference.

Constraints and limitations: The type parameter `T` must be inferred from context or explicitly specified.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Creating an empty Optional**

```java
import java.util.Optional;

public class EmptyExample {
    public static void main(String[] args) {
        Optional<String> empty = Optional.empty();
        System.out.println("Is present: " + empty.isPresent());
        System.out.println("Is empty: " + empty.isEmpty());
        System.out.println("orElse: " + empty.orElse("fallback"));
    }
}
```

Expected Output:

```
Is present: false
Is empty: true
orElse: fallback
```

**Why the code produces this result:** `Optional.empty()` creates an empty container. `isPresent()` returns `false`, `isEmpty()` returns `true`, and `orElse` returns the fallback value because no value is present.

**References Links**

- Optional.empty (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#empty()

#### 1.2 of(value)

**Definitions**

- **Core Definition:** Wraps a non-null value. Throws a `NullPointerException` instantly if the value is `null`.
- **Technical Definition:** `public static <T> Optional<T> of(T value)` returns an `Optional` with the specified value present. The value must not be `null`; if it is, a `NullPointerException` is thrown immediately.
- **Beginner-Friendly Explanation:** `of()` puts a guaranteed non-null gift into a box. If you try to put nothing (null) in, it throws an error immediately.

**Purposes**

- To wrap a value that is known to be non-null.
- To create an `Optional` when the value is guaranteed to exist.
- To catch `null` values early with a clear error message.

**Syntax Structures and Rules**

Complete general syntax:

```java
Optional<T> of(T value)
```

Component breakdown:

- `value`: the value to be present, which must not be `null`.
- Return value: an `Optional<T>` with the value present.
- Throws `NullPointerException` if `value` is `null`.

Syntax rules: Use `of` only when the value is definitely not `null`. For potentially null values, use `ofNullable`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Wrapping a non-null value**

```java
import java.util.Optional;

public class OfExample {
    public static void main(String[] args) {
        Optional<String> value = Optional.of("Hello");
        System.out.println("Value: " + value.get());
    }
}
```

Expected Output:

```
Value: Hello
```

**Example 2: of throws exception on null**

```java
import java.util.Optional;

public class OfNullExample {
    public static void main(String[] args) {
        try {
            Optional.of(null);
        } catch (NullPointerException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Caught: NullPointerException
```

**References Links**

- Optional.of (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#of(T)

#### 1.3 ofNullable(value)

**Definitions**

- **Core Definition:** Wraps a value that may or may not be `null`. Returns `empty()` if the value is `null`.
- **Technical Definition:** `public static <T> Optional<T> ofNullable(T value)` returns an `Optional` describing the specified value if non-null, otherwise returns an empty `Optional`.
- **Beginner-Friendly Explanation:** `ofNullable()` is like `of()`, but it is safe to use with values that might be `null`. If the value is `null`, you get an empty box instead of an error.

**Purposes**

- To safely wrap a value that may be `null`.
- To convert a potentially null reference into an `Optional` without exceptions.
- To use as the standard entry point when the nullability of a value is unknown.

**Syntax Structures and Rules**

Complete general syntax:

```java
Optional<T> ofNullable(T value)
```

Component breakdown:

- `value`: the value to be present, which may be `null`.
- Return value: an `Optional<T>` with the value present if non-null, or an empty `Optional` if `null`.

Syntax rules: This is the safest way to create an `Optional` from a potentially null value.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: ofNullable with non-null value**

```java
import java.util.Optional;

public class OfNullableExample {
    public static void main(String[] args) {
        String value = "Hello";
        Optional<String> opt = Optional.ofNullable(value);
        System.out.println(opt.orElse("default"));
    }
}
```

Expected Output:

```
Hello
```

**Example 2: ofNullable with null value**

```java
import java.util.Optional;

public class OfNullableNullExample {
    public static void main(String[] args) {
        String value = null;
        Optional<String> opt = Optional.ofNullable(value);
        System.out.println("Is present: " + opt.isPresent());
        System.out.println("orElse: " + opt.orElse("default"));
    }
}
```

Expected Output:

```
Is present: false
orElse: default
```

**References Links**

- Optional.ofNullable (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#ofNullable(T)


### 2. Value Transformation & Pipeline Integration

#### 2.1 map(Function)

**Definitions**

- **Core Definition:** Transforms the value inside if present; returns empty if not.
- **Technical Definition:** `public <U> Optional<U> map(Function<? super T, ? extends U> mapper)` applies the mapping function to the value if present, returning an `Optional` describing the result. If no value is present, returns an empty `Optional`. If the mapper returns `null`, the result is an empty `Optional`.
- **Beginner-Friendly Explanation:** `map` takes the value out of the box, changes it into something else, and puts the result back in a new box. If the original box was empty, you get an empty box back.

**Purposes**

- To transform the value inside an `Optional` without checking for presence.
- To chain transformations in a functional style.
- To avoid explicit null checks when applying a function to a potentially absent value.

**Syntax Structures and Rules**

Complete general syntax:

```java
<U> Optional<U> map(Function<? super T, ? extends U> mapper)
```

Component breakdown:

- `mapper`: a function to apply to the value if present.
- `U`: the type of the result.
- Return value: an `Optional<U>` containing the transformed value, or empty.

Syntax rules: The mapper must not return `null` (doing so results in an empty `Optional`).

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Mapping a String to its length**

```java
import java.util.Optional;

public class MapExample {
    public static void main(String[] args) {
        Optional<String> name = Optional.of("Alice");
        Optional<Integer> length = name.map(String::length);
        System.out.println("Length: " + length.orElse(0));
    }
}
```

Expected Output:

```
Length: 5
```

**Example 2: map on empty Optional**

```java
import java.util.Optional;

public class MapEmptyExample {
    public static void main(String[] args) {
        Optional<String> empty = Optional.empty();
        Optional<Integer> length = empty.map(String::length);
        System.out.println("Is present: " + length.isPresent());
    }
}
```

Expected Output:

```
Is present: false
```

**References Links**

- Optional.map (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#map(java.util.function.Function)

#### 2.2 flatMap(Function)

**Definitions**

- **Core Definition:** Transforms the value inside into another `Optional`, avoiding nested wrappers (`Optional<Optional<T>>`).
- **Technical Definition:** `public <U> Optional<U> flatMap(Function<? super T, ? extends Optional<? extends U>> mapper)` applies the mapping function to the value if present, returning the `Optional` result directly. If no value is present, returns an empty `Optional`.
- **Beginner-Friendly Explanation:** `flatMap` is like `map`, but the function you provide returns another `Optional`. `flatMap` unwraps the result so you do not end up with a box inside a box.

**Purposes**

- To chain operations that themselves return `Optional`.
- To avoid nested `Optional<Optional<T>>` structures.
- To compose multiple methods that return `Optional` in a single pipeline.

**Syntax Structures and Rules**

Complete general syntax:

```java
<U> Optional<U> flatMap(Function<? super T, ? extends Optional<? extends U>> mapper)
```

Component breakdown:

- `mapper`: a function that returns an `Optional<U>`.
- Return value: the `Optional` returned by the mapper, or empty.

Syntax rules: The mapper must return a non-null `Optional`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: flatMap to avoid nesting**

```java
import java.util.Optional;

class User {
    String name;
    Optional<Address> address;
    User(String name, Optional<Address> address) { this.name = name; this.address = address; }
    Optional<Address> getAddress() { return address; }
}

class Address {
    String city;
    Address(String city) { this.city = city; }
    String getCity() { return city; }
}

public class FlatMapExample {
    public static void main(String[] args) {
        User user = new User("Alice", Optional.of(new Address("NYC")));

        // Without flatMap: Optional<Optional<String>>
        Optional<Optional<String>> nested = user.getAddress().map(Address::getCity);

        // With flatMap: Optional<String>
        Optional<String> city = user.getAddress().flatMap(addr -> Optional.of(addr.getCity()));
        System.out.println("City: " + city.orElse("unknown"));
    }
}
```

Expected Output:

```
City: NYC
```

**References Links**

- Optional.flatMap (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#flatMap(java.util.function.Function)

#### 2.3 filter(Predicate)

**Definitions**

- **Core Definition:** Evaluates the wrapped value. If the value matches the predicate, it is kept; otherwise, it returns empty.
- **Technical Definition:** `public Optional<T> filter(Predicate<? super T> predicate)` returns an `Optional` describing the value if a value is present and the value matches the given predicate, otherwise returns an empty `Optional`.
- **Beginner-Friendly Explanation:** `filter` checks the value inside the box against a condition. If it passes, you get the same box back; if it fails, you get an empty box.

**Purposes**

- To conditionally keep or discard a value based on a predicate.
- To validate the value inside an `Optional` without unwrapping it.
- To chain filtering operations in a functional pipeline.

**Syntax Structures and Rules**

Complete general syntax:

```java
Optional<T> filter(Predicate<? super T> predicate)
```

Component breakdown:

- `predicate`: the condition to test the value against.
- Return value: the same `Optional` if the value matches, or empty.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Filtering a value**

```java
import java.util.Optional;

public class FilterExample {
    public static void main(String[] args) {
        Optional<Integer> number = Optional.of(10);
        Optional<Integer> even = number.filter(n -> n % 2 == 0);
        System.out.println("Even: " + even.orElse(-1));

        Optional<Integer> odd = number.filter(n -> n % 2 != 0);
        System.out.println("Odd: " + odd.orElse(-1));
    }
}
```

Expected Output:

```
Even: 10
Odd: -1
```

**References Links**

- Optional.filter (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#filter(java.util.function.Predicate)

#### 2.4 stream() (Added in Java 9)

**Definitions**

- **Core Definition:** Converts the `Optional` directly into a `Stream` of zero or one element.
- **Technical Definition:** `public Stream<T> stream()` returns a sequential `Stream` containing the value if present, otherwise returns an empty `Stream`. This method was added in Java 9 to enable filtering out empty optionals inside a stream pipeline.
- **Beginner-Friendly Explanation:** `stream()` turns the box into a stream that either has one item (if the box was not empty) or no items (if the box was empty). This is perfect for filtering out empty optionals when you have a stream of boxes.

**Purposes**

- To convert an `Optional` into a `Stream` for use in stream pipelines.
- To filter out empty optionals in a stream of optionals using `flatMap(Optional::stream)`.
- To integrate `Optional` into the Stream API ecosystem.

**Syntax Structures and Rules**

Complete general syntax:

```java
Stream<T> stream()
```

Component breakdown:

- Return value: a `Stream<T>` containing zero or one element.

Syntax rules: The stream contains the value if present, or is empty if not present.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Filtering empty optionals in a stream**

```java
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

public class StreamExample {
    public static void main(String[] args) {
        List<Optional<String>> optionals = List.of(
                Optional.of("apple"),
                Optional.empty(),
                Optional.of("banana"),
                Optional.empty(),
                Optional.of("cherry")
        );

        List<String> present = optionals.stream()
                .flatMap(Optional::stream)
                .collect(Collectors.toList());

        System.out.println(present);
    }
}
```

Expected Output:

```
[apple, banana, cherry]
```

**Why the code produces this result:** `flatMap(Optional::stream)` converts each `Optional` into a stream (zero or one element) and flattens all streams into a single stream, effectively removing the empty optionals.

**References Links**

- Optional.stream (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#stream()
- Java 9 Optional API Additions (Baeldung) – https://www.baeldung.com/java-9-optional


### 3. Conditional Branching & Safe Recovery

#### 3.1 orElse(defaultValue)

**Definitions**

- **Core Definition:** Returns the value if present, otherwise returns the pre-constructed fallback value.
- **Technical Definition:** `public T orElse(T other)` returns the value if present, otherwise returns `other`.
- **Beginner-Friendly Explanation:** `orElse` is like saying "give me the gift if it is there; otherwise give me this backup gift." The backup is always evaluated, even if the gift is present.

**Purposes**

- To provide a default value when the `Optional` is empty.
- To safely unwrap an `Optional` with a fallback.

**Syntax Structures and Rules**

Complete general syntax:

```java
T orElse(T other)
```

Component breakdown:

- `other`: the value to return if no value is present.
- Return value: the value if present, otherwise `other`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: orElse with present value**

```java
import java.util.Optional;

public class OrElseExample {
    public static void main(String[] args) {
        Optional<String> value = Optional.of("Hello");
        System.out.println(value.orElse("World"));
    }
}
```

Expected Output:

```
Hello
```

**Example 2: orElse with empty Optional**

```java
import java.util.Optional;

public class OrElseEmptyExample {
    public static void main(String[] args) {
        Optional<String> empty = Optional.empty();
        System.out.println(empty.orElse("World"));
    }
}
```

Expected Output:

```
World
```

**References Links**

- Optional.orElse (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#orElse(T)

#### 3.2 orElseGet(Supplier)

**Definitions**

- **Core Definition:** Lazily returns the fallback value from the supplier only if the `Optional` is empty.
- **Technical Definition:** `public T orElseGet(Supplier<? extends T> supplier)` returns the value if present, otherwise invokes the supplier and returns its result.
- **Beginner-Friendly Explanation:** `orElseGet` is like `orElse`, but the backup gift is only created if the box is actually empty. This is more efficient if creating the backup is expensive.

**Purposes**

- To provide a lazily computed default value when the `Optional` is empty.
- To avoid unnecessary computation of the fallback value.

**Syntax Structures and Rules**

Complete general syntax:

```java
T orElseGet(Supplier<? extends T> supplier)
```

Component breakdown:

- `supplier`: a function that produces the fallback value.
- Return value: the value if present, otherwise the supplier's result.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: orElseGet with empty Optional**

```java
import java.util.Optional;

public class OrElseGetExample {
    public static void main(String[] args) {
        Optional<String> empty = Optional.empty();
        String result = empty.orElseGet(() -> {
            System.out.println("Supplier called");
            return "Generated";
        });
        System.out.println(result);
    }
}
```

Expected Output:

```
Supplier called
Generated
```

**References Links**

- Optional.orElseGet (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#orElseGet(java.util.function.Supplier)

#### 3.3 orElseThrow() (Added in Java 10)

**Definitions**

- **Core Definition:** Returns the value if present; otherwise throws a `NoSuchElementException`.
- **Technical Definition:** `public T orElseThrow()` returns the contained value if present, otherwise throws `NoSuchElementException`. This method was added in Java 10 as the preferred alternative to calling `get()`.
- **Beginner-Friendly Explanation:** `orElseThrow()` is like `get()` but with a clearer name. It is the recommended way to extract a value when you are certain it is present.

**Purposes**

- To safely extract the value when presence is guaranteed or when an exception is acceptable.
- To replace the discouraged `get()` method.

**Syntax Structures and Rules**

Complete general syntax:

```java
T orElseThrow()
```

Component breakdown:

- Return value: the contained value.
- Throws `NoSuchElementException` if no value is present.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: orElseThrow with present value**

```java
import java.util.Optional;

public class OrElseThrowExample {
    public static void main(String[] args) {
        Optional<String> value = Optional.of("Hello");
        System.out.println(value.orElseThrow());
    }
}
```

Expected Output:

```
Hello
```

**Example 2: orElseThrow with empty Optional**

```java
import java.util.Optional;

public class OrElseThrowEmptyExample {
    public static void main(String[] args) {
        Optional<String> empty = Optional.empty();
        try {
            empty.orElseThrow();
        } catch (java.util.NoSuchElementException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Caught: NoSuchElementException
```

**References Links**

- Optional.orElseThrow (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#orElseThrow()
- New Features in Java 10 (Baeldung) – https://www.baeldung.com/java-10-overview

#### 3.4 orElseThrow(Supplier)

**Definitions**

- **Core Definition:** Throws a custom exception if the `Optional` is empty.
- **Technical Definition:** `public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X` returns the value if present, otherwise throws the exception produced by the supplier.
- **Beginner-Friendly Explanation:** `orElseThrow` with a supplier lets you throw your own exception instead of the default `NoSuchElementException`.

**Purposes**

- To throw a domain-specific exception when a value is absent.
- To provide a clear error message for missing values.

**Syntax Structures and Rules**

Complete general syntax:

```java
<X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X
```

Component breakdown:

- `exceptionSupplier`: a supplier that creates the exception to throw.
- Return value: the value if present.
- Throws the exception produced by the supplier if no value is present.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Throwing a custom exception**

```java
import java.util.Optional;

public class OrElseThrowCustomExample {
    public static void main(String[] args) {
        Optional<String> empty = Optional.empty();
        try {
            empty.orElseThrow(() -> new IllegalStateException("Value not found"));
        } catch (IllegalStateException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

Expected Output:

```
Caught: Value not found
```

**References Links**

- Optional.orElseThrow(Supplier) (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#orElseThrow(java.util.function.Supplier)

#### 3.5 ifPresent(Consumer)

**Definitions**

- **Core Definition:** Executes a block of code only if a value is present.
- **Technical Definition:** `public void ifPresent(Consumer<? super T> action)` performs the given action with the value if present, otherwise does nothing.
- **Beginner-Friendly Explanation:** `ifPresent` is like saying "if the box has a gift, do something with it; if not, do nothing."

**Purposes**

- To perform a side effect only when a value is present.
- To replace `if (opt.isPresent()) { ... }` with a functional style.

**Syntax Structures and Rules**

Complete general syntax:

```java
void ifPresent(Consumer<? super T> action)
```

Component breakdown:

- `action`: the action to perform with the value if present.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: ifPresent with value**

```java
import java.util.Optional;

public class IfPresentExample {
    public static void main(String[] args) {
        Optional<String> value = Optional.of("Hello");
        value.ifPresent(v -> System.out.println("Value: " + v));
    }
}
```

Expected Output:

```
Value: Hello
```

**References Links**

- Optional.ifPresent (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#ifPresent(java.util.function.Consumer)

#### 3.6 ifPresentOrElse(Consumer, Runnable) (Added in Java 9)

**Definitions**

- **Core Definition:** The functional equivalent of an if-else statement. Executes the consumer if present, otherwise runs the fallback `Runnable`.
- **Technical Definition:** `public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)` performs the given action with the value if present, otherwise performs the given empty-based action. This method was added in Java 9.
- **Beginner-Friendly Explanation:** `ifPresentOrElse` lets you say "if the box has a gift, do this; otherwise, do that." It handles both cases in one call.

**Purposes**

- To handle both the present and absent cases in a single method call.
- To replace imperative if-else blocks with functional style.

**Syntax Structures and Rules**

Complete general syntax:

```java
void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)
```

Component breakdown:

- `action`: the action to perform if a value is present.
- `emptyAction`: the action to perform if no value is present.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: ifPresentOrElse with present value**

```java
import java.util.Optional;

public class IfPresentOrElseExample {
    public static void main(String[] args) {
        Optional<String> value = Optional.of("Hello");
        value.ifPresentOrElse(
                v -> System.out.println("Present: " + v),
                () -> System.out.println("Empty")
        );
    }
}
```

Expected Output:

```
Present: Hello
```

**Example 2: ifPresentOrElse with empty Optional**

```java
import java.util.Optional;

public class IfPresentOrElseEmptyExample {
    public static void main(String[] args) {
        Optional<String> empty = Optional.empty();
        empty.ifPresentOrElse(
                v -> System.out.println("Present: " + v),
                () -> System.out.println("Empty")
        );
    }
}
```

Expected Output:

```
Empty
```

**References Links**

- Optional.ifPresentOrElse (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#ifPresentOrElse(java.util.function.Consumer,java.lang.Runnable)
- Java 9 Optional API Additions (Baeldung) – https://www.baeldung.com/java-9-optional

#### 3.7 or(Supplier<Optional>) (Added in Java 9)

**Definitions**

- **Core Definition:** If empty, lazily returns a different fallback `Optional` container rather than unwrapping to a raw value.
- **Technical Definition:** `public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)` returns the same `Optional` if a value is present, otherwise returns the `Optional` produced by the supplier. This method was added in Java 9.
- **Beginner-Friendly Explanation:** `or` is like saying "if my box is empty, give me another box to try." It keeps you in the `Optional` world instead of unwrapping to a raw value.

**Purposes**

- To chain fallback `Optional` containers without unwrapping.
- To provide an alternative `Optional` when the current one is empty.
- To compose multiple sources of optional data lazily.

**Syntax Structures and Rules**

Complete general syntax:

```java
Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)
```

Component breakdown:

- `supplier`: a supplier that produces the fallback `Optional`.
- Return value: the current `Optional` if present, otherwise the supplier's `Optional`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: or with fallback Optional**

```java
import java.util.Optional;

public class OrExample {
    public static void main(String[] args) {
        Optional<String> primary = Optional.empty();
        Optional<String> fallback = Optional.of("Fallback");

        Optional<String> result = primary.or(() -> fallback);
        System.out.println(result.orElse("None"));
    }
}
```

Expected Output:

```
Fallback
```

**References Links**

- Optional.or (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#or(java.util.function.Supplier)
- Java 9 Optional API Additions (Baeldung) – https://www.baeldung.com/java-9-optional


### 4. Direct Inspection

#### 4.1 isPresent()

**Definitions**

- **Core Definition:** Returns `true` if a value exists.
- **Technical Definition:** `public boolean isPresent()` returns `true` if a value is present, otherwise `false`.
- **Beginner-Friendly Explanation:** `isPresent` checks if the box has anything inside. It returns `true` if there is a gift, `false` if the box is empty.

**Purposes**

- To check if a value is present before extracting it.
- To use in imperative-style null checks.

**Syntax Structures and Rules**

Complete general syntax:

```java
boolean isPresent()
```

Component breakdown:

- Return value: `true` if a value is present, otherwise `false`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: isPresent with value**

```java
import java.util.Optional;

public class IsPresentExample {
    public static void main(String[] args) {
        Optional<String> value = Optional.of("Hello");
        System.out.println(value.isPresent());
    }
}
```

Expected Output:

```
true
```

**References Links**

- Optional.isPresent (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#isPresent()

#### 4.2 isEmpty() (Added in Java 11)

**Definitions**

- **Core Definition:** Returns `true` if the container is empty. Saves you from writing `!opt.isPresent()`.
- **Technical Definition:** `public boolean isEmpty()` returns `true` if no value is present, otherwise `false`. This method was added in Java 11 as the logical opposite of `isPresent()`.
- **Beginner-Friendly Explanation:** `isEmpty` is the opposite of `isPresent`. It returns `true` if the box is empty.

**Purposes**

- To check for absence of a value without negation.
- To improve readability compared to `!opt.isPresent()`.

**Syntax Structures and Rules**

Complete general syntax:

```java
boolean isEmpty()
```

Component breakdown:

- Return value: `true` if no value is present, otherwise `false`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: isEmpty with empty Optional**

```java
import java.util.Optional;

public class IsEmptyExample {
    public static void main(String[] args) {
        Optional<String> empty = Optional.empty();
        System.out.println(empty.isEmpty());
    }
}
```

Expected Output:

```
true
```

**References Links**

- Optional.isEmpty (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#isEmpty()

#### 4.3 get()

**Definitions**

- **Core Definition:** Directly extracts the value. Highly discouraged because it throws an exception if empty; use `orElseThrow()` instead.
- **Technical Definition:** `public T get()` returns the value if present, otherwise throws `NoSuchElementException`. The Java documentation states: "The preferred alternative to this method is `orElseThrow()`".
- **Beginner-Friendly Explanation:** `get()` is like reaching into the box without checking first. If the box is empty, you get an error. It is better to use safer alternatives.

**Purposes**

- To extract the value when presence is absolutely guaranteed.
- Legacy code compatibility (but `orElseThrow()` is preferred).

**Syntax Structures and Rules**

Complete general syntax:

```java
T get()
```

Component breakdown:

- Return value: the contained value.
- Throws `NoSuchElementException` if no value is present.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: get with present value**

```java
import java.util.Optional;

public class GetExample {
    public static void main(String[] args) {
        Optional<String> value = Optional.of("Hello");
        System.out.println(value.get());
    }
}
```

Expected Output:

```
Hello
```

**Example 2: get with empty Optional**

```java
import java.util.Optional;

public class GetEmptyExample {
    public static void main(String[] args) {
        Optional<String> empty = Optional.empty();
        try {
            empty.get();
        } catch (java.util.NoSuchElementException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Caught: NoSuchElementException
```

**References Links**

- Optional.get (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#get()


## Summary Table of Optional Methods

| Method | Category | Java Version | Return Type | Description |
|---|---|---|---|---|
| empty() | Initialization | 8 | Optional<T> | Creates an empty Optional |
| of(value) | Initialization | 8 | Optional<T> | Wraps a non-null value |
| ofNullable(value) | Initialization | 8 | Optional<T> | Wraps a possibly null value |
| map(Function) | Transformation | 8 | Optional<U> | Transforms the value if present |
| flatMap(Function) | Transformation | 8 | Optional<U> | Transforms into another Optional |
| filter(Predicate) | Transformation | 8 | Optional<T> | Keeps value if it matches predicate |
| stream() | Pipeline | 9 | Stream<T> | Converts to a stream of 0-1 elements |
| orElse(default) | Branching | 8 | T | Returns default if empty |
| orElseGet(Supplier) | Branching | 8 | T | Lazily returns default if empty |
| orElseThrow() | Branching | 10 | T | Throws NoSuchElementException if empty |
| orElseThrow(Supplier) | Branching | 8 | T | Throws custom exception if empty |
| ifPresent(Consumer) | Branching | 8 | void | Executes action if present |
| ifPresentOrElse | Branching | 9 | void | Executes action or fallback |
| or(Supplier) | Branching | 9 | Optional<T> | Returns fallback Optional if empty |
| isPresent() | Inspection | 8 | boolean | Returns true if value exists |
| isEmpty() | Inspection | 11 | boolean | Returns true if empty |
| get() | Inspection | 8 | T | Extracts value (discouraged) |


## Version-Specific Notes

- `Optional` was introduced in Java 8.
- `stream()`, `ifPresentOrElse()`, and `or()` were added in Java 9.
- `orElseThrow()` (no-argument) was added in Java 10 as the preferred alternative to `get()`.
- `isEmpty()` was added in Java 11.
- `Optional` is a value-based class; identity-sensitive operations should be avoided.
- `Optional` is not serializable and should not be used as a field type in serializable classes.
- `Optional` is primarily intended for use as a method return type, not for fields, parameters, or collections.


## References

- Optional (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html
- Optional (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html
- Java 9 Optional API Additions (Baeldung) – https://www.baeldung.com/java-9-optional
- New Features in Java 10 (Baeldung) – https://www.baeldung.com/java-10-overview
- Java Optional as Return Type (Baeldung) – https://www.baeldung.com/java-optional-return
- OpenJDK: Optional stream() diff – https://android.googlesource.com/platform/external/guava/+/b72102d427be43b5c092042c9a9e5d972d55510c^!/guava
- OpenJDK: Optional.isEmpty() diff – https://cr.openjdk.org/~iris/se/11/pr/java-se-11-pr-spec-02/api/java.base/java/util/Optional.html