# Java Functional Interfaces: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** A Java functional interface is an interface that contains exactly one abstract method, serving as the target type for lambda expressions and method references, thereby enabling functional programming within Java's object-oriented framework.

**Technical Definition:** A functional interface, as defined in the Java Language Specification §9.8, is an interface that has just one abstract method (aside from the methods of `Object`), and thus represents a single function contract. The `java.util.function` package, introduced in Java 8, provides a comprehensive set of general-purpose functional interfaces that are used throughout the JDK and are available for user code. These interfaces follow an extensible naming convention based on function arity (e.g., `BiFunction` for binary functions) and primitive type specializations (e.g., `IntPredicate`). The core function shapes are `Function` (unary function from T to R), `Consumer` (unary function from T to void), `Predicate` (unary function from T to boolean), and `Supplier` (nullary function to R). Derived shapes include `UnaryOperator` (extends `Function`) and `BinaryOperator` (extends `BiFunction`).

**Beginner-Friendly Explanation:** A functional interface is like a job description with exactly one task. In Java, you can write a short lambda expression to fill that job instead of writing an entire class. Java provides a toolbox of ready-made interfaces for the most common tasks: `Predicate` for checking conditions, `Function` for transforming data, `Consumer` for performing actions, and `Supplier` for producing values. There are also "bi" versions that take two arguments instead of one.

### Key Characteristics

The `java.util.function` package is annotated with `@FunctionalInterface`, which is not a requirement for the compiler to recognize an interface as functional but serves to capture design intent and identify accidental violations. Functional interfaces can provide a target type in multiple contexts: assignment context, method invocation context, and cast context. The interfaces follow a naming convention: basic shapes (`Function`, `Consumer`, `Predicate`, `Supplier`) can be modified by an arity prefix (`BiFunction`, `BiConsumer`, `BiPredicate`) and specialized with primitive type prefixes (`IntFunction`, `LongPredicate`, `DoubleConsumer`, etc.). Functional interfaces often represent abstract concepts like functions, actions, or predicates, and in documentation, it is common to refer directly to those abstract concepts (e.g., "this function" instead of "the function represented by this object").

### Prerequisites

Readers should be familiar with Java interfaces, generics, and lambda expressions. A basic understanding of the `@FunctionalInterface` annotation and method references is helpful. For advanced sections, familiarity with the Stream API and the `java.util.function` package hierarchy is assumed.

### Related Programming Areas with Explanation

Functional interfaces are related to lambda expressions (they are the target types that make lambdas possible), method references (which are shorthand for lambdas that call existing methods), the Stream API (where `Predicate`, `Function`, `Consumer`, and `Supplier` are used extensively in operations like `filter`, `map`, `forEach`, and `collect`), and concurrent programming (where `Supplier`, `Consumer`, and `Function` are used with `CompletableFuture` and parallel streams). They are also closely related to design patterns such as Strategy, Command, and Template Method.

## Core Concepts and Key Features

### 1. The Core Four and Their Bi-Variations

**Core Definition:** The "Core Four" functional interfaces are `Predicate`, `Function`, `Consumer`, and `Supplier`, representing the four fundamental function shapes: boolean-valued, value-transforming, void-returning, and value-producing. Their "Bi" variations (`BiPredicate`, `BiFunction`, `BiConsumer`) extend these shapes to accept two arguments.

**Technical Definition:** These interfaces are all declared in the `java.util.function` package and are annotated with `@FunctionalInterface`. Each has a single abstract method (SAM) that defines its function contract. The Bi-variations introduce a second type parameter to accommodate two-argument functions.

**Beginner-Friendly Explanation:** Think of these as the four basic tools in your functional programming toolbox. `Predicate` asks a yes/no question. `Function` transforms one thing into another. `Consumer` does something with a value but gives nothing back. `Supplier` gives you a value without needing any input. The "Bi" versions just mean they take two inputs instead of one.

#### 1.1 `Predicate<T>` and `BiPredicate<T, U>`

**Definitions**

- **Core Definition:** 
  - `Predicate<T>` represents a boolean-valued function of one argument.
  - `BiPredicate<T, U>` represents a boolean-valued function of two arguments.
- **Technical Definition:** 
  - `Predicate<T>` is a functional interface whose functional method is `boolean test(T t)`. It provides default methods `and()`, `or()`, and `negate()`, plus static methods `isEqual()` and `not()`. 
  - `BiPredicate<T, U>` is a functional interface whose functional method is `boolean test(T t, U u)`, with default methods `and()`, `or()`, and `negate()`.
