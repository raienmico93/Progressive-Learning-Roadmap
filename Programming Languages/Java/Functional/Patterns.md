# Java Functional Programming Patterns: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Functional programming (FP) is a programming paradigm that treats computation as the evaluation of mathematical functions, emphasising immutability, pure functions, and the avoidance of side effects. In Java, FP patterns are supported through lambda expressions, functional interfaces, the Stream API, and `Optional`.

**Technical Definition:** Functional programming is a programming paradigm centred on the structure and use of functions, emphasising immutability, pure functions, and declarative logic. Computations are treated as the evaluation of mathematical functions where the output of one function serves as the input to the next. Java provides FP tools such as lambda expressions and Streams, allowing developers to augment object-oriented code with functional principles.

**Beginner-Friendly Explanation:** Functional programming is a way of writing code that treats functions like mathematical operations — you give them inputs, they produce outputs, and they don't secretly change anything else in your program. Instead of telling the computer step-by-step how to do something (like a recipe), you describe what you want (like ordering from a menu). This makes your code easier to test, understand, and run safely on multiple threads.

### Key Characteristics

- **Pure Functions:** Functions that always produce the same output for the same input and have no side effects.
- **Immutability:** Data structures that cannot be modified after creation, avoiding unintended state changes.
- **Declarative Style:** Describing what to achieve rather than how to achieve it.
- **Function Composition:** Building complex behaviour by combining simpler functions.
- **Lazy Evaluation:** Delaying computation until the result is actually needed.
- **Higher-Order Functions:** Functions that accept or return other functions.

### Prerequisites

- Basic understanding of Java syntax (classes, methods, variables)
- Familiarity with lambda expressions and functional interfaces
- Knowledge of the `java.util.function` package
- Understanding of generics and type inference

### Related Programming Areas

- **Lambda Expressions:** The foundation for functional programming in Java.
- **Functional Interfaces:** Provide target types for lambdas.
- **Stream API:** Declarative data processing pipelines.
- **Optional:** Null-safety and functional value handling.
- **Concurrency:** Immutability enables safe parallel execution.
- **Design Patterns:** Many traditional patterns can be implemented functionally.

### Core Concepts / Features

---

## 1. Pure Functions

### Definitions

**Core Definition:** A pure function is a function that always returns the same output for the same input and has no side effects — it does not modify any external state or depend on any hidden information.

**Technical Definition:** "A pure function is a function that has the following properties: the function always evaluates the same result value given the same argument value(s). The function result value cannot depend on any hidden information or state that may change as program execution proceeds or between different executions of the program. Evaluation of the result does not cause any semantically observable side effect or output". Pure functions have referential transparency — they depend only on their parameters, have no side effects, and do not maintain any state.

**Beginner-Friendly Explanation:** A pure function is like a vending machine: you put in the same coins, you get the same snack every time. It doesn't remember previous purchases, doesn't randomly give you something different, and doesn't break anything else in the process. A function that prints to the console, modifies a global variable, or reads a file is not pure because it interacts with the outside world.

### Purposes

- To make code predictable and easier to reason about
- To enable safe parallel execution without race conditions
- To simplify testing (no mocks needed)
- To enable referential transparency (an expression can be replaced by its value)
- To reduce bugs caused by hidden state and side effects

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Pure function example
public static int add(int a, int b) {
    return a + b;  // No side effects, depends only on parameters
}

