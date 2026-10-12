# Java Functional Interfaces: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A functional interface is an interface that contains exactly one abstract method, representing a single function contract that can be implemented using lambda expressions, method references, or constructor references.

**Technical Definition:** "A functional interface is an interface that has just one abstract method (aside from the methods of Object), and thus represents a single function contract". "Conceptually, a functional interface has exactly one abstract method. Since default methods have an implementation, they are not abstract. If an interface declares an abstract method overriding one of the public methods of java.lang.Object, that also does not count toward the interface's abstract method count". "Functional interfaces provide target types for lambda expressions and method references".

**Beginner-Friendly Explanation:** Think of a functional interface as a job description with exactly one task. Anyone (a lambda expression) who can perform that one task qualifies for the job. The interface defines what the task is; the lambda provides how to do it. For example, a `Comparator` interface says "compare two objects" — a lambda can fill in the specific comparison logic.

### Key Characteristics

- **Single Abstract Method (SAM):** Exactly one abstract method defines the function contract.
- **Target for Lambdas:** Functional interfaces are the target types for lambda expressions and method references.
- **Default and Static Methods Allowed:** Despite having only one abstract method, a functional interface may contain any number of default and static methods.
- **@FunctionalInterface Annotation:** An optional annotation that ensures compile-time checking of the SAM constraint.
- **Built-in and Custom:** Java provides a rich set of built-in functional interfaces in `java.util.function`, and developers can create custom ones.
- **Primitive Specializations:** Specialised interfaces exist to avoid boxing/unboxing overhead for `int`, `long`, and `double`.

### Prerequisites

- Basic understanding of Java interfaces and abstract methods
- Familiarity with lambda expressions and method references
- Knowledge of generics (type parameters)
- Understanding of the `java.util.function` package

### Related Programming Areas

- **Lambda Expressions:** Functional interfaces are the target types for lambdas.
- **Stream API:** The Streams API relies heavily on functional interfaces like `Function`, `Predicate`, and `Consumer`.
- **Optional:** `Optional` methods accept `Function`, `Predicate`, and `Supplier`.
- **Concurrency:** `Runnable`, `Callable`, and `CompletableFuture` use functional interfaces.
- **Design Patterns:** Strategy, Template Method, and Factory patterns can be implemented with functional interfaces.

### Core Concepts / Features

---

## 1. `@FunctionalInterface`

### Definitions

**Core Definition:** `@FunctionalInterface` is an informative annotation that indicates an interface type declaration is intended to be a functional interface as defined by the Java Language Specification.

**Technical Definition:** "An informative annotation type used to indicate that an interface type declaration is intended to be a functional interface as defined by the Java Language Specification. If a type is annotated with this annotation type, compilers are required to generate an error message unless: the type is an interface type and not an annotation type, enum, or class; the annotated type satisfies the requirements of a functional interface. However, the compiler will treat any interface meeting the definition of a functional interface as a functional interface regardless of whether or not a FunctionalInterface annotation is present on the interface declaration".

**Beginner-Friendly Explanation:** `@FunctionalInterface` is like a label you put on an interface to say "this is meant to have exactly one abstract method." It's not required, but it helps catch mistakes at compile time if you accidentally add a second abstract method.

### Purposes

- To explicitly document that an interface is intended to be a functional interface
- To enable compile-time verification of the single abstract method constraint
- To improve code readability and communicate design intent

### Syntax Rules and Structure

**Complete General Syntax:**

```java
@FunctionalInterface
public interface InterfaceName {
    returnType methodName(parameters);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@FunctionalInterface` | Annotation marking the interface as a functional interface. |
| Single abstract method | Exactly one abstract method; default and static methods are allowed. |
| Default methods | Allowed; they have implementations and do not count toward the abstract method count. |
| Static methods | Allowed; they belong to the interface and do not count toward the abstract method count. |

**Syntax Rules:**

- The annotation is optional but recommended for clarity.
- The interface must have exactly one abstract method.
- Methods of `java.lang.Object` (e.g., `equals`, `hashCode`) do not count toward the abstract method count.
- Default and static methods do not count toward the abstract method count.

**Constraints and Limitations:**

- The annotation cannot be applied to classes, enums, or annotation types.
- A compile-time error occurs if the annotated interface has zero or more than one abstract method.
- The compiler treats any interface meeting the functional interface definition as functional, regardless of the annotation.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Valid Functional Interface**

```java
// File: ValidFunctionalInterface.java
@FunctionalInterface
public interface ValidFunctionalInterface {

    // Single abstract method — the functional method
    int calculate(int a, int b);

    // Default method — does not count toward abstract method count
    default int calculateWithMessage(int a, int b) {
        System.out.println("Calculating...");
        return calculate(a, b);
    }

    // Static method — does not count toward abstract method count
    static ValidFunctionalInterface addition() {
        return (a, b) -> a + b;
    }
}
```

**Expected Output:** No output (compiles successfully).

**Step-by-Step Explanation:**

1. `@FunctionalInterface` is applied to the interface.
2. The interface has exactly one abstract method: `calculate(int, int)`.
3. The `default` method `calculateWithMessage` and the `static` method `addition` are allowed because they are not abstract.
4. The compiler verifies the SAM constraint and produces no errors.

**Example 2: Invalid Functional Interface (Compile Error)**

```java
// File: InvalidFunctionalInterface.java
@FunctionalInterface
public interface InvalidFunctionalInterface {
    int calculate(int a, int b);
    // int anotherMethod();  // COMPILE ERROR: multiple abstract methods
}
```

**Expected Output:** Compile error: "InvalidFunctionalInterface is not a functional interface: multiple non-overriding abstract methods found."

**Step-by-Step Explanation:**

1. The interface is annotated with `@FunctionalInterface`.
2. It has one abstract method `calculate`.
3. If a second abstract method `anotherMethod` were added, the compiler would produce an error because the interface would no longer satisfy the SAM constraint.

