# Java Streams API: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The Java Streams API, introduced in Java 8, is a framework for declaratively processing sequences of elements. A stream is a sequence of elements supporting sequential and parallel aggregate operations. Streams enable developers to express complex data processing queries with a functional-style, pipeline-based approach.

**Technical Definition:** According to the Java API Specification, "A stream is a sequence of elements supporting sequential and parallel aggregate operations." A stream pipeline consists of a source (which might be an array, a collection, a generator function, an I/O channel, etc.), zero or more intermediate operations (which transform a stream into another stream, such as `filter`), and a terminal operation (which produces a result or side-effect, such as `count` or `forEach`). Streams are lazy; computation on the source data is only performed when the terminal operation is initiated, and source elements are consumed only as needed.

**Beginner-Friendly Explanation:** Think of a stream as an assembly line in a factory. Raw materials (the data source) enter the line. Along the way, workers (intermediate operations) filter, transform, and reshape the materials. At the end of the line, a final worker (terminal operation) packages everything into a finished product (the result). The assembly line doesn't start moving until the final worker is ready — that's lazy evaluation. You describe what you want done, not how to loop through everything step by step.

### Key Characteristics

- **Declarative:** You describe what you want to achieve, not how to iterate through the data.
- **Lazy:** Intermediate operations are not executed until a terminal operation is invoked.
- **Pipeline-Based:** Operations are chained together into a stream pipeline.
- **Single-Use:** A stream can be operated on only once; it cannot be reused after a terminal operation.
- **Non-Interfering:** Stream operations do not modify the underlying data source.
- **Functional:** Stream operations accept lambda expressions and method references.
- **Possibly Unbounded:** Streams can represent infinite sequences (e.g., generators).
- **Parallelisable:** Streams can be processed in parallel with a single method call.

### Prerequisites

- Basic understanding of Java collections and arrays
- Familiarity with lambda expressions and method references
- Knowledge of functional interfaces (`Function`, `Predicate`, `Consumer`, `Supplier`)
- Understanding of generics

### Related Programming Areas

- **Functional Programming:** Streams bring functional-style data processing to Java.
- **Lambda Expressions:** Stream operations rely on lambdas and method references.
- **Collectors:** The `Collectors` utility class provides common reduction implementations.
- **Parallel Computing:** Parallel streams leverage the Fork-Join framework.
- **Optional:** Terminal operations like `findFirst` and `max` return `Optional`.
- **I/O:** `Files.lines()` and `BufferedReader.lines()` return streams.

### Core Concepts / Features

---

## 1. Streams

### Definitions

**Core Definition:** A stream is a sequence of elements from a data source that supports aggregate operations.

**Technical Definition:** "A sequence of elements supporting sequential and parallel aggregate operations." Streams are not data structures; they do not store elements. Instead, they carry values from a source through a pipeline of operations.

**Beginner-Friendly Explanation:** A stream is like a conveyor belt carrying items past a series of workstations. The items come from somewhere (the source), pass through each workstation (operation), and eventually reach the end where something is produced (the result).

### Purposes

- To process collections and arrays declaratively
- To enable functional-style data transformations
- To support parallel processing with minimal code changes
- To express complex queries concisely
- To avoid explicit iteration and mutable state

### Syntax Rules and Structure

**Complete General Syntax:**

```
source.stream()                          // Create a stream
    .intermediateOperation1(...)         // Zero or more intermediate operations
    .intermediateOperation2(...)
    .terminalOperation();                // Exactly one terminal operation
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `source` | The data source (collection, array, I/O channel, generator, etc.). |
| `.stream()` | Creates a sequential stream from the source. |
| Intermediate operations | Transform the stream; return a new stream; lazy. |
| Terminal operation | Produces a result or side-effect; triggers execution. |

**Syntax Rules:**

- A stream pipeline must end with a terminal operation.
- Intermediate operations are lazy and do not execute until the terminal operation is invoked.
- A stream can be consumed only once.
- Behavioral parameters (lambdas) must be non-interfering and stateless.

**Constraints and Limitations:**

- Streams cannot be reused after a terminal operation.
- Modifying the source during stream execution can cause exceptions or incorrect results.
- Streams have no direct element access (no `get(int)` like `List`).

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Stream Pipeline**

```java
// File: BasicStreamDemo.java
import java.util.*;
import java.util.stream.*;

