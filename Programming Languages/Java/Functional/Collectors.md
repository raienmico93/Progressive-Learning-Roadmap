# Java Collectors: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Java Collectors are a set of predefined implementations of the `Collector` interface that provide reusable reduction operations for transforming the elements of a stream into various result containers, such as collections, maps, strings, or statistical summaries.

**Technical Definition:** `java.util.stream.Collectors` is a final utility class that provides static factory methods for creating `Collector<T, A, R>` instances, where `T` is the type of input elements, `A` is the intermediate accumulation type, and `R` is the final result type. A `Collector` encapsulates a mutable reduction operation: it supplies a supplier function to create a mutable result container, an accumulator function to incorporate elements into the container, a combiner function for parallel processing, and a finisher function to perform an optional final transformation. Collectors are used as arguments to the terminal operation `Stream.collect(Collector)`.

**Beginner-Friendly Explanation:** Imagine you are at a factory where items come down a conveyor belt (the stream). You want to do something useful with all those items, like put them in boxes (lists), sort them into different bins (grouping), count them, add them up, or stick them together into one long label (joining). Collectors are like the instructions you give to the factory workers: "Put all these items into a list," "Group these items by their colour," "Calculate the total price," and so on. The `Collectors` class provides a menu of ready-made instructions so you do not have to write the low-level logic yourself.

### Key Characteristics

Collectors are terminal operations used with `Stream.collect()`. They perform mutable reductions, meaning they accumulate elements into a mutable result container rather than creating a new immutable result at each step. Collectors are designed to work seamlessly with both sequential and parallel streams, providing a `combiner` function for merging partial results. The `Collectors` class was introduced in Java 8, with additional collectors added in Java 9 (`filtering`, `flatMapping`) and Java 12 (`teeing`). Collectors are composable: many collectors accept a downstream collector, allowing for multi-level reductions such as grouping and then summarizing.

### Prerequisites

Readers should be familiar with Java generics, lambda expressions, method references, and the `java.util.function` package (particularly `Function`, `Predicate`, `Supplier`, `BiConsumer`, and `BinaryOperator`). A solid understanding of the `Stream` interface, intermediate operations (`filter`, `map`, `flatMap`), and terminal operations (`collect`, `reduce`) is assumed. Knowledge of the `Optional` class is necessary for `minBy` and `maxBy`.

### Related Programming Areas with Explanation

Java Collectors are related to **functional programming** (through the use of higher-order functions and immutable data processing), **data warehousing and analytics** (through grouping, partitioning, and summarization operations analogous to SQL `GROUP BY`, `SUM`, and `AVG`), **parallel computing** (through the `combiner` function that enables parallel reduction), and **report generation** (through string joining and statistical summaries). They are also closely related to the **Java Collections Framework**, as most collectors produce standard collection types.

## Core Concepts and Key Features

### 1. Basic Collection (Accumulating Elements)

#### 1.1 toList()

**Definitions**

- **Core Definition:** Collects elements into an unspecified `List` implementation (usually `ArrayList`).
- **Technical Definition:** `public static <T> Collector<T, ?, List<T>> toList()` returns a `Collector` that accumulates the input elements into a new `List`. There are no guarantees on the type, mutability, serializability, or thread-safety of the `List` returned; if more control over the returned `List` is required, use `toCollection(Supplier)` .
- **Beginner-Friendly Explanation:** `toList()` is the simplest collector. It takes all the items from the stream and puts them into a list, like putting groceries into a shopping bag.

**Purposes**

- To accumulate stream elements into a `List` for further processing or storage.
- To provide a simple, readable way to materialize a stream into a collection.
- To serve as a downstream collector in grouping and partitioning operations.

**Syntax Structures and Rules**

Complete general syntax:

```java
Collector<T, ?, List<T>> toList()
```

Component breakdown:

- `T`: the type of input elements.
- Return value: a `Collector` that produces a `List<T>`.

Syntax rules: The returned `List` is not guaranteed to be mutable, serializable, or thread-safe. The order of elements in the list follows the encounter order of the stream.

Constraints and limitations: The type of the returned `List` is unspecified and implementation-dependent. As of Java 16, the `Stream` interface provides a `toList()` method that returns an unmodifiable list .

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Collecting names into a list**

```java
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToListExample {
    public static void main(String[] args) {
        // Step 1: Create a stream of names
        Stream<String> names = Stream.of("Alice", "Bob", "Charlie");

        // Step 2: Collect into a List
        List<String> nameList = names.collect(Collectors.toList());

        System.out.println(nameList);
    }
}
```

Expected Output:

```
[Alice, Bob, Charlie]
```

**Why the code produces this result:** `Collectors.toList()` accumulates each stream element into an `ArrayList`, preserving encounter order.

**Example 2: Comparing Stream.toList() (Java 16+) with Collectors.toList()**

```java
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToListComparisonExample {
    public static void main(String[] args) {
        // Java 16+ Stream.toList() returns an unmodifiable list
        List<String> immutableList = Stream.of("a", "b", "c").toList();

        // Collectors.toList() returns a mutable list (typically ArrayList)
        List<String> mutableList = Stream.of("a", "b", "c")
                .collect(Collectors.toList());

        System.out.println("Immutable list: " + immutableList);
        System.out.println("Mutable list: " + mutableList);

        // Uncommenting the following line will throw UnsupportedOperationException
        // immutableList.add("d");

        mutableList.add("d");
        System.out.println("After adding to mutable: " + mutableList);
    }
}
```

Expected Output:

```
Immutable list: [a, b, c]
Mutable list: [a, b, c]
After adding to mutable: [a, b, c, d]
```

**Why the code produces this result:** `Stream.toList()` returns an unmodifiable list, while `Collectors.toList()` returns a mutable `ArrayList`. Attempting to modify the immutable list throws `UnsupportedOperationException` .

**Real-World Cases with Explanation**

In a data export pipeline, `toList()` is used to collect database records into a list for batch processing. In a web application, `toList()` is used to collect search results into a list for JSON serialization. In a testing framework, `toList()` is used to collect assertion results into a list for reporting.

**References Links**

