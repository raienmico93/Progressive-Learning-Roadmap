# Java Enhanced For Loop: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The enhanced `for` loop (commonly called the for-each loop) is a Java language construct introduced in Java 5 that provides a simplified, read-only traversal of arrays and any object implementing `java.lang.Iterable`, eliminating the need for explicit index management or iterator handling.

**Technical Definition:** According to the Java Language Specification (JLS §14.14.2), the enhanced `for` statement has the form `for (LocalVariableDeclaration : Expression) Statement`, where “the type of the Expression must be an array type or a subtype of the raw type Iterable, or a compile-time error occurs”. On each iteration, “the local variable is initialized ... to successive elements of the Iterable or the array produced by the expression”. The construct is syntactic sugar: for an `Iterable`, the compiler translates the loop into an explicit `Iterator`-based `while` loop (obtaining the iterator once via `iterator()`, then calling `hasNext()` and `next()` on each iteration); for an array, it translates to an index-based `for` loop.

**Beginner-Friendly Explanation:** Imagine you have a basket of apples and you need to inspect each one. The traditional `for` loop is like being told “start at position 0, check if the position is less than the number of apples, and increment the position each time.” The enhanced `for` loop is like being told simply “for each apple in the basket, inspect it.” You don’t manage the position, you don’t call the basket’s internal methods — Java does all of that behind the scenes. The enhanced `for` loop is the idiomatic, recommended way to traverse arrays and collections when you only need to read each element.

### Key Characteristics

- **Read-Only Traversal:** The loop variable is a local copy of the current element; assigning to it does not modify the underlying array or collection.
- **Works with Arrays and `Iterable`:** The expression must be an array or a subtype of `Iterable`; otherwise, a compile-time error occurs.
- **Hides Iteration Details:** For collections, the compiler generates an `Iterator` internally; for arrays, it generates an index-based loop.
- **Concise and Idiomatic:** Oracle recommends the enhanced `for` loop for arrays and collections when full traversal is needed.
- **Cannot Modify the Structure:** Adding or removing elements from a collection during for-each iteration (except via `Iterator.remove()`) causes `ConcurrentModificationException` in most implementations.
- **Single Structure per Loop:** The enhanced `for` loop iterates over exactly one array or collection at a time.

### Prerequisites

- Basic Java syntax (variables, operators, statements)
- Understanding of arrays and the Collections Framework
- Familiarity with the `java.lang.Iterable` and `java.util.Iterator` interfaces
- Knowledge of generics (for type-safe collection iteration)

### Related Programming Areas

- **Collections Framework:** The enhanced `for` loop is the primary way to traverse `List`, `Set`, and `Queue` implementations.
- **Arrays:** It simplifies reading every element of an array without index management.
- **Iterator Pattern:** The loop is built on the `Iterator` interface, which provides `hasNext()`, `next()`, and `remove()`.
- **Streams API:** For declarative, functional-style processing, the Streams API offers an alternative to for-each loops.
- **Map Iteration:** Maps do not implement `Iterable`, so for-each requires iterating over `entrySet()`, `keySet()`, or `values()`.

---

## Core Concepts / Features

---

## 1. For-Each Loop

### Definitions

**Core Definition:** The for-each loop is a simplified iteration statement that executes a block of code for each element of an array or `Iterable`.

**Technical Definition:** The enhanced `for` statement is defined in JLS §14.14.2. It declares a single local variable (the loop variable) and an expression that evaluates to an array or an `Iterable`. The loop variable is assigned successive elements on each iteration. The loop variable’s type must be compatible with the element type of the expression. The loop variable can be declared `final` to prevent accidental assignment inside the body.

**Beginner-Friendly Explanation:** The for-each loop is the “do this for every item” loop. You name a variable to hold each item, name the collection or array to walk through, and write the body. Java handles the mechanics of moving from one item to the next.

### Purposes

- To traverse every element of an array or `Iterable` without managing an index or iterator
- To improve code readability and reduce off-by-one errors
- To express the intent of full traversal clearly and idiomatically
- To serve as the recommended loop form for arrays and collections

### Syntax Rules and Structure

**Complete General Syntax:**

```java
for (Type element : expression) {
    // statement(s) using element
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Type` | The type of the elements (or a supertype); can be `var` (Java 10+) for local type inference. |
| `element` | The loop variable, holding the current element. |
| `expression` | An array or an `Iterable`; otherwise, compile-time error. |
| `statement(s)` | The body executed once per element. |