public class BasicStreamDemo {
    public static void main(String[] args) {
        List<String> names = List.of("Alice", "Bob", "Charlie", "David", "Eve");

        // Step 1: Create a stream from the list
        // Step 2: Filter names starting with 'A' or 'B'
        // Step 3: Map to uppercase
        // Step 4: Collect into a list
        List<String> result = names.stream()
            .filter(name -> name.startsWith("A") || name.startsWith("B"))
            .map(String::toUpperCase)
            .collect(Collectors.toList());

        System.out.println(result);  // Output: [ALICE, BOB]
    }
}
```

**Expected Output:**
```
[ALICE, BOB]
```

**Step-by-Step Explanation:**

1. `names.stream()` creates a sequential stream from the list.
2. `.filter(name -> name.startsWith("A") || name.startsWith("B"))` is an intermediate operation that keeps only names starting with A or B.
3. `.map(String::toUpperCase)` transforms each remaining name to uppercase.
4. `.collect(Collectors.toList())` is the terminal operation that gathers the results into a new list.
5. Without the terminal operation, nothing would be executed — the pipeline is lazy.

### Real-World Cases

- **Data Filtering:** Filtering a list of transactions by amount or date.
- **Data Transformation:** Converting entities to DTOs.
- **Aggregation:** Computing totals, averages, and counts.
- **Reporting:** Grouping and summarising data.

---

## 2. Stream Pipeline

### Definitions

**Core Definition:** A stream pipeline is a sequence of operations composed of a source, zero or more intermediate operations, and a terminal operation.

**Technical Definition:** "A stream pipeline consists of a source (which might be an array, a collection, a generator function, an I/O channel, etc.), zero or more intermediate operations (which transform a stream into another stream), and a terminal operation (which produces a result or side-effect)".

**Beginner-Friendly Explanation:** A stream pipeline is the complete assembly line — from where the raw materials enter to where the finished product exits. You can have as many workstations (intermediate operations) as you need, but there must be exactly one final station (terminal operation) that produces the result.

### Purposes

- To structure data processing as a clear, readable sequence of steps
- To separate the "what" (operations) from the "how" (iteration)
- To enable lazy evaluation and optimisation
- To support both sequential and parallel execution

### Syntax Rules and Structure

```
Source → Intermediate Operations (0..n) → Terminal Operation (1)
```

**Component Breakdown:**

| Component | Required? | Description |
|-----------|-----------|-------------|
| Source | Yes | Data source (collection, array, etc.). |
| Intermediate Operations | No (zero or more) | Transform the stream; lazy. |
| Terminal Operation | Yes | Produces result or side-effect; eager. |

**Syntax Rules:**

- Intermediate operations return a new stream, enabling chaining.
- Terminal operations return a non-stream result (value, collection, `Optional`).
- The pipeline is executed only when the terminal operation is invoked.
- Intermediate operations are lazy; terminal operations are eager.

**Constraints and Limitations:**

- A pipeline must have exactly one terminal operation.
- Once a terminal operation is invoked, the stream is consumed and cannot be reused.
- Pipeline order matters for performance (e.g., filter before map).

### Annotated Complete Step-by-Step Code Examples

```java
// File: PipelineDemo.java
import java.util.*;
import java.util.stream.*;

public class PipelineDemo {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // Pipeline: source -> filter -> map -> reduce (terminal)
        int sumOfSquares = numbers.stream()           // Source
            .filter(n -> n % 2 == 0)                  // Intermediate: keep evens
            .map(n -> n * n)                          // Intermediate: square each
            .reduce(0, Integer::sum);                 // Terminal: sum

        System.out.println("Sum of squares of evens: " + sumOfSquares);
        // Output: Sum of squares of evens: 220 (4 + 16 + 36 + 64 + 100)
    }
}
```

**Expected Output:**
```
Sum of squares of evens: 220
```

**Step-by-Step Explanation:**

1. `numbers.stream()` creates the source stream.
2. `.filter(n -> n % 2 == 0)` keeps only even numbers: 2, 4, 6, 8, 10.
3. `.map(n -> n * n)` squares each: 4, 16, 36, 64, 100.
4. `.reduce(0, Integer::sum)` is the terminal operation that sums all elements: 4 + 16 + 36 + 64 + 100 = 220.
5. Without `reduce`, nothing would be executed.

### Real-World Cases

- **E-commerce:** Filter products by category, map to prices, sum total.
- **Log Analysis:** Parse log lines, filter by severity, count occurrences.
- **Data Processing:** Read CSV, parse rows, filter invalid, transform, collect.

---

## 3. Stream Sources

### Definitions

**Core Definition:** A stream source is the origin of the data that flows through a stream pipeline.

**Technical Definition:** "A stream pipeline consists of a source (which might be an array, a collection, a generator function, an I/O channel, etc)". The `java.util.stream.Stream` interface provides static factory methods for creating streams from various sources.

**Beginner-Friendly Explanation:** The stream source is where the data comes from — it's the starting point of the assembly line. Data can come from a list, an array, a file, a random generator, or even a range of numbers.

### Purposes

- To create streams from different types of data sources
- To enable stream processing for collections, arrays, I/O, and generated data
- To support both finite and infinite data sources

### Syntax Rules and Structure

**Stream Sources Table:**

| Source Type | Method | Example |
|-------------|--------|---------|
| Collections | `.stream()` | `list.stream()` |
| Arrays | `Arrays.stream(array)` or `Stream.of(array)` | `Arrays.stream(new int[]{1,2,3})` |
| I/O | `Files.lines(path)` | `Files.lines(Path.of("file.txt"))` |
| Generators | `Stream.generate()` or `Stream.iterate()` | `Stream.generate(Math::random)` |
| Ranges | `IntStream.range()` or `rangeClosed()` | `IntStream.range(1, 10)` |
| Values | `Stream.of()` | `Stream.of("a", "b", "c")` |
| Empty | `Stream.empty()` | `Stream.empty()` |

### Annotated Complete Step-by-Step Code Examples

```java
// File: StreamSourcesDemo.java
import java.util.*;
import java.util.stream.*;
import java.nio.file.*;
import java.io.IOException;