- **Beginner-Friendly Explanation:** 
  - A `Predicate` is like a filter: you give it a value, and it tells you "yes" or "no" based on some condition. 
  - A `BiPredicate` does the same but takes two values.

**Purposes**

- To test whether a value satisfies a condition.
- To compose multiple conditions using `and`, `or`, and `negate`.
- To filter collections and streams (e.g., `stream.filter(predicate)`).

**Syntax Structures and Rules**

Complete general syntax:

```java
// Predicate
Predicate<T> predicate = value -> condition;
boolean result = predicate.test(value);

// BiPredicate
BiPredicate<T, U> biPredicate = (value1, value2) -> condition;
boolean result = biPredicate.test(value1, value2);

// Composition
Predicate<T> combined = predicate1.and(predicate2);
Predicate<T> negated = predicate.negate();
Predicate<T> orCombined = predicate1.or(predicate2);
```

Component breakdown:

- `test(T t)`: the functional method of `Predicate`.
- `test(T t, U u)`: the functional method of `BiPredicate`.
- `and(Predicate other)`: returns a composed predicate representing short-circuiting logical AND.
- `or(Predicate other)`: returns a composed predicate representing short-circuiting logical OR.
- `negate()`: returns a predicate representing the logical negation.

Syntax rules: `and()` and `or()` are default methods that return new predicates. `negate()` returns a new predicate. `isEqual()` is a static method that returns a predicate testing equality with a target reference.

Constraints and limitations: `Predicate` and `BiPredicate` cannot be composed with each other directly due to differing arities.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic `Predicate` and composition**

```java
import java.util.function.Predicate;

public class PredicateExample {
    public static void main(String[] args) {
        // Simple predicate
        Predicate<String> isEmpty = s -> s.isEmpty();
        System.out.println("isEmpty(''): " + isEmpty.test(""));
        System.out.println("isEmpty('hi'): " + isEmpty.test("hi"));

        // Composition: AND
        Predicate<String> startsWithA = s -> s.startsWith("A");
        Predicate<String> longerThan3 = s -> s.length() > 3;

        Predicate<String> startsWithAAndLong = startsWithA.and(longerThan3);
        
        System.out.println("'Apple': " + startsWithAAndLong.test("Apple"));
        System.out.println("'Ant': " + startsWithAAndLong.test("Ant"));

        // Negation
        Predicate<String> notEmpty = isEmpty.negate();
        System.out.println("notEmpty('hi'): " + notEmpty.test("hi"));
    }
}
```

Expected Output:

```
isEmpty(''): true
isEmpty('hi'): false
'Apple': true
'Ant': false
notEmpty('hi'): true
```

**Why the code produces this result:** The `and()` method composes two predicates into one that returns `true` only if both are satisfied. "Apple" passes both, but "Ant" fails the length check. `negate()` inverts the result of `isEmpty`.

**Example 2: `BiPredicate` example**

```java
import java.util.function.BiPredicate;

public class BiPredicateExample {
    public static void main(String[] args) {
        BiPredicate<String, Integer> isLongerThan = (s, n) -> s.length() > n;
        System.out.println("'Hello' > 3: " + isLongerThan.test("Hello", 3));
        System.out.println("'Hi' > 3: " + isLongerThan.test("Hi", 3));
    }
}
```

Expected Output:

```
'Hello' > 3: true
'Hi' > 3: false
```

**Real-World Cases with Explanation**

`Predicate` is used in `Collection.removeIf()`, `Stream.filter()`, and validation frameworks. `BiPredicate` is used in two-factor authentication checks, comparing two records, and in `Map` operations that require key-value condition testing.

**References Links**