**Syntax Rules:**

- The expression must be an array type or a subtype of `Iterable`.
- The loop variable is scoped to the loop body.
- The loop variable can be declared `final` to prevent reassignment.
- `break`, `continue`, and labelled forms work as in other loops.

**Constraints and Limitations:**

- The loop variable is a copy; assigning to it does not modify the underlying data.
- The loop cannot iterate in reverse or skip elements without `continue`.
- Modifying the collection structurally during iteration (except via `Iterator.remove()`) is unsafe.

### Annotated Complete Step-by-Step Code Examples

**Example: Array and Collection Iteration**

```java
// File: EnhancedForDemo.java
import java.util.List;

public class EnhancedForDemo {
    public static void main(String[] args) {
        // Array iteration
        int[] numbers = {10, 20, 30, 40, 50};
        for (int number : numbers) {
            System.out.println("Array element: " + number);
        }

        // Collection iteration
        List<String> names = List.of("Alice", "Bob", "Charlie");
        for (String name : names) {
            System.out.println("Collection element: " + name);
        }
    }
}
```

**Expected Output:**
```
Array element: 10
Array element: 20
Array element: 30
Array element: 40
Array element: 50
Collection element: Alice
Collection element: Bob
Collection element: Charlie
```

**Step-by-Step Explanation:**

1. For the array `numbers`, the compiler generates an index-based loop that reads `numbers[0]`, `numbers[1]`, and so on, assigning each to `number` before the body executes.
2. For the `List` `names`, the compiler calls `names.iterator()` once, then repeatedly calls `hasNext()` and `next()`, assigning each returned element to `name`.
3. The body prints the current element on each iteration.
4. The loop terminates when all elements have been processed.

### Real-World Cases

- **Summing Collections:** Computing the total of all numeric elements.
- **Filtering:** Checking each element against a predicate.
- **Rendering Lists:** Displaying every item in a UI list.
- **Batch Processing:** Applying an operation to every record in a data set.

---

## 2. `Iterable` Interface

### Definitions

**Core Definition:** `java.lang.Iterable<T>` is the root interface that enables an object to be the target of the enhanced `for` statement.

**Technical Definition:** The `Iterable` interface declares a single abstract method, `Iterator<T> iterator()`, which returns an iterator over the elements. Implementing `Iterable` allows an object to be used in a for-each loop. The interface also provides default methods `forEach(Consumer)` and `spliterator()`.

**Beginner-Friendly Explanation:** `Iterable` is a promise: “I can give you an iterator so you can visit my elements one by one.” Anything that implements this promise can be used with the for-each loop.

### Purposes

- To mark classes as “for-each compatible”
- To provide a standard way to obtain an iterator over elements
- To decouple iteration from the underlying data structure
- To enable the compiler to translate for-each into iterator-based traversal

### Syntax Rules and Structure

```java
public interface Iterable<T> {
    Iterator<T> iterator();
    default void forEach(Consumer<? super T> action) { ... }
    default Spliterator<T> spliterator() { ... }
}
```

**Syntax Rules:**

- The enhanced `for` loop works with any type that is a subtype of `Iterable` (raw type) or an array.
- The `iterator()` method must return a non-null `Iterator`.
- The `forEach` default method performs the given action for each element, behaving as if by a for-each loop.

### Annotated Code Example

```java
// File: IterableDemo.java
import java.util.Iterator;
import java.util.List;

public class IterableDemo {
    static class NumberBox implements Iterable<Integer> {
        private final List<Integer> numbers;

        NumberBox(List<Integer> numbers) {
            this.numbers = numbers;
        }

        @Override
        public Iterator<Integer> iterator() {
            return numbers.iterator();
        }
    }

    public static void main(String[] args) {
        NumberBox box = new NumberBox(List.of(1, 2, 3));
        for (int n : box) {
            System.out.println(n);
        }
    }
}
```

**Expected Output:**
```
1
2
3
```

**Step-by-Step Explanation:**

1. `NumberBox` implements `Iterable<Integer>` and delegates `iterator()` to the internal list.
2. Because `NumberBox` is `Iterable`, it can be the target of the for-each loop.
3. The compiler translates the loop into iterator-based traversal using `box.iterator()`.

### Real-World Cases