### Real-World Cases

- **JDK Source Code:** Interfaces like `Runnable`, `Callable`, `Comparator`, and all interfaces in `java.util.function` are annotated with `@FunctionalInterface`.
- **API Design:** Library authors use `@FunctionalInterface` to document callback interfaces and ensure they remain functional.
- **Code Reviews:** The annotation serves as a documentation aid, making the functional nature of an interface explicit.

---

## 2. Built-in Functional Interfaces

Java provides a rich set of built-in functional interfaces in the `java.util.function` package, designed to cover the most common function shapes.

### 2.1 `Function<T, R>`

#### Definitions

**Core Definition:** `Function<T, R>` represents a function that accepts one argument of type `T` and produces a result of type `R`.

**Technical Definition:** "Represents a function that accepts one argument and produces a result. This is a functional interface whose functional method is `apply(Object)`".

**Beginner-Friendly Explanation:** `Function` is a transformer — you give it something, and it gives you back something else. For example, you give it a `String` and it gives you back its length as an `Integer`.

#### Purposes

- To transform an input value into an output value
- To map elements in a stream from one type to another
- To compose reusable transformation pipelines

#### Syntax Rules and Structure

**Complete General Syntax:**

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `T` | The type of the input to the function. |
| `R` | The type of the result of the function. |
| `apply(T t)` | The functional method that applies the function to the given argument. |

**Syntax Rules:**

- The lambda takes one parameter of type `T` and returns a value of type `R`.
- The `apply` method is the functional method.
- Additional default methods include `andThen`, `compose`, and `identity`.

**Constraints and Limitations:**

- Input and output types are fixed by the type parameters.
- Boxing/unboxing overhead occurs when using primitive types with generic `Function`.

#### Annotated Code Example

```java
// File: FunctionDemo.java
import java.util.function.Function;

public class FunctionDemo {
    public static void main(String[] args) {
        // Function: String -> Integer (length)
        Function<String, Integer> length = s -> s.length();
        System.out.println("Length: " + length.apply("Hello"));  // Output: 5

        // Function: Integer -> Integer (square)
        Function<Integer, Integer> square = x -> x * x;
        System.out.println("Square: " + square.apply(4));  // Output: 16

        // Composition: length then square
        Function<String, Integer> lengthThenSquare = length.andThen(square);
        System.out.println("Length squared: " + lengthThenSquare.apply("Hello"));  // Output: 25

        // Composition: square then length (compose)
        Function<Integer, Integer> squareThenLength = length.compose(square);
        // Note: types must align for compose
    }
}
```

**Expected Output:**
```
Length: 5
Square: 16
Length squared: 25
```

**Step-by-Step Explanation:**

1. `length` is a `Function<String, Integer>` that returns the length of a string.
2. `square` is a `Function<Integer, Integer>` that squares an integer.
3. `length.andThen(square)` composes the two functions: first `length` is applied, then `square`.
4. `lengthThenSquare.apply("Hello")` computes `5` then `25`.

#### Real-World Cases

- **Stream Mapping:** `stream.map(String::length)` uses `Function<String, Integer>`.
- **Optional Mapping:** `optional.map(s -> s.toUpperCase())` uses `Function<T, R>`.
- **Data Transformation:** Converting entities to DTOs in enterprise applications.

---

### 2.2 `BiFunction<T, U, R>`

#### Definitions

**Core Definition:** `BiFunction<T, U, R>` represents a function that accepts two arguments of types `T` and `U` and produces a result of type `R`.

**Technical Definition:** "Represents a function that accepts two arguments and produces a result. This is the two-arity specialization of `Function`".

**Beginner-Friendly Explanation:** `BiFunction` is like `Function` but takes two inputs instead of one. For example, you give it two numbers and it returns their sum.

#### Purposes

- To transform two input values into a single output value
- To perform binary operations with type flexibility
- To combine values from two sources

#### Syntax Rules and Structure

**Complete General Syntax:**

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `T` | The type of the first argument. |
| `U` | The type of the second argument. |
| `R` | The type of the result. |
| `apply(T t, U u)` | The functional method. |

**Constraints and Limitations:**

- Two inputs, one output; cannot return multiple values.
- Primitive specializations exist as `ToIntBiFunction`, `ToLongBiFunction`, `ToDoubleBiFunction`.

#### Annotated Code Example

```java
// File: BiFunctionDemo.java
import java.util.function.BiFunction;

public class BiFunctionDemo {
    public static void main(String[] args) {
        // BiFunction: (String, Integer) -> String
        BiFunction<String, Integer, String> repeat = (s, n) -> s.repeat(n);
        System.out.println(repeat.apply("Hi", 3));  // Output: HiHiHi

        // BiFunction: (Integer, Integer) -> Integer
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        System.out.println("Sum: " + add.apply(10, 20));  // Output: 30

        // Composition with andThen
        BiFunction<Integer, Integer, Integer> multiply = (a, b) -> a * b;
        BiFunction<Integer, Integer, Integer> multiplyThenAdd = multiply.andThen(x -> x + 1);
        System.out.println("Result: " + multiplyThenAdd.apply(3, 4));  // Output: 13
    }
}
```

**Expected Output:**
```
HiHiHi
Sum: 30
Result: 13
```

**Step-by-Step Explanation:**

1. `repeat` takes a `String` and an `Integer` and repeats the string `n` times.
2. `add` takes two integers and returns their sum.
3. `multiply.andThen(x -> x + 1)` composes multiplication with addition of 1: `3 * 4 = 12`, then `12 + 1 = 13`.

#### Real-World Cases

- **Map Merging:** `map.merge(key, value, (v1, v2) -> v1 + v2)` uses `BiFunction`.
- **Reduction:** `stream.reduce(identity, accumulator)` uses `BinaryOperator`, a `BiFunction` variant.
- **Calculator Operations:** Binary arithmetic operations.