public class StreamSourcesDemo {
    public static void main(String[] args) throws IOException {
        // 1. Collection source
        List<String> list = List.of("A", "B", "C");
        long listCount = list.stream().count();
        System.out.println("List count: " + listCount);  // Output: 3

        // 2. Array source
        int[] array = {1, 2, 3, 4, 5};
        int arraySum = Arrays.stream(array).sum();
        System.out.println("Array sum: " + arraySum);  // Output: 15

        // 3. Generator source (infinite)
        List<Double> randoms = Stream.generate(Math::random)
            .limit(3)
            .collect(Collectors.toList());
        System.out.println("Random count: " + randoms.size());  // Output: 3

        // 4. Range source
        int rangeSum = IntStream.rangeClosed(1, 5).sum();
        System.out.println("Range sum: " + rangeSum);  // Output: 15

        // 5. Stream.of source
        long ofCount = Stream.of("X", "Y", "Z").count();
        System.out.println("Of count: " + ofCount);  // Output: 3

        // 6. Files.lines source (I/O)
        // Path path = Path.of("example.txt");
        // try (Stream<String> lines = Files.lines(path)) {
        //     lines.filter(line -> line.contains("error")).forEach(System.out::println);
        // }
    }
}
```

**Expected Output:**
```
List count: 3
Array sum: 15
Random count: 3
Range sum: 15
Of count: 3
```

**Step-by-Step Explanation:**

1. `list.stream()` creates a stream from a `List`.
2. `Arrays.stream(array)` creates an `IntStream` from an `int[]`.
3. `Stream.generate(Math::random)` creates an infinite stream of random doubles; `.limit(3)` bounds it.
4. `IntStream.rangeClosed(1, 5)` creates a stream of integers from 1 to 5 inclusive.
5. `Stream.of("X", "Y", "Z")` creates a stream from varargs.
6. `Files.lines(path)` creates a stream of lines from a file (commented out as it requires a file).

### Real-World Cases

- **Log Processing:** `Files.lines(logPath).filter(line -> line.contains("ERROR"))`.
- **Data Generation:** `Stream.generate(UUID::randomUUID).limit(100)`.
- **Batch Processing:** `IntStream.range(0, batches).mapToObj(i -> processBatch(i))`.

---

## 4. Intermediate Operations

Intermediate operations transform a stream into another stream. They are lazy — they do not execute until a terminal operation is invoked.

### 4.1 `filter()`

#### Definitions

**Core Definition:** `filter()` returns a stream consisting of the elements that match a given predicate.

**Technical Definition:** "Returns a stream consisting of the elements of this stream that match the given predicate. This is an intermediate operation".

**Beginner-Friendly Explanation:** `filter()` is like a sieve — it keeps only the items that pass the test.

#### Purposes

- To select elements based on a condition
- To remove unwanted elements from a stream
- To implement search and retrieval logic

#### Syntax Rules and Structure

```
Stream<T> filter(Predicate<? super T> predicate)
```

| Component | Description |
|-----------|-------------|
| `predicate` | A `Predicate<T>` that returns `true` for elements to keep. |

#### Annotated Code Example

```java
List<String> names = List.of("Alice", "Bob", "Charlie", "Anna");
List<String> aNames = names.stream()
    .filter(name -> name.startsWith("A"))
    .collect(Collectors.toList());