- **Custom Collections:** Making user-defined data structures for-each compatible.
- **Domain Models:** Exposing an iterable view of internal elements (e.g., a `Playlist` iterating over `Song`s).
- **Frameworks:** Many frameworks accept `Iterable` parameters to allow flexible input.

---

## 3. `Iterator` Interface

### Definitions

**Core Definition:** `java.util.Iterator<E>` is the interface that provides sequential access to elements of a collection, with methods to check for more elements, retrieve the next element, and optionally remove elements.

**Technical Definition:** The `Iterator` interface declares `boolean hasNext()`, `E next()`, and `default void remove()`, plus `forEachRemaining(Consumer)`. `remove()` removes the last element returned by `next()` and can be called only once per `next()` call; otherwise, it throws `IllegalStateException`. The behaviour of an iterator is unspecified if the underlying collection is modified during iteration except by `remove()`.

**Beginner-Friendly Explanation:** An `Iterator` is a remote control for a collection. `hasNext()` tells you if there are more channels; `next()` changes to the next channel and returns it; `remove()` deletes the current channel (if supported).

### Purposes

- To provide a uniform way to traverse different collection implementations
- To allow safe removal of elements during iteration via `remove()`
- To serve as the underlying mechanism for the enhanced `for` loop
- To support streaming-style processing with `forEachRemaining`

### Syntax Rules and Structure

```java
Iterator<T> it = collection.iterator();
while (it.hasNext()) {
    T element = it.next();
    // process
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `hasNext()` | Returns `true` if more elements remain |
| `next()` | Returns the next element; throws `NoSuchElementException` if none |
| `remove()` | Removes the last returned element (optional operation) |
| `forEachRemaining(Consumer)` | Performs an action for each remaining element |

**Constraints and Limitations:**

- `remove()` is optional; some iterators throw `UnsupportedOperationException`.
- Modifying the collection during iteration (other than via `remove()`) may cause `ConcurrentModificationException` or unspecified behaviour.

### Annotated Code Example

```java
// File: IteratorDemo.java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorDemo {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>(List.of(1, 2, 3, 4, 5));

        Iterator<Integer> it = numbers.iterator();
        while (it.hasNext()) {
            int n = it.next();
            if (n % 2 == 0) {
                it.remove(); // Safe removal during iteration
            }
        }

        System.out.println(numbers); // [1, 3, 5]
    }
}
```

**Expected Output:**
```
[1, 3, 5]
```

**Step-by-Step Explanation:**

1. `iterator()` obtains an iterator over the list.
2. `hasNext()` checks for remaining elements.
3. `next()` returns the next element; even numbers are removed via `it.remove()`.
4. Removing via the iterator is the only safe way to modify the list during traversal.
5. The final list contains only odd numbers.

### Real-World Cases

- **Safe Removal:** Deleting elements from a list while traversing it.
- **Custom Iteration:** Implementing custom traversal logic that the for-each loop cannot express (e.g., reverse iteration).
- **Streaming:** Using `forEachRemaining` to process remaining elements in bulk.

---

## 4. Array Iteration

### Definitions

**Core Definition:** Array iteration with the enhanced `for` loop traverses every element of an array in index order, from the first element to the last.

**Technical Definition:** When the expression is an array, the enhanced `for` loop is compiled into an index-based loop that accesses each element by its index. The loop variable receives a copy of the array element. For primitive arrays, the copy is a primitive value; for object arrays, the copy is a reference to the object.

**Beginner-Friendly Explanation:** The for-each loop walks through an array from start to finish, handing you each element in turn. You do not need to know the length or manage the index.

### Purposes

- To read every element of an array without index management
- To reduce off-by-one errors and improve readability
- To express full traversal clearly

### Syntax Rules and Structure

```java
for (ElementType element : array) {
    // process element
}
```

**Syntax Rules:**

- The expression must be an array (any dimension).
- The loop variable type must be compatible with the array’s component type.
- The loop cannot be used to modify the array through the loop variable; assignments affect only the local copy.

### Annotated Code Example

```java
// File: ArrayIterationDemo.java
public class ArrayIterationDemo {
    public static void main(String[] args) {
        int[] scores = {85, 92, 78, 95, 88};
        int total = 0;

        for (int score : scores) {
            total += score;
        }

        System.out.println("Total: " + total);
        System.out.println("Average: " + (double) total / scores.length);
    }
}
```

**Expected Output:**
```
Total: 438
Average: 87.6
```

**Step-by-Step Explanation:**

1. The for-each loop assigns each element of `scores` to `score` in order.
2. The body adds `score` to `total`.
3. After the loop, `total` is `438`.
4. The average is computed using the array length.

### Real-World Cases

- **Grade Processing:** Summing or averaging student scores.
- **Data Validation:** Checking every element against constraints.
- **Array Search:** Finding an element that matches a condition.

---

## 5. Collection Iteration

### Definitions

**Core Definition:** Collection iteration with the enhanced `for` loop traverses the elements of any `Iterable` collection, including `List`, `Set`, and `Queue` implementations.

**Technical Definition:** For a collection, the enhanced `for` loop obtains an iterator via `iterator()` and repeatedly calls `hasNext()` and `next()`. The iteration order depends on the collection type: `List` preserves insertion order; `HashSet` does not guarantee order; `TreeSet` iterates in sorted order; `LinkedHashSet` preserves insertion order.

**Beginner-Friendly Explanation:** The for-each loop is the standard way to visit every item in a list, set, or queue. It hides the iterator mechanics and lets you focus on the elements.

### Purposes

- To traverse lists, sets, and queues idiomatically
- To eliminate iterator boilerplate
- To provide a consistent iteration syntax across collection types
- To serve as the recommended form for collection traversal

### Syntax Rules and Structure

```java
for (ElementType element : collection) {
    // process element
}
```

**Syntax Rules:**

- The collection must implement `Iterable`.
- The loop variable receives each element in the collection’s iteration order.
- Structural modification during iteration (except via `Iterator.remove()`) is unsafe.

### Annotated Code Example

```java
// File: CollectionIterationDemo.java
import java.util.*;

