# Java Method References: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** A method reference is a compact, readable shorthand syntax for a lambda expression that invokes an existing method by name, using the double-colon operator `::`.

**Technical Definition:** A method reference expression (JLS §15.13) is a syntactic construct of the form `ReferenceType::Identifier`, `Primary::Identifier`, `super::Identifier`, `TypeName.super::Identifier`, `ClassType::new`, or `ArrayType::new`. It is compatible in an assignment, invocation, or casting context with a target type `T` if `T` is a functional interface type and the expression is congruent with the function type of the ground target type derived from `T`. A method reference is a compile-time construct: the compiler resolves which method is being referenced and generates an `invokedynamic` call site, avoiding the boilerplate of an explicit lambda body.

**Beginner-Friendly Explanation:** A method reference is a shortcut for a lambda expression that does nothing but call an existing method. Instead of writing `s -> System.out.println(s)`, you write `System.out::println`. The `::` operator says "use this existing method as the implementation of the functional interface." Method references make code shorter, cleaner, and easier to read when the lambda body is just a method call.

### Key Characteristics

Method references are characterized by their **conciseness** (they eliminate redundant parameter declarations), **type inference** (the compiler determines the method signature from the target functional interface), and **readability** (they clearly express intent by naming the method being called). They are not first-class objects with an inherent type; their type is always inferred from the context in which they appear. There are exactly four kinds of method references: static method references, bound instance method references, unbound instance method references, and constructor references. Method references are non-capturing when they do not close over any variables, allowing the JVM to create a single instance that can be reused, which can offer a slight performance advantage over capturing lambdas.

### Prerequisites

Readers should be familiar with Java generics, lambda expressions, and functional interfaces (particularly `Function`, `Predicate`, `Consumer`, `Supplier`, and `BiFunction`). A basic understanding of the Stream API is helpful for the array constructor reference examples.

### Related Programming Areas with Explanation