// Impure function example (for comparison)
public static int counter = 0;
public static int increment() {
    counter++;     // Side effect: modifies external state
    return counter;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Parameters | All inputs are declared as parameters; no hidden state. |
| Return value | Depends only on parameters; no randomness or external reads. |
| No side effects | Does not modify external state, print, or perform I/O. |

**Syntax Rules:**

- A pure function must not modify any external state (no assignments to fields, no `System.out.println`, no file writes).
- A pure function must not depend on mutable external state (no reading of mutable fields, no random number generation).
- A pure function may call other pure functions.
- In Java, static methods with only `final` or effectively final parameters are typically pure.

**Constraints and Limitations:**

- Java is not a purely functional language; side effects are inevitable for I/O and state management.
- Immutable data structures are required to maintain purity when working with objects.
- Pure functions cannot perform logging, which is a common source of impurity.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Pure vs Impure Functions**

```java
// File: PureFunctionDemo.java
public class PureFunctionDemo {

    // PURE: depends only on parameters, no side effects
    public static int add(int a, int b) {
        return a + b;
    }

    // PURE: mathematical function
    public static double circleArea(double radius) {
        return Math.PI * radius * radius;
    }

    // IMPURE: depends on external mutable state
    private static int counter = 0;
    public static int incrementAndGet() {
        counter++;          // Side effect: modifies external state
        return counter;
    }

    // IMPURE: prints to console (side effect)
    public static int addAndPrint(int a, int b) {
        System.out.println("Adding " + a + " and " + b);  // Side effect
        return a + b;
    }

    public static void main(String[] args) {
        // Pure function: same input, same output, no side effects
        System.out.println(add(3, 4));          // Output: 7
        System.out.println(add(3, 4));          // Output: 7 (always)
  
        System.out.println(circleArea(5.0));    // Output: 78.53981633974483

        // Impure function: output changes with each call
        System.out.println(incrementAndGet());  // Output: 1
        System.out.println(incrementAndGet());  // Output: 2 (different!)
    }
}
```

**Expected Output:**
```
7
7
78.53981633974483
1
2
```

**Step-by-Step Explanation:**

1. `add(3, 4)` is pure: it always returns `7` for the same inputs, regardless of when or how many times it is called.
2. `circleArea(5.0)` is pure: it depends only on the `radius` parameter and returns the same value every time.
3. `incrementAndGet()` is impure: it modifies the static `counter` field, so the output changes with each call.
4. `addAndPrint` is impure because it produces a side effect (console output).

### Real-World Cases

- **Mathematical Libraries:** `Math.sqrt()`, `Math.max()` are pure functions.
- **Validation:** `isValidEmail(String email)` is pure if it only checks the string format.
- **Data Transformation:** `String.toUpperCase()` is pure (returns a new string).
- **Business Rules:** Tax calculation functions that depend only on input amounts.

---

## 2. Immutability

### Definitions

**Core Definition:** Immutability is the property of an object whose state cannot be modified after it is created.

**Technical Definition:** An immutable object is one whose state cannot change after construction. All fields are `final`, the class is `final` (or all constructors are private), and any mutable objects referenced by the class are defensively copied. Java provides several built-in immutable types, including `String`, `Integer`, `Long`, `Double`, `BigDecimal`, `BigInteger`, and the `java.time` classes. Records (Java 16+) provide a concise way to create immutable data carriers with automatic `equals`, `hashCode`, and `toString`.

**Beginner-Friendly Explanation:** An immutable object is like a stone tablet with writing on it — once it's carved, it cannot be changed. You can make a copy with different writing, but the original stays the same. This is in contrast to a whiteboard, which can be erased and rewritten (mutable). Immutability makes programs safer because no one can accidentally change data that another part of the program is relying on.

### Purposes

- To eliminate unintended state changes and race conditions
- To enable safe sharing of objects across threads without synchronization
- To simplify reasoning about program state
- To enable reliable caching and memoization
- To support functional programming patterns

### Syntax Rules and Structure

**Complete General Syntax (Immutable Class):**

```java
public final class ImmutablePoint {
    private final int x;
    private final int y;

    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }

    // "Modification" returns a new instance
    public ImmutablePoint withX(int newX) {
        return new ImmutablePoint(newX, this.y);
    }
}
```

**Complete General Syntax (Record — Java 16+):**

```java
public record Point(int x, int y) {}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `final` class | Prevents subclassing that could introduce mutation. |
| `final` fields | All fields must be final and set in the constructor. |
| No setters | No methods that modify fields. |
| Defensive copies | Mutable objects (e.g., arrays, lists) must be copied on input and output. |
| Records | Concise syntax for immutable data carriers (Java 16+). |

**Syntax Rules:**

- Declare the class as `final`.
- Declare all fields as `private final`.
- Do not provide setter methods.
- If a field is a mutable object, return a defensive copy in getters.
- Use records for simple immutable data carriers.

**Constraints and Limitations:**

- Creating modified copies can be expensive for large objects.
- Immutability requires discipline; Java does not enforce it automatically.
- Some frameworks (e.g., JPA/Hibernate) expect mutable entities.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Immutable Class vs Mutable Class**

```java
// File: ImmutabilityDemo.java
import java.util.*;

// IMMUTABLE class
final class ImmutablePerson {
    private final String name;
    private final int age;
    private final List<String> hobbies;

    public ImmutablePerson(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        // Defensive copy: prevents external modification of the list
        this.hobbies = new ArrayList<>(hobbies);
    }

    public String getName() { return name; }
    public int getAge() { return age; }

    // Returns a defensive copy, not the internal list
    public List<String> getHobbies() {
        return new ArrayList<>(hobbies);
    }

    // "Modification" returns a new instance
    public ImmutablePerson withAge(int newAge) {
        return new ImmutablePerson(this.name, newAge, this.hobbies);
    }

    @Override
    public String toString() {
        return "Person[" + name + ", " + age + ", " + hobbies + "]";
    }
}

// MUTABLE class (for comparison)
class MutablePerson {
    private String name;
    private int age;
    public MutablePerson(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public void setName(String name) { this.name = name; }
    public void setAge(int age) { this.age = age; }
}

public class ImmutabilityDemo {
    public static void main(String[] args) {
        // Immutable: cannot be changed
        ImmutablePerson alice = new ImmutablePerson("Alice", 30, List.of("Reading", "Hiking"));
        System.out.println(alice);  // Person[Alice, 30, [Reading, Hiking]]

        // "Modification" creates a new object
        ImmutablePerson olderAlice = alice.withAge(31);
        System.out.println(olderAlice);  // Person[Alice, 31, [Reading, Hiking]]
        System.out.println(alice);       // Person[Alice, 30, [Reading, Hiking]] (unchanged)

        // Mutable: can be changed in place
        MutablePerson bob = new MutablePerson("Bob", 25);
        bob.setAge(26);  // Modifies the same object
        System.out.println("Bob's age: " + bob.age);
    }
}
```

**Expected Output:**
```
Person[Alice, 30, [Reading, Hiking]]
Person[Alice, 31, [Reading, Hiking]]
Person[Alice, 30, [Reading, Hiking]]
Bob's age: 26
```

**Step-by-Step Explanation:**

1. `ImmutablePerson` has `final` fields and no setters.
2. The constructor makes a defensive copy of the `hobbies` list.
3. `getHobbies()` returns a copy, so callers cannot modify the internal list.
4. `withAge(31)` returns a new `ImmutablePerson` with the updated age, leaving the original unchanged.
5. `MutablePerson` allows modification via setters, demonstrating the contrast.

### Real-World Cases

- **String Manipulation:** `String` is immutable; `substring()`, `toUpperCase()` return new strings.
- **Configuration:** Immutable configuration objects shared across threads.
- **Value Objects:** Money, dates, coordinates — objects that represent values.
- **DTOs:** Records used for data transfer in REST APIs.

---

## 3. Higher-Order Functions

### Definitions

**Core Definition:** A higher-order function is a function that takes one or more functions as parameters, or returns a function as its result.

**Technical Definition:** "Functions that take another function as a parameter or return a function as a result" are called higher-order functions. In Java, higher-order functions are expressed using functional interfaces and lambda expressions. Java does not have first-class functions in the traditional sense, but lambdas and method references allow functions to be passed as objects.

**Beginner-Friendly Explanation:** A higher-order function is like a machine that accepts other machines as input. For example, a "sorting machine" might accept a "comparison machine" to decide how to sort things. Or a "pipeline" might accept a "transformation machine" to apply to each item.

### Purposes

- To abstract over behaviour, not just data
- To enable code reuse by parameterising logic
- To implement functional design patterns (Strategy, Template Method, etc.)
- To compose complex behaviour from simpler functions

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Higher-order function that takes a function
public static <T, R> List<R> transform(List<T> list, Function<T, R> mapper) {
    List<R> result = new ArrayList<>();
    for (T item : list) {
        result.add(mapper.apply(item));
    }
    return result;
}

// Higher-order function that returns a function
public static Function<Integer, Integer> multiplier(int factor) {
    return x -> x * factor;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Functional interface parameter | Accepts a lambda or method reference. |
| Return type | May be a functional interface (returning a function). |
| `apply()` call | Invokes the passed function. |

**Syntax Rules:**

- Higher-order functions accept functional interfaces as parameters.
- They may return functional interfaces as results.
- Lambdas and method references provide the function arguments.
- Generic type parameters allow flexible input and output types.

**Constraints and Limitations:**

- Java's type system can become complex with deeply nested generics.
- Higher-order functions may introduce performance overhead due to object allocation.
- Debugging can be harder because the function body is not visible in the caller.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Higher-Order Function Taking a Function**

```java
// File: HigherOrderDemo.java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class HigherOrderDemo {

    // Higher-order function: takes a Predicate and a Consumer
    public static <T> void processIf(List<T> list, Predicate<T> condition, Consumer<T> action) {
        for (T item : list) {
            if (condition.test(item)) {
                action.accept(item);
            }
        }
    }

    // Higher-order function: returns a function
    public static Function<Integer, Integer> multiplier(int factor) {
        return x -> x * factor;  // Returns a function
    }

    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // Pass behaviour to a higher-order function
        processIf(numbers, n -> n % 2 == 0, n -> System.out.print(n + " "));
        // Output: 2 4 6 8 10

        System.out.println();

        // Get a function back from a higher-order function
        Function<Integer, Integer> triple = multiplier(3);
        Function<Integer, Integer> quadruple = multiplier(4);

        System.out.println("Triple 5: " + triple.apply(5));    // Output: Triple 5: 15
        System.out.println("Quadruple 5: " + quadruple.apply(5)); // Output: Quadruple 5: 20
    }
}
```

**Expected Output:**
```
2 4 6 8 10
Triple 5: 15
Quadruple 5: 20
```

**Step-by-Step Explanation:**

1. `processIf` is a higher-order function that takes a `Predicate` and a `Consumer`.
2. The lambda `n -> n % 2 == 0` is passed as the condition, and `n -> System.out.print(n + " ")` as the action.
3. `multiplier(3)` returns a function that multiplies its input by `3`.
4. `triple.apply(5)` applies the returned function, computing `5 * 3 = 15`.

### Real-World Cases

- **Stream API:** `filter(Predicate)`, `map(Function)`, `forEach(Consumer)` are all higher-order functions.
- **Comparator Chaining:** `Comparator.comparing(...).thenComparing(...)`.
- **Event Handling:** `addActionListener(Consumer<ActionEvent>)`.
- **Template Method Pattern:** A method that accepts a callback function.

---

## 4. Function Composition

### Definitions

**Core Definition:** Function composition is the process of combining two or more functions to create a new function, where the output of one function becomes the input of the next.

**Technical Definition:** "Function composition: the output of one function serves as the input to the next function". In Java, `Function` provides `andThen()` and `compose()` methods. `andThen()` applies the current function first, then the `after` function. `compose()` applies the `before` function first, then the current function.

**Beginner-Friendly Explanation:** Function composition is like an assembly line. One worker shapes a piece, passes it to the next worker who paints it, who passes it to a third who packages it. Each step builds on the previous one.

### Purposes

- To build complex transformations from simple, reusable functions
- To improve code readability by naming each step of a pipeline
- To enable point-free style (writing functions without explicitly mentioning arguments)
- To support declarative data processing

### Syntax Rules and Structure

**Complete General Syntax:**

```java
Function<T, R> f = ...;
Function<R, V> g = ...;