public class CollectionIterationDemo {
    public static void main(String[] args) {
        List<String> list = List.of("Apple", "Banana", "Cherry");
        Set<Integer> set = new TreeSet<>(Set.of(30, 10, 20));

        System.out.println("List:");
        for (String fruit : list) {
            System.out.println(fruit);
        }

        System.out.println("TreeSet (sorted):");
        for (int n : set) {
            System.out.println(n);
        }
    }
}
```

**Expected Output:**
```
List:
Apple
Banana
Cherry
TreeSet (sorted):
10
20
30
```

**Step-by-Step Explanation:**

1. The `List` is traversed in insertion order: Apple, Banana, Cherry.
2. The `TreeSet` is traversed in sorted order: 10, 20, 30.
3. The for-each loop adapts to the collection’s iteration order automatically.

### Real-World Cases

- **Shopping Cart:** Iterating over cart items.
- **Unique Visitors:** Iterating over a `Set` of user IDs.
- **Task Queues:** Processing queued tasks in FIFO order.

---

## 6. Map Iteration

### Definitions

**Core Definition:** Map iteration with the enhanced `for` loop requires iterating over one of the map’s collection views: `entrySet()`, `keySet()`, or `values()`.

**Technical Definition:** `Map` does not implement `Iterable`, so it cannot be the direct target of a for-each loop. Instead, you iterate over the set views returned by `entrySet()` (key-value pairs as `Map.Entry`), `keySet()` (keys only), or `values()` (values only). The `entrySet()` approach is the most efficient when both keys and values are needed, because it avoids repeated lookups.

**Beginner-Friendly Explanation:** A `Map` is like a dictionary — it’s not a list, so you can’t walk through it directly with for-each. But you can walk through its list of entries (key-value pairs), its list of keys, or its list of values. The entry-set approach is best because each entry contains both the key and the value, so you don’t have to look up the value separately.

### Purposes

- To iterate over map entries, keys, or values using for-each
- To choose the most efficient view for the task
- To avoid the boilerplate of explicit iterators

### Syntax Rules and Structure

```java
// Entry set (most common, most efficient for key + value)
for (Map.Entry<K, V> entry : map.entrySet()) {
    K key = entry.getKey();
    V value = entry.getValue();
}

// Key set (use when only keys are needed, or values are looked up separately)
for (K key : map.keySet()) {
    V value = map.get(key);
}

// Values (use when only values are needed)
for (V value : map.values()) {
    // process value
}
```

**Syntax Rules:**

- `entrySet()` returns a `Set<Map.Entry<K,V>>`.
- `keySet()` returns a `Set<K>`.
- `values()` returns a `Collection<V>` (not a `Set`).
- All three views are backed by the map, so changes to the map are reflected in the views.

### Annotated Code Example

```java
// File: MapIterationDemo.java
import java.util.*;