- Predicate (Java SE 11 API Documentation) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html
- BiPredicate (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/BiPredicate.html

#### 1.2 `Function<T, R>` and `BiFunction<T, U, R>`

**Definitions**

- **Core Definition:** 
  - `Function<T, R>` represents a function that accepts one argument of type T and produces a result of type R. 
  - `BiFunction<T, U, R>` accepts two arguments (T and U) and produces a result of type R.
- **Technical Definition:** `
  - Function<T, R>` is a functional interface whose functional method is `R apply(T t)`. It provides default methods `compose()` and `andThen()`, plus a static method `identity()`. 
  - `BiFunction<T, U, R>` is a functional interface whose functional method is `R apply(T t, U u)`, with a default method `andThen()`.
- **Beginner-Friendly Explanation:** 
  - A `Function` is a transformer: you give it one thing, and it gives you back another thing (possibly of a different type). 
  - A `BiFunction` takes two things and produces one result.

**Purposes**

- To transform a value from one type to another.
- To compose functions using `compose` (apply the argument function first) and `andThen` (apply this function first).
- To map elements in collections and streams (e.g., `stream.map(function)`).

**Syntax Structures and Rules**

Complete general syntax:

```java
// Function
Function<T, R> function = value -> transformedValue;
R result = function.apply(value);

// BiFunction
BiFunction<T, U, R> biFunction = (value1, value2) -> result;
R result = biFunction.apply(value1, value2);

// Composition
Function<T, R> composed1 = function1.andThen(function2); // function1 first
Function<T, R> composed2 = function1.compose(function2); // function2 first
```

Component breakdown:

- `apply(T t)`: the functional method of `Function`.
- `apply(T t, U u)`: the functional method of `BiFunction`.
- `andThen(Function after)`: returns a composed function that applies this function first, then the `after` function.
- `compose(Function before)`: returns a composed function that applies the `before` function first, then this function.
- `identity()`: a static method that returns a function that always returns its input argument.

Syntax rules: `andThen` is available on both `Function` and `BiFunction`. `compose` is only available on `Function`.

Constraints and limitations: The `andThen` method on `BiFunction` accepts a `Function`, not a `BiFunction`, because the result of the `BiFunction` is a single value.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `Function` with `andThen` and `compose`**

```java
import java.util.function.Function;

public class FunctionExample {
    public static void main(String[] args) {
        Function<String, Integer> parse = Integer::parseInt;
        Function<Integer, Integer> square = x -> x * x;

        // andThen: parse first, then square
        Function<String, Integer> parseAndSquare = parse.andThen(square);
        System.out.println("'5' -> " + parseAndSquare.apply("5"));

        // compose: square first, then parse (not applicable here, use different example)
        Function<Integer, String> intToString = Object::toString;
        Function<String, String> decorated = intToString.compose(square);
        System.out.println("Square of 4 -> " + decorated.apply(4));
    }
}
```

Expected Output:

```
'5' -> 25
Square of 4 -> 16
```

**Example 2: `BiFunction` with `andThen`**

```java
import java.util.function.BiFunction;
import java.util.function.Function;

public class BiFunctionExample {
    public static void main(String[] args) {
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        Function<Integer, Integer> triple = x -> x * 3;

        BiFunction<Integer, Integer, Integer> addThenTriple = add.andThen(triple);
        System.out.println("(2 + 3) * 3 = " + addThenTriple.apply(2, 3));
    }
}
```

Expected Output:

```
(2 + 3) * 3 = 15
```

**Real-World Cases with Explanation**

`Function` is used in `Stream.map()` to transform elements (e.g., converting strings to integers). `BiFunction` is used in `Map.merge()` and `Map.compute()` to combine old and new values, and in `Collectors.toMap()` for merge functions.

**References Links**

- Function (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/Function.html
- BiFunction (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/BiFunction.html

#### 1.3 `Consumer<T>` and `BiConsumer<T, U>`

**Definitions**

- **Core Definition:** 
  - `Consumer<T>` represents an operation that accepts a single input argument and returns no result (void). 
  - `BiConsumer<T, U>` accepts two input arguments and returns no result.
- **Technical Definition:** 
  - `Consumer<T>` is a functional interface whose functional method is `void accept(T t)`. It provides a default method `andThen()`. 
  - `BiConsumer<T, U>` is a functional interface whose functional method is `void accept(T t, U u)`, with a default method `andThen()`.
- **Beginner-Friendly Explanation:** 
  - A `Consumer` is an action: it takes a value and does something with it (like printing or saving), but does not return anything. 
  - A `BiConsumer` takes two values and performs an action with them.

**Purposes**

- To perform side-effect operations on a value (e.g., printing, logging, saving).
- To iterate over collections with `forEach`.
- To compose multiple actions using `andThen`.

**Syntax Structures and Rules**

Complete general syntax:

```java
// Consumer
Consumer<T> consumer = value -> { /* action */ };
consumer.accept(value);

// BiConsumer
BiConsumer<T, U> biConsumer = (value1, value2) -> { /* action */ };
biConsumer.accept(value1, value2);

// Composition
Consumer<T> combined = consumer1.andThen(consumer2);
```

Component breakdown:

- `accept(T t)`: the functional method of `Consumer`.
- `accept(T t, U u)`: the functional method of `BiConsumer`.
- `andThen(Consumer after)`: returns a composed consumer that performs this operation followed by the `after` operation.

Syntax rules: `Consumer` operates via side effects. `andThen` performs operations in sequence. If the first operation throws an exception, the second is not performed.

Constraints and limitations: `BiConsumer` has no `andThen` that accepts another `BiConsumer` with a compatible signature in the same way as `Consumer`, but it does have an `andThen` method.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `Consumer` and `andThen`**

```java
import java.util.function.Consumer;

public class ConsumerExample {
    public static void main(String[] args) {
        Consumer<String> printer = s -> System.out.println("Printing: " + s);
        Consumer<String> logger = s -> System.out.println("Logging: " + s);

        Consumer<String> combined = printer.andThen(logger);
        combined.accept("Hello");
    }
}
```

Expected Output:

```
Printing: Hello
Logging: Hello
```

**Example 2: `BiConsumer` with a map**

```java
import java.util.HashMap;
import java.util.Map;
import java.util.function.BiConsumer;

public class BiConsumerExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put("Alice", 30);
        map.put("Bob", 25);

        BiConsumer<String, Integer> printer = (name, age) ->
            System.out.println(name + " is " + age + " years old");

        map.forEach(printer);
    }
}
```

Expected Output:

```
Alice is 30 years old
Bob is 25 years old
```

**Real-World Cases with Explanation**

`Consumer` is used in `Iterable.forEach()`, `Stream.forEach()`, and `Optional.ifPresent()`. `BiConsumer` is used in `Map.forEach()` to process key-value pairs, and in event handling systems where two pieces of data are needed.

**References Links**

- Consumer (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/Consumer.html
- BiConsumer (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/BiConsumer.html

#### 1.4 `Supplier<T>`

**Definitions**

- **Core Definition:** `Supplier<T>` represents a supplier of results, taking no arguments and returning a result of type T.
- **Technical Definition:** `Supplier<T>` is a functional interface whose functional method is `T get()`. There is no requirement that a new or distinct result be returned each time the supplier is invoked. There is no `BiSupplier` because a method can only return one value.
- **Beginner-Friendly Explanation:** A `Supplier` is a value producer: you call it, and it gives you something. It does not need any input to do its job.

**Purposes**

- To generate or provide values on demand.
- To implement lazy initialization and deferred execution.
- To serve as a factory for objects.

**Syntax Structures and Rules**

Complete general syntax:

```java
Supplier<T> supplier = () -> value;
T result = supplier.get();
```

Component breakdown:

- `get()`: the functional method that returns a result of type T.

Syntax rules: The `get()` method takes no arguments and returns a value. It may return the same value or a new value each time.

Constraints and limitations: There is no `BiSupplier` because a method can only return one value.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic `Supplier`**

```java
import java.util.function.Supplier;

public class SupplierExample {
    public static void main(String[] args) {
        Supplier<String> greeting = () -> "Hello, World!";
        System.out.println(greeting.get());

        Supplier<Double> random = Math::random;
        System.out.println("Random >= 0: " + (random.get() >= 0));
    }
}
```

Expected Output:

```
Hello, World!
Random >= 0: true
```

**Example 2: `Supplier` for lazy initialization**

```java
import java.util.function.Supplier;

public class LazyInitExample {
    private static Supplier<String> expensiveValue = () -> {
        System.out.println("Computing...");
        return "Expensive Result";
    };

    public static void main(String[] args) {
        System.out.println("Before get");
        System.out.println(expensiveValue.get());
        System.out.println("After get");
    }
}
```

Expected Output:

```
Before get
Computing...
Expensive Result
After get
```

**Real-World Cases with Explanation**

`Supplier` is used in `Optional.orElseGet()` for lazy default values, in `CompletableFuture.supplyAsync()` for asynchronous value generation, and in factory patterns where object creation is deferred.

**References Links**

- Supplier (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/Supplier.html

### 2. Operators (Special Functions)

**Core Definition:** `UnaryOperator<T>` and `BinaryOperator<T>` are specialized functional interfaces that extend `Function` and `BiFunction` respectively, where all type parameters are the same type, representing operations that return a result of the same type as their operands.

**Technical Definition:** `UnaryOperator<T>` extends `Function<T, T>` and provides a static `identity()` method. `BinaryOperator<T>` extends `BiFunction<T, T, T>` and provides static `minBy()` and `maxBy()` methods that accept a `Comparator`.

**Beginner-Friendly Explanation:** An operator is a function where the input and output types are the same. A `UnaryOperator` takes one value and returns a value of the same type (like squaring a number). A `BinaryOperator` takes two values of the same type and returns a value of the same type (like adding two numbers).

#### 2.1 `UnaryOperator<T>`

**Definitions**

- **Core Definition:** `UnaryOperator<T>` is a function that takes one argument of type T and returns a result of the same type T.
- **Technical Definition:** `UnaryOperator<T>` extends `Function<T, T>` and is annotated with `@FunctionalInterface`. Its functional method is inherited from `Function`: `T apply(T t)`. It provides a static `identity()` method that returns a unary operator that always returns its input argument.
- **Beginner-Friendly Explanation:** A `UnaryOperator` is a transformation that preserves the type. For example, converting a string to uppercase is a `UnaryOperator<String>`.

**Purposes**

- To perform type-preserving transformations.
- To provide a more precise type signature than `Function<T, T>`.
- To support operations like incrementing, negating, or formatting a value.

**Syntax Structures and Rules**

Complete general syntax:

```java
UnaryOperator<T> operator = value -> transformedValue;
T result = operator.apply(value);

// Identity operator
UnaryOperator<T> identity = UnaryOperator.identity();
```

Component breakdown:

- `apply(T t)`: inherited from `Function<T, T>`.
- `identity()`: static method returning a unary operator that returns its input.

Syntax rules: The input and output types must be the same.

Constraints and limitations: `UnaryOperator` does not add new abstract methods; it only provides the `identity()` static method.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `UnaryOperator` with `identity`**

```java
import java.util.function.UnaryOperator;

public class UnaryOperatorExample {
    public static void main(String[] args) {
        UnaryOperator<Integer> square = x -> x * x;
        System.out.println("Square of 5: " + square.apply(5));

        UnaryOperator<String> identity = UnaryOperator.identity();
        System.out.println("Identity: " + identity.apply("Hello"));
    }
}
```

Expected Output:

```
Square of 5: 25
Identity: Hello
```

**Real-World Cases with Explanation**

`UnaryOperator` is used in `List.replaceAll()` to transform every element in place, and in `Stream.map()` when the input and output types are the same.

**References Links**

- UnaryOperator (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/UnaryOperator.html

#### 2.2 `BinaryOperator<T>`

**Definitions**

- **Core Definition:** `BinaryOperator<T>` is a function that takes two arguments of type T and returns a result of the same type T.
- **Technical Definition:** `BinaryOperator<T>` extends `BiFunction<T, T, T>` and is annotated with `@FunctionalInterface`. Its functional method is inherited from `BiFunction`: `T apply(T t, T u)`. It provides static `minBy()` and `maxBy()` methods that return a `BinaryOperator` based on a supplied `Comparator`.
- **Beginner-Friendly Explanation:** A `BinaryOperator` combines two values of the same type into one value of that type. For example, adding two integers or finding the maximum of two strings.

**Purposes**

- To combine two values of the same type into one.
- To support reduction operations (e.g., `Stream.reduce(BinaryOperator)`).
- To provide convenient `minBy` and `maxBy` factory methods.

**Syntax Structures and Rules**

Complete general syntax:

```java
BinaryOperator<T> operator = (a, b) -> result;
T result = operator.apply(value1, value2);

// Factory methods
BinaryOperator<T> min = BinaryOperator.minBy(comparator);
BinaryOperator<T> max = BinaryOperator.maxBy(comparator);
```

Component breakdown:

- `apply(T t, T u)`: inherited from `BiFunction<T, T, T>`.
- `minBy(Comparator)`: returns the lesser of two values according to the comparator.
- `maxBy(Comparator)`: returns the greater of two values according to the comparator.

Syntax rules: Both arguments and the return type must be the same type.

Constraints and limitations: The comparator passed to `minBy` and `maxBy` must be non-null.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `BinaryOperator` with `minBy` and `maxBy`**

```java
import java.util.Comparator;
import java.util.function.BinaryOperator;

public class BinaryOperatorExample {
    public static void main(String[] args) {
        BinaryOperator<Integer> add = (a, b) -> a + b;
        System.out.println("10 + 5 = " + add.apply(10, 5));

        BinaryOperator<Integer> min = BinaryOperator.minBy(Comparator.naturalOrder());
        System.out.println("Min of 10, 5: " + min.apply(10, 5));

        BinaryOperator<Integer> max = BinaryOperator.maxBy(Comparator.naturalOrder());
        System.out.println("Max of 10, 5: " + max.apply(10, 5));
    }
}
```

Expected Output:

```
10 + 5 = 15
Min of 10, 5: 5
Max of 10, 5: 10
```

**Example 2: `BinaryOperator` in `Stream.reduce`**

```java
import java.util.List;
import java.util.function.BinaryOperator;

public class ReduceExample {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5);
        BinaryOperator<Integer> sum = (a, b) -> a + b;
        int total = numbers.stream().reduce(0, sum);
        System.out.println("Sum: " + total);
    }
}
```

Expected Output:

```
Sum: 15
```

**Real-World Cases with Explanation**

`BinaryOperator` is used in `Stream.reduce()` for aggregation, in `Map.merge()` for combining values, and in mathematical operations where two operands produce a result of the same type.

**References Links**

- BinaryOperator (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/BinaryOperator.html

### 3. Primitive Specializations

The `java.util.function` package provides primitive type specializations to avoid boxing overhead. These include `IntPredicate`, `LongPredicate`, `DoublePredicate`, `IntFunction`, `LongFunction`, `DoubleFunction`, `IntConsumer`, `LongConsumer`, `DoubleConsumer`, `IntSupplier`, `LongSupplier`, `DoubleSupplier`, `IntUnaryOperator`, `LongUnaryOperator`, `DoubleUnaryOperator`, `IntBinaryOperator`, `LongBinaryOperator`, and `DoubleBinaryOperator`. These interfaces accept or return primitive types directly, improving performance in numeric-heavy applications.

## Summary Table of Functional Interfaces

| Interface | Arity | Input Type(s) | Return Type | Functional Method | Key Methods |
|---|---|---|---|---|---|
| `Predicate<T>` | 1 | T | boolean | `test(T)` | `and`, `or`, `negate`, `isEqual`, `not` |
| `BiPredicate<T,U>` | 2 | T, U | boolean | `test(T,U)` | `and`, `or`, `negate` |
| `Function<T,R>` | 1 | T | R | `apply(T)` | `compose`, `andThen`, `identity` |
| `BiFunction<T,U,R>` | 2 | T, U | R | `apply(T,U)` | `andThen` |
| `Consumer<T>` | 1 | T | void | `accept(T)` | `andThen` |
| `BiConsumer<T,U>` | 2 | T, U | void | `accept(T,U)` | `andThen` |
| `Supplier<T>` | 0 | — | T | `get()` | — |
| `UnaryOperator<T>` | 1 | T | T | `apply(T)` | `identity` |
| `BinaryOperator<T>` | 2 | T, T | T | `apply(T,T)` | `minBy`, `maxBy` |

## Deprecated, Unsafe, and Version-Specific Notes

- The `java.util.function` package was introduced in Java SE 8 (JSR 335). It is not available in Java 7 or earlier.
- The `@FunctionalInterface` annotation is optional but recommended for documenting design intent and enabling compiler checks.
- The `Predicate.not()` static method was introduced in Java 11. Prior to Java 11, `negate()` was the only way to obtain a negated predicate.
- Primitive specializations (e.g., `IntPredicate`, `LongFunction`) should be used in performance-critical code to avoid boxing and unboxing overhead.
- `BiConsumer` and `BiPredicate` do not have primitive specializations; for two-argument primitive operations, custom interfaces or autoboxing must be used.
- The `andThen` method on `BiFunction` accepts a `Function`, not a `BiFunction`, because the result of a `BiFunction` is a single value.

## References

- Package java.util.function (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html
- Predicate (Java SE 11 API Documentation) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html
- Function (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/Function.html
- Consumer (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/Consumer.html
- Supplier (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/Supplier.html
- BiPredicate (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/BiPredicate.html
- BiFunction (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/BiFunction.html
- BiConsumer (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/BiConsumer.html
- UnaryOperator (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/UnaryOperator.html
- BinaryOperator (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/BinaryOperator.html
- Functional Interfaces (Java Language Specification, Java SE 17, §9.8) – https://docs.oracle.com/javase/specs/jls/se17/html/jls-9.html#jls-9.8
- Lambda Expressions (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html