Function<T, V> composedAndThen = f.andThen(g);  // f first, then g
Function<T, V> composedCompose = g.compose(f);  // f first, then g (same as andThen)
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `andThen(after)` | Applies this function first, then the `after` function. |
| `compose(before)` | Applies the `before` function first, then this function. |

**Syntax Rules:**

- `andThen()` and `compose()` are default methods on `Function`.
- The output type of the first function must match the input type of the second.
- Both methods return a new `Function` instance.

**Constraints and Limitations:**

- Composition is only available on `Function` (and its specialisations).
- Deeply composed functions can be harder to debug.
- Type inference can become challenging with complex compositions.

### Annotated Complete Step-by-Step Code Examples

```java
// File: FunctionCompositionDemo.java
import java.util.function.Function;

public class FunctionCompositionDemo {
    public static void main(String[] args) {
        Function<String, String> trim = String::trim;
        Function<String, String> upper = String::toUpperCase;
        Function<String, String> exclaim = s -> s + "!";

        // andThen: trim -> upper -> exclaim
        Function<String, String> pipeline = trim.andThen(upper).andThen(exclaim);
        System.out.println(pipeline.apply("  hello  "));  // Output: HELLO!

        // compose: same as andThen but in reverse order
        Function<String, String> composed = exclaim.compose(upper).compose(trim);
        System.out.println(composed.apply("  world  "));  // Output: WORLD!

        // Identity function
        Function<String, String> identity = Function.identity();
        System.out.println(identity.apply("unchanged"));  // Output: unchanged

        // Practical example: parse -> validate -> format
        Function<String, Integer> parse = Integer::parseInt;
        Function<Integer, Integer> doubleIt = x -> x * 2;
        Function<Integer, String> format = x -> "Result: " + x;

        Function<String, String> fullPipeline = parse.andThen(doubleIt).andThen(format);
        System.out.println(fullPipeline.apply("21"));  // Output: Result: 42
    }
}
```