System.out.println(aNames);  // Output: [Alice, Anna]
```

#### Real-World Cases

- Filtering users by age, status, or role
- Filtering transactions above a threshold
- Filtering log entries by severity level

### 4.2 `map()`

#### Definitions

**Core Definition:** `map()` transforms each element of a stream using a given function.

**Technical Definition:** "Returns a stream consisting of the results of applying the given function to the elements of this stream".

**Beginner-Friendly Explanation:** `map()` is like a translator — it converts each item into something else.

#### Purposes

- To transform elements from one type to another
- To extract properties from objects
- To apply calculations or formatting

#### Syntax Rules and Structure

```
<R> Stream<R> map(Function<? super T, ? extends R> mapper)
```

| Component | Description |
|-----------|-------------|
| `mapper` | A `Function<T, R>` that transforms each element. |

#### Annotated Code Example

```java
List<String> names = List.of("Alice", "Bob", "Charlie");
List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
System.out.println(lengths);  // Output: [5, 3, 7]
```

#### Real-World Cases

- Converting entities to DTOs
- Extracting names from a list of Person objects
- Computing derived values (e.g., tax from price)

### 4.3 `flatMap()`

#### Definitions

**Core Definition:** `flatMap()` transforms each element into a stream and then flattens all the resulting streams into a single stream.

**Technical Definition:** "Returns a stream consisting of the results of replacing each element of this stream with the contents of a mapped stream produced by applying the provided mapping function to each element".

**Beginner-Friendly Explanation:** `flatMap()` is like taking a list of lists and flattening it into a single list. Instead of having nested collections, you get one flat collection.

#### Purposes

- To flatten nested collections
- To expand one element into multiple elements
- To process one-to-many relationships

#### Syntax Rules and Structure

```
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)
```

#### Annotated Code Example

```java
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3, 4), List.of(5));
List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
System.out.println(flat);  // Output: [1, 2, 3, 4, 5]
```

#### Real-World Cases

- Flattening orders per customer
- Processing lines of text into words
- Expanding tags or categories

### 4.4 `distinct()`

#### Definitions

**Core Definition:** `distinct()` returns a stream with duplicate elements removed.

**Technical Definition:** "Returns a stream consisting of the distinct elements (according to `Object.equals(Object)`) of this stream".

**Beginner-Friendly Explanation:** `distinct()` removes duplicates, like a `Set`.

#### Purposes

- To eliminate duplicate elements
- To ensure uniqueness in results

#### Syntax Rules and Structure

```
Stream<T> distinct()
```

#### Annotated Code Example

```java
List<Integer> numbers = List.of(1, 2, 2, 3, 3, 3, 4);
List<Integer> distinct = numbers.stream().distinct().collect(Collectors.toList());
System.out.println(distinct);  // Output: [1, 2, 3, 4]
```

### 4.5 `sorted()`

#### Definitions

**Core Definition:** `sorted()` returns a stream with elements sorted according to natural order or a provided comparator.

**Technical Definition:** "Returns a stream consisting of the elements of this stream, sorted according to natural order" or "sorted according to the provided Comparator".

**Beginner-Friendly Explanation:** `sorted()` arranges elements in order.

#### Purposes

- To sort elements
- To arrange data for presentation or further processing

#### Syntax Rules and Structure

```
Stream<T> sorted()
Stream<T> sorted(Comparator<? super T> comparator)
```

#### Annotated Code Example

```java
List<String> names = List.of("Charlie", "Alice", "Bob");
List<String> sorted = names.stream().sorted().collect(Collectors.toList());
System.out.println(sorted);  // Output: [Alice, Bob, Charlie]
```

### 4.6 `peek()`

#### Definitions

**Core Definition:** `peek()` returns a stream consisting of the elements of this stream, additionally performing the provided action on each element as they are consumed from the resulting stream.

**Technical Definition:** "Returns a stream consisting of the elements of this stream, additionally performing the provided action on each element as elements are consumed from the resulting stream. This is an intermediate operation".

**Beginner-Friendly Explanation:** `peek()` lets you look at each element as it passes through the pipeline, without changing it. It's primarily useful for debugging.

#### Purposes

- To debug stream pipelines
- To inspect elements without modifying them

#### Syntax Rules and Structure

```
Stream<T> peek(Consumer<? super T> action)
```

**Note:** `peek()` is intended for debugging. Using it for side effects in production code is discouraged.

### 4.7 `limit()` and `skip()`

#### Definitions

**`limit()`:** Returns a stream truncated to be no longer than a given size.

**`skip()`:** Returns a stream consisting of the remaining elements after discarding the first n elements.

**Beginner-Friendly Explanation:** `limit()` takes the first n elements; `skip()` skips the first n elements.

#### Syntax Rules and Structure

```
Stream<T> limit(long maxSize)
Stream<T> skip(long n)
```

#### Annotated Code Example

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
List<Integer> limited = numbers.stream().limit(3).collect(Collectors.toList());
System.out.println(limited);  // Output: [1, 2, 3]

List<Integer> skipped = numbers.stream().skip(7).collect(Collectors.toList());
System.out.println(skipped);  // Output: [8, 9, 10]
```

### 4.8 `takeWhile()` and `dropWhile()` (Java 9+)

#### Definitions

**`takeWhile()`:** Returns a stream consisting of the longest prefix of elements that match the given predicate.

**`dropWhile()`:** Returns a stream consisting of the remaining elements after dropping the longest prefix of elements that match the given predicate.

**Technical Definition:** "If this stream is ordered, returns a stream consisting of the longest prefix of elements taken from this stream that match the given predicate" (takeWhile); "returns a stream consisting of the remaining elements of this stream after dropping the longest prefix of elements that match the given predicate" (dropWhile).

**Beginner-Friendly Explanation:** `takeWhile()` takes elements until a condition fails; `dropWhile()` drops elements until a condition fails.

#### Syntax Rules and Structure

```
Stream<T> takeWhile(Predicate<? super T> predicate)
Stream<T> dropWhile(Predicate<? super T> predicate)
```

#### Annotated Code Example

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
List<Integer> taken = numbers.stream().takeWhile(n -> n < 5).collect(Collectors.toList());
System.out.println(taken);  // Output: [1, 2, 3, 4]