public class MapIterationDemo {
    public static void main(String[] args) {
        Map<String, Integer> ages = new LinkedHashMap<>();
        ages.put("Alice", 30);
        ages.put("Bob", 25);
        ages.put("Charlie", 35);

        // Entry set: most efficient for key + value
        System.out.println("Entries:");
        for (Map.Entry<String, Integer> entry : ages.entrySet()) {
            System.out.println(entry.getKey() + " -> " + entry.getValue());
        }

        // Key set: only keys
        System.out.println("Keys:");
        for (String key : ages.keySet()) {
            System.out.println(key);
        }

        // Values: only values
        System.out.println("Values:");
        for (int age : ages.values()) {
            System.out.println(age);
        }
    }
}
```

**Expected Output:**
```
Entries:
Alice -> 30
Bob -> 25
Charlie -> 35
Keys:
Alice
Bob
Charlie
Values:
30
25
35
```

**Step-by-Step Explanation:**

1. `entrySet()` provides both key and value in a single iteration, avoiding a separate `get()` lookup.
2. `keySet()` iterates only keys, suitable when the value is not needed.
3. `values()` iterates only values, suitable when the key is not needed.
4. The `LinkedHashMap` preserves insertion order, so the output follows the order in which entries were added.

### Real-World Cases

- **Frequency Maps:** Counting occurrences of words in a document.
- **Configuration Lookup:** Iterating over property key-value pairs.
- **Inventory Reports:** Printing every product and its stock level.

---

## 7. Limitations

### Definitions

**Core Definition:** The enhanced `for` loop has several limitations that restrict its applicability, including read-only element access, inability to modify the underlying structure safely, and lack of access to the index.

**Technical Definition:** Because the loop variable is a local copy, assignments to it do not affect the underlying array or collection. For collections, structural modification during iteration (adding or removing elements) causes `ConcurrentModificationException` in most implementations, except when using `Iterator.remove()` via an explicit iterator. The loop also cannot iterate over multiple structures simultaneously or access the index of the current element.

**Beginner-Friendly Explanation:** The for-each loop is a “look, don’t touch” tool. You can read every element, but you cannot change the elements, remove them, or add new ones safely. If you need to modify the collection, you need a traditional loop or an explicit iterator.

### Key Limitations Table

| Limitation | Description | Alternative |
|------------|-------------|-------------|
| **Read-only element access** | Assigning to the loop variable does not modify the underlying data. | Use a traditional `for` loop with index assignment. |
| **Cannot remove elements** | Structural modification causes `ConcurrentModificationException`. | Use an explicit `Iterator` with `remove()`. |
| **Cannot add elements** | Adding elements during for-each is unsafe. | Collect additions and apply after the loop. |
| **No index access** | The loop does not provide the current index. | Use a traditional `for` loop. |
| **Single structure** | Cannot iterate over two structures simultaneously. | Use nested loops or a traditional `for` with shared index. |
| **No reverse iteration** | Always iterates in the structure’s natural order. | Use a traditional `for` loop or `ListIterator`. |
| **No element replacement** | Cannot replace elements through the loop variable. | Use `List.set()` with an index-based loop. |

### Annotated Code Example: Read-Only Limitation

```java
// File: LimitationDemo.java
public class LimitationDemo {
    public static void main(String[] args) {
        int[] numbers = {1, 2, 3};

        // Does NOT modify the array
        for (int n : numbers) {
            n = n * 2;
        }
        System.out.println(java.util.Arrays.toString(numbers)); // [1, 2, 3]

        // Correct way: index-based loop
        for (int i = 0; i < numbers.length; i++) {
            numbers[i] = numbers[i] * 2;
        }
        System.out.println(java.util.Arrays.toString(numbers)); // [2, 4, 6]
    }
}
```

**Expected Output:**
```
[1, 2, 3]
[2, 4, 6]
```

**Step-by-Step Explanation:**

1. The for-each loop assigns each array element to the local variable `n`; assigning `n = n * 2` changes only the local variable, not the array.
2. The index-based loop accesses `numbers[i]` directly, so assignment modifies the array.

### Real-World Cases

- **In-Place Transformation:** Doubling every element of an array requires an index-based loop.
- **Filtering with Removal:** Removing elements based on a condition requires an explicit `Iterator`.
- **Index-Based Algorithms:** Algorithms that need neighbouring elements (e.g., `arr[i] + arr[i+1]`) require index access.

---

## 8. Best Practices

### Definitions

**Core Definition:** Best practices for the enhanced `for` loop are guidelines for using it correctly, safely, and idiomatically, while knowing when to choose a different loop form.

**Technical Definition:** The SEI CERT Oracle Coding Standard for Java provides specific rules: DCL30-J recommends declaring the loop variable `final` to prevent accidental assignment, because “assignments to the loop variable of an enhanced for loop (for-each idiom) fail to affect the overall iteration order … and can leave data in a fragile or inconsistent state”. DCL02-J advises against modifying the collection’s elements during an enhanced for statement. Oracle recommends the enhanced `for` loop for arrays and collections when full traversal is needed.

**Beginner-Friendly Explanation:** Best practices help you use the for-each loop where it shines — simple, read-only traversal — and switch to a different tool when you need to modify data or access indices. Declaring the loop variable `final` is a simple habit that prevents a whole class of confusing bugs.

### Key Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use for-each for read-only traversal** | The enhanced `for` loop is the idiomatic choice for arrays and collections when no modification is needed. |
| **Declare the loop variable `final`** | Prevents accidental assignment that has no effect on the underlying data. |
| **Do not modify the collection structurally** | Adding or removing elements during for-each causes `ConcurrentModificationException`; use an explicit `Iterator` with `remove()` instead. |
| **Use `entrySet()` for map iteration** | When both keys and values are needed, `entrySet()` is more efficient than `keySet()` + `get()`. |
| **Choose the right loop for modification** | Use a traditional index-based `for` loop or an explicit `Iterator` when elements must be modified or removed. |
| **Prefer for-each for readability** | When no index or modification is needed, for-each is clearer and less error-prone than manual index management. |
| **Be aware of performance** | For arrays, for-each is generally as fast as index-based loops; for `LinkedList`, for-each is significantly faster than index-based access. |

### Annotated Complete Step-by-Step Code Examples

```java
// File: BestPracticesDemo.java
import java.util.*;