**Expected Output:**
```
HELLO!
WORLD!
unchanged
Result: 42
```

**Step-by-Step Explanation:**

1. `trim.andThen(upper).andThen(exclaim)` creates a pipeline: trim, then uppercase, then add "!".
2. `exclaim.compose(upper).compose(trim)` achieves the same result using `compose()`.
3. `Function.identity()` returns a function that returns its input unchanged.
4. The practical example composes parsing, doubling, and formatting into a single function.

### Real-World Cases

- **Data Validation Pipelines:** `trim.andThen(validate).andThen(normalize)`.
- **Text Processing:** `String::trim` then `String::toLowerCase` then `String::replace`.
- **Stream API:** `stream.map(f).map(g)` is equivalent to `stream.map(f.andThen(g))`.
- **Predicate Composition:** `predicate1.and(predicate2).or(predicate3)`.

---

## 5. Currying

### Definitions

**Core Definition:** Currying is the technique of transforming a function that takes multiple arguments into a series of functions that each take a single argument.

**Technical Definition:** "Currying is the act of taking `#R(A1,A2,...,An)` and turn it into `#R(An)...(A2)(A1)`". Currying restructures a multi-parameter function into nested single-parameter functions, enabling partial application.

**Beginner-Friendly Explanation:** Currying is like ordering a custom sandwich. Instead of saying "give me a ham and cheese on rye," you say "give me a sandwich maker," then "with ham," then "with cheese," then "on rye." Each step returns a new function that remembers the previous choices.

### Purposes

- To enable partial application (pre-setting some arguments)
- To create specialised functions from general ones
- To improve code reuse and modularity
- To support function composition in a point-free style

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Uncurried: takes two arguments
Function<Integer, Function<Integer, Integer>> curriedAdd = a -> b -> a + b;

// Usage
Function<Integer, Integer> add5 = curriedAdd.apply(5);
int result = add5.apply(3);  // 8
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Outer function | Takes the first argument and returns a function. |
| Inner function | Takes the second argument and returns the result. |
| `apply().apply()` | Chained application to supply arguments one at a time. |

**Syntax Rules:**

- Curried functions return nested `Function` instances.
- Each `apply()` call supplies one argument and returns the next function.
- The final `apply()` call returns the result.

**Constraints and Limitations:**

- Java's syntax for curried functions is verbose due to nested generics.
- Currying is less common in Java than in languages like Haskell.
- Readability can suffer with deeply nested curried functions.