List<Integer> dropped = numbers.stream().dropWhile(n -> n < 5).collect(Collectors.toList());
System.out.println(dropped);  // Output: [5, 6, 7, 8, 9, 10]
```

### 4.9 `mapToInt()`, `mapToLong()`, `mapToDouble()`

#### Definitions

**Core Definition:** These methods transform a stream of objects into a primitive stream (`IntStream`, `LongStream`, or `DoubleStream`).

**Technical Definition:** "Returns an `IntStream` consisting of the results of applying the given function to the elements of this stream".

**Beginner-Friendly Explanation:** These methods convert objects into primitive numbers, which is more efficient and provides additional numeric operations like `sum()`, `average()`, etc.

#### Syntax Rules and Structure

```
IntStream mapToInt(ToIntFunction<? super T> mapper)
LongStream mapToLong(ToLongFunction<? super T> mapper)
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper)
```

#### Annotated Code Example

```java
List<String> names = List.of("Alice", "Bob", "Charlie");
int totalLength = names.stream()
    .mapToInt(String::length)
    .sum();
System.out.println("Total length: " + totalLength);  // Output: 15
```

### 4.10 `boxed()`

#### Definitions

**Core Definition:** `boxed()` converts a primitive stream (`IntStream`, `LongStream`, `DoubleStream`) back to a stream of boxed objects.

**Technical Definition:** "Returns a `Stream` consisting of the elements of this stream, each boxed to an `Integer`" (or `Long`, `Double`).

**Beginner-Friendly Explanation:** `boxed()` wraps primitive values into their object equivalents, allowing you to use object stream operations.

#### Syntax Rules and Structure

```
Stream<Integer> boxed()  // IntStream
Stream<Long> boxed()     // LongStream
Stream<Double> boxed()   // DoubleStream
```

#### Annotated Code Example

```java
List<Integer> boxedList = IntStream.range(1, 5)
    .boxed()
    .collect(Collectors.toList());
System.out.println(boxedList);  // Output: [1, 2, 3, 4]
```

### Real-World Cases

- **`filter`:** Filtering active users from a list.
- **`map`:** Extracting names from a list of Person objects.
- **`flatMap`:** Flattening a list of orders per customer into a single list of order items.
- **`distinct`:** Removing duplicate product IDs.
- **`sorted`:** Sorting employees by salary.
- **`limit`/`skip`:** Implementing pagination.
- **`mapToInt`:** Summing ages of a list of Person objects.
- **`boxed`:** Converting primitive stream results to a `List<Integer>`.

---

## 5. Terminal Operations

Terminal operations produce a result or side-effect and trigger the execution of the stream pipeline.

### 5.1 `forEach()` and `forEachOrdered()`

#### Definitions

**`forEach()`:** Performs an action for each element of the stream. The order is not guaranteed for parallel streams.

**`forEachOrdered()`:** Performs an action for each element, respecting encounter order even for parallel streams.

**Beginner-Friendly Explanation:** `forEach()` does something with each element; `forEachOrdered()` does the same but in order.

#### Syntax Rules and Structure

```
void forEach(Consumer<? super T> action)
void forEachOrdered(Consumer<? super T> action)
```

#### Annotated Code Example

```java
List<String> names = List.of("Alice", "Bob", "Charlie");
names.stream().forEach(System.out::println);
// Output: Alice, Bob, Charlie (order not guaranteed in parallel)
```

### 5.2 `toArray()`

#### Definitions

**Core Definition:** Returns an array containing the elements of the stream.

**Syntax:** `Object[] toArray()` or `A[] toArray(IntFunction<A[]> generator)`

#### Annotated Code Example

```java
String[] array = names.stream().toArray(String[]::new);
System.out.println(Arrays.toString(array));  // Output: [Alice, Bob, Charlie]
```

### 5.3 `reduce()`

#### Definitions

**Core Definition:** `reduce()` combines the elements of a stream into a single result using an associative accumulation function.

**Technical Definition:** "Performs a reduction on the elements of this stream, using an associative accumulation function, and returns an `Optional` describing the reduced value, if any".

**Beginner-Friendly Explanation:** `reduce()` folds all elements into one — like summing a list of numbers.

#### Syntax Rules and Structure

```
Optional<T> reduce(BinaryOperator<T> accumulator)
T reduce(T identity, BinaryOperator<T> accumulator)
<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner)
```

#### Annotated Code Example

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);
int sum = numbers.stream().reduce(0, Integer::sum);
System.out.println("Sum: " + sum);  // Output: 15
```

### 5.4 `collect()`

#### Definitions

**Core Definition:** `collect()` performs a mutable reduction operation on the elements of a stream, gathering them into a container such as a `List`, `Set`, or `Map`.

**Technical Definition:** "Performs a mutable reduction operation on the elements of this stream using a Collector".

**Beginner-Friendly Explanation:** `collect()` gathers the elements into a collection or summary.

#### Syntax Rules and Structure

```
<R, A> R collect(Collector<? super T, A, R> collector)
```

#### Annotated Code Example

```java
List<String> names = List.of("Alice", "Bob", "Charlie");
List<String> result = names.stream()
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());
System.out.println(result);  // Output: [Alice, Charlie]
```

### 5.5 `min()`, `max()`, `count()`

#### Definitions