---

### 2.3 `UnaryOperator<T>`

#### Definitions

**Core Definition:** `UnaryOperator<T>` is a specialization of `Function<T, T>` where the input and output types are the same.

**Technical Definition:** "Represents an operation on a single operand that produces a result of the same type as its operand. This is a specialization of `Function` for the case where the operand and result are of the same type".

**Beginner-Friendly Explanation:** `UnaryOperator` is like `Function` but the input and output are the same type. For example, a function that takes a `String` and returns a modified `String`.

#### Purposes

- To perform operations that transform a value into another value of the same type
- To replace elements in a list with modified versions
- To provide a more type-safe alternative to `Function<T, T>`

#### Syntax Rules and Structure

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `T` | The type of the operand and the result. |
| `apply(T t)` | Inherited from `Function`; returns a `T`. |
| `identity()` | Static factory method returning a unary operator that returns its input unchanged. |

#### Annotated Code Example

```java
// File: UnaryOperatorDemo.java
import java.util.function.UnaryOperator;

public class UnaryOperatorDemo {
    public static void main(String[] args) {
        // UnaryOperator: String -> String (uppercase)
        UnaryOperator<String> upper = s -> s.toUpperCase();
        System.out.println(upper.apply("hello"));  // Output: HELLO

        // UnaryOperator: Integer -> Integer (double)
        UnaryOperator<Integer> doubleIt = x -> x * 2;
        System.out.println("Double: " + doubleIt.apply(5));  // Output: 10

        // Identity operator
        UnaryOperator<String> identity = UnaryOperator.identity();
        System.out.println("Identity: " + identity.apply("unchanged"));  // Output: unchanged
    }
}
```

**Expected Output:**
```
HELLO
Double: 10
Identity: unchanged
```

**Step-by-Step Explanation:**

1. `upper` takes a `String` and returns its uppercase version.
2. `doubleIt` takes an `Integer` and returns double its value.
3. `UnaryOperator.identity()` returns a function that returns its input unchanged.

#### Real-World Cases

- **List Replacement:** `list.replaceAll(s -> s.trim())` uses `UnaryOperator<String>`.
- **String Processing:** Trimming, formatting, or normalising strings.
- **Numerical Transformations:** Applying mathematical operations.

---

### 2.4 `BinaryOperator<T>`

#### Definitions

**Core Definition:** `BinaryOperator<T>` is a specialization of `BiFunction<T, T, T>` where both operands and the result are of the same type.

**Technical Definition:** "Represents an operation upon two operands of the same type, producing a result of the same type as the operands".

**Beginner-Friendly Explanation:** `BinaryOperator` takes two values of the same type and returns a result of the same type. For example, adding two numbers or concatenating two strings.

#### Purposes

- To combine two values of the same type into one
- To serve as accumulator functions in reductions
- To find minimum or maximum values

#### Syntax Rules and Structure

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) { ... }
    static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) { ... }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `T` | The type of both operands and the result. |
| `apply(T t, T u)` | Inherited from `BiFunction`; returns a `T`. |
| `minBy(Comparator)` | Returns a binary operator that returns the lesser of two elements. |
| `maxBy(Comparator)` | Returns a binary operator that returns the greater of two elements. |

#### Annotated Code Example

```java
// File: BinaryOperatorDemo.java
import java.util.function.BinaryOperator;
import java.util.Comparator;

public class BinaryOperatorDemo {
    public static void main(String[] args) {
        // BinaryOperator: Integer addition
        BinaryOperator<Integer> add = (a, b) -> a + b;
        System.out.println("Sum: " + add.apply(10, 20));  // Output: 30

        // BinaryOperator: String concatenation
        BinaryOperator<String> concat = (a, b) -> a + b;
        System.out.println(concat.apply("Hello, ", "World!"));  // Output: Hello, World!

        // minBy and maxBy
        BinaryOperator<Integer> min = BinaryOperator.minBy(Comparator.naturalOrder());
        BinaryOperator<Integer> max = BinaryOperator.maxBy(Comparator.naturalOrder());
        System.out.println("Min: " + min.apply(5, 3));  // Output: 3
        System.out.println("Max: " + max.apply(5, 3));  // Output: 5
    }
}
```

**Expected Output:**
```
Sum: 30
Hello, World!
Min: 3
Max: 5
```

**Step-by-Step Explanation:**

1. `add` takes two integers and returns their sum.
2. `concat` takes two strings and returns their concatenation.
3. `BinaryOperator.minBy` and `maxBy` return operators that select the minimum or maximum of two values based on a comparator.

#### Real-World Cases

- **Stream Reduction:** `stream.reduce(0, (a, b) -> a + b)` uses `BinaryOperator<Integer>`.
- **Map Merging:** `map.merge(key, value, (v1, v2) -> v1 + v2)`.
- **Finding Extremes:** `stream.reduce(BinaryOperator.maxBy(Comparator.naturalOrder()))`.

---

### 2.5 `Predicate<T>`

#### Definitions

**Core Definition:** `Predicate<T>` represents a boolean-valued function of one argument.

**Technical Definition:** "Represents a predicate (boolean-valued function) of one argument. This is a functional interface whose functional method is `test(Object)`".

**Beginner-Friendly Explanation:** `Predicate` is a test — you give it a value, and it tells you `true` or `false`. For example, "is this string empty?" or "is this number positive?"

#### Purposes

- To test a condition on a value
- To filter elements in a stream
- To combine conditions using logical operators (`and`, `or`, `negate`)

#### Syntax Rules and Structure

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    default Predicate<T> and(Predicate<? super T> other) { ... }
    default Predicate<T> or(Predicate<? super T> other) { ... }
    default Predicate<T> negate() { ... }
    static <T> Predicate<T> isEqual(Object targetRef) { ... }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `T` | The type of the input to the predicate. |
