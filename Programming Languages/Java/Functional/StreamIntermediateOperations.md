# Java Stream API Intermediate Operations: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** The Java Stream API intermediate operations are a set of lazy, stream-returning operations that transform, filter, or reorder the elements of a stream pipeline. They are the building blocks of functional-style data processing in Java, introduced in Java 8 and expanded in Java 9 and Java 16.

**Technical Definition:** Intermediate operations are methods on the `Stream<T>` interface (and its primitive specializations `IntStream`, `LongStream`, and `DoubleStream`) that return a new stream as their result. They are divided into two categories: *stateless* operations, which retain no state from previously seen elements when processing a new value (e.g., `filter`, `map`, `flatMap`, `peek`, `mapMulti`), and *stateful* operations, which may incorporate state from previously seen elements in processing new values (e.g., `distinct`, `sorted`, `limit`, `skip`, `takeWhile`, `dropWhile`). Intermediate operations are lazy: they do not begin processing the source data until a terminal operation is invoked.

**Beginner-Friendly Explanation:** Imagine you have a conveyor belt of items (your data). Intermediate operations are the stations along the belt where you can inspect, transform, remove, or reorder the items. Nothing happens at these stations until someone at the end of the belt (the terminal operation) actually asks for the final product. This laziness means you can chain many operations together without creating intermediate collections, making your code both concise and efficient.

### Key Characteristics

- Intermediate operations are characterized by several fundamental properties. 
- They are **lazy**, meaning they are not executed until a terminal operation is called. 
- They are **composable**, allowing multiple operations to be chained into a pipeline. 
- Stateless operations can be processed in a single pass, whether sequential or parallel, with minimal data buffering, while stateful operations may require multiple passes or significant data buffering. 
- Some intermediate operations are **short-circuiting** (`limit`, `takeWhile`, `dropWhile`), meaning they may produce a finite result from an infinite input. 
- The operations are **non-interfering** (they do not modify the stream source) and generally **stateless** in terms of their behavioral parameters.

### Prerequisites

- Readers should be familiar with Java generics, lambda expressions, method references, and the `java.util.function` package (particularly `Predicate`, `Function`, `Consumer`, and `BiConsumer`). 
- A basic understanding of the `Stream` interface, stream pipelines, and terminal operations (such as `collect`, `forEach`, and `reduce`) is assumed. 
- For `mapMulti`, knowledge of Java 16 features is helpful. For `takeWhile` and `dropWhile`, awareness of Java 9 enhancements to the Stream API is beneficial.

### Related Programming Areas with Explanation

- The Stream API intermediate operations are related to **functional programming** (through the use of higher-order functions and immutability).
- **Data processing pipelines** (similar to Unix pipes and SQL query planning).
- **Parallel computing** (through the `parallel()` method and the `ForkJoinPool`).
- **Reactive programming** (which shares the concept of lazy, composable data streams). 
- They are also closely related to the **Java Collections Framework**, as streams are often created from collections via the `stream()` and `parallelStream()` methods.

## Core Concepts and Key Features

### 1. Stateless Intermediate Operations

Stateless operations process each element independently, without retaining any state from previously seen elements. They can be processed in a single pass, whether sequential or parallel, with minimal data buffering.

#### 1.1 filter(Predicate)

**Definitions**

- **Core Definition:** Evaluates each element against a condition and keeps only those that match.
- **Technical Definition:** `Stream<T> filter(Predicate<? super T> predicate)` returns a stream consisting of the elements of this stream that match the given predicate.
- **Beginner-Friendly Explanation:** `filter` is like a sieve: it lets through only the items that satisfy a condition and blocks everything else.

**Purposes**

- To select a subset of elements based on a boolean condition.
- To reduce the number of elements passed to subsequent operations.
- To implement where-clause semantics in stream pipelines.

**Syntax Structures and Rules**

Complete general syntax:

```java
Stream<T> filter(Predicate<? super T> predicate)
```

Component breakdown:

- `predicate`: a non-interfering, stateless predicate to apply to each element to determine if it should be included.
- Return value: the new stream containing only matching elements.

Syntax rules: The predicate must be non-interfering and stateless. The number of output elements may be less than the number of input elements. `filter` cannot change the type or value of the elements it processes.

Constraints and limitations: The predicate must not modify the stream source. For parallel streams, the predicate must be thread-safe if it accesses shared state.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Filtering even numbers**

```java
import java.util.stream.Stream;

public class FilterExample {
    public static void main(String[] args) {
        // Step 1: Create a stream of integers
        Stream<Integer> numbers = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // Step 2: Apply filter to keep only even numbers
        numbers.filter(n -> n % 2 == 0)
               .forEach(System.out::println); // Terminal operation
    }
}
```

Expected Output:

```
2
4
6
8
10
```