**`min(Comparator)`:** Returns the minimum element according to the comparator.

**`max(Comparator)`:** Returns the maximum element according to the comparator.

**`count()`:** Returns the number of elements in the stream.

#### Annotated Code Example

```java
List<Integer> numbers = List.of(3, 1, 4, 1, 5, 9, 2, 6);
Optional<Integer> min = numbers.stream().min(Integer::compareTo);
System.out.println("Min: " + min.orElse(-1));  // Output: 1

long count = numbers.stream().count();
System.out.println("Count: " + count);  // Output: 8
```

### 5.6 `anyMatch()`, `allMatch()`, `noneMatch()`

#### Definitions

**`anyMatch(Predicate)`:** Returns `true` if any element matches the predicate.

**`allMatch(Predicate)`:** Returns `true` if all elements match the predicate.

**`noneMatch(Predicate)`:** Returns `true` if no elements match the predicate.

#### Annotated Code Example

```java
List<Integer> numbers = List.of(2, 4, 6, 8, 10);
System.out.println(numbers.stream().anyMatch(n -> n > 5));   // Output: true
System.out.println(numbers.stream().allMatch(n -> n % 2 == 0)); // Output: true
System.out.println(numbers.stream().noneMatch(n -> n < 0));  // Output: true
```

### 5.7 `findFirst()` and `findAny()`

#### Definitions

**`findFirst()`:** Returns an `Optional` describing the first element of the stream.

**`findAny()`:** Returns an `Optional` describing some element of the stream (useful in parallel streams).

#### Annotated Code Example

```java
List<String> names = List.of("Alice", "Bob", "Charlie");
Optional<String> first = names.stream().findFirst();
System.out.println(first.orElse("none"));  // Output: Alice
```

### Real-World Cases

- **`forEach`:** Printing results.
- **`toArray`:** Converting a stream to an array for legacy APIs.
- **`reduce`:** Summing, multiplying, or finding the longest string.
- **`collect`:** Gathering filtered elements into a list.
- **`min`/`max`:** Finding the youngest/oldest employee.
- **`count`:** Counting matching records.
- **`anyMatch`/`allMatch`/`noneMatch`:** Validating conditions.
- **`findFirst`/`findAny`:** Retrieving a single result.

---

## 6. Collectors

The `Collectors` utility class provides implementations of `Collector` for common reduction operations.

### 6.1 `toList()`, `toSet()`, `toMap()`, `toCollection()`

| Collector | Description |
|-----------|-------------|
| `toList()` | Collects elements into a `List`. |
| `toSet()` | Collects elements into a `Set`. |
| `toMap(keyMapper, valueMapper)` | Collects elements into a `Map`. |
| `toCollection(supplier)` | Collects into a custom collection. |

#### Annotated Code Example

```java
List<String> names = List.of("Alice", "Bob", "Charlie");

List<String> list = names.stream().collect(Collectors.toList());
Set<String> set = names.stream().collect(Collectors.toSet());
Map<Integer, String> map = names.stream()
    .collect(Collectors.toMap(String::length, s -> s, (a, b) -> a));
System.out.println(list);  // [Alice, Bob, Charlie]
System.out.println(set);   // [Alice, Bob, Charlie] (order may vary)
System.out.println(map);   // {3=Bob, 5=Alice, 7=Charlie}
```

### 6.2 `joining()`

#### Definitions

**Core Definition:** `joining()` concatenates the elements of a stream into a single `String`.

#### Syntax Rules and Structure

```
Collector<CharSequence, ?, String> joining()
Collector<CharSequence, ?, String> joining(CharSequence delimiter)
Collector<CharSequence, ?, String> joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix)
```

#### Annotated Code Example

```java
String joined = names.stream().collect(Collectors.joining(", "));
System.out.println(joined);  // Output: Alice, Bob, Charlie

String joinedWithBrackets = names.stream()
    .collect(Collectors.joining(", ", "[", "]"));
System.out.println(joinedWithBrackets);  // Output: [Alice, Bob, Charlie]
```

### 6.3 `groupingBy()` and `partitioningBy()`

#### Definitions

**`groupingBy()`:** Groups elements by a classification function, returning a `Map`.

**`partitioningBy()`:** Partitions elements into two groups based on a predicate, returning a `Map<Boolean, List<T>>`.

**Technical Definition:** "Returns a `Collector` implementing a 'group by' operation on input elements of type `T`, grouping elements according to a classification function, and returning the results in a `Map`".

**Beginner-Friendly Explanation:** `groupingBy` groups items by a property (e.g., department). `partitioningBy` splits items into two groups: those that pass a test and those that don't.

#### Syntax Rules and Structure

```
Collector<T, ?, Map<K, List<T>>> groupingBy(Function<? super T, ? extends K> classifier)
Collector<T, ?, Map<Boolean, List<T>>> partitioningBy(Predicate<? super T> predicate)
```

#### Annotated Code Example