### Annotated Complete Step-by-Step Code Examples

```java
// File: CurryingDemo.java
import java.util.function.*;

public class CurryingDemo {
    public static void main(String[] args) {
        // Curried function: a -> b -> a + b
        Function<Integer, Function<Integer, Integer>> curriedAdd = a -> b -> a + b;

        // Partial application: fix a = 5
        Function<Integer, Integer> add5 = curriedAdd.apply(5);
        System.out.println("5 + 3 = " + add5.apply(3));   // Output: 5 + 3 = 8
        System.out.println("5 + 10 = " + add5.apply(10)); // Output: 5 + 10 = 15

        // Curried function for formatting
        Function<String, Function<String, Function<String, String>>> format =
            greeting -> name -> punctuation -> greeting + ", " + name + punctuation;

        Function<String, Function<String, String>> hello = format.apply("Hello");
        Function<String, String> helloAlice = hello.apply("Alice");
        System.out.println(helloAlice.apply("!"));  // Output: Hello, Alice!

        // Using method reference to create a curried function
        Function<Integer, Function<Integer, Integer>> curriedMultiply = a -> b -> a * b;
        Function<Integer, Integer> triple = curriedMultiply.apply(3);
        System.out.println("Triple 7: " + triple.apply(7));  // Output: Triple 7: 21
    }
}
```

**Expected Output:**
```
5 + 3 = 8
5 + 10 = 15
Hello, Alice!
Triple 7: 21
```

**Step-by-Step Explanation:**

1. `curriedAdd` is a curried function: `apply(5)` returns a function that adds `5` to its argument.
2. `add5` is a partially applied function with `a = 5` fixed.
3. The `format` function is curried three times, allowing arguments to be supplied one at a time.
4. `helloAlice.apply("!")` completes the function and produces the final result.

### Real-World Cases

- **Configuration Factories:** `createConnection(host).port(8080).build()`.
- **Logging:** `log(level).message(msg).withException(e)`.
- **Validation:** `validate(field).withRule(rule).against(value)`.
- **Builder Pattern:** Currying is conceptually similar to fluent builders.

---

## 6. Partial Application

### Definitions

**Core Definition:** Partial application is the process of fixing some arguments of a function, producing a new function that takes the remaining arguments.

**Technical Definition:** "Partial application is more general than currying. It means giving a function that holds some arguments fixed and leaves the rest for later calls". In Java, partial application can be achieved using lambda expressions or by binding arguments to a function.

**Beginner-Friendly Explanation:** Partial application is like filling in part of a form. If you have a function that needs a name, address, and phone number, you can fill in the address part now and leave the rest for later.

### Purposes

- To create specialised functions from general ones
- To reduce the number of parameters at call sites
- To improve code readability by giving meaningful names to partially applied functions
- To enable function reuse across different contexts

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Original function
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;

// Partial application: fix a = 10
Function<Integer, Integer> add10 = b -> add.apply(10, b);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Original function | Takes multiple arguments. |
| Fixed argument | An argument that is pre-set. |
| Resulting function | Takes the remaining arguments. |

**Syntax Rules:**

- Partial application is achieved by creating a new function that calls the original with some arguments fixed.
- The result is a function with fewer parameters.
- Partial application can be combined with currying.

**Constraints and Limitations:**

- Java does not have built-in partial application syntax; it must be simulated with lambdas.
- The resulting function's type must be expressed as a functional interface.
- Partial application can lead to many small functions, which may reduce readability if overused.

### Annotated Complete Step-by-Step Code Examples

```java
// File: PartialApplicationDemo.java
import java.util.function.*;

public class PartialApplicationDemo {
    public static void main(String[] args) {
        // Original function: three arguments
        TriFunction<String, String, String, String> formatName =
            (title, first, last) -> title + " " + first + " " + last;

        // Partial application: fix the title
        BiFunction<String, String, String> drName =
            (first, last) -> formatName.apply("Dr.", first, last);

        System.out.println(drName.apply("Alice", "Smith"));  // Output: Dr. Alice Smith
        System.out.println(drName.apply("Bob", "Jones"));    // Output: Dr. Bob Jones

        // Partial application: fix the title and first name
        Function<String, String> drAlice =
            last -> formatName.apply("Dr.", "Alice", last);

        System.out.println(drAlice.apply("Smith"));  // Output: Dr. Alice Smith
        System.out.println(drAlice.apply("Brown"));  // Output: Dr. Alice Brown

        // Practical example: discount application
        BiFunction<Double, Double, Double> applyDiscount =
            (price, discount) -> price * (1 - discount);

        Function<Double, Double> tenPercentOff = price -> applyDiscount.apply(price, 0.10);
        Function<Double, Double> twentyPercentOff = price -> applyDiscount.apply(price, 0.20);

        System.out.println("$100 with 10% off: $" + tenPercentOff.apply(100.0));
        // Output: $100 with 10% off: $90.0
        System.out.println("$100 with 20% off: $" + twentyPercentOff.apply(100.0));
        // Output: $100 with 20% off: $80.0
    }

    // Custom functional interface for three arguments
    @FunctionalInterface
    interface TriFunction<A, B, C, R> {
        R apply(A a, B b, C c);
    }
}
```

