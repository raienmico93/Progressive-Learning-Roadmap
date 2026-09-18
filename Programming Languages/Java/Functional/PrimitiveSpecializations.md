# Java Primitive Specializations: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Java primitive specializations are a set of functional interfaces in the `java.util.function` package that operate directly on primitive types (`int`, `long`, `double`, `boolean`), avoiding the performance overhead of boxing and unboxing that occurs when using generic functional interfaces like `Function<Integer, Integer>`.

**Technical Definition:** The `java.util.function` package provides primitive type specializations for its generic functional interfaces to improve performance in numeric-heavy applications. Because generics in Java do not support primitive types, using a generic interface such as `Predicate<Integer>` requires each primitive `int` to be boxed into an `Integer` object, and each result to be unboxed. The primitive specializations eliminate this overhead by declaring functional methods that accept and return primitive types directly. These interfaces follow a consistent naming convention: input-type prefixes (`Int`, `Long`, `Double`), output-type prefixes (`ToInt`, `ToLong`, `ToDouble`), and conversion patterns (`IntToLong`, `LongToDouble`, etc.).

**Beginner-Friendly Explanation:** Java's regular functional interfaces like `Predicate<T>` and `Function<T, R>` work with objects. But if you use them with numbers, Java has to convert each primitive number into an object (boxing) and back again (unboxing). This conversion takes time. The primitive specializations are just like the regular interfaces, but they work directly with `int`, `long`, `double`, and `boolean` values, making your code faster when working with lots of numbers.

### Key Characteristics

Primitive specializations were introduced in Java SE 8 as part of the `java.util.function` package (JSR 335). They are all annotated with `@FunctionalInterface`. The interfaces follow an extensible naming convention: the basic function shapes (`Predicate`, `Consumer`, `Function`, `Supplier`, `UnaryOperator`, `BinaryOperator`) are modified by primitive type prefixes. The package provides specializations for `int`, `long`, and `double` (and `boolean` for `Supplier`), but not for other primitive types like `byte`, `short`, `char`, or `float`. Many of these interfaces provide default methods for composition (e.g., `and`, `or`, `negate`, `andThen`, `compose`) and static factory methods (e.g., `identity`).

### Prerequisites

Readers should be familiar with Java functional interfaces, lambda expressions, and method references. A basic understanding of Java's type system (primitive types vs. wrapper classes) and the concept of autoboxing is helpful. For advanced sections, familiarity with the Stream API and its primitive stream specializations (`IntStream`, `LongStream`, `DoubleStream`) is beneficial.

### Related Programming Areas with Explanation

Primitive specializations are related to performance optimization (avoiding boxing overhead), the Stream API (where `IntStream`, `LongStream`, and `DoubleStream` use these interfaces extensively), numerical computing (where performance with primitive types is critical), and the broader functional programming model in Java (where these interfaces follow the same composition patterns as their generic counterparts).

## Core Concepts and Key Features

### 1. Primitive Input Specializations

**Core Definition:** Interfaces that accept primitive values as input, including `IntPredicate`, `LongPredicate`, `DoublePredicate`, `IntConsumer`, `LongConsumer`, `DoubleConsumer`, `IntFunction<R>`, `LongFunction<R>`, and `DoubleFunction<R>`.

**Technical Definition:** These interfaces are the primitive-consuming specializations of `Predicate<T>`, `Consumer<T>`, and `Function<T, R>`. They declare functional methods that accept a primitive value (`int`, `long`, or `double`) and either return a `boolean` (predicates), return `void` (consumers), or return a reference type `R` (functions).

**Beginner-Friendly Explanation:** These are like the regular `Predicate`, `Consumer`, and `Function`, but they take primitive numbers as input instead of objects, making them faster for numeric operations.

#### 1.1 `IntPredicate`, `LongPredicate`, `DoublePredicate`

**Definitions**

- **Core Definition:** Primitive specializations of `Predicate<T>` that test a primitive value against a condition and return a `boolean`.
- **Technical Definition:** `IntPredicate` is a functional interface whose functional method is `boolean test(int value)`. `LongPredicate` has `boolean test(long value)`. `DoublePredicate` has `boolean test(double value)`. Each provides default methods `and()`, `or()`, and `negate()` for composition, and a static `not()` method (since Java 11).
- **Beginner-Friendly Explanation:** `IntPredicate` is like `Predicate<Integer>`, but it works directly with `int` values. You use it to test whether a number meets a condition.

**Purposes**

- To test whether a primitive value satisfies a condition.
- To compose multiple conditions using `and`, `or`, and `negate`.
- To filter primitive streams (e.g., `IntStream.filter(IntPredicate)`).

**Syntax Structures and Rules**

Complete general syntax:

```java
IntPredicate predicate = value -> condition;
boolean result = predicate.test(value);

// Composition
IntPredicate combined = predicate1.and(predicate2);
IntPredicate negated = predicate.negate();
```

Component breakdown:

- `test(int value)`: the functional method of `IntPredicate`.
- `and(IntPredicate other)`: returns a composed predicate representing short-circuiting logical AND.
- `or(IntPredicate other)`: returns a composed predicate representing short-circuiting logical OR.
- `negate()`: returns a predicate representing the logical negation.

Syntax rules: `and()`, `or()`, and `negate()` are default methods that return new predicates. `not()` (Java 11+) is a static method that returns a predicate that is the negation of the supplied predicate.

Constraints and limitations: These predicates accept only one primitive type; no `byte`, `short`, `char`, or `float` specializations exist.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic `IntPredicate` and composition**

```java
import java.util.function.IntPredicate;

public class IntPredicateExample {
    public static void main(String[] args) {
        // Simple predicate: is the number even?
        IntPredicate isEven = n -> n % 2 == 0;
        System.out.println("isEven(4): " + isEven.test(4));
        System.out.println("isEven(7): " + isEven.test(7));

        // Composition: even AND positive
        IntPredicate isPositive = n -> n > 0;
        IntPredicate isEvenAndPositive = isEven.and(isPositive);
        System.out.println("4: " + isEvenAndPositive.test(4));
        System.out.println("-4: " + isEvenAndPositive.test(-4));

        // Negation
        IntPredicate isOdd = isEven.negate();
        System.out.println("isOdd(7): " + isOdd.test(7));
    }
}
```

Expected Output:

```
isEven(4): true
isEven(7): false
4: true
-4: false
isOdd(7): true
```

**Why the code produces this result:** The `and()` method composes two predicates into one that returns `true` only if both are satisfied. The `negate()` method inverts the result of `isEven`.

**Example 2: `IntPredicate` with `IntStream`**

```java
import java.util.stream.IntStream;

public class IntStreamFilterExample {
    public static void main(String[] args) {
        int sum = IntStream.rangeClosed(1, 10)
                           .filter(n -> n % 2 == 0)
                           .sum();
        System.out.println("Sum of evens 1-10: " + sum);
    }
}
```

Expected Output:

```
Sum of evens 1-10: 30
```

**Real-World Cases with Explanation**

`IntPredicate` is used in `IntStream.filter()` to select elements matching a condition, in `Collection.removeIf()` when working with `IntStream`, and in numeric validation frameworks.

**References Links**