**Why the code produces this result:** The predicate `n -> n % 2 == 0` returns `true` only for even numbers. The `filter` operation passes only those elements to `forEach`, which prints them. The odd numbers are discarded.

**Example 2: Filtering strings by length**

```java
import java.util.List;
import java.util.stream.Collectors;

public class FilterStringExample {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "cherry", "date", "fig");

        // Keep words with length greater than 4
        List<String> longWords = words.stream()
            .filter(w -> w.length() > 4)
            .collect(Collectors.toList()); // Terminal operation

        System.out.println(longWords);
    }
}
```

Expected Output:

```
[apple, banana, cherry]
```

**Why the code produces this result:** The predicate checks if the length of each word is greater than 4. “date” (4) and “fig” (3) are filtered out. The remaining words are collected into a list.

**Real-World Cases with Explanation**

In an e-commerce application, `filter` is used to select products within a price range or with a specific category. In a log analysis system, `filter` is used to extract only error-level log entries. In a user management system, `filter` is used to find active users or users with a specific role.

**References Links**

- Stream.filter (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#filter-java.util.function.Predicate-
- Stream.filter (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#filter(java.util.function.Predicate)

#### 1.2 map(Function)

**Definitions**

- **Core Definition:** Transforms each element into another object or value.
- **Technical Definition:** `<R> Stream<R> map(Function<? super T, ? extends R> mapper)` returns a stream consisting of the results of applying the given function to the elements of this stream.
- **Beginner-Friendly Explanation:** `map` is like a translator: it takes each item and converts it into something else, one by one.

**Purposes**

- To transform each element of the stream into a new value or type.
- To extract a property from each element.
- To prepare data for subsequent operations that require a different type.

**Syntax Structures and Rules**

Complete general syntax:

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper)
```

Component breakdown:

- `mapper`: a non-interfering, stateless function to apply to each element.
- `R`: the element type of the new stream.
- Return value: the new stream of transformed elements.

Syntax rules: The mapper must be non-interfering and stateless. The number of output elements is always equal to the number of input elements (one-to-one mapping).

Constraints and limitations: The mapper must not modify the stream source. For parallel streams, the mapper must be thread-safe if it accesses shared state.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Mapping to uppercase**

```java
import java.util.stream.Stream;

public class MapExample {
    public static void main(String[] args) {
        // Step 1: Create a stream of strings
        Stream<String> words = Stream.of("hello", "world", "java");

        // Step 2: Map each string to its uppercase form
        words.map(String::toUpperCase)
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

**Why the code produces this result:** The `map` operation applies the method reference `String::toUpperCase` to each string, producing a new stream of uppercase strings.

**Example 2: Mapping objects to properties**

```java
import java.util.List;
import java.util.stream.Collectors;

class Person {
    String name;
    int age;
    Person(String name, int age) { this.name = name; this.age = age; }
    String getName() { return name; }
    int getAge() { return age; }
}

public class MapPropertyExample {
    public static void main(String[] args) {
        List<Person> people = List.of(
            new Person("Alice", 30),
            new Person("Bob", 25),
            new Person("Charlie", 35)
        );

        // Extract names from Person objects
        List<String> names = people.stream()
            .map(Person::getName)
            .collect(Collectors.toList());

        System.out.println(names);
    }
}
```

Expected Output:

```
[Alice, Bob, Charlie]
```

**Why the code produces this result:** The `map` operation extracts the `name` property from each `Person` object using the method reference `Person::getName`, resulting in a stream of strings.

**Real-World Cases with Explanation**

In a data processing pipeline, `map` is used to convert database entities to DTOs (Data Transfer Objects). In a financial application, `map` is used to convert transaction records to monetary amounts. In a graphics application, `map` is used to transform 3D coordinates to 2D screen coordinates.

**References Links**

- Stream.map (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#map-java.util.function.Function-
- Stream.map (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#map(java.util.function.Function)

#### 1.3 flatMap(Function)

**Definitions**

- **Core Definition:** Replaces each element with a stream of elements, flattening multiple streams into one.
- **Technical Definition:** `<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)` returns a stream consisting of the results of replacing each element of this stream with the contents of a mapped stream produced by applying the provided mapping function to each element.
- **Beginner-Friendly Explanation:** `flatMap` is like opening a box that contains smaller boxes and spreading all their contents onto a single table. Each element is expanded into a stream, and all these streams are concatenated.

**Purposes**

- To flatten nested structures (e.g., a list of lists into a single list).
- To transform each element into zero or more elements.
- To handle one-to-many mappings in a stream pipeline.

**Syntax Structures and Rules**

Complete general syntax:

```java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)
```

Component breakdown:

- `mapper`: a non-interfering, stateless function that produces a stream for each element.
- `R`: the element type of the new stream.
- Return value: the flattened stream.

Syntax rules: The mapper must return a stream for each element. The resulting streams are concatenated into a single stream. `flatMap` can change the number of elements (zero or more per input element).

Constraints and limitations: The mapper must not modify the stream source. Creating intermediate streams for each element can be costly for small numbers of elements; `mapMulti` (Java 16) is often more efficient in such cases.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Flattening a list of lists**

```java
import java.util.List;
import java.util.stream.Collectors;

public class FlatMapExample {
    public static void main(String[] args) {
        List<List<Integer>> nested = List.of(
            List.of(1, 2),
            List.of(3, 4),
            List.of(5, 6)
        );

        // Flatten the nested list into a single list
        List<Integer> flat = nested.stream()
            .flatMap(List::stream)
            .collect(Collectors.toList());

        System.out.println(flat);
    }
}
```

Expected Output:

```
[1, 2, 3, 4, 5, 6]
```

**Why the code produces this result:** Each inner list is converted to a stream using `List::stream`, and `flatMap` concatenates all these streams into a single stream of integers.

**Example 2: Splitting strings into words**

```java
import java.util.stream.Stream;

public class FlatMapWordsExample {
    public static void main(String[] args) {
        Stream<String> lines = Stream.of(
            "Hello World",
            "Java Stream API"
        );

        lines.flatMap(line -> Stream.of(line.split(" ")))
             .forEach(System.out::println);
    }
}
```

Expected Output:

```
Hello
World
Java
Stream
API
```

**Why the code produces this result:** Each line is split into an array of words, which is converted to a stream. `flatMap` flattens all the word streams into a single stream.

**Real-World Cases with Explanation**

In a search engine, `flatMap` is used to flatten search results from multiple indexes. In a file processing system, `flatMap` is used to read multiple files and process all lines as a single stream. In a recommendation system, `flatMap` is used to expand each user's list of interests into individual interest items.

**References Links**

- Stream.flatMap (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#flatMap-java.util.function.Function-
- Stream.flatMap (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#flatMap(java.util.function.Function)

#### 1.4 peek(Consumer)

**Definitions**

- **Core Definition:** Performs an action on each element (primarily used for debugging) without altering the stream.
- **Technical Definition:** `Stream<T> peek(Consumer<? super T> action)` returns a stream consisting of the elements of this stream, additionally performing the provided action on each element as elements are consumed from the resulting stream.
- **Beginner-Friendly Explanation:** `peek` is like a window in a pipeline: you can look at each item as it passes by without changing it. It is mainly used to see what is happening inside a stream for debugging.

**Purposes**

- To inspect elements at a specific point in a stream pipeline for debugging.
- To perform side effects (e.g., logging) without modifying the stream.
- To verify the state of elements before and after other operations.

**Syntax Structures and Rules**

Complete general syntax:

```java
Stream<T> peek(Consumer<? super T> action)
```

Component breakdown:

- `action`: a non-interfering action to perform on each element as it is consumed.
- Return value: the same stream (with the action performed on each element).

Syntax rules: `peek` is an intermediate operation that does not change the elements. The action is performed only when the stream is consumed by a terminal operation. For parallel streams, the action may be called at whatever time and in whatever thread the element is made available.

Constraints and limitations: `peek` should not be used for stateful operations in parallel streams without proper synchronization. In cases where the stream implementation can optimize away the production of some elements (such as with short-circuiting operations), the action may not be invoked for those elements.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Debugging a stream pipeline**

```java
import java.util.stream.Stream;
import java.util.stream.Collectors;

public class PeekExample {
    public static void main(String[] args) {
        Stream.of("one", "two", "three", "four")
            .filter(e -> e.length() > 3)
            .peek(e -> System.out.println("Filtered value: " + e))
            .map(String::toUpperCase)
            .peek(e -> System.out.println("Mapped value: " + e))
            .collect(Collectors.toList());
    }
}
```

Expected Output:

```
Filtered value: three
Mapped value: THREE
Filtered value: four
Mapped value: FOUR
```

**Why the code produces this result:** The first `peek` prints elements that pass the `filter` (length > 3). The second `peek` prints elements after `map` has converted them to uppercase. The order reflects the pipeline: filter → peek → map → peek → collect.

**Example 2: Logging without modifying the stream**

```java
import java.util.stream.Stream;

public class PeekLoggingExample {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5)
            .peek(n -> System.out.println("Processing: " + n))
            .map(n -> n * n)
            .forEach(System.out::println);
    }
}
```

Expected Output:

```
Processing: 1
1
Processing: 2
4
Processing: 3
9
Processing: 4
16
Processing: 5
25
```

**Why the code produces this result:** The `peek` operation logs each element before it is squared by `map`. The output interleaves the log messages with the squared results, showing the order of processing.

**Real-World Cases with Explanation**

In a data pipeline, `peek` is used to log intermediate results for debugging. In a monitoring system, `peek` is used to count elements passing through a specific stage without altering the stream. In a testing framework, `peek` is used to verify that certain elements reach a particular point in the pipeline.

**References Links**

- Stream.peek (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#peek-java.util.function.Consumer-
- Stream.peek (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#peek(java.util.function.Consumer)

#### 1.5 mapMulti(BiConsumer) (Added in Java 16)

**Definitions**

- **Core Definition:** An alternative to `flatMap` that allows you to map elements to multiple elements using a pusher-style buffer, which is highly efficient for flat-mapping small numbers of elements.
- **Technical Definition:** `<R> Stream<R> mapMulti(BiConsumer<? super T, ? super Consumer<R>> mapper)` returns a stream consisting of the results of replacing each element of this stream with multiple elements, specifically zero or more elements. Replacement is performed by applying the provided mapping function to each element in conjunction with a consumer argument that accepts replacement elements.
- **Beginner-Friendly Explanation:** `mapMulti` is like `flatMap` but more direct. Instead of creating a new stream for each element, you simply push the resulting elements into a consumer. This makes it faster when you only need to produce a few elements per input element.

**Purposes**

- To replace each element with zero or more elements using an imperative style.
- To improve performance over `flatMap` when the number of output elements per input is small.
- To perform conditional logic during element expansion without creating intermediate streams.

**Syntax Structures and Rules**

Complete general syntax:

```java
<R> Stream<R> mapMulti(BiConsumer<? super T, ? super Consumer<R>> mapper)
```

Component breakdown:

- `mapper`: a non-interfering, stateless function that accepts the current element and a `Consumer<R>` to which replacement elements are pushed.
- `R`: the element type of the new stream.
- Return value: the new stream containing the pushed elements.

Syntax rules: The mapper is called once for each input element. Inside the mapper, you call `consumer.accept(value)` zero or more times to emit replacement elements. The consumer must only be used within the scope of the mapper function.

Constraints and limitations: The mapper must not modify the stream source. The consumer should not be used outside the mapper function. `mapMulti` is not a short-circuiting operation.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Expanding strings to uppercase and lowercase**

```java
import java.util.List;
import java.util.stream.Stream;

public class MapMultiExample {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "cherry");

        Stream<String> expanded = words.stream()
            .mapMulti((word, consumer) -> {
                consumer.accept(word.toUpperCase());
                consumer.accept(word.toLowerCase());
            });

        expanded.forEach(System.out::println);
    }
}
```

Expected Output:

```
APPLE
apple
BANANA
banana
CHERRY
cherry
```

**Why the code produces this result:** For each word, the mapper pushes the uppercase and lowercase versions into the consumer. The consumer accumulates these elements, which are then emitted as the resulting stream.

**Example 2: Conditional element emission**

```java
import java.util.stream.Stream;

public class MapMultiConditionalExample {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
            .mapMulti((num, consumer) -> {
                if (num % 2 == 0) {
                    consumer.accept(num * 10);
                }
                if (num % 3 == 0) {
                    consumer.accept(num * 100);
                }
            })
            .forEach(System.out::println);
    }
}
```

Expected Output:

```
20
30
40
60
600
80
90
900
100
```

**Why the code produces this result:** For each number, the mapper checks divisibility. Even numbers are multiplied by 10; numbers divisible by 3 are multiplied by 100. Some numbers (like 6) produce two outputs. This shows how `mapMulti` allows conditional logic without creating intermediate streams.

**Real-World Cases with Explanation**

In a data transformation pipeline, `mapMulti` is used to expand each record into multiple output records based on complex business rules. In a JSON processing system, `mapMulti` is used to flatten nested arrays while applying conditional filtering. In a game engine, `mapMulti` is used to generate multiple particles or effects from a single event.

**References Links**

- Stream.mapMulti (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#mapMulti(java.util.function.BiConsumer)
- IntStream.mapMulti (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/IntStream.html#mapMulti(java.util.stream.IntStream.IntMapMultiConsumer)

### 2. Stateful Intermediate Operations

Stateful operations may incorporate state from previously seen elements in processing new values. They may need to process the entire input before producing a result, and under parallel computation, some pipelines containing stateful intermediate operations may need to be executed in multiple passes or may need to buffer significant data.

#### 2.1 distinct()

**Definitions**

- **Core Definition:** Removes duplicate elements based on their `equals()` method.
- **Technical Definition:** `Stream<T> distinct()` returns a stream consisting of the distinct elements (according to `Object.equals(Object)`) of this stream. For ordered streams, the selection of distinct elements is stable (for duplicated elements, the element appearing first in the encounter order is preserved). For unordered streams, no stability guarantees are made.
- **Beginner-Friendly Explanation:** `distinct` is like a filter that removes repeated items, keeping only the first occurrence of each unique item.

**Purposes**

- To eliminate duplicate elements from a stream.
- To ensure that each element appears only once in the result.
- To prepare data for operations that require unique elements.

**Syntax Structures and Rules**

Complete general syntax:

```java
Stream<T> distinct()
```

Component breakdown:

- Return value: the new stream containing only distinct elements.
- No parameters.

Syntax rules: Elements are compared using `equals()`. For ordered streams, the first occurrence of each duplicate is preserved. `distinct` is a stateful intermediate operation that buffers seen elements.

Constraints and limitations: Requires elements to have a consistent `equals()` implementation. If the stream is sorted, the comparison method used for sorting must be consistent with `equals()` for `distinct` to work correctly.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Removing duplicate integers**

```java
import java.util.stream.Stream;

public class DistinctExample {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 2, 4, 1, 5, 3, 6)
            .distinct()
            .forEach(System.out::println);
    }
}
```

Expected Output:

```
1
2
3
4
5
6
```

**Why the code produces this result:** The `distinct` operation removes duplicate values, keeping only the first occurrence of each integer. The encounter order is preserved.

**Example 2: Removing duplicate strings**

```java
import java.util.List;
import java.util.stream.Collectors;

public class DistinctStringExample {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "apple", "cherry", "banana", "date");

        List<String> unique = words.stream()
            .distinct()
            .collect(Collectors.toList());

        System.out.println(unique);
    }
}
```

Expected Output:

```
[apple, banana, cherry, date]
```

**Why the code produces this result:** Duplicate strings are removed using `equals()`. The first occurrence of each unique word is preserved in the encounter order.

**Real-World Cases with Explanation**

In a data cleansing pipeline, `distinct` is used to remove duplicate records from a dataset. In a analytics system, `distinct` is used to count unique visitors or unique product views. In a text processing application, `distinct` is used to extract a list of unique words from a document.

**References Links**

- Stream.distinct (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#distinct--
- Stream.distinct (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#distinct()

#### 2.2 sorted() / sorted(Comparator)

**Definitions**

- **Core Definition:** Sorts the elements of the stream.
- **Technical Definition:** `Stream<T> sorted()` returns a stream consisting of the elements of this stream, sorted according to natural order. If the elements of this stream are not `Comparable`, a `ClassCastException` may be thrown when the terminal operation is executed. `Stream<T> sorted(Comparator<? super T> comparator)` returns a stream consisting of the elements of this stream, sorted according to the provided `Comparator`. For ordered streams, the sort is stable. For unordered streams, no stability guarantees are made.
- **Beginner-Friendly Explanation:** `sorted` arranges the elements in order. You can use natural order (like numbers from smallest to largest) or provide your own comparison rule.

**Purposes**

- To arrange elements in ascending or descending order.
- To prepare data for operations that require sorted input (e.g., `takeWhile`, `dropWhile`).
- To implement sorting logic for custom objects.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
Stream<T> sorted()
Stream<T> sorted(Comparator<? super T> comparator)
```

Component breakdown:

- No parameters for natural ordering.
- `comparator`: a non-interfering, stateless `Comparator` to be used to compare stream elements.
- Return value: the new sorted stream.

Syntax rules: For natural ordering, elements must implement `Comparable`. For custom ordering, a `Comparator` must be provided. The sort is stable for ordered streams.

Constraints and limitations: `sorted()` is a stateful operation that buffers all elements before producing any output. On parallel streams, sorting can be expensive. The comparator must be non-interfering and stateless.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Natural ordering**

```java
import java.util.stream.Stream;

public class SortedNaturalExample {
    public static void main(String[] args) {
        Stream.of(5, 2, 8, 1, 9, 3)
            .sorted()
            .forEach(System.out::println);
    }
}
```

Expected Output:

```
1
2
3
5
8
9
```

**Why the code produces this result:** The `sorted()` method uses natural ordering for integers, producing ascending order.

**Example 2: Custom comparator (descending order)**

```java
import java.util.Comparator;
import java.util.stream.Stream;

public class SortedComparatorExample {
    public static void main(String[] args) {
        Stream.of("banana", "apple", "cherry", "date")
            .sorted(Comparator.reverseOrder())
            .forEach(System.out::println);
    }
}
```

Expected Output:

```
date
cherry
banana
apple
```

**Why the code produces this result:** The `Comparator.reverseOrder()` comparator sorts strings in reverse lexicographic order.

**Example 3: Sorting custom objects**

```java
import java.util.List;
import java.util.Comparator;
import java.util.stream.Collectors;

class Person {
    String name;
    int age;
    Person(String name, int age) { this.name = name; this.age = age; }
    public String toString() { return name + " (" + age + ")"; }
}

public class SortedCustomExample {
    public static void main(String[] args) {
        List<Person> people = List.of(
            new Person("Alice", 30),
            new Person("Bob", 25),
            new Person("Charlie", 35)
        );

        List<Person> sorted = people.stream()
            .sorted(Comparator.comparingInt(p -> p.age))
            .collect(Collectors.toList());

        System.out.println(sorted);
    }
}
```

Expected Output:

```
[Bob (25), Alice (30), Charlie (35)]
```

**Why the code produces this result:** The comparator `Comparator.comparingInt(p -> p.age)` sorts `Person` objects by age in ascending order.

**Real-World Cases with Explanation**

In an e-commerce application, `sorted` is used to display products by price, rating, or popularity. In a reporting system, `sorted` is used to arrange data by date or alphabetical order. In a leaderboard system, `sorted` is used to rank players by score.

**References Links**

- Stream.sorted() (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#sorted--
- Stream.sorted(Comparator) (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#sorted-java.util.Comparator-
- Stream.sorted (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#sorted()

#### 2.3 limit(long)

**Definitions**

- **Core Definition:** Truncates the stream to not exceed a maximum given size.
- **Technical Definition:** `Stream<T> limit(long maxSize)` returns a stream consisting of the elements of this stream, truncated to be no longer than `maxSize` in length.
- **Beginner-Friendly Explanation:** `limit` is like saying “I only want the first N items” from the stream.

**Purposes**

- To restrict the number of elements processed or collected.
- To implement pagination-like behaviour in streams.
- To avoid processing unnecessary elements from an infinite or large stream.

**Syntax Structures and Rules**

Complete general syntax:

```java
Stream<T> limit(long maxSize)
```

Component breakdown:

- `maxSize`: the number of elements the stream should be limited to.
- Return value: the new stream containing at most `maxSize` elements.
- Throws `IllegalArgumentException` if `maxSize` is negative.

Syntax rules: `limit` is a short-circuiting stateful operation. It returns the first `maxSize` elements in encounter order. On parallel streams, `limit` can be expensive for large `maxSize` because it must preserve encounter order.

Constraints and limitations: `maxSize` must be non-negative. The operation may need to buffer elements in parallel streams.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Limiting to first 3 elements**

```java
import java.util.stream.Stream;

public class LimitExample {
    public static void main(String[] args) {
        Stream.of(10, 20, 30, 40, 50, 60)
            .limit(3)
            .forEach(System.out::println);
    }
}
```

Expected Output:

```
10
20
30
```

**Why the code produces this result:** `limit(3)` truncates the stream to the first three elements.

**Example 2: Using `limit` with `skip` for pagination**

```java
import java.util.stream.Stream;

public class LimitSkipExample {
    public static void main(String[] args) {
        // Page 2 (elements 4-6) of a stream
        Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
            .skip(3)
            .limit(3)
            .forEach(System.out::println);
    }
}
```

Expected Output:

```
4
5
6
```

**Why the code produces this result:** `skip(3)` discards the first three elements, and `limit(3)` takes the next three, effectively implementing pagination.

**Real-World Cases with Explanation**

In a web application, `limit` is used to restrict the number of search results returned per page. In a data sampling system, `limit` is used to take a small sample from a large dataset. In a stream processing pipeline, `limit` is used to process only the first N records.

**References Links**

- Stream.limit (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#limit-long-
- Stream.limit (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#limit(long)

#### 2.4 skip(long)

**Definitions**

- **Core Definition:** Discards the first n elements of the stream.
- **Technical Definition:** `Stream<T> skip(long n)` returns a stream consisting of the remaining elements of this stream after discarding the first `n` elements of the stream. If this stream contains fewer than `n` elements then an empty stream will be returned.
- **Beginner-Friendly Explanation:** `skip` is like saying “ignore the first N items” and take the rest.

**Purposes**

- To bypass a specified number of elements at the beginning of a stream.
- To implement offset-based pagination in streams.
- To discard header or metadata elements in a data stream.

**Syntax Structures and Rules**

Complete general syntax:

```java
Stream<T> skip(long n)
```

Component breakdown:

- `n`: the number of leading elements to skip.
- Return value: the new stream after skipping `n` elements.
- Throws `IllegalArgumentException` if `n` is negative.

Syntax rules: `skip` is a stateful operation that must count and discard elements. On parallel streams, `skip` can be expensive for large `n` because it must preserve encounter order.

Constraints and limitations: `n` must be non-negative.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Skipping the first 3 elements**

```java
import java.util.stream.Stream;

public class SkipExample {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5, 6)
            .skip(3)
            .forEach(System.out::println);
    }
}
```

Expected Output:

```
4
5
6
```

**Why the code produces this result:** `skip(3)` discards the first three elements (1, 2, 3) and passes the remaining elements to `forEach`.

**Example 2: Skipping more elements than available**

```java
import java.util.stream.Stream;

public class SkipEmptyExample {
    public static void main(String[] args) {
        Stream.of(1, 2, 3)
            .skip(5)
            .forEach(System.out::println);
    }
}
```

Expected Output:

```
(no output)
```

**Why the code produces this result:** Since the stream contains fewer than 5 elements, `skip(5)` results in an empty stream, and `forEach` does nothing.

**Real-World Cases with Explanation**

In a data import process, `skip` is used to bypass the header row of a CSV file. In a log analysis system, `skip` is used to ignore initial startup log entries. In a pagination system, `skip` is combined with `limit` to retrieve specific pages.

**References Links**

- Stream.skip (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#skip-long-
- Stream.skip (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#skip(long)

#### 2.5 takeWhile(Predicate) (Added in Java 9)

**Definitions**

- **Core Definition:** Returns elements from the stream as long as they match the predicate, stopping as soon as the first element fails the condition (useful for ordered streams).
- **Technical Definition:** `default Stream<T> takeWhile(Predicate<? super T> predicate)` returns, if this stream is ordered, a stream consisting of the longest prefix of elements taken from this stream that match the given predicate. Otherwise, returns a stream consisting of a subset of elements taken from this stream that match the given predicate.
- **Beginner-Friendly Explanation:** `takeWhile` is like taking items off a conveyor belt as long as they meet a condition. As soon as an item fails the condition, you stop and ignore everything after it.

**Purposes**

- To take elements from the beginning of an ordered stream while a condition holds.
- To implement “prefix” operations on sorted or ordered streams.
- To stop processing as soon as a condition fails (short-circuiting).

**Syntax Structures and Rules**

Complete general syntax:

```java
default Stream<T> takeWhile(Predicate<? super T> predicate)
```

Component breakdown:

- `predicate`: a non-interfering, stateless predicate to apply to elements to determine whether to take them.
- Return value: the new stream containing the prefix of matching elements.

Syntax rules: For ordered streams, `takeWhile` returns the longest prefix of elements that match the predicate. For unordered streams, the behaviour is non-deterministic. `takeWhile` is a short-circuiting stateful operation.

Constraints and limitations: On ordered parallel streams, `takeWhile` can be costly because threads must cooperate to find the longest prefix.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Taking numbers while less than 5**

```java
import java.util.stream.Stream;

public class TakeWhileExample {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5, 6, 7, 8)
            .takeWhile(n -> n < 5)
            .forEach(System.out::println);
    }
}
```

Expected Output:

```
1
2
3
4
```

**Why the code produces this result:** The predicate `n -> n < 5` is true for 1, 2, 3, and 4. When the stream reaches 5, the predicate fails, and `takeWhile` stops, discarding 5 and all subsequent elements.

**Example 2: Taking strings while length is less than 4**

```java
import java.util.List;
import java.util.stream.Collectors;

public class TakeWhileStringExample {
    public static void main(String[] args) {
        List<String> words = List.of("a", "ab", "abc", "abcd", "abcde");

        List<String> taken = words.stream()
            .takeWhile(w -> w.length() < 4)
            .collect(Collectors.toList());

        System.out.println(taken);
    }
}
```

Expected Output:

```
[a, ab, abc]
```

**Why the code produces this result:** The stream takes words while their length is less than 4. When “abcd” (length 4) is encountered, the predicate fails, and the operation stops.

**Real-World Cases with Explanation**

In a log processing system, `takeWhile` is used to read log entries until a specific marker is encountered. In a financial application, `takeWhile` is used to process transactions until a certain date. In a network protocol parser, `takeWhile` is used to read data until a delimiter is found.

**References Links**

- Stream.takeWhile (Java SE 9) – https://docs.oracle.com/javase/9/docs/api/java/util/stream/Stream.html#takeWhile-java.util.function.Predicate-
- Stream.takeWhile (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#takeWhile(java.util.function.Predicate)

#### 2.6 dropWhile(Predicate) (Added in Java 9)

**Definitions**

- **Core Definition:** Deletes elements from the stream as long as they match the predicate, and returns the rest of the stream once an element fails the condition.
- **Technical Definition:** `default Stream<T> dropWhile(Predicate<? super T> predicate)` returns, if this stream is ordered, a stream consisting of the remaining elements of this stream after dropping the longest prefix of elements that match the given predicate. Otherwise, returns a stream consisting of the remaining elements of this stream after dropping a subset of elements that match the given predicate.
- **Beginner-Friendly Explanation:** `dropWhile` is like skipping items on a conveyor belt as long as they meet a condition. Once an item fails the condition, you stop skipping and take everything from that point on.

**Purposes**

- To discard elements from the beginning of an ordered stream while a condition holds.
- To implement “suffix” operations on sorted or ordered streams.
- To remove a prefix of elements that match a condition.

**Syntax Structures and Rules**

Complete general syntax:

```java
default Stream<T> dropWhile(Predicate<? super T> predicate)
```

Component breakdown:

- `predicate`: a non-interfering, stateless predicate to apply to elements to determine whether to drop them.
- Return value: the new stream containing the remaining elements.

Syntax rules: For ordered streams, `dropWhile` drops the longest prefix of elements that match the predicate. For unordered streams, the behaviour is non-deterministic. `dropWhile` is a short-circuiting stateful operation.

Constraints and limitations: On ordered parallel streams, `dropWhile` can be costly because threads must cooperate to find the longest prefix.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Dropping numbers while less than 5**

```java
import java.util.stream.Stream;

public class DropWhileExample {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5, 6, 7, 8)
            .dropWhile(n -> n < 5)
            .forEach(System.out::println);
    }
}
```

Expected Output:

```
5
6
7
8
```

**Why the code produces this result:** The predicate `n -> n < 5` is true for 1, 2, 3, and 4, so these are dropped. When 5 is encountered, the predicate fails, and `dropWhile` stops dropping, passing 5 and all subsequent elements.

**Example 2: Dropping strings while length is less than 4**

```java
import java.util.List;
import java.util.stream.Collectors;

public class DropWhileStringExample {
    public static void main(String[] args) {
        List<String> words = List.of("a", "ab", "abc", "abcd", "abcde");

        List<String> dropped = words.stream()
            .dropWhile(w -> w.length() < 4)
            .collect(Collectors.toList());

        System.out.println(dropped);
    }
}
```

Expected Output:

```
[abcd, abcde]
```

**Why the code produces this result:** The stream drops words while their length is less than 4. When “abcd” (length 4) is encountered, the predicate fails, and the operation stops dropping, returning “abcd” and “abcde”.

**Real-World Cases with Explanation**

In a configuration file parser, `dropWhile` is used to skip comment lines at the beginning of a file. In a data stream, `dropWhile` is used to ignore initial readings until a stable state is reached. In a text processing system, `dropWhile` is used to remove leading whitespace or special characters.

**References Links**

- Stream.dropWhile (Java SE 9) – https://docs.oracle.com/javase/9/docs/api/java/util/stream/Stream.html#dropWhile-java.util.function.Predicate-
- Stream.dropWhile (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#dropWhile(java.util.function.Predicate)

## Summary Table of Intermediate Operations

| Operation | Type | Short-Circuiting | Java Version | Description |
|---|---|---|---|---|
| filter | Stateless | No | 8 | Selects elements matching a predicate |
| map | Stateless | No | 8 | Transforms each element |
| flatMap | Stateless | No | 8 | Flattens one-to-many mappings |
| peek | Stateless | No | 8 | Performs a side effect without modifying |
| mapMulti | Stateless | No | 16 | Pushes replacement elements to a consumer |
| distinct | Stateful | No | 8 | Removes duplicates |
| sorted | Stateful | No | 8 | Sorts elements |
| limit | Stateful | Yes | 8 | Truncates to a maximum size |
| skip | Stateful | No | 8 | Discards the first n elements |
| takeWhile | Stateful | Yes | 9 | Takes prefix while predicate is true |
| dropWhile | Stateful | Yes | 9 | Drops prefix while predicate is true |

## Version-Specific Notes

- `filter`, `map`, `flatMap`, `peek`, `distinct`, `sorted`, `limit`, and `skip` were introduced in Java 8.
- `takeWhile` and `dropWhile` were introduced in Java 9.
- `mapMulti` was introduced in Java 16.
- `mapMulti` is available on `Stream`, `IntStream`, `LongStream`, and `DoubleStream`.

## References

- Stream (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html
- Stream (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html
- Stream (Java SE 9) – https://docs.oracle.com/javase/9/docs/api/java/util/stream/Stream.html
- IntStream (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/IntStream.html
- Package java.util.stream (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html
- The Java Tutorials: Aggregate Operations – https://docs.oracle.com/javase/tutorial/collections/streams/index.html
- Java Streams Intermediate Operations (Vanderbilt University) – http://www.dre.vanderbilt.edu/~schmidt/cs253/2022-PDFs/2.5.2-Java-streams-intermediate-operations-filter-and-flatMap.pdf
- Guide to mapMulti in Stream API (Baeldung) – https://www.baeldung.com/java-stream-mapmulti
- Java Stream mapMulti (ZetCode) – https://zetcode.cn/java/stream-mapmulti/
- OpenJDK: Add Stream dropWhile and takeWhile operations – https://mail.openjdk.org/pipermail/lambda-dev/2013-August/010865.html