**Expected Output:**
```
Dr. Alice Smith
Dr. Bob Jones
Dr. Alice Smith
Dr. Alice Brown
$100 with 10% off: $90.0
$100 with 20% off: $80.0
```

**Step-by-Step Explanation:**

1. `formatName` is a `TriFunction` that takes a title, first name, and last name.
2. `drName` is a partial application that fixes the title to `"Dr."`.
3. `drAlice` fixes both the title and the first name, leaving only the last name.
4. `tenPercentOff` and `twentyPercentOff` are partially applied discount functions.

### Real-World Cases

- **Tax Calculation:** `calculateTax(rate, amount)` partially applied with a fixed rate.
- **Logging:** `log(level)` returns a function that logs at a specific level.
- **Validation:** `validateWithRule(rule)` returns a validator for different values.
- **Configuration:** `configureWithDefaults(defaults)` returns a configurer.

---

## 7. Memoization

### Definitions

**Core Definition:** Memoization is an optimisation technique that caches the results of expensive function calls and returns the cached result when the same inputs occur again.

**Technical Definition:** Memoization is a form of caching where the results of pure functions are stored in a map keyed by their inputs. Since pure functions always return the same output for the same input, the cached result is guaranteed to be correct. In Java, memoization can be implemented using a `Map` and `computeIfAbsent()`.

**Beginner-Friendly Explanation:** Memoization is like keeping a notebook of answers to difficult questions. When someone asks a question you've already answered, you just look it up in your notebook instead of solving it again. This saves time for expensive calculations.

### Purposes