| `test(T t)` | The functional method that evaluates the predicate on the given argument. |
| `and(Predicate)` | Returns a composed predicate representing a short-circuiting logical AND. |
| `or(Predicate)` | Returns a composed predicate representing a short-circuiting logical OR. |
| `negate()` | Returns a predicate representing the logical negation. |

#### Annotated Code Example

```java
// File: PredicateDemo.java
import java.util.function.Predicate;

public class PredicateDemo {
    public static void main(String[] args) {
        // Predicate: String is empty
        Predicate<String> isEmpty = s -> s.isEmpty();
        System.out.println("Empty: " + isEmpty.test(""));       // Output: true
        System.out.println("Empty: " + isEmpty.test("Hello"));  // Output: false

        // Predicate: Integer is positive
        Predicate<Integer> isPositive = n -> n > 0;
        System.out.println("Positive: " + isPositive.test(5));   // Output: true
        System.out.println("Positive: " + isPositive.test(-3));  // Output: false

        // Combining predicates
        Predicate<String> isNotEmpty = isEmpty.negate();
        Predicate<String> startsWithA = s -> s.startsWith("A");
        Predicate<String> startsWithAAndNotEmpty = startsWithA.and(isNotEmpty);

        System.out.println(startsWithAAndNotEmpty.test("Apple"));  // Output: true
        System.out.println(startsWithAAndNotEmpty.test(""));       // Output: false
        System.out.println(startsWithAAndNotEmpty.test("Banana")); // Output: false
    }
}
```

**Expected Output:**
```
Empty: true
Empty: false
Positive: true
Positive: false
true
false
false
```

**Step-by-Step Explanation:**

1. `isEmpty` tests whether a string is empty.
2. `isPositive` tests whether an integer is positive.
3. `isEmpty.negate()` creates a predicate that is the logical opposite.
4. `startsWithA.and(isNotEmpty)` combines two predicates with AND.
5. The combined predicate returns `true` only when both conditions are met.

#### Real-World Cases

- **Stream Filtering:** `stream.filter(s -> s.startsWith("A"))` uses `Predicate<String>`.
- **Collection Removal:** `list.removeIf(s -> s.isEmpty())` uses `Predicate<String>`.
- **Validation:** `Optional.filter(predicate)` uses `Predicate<T>`.

---

### 2.6 `BiPredicate<T, U>`

#### Definitions

**Core Definition:** `BiPredicate<T, U>` represents a boolean-valued function of two arguments.

**Technical Definition:** "Represents a predicate (boolean-valued function) of two arguments. This is the two-arity specialization of `Predicate`".

**Beginner-Friendly Explanation:** `BiPredicate` is like `Predicate` but takes two inputs. For example, "is this string longer than this number?" or "are these two values equal?"

#### Purposes

- To test a condition involving two values
- To compare two objects
- To validate pairs of values

#### Syntax Rules and Structure

```java
@FunctionalInterface
public interface BiPredicate<T, U> {
    boolean test(T t, U u);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `T` | The type of the first argument. |
| `U` | The type of the second argument. |
| `test(T t, U u)` | The functional method. |

#### Annotated Code Example

```java
// File: BiPredicateDemo.java
import java.util.function.BiPredicate;

public class BiPredicateDemo {
    public static void main(String[] args) {
        // BiPredicate: check if string length equals integer
        BiPredicate<String, Integer> lengthEquals = (s, n) -> s.length() == n;
        System.out.println(lengthEquals.test("Hello", 5));  // Output: true
        System.out.println(lengthEquals.test("Hi", 5));     // Output: false

        // BiPredicate: check if two strings are equal ignoring case
        BiPredicate<String, String> equalsIgnoreCase = (a, b) -> a.equalsIgnoreCase(b);
        System.out.println(equalsIgnoreCase.test("Java", "JAVA"));  // Output: true
    }
}
```

**Expected Output:**
```
true
false
true
```

**Step-by-Step Explanation:**

1. `lengthEquals` checks if the length of a string equals an integer.
2. `equalsIgnoreCase` compares two strings ignoring case.

#### Real-World Cases

- **Map Operations:** `map.remove(key, value)` uses `BiPredicate` internally.
- **Validation:** Checking if two fields match.
- **Filtering Pairs:** Filtering streams of key-value pairs.

---

### 2.7 `Consumer<T>`

#### Definitions

**Core Definition:** `Consumer<T>` represents an operation that accepts a single input argument and returns no result.

**Technical Definition:** "Represents an operation that accepts a single input argument and returns no result. Unlike most other functional interfaces, `Consumer` is expected to operate via side-effects".

**Beginner-Friendly Explanation:** `Consumer` is an action — you give it something, and it does something with it, but it doesn't give anything back. For example, printing a value or saving it to a database.

#### Purposes

- To perform side effects on a value
- To process elements in a stream
- To iterate over collections

#### Syntax Rules and Structure

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    default Consumer<T> andThen(Consumer<? super T> after) { ... }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `T` | The type of the input to the operation. |
| `accept(T t)` | The functional method that performs the operation on the given argument. |
| `andThen(Consumer)` | Returns a composed consumer that performs this operation followed by the after operation. |

#### Annotated Code Example

```java
// File: ConsumerDemo.java
import java.util.function.Consumer;
import java.util.List;