public class BestPracticesDemo {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>(List.of("Alice", "Bob", "Charlie"));

        // GOOD: Read-only traversal with final loop variable
        for (final String name : names) {
            System.out.println(name);
        }

        // GOOD: Safe removal using explicit Iterator
        Iterator<String> it = names.iterator();
        while (it.hasNext()) {
            if (it.next().startsWith("B")) {
                it.remove(); // Safe removal
            }
        }
        System.out.println("After removal: " + names);

        // GOOD: Map iteration with entrySet()
        Map<String, Integer> ages = Map.of("Alice", 30, "Bob", 25);
        for (Map.Entry<String, Integer> entry : ages.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }
    }
}
```

**Expected Output:**
```
Alice
Bob
Charlie
After removal: [Alice, Charlie]
Alice: 30
Bob: 25
```

**Step-by-Step Explanation:**

1. The for-each loop with `final String name` prevents accidental reassignment.
2. The explicit `Iterator` allows safe removal of “Bob” without `ConcurrentModificationException`.
3. The map iteration uses `entrySet()` for efficient key-value access.
4. Each best practice is applied in the appropriate context.

### Real-World Cases

- **Code Reviews:** Checking that for-each loops do not attempt to modify collections.
- **Static Analysis:** Tools like Checkstyle and Error Prone flag assignments to for-each loop variables.
- **API Design:** Library authors document whether returned collections are safe for for-each iteration.

---

## References

- The Java Language Specification, §14.14.2 The enhanced `for` statement — https://docs.oracle.com/en/java/javase/22/docs/specs/unnamed-jls.html#5 
- Interface `Iterable<T>` — Java Platform SE 11 API Specification — https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Iterable.html 
- Interface `Iterator<E>` — Java Platform SE 8 API Specification — https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html 
- DCL30-J. Do not attempt to assign to the loop variable in an enhanced for loop — SEI CERT — https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88505597 
- DCL02-J. Do not modify the collection’s elements during an enhanced for statement — SEI CERT — https://cmu-sei.github.io 
- Iterate Over a Map in Java — Baeldung — https://www.baeldung.com/java-iterate-map 
- The For-Each Loop — Oracle Java Tutorials — https://docs.oracle.com/javase/tutorial/java/nutsandbolts/for.html 
- Enhanced for loop in Java — CodeGym — https://codegym.cc 