Method references are related to **functional programming** (they are a syntactic sugar for lambdas, which are the primary vehicle for functional-style code in Java), **API design** (they encourage the use of existing, named methods rather than anonymous lambda bodies), **the Stream API** (many stream operations accept functional interfaces that can be satisfied by method references), and **code readability and maintainability** (method references name the operation being performed, making the code's intent clearer than an equivalent lambda).


## 1. The Four Types of Method References

### 1.1 Static Method References

#### Definitions

- **Core Definition:** A method reference that refers to a `static` method of a class.
- **Technical Definition:** Static method references have the syntax `ContainingClass::staticMethodName`. The arguments of the functional interface map directly to the parameters of the static method. The target type's function type must be congruent with the signature of the referenced static method.
- **Beginner-Friendly Explanation:** You are pointing to a method that belongs to the class itself, not to any particular object. The method is called without needing an instance.

#### Purposes

- To refer to a utility or helper method as a functional interface implementation.
- To replace a lambda whose body is a single static method call.
- To pass a class-level method as a callback or stream operation.

#### Syntax Structures and Rules

Complete general syntax:

```java
ContainingClass::staticMethodName
```

Component breakdown:

- `ContainingClass`: the name of the class that declares the static method.
- `staticMethodName`: the name of the static method being referenced.
- The arguments of the functional interface are passed as arguments to the static method.

Syntax rules: The static method must be accessible from the context where the method reference appears. The signature of the static method (after type inference) must be congruent with the function type of the target functional interface.

Constraints and limitations: You cannot pass explicit arguments inside the method reference syntax. If the static method requires arguments that are not supplied by the functional interface, the method reference is not valid.

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Static method reference with `Integer::parseInt`**

```java
import java.util.function.Function;

public class StaticMethodRefExample {
    public static void main(String[] args) {
        // Step 1: Define a Function that takes a String and returns an int
        // Integer::parseInt is a static method reference to Integer.parseInt(String)
        Function<String, Integer> parser = Integer::parseInt;

        // Step 2: Apply the function
        int result = parser.apply("123");
        System.out.println("Parsed: " + result);
    }
}
```

Expected Output:

```
Parsed: 123
```

**Why the code produces this result:** `Integer::parseInt` is a static method reference. The `Function<String, Integer>` interface has a functional method `apply(String)` that returns `Integer`. The compiler matches this to the static method `Integer.parseInt(String)`, which takes a `String` and returns an `int` (autoboxed to `Integer`).

**Example 2: Static method reference in a stream pipeline**

```java
import java.util.stream.Stream;

public class StaticMethodRefStreamExample {
    public static void main(String[] args) {
        // Math::sqrt is a static method reference to Math.sqrt(double)
        Stream.of(4.0, 9.0, 16.0, 25.0)
              .map(Math::sqrt)
              .forEach(System.out::println);
    }
}
```

Expected Output:

```
2.0
3.0
4.0
5.0
```

**Why the code produces this result:** `Math::sqrt` refers to the static method `Math.sqrt(double)`. The `map` operation applies this function to each element, producing a stream of square roots.

**Real-World Cases with Explanation**

In a data parsing pipeline, `Integer::parseInt` or `Double::parseDouble` are used to convert strings to numbers. In a scientific computing application, `Math::sqrt`, `Math::sin`, or `Math::log` are used as transformation functions. In a validation system, `Objects::isNull` or `Objects::nonNull` are used as predicates.

**References Links**

- Method References (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html
- JLS §15.13: Method Reference Expressions – https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.13

### 1.2 Bound Instance Method References

#### Definitions

- **Core Definition:** A method reference that refers to an instance method of an existing, specific external object.
- **Technical Definition:** Bound instance method references have the syntax `containingObject::instanceMethodName`. The receiver object is fixed at the time the method reference is created. The arguments of the functional interface map directly to the parameters of the instance method.
- **Beginner-Friendly Explanation:** You are pointing to a method of a particular object that already exists. The object is "bound" to the method reference, so you do not need to supply it as an argument.

#### Purposes

- To refer to a method of a specific object instance as a functional interface.
- To create a callback that always operates on a known target object.
- To replace a lambda that calls a method on a captured or existing object.

#### Syntax Structures and Rules

Complete general syntax:

```java
containingObject::instanceMethodName
```

Component breakdown:

- `containingObject`: an expression that evaluates to the object whose method is being referenced.
- `instanceMethodName`: the name of the instance method.
- The arguments of the functional interface are passed as arguments to the instance method. The receiver (`containingObject`) is supplied by the method reference itself.

Syntax rules: The instance method must be accessible. The functional interface's parameter list must match the instance method's parameter list (excluding the receiver).

Constraints and limitations: The object must exist before the method reference is created. The method reference captures the object reference at creation time.

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Bound method reference with `System.out::println`**

```java
import java.util.function.Consumer;

public class BoundMethodRefExample {
    public static void main(String[] args) {
        // System.out is the bound object; println is the instance method
        Consumer<String> printer = System.out::println;

        printer.accept("Hello, Bound Method Reference!");
    }
}
```

Expected Output:

```
Hello, Bound Method Reference!
```

**Why the code produces this result:** `System.out` is the receiver object, and `println` is an instance method of `PrintStream`. The `Consumer<String>` interface's `accept(String)` method takes one argument, which is passed directly to `println(String)`.

**Example 2: Bound method reference on a list instance**

```java
import java.util.ArrayList;
import java.util.List;

public class BoundListExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();

        // list::add is a bound instance method reference to list.add
        list.forEach(System.out::println); // No output; list is empty

        // Using a bound reference to add elements
        list.add("Alpha");
        list.add("Beta");

        list.forEach(System.out::println);
    }
}
```

Expected Output:

```
Alpha
Beta
```

**Why the code produces this result:** `System.out::println` is a bound method reference where `System.out` is the receiver. It is passed to `forEach` as a `Consumer<String>`.

**Real-World Cases with Explanation**

In a GUI application, `button::setEnabled` might be used as a bound method reference to enable a button when a condition is met. In a logging system, `logger::info` is a bound method reference used as a `Consumer<String>` to log messages. In a collection processing pipeline, `list::add` can be used as a `Consumer` to accumulate elements.

**References Links**

- Bound Method References (Dev.java) – https://dev.java/learn/lambdas/method-references/#bound-method-references
- Method References (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html

### 1.3 Unbound Instance Method References

#### Definitions

- **Core Definition:** A method reference that refers to an instance method of an arbitrary object of a specific type. The first argument of the functional interface becomes the target object itself.
- **Technical Definition:** Unbound instance method references have the syntax `ContainingType::instanceMethodName`. The receiver is not fixed; instead, the first parameter of the functional interface supplies the receiver object, and any subsequent parameters become the method arguments.
- **Beginner-Friendly Explanation:** You are pointing to an instance method that will be called on whatever object is passed in as the first argument. The object is "unbound" because it is not fixed at the time the method reference is created.

#### Purposes

- To refer to an instance method that should be applied to an object supplied at call time.
- To create a function that operates on any instance of a given type.
- To replace lambdas like `s -> s.toLowerCase()` with `String::toLowerCase`.

#### Syntax Structures and Rules

Complete general syntax:

```java
ContainingType::instanceMethodName
```

Component breakdown:

- `ContainingType`: the name of the class or interface that declares the instance method.
- `instanceMethodName`: the name of the instance method.
- The first argument of the functional interface becomes the receiver object; subsequent arguments become the method parameters.

Syntax rules: The functional interface's first parameter type must be compatible with `ContainingType`. The remaining parameters must match the instance method's parameter list.

Constraints and limitations: The method must not be `static`. The type must not be a raw type if the method is generic.

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Unbound method reference with `String::toUpperCase`**

```java
import java.util.function.Function;
import java.util.stream.Stream;

public class UnboundMethodRefExample {
    public static void main(String[] args) {
        // String::toUpperCase is an unbound instance method reference
        // The first argument (the String) becomes the receiver
        Stream.of("hello", "world", "java")
              .map(String::toUpperCase)
              .forEach(System.out::println);
    }
}
```

Expected Output:

```
HELLO
WORLD
JAVA
```

**Why the code produces this result:** `String::toUpperCase` refers to the instance method `toUpperCase()` of `String`. In the `map` operation, each string element becomes the receiver for the method call. The result is a new stream of uppercase strings.

**Example 2: Unbound method reference with `String::concat`**

```java
import java.util.function.BiFunction;

public class UnboundConcatExample {
    public static void main(String[] args) {
        // String::concat takes two arguments: the receiver String and the argument String
        BiFunction<String, String, String> concat = String::concat;

        String result = concat.apply("Hello, ", "World!");
        System.out.println(result);
    }
}
```

Expected Output:

```
Hello, World!
```

**Why the code produces this result:** `String::concat` is an unbound instance method reference. The `BiFunction<String, String, String>` interface takes two arguments: the first becomes the receiver (`"Hello, "`), and the second becomes the argument to `concat` (`"World!"`).

**Real-World Cases with Explanation**

In a stream pipeline, `String::length` or `String::isEmpty` are used as unbound method references to extract properties from strings. In a data processing system, `Person::getName` is used to extract names from `Person` objects. In a sorting operation, `String::compareTo` is used as a `Comparator` to sort strings alphabetically.

**References Links**

- Unbound Method References (Dev.java) – https://dev.java/learn/lambdas/method-references/#unbound-method-references
- Method References (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html

### 1.4 Constructor References

#### Definitions

- **Core Definition:** A method reference that refers to a constructor of a class.
- **Technical Definition:** Constructor references have the syntax `ClassName::new`. The functional interface determines which constructor is called based on the number and types of arguments provided by the interface's functional method. A constructor reference `ClassName::new` is treated like a "diamond" instance creation (`new ClassName<>()`), with the type arguments inferred from the target type.
- **Beginner-Friendly Explanation:** You are pointing to a constructor. When the functional interface is invoked, it calls `new` with the appropriate arguments, creating a new instance of the class.

#### Purposes

- To use a constructor as a factory function.
- To create new instances in a stream pipeline (e.g., mapping values to new objects).
- To defer object creation until the functional interface is invoked.

#### Syntax Structures and Rules

Complete general syntax:

```java
ClassName::new
```

Component breakdown:

- `ClassName`: the name of the class whose constructor is being referenced.
- The arguments of the functional interface are passed to the constructor.
- The return type is the class itself.

Syntax rules: The compiler selects the constructor whose signature matches the functional interface's parameter list. If the class is generic, the type arguments are inferred from the target type. A reference to the constructor of a raw type is not expressible.

Constraints and limitations: The constructor must be accessible. If multiple constructors match, the compiler resolves the ambiguity based on the target type.

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Constructor reference with `ArrayList::new`**

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Supplier;

public class ConstructorRefExample {
    public static void main(String[] args) {
        // ArrayList::new is a constructor reference to the no-arg constructor
        Supplier<List<String>> listFactory = ArrayList::new;

        List<String> list = listFactory.get();
        list.add("Hello");
        list.add("World");
        System.out.println(list);
    }
}
```

Expected Output:

```
[Hello, World]
```

**Example 2: Constructor reference with `String::new` in a stream**

```java
import java.util.stream.Stream;

public class ConstructorRefStreamExample {
    public static void main(String[] args) {
        // String::new takes a String and creates a new String (copy constructor)
        Stream.of("a", "b", "c")
              .map(String::new)
              .forEach(s -> System.out.println(s + " (class: " + s.getClass().getSimpleName() + ")"));
    }
}
```

Expected Output:

```
a (class: String)
b (class: String)
c (class: String)
```

**Real-World Cases with Explanation**

In a stream pipeline, `ArrayList::new` or `HashSet::new` are used as downstream collectors to specify the collection type. In a factory pattern, `Person::new` is used to create new `Person` objects from stream elements. In a testing framework, `StringBuilder::new` is used to create a new builder for each test case.

**References Links**

- Constructor References (Dev.java) – https://dev.java/learn/lambdas/method-references/#constructor-method-references
- JLS §15.13.1: Compile-Time Declaration of a Method Reference – https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.13.1


## 2. Syntax, Typing, and Readability Mechanics

### 2.1 Target Typing

#### Definitions

- **Core Definition:** A method reference does not have an implicit type. The compiler figures out what it is by matching its signature against the functional interface expected in that exact context.
- **Technical Definition:** A method reference expression is compatible in an assignment context, invocation context, or casting context with a target type `T` if `T` is a functional interface type and the expression is congruent with the function type of the ground target type derived from `T`. The ground target type is derived from `T` by resolving any wildcards. A method reference expression is congruent with a function type if the function type identifies a single compile-time declaration corresponding to the reference and the return types are compatible.
- **Beginner-Friendly Explanation:** When you write `System.out::println`, the compiler does not know what type it is until it sees how it is used. If you assign it to a `Consumer<String>`, it matches `println(String)`. If you assign it to a `Consumer<Object>`, it matches `println(Object)`. The context provides the target type.

#### Purposes

- To allow the same method reference to be used in different functional interface contexts.
- To enable type inference without explicit type arguments.
- To support the diamond-style inference for generic types.

#### Syntax Structures and Rules

There is no special syntax for target typing; it is an inherent part of how the compiler resolves method references. The target type is always provided by the context: assignment, method invocation, or cast.

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Same method reference, different target types**

```java
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Supplier;

public class TargetTypingExample {
    public static void main(String[] args) {
        // System.out::println can be a Consumer
        Consumer<String> consumer = System.out::println;
        consumer.accept("Hello from Consumer");

        // String::new can be a Supplier<String>
        Supplier<String> supplier = String::new;
        System.out.println("Empty string: '" + supplier.get() + "'");

        // Integer::parseInt can be a Function<String, Integer>
        Function<String, Integer> function = Integer::parseInt;
        System.out.println("Parsed: " + function.apply("42"));
    }
}
```

Expected Output:

```
Hello from Consumer
Empty string: ''
Parsed: 42
```

**Why the code produces this result:** The compiler uses the target type (the variable's declared type) to determine which method signature is being referenced. `System.out::println` matches `Consumer<String>` because `println` accepts a `String`. `String::new` matches `Supplier<String>` because it has a no-arg constructor. `Integer::parseInt` matches `Function<String, Integer>` because it takes a `String` and returns an `int` (autoboxed to `Integer`).

**Real-World Cases with Explanation**

In a generic utility method that accepts a `Function<T, R>`, the caller can pass `String::length` and the compiler will infer the correct types. In a stream pipeline, `.map(String::toUpperCase)` works because the `map` method's parameter type (`Function<? super T, ? extends R>`) provides the target type.

**References Links**

- JLS §15.13.2: Type of a Method Reference – https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.13.2
- Package java.util.function (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html

### 2.2 The "No Argument" Rule

#### Definitions

- **Core Definition:** You cannot pass explicit arguments inside a method reference syntax. If you need to supply hardcoded parameters, you must fall back to a traditional lambda expression.
- **Technical Definition:** A method reference expression does not contain an argument list. The arguments are supplied implicitly by the functional interface when the method reference is invoked. The syntax `System.out::println` is valid; `System.out::println("Item: ")` is a compile-time error because parentheses and arguments are not part of the method reference syntax.
- **Beginner-Friendly Explanation:** A method reference is just a name, not a call. You cannot put arguments inside it. If you need to pass a specific value to the method, you must use a lambda instead.

#### Purposes

- To clarify the distinction between a method reference (a name) and a method invocation (a call with arguments).
- To guide developers toward the correct syntax when hardcoded arguments are needed.

#### Syntax Structures and Rules

Valid method reference syntax:

```java
System.out::println
```

Invalid method reference syntax (compile-time error):

```java
System.out::println("Item: ")  // ERROR: parentheses and arguments not allowed
```

If hardcoded arguments are needed, use a lambda:

```java
item -> System.out.println("Item: " + item)
```

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Valid method reference without arguments**

```java
import java.util.List;

public class ValidMethodRefExample {
    public static void main(String[] args) {
        List<String> items = List.of("Alpha", "Beta", "Gamma");
        items.forEach(System.out::println);
    }
}
```

Expected Output:

```
Alpha
Beta
Gamma
```

**Example 2: Lambda needed for hardcoded arguments**

```java
import java.util.List;

public class LambdaForArgumentsExample {
    public static void main(String[] args) {
        List<String> items = List.of("Alpha", "Beta", "Gamma");

        // This is a lambda because we need to prepend "Item: "
        items.forEach(item -> System.out.println("Item: " + item));
    }
}
```

Expected Output:

```
Item: Alpha
Item: Beta
Item: Gamma
```

**Why the lambda is necessary:** The method reference `System.out::println` can only pass the argument supplied by `forEach` (the item itself). To prepend a fixed string, a lambda expression is required to perform the string concatenation.

**Real-World Cases with Explanation**

In logging, `logger::info` is a method reference that logs the message supplied by the caller. If you need to log a formatted message with a fixed prefix, you use a lambda: `msg -> logger.info("[PREFIX] " + msg)`.

**References Links**

- Method References (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html

### 2.3 Array Constructor References

#### Definitions

- **Core Definition:** A specialized extension of constructor references used frequently inside streams to convert a stream back into a concrete native array.
- **Technical Definition:** Array constructor references have the syntax `TypeName[]::new`. The compiler desugars an array constructor reference `Foo[]::new` to a lambda `i -> new Foo[i]`, and then proceeds as with any other lambda or method reference. The functional interface must accept an `int` (the array size) and return an array of the specified type.
- **Beginner-Friendly Explanation:** `String[]::new` is a recipe for creating a new `String` array of a given size. It is used with `Stream.toArray()` to tell the stream what kind of array to produce.

#### Purposes

- To convert a stream of objects into a native array of the correct type.
- To avoid the unsafe cast from `Object[]` to a typed array.
- To provide a concise way to specify the array type in `toArray()`.

#### Syntax Structures and Rules

Complete general syntax:

```java
TypeName[]::new
```

Component breakdown:

- `TypeName`: the element type of the array.
- `new`: the constructor reference keyword.
- The functional interface must have a functional method that takes an `int` and returns an array of `TypeName`.

Syntax rules: An array constructor reference is always exact. It is typically used with `Stream.toArray(IntFunction<A[]>)` or `IntStream.toArray()`.

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Converting a stream to a String array**

```java
import java.util.stream.Stream;

public class ArrayConstructorRefExample {
    public static void main(String[] args) {
        String[] array = Stream.of("apple", "banana", "cherry")
                               .toArray(String[]::new);

        for (String s : array) {
            System.out.println(s);
        }
    }
}
```

Expected Output:

```
apple
banana
cherry
```

**Why the code produces this result:** `String[]::new` is an `IntFunction<String[]>` that creates a new `String` array of the specified size. The `toArray` method uses this function to allocate the array and fill it with the stream elements.

**Example 2: Converting an IntStream to an int array**

```java
import java.util.stream.IntStream;

public class IntArrayConstructorRefExample {
    public static void main(String[] args) {
        int[] array = IntStream.rangeClosed(1, 5).toArray();

        for (int n : array) {
            System.out.print(n + " ");
        }
    }
}
```

Expected Output:

```
1 2 3 4 5 
```

**Why the code produces this result:** `IntStream.toArray()` internally uses an array constructor reference equivalent to `int[]::new` to create the result array.

**Real-World Cases with Explanation**

In a data export system, `toArray(String[]::new)` is used to convert a stream of records into an array for use with a legacy API. In a numerical computing application, `IntStream.toArray()` is used to convert a range of integers into a native `int[]` for use with a matrix library.

**References Links**

- How do Java 8 array constructor references work? (Stack Overflow, Brian Goetz) – https://stackoverflow.com/questions/29447561/how-do-java-8-array-constructor-references-work
- JLS §15.13.1: Compile-Time Declaration of a Method Reference – https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.13.1


## Summary Table of Method Reference Types

| Type | Syntax | Example | Corresponding Lambda |
|---|---|---|---|
| Static Method Reference | `ContainingClass::staticMethodName` | `Integer::parseInt` | `s -> Integer.parseInt(s)` |
| Bound Instance Method Reference | `containingObject::instanceMethodName` | `System.out::println` | `s -> System.out.println(s)` |
| Unbound Instance Method Reference | `ContainingType::instanceMethodName` | `String::toUpperCase` | `s -> s.toUpperCase()` |
| Constructor Reference | `ClassName::new` | `ArrayList::new` | `() -> new ArrayList<>()` |
| Array Constructor Reference | `TypeName[]::new` | `String[]::new` | `i -> new String[i]` |


## Version-Specific Notes

- Method references were introduced in Java 8 as part of JSR 335 (Lambda Expressions for the Java Programming Language).
- Array constructor references (`TypeName[]::new`) are always exact and were part of the Java 8 specification.
- Method references to generic methods or generic constructors may require explicit type arguments in some contexts, although the compiler often infers them from the target type.
- Method references are not serializable by default; if a method reference needs to be serialized, the target functional interface must extend `Serializable`.


## References

- Method References (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html
- JLS §15.13: Method Reference Expressions – https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.13
- JLS §15.13.1: Compile-Time Declaration of a Method Reference – https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.13.1
- JLS §15.13.2: Type of a Method Reference – https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.13.2
- Writing Lambda Expressions as Method References (Dev.java) – https://dev.java/learn/lambdas/method-references/
- Overview of Java Method References (Vanderbilt University) – http://www.dre.vanderbilt.edu/~schmidt/cs254/2023-PDFs/2.5.6-overview-of-java-method-references.pdf
- Get started with method references in Java (InfoWorld) – https://www.infoworld.com/article/2264379/get-started-with-method-references-in-java.html
- How do Java 8 array constructor references work? (Stack Overflow) – https://stackoverflow.com/questions/29447561/how-do-java-8-array-constructor-references-work
- Package java.util.function (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html
- JSR 335: Lambda Expressions for the Java Programming Language – https://cr.openjdk.org/~briangoetz/lambda/lambda-libraries-final.html