public class ConsumerDemo {
    public static void main(String[] args) {
        // Consumer: print a string
        Consumer<String> printer = s -> System.out.println("Value: " + s);
        printer.accept("Hello");  // Output: Value: Hello

        // Consumer: print and then log
        Consumer<String> logger = s -> System.out.println("Logged: " + s);
        Consumer<String> printAndLog = printer.andThen(logger);
        printAndLog.accept("World");
        // Output: Value: World
        //         Logged: World

        // forEach uses Consumer
        List<String> names = List.of("Alice", "Bob", "Charlie");
        names.forEach(s -> System.out.println("Name: " + s));
        // Output: Name: Alice
        //         Name: Bob
        //         Name: Charlie
    }
}
```

**Expected Output:**
```
Value: Hello
Value: World
Logged: World
Name: Alice
Name: Bob
Name: Charlie
```

**Step-by-Step Explanation:**

1. `printer` is a `Consumer<String>` that prints a value.
2. `logger` is another consumer that logs a value.
3. `printer.andThen(logger)` creates a consumer that performs both actions in sequence.
4. `names.forEach(...)` uses a `Consumer<String>` to process each element.

#### Real-World Cases

- **Stream forEach:** `stream.forEach(System.out::println)` uses `Consumer<T>`.
- **Optional ifPresent:** `optional.ifPresent(value -> save(value))` uses `Consumer<T>`.
- **List Iteration:** `list.forEach(item -> process(item))`.

---

### 2.8 `BiConsumer<T, U>`

#### Definitions

**Core Definition:** `BiConsumer<T, U>` represents an operation that accepts two input arguments and returns no result.

**Technical Definition:** "Represents an operation that accepts two input arguments and returns no result. This is the two-arity specialization of `Consumer`".

**Beginner-Friendly Explanation:** `BiConsumer` is like `Consumer` but takes two inputs. For example, processing a key-value pair.

#### Purposes

- To process two values with a side effect
- To iterate over map entries
- To handle pairs of values

#### Syntax Rules and Structure

```java
@FunctionalInterface
public interface BiConsumer<T, U> {
    void accept(T t, U u);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `T` | The type of the first argument. |
| `U` | The type of the second argument. |
| `accept(T t, U u)` | The functional method. |

#### Annotated Code Example

```java
// File: BiConsumerDemo.java
import java.util.function.BiConsumer;
import java.util.Map;
import java.util.HashMap;

public class BiConsumerDemo {
    public static void main(String[] args) {
        // BiConsumer: print key-value pair
        BiConsumer<String, Integer> printer = (k, v) -> System.out.println(k + " -> " + v);
        printer.accept("Age", 30);  // Output: Age -> 30

        // Map.forEach uses BiConsumer
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 87);
        scores.put("Charlie", 92);

        scores.forEach((name, score) -> System.out.println(name + ": " + score));
        // Output: Alice: 95
        //         Bob: 87
        //         Charlie: 92
    }
}
```

**Expected Output:**
```
Age -> 30
Alice: 95
Bob: 87
Charlie: 92
```

**Step-by-Step Explanation:**

1. `printer` is a `BiConsumer<String, Integer>` that prints a key-value pair.
2. `Map.forEach` accepts a `BiConsumer<K, V>` to process each entry.

#### Real-World Cases

- **Map Iteration:** `map.forEach((k, v) -> ...)` uses `BiConsumer<K, V>`.
- **Event Handling:** Processing events with two pieces of data.
- **Pair Processing:** Handling key-value pairs from configuration.

---

### 2.9 `Supplier<T>`

#### Definitions

**Core Definition:** `Supplier<T>` represents a supplier of results, taking no arguments and returning a value of type `T`.

**Technical Definition:** "Represents a supplier of results. There is no requirement that a new or distinct result be returned each time the supplier is invoked".

**Beginner-Friendly Explanation:** `Supplier` is a factory — you ask it for something, and it gives you something, but you don't provide any input. For example, "give me a random number" or "give me the current date."

#### Purposes

- To provide or generate values on demand
- To implement lazy initialisation
- To create factory methods

#### Syntax Rules and Structure

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `T` | The type of the result. |
| `get()` | The functional method that returns a result. |

#### Annotated Code Example

```java
// File: SupplierDemo.java
import java.util.function.Supplier;

public class SupplierDemo {
    public static void main(String[] args) {
        // Supplier: return a constant string
        Supplier<String> greeting = () -> "Hello, World!";
        System.out.println(greeting.get());  // Output: Hello, World!

        // Supplier: return a random number
        Supplier<Double> random = () -> Math.random();
        System.out.println("Random: " + (random.get() >= 0));  // Output: Random: true

        // Supplier: lazy initialisation
        Supplier<String> lazy = () -> {
            System.out.println("Initialising...");
            return "Lazy Value";
        };
        System.out.println("Before get()");
        System.out.println(lazy.get());
        // Output: Before get()
        //         Initialising...
        //         Lazy Value
    }
}
```

**Expected Output:**
```
Hello, World!
Random: true
Before get()
Initialising...
Lazy Value
```

**Step-by-Step Explanation:**

1. `greeting` is a `Supplier<String>` that returns a constant string.
2. `random` is a `Supplier<Double>` that returns a random number.
3. `lazy` demonstrates lazy initialisation: the "Initialising..." message is only printed when `get()` is called.

#### Real-World Cases

- **Optional orElseGet:** `optional.orElseGet(() -> defaultValue)` uses `Supplier<T>`.
- **Logging:** Lazy message generation for logging.
- **Configuration:** Providing default configuration values.

---

### 2.10 `Callable<V>`

#### Definitions

**Core Definition:** `Callable<V>` represents a task that returns a result and may throw an exception.

**Technical Definition:** "A task that returns a result and may throw an exception. Implementors define a single method with no arguments called `call`".

**Beginner-Friendly Explanation:** `Callable` is like `Runnable` but it can return a result and throw exceptions. It's used for tasks that produce a value.

#### Purposes

- To define tasks that return results
- To execute tasks asynchronously with `ExecutorService`
- To propagate checked exceptions from tasks

#### Syntax Rules and Structure

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `V` | The type of the result. |
| `call()` | The functional method that computes a result. |

#### Annotated Code Example

```java
// File: CallableDemo.java
import java.util.concurrent.Callable;
import java.util.concurrent.FutureTask;

public class CallableDemo {
    public static void main(String[] args) throws Exception {
        // Callable: return a computed value
        Callable<Integer> task = () -> {
            System.out.println("Computing...");
            return 42;
        };

        FutureTask<Integer> future = new FutureTask<>(task);
        new Thread(future).start();

        System.out.println("Result: " + future.get());  // Output: Result: 42
    }
}
```

**Expected Output:**
```
Computing...
Result: 42
```

**Step-by-Step Explanation:**

1. `task` is a `Callable<Integer>` that computes and returns `42`.
2. `FutureTask` wraps the callable, and a new thread executes it.
3. `future.get()` retrieves the result.

#### Real-World Cases

- **Thread Pools:** `ExecutorService.submit(Callable)` returns a `Future`.
- **Parallel Computation:** Computing results in parallel and collecting them.
- **I/O Tasks:** Reading files or network resources that return data.

---

### 2.11 `Runnable`

#### Definitions

**Core Definition:** `Runnable` represents a task that can be executed by a thread, taking no arguments and returning no result.

**Technical Definition:** "The `Runnable` interface should be implemented by any class whose instances are intended to be executed by a thread. The class must define a method of no arguments called `run`".

**Beginner-Friendly Explanation:** `Runnable` is a task — you give it to a thread, and the thread runs it. It doesn't return anything and doesn't take any arguments.

#### Purposes

- To define tasks for thread execution
- To pass behaviour to `ExecutorService`
- To implement simple concurrent operations

#### Syntax Rules and Structure

```java
@FunctionalInterface
public interface Runnable {
    void run();
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `run()` | The functional method. |

#### Annotated Code Example

```java
// File: RunnableDemo.java
public class RunnableDemo {
    public static void main(String[] args) {
        // Runnable: print a message
        Runnable task = () -> System.out.println("Running in a thread");
        new Thread(task).start();
        // Output: Running in a thread

        // Runnable with a block body
        Runnable multiLine = () -> {
            System.out.println("Step 1");
            System.out.println("Step 2");
        };
        new Thread(multiLine).start();
        // Output: Step 1
        //         Step 2
    }
}
```

**Expected Output:**
```
Running in a thread
Step 1
Step 2
```

**Step-by-Step Explanation:**

1. `task` is a `Runnable` that prints a message.
2. `new Thread(task).start()` creates a thread and executes the runnable.
3. `multiLine` demonstrates a runnable with multiple statements.

#### Real-World Cases

- **Thread Creation:** `new Thread(() -> ...)`.
- **ExecutorService:** `executor.submit(() -> ...)`.
- **Event Dispatch:** Swing's `SwingUtilities.invokeLater(Runnable)`.

---

### 2.12 `Comparator<T>`

#### Definitions

**Core Definition:** `Comparator<T>` represents a comparison function that imposes a total ordering on a collection of objects.

**Technical Definition:** "A comparison function, which imposes a total ordering on some collection of objects. Comparators can be passed to a sort method (such as `Collections.sort` or `Arrays.sort`) to allow precise control over the sort order".

**Beginner-Friendly Explanation:** `Comparator` is a rule for comparing two objects. You tell it how to compare them, and it tells you which one comes first.

#### Purposes

- To define custom sorting orders
- To compare objects for sorting
- To chain multiple comparison criteria

#### Syntax Rules and Structure

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `T` | The type of objects to compare. |
| `compare(T o1, T o2)` | The functional method; returns a negative integer, zero, or a positive integer. |

#### Annotated Code Example

```java
// File: ComparatorDemo.java
import java.util.*;

public class ComparatorDemo {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>(List.of("Charlie", "Alice", "Bob"));