```java
List<String> words = List.of("apple", "banana", "avocado", "blueberry", "cherry");

Map<Character, List<String>> grouped = words.stream()
    .collect(Collectors.groupingBy(w -> w.charAt(0)));
System.out.println(grouped);
// Output: {a=[apple, avocado], b=[banana, blueberry], c=[cherry]}

Map<Boolean, List<String>> partitioned = words.stream()
    .collect(Collectors.partitioningBy(w -> w.length() > 5));
System.out.println(partitioned);
// Output: {false=[apple, banana, cherry], true=[avocado, blueberry]}
```

### 6.4 `counting()`, `summingInt()`, `averagingInt()`

| Collector | Description |
|-----------|-------------|
| `counting()` | Counts the number of elements. |
| `summingInt(mapper)` | Sums `int` values extracted by a mapper. |
| `averagingInt(mapper)` | Computes the average of `int` values. |

#### Annotated Code Example

```java
List<String> names = List.of("Alice", "Bob", "Charlie", "David");

long count = names.stream().collect(Collectors.counting());
System.out.println("Count: " + count);  // Output: 4

int totalLength = names.stream()
    .collect(Collectors.summingInt(String::length));
System.out.println("Total length: " + totalLength);  // Output: 20
```

### 6.5 `reducing()`, `mapping()`, `filtering()`, `flatMapping()`

| Collector | Description |
|-----------|-------------|
| `reducing()` | Performs a reduction on elements. |
| `mapping(mapper, downstream)` | Applies a mapper before collecting. |
| `filtering(predicate, downstream)` | Filters before collecting. |
| `flatMapping(mapper, downstream)` | Flattens before collecting. |

### 6.6 `teeing()` (Java 12+)

#### Definitions

**Core Definition:** `teeing()` combines the results of two collectors into a single result.

**Technical Definition:** "Returns a `Collector` that is a composite of two downstream collectors. Every element passed to the resulting collector is processed by both downstream collectors, then their results are merged using the specified merge function into the final result".

#### Syntax Rules and Structure

```
static <T, R1, R2, R> Collector<T, ?, R> teeing(
    Collector<? super T, ?, R1> downstream1,
    Collector<? super T, ?, R2> downstream2,
    BiFunction<? super R1, ? super R2, R> merger
)
```

#### Annotated Code Example

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

String result = numbers.stream().collect(Collectors.teeing(
    Collectors.summingInt(Integer::intValue),
    Collectors.counting(),
    (sum, count) -> "Sum: " + sum + ", Count: " + count
));
System.out.println(result);  // Output: Sum: 15, Count: 5
```

### Real-World Cases

- **`toList`/`toSet`:** Gathering results.
- **`toMap`:** Creating lookup maps.
- **`joining`:** Building comma-separated strings for reports.
- **`groupingBy`:** Grouping employees by department.
- **`partitioningBy`:** Splitting students into passing/failing.
- **`counting`/`summingInt`/`averagingInt`:** Computing statistics.
- **`teeing`:** Computing multiple statistics in one pass.

---

## 7. Parallel Streams

### Definitions

**Core Definition:** A parallel stream is a stream that processes its elements concurrently using multiple threads.

**Technical Definition:** "Parallel streams enable us to execute code in parallel on separate cores. The final result is the combination of each individual outcome. However, the order of execution is out of our control". Parallel streams use the Fork-Join framework and its common pool of worker threads.

**Beginner-Friendly Explanation:** A parallel stream is like having multiple assembly lines working on the same task simultaneously. The work is divided among them, and the results are combined at the end.

### Purposes

- To leverage multiple CPU cores for faster processing
- To improve throughput for large data sets
- To simplify concurrent programming

### Syntax Rules and Structure

```
collection.parallelStream()      // Create a parallel stream
stream.parallel()                // Convert sequential to parallel
stream.sequential()              // Convert parallel to sequential
```

### Annotated Code Example

```java
// File: ParallelStreamDemo.java
import java.util.*;
import java.util.stream.*;