- Collectors.toList (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#toList()
- Collectors.toList (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toList--
- Stream.toList (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#toList()

#### 1.2 toSet()

**Definitions**

- **Core Definition:** Collects elements into a `Set`, dropping duplicates.
- **Technical Definition:** `public static <T> Collector<T, ?, Set<T>> toSet()` returns a `Collector` that accumulates the input elements into a new `Set`. There are no guarantees on the type, mutability, serializability, or thread-safety of the `Set` returned; if more control over the returned `Set` is required, use `toCollection(Supplier)`.
- **Beginner-Friendly Explanation:** `toSet()` puts all items from the stream into a set, which automatically removes duplicates. If you have a list of words with repeats, `toSet()` gives you only the unique words.

**Purposes**

- To accumulate stream elements into a `Set`, eliminating duplicates.
- To obtain a collection of unique elements.
- To prepare data for operations that require uniqueness.

**Syntax Structures and Rules**

Complete general syntax:

```java
Collector<T, ?, Set<T>> toSet()
```

Component breakdown:

- `T`: the type of input elements.
- Return value: a `Collector` that produces a `Set<T>`.

Syntax rules: Elements are compared using `equals()` and `hashCode()`. The encounter order of the resulting `Set` is unspecified.

Constraints and limitations: The type of the returned `Set` is unspecified (typically `HashSet`). The order is not preserved.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Removing duplicates with toSet**

```java
import java.util.Set;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToSetExample {
    public static void main(String[] args) {
        Set<String> uniqueWords = Stream.of("apple", "banana", "apple", "cherry", "banana")
                .collect(Collectors.toSet());

        System.out.println(uniqueWords);
    }
}
```

Expected Output (order may vary):

```
[banana, cherry, apple]
```

**Why the code produces this result:** `toSet()` removes duplicate strings using `equals()`, keeping only unique values. The order is non-deterministic because `HashSet` does not preserve insertion order.

**Example 2: Collecting custom objects into a Set**

```java
import java.util.Set;
import java.util.stream.Collectors;

class Product {
    String name;
    Product(String name) { this.name = name; }
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Product product = (Product) o;
        return name.equals(product.name);
    }
    @Override
    public int hashCode() { return name.hashCode(); }
    @Override
    public String toString() { return name; }
}

public class ToSetCustomExample {
    public static void main(String[] args) {
        Set<Product> products = java.util.List.of(
                new Product("Laptop"), new Product("Phone"), new Product("Laptop")
        ).stream().collect(Collectors.toSet());

        System.out.println(products);
    }
}
```

Expected Output (order may vary):

```
[Phone, Laptop]
```

**Why the code produces this result:** The `Product` class overrides `equals()` and `hashCode()` based on the product name. The duplicate “Laptop” is removed.

**Real-World Cases with Explanation**

In a tag management system, `toSet()` is used to collect unique tags from a list of articles. In a user analytics system, `toSet()` is used to collect unique visitor IDs. In a configuration system, `toSet()` is used to collect unique configuration keys.

**References Links**

- Collectors.toSet (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#toSet()
- Collectors.toSet (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toSet--

#### 1.3 toMap(keyMapper, valueMapper)

**Definitions**

- **Core Definition:** Collects elements into a `Map`. Throws an exception if duplicate keys are detected unless a merge function is provided.
- **Technical Definition:** `public static <T, K, U> Collector<T, ?, Map<K, U>> toMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends U> valueMapper)` returns a `Collector` that accumulates elements into a `Map` whose keys and values are the result of applying the provided mapping functions to the input elements. If the mapped keys contain duplicates (according to `Object.equals(Object)`), an `IllegalStateException` is thrown when the collection operation is performed. If the mapped keys might have duplicates, use the `toMap(keyMapper, valueMapper, mergeFunction)` overload instead .
- **Beginner-Friendly Explanation:** `toMap` takes each item from the stream and creates a key-value pair from it. If two items produce the same key, it throws an error unless you provide a rule for how to merge them.

**Purposes**

- To transform a stream of objects into a `Map` for fast lookups.
- To index elements by a unique attribute (e.g., ID, email).
- To create a mapping from one representation to another.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
Collector<T, ?, Map<K, U>> toMap(Function<? super T, ? extends K> keyMapper,
                                 Function<? super T, ? extends U> valueMapper)
Collector<T, ?, Map<K, U>> toMap(Function<? super T, ? extends K> keyMapper,
                                 Function<? super T, ? extends U> valueMapper,
                                 BinaryOperator<U> mergeFunction)
```

Component breakdown:

- `keyMapper`: a function that extracts the key from each element.
- `valueMapper`: a function that extracts the value from each element.
- `mergeFunction`: a binary operator that resolves collisions between values associated with the same key.
- Return value: a `Collector` that produces a `Map<K, U>`.

Syntax rules: The key mapper must not produce `null` keys. Duplicate keys cause `IllegalStateException` in the two-argument version. The merge function is called with the existing value and the new value.

Constraints and limitations: The two-argument version does not handle duplicate keys. The `Map` implementation is unspecified (typically `HashMap`).

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic toMap (unique keys)**

```java
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToMapExample {
    public static void main(String[] args) {
        Map<String, Integer> wordLengths = Stream.of("apple", "banana", "cherry")
                .collect(Collectors.toMap(
                        word -> word,           // key: the word itself
                        word -> word.length()   // value: the length
                ));

        System.out.println(wordLengths);
    }
}
```

Expected Output (order may vary):

```
{apple=5, banana=6, cherry=6}
```

**Why the code produces this result:** Each word becomes a key, and its length becomes the value. Since all keys are unique, no exception is thrown.

**Example 2: toMap with merge function (duplicate keys)**

```java
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToMapMergeExample {
    public static void main(String[] args) {
        Map<String, Integer> wordCounts = Stream.of("apple", "banana", "apple", "cherry", "banana")
                .collect(Collectors.toMap(
                        word -> word,
                        word -> 1,
                        Integer::sum   // merge function: sum the counts
                ));

        System.out.println(wordCounts);
    }
}
```

Expected Output (order may vary):

```
{apple=2, banana=2, cherry=1}
```

**Why the code produces this result:** The merge function `Integer::sum` combines the values for duplicate keys. “apple” appears twice, so its count becomes 2.

**Example 3: toMap with duplicate keys (throws exception)**

```java
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToMapDuplicateExample {
    public static void main(String[] args) {
        try {
            Map<String, Integer> map = Stream.of("apple", "banana", "apple")
                    .collect(Collectors.toMap(
                            word -> word,
                            word -> word.length()
                    ));
        } catch (IllegalStateException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Caught: IllegalStateException
```

**Why the code produces this result:** The key “apple” appears twice. The two-argument `toMap` does not know how to merge the values, so it throws `IllegalStateException`.

**Real-World Cases with Explanation**

In a user management system, `toMap` is used to create a map from user ID to user object. In a configuration system, `toMap` is used to map configuration keys to their values. In a caching system, `toMap` is used to build an in-memory cache from a stream of cache entries.

**References Links**

- Collectors.toMap (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#toMap(java.util.function.Function,java.util.function.Function)
- Collectors.toMap with merge (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#toMap(java.util.function.Function,java.util.function.Function,java.util.function.BinaryOperator)

#### 1.4 toUnmodifiableList() / toUnmodifiableSet() / toUnmodifiableMap() (Added in Java 10)

**Definitions**

- **Core Definition:** Ensures the generated collections are strictly immutable.
- **Technical Definition:** `public static <T> Collector<T, ?, List<T>> toUnmodifiableList()`, `public static <T> Collector<T, ?, Set<T>> toUnmodifiableSet()`, and `public static <T, K, U> Collector<T, ?, Map<K, U>> toUnmodifiableMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends U> valueMapper)` return collectors that accumulate elements into unmodifiable collections. Any attempt to modify the returned collection results in an `UnsupportedOperationException` .
- **Beginner-Friendly Explanation:** These collectors work like `toList`, `toSet`, and `toMap`, but the resulting collections cannot be changed. They are “read-only” versions.

**Purposes**

- To guarantee that the resulting collection cannot be modified after creation.
- To enforce immutability in functional programming pipelines.
- To safely share collections between threads without synchronization.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
Collector<T, ?, List<T>> toUnmodifiableList()
Collector<T, ?, Set<T>> toUnmodifiableSet()
Collector<T, ?, Map<K, U>> toUnmodifiableMap(Function<? super T, ? extends K> keyMapper,
                                              Function<? super T, ? extends U> valueMapper)
Collector<T, ?, Map<K, U>> toUnmodifiableMap(Function<? super T, ? extends K> keyMapper,
                                              Function<? super T, ? extends U> valueMapper,
                                              BinaryOperator<U> mergeFunction)
```

Component breakdown:

- Parameters are the same as for `toMap`.
- Return value: a `Collector` that produces an unmodifiable collection.

Syntax rules: The returned collections throw `UnsupportedOperationException` on any modification attempt. For `toUnmodifiableMap`, duplicate keys cause `IllegalStateException` unless a merge function is provided . `null` elements are not permitted.

Constraints and limitations: The unmodifiable collections do not permit `null` elements. Iteration order for sets and maps is randomized.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Unmodifiable list**

```java
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToUnmodifiableListExample {
    public static void main(String[] args) {
        List<String> list = Stream.of("a", "b", "c")
                .collect(Collectors.toUnmodifiableList());

        System.out.println(list);

        try {
            list.add("d");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
[a, b, c]
Cannot modify: UnsupportedOperationException
```

**Example 2: Unmodifiable set**

```java
import java.util.Set;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToUnmodifiableSetExample {
    public static void main(String[] args) {
        Set<String> set = Stream.of("a", "b", "a", "c")
                .collect(Collectors.toUnmodifiableSet());

        System.out.println(set);
    }
}
```

Expected Output (order may vary):

```
[b, c, a]
```

**Example 3: Unmodifiable map with merge function**

```java
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToUnmodifiableMapExample {
    public static void main(String[] args) {
        Map<String, Integer> map = Stream.of("apple", "banana", "apple")
                .collect(Collectors.toUnmodifiableMap(
                        word -> word,
                        word -> 1,
                        Integer::sum
                ));

        System.out.println(map);
    }
}
```

Expected Output (order may vary):

```
{apple=2, banana=1}
```

**Real-World Cases with Explanation**

In a microservices architecture, `toUnmodifiableList` is used to create immutable configuration lists shared across threads. In a security-sensitive application, `toUnmodifiableMap` is used to create immutable permission mappings.

**References Links**

- Collectors.toUnmodifiableList (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#toUnmodifiableList()
- Collectors.toUnmodifiableSet (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#toUnmodifiableSet()
- Collectors.toUnmodifiableMap (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#toUnmodifiableMap(java.util.function.Function,java.util.function.Function)

#### 1.5 toCollection(Supplier)

**Definitions**

- **Core Definition:** Allows you to choose a custom collection instance (e.g., `toCollection(TreeSet::new)`).
- **Technical Definition:** `public static <T, C extends Collection<T>> Collector<T, ?, C> toCollection(Supplier<C> collectionFactory)` returns a `Collector` that accumulates the input elements into a new `Collection`, in encounter order. The `Collection` is created by the provided factory .
- **Beginner-Friendly Explanation:** `toCollection` is like `toList` or `toSet`, but you get to decide exactly what kind of collection you want—a `TreeSet`, a `LinkedList`, a `PriorityQueue`, or any other `Collection` implementation.

**Purposes**

- To collect elements into a specific `Collection` implementation.
- To control the ordering, performance characteristics, or behaviour of the resulting collection.
- To use a custom collection class that is not covered by `toList` or `toSet`.

**Syntax Structures and Rules**

Complete general syntax:

```java
Collector<T, ?, C> toCollection(Supplier<C> collectionFactory)
```

Component breakdown:

- `collectionFactory`: a `Supplier` that returns a new, empty `Collection` of the appropriate type.
- `C`: the type of the resulting `Collection`.
- Return value: a `Collector` that produces a `C`.

Syntax rules: The supplier must return a new, mutable collection. The collector adds elements to this collection using `add()`.

Constraints and limitations: The collection must not be `null`. The collection must support `add()`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Collecting into a TreeSet**

```java
import java.util.Set;
import java.util.TreeSet;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToCollectionTreeSetExample {
    public static void main(String[] args) {
        TreeSet<String> sortedSet = Stream.of("banana", "apple", "cherry")
                .collect(Collectors.toCollection(TreeSet::new));

        System.out.println(sortedSet);
    }
}
```

Expected Output:

```
[apple, banana, cherry]
```

**Why the code produces this result:** `TreeSet` maintains elements in sorted order. The collector adds each element to the `TreeSet`, which automatically sorts them.

**Example 2: Collecting into a LinkedList**

```java
import java.util.LinkedList;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToCollectionLinkedListExample {
    public static void main(String[] args) {
        LinkedList<Integer> linkedList = Stream.of(3, 1, 4, 1, 5)
                .collect(Collectors.toCollection(LinkedList::new));

        System.out.println(linkedList);
    }
}
```

Expected Output:

```
[3, 1, 4, 1, 5]
```

**Real-World Cases with Explanation**

In a priority-based scheduling system, `toCollection(PriorityQueue::new)` is used to collect tasks into a priority queue. In a system requiring sorted unique elements, `toCollection(TreeSet::new)` is used.

**References Links**

- Collectors.toCollection (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#toCollection(java.util.function.Supplier)

#### 1.6 joining()

**Definitions**

- **Core Definition:** Concatenates `CharSequence`/`String` elements into a single string. Supports optional delimiters, prefixes, and suffixes.
- **Technical Definition:** `public static Collector<CharSequence, ?, String> joining()`, `joining(CharSequence delimiter)`, and `joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix)` return a `Collector` that concatenates the input elements into a `String`, in encounter order .
- **Beginner-Friendly Explanation:** `joining()` takes all the strings in the stream and glues them together into one string. You can optionally put a separator (like a comma) between them, and even add a prefix and suffix.

**Purposes**

- To concatenate a stream of strings into a single string.
- To generate CSV lines, JSON arrays, or formatted reports.
- To build human-readable output from a collection of values.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
Collector<CharSequence, ?, String> joining()
Collector<CharSequence, ?, String> joining(CharSequence delimiter)
Collector<CharSequence, ?, String> joining(CharSequence delimiter,
                                           CharSequence prefix,
                                           CharSequence suffix)
```

Component breakdown:

- `delimiter`: the sequence of characters placed between each element.
- `prefix`: the sequence of characters placed at the beginning of the result.
- `suffix`: the sequence of characters placed at the end of the result.
- Return value: a `Collector` that produces a `String`.

Syntax rules: The elements must be `CharSequence` (e.g., `String`). If the stream is empty, `joining()` returns an empty string, `joining(delimiter)` returns an empty string, and `joining(delimiter, prefix, suffix)` returns `prefix + suffix`.

Constraints and limitations: Only works with `CharSequence` streams. To join non-string elements, use `map(Object::toString)` first.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Simple joining**

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class JoiningExample {
    public static void main(String[] args) {
        String result = Stream.of("Java", "Stream", "API")
                .collect(Collectors.joining());
        System.out.println(result);
    }
}
```

Expected Output:

```
JavaStreamAPI
```

**Example 2: Joining with delimiter**

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class JoiningDelimiterExample {
    public static void main(String[] args) {
        String result = Stream.of("apple", "banana", "cherry")
                .collect(Collectors.joining(", "));
        System.out.println(result);
    }
}
```

Expected Output:

```
apple, banana, cherry
```

**Example 3: Joining with delimiter, prefix, and suffix**

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class JoiningFullExample {
    public static void main(String[] args) {
        String result = Stream.of("apple", "banana", "cherry")
                .collect(Collectors.joining(", ", "[", "]"));
        System.out.println(result);
    }
}
```

Expected Output:

```
[apple, banana, cherry]
```

**Real-World Cases with Explanation**

In a report generation system, `joining` is used to create a comma-separated list of names. In a web application, `joining` is used to build a JSON array string from a list of values. In a logging system, `joining` is used to concatenate log messages into a single line.

**References Links**

- Collectors.joining (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#joining()
- Collectors.joining with delimiter (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#joining(java.lang.CharSequence)

### 2. Grouping & Partitioning (Classification)

#### 2.1 groupingBy(classifier)

**Definitions**

- **Core Definition:** Groups elements into a `Map<K, List<T>>` using a key function.
- **Technical Definition:** `public static <T, K> Collector<T, ?, Map<K, List<T>>> groupingBy(Function<? super T, ? extends K> classifier)` returns a `Collector` implementing a “group by” operation on input elements of type `T`, grouping elements according to a classification function, and returning the results in a `Map` . The downstream collector defaults to `toList()`.
- **Beginner-Friendly Explanation:** `groupingBy` is like sorting a pile of objects into different boxes based on a label. You provide a function that extracts the label (key) from each object, and `groupingBy` puts all objects with the same label into the same box (list).

**Purposes**

- To classify elements into groups based on a common attribute.
- To perform multi-level reductions (e.g., group by department, then count employees).
- To create a map of lists from a flat stream of elements.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
Collector<T, ?, Map<K, List<T>>> groupingBy(Function<? super T, ? extends K> classifier)
Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier,
                                       Collector<? super T, A, D> downstream)
Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier,
                               Supplier<M> mapFactory,
                               Collector<? super T, A, D> downstream)
```

Component breakdown:

- `classifier`: a function that maps each element to a key.
- `downstream`: a collector that processes the elements in each group.
- `mapFactory`: a supplier that creates the `Map` implementation.
- Return value: a `Collector` that produces a `Map<K, D>`.

Syntax rules: The classifier must not return `null` keys. The downstream collector operates on elements of type `T` and produces a result of type `D`.

Constraints and limitations: The default `Map` implementation is `HashMap`. For ordered maps, use the three-argument version with `TreeMap::new`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic groupingBy**

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

class Person {
    String name;
    String city;
    Person(String name, String city) { this.name = name; this.city = city; }
    String getCity() { return city; }
    String getName() { return name; }
    @Override
    public String toString() { return name; }
}

public class GroupingByExample {
    public static void main(String[] args) {
        List<Person> people = List.of(
                new Person("Alice", "NYC"),
                new Person("Bob", "LA"),
                new Person("Charlie", "NYC"),
                new Person("Dave", "LA")
        );

        Map<String, List<Person>> byCity = people.stream()
                .collect(Collectors.groupingBy(Person::getCity));

        System.out.println(byCity);
    }
}
```

Expected Output (order may vary):

```
{NYC=[Alice, Charlie], LA=[Bob, Dave]}
```

**Example 2: groupingBy with downstream collector (counting)**

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class GroupingByCountingExample {
    public static void main(String[] args) {
        List<Person> people = List.of(
                new Person("Alice", "NYC"),
                new Person("Bob", "LA"),
                new Person("Charlie", "NYC"),
                new Person("Dave", "LA")
        );

        Map<String, Long> countByCity = people.stream()
                .collect(Collectors.groupingBy(Person::getCity, Collectors.counting()));

        System.out.println(countByCity);
    }
}
```

Expected Output (order may vary):

```
{NYC=2, LA=2}
```

**Example 3: groupingBy with custom map factory (TreeMap)**

```java
import java.util.List;
import java.util.Map;
import java.util.TreeMap;
import java.util.stream.Collectors;

public class GroupingByTreeMapExample {
    public static void main(String[] args) {
        List<Person> people = List.of(
                new Person("Alice", "NYC"),
                new Person("Bob", "LA"),
                new Person("Charlie", "NYC")
        );

        TreeMap<String, List<Person>> byCity = people.stream()
                .collect(Collectors.groupingBy(
                        Person::getCity,
                        TreeMap::new,
                        Collectors.toList()
                ));

        System.out.println(byCity);
    }
}
```

Expected Output:

```
{LA=[Bob], NYC=[Alice, Charlie]}
```

**Real-World Cases with Explanation**

In an e-commerce system, `groupingBy` is used to group orders by customer. In a human resources system, `groupingBy` is used to group employees by department. In a log analysis system, `groupingBy` is used to group log entries by severity level.

**References Links**

- Collectors.groupingBy (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#groupingBy(java.util.function.Function)
- Collectors.groupingBy with downstream (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#groupingBy(java.util.function.Function,java.util.stream.Collector)

#### 2.2 partitioningBy(predicate)

**Definitions**

- **Core Definition:** Special case of grouping that splits data into a `Map<Boolean, List<T>>` based on a true/false condition.
- **Technical Definition:** `public static <T> Collector<T, ?, Map<Boolean, List<T>>> partitioningBy(Predicate<? super T> predicate)` returns a `Collector` which partitions the input elements according to a `Predicate`, and organizes them into a `Map<Boolean, List<T>>`. The returned `Map` always contains mappings for both `false` and `true` keys .
- **Beginner-Friendly Explanation:** `partitioningBy` splits a stream into two groups: those that match a condition (true) and those that do not (false). It is like a fork in the road where every item goes one way or the other.

**Purposes**

- To split elements into two categories based on a boolean condition.
- To perform separate processing on matching and non-matching elements.
- To implement binary classification in a stream pipeline.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
Collector<T, ?, Map<Boolean, List<T>>> partitioningBy(Predicate<? super T> predicate)
Collector<T, ?, Map<Boolean, D>> partitioningBy(Predicate<? super T> predicate,
                                                 Collector<? super T, A, D> downstream)
```

Component breakdown:

- `predicate`: the condition used for partitioning.
- `downstream`: a collector that processes the elements in each partition.
- Return value: a `Collector` that produces a `Map<Boolean, D>`.

Syntax rules: The `Map` always contains both `true` and `false` keys, even if one partition is empty. The downstream collector operates on elements of type `T`.

Constraints and limitations: The `Map` implementation is not specified. The result is always a two-entry map.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic partitioningBy**

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class PartitioningByExample {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        Map<Boolean, List<Integer>> evenOdd = numbers.stream()
                .collect(Collectors.partitioningBy(n -> n % 2 == 0));

        System.out.println(evenOdd);
    }
}
```

Expected Output:

```
{false=[1, 3, 5, 7, 9], true=[2, 4, 6, 8, 10]}
```

**Example 2: partitioningBy with downstream collector (counting)**

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class PartitioningByCountingExample {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        Map<Boolean, Long> countEvenOdd = numbers.stream()
                .collect(Collectors.partitioningBy(
                        n -> n % 2 == 0,
                        Collectors.counting()
                ));

        System.out.println(countEvenOdd);
    }
}
```

Expected Output:

```
{false=5, true=5}
```

**Real-World Cases with Explanation**

In a student grading system, `partitioningBy` is used to split students into passing and failing groups. In a quality control system, `partitioningBy` is used to separate defective products from good ones.

**References Links**

- Collectors.partitioningBy (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#partitioningBy(java.util.function.Predicate)

### 3. Numerical Summarization & Aggregation

#### 3.1 summarizingInt() / summarizingLong() / summarizingDouble()

**Definitions**

- **Core Definition:** Computes a single state object (`IntSummaryStatistics`, `LongSummaryStatistics`, or `DoubleSummaryStatistics`) containing count, min, max, sum, and average in a single pass.
- **Technical Definition:** `public static <T> Collector<T, ?, IntSummaryStatistics> summarizingInt(ToIntFunction<? super T> mapper)` returns a `Collector` that applies an int-producing mapping function to each input element, and returns summary statistics for the resulting values. The `IntSummaryStatistics` class provides methods such as `getCount()`, `getSum()`, `getMin()`, `getMax()`, and `getAverage()` .
- **Beginner-Friendly Explanation:** `summarizingInt` is like a Swiss Army knife for numbers. In one go, it tells you how many numbers there are, what the smallest is, what the largest is, what they add up to, and what the average is.

**Purposes**

- To compute multiple statistics (count, min, max, sum, average) in a single pass over the stream.
- To avoid multiple iterations over the same data.
- To generate summary reports for numerical data.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
Collector<T, ?, IntSummaryStatistics> summarizingInt(ToIntFunction<? super T> mapper)
Collector<T, ?, LongSummaryStatistics> summarizingLong(ToLongFunction<? super T> mapper)
Collector<T, ?, DoubleSummaryStatistics> summarizingDouble(ToDoubleFunction<? super T> mapper)
```

Component breakdown:

- `mapper`: a function that extracts an int, long, or double value from each element.
- Return value: a `Collector` that produces the corresponding `SummaryStatistics` object.

Syntax rules: The summary statistics object is mutable during accumulation but is finished as an immutable result.

Constraints and limitations: The average is computed as a double for `summarizingInt` and `summarizingLong`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: summarizingInt**

```java
import java.util.IntSummaryStatistics;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class SummarizingIntExample {
    public static void main(String[] args) {
        IntSummaryStatistics stats = Stream.of(3, 1, 4, 1, 5, 9, 2, 6)
                .collect(Collectors.summarizingInt(Integer::intValue));

        System.out.println("Count: " + stats.getCount());
        System.out.println("Min: " + stats.getMin());
        System.out.println("Max: " + stats.getMax());
        System.out.println("Sum: " + stats.getSum());
        System.out.println("Average: " + stats.getAverage());
    }
}
```

Expected Output:

```
Count: 8
Min: 1
Max: 9
Sum: 31
Average: 3.875
```

**Example 2: summarizingDouble**

```java
import java.util.DoubleSummaryStatistics;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class SummarizingDoubleExample {
    public static void main(String[] args) {
        DoubleSummaryStatistics stats = Stream.of(1.5, 2.7, 3.2, 0.8)
                .collect(Collectors.summarizingDouble(Double::doubleValue));

        System.out.println(stats);
    }
}
```

Expected Output:

```
DoubleSummaryStatistics{count=4, sum=8.200000, min=0.800000, average=2.050000, max=3.200000}
```

**Real-World Cases with Explanation**

In a financial application, `summarizingDouble` is used to compute statistics on stock prices. In an IoT system, `summarizingInt` is used to summarize sensor readings.

**References Links**

- Collectors.summarizingInt (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#summarizingInt(java.util.function.ToIntFunction)
- IntSummaryStatistics (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/IntSummaryStatistics.html

#### 3.2 summingInt() / summingLong() / summingDouble()

**Definitions**

- **Core Definition:** Calculates only the total sum.
- **Technical Definition:** `public static <T> Collector<T, ?, Integer> summingInt(ToIntFunction<? super T> mapper)` returns a `Collector` that produces the sum of an int-valued function applied to the input elements. If no elements are present, the result is 0.
- **Beginner-Friendly Explanation:** `summingInt` adds up all the numbers in the stream. It is a shortcut for `reduce(0, Integer::sum)`.

**Purposes**

- To compute the total of a numerical attribute across all elements.
- To perform aggregation without creating a full statistics object.
- To serve as a downstream collector in grouping operations.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
Collector<T, ?, Integer> summingInt(ToIntFunction<? super T> mapper)
Collector<T, ?, Long> summingLong(ToLongFunction<? super T> mapper)
Collector<T, ?, Double> summingDouble(ToDoubleFunction<? super T> mapper)
```

Component breakdown:

- `mapper`: a function that extracts a numerical value from each element.
- Return value: a `Collector` that produces the sum.

Syntax rules: If the stream is empty, the result is 0 (for all three variants).

Constraints and limitations: `summingDouble` may have floating-point rounding issues.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: summingInt**

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class SummingIntExample {
    public static void main(String[] args) {
        int total = Stream.of("apple", "banana", "cherry")
                .collect(Collectors.summingInt(String::length));

        System.out.println("Total length: " + total);
    }
}
```

Expected Output:

```
Total length: 17
```

**Example 2: summingDouble**

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class SummingDoubleExample {
    public static void main(String[] args) {
        double total = Stream.of(1.5, 2.5, 3.5)
                .collect(Collectors.summingDouble(Double::doubleValue));

        System.out.println("Total: " + total);
    }
}
```

Expected Output:

```
Total: 7.5
```

**Real-World Cases with Explanation**

In a shopping cart application, `summingDouble` is used to calculate the total price of items. In a payroll system, `summingInt` is used to sum the total number of hours worked by all employees.

**References Links**

- Collectors.summingInt (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#summingInt(java.util.function.ToIntFunction)
- Collectors.summingDouble (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#summingDouble(java.util.function.ToDoubleFunction)

#### 3.3 averagingInt() / averagingLong() / averagingDouble()

**Definitions**

- **Core Definition:** Calculates only the average value.
- **Technical Definition:** `public static <T> Collector<T, ?, Double> averagingInt(ToIntFunction<? super T> mapper)` returns a `Collector` that produces the arithmetic mean of an int-valued function applied to the input elements. If no elements are present, the result is 0.
- **Beginner-Friendly Explanation:** `averagingInt` calculates the average (mean) of all the numbers in the stream.

**Purposes**

- To compute the arithmetic mean of a numerical attribute.
- To perform statistical analysis on stream data.
- To serve as a downstream collector in grouping operations.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
Collector<T, ?, Double> averagingInt(ToIntFunction<? super T> mapper)
Collector<T, ?, Double> averagingLong(ToLongFunction<? super T> mapper)
Collector<T, ?, Double> averagingDouble(ToDoubleFunction<? super T> mapper)
```

Component breakdown:

- `mapper`: a function that extracts a numerical value from each element.
- Return value: a `Collector` that produces the average as a `Double`.

Syntax rules: The average is always returned as a `Double`, even for `averagingInt` and `averagingLong`. If the stream is empty, the result is 0.

Constraints and limitations: Division by zero is not an issue because the empty case returns 0.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: averagingInt**

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class AveragingIntExample {
    public static void main(String[] args) {
        double average = Stream.of(1, 2, 3, 4, 5)
                .collect(Collectors.averagingInt(Integer::intValue));

        System.out.println("Average: " + average);
    }
}
```

Expected Output:

```
Average: 3.0
```

**Example 2: averagingDouble**

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class AveragingDoubleExample {
    public static void main(String[] args) {
        double average = Stream.of(1.5, 2.5, 3.5)
                .collect(Collectors.averagingDouble(Double::doubleValue));

        System.out.println("Average: " + average);
    }
}
```

Expected Output:

```
Average: 2.5
```

**Real-World Cases with Explanation**

In a student grading system, `averagingInt` is used to calculate the average score of a class. In a financial application, `averagingDouble` is used to compute the average transaction amount.

**References Links**

- Collectors.averagingInt (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#averagingInt(java.util.function.ToIntFunction)
- Collectors.averagingDouble (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#averagingDouble(java.util.function.ToDoubleFunction)

#### 3.4 counting()

**Definitions**

- **Core Definition:** Returns the total element count as a `Long`.
- **Technical Definition:** `public static <T> Collector<T, ?, Long> counting()` returns a `Collector` accepting elements of type `T` that counts the number of input elements. If no elements are present, the result is 0 .
- **Beginner-Friendly Explanation:** `counting` simply counts how many items are in the stream. It is similar to `Stream.count()` but can be used as a downstream collector.

**Purposes**

- To count the number of elements in a stream or group.
- To serve as a downstream collector for grouping and partitioning.
- To perform conditional counting when combined with `filtering`.

**Syntax Structures and Rules**

Complete general syntax:

```java
Collector<T, ?, Long> counting()
```

Component breakdown:

- Return value: a `Collector` that produces a `Long` count.

Syntax rules: The result is always a `Long`. For an empty stream, the result is 0.

Constraints and limitations: None.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic counting**

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class CountingExample {
    public static void main(String[] args) {
        Long count = Stream.of("a", "b", "c", "d", "e")
                .collect(Collectors.counting());

        System.out.println("Count: " + count);
    }
}
```

Expected Output:

```
Count: 5
```

**Example 2: Counting as a downstream collector**

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class CountingDownstreamExample {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "apple", "cherry", "banana", "apple");

        Map<String, Long> wordCounts = words.stream()
                .collect(Collectors.groupingBy(
                        word -> word,
                        Collectors.counting()
                ));

        System.out.println(wordCounts);
    }
}
```

Expected Output (order may vary):

```
{apple=3, banana=2, cherry=1}
```

**Real-World Cases with Explanation**

In a voting system, `counting` is used to count the number of votes for each candidate. In a log analysis system, `counting` is used to count the number of occurrences of each error type.

**References Links**

- Collectors.counting (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#counting()

#### 3.5 minBy(Comparator) / maxBy(Comparator)

**Definitions**

- **Core Definition:** Finds the minimum or maximum element wrapped in an `Optional`.
- **Technical Definition:** `public static <T> Collector<T, ?, Optional<T>> minBy(Comparator<? super T> comparator)` returns a `Collector` that produces the minimum element according to the given `Comparator`. `maxBy(Comparator)` returns the maximum. Both return an `Optional` because the stream may be empty.
- **Beginner-Friendly Explanation:** `minBy` finds the smallest item in the stream based on a comparison rule, and `maxBy` finds the largest. They return the result in an `Optional` in case the stream is empty.

**Purposes**

- To find the minimum or maximum element according to a custom ordering.
- To serve as a downstream collector for grouping operations (e.g., find the oldest person in each city).
- To perform comparisons without sorting the entire stream.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
Collector<T, ?, Optional<T>> minBy(Comparator<? super T> comparator)
Collector<T, ?, Optional<T>> maxBy(Comparator<? super T> comparator)
```

Component breakdown:

- `comparator`: a non-interfering, stateless `Comparator` to compare elements.
- Return value: a `Collector` that produces an `Optional<T>`.

Syntax rules: The result is an `Optional` that may be empty if the stream is empty.

Constraints and limitations: The comparator must be consistent with `equals` for correct results.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: minBy and maxBy on integers**

```java
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class MinMaxExample {
    public static void main(String[] args) {
        Optional<Integer> min = Stream.of(5, 2, 8, 1, 9)
                .collect(Collectors.minBy(Integer::compareTo));
        Optional<Integer> max = Stream.of(5, 2, 8, 1, 9)
                .collect(Collectors.maxBy(Integer::compareTo));

        System.out.println("Min: " + min.orElse(-1));
        System.out.println("Max: " + max.orElse(-1));
    }
}
```

Expected Output:

```
Min: 1
Max: 9
```

**Example 2: minBy with custom comparator**

```java
import java.util.Comparator;
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class MinByStringExample {
    public static void main(String[] args) {
        Optional<String> shortest = Stream.of("apple", "fig", "banana", "kiwi")
                .collect(Collectors.minBy(Comparator.comparingInt(String::length)));

        System.out.println("Shortest: " + shortest.orElse("none"));
    }
}
```

Expected Output:

```
Shortest: fig
```

**Real-World Cases with Explanation**

In an e-commerce system, `minBy` is used to find the cheapest product. In a sports application, `maxBy` is used to find the player with the highest score.

**References Links**

- Collectors.minBy (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#minBy(java.util.Comparator)
- Collectors.maxBy (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#maxBy(java.util.Comparator)

### 4. Downstream & Specialized Modifiers

#### 4.1 mapping(mapper, downstream)

**Definitions**

- **Core Definition:** Transforms elements before passing them to another collector.
- **Technical Definition:** `public static <T, U, A, R> Collector<T, ?, R> mapping(Function<? super T, ? extends U> mapper, Collector<? super U, A, R> downstream)` adapts a `Collector` accepting elements of type `U` to one accepting elements of type `T` by applying a mapping function to each input element before accumulation.
- **Beginner-Friendly Explanation:** `mapping` is like a two-step process: first, you transform each item in the stream into something else, and then you collect the transformed items using another collector. It is particularly useful inside `groupingBy`.

**Purposes**

- To apply a transformation before a downstream collection operation.
- To extract a specific property from objects before collecting them.
- To use as a downstream collector in `groupingBy` or `partitioningBy`.

**Syntax Structures and Rules**

Complete general syntax:

```java
Collector<T, ?, R> mapping(Function<? super T, ? extends U> mapper,
                           Collector<? super U, A, R> downstream)
```

Component breakdown:

- `mapper`: a function that transforms each element of type `T` to type `U`.
- `downstream`: a collector that accumulates elements of type `U`.
- Return value: a `Collector` that produces the result of the downstream collector.

Syntax rules: The mapper is applied to each element before it is passed to the downstream collector.

Constraints and limitations: The mapper must be non-interfering and stateless.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: mapping with groupingBy**

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

class Employee {
    String name;
    String department;
    Employee(String name, String department) { this.name = name; this.department = department; }
    String getDepartment() { return department; }
    String getName() { return name; }
}

public class MappingExample {
    public static void main(String[] args) {
        List<Employee> employees = List.of(
                new Employee("Alice", "HR"),
                new Employee("Bob", "IT"),
                new Employee("Charlie", "HR")
        );

        Map<String, List<String>> namesByDept = employees.stream()
                .collect(Collectors.groupingBy(
                        Employee::getDepartment,
                        Collectors.mapping(Employee::getName, Collectors.toList())
                ));

        System.out.println(namesByDept);
    }
}
```

Expected Output (order may vary):

```
{HR=[Alice, Charlie], IT=[Bob]}
```

**Real-World Cases with Explanation**

In a reporting system, `mapping` is used to extract only the names of employees grouped by department. In a data transformation pipeline, `mapping` is used to convert objects to their string representations before collecting.

**References Links**

- Collectors.mapping (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#mapping(java.util.function.Function,java.util.stream.Collector)

#### 4.2 flatMapping(mapper, downstream) (Added in Java 9)

**Definitions**

- **Core Definition:** Flattens collections before passing them to a downstream collector.
- **Technical Definition:** `public static <T, U, A, R> Collector<T, ?, R> flatMapping(Function<? super T, ? extends Stream<? extends U>> mapper, Collector<? super U, A, R> downstream)` adapts a `Collector` accepting elements of type `U` to one accepting elements of type `T` by applying a flat mapping function to each input element before accumulation .
- **Beginner-Friendly Explanation:** `flatMapping` is like `mapping`, but each element is expanded into multiple elements (a stream) before being collected. It is useful when each element contains a collection that you want to flatten.

**Purposes**

- To flatten nested collections before collecting.
- To handle one-to-many relationships in grouping operations.
- To replace `flatMap` when using collectors.

**Syntax Structures and Rules**

Complete general syntax:

```java
Collector<T, ?, R> flatMapping(Function<? super T, ? extends Stream<? extends U>> mapper,
                                Collector<? super U, A, R> downstream)
```

Component breakdown:

- `mapper`: a function that transforms each element into a stream of elements.
- `downstream`: a collector that accumulates the flattened elements.
- Return value: a `Collector` that produces the result of the downstream collector.

Syntax rules: The mapper must return a stream for each element. The streams are closed after their contents are placed downstream.

Constraints and limitations: The mapper must not return `null` (a `null` stream is treated as an empty stream).

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: flatMapping with groupingBy**

```java
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.stream.Collectors;

class Order {
    String customer;
    List<String> items;
    Order(String customer, List<String> items) { this.customer = customer; this.items = items; }
    String getCustomer() { return customer; }
    List<String> getItems() { return items; }
}

public class FlatMappingExample {
    public static void main(String[] args) {
        List<Order> orders = List.of(
                new Order("Alice", List.of("Laptop", "Mouse")),
                new Order("Bob", List.of("Keyboard")),
                new Order("Alice", List.of("Monitor"))
        );

        Map<String, Set<String>> itemsByCustomer = orders.stream()
                .collect(Collectors.groupingBy(
                        Order::getCustomer,
                        Collectors.flatMapping(
                                order -> order.getItems().stream(),
                                Collectors.toSet()
                        )
                ));

        System.out.println(itemsByCustomer);
    }
}
```

Expected Output (order may vary):

```
{Alice=[Laptop, Mouse, Monitor], Bob=[Keyboard]}
```

**Real-World Cases with Explanation**

In an order management system, `flatMapping` is used to collect all unique items ordered by each customer. In a social media system, `flatMapping` is used to collect all unique hashtags used by each user.

**References Links**

- Collectors.flatMapping (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#flatMapping(java.util.function.Function,java.util.stream.Collector)

#### 4.3 filtering(predicate, downstream) (Added in Java 9)

**Definitions**

- **Core Definition:** Filters elements during grouping (preventing keys from missing if no elements match).
- **Technical Definition:** `public static <T, A, R> Collector<T, ?, R> filtering(Predicate<? super T> predicate, Collector<? super T, A, R> downstream)` adapts a `Collector` to one accepting elements of the same type `T` by applying the predicate to each input element and only accumulating if the predicate returns `true` .
- **Beginner-Friendly Explanation:** `filtering` is like `filter`, but it is used inside a grouping or partitioning operation. It allows you to filter elements after grouping, and it ensures that groups with no matching elements still appear in the result.

**Purposes**

- To filter elements as part of a downstream collection operation.
- To preserve group keys even when no elements match the filter.
- To perform conditional aggregation within groups.

**Syntax Structures and Rules**

Complete general syntax:

```java
Collector<T, ?, R> filtering(Predicate<? super T> predicate,
                              Collector<? super T, A, R> downstream)
```

Component breakdown:

- `predicate`: the condition to apply to each element.
- `downstream`: a collector that accumulates the matching elements.
- Return value: a `Collector` that produces the result of the downstream collector.

Syntax rules: The predicate must be non-interfering and stateless.

Constraints and limitations: The `filtering` collector is most useful when used in a multi-level reduction, such as downstream of `groupingBy` or `partitioningBy`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: filtering within groupingBy**

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

class Employee {
    String name;
    String department;
    int salary;
    Employee(String name, String department, int salary) {
        this.name = name; this.department = department; this.salary = salary;
    }
    String getDepartment() { return department; }
    int getSalary() { return salary; }
    String getName() { return name; }
}

public class FilteringExample {
    public static void main(String[] args) {
        List<Employee> employees = List.of(
                new Employee("Alice", "HR", 50000),
                new Employee("Bob", "IT", 80000),
                new Employee("Charlie", "HR", 90000),
                new Employee("Dave", "IT", 60000)
        );

        Map<String, List<Employee>> highEarnersByDept = employees.stream()
                .collect(Collectors.groupingBy(
                        Employee::getDepartment,
                        Collectors.filtering(
                                e -> e.getSalary() > 70000,
                                Collectors.toList()
                        )
                ));

        System.out.println(highEarnersByDept);
    }
}
```

Expected Output (order may vary):

```
{HR=[Charlie], IT=[Bob]}
```

**Real-World Cases with Explanation**

In a sales reporting system, `filtering` is used to show only high-value transactions per region while still displaying regions with no high-value transactions. In a human resources system, `filtering` is used to list employees with salaries above a threshold in each department.

**References Links**

- Collectors.filtering (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#filtering(java.util.function.Predicate,java.util.stream.Collector)

#### 4.4 collectingAndThen(downstream, finisher)

**Definitions**

- **Core Definition:** Collects the elements and immediately performs a finishing transformation on the final result (e.g., turning a list immutable).
- **Technical Definition:** `public static <T, A, R, RR> Collector<T, A, RR> collectingAndThen(Collector<T, A, R> downstream, Function<R, RR> finisher)` adapts a `Collector` to perform an additional finishing transformation .
- **Beginner-Friendly Explanation:** `collectingAndThen` lets you collect the stream into a result, and then immediately transform that result into something else. For example, you can collect into a list and then make it unmodifiable.

**Purposes**

- To apply a final transformation to the result of a collector.
- To make a collected result immutable.
- To convert a collected result to a different type.

**Syntax Structures and Rules**

Complete general syntax:

```java
Collector<T, A, RR> collectingAndThen(Collector<T, A, R> downstream,
                                       Function<R, RR> finisher)
```

Component breakdown:

- `downstream`: the collector that performs the initial collection.
- `finisher`: a function that transforms the result of the downstream collector.
- Return value: a `Collector` that produces the result of the finisher function.

Syntax rules: The finisher function is applied to the final result of the downstream collector.

Constraints and limitations: The finisher function must not modify the result of the downstream collector if immutability is desired.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: collectingAndThen to create an immutable list**

```java
import java.util.Collections;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class CollectingAndThenExample {
    public static void main(String[] args) {
        List<String> immutableList = Stream.of("a", "b", "c")
                .collect(Collectors.collectingAndThen(
                        Collectors.toList(),
                        Collections::unmodifiableList
                ));

        System.out.println(immutableList);

        try {
            immutableList.add("d");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
[a, b, c]
Cannot modify: UnsupportedOperationException
```

**Example 2: collectingAndThen to get the size of a list**

```java
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class CollectingAndThenSizeExample {
    public static void main(String[] args) {
        Integer size = Stream.of("a", "b", "c", "d")
                .collect(Collectors.collectingAndThen(
                        Collectors.toList(),
                        List::size
                ));

        System.out.println("Size: " + size);
    }
}
```

Expected Output:

```
Size: 4
```

**Real-World Cases with Explanation**

In a configuration system, `collectingAndThen` is used to collect configuration entries into an immutable map. In a data processing pipeline, `collectingAndThen` is used to collect results into a list and then compute a summary statistic.

**References Links**

- Collectors.collectingAndThen (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#collectingAndThen(java.util.stream.Collector,java.util.function.Function)

#### 4.5 teeing(collector1, collector2, merger) (Added in Java 12)

**Definitions**

- **Core Definition:** Splits the stream into two independent collectors and combines their final results using a user-defined function.
- **Technical Definition:** `public static <T, R1, R2, R> Collector<T, ?, R> teeing(Collector<? super T, ?, R1> downstream1, Collector<? super T, ?, R2> downstream2, BiFunction<? super R1, ? super R2, R> merger)` returns a `Collector` that is a composite of two downstream collectors. Each element is processed by both downstream collectors, and their results are merged using the specified merger function .
- **Beginner-Friendly Explanation:** `teeing` lets you process a stream in two different ways at the same time and then combine the results. For example, you can find both the minimum and maximum value in a single pass.

**Purposes**

- To perform two independent reductions in a single pass over the stream.
- To compute multiple results that depend on different collectors.
- To avoid iterating over the stream multiple times.

**Syntax Structures and Rules**

Complete general syntax:

```java
Collector<T, ?, R> teeing(Collector<? super T, ?, R1> downstream1,
                           Collector<? super T, ?, R2> downstream2,
                           BiFunction<? super R1, ? super R2, R> merger)
```

Component breakdown:

- `downstream1`: the first collector.
- `downstream2`: the second collector.
- `merger`: a function that combines the results of the two collectors.
- Return value: a `Collector` that produces the merged result.

Syntax rules: Each element is processed by both downstream collectors. The merger function is applied to the final results.

Constraints and limitations: Introduced in Java 12. Not available in earlier versions.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: teeing to get min and max**

```java
import java.util.Comparator;
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class TeeingMinMaxExample {
    public static void main(String[] args) {
        String result = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
                .collect(Collectors.teeing(
                        Collectors.minBy(Integer::compareTo),
                        Collectors.maxBy(Integer::compareTo),
                        (min, max) -> "Min: " + min.orElse(-1) + ", Max: " + max.orElse(-1)
                ));

        System.out.println(result);
    }
}
```

Expected Output:

```
Min: 1, Max: 10
```

**Example 2: teeing to compute average and count**

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class TeeingAverageCountExample {
    public static void main(String[] args) {
        String result = Stream.of(10, 20, 30, 40, 50)
                .collect(Collectors.teeing(
                        Collectors.averagingInt(Integer::intValue),
                        Collectors.counting(),
                        (avg, count) -> "Average: " + avg + ", Count: " + count
                ));

        System.out.println(result);
    }
}
```

Expected Output:

```
Average: 30.0, Count: 5
```

**Real-World Cases with Explanation**

In a financial application, `teeing` is used to compute both the average and the total sum of transactions in a single pass. In a performance monitoring system, `teeing` is used to find both the fastest and slowest response times.

**References Links**

- Collectors.teeing (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#teeing(java.util.stream.Collector,java.util.stream.Collector,java.util.function.BiFunction)

## Summary Table of Collectors

| Collector | Category | Java Version | Return Type | Description |
|---|---|---|---|---|
| toList | Basic | 8 | List<T> | Collects into a list |
| toSet | Basic | 8 | Set<T> | Collects into a set |
| toMap | Basic | 8 | Map<K,U> | Collects into a map |
| toUnmodifiableList | Basic | 10 | List<T> | Collects into an immutable list |
| toUnmodifiableSet | Basic | 10 | Set<T> | Collects into an immutable set |
| toUnmodifiableMap | Basic | 10 | Map<K,U> | Collects into an immutable map |
| toCollection | Basic | 8 | C extends Collection<T> | Collects into a custom collection |
| joining | Basic | 8 | String | Concatenates strings |
| groupingBy | Grouping | 8 | Map<K,List<T>> | Groups by classifier |
| partitioningBy | Grouping | 8 | Map<Boolean,List<T>> | Partitions by predicate |
| summarizingInt | Numerical | 8 | IntSummaryStatistics | Summary statistics |
| summingInt | Numerical | 8 | Integer | Sum of values |
| averagingInt | Numerical | 8 | Double | Average of values |
| counting | Numerical | 8 | Long | Count of elements |
| minBy | Numerical | 8 | Optional<T> | Minimum element |
| maxBy | Numerical | 8 | Optional<T> | Maximum element |
| mapping | Downstream | 8 | R | Maps before collecting |
| flatMapping | Downstream | 9 | R | Flattens before collecting |
| filtering | Downstream | 9 | R | Filters during grouping |
| collectingAndThen | Downstream | 8 | RR | Finishing transformation |
| teeing | Downstream | 12 | R | Combines two collectors |

## Version-Specific Notes

- `toUnmodifiableList`, `toUnmodifiableSet`, and `toUnmodifiableMap` were introduced in Java 10.
- `filtering` and `flatMapping` were introduced in Java 9.
- `teeing` was introduced in Java 12.
- `Stream.toList()` was introduced in Java 16 and returns an unmodifiable list, differing from `Collectors.toList()`.
- `Collectors.toList()` makes no guarantees about the type, mutability, serializability, or thread-safety of the returned list.

## References

- Collectors (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html
- Collectors (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collectors.html
- Collectors (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html
- IntSummaryStatistics (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/IntSummaryStatistics.html
- DoubleSummaryStatistics (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/DoubleSummaryStatistics.html
- LongSummaryStatistics (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LongSummaryStatistics.html
- Stream (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html
- Reduction (Java Tutorials) – https://docs.oracle.com/javase/tutorial/collections/streams/reduction.html
- Collectors.teeing (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#teeing(java.util.stream.Collector,java.util.stream.Collector,java.util.function.BiFunction)
- Collectors.filtering (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#filtering(java.util.function.Predicate,java.util.stream.Collector)
- Collectors.flatMapping (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#flatMapping(java.util.function.Function,java.util.stream.Collector)