        // Comparator: natural order
        names.sort((a, b) -> a.compareTo(b));
        System.out.println("Natural: " + names);  // Output: [Alice, Bob, Charlie]

        // Comparator: reverse order
        names.sort((a, b) -> b.compareTo(a));
        System.out.println("Reverse: " + names);  // Output: [Charlie, Bob, Alice]

        // Comparator: by length
        names.sort(Comparator.comparingInt(String::length));
        System.out.println("By length: " + names);  // Output: [Bob, Alice, Charlie]
    }
}
```

**Expected Output:**
```
Natural: [Alice, Bob, Charlie]
Reverse: [Charlie, Bob, Alice]
By length: [Bob, Alice, Charlie]
```

**Step-by-Step Explanation:**

1. `names.sort((a, b) -> a.compareTo(b))` sorts in natural (alphabetical) order.
2. `names.sort((a, b) -> b.compareTo(a))` sorts in reverse order.
3. `Comparator.comparingInt(String::length)` sorts by string length.

#### Real-World Cases

- **Collection Sorting:** `Collections.sort(list, comparator)`.
- **Stream Sorted:** `stream.sorted(Comparator.comparing(Entity::getName))`.
- **TreeMap/TreeSet:** Custom ordering for sorted collections.

---

## 3. Primitive Specializations

Java provides primitive specializations of functional interfaces to avoid the overhead of boxing and unboxing primitive types.

### 3.1 `IntFunction`, `LongFunction`, `DoubleFunction`

| Interface | Functional Method | Description |
|-----------|-------------------|-------------|
| `IntFunction<R>` | `R apply(int value)` | Accepts an `int` and produces a result |
| `LongFunction<R>` | `R apply(long value)` | Accepts a `long` and produces a result |
| `DoubleFunction<R>` | `R apply(double value)` | Accepts a `double` and produces a result |

### 3.2 `ToIntFunction`, `ToLongFunction`, `ToDoubleFunction`

| Interface | Functional Method | Description |
|-----------|-------------------|-------------|
| `ToIntFunction<T>` | `int applyAsInt(T value)` | Accepts a `T` and produces an `int` |
| `ToLongFunction<T>` | `long applyAsLong(T value)` | Accepts a `T` and produces a `long` |
| `ToDoubleFunction<T>` | `double applyAsDouble(T value)` | Accepts a `T` and produces a `double` |

### 3.3 `IntPredicate`, `LongPredicate`, `DoublePredicate`

| Interface | Functional Method | Description |
|-----------|-------------------|-------------|
| `IntPredicate` | `boolean test(int value)` | Tests an `int` value |
| `LongPredicate` | `boolean test(long value)` | Tests a `long` value |
| `DoublePredicate` | `boolean test(double value)` | Tests a `double` value |

### 3.4 `IntConsumer`, `LongConsumer`, `DoubleConsumer`

| Interface | Functional Method | Description |
|-----------|-------------------|-------------|
| `IntConsumer` | `void accept(int value)` | Consumes an `int` value |
| `LongConsumer` | `void accept(long value)` | Consumes a `long` value |
| `DoubleConsumer` | `void accept(double value)` | Consumes a `double` value |

### 3.5 `IntSupplier`, `LongSupplier`, `DoubleSupplier`

| Interface | Functional Method | Description |
|-----------|-------------------|-------------|
| `IntSupplier` | `int getAsInt()` | Supplies an `int` value |
| `LongSupplier` | `long getAsLong()` | Supplies a `long` value |
| `DoubleSupplier` | `double getAsDouble()` | Supplies a `double` value |

### 3.6 `IntUnaryOperator`, `LongUnaryOperator`, `DoubleUnaryOperator`

| Interface | Functional Method | Description |
|-----------|-------------------|-------------|
| `IntUnaryOperator` | `int applyAsInt(int operand)` | Operates on a single `int` |
| `LongUnaryOperator` | `long applyAsLong(long operand)` | Operates on a single `long` |
| `DoubleUnaryOperator` | `double applyAsDouble(double operand)` | Operates on a single `double` |

### 3.7 `IntBinaryOperator`, `LongBinaryOperator`, `DoubleBinaryOperator`

| Interface | Functional Method | Description |
|-----------|-------------------|-------------|
| `IntBinaryOperator` | `int applyAsInt(int left, int right)` | Operates on two `int` values |
| `LongBinaryOperator` | `long applyAsLong(long left, long right)` | Operates on two `long` values |
| `DoubleBinaryOperator` | `double applyAsDouble(double left, double right)` | Operates on two `double` values |

### Annotated Code Example

```java
// File: PrimitiveSpecializationDemo.java
import java.util.function.*;