- To avoid redundant computation for expensive functions
- To improve performance for recursive algorithms (e.g., Fibonacci)
- To cache results of pure functions safely
- To enable dynamic programming solutions

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public static <T, R> Function<T, R> memoize(Function<T, R> function) {
    Map<T, R> cache = new HashMap<>();
    return input -> cache.computeIfAbsent(input, function);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Cache | A `Map` storing computed results. |
| `computeIfAbsent()` | Checks the cache; if absent, computes and stores. |
| Wrapped function | The original expensive function. |

**Syntax Rules:**

- Memoization requires a pure function (same input always produces same output).
- The cache must be thread-safe for concurrent use (`ConcurrentHashMap`).
- The cache should be bounded to prevent memory leaks.

**Constraints and Limitations:**

- Memoization is only valid for pure functions.
- Memory usage grows with the number of distinct inputs.
- Mutable return values can break memoization if modified by callers.
- Thread safety requires additional synchronization.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Memoizing Fibonacci**

```java
// File: MemoizationDemo.java
import java.util.*;
import java.util.function.*;

public class MemoizationDemo {

    // Memoizer: wraps a function with a cache
    public static <T, R> Function<T, R> memoize(Function<T, R> function) {
        Map<T, R> cache = new HashMap<>();
        return input -> cache.computeIfAbsent(input, function);
    }

    public static void main(String[] args) {
        // Memoized Fibonacci
        Function<Integer, Long> fib = memoize(new Function<Integer, Long>() {
            Map<Integer, Long> fibCache = new HashMap<>();

            @Override
            public Long apply(Integer n) {
                if (n <= 1) return (long) n;
                return this.apply(n - 1) + this.apply(n - 2);
            }
        });

        // Without memoization: exponential time
        // With memoization: linear time
        System.out.println("Fib(10): " + fib.apply(10));   // Output: Fib(10): 55
        System.out.println("Fib(20): " + fib.apply(20));   // Output: Fib(20): 6765
        System.out.println("Fib(30): " + fib.apply(30));   // Output: Fib(30): 832040

        // Simple memoization example
        Function<Integer, Integer> square = memoize(x -> {
            System.out.println("Computing square of " + x);
            return x * x;
        });

        System.out.println(square.apply(5));  // Output: Computing square of 5, then 25
        System.out.println(square.apply(5));  // Output: 25 (no computation message)
    }
}
```

**Expected Output:**
```
Fib(10): 55
Fib(20): 6765
Fib(30): 832040
Computing square of 5
25
25
```

**Step-by-Step Explanation:**

1. `memoize` wraps a function with a `HashMap` cache.
2. `computeIfAbsent` checks the cache before computing.
3. For Fibonacci, the memoized version computes each value once, dramatically reducing time.
4. For `square`, the first call to `apply(5)` computes and caches `25`; the second call returns the cached value without printing the computation message.

### Real-World Cases

- **Dynamic Programming:** Fibonacci, factorial, edit distance.
- **Database Queries:** Caching results of expensive queries.
- **Web Services:** Caching API responses.
- **Configuration Parsing:** Caching parsed configuration values.

---

## 8. Lazy Evaluation

### Definitions

**Core Definition:** Lazy evaluation is an evaluation strategy that delays the computation of an expression until its value is actually needed.

**Technical Definition:** "Lazy evaluation allows the definition of infinite streams... Such streams are infinite in the sense that arbitrary many elements can be accessed as these elements are computed 'on demand'". "The Java language does not support lazy evaluation directly, but it can be simulated with a functor object definition". Java streams use lazy evaluation for intermediate operations.

**Beginner-Friendly Explanation:** Lazy evaluation is like a lazy student who only does homework when the teacher asks for it. If the homework is never needed, it never gets done. This saves effort when the result isn't required.

### Purposes

- To avoid unnecessary computation
- To work with infinite data structures
- To improve performance by deferring expensive operations
- To enable short-circuit evaluation

### Syntax Rules and Structure

**Complete General Syntax (Simulating Lazy Evaluation):**

```java
// Using Supplier for lazy evaluation
Supplier<String> lazyValue = () -> expensiveComputation();

// Computation is deferred until get() is called
String value = lazyValue.get();
```

**Complete General Syntax (Stream Lazy Evaluation):**

```java
// Stream operations are lazy until a terminal operation is invoked
Stream<String> stream = list.stream()
    .filter(s -> {
        System.out.println("Filtering: " + s);
        return s.length() > 3;
    });  // Nothing printed yet

// Terminal operation triggers execution
List<String> result = stream.collect(Collectors.toList());  // Now filtering happens
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Supplier` | Encapsulates a computation that is deferred. |
| Stream intermediate operations | Lazy; do not execute until terminal operation. |
| `limit()` | Short-circuits infinite streams. |

**Syntax Rules:**

- Use `Supplier` or `Function` to encapsulate deferred computations.
- Stream intermediate operations are lazy by design.
- Terminal operations trigger the execution of the entire pipeline.

**Constraints and Limitations:**

- Java does not support lazy evaluation natively; it must be simulated.
- Lazy evaluation can cause unexpected timing of side effects.
- Infinite streams must be bounded with `limit()` to avoid infinite loops.

### Annotated Complete Step-by-Step Code Examples

```java
// File: LazyEvaluationDemo.java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class LazyEvaluationDemo {
    public static void main(String[] args) {
        // 1. Lazy with Supplier
        Supplier<Double> lazyRandom = () -> {
            System.out.println("Generating random number...");
            return Math.random();
        };

        System.out.println("Before get()");  // Printed first
        // Nothing happens until get() is called
        double value = lazyRandom.get();     // "Generating..." printed here
        System.out.println("After get()");

        // 2. Stream laziness
        List<String> names = List.of("Alice", "Bob", "Charlie", "David");

        System.out.println("Before stream pipeline");
        Stream<String> filtered = names.stream()
            .filter(name -> {
                System.out.println("Filtering: " + name);
                return name.length() > 3;
            });

        System.out.println("After stream creation (nothing filtered yet)");

        // Terminal operation triggers filtering
        List<String> result = filtered.collect(Collectors.toList());
        System.out.println("Result: " + result);

        // 3. Infinite stream with limit (lazy)
        System.out.println("Infinite stream example:");
        Stream.iterate(1, n -> n + 1)
            .filter(n -> n % 2 == 0)
            .limit(5)
            .forEach(n -> System.out.print(n + " "));
        System.out.println();
        // Output: 2 4 6 8 10
    }
}
```

**Expected Output:**
```
Before get()
Generating random number...
After get()
Before stream pipeline
After stream creation (nothing filtered yet)
Filtering: Alice
Filtering: Bob
Filtering: Charlie
Filtering: David
Result: [Alice, Charlie, David]
Infinite stream example:
2 4 6 8 10
```

**Step-by-Step Explanation:**

1. `lazyRandom` is a `Supplier` that is not executed until `get()` is called.
2. The stream pipeline is created but nothing is filtered until `collect()` is invoked.
3. `Stream.iterate(1, n -> n + 1)` creates an infinite stream; `limit(5)` bounds it.
4. Because of laziness, only the necessary elements are generated and processed.

### Real-World Cases

- **Stream Processing:** Java streams are lazy, enabling efficient pipelines.
- **Configuration Loading:** Loading configuration values only when accessed.
- **Database Queries:** Deferring query execution until results are needed.
- **Infinite Sequences:** Generating prime numbers or random values on demand.

---

## 9. Functional Programming Best Practices

### Definitions

**Core Definition:** Functional programming best practices are guidelines for applying FP principles effectively in Java without sacrificing readability or maintainability.

**Technical Definition:** Best practices derive from authoritative sources including *Functional Programming in Java* by Venkat Subramaniam and *A Functional Approach to Java* by Ben Weidig. Key principles include preferring pure functions, honouring immutability, using streams judiciously, and avoiding overly complex lambda expressions.

**Beginner-Friendly Explanation:** Best practices help you use functional programming effectively without making your code harder to understand. They're like cooking tips — they help you avoid common mistakes and produce better results.

### Purposes

- To improve code readability and maintainability
- To avoid common functional programming pitfalls
- To ensure performance is not degraded by functional constructs
- To create a consistent style across a codebase
- To balance functional and object-oriented approaches

### Key Best Practices

| Practice | Description |
|----------|-------------|
| Prefer Pure Functions | Pure functions are easier to test, reason about, and parallelise. |
| Honour Immutability | Use records, `final` fields, and immutable collections. |
| Use Streams Judiciously | Don't use streams when a simple loop is clearer and more efficient. |
| Keep Lambdas Simple | Avoid overly complex lambda expressions; extract to named methods. |
| Use Method References | Prefer method references over lambdas when possible. |
| Avoid Side Effects in Streams | Side effects in `forEach` or `peek` can cause bugs. |
| Use `Optional` for Return Types | Communicate absence explicitly. |
| Understand Laziness | Know that stream operations are lazy; terminal operations trigger execution. |
| Separate Pure and Impure Code | Isolate side effects at the boundaries (Functional Core, Imperative Shell). |
| Avoid Overusing Currying | Currying is verbose in Java; use it sparingly. |
| Prefer Composition over Inheritance | Compose small functions rather than building deep class hierarchies. |
| Document Complex Pipelines | Add comments for non-obvious stream operations. |

### Annotated Complete Step-by-Step Code Examples

```java
// File: FunctionalBestPractices.java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

// GOOD: Immutable record
record Person(String name, int age) {}

public class FunctionalBestPractices {

    // GOOD: Pure function
    public static int calculateDiscount(int price, int discountPercent) {
        return price * (100 - discountPercent) / 100;
    }

    // GOOD: Higher-order function with clear purpose
    public static <T> List<T> filterList(List<T> list, Predicate<T> predicate) {
        return list.stream().filter(predicate).collect(Collectors.toList());
    }

    // BAD: Impure function with side effect
    private static int total = 0;
    public static int addToTotal(int value) {
        total += value;  // Side effect
        return total;
    }

    public static void main(String[] args) {
        // GOOD: Use pure functions
        System.out.println("Discounted: " + calculateDiscount(100, 10));  // 90

        // GOOD: Use immutable data
        List<Person> people = List.of(
            new Person("Alice", 30),
            new Person("Bob", 25),
            new Person("Charlie", 35)
        );

        // GOOD: Simple, readable stream pipeline
        List<String> adultNames = people.stream()
            .filter(p -> p.age() >= 30)
            .map(Person::name)
            .collect(Collectors.toList());
        System.out.println("Adults: " + adultNames);  // [Alice, Charlie]

        // GOOD: Method reference instead of lambda
        people.stream()
            .map(Person::name)
            .forEach(System.out::println);

        // GOOD: Use Optional for return types
        Optional<Person> youngest = people.stream()
            .min(Comparator.comparingInt(Person::age));
        youngest.ifPresent(p -> System.out.println("Youngest: " + p.name()));
    }
}
```

**Expected Output:**
```
Discounted: 90
Adults: [Alice, Charlie]
Alice
Bob
Charlie
Youngest: Bob
```

**Step-by-Step Explanation:**

1. `calculateDiscount` is a pure function: it depends only on its parameters and has no side effects.
2. `Person` is an immutable record.
3. The stream pipeline is simple and readable: filter, map, collect.
4. Method references (`Person::name`, `System.out::println`) are used instead of lambdas.
5. `Optional` is used for `min()`, which may return no result.

### Real-World Cases

- **Spring Boot:** Functional programming patterns with immutability, pure functions, and functional core/imperative shell architecture.
- **Data Processing:** Stream pipelines for filtering, mapping, and aggregating data.
- **API Design:** `Optional` return types communicate absence; pure functions simplify testing.
- **Concurrency:** Immutability and pure functions enable safe parallel execution.

---

## References Links

- Understand Java's Key Functional Programming Concepts & Features — Douglas C. Schmidt, Vanderbilt University - http://www.dre.vanderbilt.edu/~schmidt/cs253/2020-PDFs/2.2.3-overview-of-java-key-functional-programming-concepts-and-features.pdf
- A Functional Approach to Java — Ben Weidig, O'Reilly Media - https://ebooks.mpdl.mpg.de/ebooks/Record/EB002164551/#content
- Java™ Platform, Standard Edition 11 API Specification — Interface Function<T,R> - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Function.html
- Lambda and JSR 292 Method Handle — OpenJDK Mailing List - https://mail.openjdk.org/pipermail/lambda-dev/2009-December/000150.html
- Functional Programming in Java — Venkat Subramaniam, The Pragmatic Bookshelf - https://pragprog.com/titles/vsjava2/functional-programming-in-java/
- Functional Programming in Java: Concepts, Code Examples, and Best Practices — upGrad - https://www.upgrad.com/blog/functional-programming-in-java/
- Spring Boot Functional Programming — ose-primer (GitHub) - https://github.com/wahidyankf/ose-primer/blob/main/docs/explanation/software-engineering/platform-web/tools/jvm-spring-boot/functional-programming.md
- Infinite Streams in Java — Dominik Gruntz, ACM Digital Library - https://dlnext.acm.org/doi/epdf/10.1145/1168054.1168080
- Java Functional Programming Rules — jabrena/cursor-rules-examples (GitHub) - https://github.com/jabrena/cursor-rules-examples/blob/main/.agents/skills/142-java-functional-programming/references/142-java-functional-programming.md
- Functional Java — GitHub - https://github.com/FunctionalJava/FunctionalJava