- IntPredicate (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntPredicate.html
- Package java.util.function (Java SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/function/compact2-package-summary.html

#### 1.2 `IntConsumer`, `LongConsumer`, `DoubleConsumer`

**Definitions**

- **Core Definition:** Primitive specializations of `Consumer<T>` that accept a primitive value and perform an action, returning no result.
- **Technical Definition:** `IntConsumer` is a functional interface whose functional method is `void accept(int value)`. `LongConsumer` has `void accept(long value)`. `DoubleConsumer` has `void accept(double value)`. Each provides a default `andThen()` method for composition.
- **Beginner-Friendly Explanation:** `IntConsumer` is like `Consumer<Integer>`, but it takes an `int` directly. You use it to do something with a number without returning a result.

**Purposes**

- To perform side-effect operations on a primitive value.
- To iterate over primitive streams with `forEach`.
- To compose multiple actions using `andThen`.

**Syntax Structures and Rules**

Complete general syntax:

```java
IntConsumer consumer = value -> { /* action */ };
consumer.accept(value);

// Composition
IntConsumer combined = consumer1.andThen(consumer2);
```

Component breakdown:

- `accept(int value)`: the functional method of `IntConsumer`.
- `andThen(IntConsumer after)`: returns a composed consumer that performs this operation followed by the `after` operation.

Syntax rules: `andThen` performs operations in sequence. If the first operation throws an exception, the second is not performed.

Constraints and limitations: `IntConsumer` is a `@FunctionalInterface` and can be used as a lambda target. It is not a subinterface of `Consumer<Integer>`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic `IntConsumer` and `andThen`**

```java
import java.util.function.IntConsumer;

public class IntConsumerExample {
    public static void main(String[] args) {
        IntConsumer printer = n -> System.out.println("Value: " + n);
        IntConsumer doubler = n -> System.out.println("Double: " + (n * 2));

        IntConsumer combined = printer.andThen(doubler);
        combined.accept(5);
    }
}
```

Expected Output:

```
Value: 5
Double: 10
```

**Example 2: `IntConsumer` with `IntStream.forEach`**

```java
import java.util.stream.IntStream;

public class IntStreamForEachExample {
    public static void main(String[] args) {
        IntStream.rangeClosed(1, 5)
                 .forEach(n -> System.out.print(n + " "));
        System.out.println();
    }
}
```

Expected Output:

```
1 2 3 4 5
```

**Real-World Cases with Explanation**

`IntConsumer` is used in `IntStream.forEach()` to process each element, in logging frameworks that record numeric values, and in accumulation patterns where a mutable state is updated with each primitive value.

**References Links**

- IntConsumer (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntConsumer.html
- Package java.util.function (Java SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/function/compact2-package-summary.html

#### 1.3 `IntFunction<R>`, `LongFunction<R>`, `DoubleFunction<R>`

**Definitions**

- **Core Definition:** Primitive specializations of `Function<T, R>` that accept a primitive value and produce a reference type result.
- **Technical Definition:** `IntFunction<R>` is a functional interface whose functional method is `R apply(int value)`. `LongFunction<R>` has `R apply(long value)`. `DoubleFunction<R>` has `R apply(double value)`. These interfaces have no default methods.
- **Beginner-Friendly Explanation:** `IntFunction<R>` is like `Function<Integer, R>`, but it takes an `int` directly. You use it to transform a number into an object of some type.

**Purposes**

- To transform a primitive value into an object of a reference type.
- To map primitive values in streams to objects (e.g., `IntStream.mapToObj(IntFunction)`).
- To serve as a factory that takes a primitive configuration parameter.

**Syntax Structures and Rules**

Complete general syntax:

```java
IntFunction<R> function = value -> result;
R result = function.apply(value);
```

Component breakdown:

- `apply(int value)`: the functional method of `IntFunction`.

Syntax rules: The return type `R` can be any reference type. There are no composition methods like `andThen` or `compose`.

Constraints and limitations: These interfaces are not subinterfaces of `Function<Integer, R>`. The `apply` method takes a primitive and returns a reference.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `IntFunction` to create a string**

```java
import java.util.function.IntFunction;

public class IntFunctionExample {
    public static void main(String[] args) {
        IntFunction<String> stars = n -> "*".repeat(n);
        System.out.println(stars.apply(5));
    }
}
```

Expected Output:

```
*****
```

**Example 2: `IntFunction` with `IntStream.mapToObj`**

```java
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class IntStreamMapToObjExample {
    public static void main(String[] args) {
        List<String> labels = IntStream.rangeClosed(1, 3)
                                       .mapToObj(n -> "Item " + n)
                                       .collect(Collectors.toList());
        System.out.println(labels);
    }
}
```

Expected Output:

```
[Item 1, Item 2, Item 3]
```

**Real-World Cases with Explanation**

`IntFunction` is used in `IntStream.mapToObj()` to convert primitive values into objects, in array creation (`IntFunction<String[]> generator`), and in factory patterns where a numeric parameter determines the object created.

**References Links**

- IntFunction (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntFunction.html
- Package java.util.function (Java SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/function/compact2-package-summary.html

### 2. Primitive Output Specializations

#### Core Definitions

**Core Definition:** Interfaces that produce primitive values as output from reference type inputs, including `ToIntFunction<T>`, `ToLongFunction<T>`, `ToDoubleFunction<T>`, `ToIntBiFunction<T, U>`, `ToLongBiFunction<T, U>`, and `ToDoubleBiFunction<T, U>`.

**Technical Definition:** These interfaces are the primitive-producing specializations of `Function<T, R>` and `BiFunction<T, U, R>`. They declare functional methods that accept one or two reference-type arguments and return a primitive value.

**Beginner-Friendly Explanation:** These are like `Function<T, Integer>` or `BiFunction<T, U, Integer>`, but they return a primitive `int`, `long`, or `double` directly, avoiding the boxing overhead.

#### 2.1 `ToIntFunction<T>`, `ToLongFunction<T>`, `ToDoubleFunction<T>`

**Definitions**

- **Core Definition:** Primitive specializations of `Function<T, R>` that accept a reference type and produce a primitive result.
- **Technical Definition:** `ToIntFunction<T>` is a functional interface whose functional method is `int applyAsInt(T value)`. `ToLongFunction<T>` has `long applyAsLong(T value)`. `ToDoubleFunction<T>` has `double applyAsDouble(T value)`.
- **Beginner-Friendly Explanation:** `ToIntFunction<T>` is like `Function<T, Integer>`, but it returns an `int` directly. You use it to extract a numeric value from an object.

**Purposes**

- To extract a primitive value from a reference type object.
- To map objects to primitive values in streams (e.g., `Stream.mapToInt(ToIntFunction)`).
- To provide a comparator key extractor that returns a primitive.

**Syntax Structures and Rules**

Complete general syntax:

```java
ToIntFunction<T> function = value -> intResult;
int result = function.applyAsInt(value);
```

Component breakdown:

- `applyAsInt(T value)`: the functional method of `ToIntFunction`.

Syntax rules: The functional method name follows the pattern `applyAs` + primitive type name.

Constraints and limitations: These interfaces have no default or static methods for composition.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `ToIntFunction` to get string length**

```java
import java.util.function.ToIntFunction;

public class ToIntFunctionExample {
    public static void main(String[] args) {
        ToIntFunction<String> length = s -> s.length();
        System.out.println("Length: " + length.applyAsInt("Hello"));
    }
}
```

Expected Output:

```
Length: 5
```

**Example 2: `ToIntFunction` with `Stream.mapToInt`**

```java
import java.util.List;
import java.util.stream.Collectors;

public class MapToIntExample {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "cherry");
        int totalLength = words.stream()
                               .mapToInt(String::length)
                               .sum();
        System.out.println("Total length: " + totalLength);
    }
}
```

Expected Output:

```
Total length: 17
```

**Real-World Cases with Explanation**

`ToIntFunction` is used in `Stream.mapToInt()` to convert objects to primitive int values, in `Comparator.comparingInt(ToIntFunction)` for sorting by a numeric property, and in data aggregation where object properties must be summed.

**References Links**

- ToIntFunction (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/ToIntFunction.html
- Package java.util.function (Java SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/function/compact2-package-summary.html

#### 2.2 `ToIntBiFunction<T, U>`, `ToLongBiFunction<T, U>`, `ToDoubleBiFunction<T, U>`

**Definitions**

- **Core Definition:** Primitive specializations of `BiFunction<T, U, R>` that accept two reference type arguments and produce a primitive result.
- **Technical Definition:** `ToIntBiFunction<T, U>` is a functional interface whose functional method is `int applyAsInt(T t, U u)`. `ToLongBiFunction<T, U>` has `long applyAsLong(T t, U u)`. `ToDoubleBiFunction<T, U>` has `double applyAsDouble(T t, U u)`.
- **Beginner-Friendly Explanation:** These are like `BiFunction<T, U, Integer>`, but they return a primitive value directly from two object arguments.

**Purposes**

- To compute a primitive value from two reference type objects.
- To support two-argument reduction operations that produce primitive results.
- To avoid boxing in two-argument computations.

**Syntax Structures and Rules**

Complete general syntax:

```java
ToIntBiFunction<T, U> function = (t, u) -> intResult;
int result = function.applyAsInt(t, u);
```

Component breakdown:

- `applyAsInt(T t, U u)`: the functional method of `ToIntBiFunction`.

Syntax rules: These interfaces have no default or static methods.

Constraints and limitations: No `ToBooleanBiFunction` exists.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `ToIntBiFunction` to compare lengths**

```java
import java.util.function.ToIntBiFunction;

public class ToIntBiFunctionExample {
    public static void main(String[] args) {
        ToIntBiFunction<String, String> diff = (a, b) -> a.length() - b.length();
        System.out.println("Difference: " + diff.applyAsInt("apple", "banana"));
    }
}
```

Expected Output:

```
Difference: -1
```

**Real-World Cases with Explanation**

`ToIntBiFunction` is used in custom comparison logic where two objects are compared based on a numeric property, and in mathematical operations where two objects produce a primitive result.

**References Links**

- ToIntBiFunction (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/ToIntBiFunction.html

### 3. Primitive-to-Primitive Conversions

**Core Definition:** Interfaces that convert one primitive type directly to another, including `IntToLongFunction`, `IntToDoubleFunction`, `LongToIntFunction`, `LongToDoubleFunction`, `DoubleToIntFunction`, and `DoubleToLongFunction`.

**Technical Definition:** These interfaces declare functional methods that accept one primitive type and return a different primitive type. The naming convention is `SourceToTargetFunction`, where the functional method is `applyAsTarget(Source value)`.

**Beginner-Friendly Explanation:** These are special conversion functions that take one type of number and turn it into another type of number, without going through wrapper objects.

#### 3.1 `IntToLongFunction`, `IntToDoubleFunction`, `LongToIntFunction`, `LongToDoubleFunction`, `DoubleToIntFunction`, `DoubleToLongFunction`

**Definitions**

- **Core Definition:** Functional interfaces that convert between two different primitive types.
- **Technical Definition:** `IntToLongFunction` has functional method `long applyAsLong(int value)`. `IntToDoubleFunction` has `double applyAsDouble(int value)`. `LongToIntFunction` has `int applyAsInt(long value)`. `LongToDoubleFunction` has `double applyAsDouble(long value)`. `DoubleToIntFunction` has `int applyAsInt(double value)`. `DoubleToLongFunction` has `long applyAsLong(double value)`.
- **Beginner-Friendly Explanation:** These are like automatic type converters for numbers. For example, `IntToDoubleFunction` turns an `int` into a `double`.

**Purposes**

- To convert one primitive type to another without boxing.
- To map primitive streams from one type to another (e.g., `IntStream.mapToDouble(IntToDoubleFunction)`).
- To support numeric transformations that change the primitive type.

**Syntax Structures and Rules**

Complete general syntax:

```java
IntToDoubleFunction converter = value -> (double) value;
double result = converter.applyAsDouble(value);
```

Component breakdown:

- `applyAsDouble(int value)`: the functional method of `IntToDoubleFunction`.

Syntax rules: The functional method name follows the `applyAs` + target primitive pattern.

Constraints and limitations: There are no conversions involving `boolean` or `char`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `IntToDoubleFunction`**

```java
import java.util.function.IntToDoubleFunction;

public class IntToDoubleFunctionExample {
    public static void main(String[] args) {
        IntToDoubleFunction half = n -> n / 2.0;
        System.out.println("Half of 7: " + half.applyAsDouble(7));
    }
}
```

Expected Output:

```
Half of 7: 3.5
```

**Example 2: `IntToDoubleFunction` with `IntStream.mapToDouble`**

```java
import java.util.stream.IntStream;

public class MapToDoubleExample {
    public static void main(String[] args) {
        double sum = IntStream.rangeClosed(1, 5)
                              .mapToDouble(n -> n * 0.5)
                              .sum();
        System.out.println("Sum: " + sum);
    }
}
```

Expected Output:

```
Sum: 7.5
```

**Real-World Cases with Explanation**

These conversion functions are used in `IntStream.mapToDouble()`, `DoubleStream.mapToLong()`, and similar stream transformations, as well as in numeric algorithms that require type conversions while maintaining primitive performance.

**References Links**

- IntToDoubleFunction (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntToDoubleFunction.html
- Package java.util.function (Java SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/function/compact2-package-summary.html

### 4. Primitive Operators

**Core Definition:** `IntUnaryOperator`, `LongUnaryOperator`, `DoubleUnaryOperator`, `IntBinaryOperator`, `LongBinaryOperator`, and `DoubleBinaryOperator` are primitive specializations of `UnaryOperator<T>` and `BinaryOperator<T>` that operate on and return primitive types.

**Technical Definition:** `IntUnaryOperator` has functional method `int applyAsInt(int operand)`. `IntBinaryOperator` has `int applyAsInt(int left, int right)`. The long and double variants follow the same pattern. These interfaces provide default composition methods (`andThen`, `compose` for unary; none for binary) and a static `identity()` method for unary operators.

**Beginner-Friendly Explanation:** These are like `UnaryOperator<Integer>` and `BinaryOperator<Integer>`, but they work directly with primitive numbers. You use them for operations like squaring a number or adding two numbers.

#### Sub-features

#### 4.1 `IntUnaryOperator`, `LongUnaryOperator`, `DoubleUnaryOperator`

**Definitions**

- **Core Definition:** Primitive specializations of `UnaryOperator<T>` that take one primitive operand and return a result of the same primitive type.
- **Technical Definition:** `IntUnaryOperator` has functional method `int applyAsInt(int operand)`. It provides default methods `compose(IntUnaryOperator before)` and `andThen(IntUnaryOperator after)`, and a static `identity()` method.
- **Beginner-Friendly Explanation:** `IntUnaryOperator` is a function that takes an `int` and returns an `int`. You use it for type-preserving numeric operations.

**Purposes**

- To perform type-preserving operations on primitive values.
- To compose multiple primitive operations.
- To support operations like negation, incrementing, or scaling.

**Syntax Structures and Rules**

Complete general syntax:

```java
IntUnaryOperator operator = value -> transformedValue;
int result = operator.applyAsInt(value);

// Composition
IntUnaryOperator composed = operator1.andThen(operator2);
IntUnaryOperator identity = IntUnaryOperator.identity();
```

Component breakdown:

- `applyAsInt(int operand)`: the functional method.
- `andThen(IntUnaryOperator after)`: applies this operator first, then the `after` operator.
- `compose(IntUnaryOperator before)`: applies the `before` operator first, then this operator.
- `identity()`: returns an operator that always returns its input.

Syntax rules: The input and output types must be the same primitive type.

Constraints and limitations: `IntUnaryOperator` extends `UnaryOperator<Integer>` but overrides the `apply` method with a primitive version, so it cannot be used where a `Function<Integer, Integer>` is expected.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `IntUnaryOperator` with composition**

```java
import java.util.function.IntUnaryOperator;

public class IntUnaryOperatorExample {
    public static void main(String[] args) {
        IntUnaryOperator doubler = n -> n * 2;
        IntUnaryOperator incrementer = n -> n + 1;

        IntUnaryOperator doubleThenIncrement = doubler.andThen(incrementer);
        System.out.println("(5 * 2) + 1 = " + doubleThenIncrement.applyAsInt(5));

        IntUnaryOperator identity = IntUnaryOperator.identity();
        System.out.println("Identity(42) = " + identity.applyAsInt(42));
    }
}
```

Expected Output:

```
(5 * 2) + 1 = 11
Identity(42) = 42
```

**Real-World Cases with Explanation**

`IntUnaryOperator` is used in `List.replaceAll()` for `ArrayList<Integer>`, in `IntStream.map(IntUnaryOperator)`, and in mathematical transformations that preserve the numeric type.

**References Links**

- IntUnaryOperator (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntUnaryOperator.html
- Package java.util.function (Java SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/function/compact2-package-summary.html

#### 4.2 `IntBinaryOperator`, `LongBinaryOperator`, `DoubleBinaryOperator`

**Definitions**

- **Core Definition:** Primitive specializations of `BinaryOperator<T>` that take two primitive operands of the same type and return a result of that type.
- **Technical Definition:** `IntBinaryOperator` has functional method `int applyAsInt(int left, int right)`. `LongBinaryOperator` has `long applyAsLong(long left, long right)`. `DoubleBinaryOperator` has `double applyAsDouble(double left, double right)`. These interfaces have no default or static methods.
- **Beginner-Friendly Explanation:** `IntBinaryOperator` is a function that takes two `int` values and returns an `int`. You use it for operations like addition, multiplication, or finding the maximum.

**Purposes**

- To combine two primitive values into one of the same type.
- To support reduction operations on primitive streams (e.g., `IntStream.reduce(IntBinaryOperator)`).
- To provide efficient two-argument primitive computations.

**Syntax Structures and Rules**

Complete general syntax:

```java
IntBinaryOperator operator = (a, b) -> result;
int result = operator.applyAsInt(a, b);
```

Component breakdown:

- `applyAsInt(int left, int right)`: the functional method.

Syntax rules: Both operands and the return type must be the same primitive type.

Constraints and limitations: These interfaces do not have `minBy` or `maxBy` factory methods (unlike generic `BinaryOperator<T>`).

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `IntBinaryOperator` with `IntStream.reduce`**

```java
import java.util.stream.IntStream;

public class IntBinaryOperatorExample {
    public static void main(String[] args) {
        int product = IntStream.rangeClosed(1, 5)
                               .reduce(1, (a, b) -> a * b);
        System.out.println("5! = " + product);
    }
}
```

Expected Output:

```
5! = 120
```

**Real-World Cases with Explanation**

`IntBinaryOperator` is used in `IntStream.reduce()` for aggregation, in mathematical libraries for scalar operations, and in performance-critical code where boxing must be avoided.

**References Links**

- IntBinaryOperator (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntBinaryOperator.html
- Package java.util.function (Java SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/function/compact2-package-summary.html

### 5. Primitive Suppliers

**Core Definition:** `IntSupplier`, `LongSupplier`, `DoubleSupplier`, and `BooleanSupplier` are primitive specializations of `Supplier<T>` that take no arguments and return a primitive value.

**Technical Definition:** `IntSupplier` has functional method `int getAsInt()`. `LongSupplier` has `long getAsLong()`. `DoubleSupplier` has `double getAsDouble()`. `BooleanSupplier` has `boolean getAsBoolean()`. There is no `BiSupplier` because a method can only return one value.

**Beginner-Friendly Explanation:** These are like `Supplier<Integer>`, but they return a primitive value directly. You use them when you need to generate a primitive value on demand.

#### 5.1 `IntSupplier`, `LongSupplier`, `DoubleSupplier`, `BooleanSupplier`

**Definitions**

- **Core Definition:** Functional interfaces that supply a primitive result without taking any input.
- **Technical Definition:** Each has a single abstract method named `getAs` + primitive type: `getAsInt()`, `getAsLong()`, `getAsDouble()`, `getAsBoolean()`.
- **Beginner-Friendly Explanation:** A `IntSupplier` is a value producer for `int` values. You call `getAsInt()` and it gives you a number.

**Purposes**

- To generate primitive values on demand.
- To provide primitive values for lazy initialization.
- To serve as factories for primitive values in streams and concurrent code.

**Syntax Structures and Rules**

Complete general syntax:

```java
IntSupplier supplier = () -> value;
int result = supplier.getAsInt();
```

Component breakdown:

- `getAsInt()`: the functional method of `IntSupplier`.

Syntax rules: The `getAsInt()` method takes no arguments. There is no requirement that the same value be returned each time.

Constraints and limitations: `BooleanSupplier` is the only supplier for a non-numeric primitive type.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `IntSupplier` and `DoubleSupplier`**

```java
import java.util.function.IntSupplier;
import java.util.function.DoubleSupplier;

public class SupplierExample {
    public static void main(String[] args) {
        IntSupplier counter = new IntSupplier() {
            private int count = 0;
            @Override
            public int getAsInt() { return ++count; }
        };
        System.out.println("Counter: " + counter.getAsInt());
        System.out.println("Counter: " + counter.getAsInt());

        DoubleSupplier random = Math::random;
        System.out.println("Random >= 0: " + (random.getAsDouble() >= 0));
    }
}
```

Expected Output:

```
Counter: 1
Counter: 2
Random >= 0: true
```

**Real-World Cases with Explanation**

`IntSupplier` is used in `IntStream.generate(IntSupplier)` to create infinite primitive streams, in `OptionalInt.orElseGet(IntSupplier)` for lazy defaults, and in benchmarks that need to generate primitive test data.

**References Links**

- IntSupplier (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntSupplier.html
- BooleanSupplier (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/BooleanSupplier.html

### 6. Two-Argument Primitive/Object Combinations

**Core Definition:** `ObjIntConsumer<T>`, `ObjLongConsumer<T>`, and `ObjDoubleConsumer<T>` are functional interfaces that accept one reference type argument and one primitive argument, and return no result.

**Technical Definition:** `ObjIntConsumer<T>` has functional method `void accept(T t, int value)`. `ObjLongConsumer<T>` has `void accept(T t, long value)`. `ObjDoubleConsumer<T>` has `void accept(T t, double value)`. These are the `(reference, primitive)` specializations of `BiConsumer<T, U>`.

**Beginner-Friendly Explanation:** These are like `BiConsumer<T, Integer>`, but they take an object and a primitive number, and perform an action without returning a result.

#### 6.1 `ObjIntConsumer<T>`, `ObjLongConsumer<T>`, `ObjDoubleConsumer<T>`

**Definitions**

- **Core Definition:** Functional interfaces that consume one object and one primitive value, returning no result.
- **Technical Definition:** Each is a `@FunctionalInterface` with a single abstract method `accept(T t, primitive value)`. They have no default or static methods.
- **Beginner-Friendly Explanation:** You use these when you need to perform an action that combines an object with a primitive number, like updating a map entry.

**Purposes**

- To perform an operation combining an object and a primitive value.
- To support iteration over map entries with primitive values.
- To provide a primitive-aware alternative to `BiConsumer<T, U>`.

**Syntax Structures and Rules**

Complete general syntax:

```java
ObjIntConsumer<T> consumer = (obj, value) -> { /* action */ };
consumer.accept(obj, value);
```

Component breakdown:

- `accept(T t, int value)`: the functional method of `ObjIntConsumer`.

Syntax rules: The primitive argument must be the second parameter. There are no composition methods.

Constraints and limitations: These interfaces do not accept two objects or two primitives.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `ObjIntConsumer` with a map**

```java
import java.util.HashMap;
import java.util.Map;
import java.util.function.ObjIntConsumer;

public class ObjIntConsumerExample {
    public static void main(String[] args) {
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 87);

        ObjIntConsumer<String> printer = (name, score) ->
            System.out.println(name + " scored " + score);

        scores.forEach(printer);
    }
}
```

Expected Output:

```
Alice scored 95
Bob scored 87
```

**Real-World Cases with Explanation**

`ObjIntConsumer` is used in `Map.forEach()` when the map's value type is `Integer`, in event systems where an object and a numeric code are processed together, and in data processing pipelines that combine object and primitive data.

**References Links**

- ObjIntConsumer (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/ObjIntConsumer.html
- Package java.util.function (Java SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/function/compact2-package-summary.html

## Summary Table of Primitive Specializations

| Category | Interface | Functional Method | Input | Output |
|---|---|---|---|---|
| Predicate (input) | `IntPredicate` | `test(int)` | int | boolean |
| Predicate (input) | `LongPredicate` | `test(long)` | long | boolean |
| Predicate (input) | `DoublePredicate` | `test(double)` | double | boolean |
| Consumer (input) | `IntConsumer` | `accept(int)` | int | void |
| Consumer (input) | `LongConsumer` | `accept(long)` | long | void |
| Consumer (input) | `DoubleConsumer` | `accept(double)` | double | void |
| Function (input) | `IntFunction<R>` | `apply(int)` | int | R |
| Function (input) | `LongFunction<R>` | `apply(long)` | long | R |
| Function (input) | `DoubleFunction<R>` | `apply(double)` | double | R |
| Function (output) | `ToIntFunction<T>` | `applyAsInt(T)` | T | int |
| Function (output) | `ToLongFunction<T>` | `applyAsLong(T)` | T | long |
| Function (output) | `ToDoubleFunction<T>` | `applyAsDouble(T)` | T | double |
| BiFunction (output) | `ToIntBiFunction<T,U>` | `applyAsInt(T,U)` | T, U | int |
| BiFunction (output) | `ToLongBiFunction<T,U>` | `applyAsLong(T,U)` | T, U | long |
| BiFunction (output) | `ToDoubleBiFunction<T,U>` | `applyAsDouble(T,U)` | T, U | double |
| Conversion | `IntToLongFunction` | `applyAsLong(int)` | int | long |
| Conversion | `IntToDoubleFunction` | `applyAsDouble(int)` | int | double |
| Conversion | `LongToIntFunction` | `applyAsInt(long)` | long | int |
| Conversion | `LongToDoubleFunction` | `applyAsDouble(long)` | long | double |
| Conversion | `DoubleToIntFunction` | `applyAsInt(double)` | double | int |
| Conversion | `DoubleToLongFunction` | `applyAsLong(double)` | double | long |
| UnaryOperator | `IntUnaryOperator` | `applyAsInt(int)` | int | int |
| UnaryOperator | `LongUnaryOperator` | `applyAsLong(long)` | long | long |
| UnaryOperator | `DoubleUnaryOperator` | `applyAsDouble(double)` | double | double |
| BinaryOperator | `IntBinaryOperator` | `applyAsInt(int,int)` | int, int | int |
| BinaryOperator | `LongBinaryOperator` | `applyAsLong(long,long)` | long, long | long |
| BinaryOperator | `DoubleBinaryOperator` | `applyAsDouble(double,double)` | double, double | double |
| Supplier | `IntSupplier` | `getAsInt()` | — | int |
| Supplier | `LongSupplier` | `getAsLong()` | — | long |
| Supplier | `DoubleSupplier` | `getAsDouble()` | — | double |
| Supplier | `BooleanSupplier` | `getAsBoolean()` | — | boolean |
| Obj Consumer | `ObjIntConsumer<T>` | `accept(T, int)` | T, int | void |
| Obj Consumer | `ObjLongConsumer<T>` | `accept(T, long)` | T, long | void |
| Obj Consumer | `ObjDoubleConsumer<T>` | `accept(T, double)` | T, double | void |

## Deprecated, Unsafe, and Version-Specific Notes

- All primitive specializations were introduced in Java SE 8 (JSR 335) as part of the `java.util.function` package.
- The `Predicate.not()` static method was introduced in Java 11. Prior to Java 11, `negate()` was the only way to obtain a negated predicate.
- The `BooleanSupplier` interface is the only primitive supplier for a non-numeric type. There is no `ByteSupplier`, `ShortSupplier`, `CharSupplier`, or `FloatSupplier`.
- Primitive specializations do not support `byte`, `short`, `char`, or `float` types. For these types, the generic interfaces (with boxing) must be used.
- The `IntUnaryOperator`, `LongUnaryOperator`, and `DoubleUnaryOperator` interfaces extend `UnaryOperator<Integer>`, `UnaryOperator<Long>`, and `UnaryOperator<Double>` respectively, but override the `apply` method with a primitive version. They cannot be used where a `Function<Integer, Integer>` is expected.
- The `IntFunction<R>`, `LongFunction<R>`, and `DoubleFunction<R>` interfaces are not subinterfaces of `Function<Integer, R>`; they are independent functional interfaces.
- There are no `BiSupplier` or `ToBooleanBiFunction` interfaces because a method can only return one value.
- These interfaces are annotated with `@FunctionalInterface` and can be used as lambda expression targets.

## References

- Package java.util.function (Java SE 8 API Documentation) – https://docs.oracle.com/javase/8/docs/api/java/util/function/compact2-package-summary.html
- Package java.util.function (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html
- IntPredicate (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntPredicate.html
- IntConsumer (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntConsumer.html
- IntFunction (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntFunction.html
- ToIntFunction (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/ToIntFunction.html
- IntUnaryOperator (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntUnaryOperator.html
- IntBinaryOperator (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntBinaryOperator.html
- IntSupplier (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/IntSupplier.html
- BooleanSupplier (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/BooleanSupplier.html
- ObjIntConsumer (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/ObjIntConsumer.html
- Why different predicate interfaces in Java 8? (Stack Overflow) – https://browse.library.kiwix.org