public class PrimitiveSpecializationDemo {
    public static void main(String[] args) {
        // IntFunction: int -> String
        IntFunction<String> intToString = i -> "Number: " + i;
        System.out.println(intToString.apply(42));  // Output: Number: 42

        // ToIntFunction: String -> int
        ToIntFunction<String> length = String::length;
        System.out.println("Length: " + length.applyAsInt("Hello"));  // Output: 5

        // IntPredicate: int -> boolean
        IntPredicate isEven = n -> n % 2 == 0;
        System.out.println("Is even: " + isEven.test(10));  // Output: true

        // IntConsumer: consume int
        IntConsumer printer = n -> System.out.println("Value: " + n);
        printer.accept(100);  // Output: Value: 100

        // IntSupplier: supply int
        IntSupplier random = () -> (int) (Math.random() * 100);
        System.out.println("Random: " + (random.getAsInt() >= 0));  // Output: Random: true

        // IntUnaryOperator: int -> int
        IntUnaryOperator doubleIt = n -> n * 2;
        System.out.println("Double: " + doubleIt.applyAsInt(5));  // Output: 10

        // IntBinaryOperator: (int, int) -> int
        IntBinaryOperator add = (a, b) -> a + b;
        System.out.println("Sum: " + add.applyAsInt(3, 4));  // Output: 7
    }
}
```

**Expected Output:**
```
Number: 42
Length: 5
Is even: true
Value: 100
Random: true
Double: 10
Sum: 7
```

**Step-by-Step Explanation:**

1. `intToString` is an `IntFunction<String>` that takes an `int` and returns a `String`.
2. `length` is a `ToIntFunction<String>` that takes a `String` and returns an `int`.
3. `isEven` is an `IntPredicate` that tests whether an `int` is even.
4. `printer` is an `IntConsumer` that consumes an `int`.
5. `random` is an `IntSupplier` that supplies a random `int`.
6. `doubleIt` is an `IntUnaryOperator` that doubles an `int`.
7. `add` is an `IntBinaryOperator` that adds two `int` values.

### Real-World Cases

- **Stream IntStream:** `IntStream.map(IntUnaryOperator)`, `IntStream.filter(IntPredicate)`.
- **Array Operations:** `Arrays.setAll(int[], IntUnaryOperator)`.
- **Performance-Critical Code:** Avoiding boxing/unboxing in numerical computations.

---

## 4. Custom Functional Interfaces

### Definitions

**Core Definition:** A custom functional interface is a user-defined interface with exactly one abstract method, designed to model a specific function contract not covered by the built-in interfaces.

**Technical Definition:** "If you want to create a functional interface, you can define your own with exactly one abstract method. The `@FunctionalInterface` annotation is optional but recommended".

**Beginner-Friendly Explanation:** Sometimes the built-in interfaces don't fit your specific need. You can create your own functional interface with exactly one abstract method to model your domain-specific behaviour.

### Purposes

- To model domain-specific function contracts
- To provide descriptive names for function types
- To enforce specific parameter and return types

### Syntax Rules and Structure

```java
@FunctionalInterface
public interface CustomInterface {
    returnType methodName(parameters);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@FunctionalInterface` | Optional annotation for compile-time checking. |
| Single abstract method | The function contract. |
| Default/static methods | Optional; do not count toward the abstract method count. |

### Annotated Code Example

```java
// File: CustomFunctionalInterfaceDemo.java
public class CustomFunctionalInterfaceDemo {