public class ParallelStreamDemo {
    public static void main(String[] args) {
        List<Integer> numbers = IntStream.rangeClosed(1, 10)
            .boxed()
            .collect(Collectors.toList());

        // Sequential stream
        long seqTime = System.nanoTime();
        int seqSum = numbers.stream().mapToInt(Integer::intValue).sum();
        seqTime = System.nanoTime() - seqTime;

        // Parallel stream
        long parTime = System.nanoTime();
        int parSum = numbers.parallelStream().mapToInt(Integer::intValue).sum();
        parTime = System.nanoTime() - parTime;

        System.out.println("Sequential sum: " + seqSum);  // Output: 55
        System.out.println("Parallel sum: " + parSum);    // Output: 55
        System.out.println("Sequential time: " + seqTime + " ns");
        System.out.println("Parallel time: " + parTime + " ns");
    }
}
```

**Expected Output (times vary):**
```
Sequential sum: 55
Parallel sum: 55
Sequential time: 1234567 ns
Parallel time: 987654 ns
```

### Real-World Cases

- **Large Data Processing:** Processing millions of records.
- **Image Processing:** Applying filters to pixels in parallel.
- **Mathematical Computations:** Monte Carlo simulations.
- **Web Crawling:** Fetching multiple URLs concurrently.

---

## 8. Stream Performance

### Key Performance Considerations

| Factor | Description |
|--------|-------------|
| Data Size | Parallel streams help only for large data sets; overhead outweighs benefits for small ones. |
| Operation Cost | Parallelism helps for CPU-intensive operations; I/O-bound operations block threads. |
| Splitting Cost | Sources that split poorly (e.g., `LinkedList`) reduce parallel efficiency. |
| Boxing Overhead | Primitive streams (`IntStream`, etc.) avoid boxing overhead. |
| Pipeline Order | Filter before map to reduce the number of elements transformed. |
| Short-Circuiting | `limit()`, `findFirst()`, `anyMatch()` stop early. |
| Stateful Operations | `sorted()`, `distinct()` require buffering and can be costly. |

### Performance Best Practices

- Prefer primitive streams (`IntStream`, `LongStream`, `DoubleStream`) to avoid boxing.
- Place `filter()` before `map()` to reduce the number of operations.
- Use short-circuiting operations (`limit`, `findFirst`) to avoid processing unnecessary elements.
- Avoid parallel streams for small data sets or I/O-bound operations.
- Use `parallelStream()` only when operations are independent and CPU-intensive.

### Annotated Code Example

```java
// Efficient pipeline: filter before map
List<String> result = list.stream()
    .filter(s -> s.length() > 3)      // Reduces stream size first
    .map(String::toUpperCase)         // Then transforms fewer elements
    .collect(Collectors.toList());
```

---

## 9. Stream Best Practices & Pitfalls

### Best Practices

| Practice | Description |
|----------|-------------|
| Prefer Streams for Bulk Operations | Use streams for filtering, mapping, and collecting. |
| Keep Pipelines Short | Complex pipelines are hard to read and debug. |
| Use Method References | `String::toUpperCase` instead of `s -> s.toUpperCase()`. |
| Filter Before Map | Reduces the number of elements transformed. |
| Use Primitive Streams | Avoid boxing with `mapToInt`, `mapToLong`, `mapToDouble`. |
| Avoid Side Effects | Do not modify the source during stream execution. |
| Use `collect()` for Mutable Reductions | Prefer `collect()` over manual accumulation. |
| Document Complex Pipelines | Add comments for non-obvious logic. |

### Common Pitfalls

| Pitfall | Description | Solution |
|---------|-------------|----------|
| Forgetting Terminal Operation | Intermediate operations do nothing without a terminal. | Always end with `collect()`, `forEach()`, etc. |
| Reusing a Stream | A stream can only be consumed once. | Create a new stream from the source each time. |
| Modifying Source During Stream | Causes `ConcurrentModificationException`. | Use a copy or a concurrent source. |
| Using `toMap()` Without Merge Function | Throws `IllegalStateException` on duplicate keys. | Provide a merge function: `(a, b) -> a`. |
| Parallel Stream with Stateful Lambdas | Causes incorrect results. | Ensure lambdas are stateless and non-interfering. |
| Infinite Stream Without `limit()` | Hangs or causes `OutOfMemoryError`. | Always bound infinite streams with `limit()`. |
| Overusing Parallel Streams | Slower than sequential for small data. | Measure performance before choosing. |

### Annotated Code Example

```java
// PITFALL: Reusing a stream
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
// stream.forEach(System.out::println);  // IllegalStateException

// SOLUTION: Create a new stream
list.stream().forEach(System.out::println);
list.stream().forEach(System.out::println);

// PITFALL: toMap with duplicate keys
// Map<String, Integer> map = words.stream()
//     .collect(Collectors.toMap(w -> w, String::length));  // Throws exception

// SOLUTION: Provide merge function
Map<String, Integer> map = words.stream()
    .collect(Collectors.toMap(w -> w, String::length, (a, b) -> a));
```

---

## References Links

- Interface Stream<T> — Java® Platform, Standard Edition & Java Development Kit Version 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html
- Class Collectors — Java® Platform, Standard Edition & Java Development Kit Version 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html
- Package java.util.stream — Java® Platform, Standard Edition & Java Development Kit Version 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html
- The Java Tutorials — Aggregate Operations - https://docs.oracle.com/javase/tutorial/collections/streams/index.html
- Answers to Questions and Exercises: Aggregate Operations - https://docs.oracle.com/javase/tutorial/collections/streams/QandE/answers.html
- JSR 335: Lambda Expressions for the Java™ Programming Language - https://cr.openjdk.org/~dlsmith/jsr335/jsr335-0.6.2/index.html
- When to Use a Parallel Stream in Java — Baeldung - https://www.baeldung.com/java-when-to-use-parallel-stream
- Java 8 Stream API Tutorial — Baeldung - https://www.baeldung.com/java-8-streams
- Effective Java (3rd Edition) by Joshua Bloch — Chapter 7: Lambdas and Streams
- Stream API: Best Practices and Pitfalls — GitHub - https://github.com/echo-cool-coding/cool-coding/blob/main/docs/language/java/17-java-stream-api/13-java-stream-best-practices.mdx