    // Custom functional interface for a two-argument operation
    @FunctionalInterface
    interface StringProcessor {
        String process(String input, int times);
    }

    // Custom functional interface for a three-argument operation
    @FunctionalInterface
    interface TripleFunction<A, B, C, R> {
        R apply(A a, B b, C c);
    }

    public static void main(String[] args) {
        // Using the custom StringProcessor
        StringProcessor repeater = (s, n) -> s.repeat(n);
        System.out.println(repeater.process("Hi", 3));  // Output: HiHiHi

        // Using the custom TripleFunction
        TripleFunction<Integer, Integer, Integer, Integer> addThree =
            (a, b, c) -> a + b + c;
        System.out.println("Sum: " + addThree.apply(1, 2, 3));  // Output: 6
    }
}
```

**Expected Output:**
```
HiHiHi
Sum: 6
```

**Step-by-Step Explanation:**

1. `StringProcessor` is a custom functional interface with a `process` method that takes a `String` and an `int` and returns a `String`.
2. `TripleFunction` is a custom functional interface that takes three arguments and returns a result.
3. The lambdas implement the single abstract method of each interface.

### Real-World Cases

- **Domain-Specific APIs:** A `PaymentProcessor` functional interface with a `process(Payment)` method.
- **Framework Callbacks:** Custom callback interfaces for event handling.
- **Validation:** A `Validator<T>` interface with a `validate(T)` method.

---

## 5. Functional Interface Best Practices

### Definitions

**Core Definition:** Functional interface best practices are established guidelines for designing and using functional interfaces effectively.

**Technical Definition:** Best practices derive from authoritative sources including *Effective Java* by Joshua Bloch and the JDK's own design patterns. Key principles include preferring standard functional interfaces over custom ones, using primitive specialisations for performance, and keeping functional interfaces focused and well-documented.

**Beginner-Friendly Explanation:** Best practices help you choose the right interface for the job, avoid unnecessary custom interfaces, and write code that is both efficient and readable.

### Purposes

- To improve code readability and maintainability
- To reduce unnecessary custom interfaces
- To optimise performance by avoiding boxing/unboxing
- To ensure consistent API design

### Key Best Practices

| Practice | Description |
|----------|-------------|
| Prefer Standard Functional Interfaces | Use `java.util.function` interfaces over custom ones when they fit (Effective Java, Item 44). |
| Use Primitive Specialisations | Use `IntPredicate`, `LongFunction`, etc., to avoid boxing overhead. |
| Annotate with `@FunctionalInterface` | Document intent and enable compile-time checking. |
| Keep Functional Interfaces Focused | One abstract method = one responsibility. |
| Document the Functional Method | Use Javadoc to describe the contract of the single abstract method. |
| Avoid Overloading with Functional Interfaces | Overloading methods that accept functional interfaces can cause ambiguity. |
| Use `andThen`, `compose`, `and`, `or`, `negate` | Leverage composition methods for reusable pipelines. |
| Prefer Method References | Use `String::length` instead of `s -> s.length()`. |

### Annotated Code Example

```java
// File: BestPracticesFunctionalInterface.java
import java.util.function.*;

public class BestPracticesFunctionalInterface {

    // GOOD: Use standard functional interface
    public static void processString(Function<String, String> transformer) {
        System.out.println(transformer.apply("hello"));
    }

    // GOOD: Use primitive specialisation
    public static int sumInts(IntBinaryOperator op, int a, int b) {
        return op.applyAsInt(a, b);
    }

    // BAD: Unnecessary custom interface (Function<String, String> suffices)
    // interface StringTransformer { String transform(String s); }

    public static void main(String[] args) {
        // GOOD: Method reference instead of lambda
        processString(String::toUpperCase);  // Output: HELLO

        // GOOD: Primitive specialisation avoids boxing
        int result = sumInts((a, b) -> a + b, 10, 20);
        System.out.println("Sum: " + result);  // Output: 30

        // GOOD: Use composition
        Function<String, String> trim = String::trim;
        Function<String, String> upper = String::toUpperCase;
        Function<String, String> trimThenUpper = trim.andThen(upper);
        System.out.println(trimThenUpper.apply("  hello  "));  // Output: HELLO
    }
}
```

**Expected Output:**
```
HELLO
Sum: 30
HELLO
```

**Step-by-Step Explanation:**

1. `processString` accepts a standard `Function<String, String>` instead of a custom interface.
2. `sumInts` uses `IntBinaryOperator` to avoid boxing overhead.
3. `String::toUpperCase` is a method reference, more concise than a lambda.
4. `trim.andThen(upper)` composes two functions into a pipeline.

### Real-World Cases

- **Stream API:** Uses standard functional interfaces throughout (`Function`, `Predicate`, `Consumer`, `Supplier`).
- **Optional:** Uses `Function`, `Predicate`, `Consumer`, `Supplier`.
- **CompletableFuture:** Uses `Function`, `Consumer`, `Runnable`, `Supplier`.
- **Spring Framework:** Uses `Function`, `Predicate`, `Consumer` for callbacks and handlers.

---

## References Links

- Annotation Type FunctionalInterface — Java™ Platform, Standard Edition 11 API Specification - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/FunctionalInterface.html
- Annotation Type FunctionalInterface — Java™ Platform, Standard Edition 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html
- Package java.util.function — Java™ Platform, Standard Edition 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html
- Package java.util.function — Java™ Platform, Standard Edition 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html
- Java Language Specification, Chapter 9: Interfaces - https://docs.oracle.com/javase/specs/jls/se21/html/jls-9.html
- JSR 335: Lambda Expressions for the Java™ Programming Language - https://cr.openjdk.org/~dlsmith/jsr335/jsr335-0.6.2/index.html
- Effective Java (3rd Edition) by Joshua Bloch — Item 44: Favor the use of standard functional interfaces
- Oracle Java Tutorials — Lambda Expressions - https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html