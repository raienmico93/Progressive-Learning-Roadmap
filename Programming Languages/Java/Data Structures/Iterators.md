# Java Iteration: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Java iteration is the process of traversing the elements of a collection, array, or other aggregate data structure in a sequential or partitioned manner, applying an operation to each element or examining elements one by one.

**Technical Definition:** Java iteration encompasses the language constructs (`for`, enhanced `for`), interfaces (`Iterable`, `Iterator`, `ListIterator`, `Spliterator`, `Enumeration`), and library methods (`forEach`, Stream API) that enable systematic traversal of data sources. The framework distinguishes between external iteration, where the client controls the traversal loop, and internal iteration, where the data structure or library controls the traversal and the client supplies a behavioral parameter. It also distinguishes between fail-fast iterators, which detect concurrent structural modification and throw `ConcurrentModificationException`, and fail-safe (weakly consistent) iterators, which operate on a snapshot or tolerate concurrent modification.

**Beginner-Friendly Explanation:** Iteration is simply going through a list of items one by one. In Java, you can do this with a classic `for` loop, a simplified `for-each` loop, an `Iterator` object, or modern functional methods like `forEach` and `stream()`. Each approach has its own strengths, and Java provides many tools so you can choose the one that fits your situation.

### Key Characteristics

- The Java iteration framework is built around the `Iterable` interface, which enables the enhanced `for` loop. 
- The `Iterator` interface provides the universal traversal mechanism for collections, while `ListIterator` adds bidirectional traversal and element modification for lists. 
- The `forEach()` method, introduced in Java 8, enables internal iteration with lambda expressions. 
- The Stream API provides functional-style traversal with lazy evaluation and parallel execution. 
- `Spliterator` supports efficient parallel traversal by partitioning elements. 
- The framework distinguishes fail-fast iterators (which throw `ConcurrentModificationException` on concurrent structural modification) from fail-safe iterators (which use snapshots or weakly consistent semantics). 
- The legacy `Enumeration` interface predates `Iterator` and is retained for compatibility with legacy classes such as `Vector` and `Hashtable`.

### Prerequisites

- Readers should be familiar with Java generics.
- The Java Collections Framework (particularly `List`, `Set`, and `Collection`). 
- Basic exception handling, and the `Comparable` and `Comparator` interfaces. 
- For modern iteration, familiarity with lambda expressions and functional interfaces (such as `Consumer`) is assumed. 
- For parallel traversal, a basic understanding of threads and concurrency is helpful.

### Related Programming Areas with Explanation

- Iteration is related to data structures and algorithms (traversal of trees, graphs, and linear structures).
- Functional programming (through the Stream API and lambda expressions)
- Concurrent programming (through fail-fast versus fail-safe semantics and parallel streams).
- Software design patterns (such as the Iterator pattern, which is a behavioral design pattern that provides a way to access elements of an aggregate object sequentially without exposing its underlying representation).

## Core Concepts and Key Features

### 1. Traditional for Loop

#### Core Definitions

**Core Definition:** An index-based loop that repeatedly executes a block of code while incrementing or decrementing a counter variable.

**Technical Definition:** The basic `for` statement in Java, defined in the Java Language Specification §14.14.1, has the form `for ( [ForInit] ; [Expression] ; [ForUpdate] ) Statement`. The `ForInit` typically declares and initializes a loop variable, the `Expression` is a boolean condition evaluated before each iteration, and the `ForUpdate` is executed after each iteration.

**Beginner-Friendly Explanation:** The traditional `for` loop is the classic way to go through a list. You start at index 0, keep going while the index is less than the list's size, and increment the index each time. It is especially useful when you need to know the index of each element.

#### 1.1 Index-Based Traversal

**Definitions**

- **Core Definition:** Traversing a collection or array using a numeric index that advances from the first position to the last.
- **Technical Definition:** Index-based traversal uses `list.get(i)` or `array[i]` to access elements, with `i` ranging from `0` to `size() - 1` or `length - 1`. The loop condition checks `i < list.size()` or `i < array.length`.
- **Beginner-Friendly Explanation:** You use the element's position number (index) to fetch it. For example, to go through an array of names, you start at position 0 and keep going until you have visited every position.

**Purposes**

- To traverse a collection or array while having access to the index of each element.
- To perform index-dependent operations, such as inserting elements at specific positions.
- To traverse arrays, which do not implement the `Iterable` interface.

**Syntax Structures and Rules**

Complete general syntax:

```java
for (int i = 0; i < collection.size(); i++) {
    E element = collection.get(i);
    // process element
}
```

Component breakdown:

- `int i = 0`: initializes the loop variable.
- `i < collection.size()`: the loop condition.
- `i++`: the update expression.
- `collection.get(i)`: retrieves the element at index `i`.

Syntax rules: The loop variable must be declared before or in the `ForInit`. The condition must be a boolean expression. The update expression must modify the loop variable.

Constraints and limitations: For linked structures such as `LinkedList`, `get(i)` is an O(n) operation, making index-based traversal O(n²). Index-based traversal is not suitable for collections that do not support positional access (e.g., `Set`).

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Index-based traversal of an array**

```java
public class ForLoopArrayExample {
    public static void main(String[] args) {
        // Step 1: Declare an array
        String[] names = {"Alice", "Bob", "Charlie"};

        // Step 2: Use a traditional for loop with index
        for (int i = 0; i < names.length; i++) {
            // Step 3: Access element by index and print with position
            System.out.println("Index " + i + ": " + names[i]);
        }
    }
}
```

Expected Output:

```
Index 0: Alice
Index 1: Bob
Index 2: Charlie
```

**Why the code produces this result:** The loop variable `i` starts at 0 and increments until it reaches `names.length` (3). At each iteration, `names[i]` accesses the element at the current index.

**Example 2: Index-based traversal of an ArrayList**

```java
import java.util.ArrayList;
import java.util.List;

public class ForLoopListExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Apple");
        list.add("Banana");
        list.add("Cherry");

        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }
}
```

Expected Output:

```
Apple
Banana
Cherry
```

**Example 3: Removing elements during index-based traversal**

```java
import java.util.ArrayList;
import java.util.List;

public class ForLoopRemoveExample {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>();
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
        numbers.add(4);

        // Remove even numbers using index-based traversal
        for (int i = 0; i < numbers.size(); i++) {
            if (numbers.get(i) % 2 == 0) {
                numbers.remove(i);
                i--; // Adjust index after removal
            }
        }
        System.out.println(numbers);
    }
}
```

Expected Output:

```
[1, 3]
```

**Why the code produces this result:** After removing an element at index `i`, all subsequent elements shift left by one position. Decrementing `i` ensures the element that shifted into position `i` is examined.

**Real-World Cases with Explanation**

Index-based `for` loops are used when the index is needed for computation, such as computing a weighted sum where the weight depends on position, or when elements must be inserted or removed at specific indices.

**References Links**

- The for Statement (Java Language Specification, Java SE 21) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.14
- ArrayList (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html

### 2. Enhanced for Loop (for-each)

#### Core Definitions

**Core Definition:** A simplified loop construct that iterates over arrays and `Iterable` types without requiring an explicit index or iterator.

**Technical Definition:** The enhanced `for` statement, defined in the Java Language Specification §14.14.2, has the form `for ( FormalParameter : Expression ) Statement`. The `Expression` must be of type `Iterable` or an array type. The enhanced `for` is equivalent to a basic `for` loop using an `Iterator` for `Iterable` types, or an index for arrays.

**Beginner-Friendly Explanation:** The for-each loop is a shortcut that says “for each item in this collection, do something.” You do not need to manage an index or an iterator; Java handles that behind the scenes.

#### 2.1 Traversal of Iterable Types

**Definitions**

- **Core Definition:** Using the enhanced `for` loop to traverse any object that implements the `Iterable` interface.
- **Technical Definition:** For an `Iterable` expression, the enhanced `for` statement is equivalent to obtaining an `Iterator` via `iterator()` and repeatedly calling `hasNext()` and `next()` until the iterator is exhausted.
- **Beginner-Friendly Explanation:** Any collection (like `ArrayList`, `HashSet`) that implements `Iterable` can be traversed with `for (Type item : collection)`.

**Purposes**

- To traverse `Iterable` collections concisely without managing an iterator.
- To improve code readability by eliminating boilerplate iterator code.
- To reduce the risk of errors associated with manual index management.

**Syntax Structures and Rules**

Complete general syntax:

```java
for (Type var : iterableExpression) {
    // process var
}
```

Component breakdown:

- `Type var`: the loop variable, which receives each element.
- `iterableExpression`: an expression of type `Iterable` or an array.

Syntax rules: The loop variable is assigned each element in turn. The loop terminates when the iterator is exhausted. The loop variable must be declared in the `FormalParameter`.

Constraints and limitations: The enhanced `for` loop does not expose the index or the iterator. The collection must not be structurally modified during iteration (except via the iterator's own `remove()` method, which is not accessible through the enhanced `for`). Arrays are also supported.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Traversing a List**

```java
import java.util.ArrayList;
import java.util.List;

public class ForEachListExample {
    public static void main(String[] args) {
        List<String> fruits = new ArrayList<>();
        fruits.add("Apple");
        fruits.add("Banana");
        fruits.add("Cherry");

        // Enhanced for loop
        for (String fruit : fruits) {
            System.out.println(fruit);
        }
    }
}
```

Expected Output:

```
Apple
Banana
Cherry
```

**Why the code produces this result:** The enhanced `for` loop obtains an iterator from the `ArrayList` and iterates through each element, assigning it to the variable `fruit`.

**Example 2: Traversing a Set**

```java
import java.util.HashSet;
import java.util.Set;

public class ForEachSetExample {
    public static void main(String[] args) {
        Set<Integer> numbers = new HashSet<>();
        numbers.add(10);
        numbers.add(20);
        numbers.add(30);

        for (int n : numbers) {
            System.out.println(n);
        }
    }
}
```

Expected Output (order may vary):

```
10
20
30
```

**Example 3: Traversing an array**

```java
public class ForEachArrayExample {
    public static void main(String[] args) {
        int[] values = {5, 10, 15};
        for (int v : values) {
            System.out.println(v);
        }
    }
}
```

Expected Output:

```
5
10
15
```

**Real-World Cases with Explanation**

The enhanced `for` loop is used wherever a collection needs to be traversed without modification, such as printing all elements, computing a sum, or applying a read-only operation to each element.

**References Links**

- The Enhanced for Statement (Java Language Specification, Java SE 21) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.14.2
- Iterable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Iterable.html

### 3. Iterator Interface

#### Core Definitions

**Core Definition:** An object that enables sequential traversal of a collection, with the ability to remove elements from the underlying collection during iteration.

**Technical Definition:** `public interface Iterator<E>` is the universal iterator for the Java Collections Framework. It declares `boolean hasNext()`, `E next()`, `default void remove()`, and (since Java 8) `default void forEachRemaining(Consumer<? super E> action)`. `Iterator` takes the place of `Enumeration` and differs from it in two ways: it allows the caller to remove elements from the underlying collection during iteration with well-defined semantics, and its method names are improved.

**Beginner-Friendly Explanation:** An `Iterator` is like a bookmark that moves through a collection. You ask “is there another element?” with `hasNext()`, and “give me the next element” with `next()`. You can also remove the last element you retrieved using `remove()`.

#### 3.1 `hasNext()` and `next()`

**Definitions**

- **Core Definition:** Methods for checking whether more elements exist and retrieving the next element.
- **Technical Definition:** `boolean hasNext()` returns `true` if the iteration has more elements. `E next()` returns the next element in the iteration and advances the cursor. `next()` throws `NoSuchElementException` if the iteration has no more elements.
- **Beginner-Friendly Explanation:** `hasNext()` is like asking “is there another item?” and `next()` is like saying “give me the next item.”

**Purposes**

- To check whether the iteration has more elements.
- To retrieve the next element and advance the cursor.
- To provide a safe way to traverse a collection without relying on indices.

**Syntax Structures and Rules**

Complete general syntax:

```java
Iterator<E> it = collection.iterator();
while (it.hasNext()) {
    E element = it.next();
    // process element
}
```

Component breakdown:

- `collection.iterator()`: obtains an iterator.
- `it.hasNext()`: checks for more elements.
- `it.next()`: retrieves the next element.

Syntax rules: `hasNext()` must be called before `next()` to avoid `NoSuchElementException`. `next()` advances the cursor.

Constraints and limitations: `next()` throws `NoSuchElementException` if no more elements exist. The iterator is only valid for a single traversal.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic Iterator usage**

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");
        list.add("C");

        Iterator<String> it = list.iterator();
        while (it.hasNext()) {
            String element = it.next();
            System.out.println(element);
        }
    }
}
```

Expected Output:

```
A
B
C
```

**Why the code produces this result:** The iterator traverses the list from the first element to the last, calling `hasNext()` and `next()` in a loop.

**Example 2: `NoSuchElementException` without `hasNext()`**

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorNoSuchElementExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Only");
        Iterator<String> it = list.iterator();
        System.out.println(it.next()); // "Only"
        try {
            it.next(); // No more elements
        } catch (java.util.NoSuchElementException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Only
Caught: NoSuchElementException
```

**Real-World Cases with Explanation**

`Iterator` is used whenever a collection must be traversed with the option to remove elements safely, such as filtering a list in place or processing a queue of tasks.

**References Links**

- Iterator (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Iterator.html
- Iterator (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html

#### 3.2 `remove()`

**Definitions**

- **Core Definition:** Removes the last element returned by the iterator from the underlying collection.
- **Technical Definition:** `default void remove()` removes from the underlying collection the last element returned by this iterator (optional operation). This method can be called only once per call to `next()`. The behavior of an iterator is unspecified if the underlying collection is modified while the iteration is in progress in any way other than by calling this method. The default implementation throws `UnsupportedOperationException`.
- **Beginner-Friendly Explanation:** `remove()` lets you delete the element you just retrieved with `next()`. You can only call it once per `next()` call.

**Purposes**

- To remove elements from the underlying collection during iteration.
- To provide a safe alternative to modifying the collection directly, which would cause `ConcurrentModificationException`.
- To support in-place filtering of collections.

**Syntax Structures and Rules**

Complete general syntax:

```java
it.remove();
```

Component breakdown:

- `it`: the iterator.

Syntax rules: `remove()` can only be called after `next()`. It removes the element last returned by `next()`. It can be called only once per `next()` call.

Constraints and limitations: `remove()` may throw `UnsupportedOperationException` if the iterator does not support removal. It throws `IllegalStateException` if `next()` has not been called or if `remove()` has already been called after the last `next()`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Removing elements during iteration**

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorRemoveExample {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>();
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
        numbers.add(4);

        Iterator<Integer> it = numbers.iterator();
        while (it.hasNext()) {
            int n = it.next();
            if (n % 2 == 0) {
                it.remove(); // Remove even numbers
            }
        }
        System.out.println(numbers);
    }
}
```

Expected Output:

```
[1, 3]
```

**Why the code produces this result:** The iterator traverses the list and removes even numbers using `it.remove()`. This is safe because the iterator is aware of the removal.

**Example 2: `IllegalStateException` when calling `remove()` twice**

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorRemoveTwiceExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A");
        Iterator<String> it = list.iterator();
        it.next();
        it.remove();
        try {
            it.remove(); // Second remove without next()
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

**Real-World Cases with Explanation**

`Iterator.remove()` is used in data-cleaning pipelines where invalid or unwanted elements must be removed from a collection during traversal.

**References Links**

- Iterator (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Iterator.html
- Iterator (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html

#### 3.3 `forEachRemaining()`

**Definitions**

- **Core Definition:** Performs a given action for each remaining element until all elements have been processed.
- **Technical Definition:** `default void forEachRemaining(Consumer<? super E> action)` performs the given action for each remaining element until all elements have been processed or the action throws an exception. Actions are performed in the order of iteration, if that order is specified.
- **Beginner-Friendly Explanation:** `forEachRemaining` lets you process all remaining elements with a single method call, using a lambda expression.

**Purposes**

- To process all remaining elements with a single method call.
- To combine iteration and action application in a concise form.
- To support internal iteration with lambda expressions.

**Syntax Structures and Rules**

Complete general syntax:

```java
it.forEachRemaining(element -> { /* action */ });
```

Component breakdown:

- `action`: a `Consumer` that is applied to each remaining element.

Syntax rules: The action is applied to each remaining element in iteration order. If the action throws an exception, iteration stops and the exception is propagated.

Constraints and limitations: The action must not modify the collection structurally.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `forEachRemaining` with lambda**

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class ForEachRemainingExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("X");
        list.add("Y");
        list.add("Z");

        Iterator<String> it = list.iterator();
        it.forEachRemaining(s -> System.out.println(s));
    }
}
```

Expected Output:

```
X
Y
Z
```

**Real-World Cases with Explanation**

`forEachRemaining` is used when a partial traversal has already been performed and the remaining elements need to be processed in bulk.

**References Links**

- Iterator (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Iterator.html
- Iterator (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html

### 4. ListIterator Interface

#### Core Definitions

**Core Definition:** A specialized iterator for lists that allows bidirectional traversal, element modification, and position retrieval.

**Technical Definition:** `public interface ListIterator<E> extends Iterator<E>` is an iterator for lists that allows the programmer to traverse the list in either direction, modify the list during iteration, and obtain the iterator's current position in the list. A `ListIterator` has no current element; its cursor position always lies between the element that would be returned by `previous()` and the element that would be returned by `next()`.

**Beginner-Friendly Explanation:** `ListIterator` is like a super-powered iterator for lists. You can go forward and backward, add elements, replace elements, and find out your current position.

#### 4.1 Bidirectional Traversal

**Definitions**

- **Core Definition:** The ability to traverse a list in both forward and backward directions.
- **Technical Definition:** `boolean hasPrevious()` returns `true` if this list iterator has more elements when traversing the list in the reverse direction. `E previous()` returns the previous element in the list and moves the cursor position backwards.
- **Beginner-Friendly Explanation:** You can move forward with `next()` and backward with `previous()`. `hasPrevious()` tells you if there is an element behind the cursor.

**Purposes**

- To traverse a list in both directions.
- To support algorithms that require backward traversal, such as reverse iteration.
- To examine elements before and after the current position.

**Syntax Structures and Rules**

Complete general syntax:

```java
ListIterator<E> it = list.listIterator();
while (it.hasNext()) { E e = it.next(); }
while (it.hasPrevious()) { E e = it.previous(); }
```

Component breakdown:

- `list.listIterator()`: obtains a `ListIterator`.
- `hasNext()` / `next()`: forward traversal.
- `hasPrevious()` / `previous()`: backward traversal.

Syntax rules: The cursor position is between `previous()` and `next()`. Calling `next()` advances the cursor; calling `previous()` moves it back.

Constraints and limitations: `previous()` throws `NoSuchElementException` if there is no previous element.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Forward and backward traversal**

```java
import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;

public class ListIteratorBidirectionalExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");
        list.add("C");

        ListIterator<String> it = list.listIterator();
        System.out.println("Forward:");
        while (it.hasNext()) {
            System.out.println(it.next());
        }
        System.out.println("Backward:");
        while (it.hasPrevious()) {
            System.out.println(it.previous());
        }
    }
}
```

Expected Output:

```
Forward:
A
B
C
Backward:
C
B
A
```

**Why the code produces this result:** After the forward traversal, the cursor is at the end of the list. The backward traversal then moves the cursor back through the list.

**Real-World Cases with Explanation**

`ListIterator` bidirectional traversal is used in text editors (moving the cursor forward and backward), undo–redo systems, and algorithms that require scanning a list in both directions.

**References Links**

- ListIterator (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ListIterator.html
- ListIterator (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/ListIterator.html

#### 4.2 Element Modification (`add`, `set`, `remove`)

**Definitions**

- **Core Definition:** Methods that allow the list to be modified during iteration.
- **Technical Definition:** `void add(E e)` inserts the specified element into the list (optional operation). `void set(E e)` replaces the last element returned by `next()` or `previous()` with the specified element (optional operation). `void remove()` removes from the list the last element returned by `next()` or `previous()` (optional operation).
- **Beginner-Friendly Explanation:** `add` inserts an element at the cursor position. `set` replaces the last element you retrieved. `remove` deletes the last element you retrieved.

**Purposes**

- To insert elements during iteration.
- To replace elements during iteration.
- To remove elements during iteration.

**Syntax Structures and Rules**

Complete general syntax:

```java
it.add(element);   // Insert at cursor
it.set(element);   // Replace last returned
it.remove();       // Remove last returned
```

Component breakdown:

- `add`: inserts before the element that would be returned by `next()`.
- `set`: replaces the last element returned by `next()` or `previous()`.
- `remove`: removes the last element returned by `next()` or `previous()`.

Syntax rules: `set` and `remove` can only be called after `next()` or `previous()`. `add` can be called at any time.

Constraints and limitations: These methods may throw `UnsupportedOperationException` if the list does not support modification. `set` and `remove` throw `IllegalStateException` if neither `next()` nor `previous()` has been called.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Replacing elements during iteration**

```java
import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;

public class ListIteratorSetExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("apple");
        list.add("banana");
        list.add("cherry");

        ListIterator<String> it = list.listIterator();
        while (it.hasNext()) {
            String s = it.next();
            it.set(s.toUpperCase());
        }
        System.out.println(list);
    }
}
```

Expected Output:

```
[APPLE, BANANA, CHERRY]
```

**Example 2: Adding elements during iteration**

```java
import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;

public class ListIteratorAddExample {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(3);

        ListIterator<Integer> it = list.listIterator();
        while (it.hasNext()) {
            int n = it.next();
            if (n == 1) {
                it.add(2); // Insert 2 after 1
            }
        }
        System.out.println(list);
    }
}
```

Expected Output:

```
[1, 2, 3]
```

**Real-World Cases with Explanation**

`ListIterator` modification is used in in-place data transformations, such as normalizing a list of strings to uppercase or inserting derived elements during traversal.

**References Links**

- ListIterator (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ListIterator.html
- ListIterator (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/ListIterator.html

### 5. Iterable Interface

#### Core Definitions

**Core Definition:** The root interface that enables an object to be the target of the enhanced `for` loop.

**Technical Definition:** `public interface Iterable<T>` is a functional interface that declares `Iterator<T> iterator()`, `default void forEach(Consumer<? super T> action)`, and `default Spliterator<T> spliterator()`. Implementing this interface allows an object to be the target of the enhanced `for` statement.

**Beginner-Friendly Explanation:** If a class implements `Iterable`, it means “you can loop over me with a for-each loop.” It is the contract that makes an object traversable.

#### 5.1 `iterator()`

**Definitions**

- **Core Definition:** Returns an iterator over elements of type `T`.
- **Technical Definition:** `Iterator<T> iterator()` returns an iterator over elements of type `T`. This is the only abstract method of `Iterable`.
- **Beginner-Friendly Explanation:** `iterator()` gives you an `Iterator` that you can use to traverse the object.

**Purposes**

- To provide an `Iterator` for the enhanced `for` loop.
- To allow custom classes to be traversed with the for-each construct.
- To serve as the foundation of the Java Collections Framework.

**Syntax Structures and Rules**

Complete general syntax:

```java
public class MyCollection<T> implements Iterable<T> {
    @Override
    public Iterator<T> iterator() {
        return new MyIterator();
    }
}
```

Component breakdown:

- `implements Iterable<T>`: declares that the class is iterable.
- `iterator()`: returns an `Iterator`.

Syntax rules: Any class implementing `Iterable` must provide an implementation of `iterator()`.

Constraints and limitations: The returned iterator must not return `null`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Custom `Iterable` class**

```java
import java.util.Iterator;
import java.util.NoSuchElementException;

public class MyIterable implements Iterable<Integer> {
    private final int[] data = {10, 20, 30};

    @Override
    public Iterator<Integer> iterator() {
        return new Iterator<Integer>() {
            private int index = 0;

            @Override
            public boolean hasNext() {
                return index < data.length;
            }

            @Override
            public Integer next() {
                if (!hasNext()) throw new NoSuchElementException();
                return data[index++];
            }
        };
    }

    public static void main(String[] args) {
        MyIterable iterable = new MyIterable();
        for (int value : iterable) {
            System.out.println(value);
        }
    }
}
```

Expected Output:

```
10
20
30
```

**Real-World Cases with Explanation**

Custom `Iterable` implementations are used in domain-specific collections, such as a tree that can be traversed in-order, or a database result set that can be iterated row by row.

**References Links**

- Iterable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Iterable.html
- Iterable (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html

### 6. forEach() Method (Java 8+)

#### Core Definitions

**Core Definition:** A default method on `Iterable` that performs a given action for each element using internal iteration.

**Technical Definition:** `default void forEach(Consumer<? super T> action)` performs the given action for each element of the `Iterable` until all elements have been processed or the action throws an exception. The default implementation behaves as if `for (T t : this) action.accept(t);`.

**Beginner-Friendly Explanation:** `forEach` is a concise way to say “do this for every element.” You pass a lambda expression or method reference, and Java handles the loop for you.

#### 6.1 Lambda Expressions and Method References

**Definitions**

- **Core Definition:** Functional parameters passed to `forEach` that specify the action to be performed on each element.
- **Technical Definition:** A `Consumer<? super T>` is a functional interface with `void accept(T t)`. Lambda expressions such as `x -> System.out.println(x)` or method references such as `System.out::println` can be used.
- **Beginner-Friendly Explanation:** You can write `list.forEach(System.out::println)` to print every element, or `list.forEach(s -> System.out.println(s.toUpperCase()))` to print each element in uppercase.

**Purposes**

- To perform an action on each element of an `Iterable` concisely.
- To support internal iteration with lambda expressions and method references.
- To improve code readability.

**Syntax Structures and Rules**

Complete general syntax:

```java
iterable.forEach(element -> { /* action */ });
```

Component breakdown:

- `element`: the loop variable.
- `action`: the lambda body.

Syntax rules: The action must be a `Consumer`. It is applied to each element in iteration order.

Constraints and limitations: The action must not modify the collection structurally. External iteration (e.g., `break`, `continue`, `return`) is not supported within `forEach`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `forEach` with lambda**

```java
import java.util.ArrayList;
import java.util.List;

public class ForEachMethodExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Java");
        list.add("Python");
        list.add("C++");

        list.forEach(lang -> System.out.println(lang));
    }
}
```

Expected Output:

```
Java
Python
C++
```

**Example 2: `forEach` with method reference**

```java
import java.util.ArrayList;
import java.util.List;

public class ForEachMethodRefExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Hello");
        list.add("World");

        list.forEach(System.out::println);
    }
}
```

Expected Output:

```
Hello
World
```

**Real-World Cases with Explanation**

`forEach` is used in logging, printing, and applying side-effect operations to each element in a collection. It is also used in streams as a terminal operation.

**References Links**

- Iterable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Iterable.html
- Iterable (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html

### 7. Stream API (Java 8+)

#### Core Definitions

**Core Definition:** A sequence of elements supporting sequential and parallel aggregate operations.

**Technical Definition:** `public interface Stream<T> extends BaseStream<T, Stream<T>>` is a sequence of elements supporting sequential and parallel aggregate operations. A stream pipeline consists of a source, zero or more intermediate operations, and a terminal operation. Streams are lazy: computation on the source data is only performed when the terminal operation is initiated.

**Beginner-Friendly Explanation:** A stream is like a pipeline: you put data in at one end, apply transformations in the middle, and produce a result at the other end. It is a modern, functional way to process collections.

#### 7.1 Stream Creation and Intermediate Operations

**Definitions**

- **Core Definition:** Creating a stream from a source and applying operations such as `filter`, `map`, and `sorted`.
- **Technical Definition:** `Collection.stream()` and `Collection.parallelStream()` create sequential and parallel streams, respectively. Intermediate operations are lazy and return a new stream.
- **Beginner-Friendly Explanation:** You start with `list.stream()`, then chain operations like `.filter(x -> x > 0)` and `.map(x -> x * 2)`.

**Purposes**

- To process collections declaratively.
- To support lazy evaluation and short-circuiting.
- To enable parallel processing with minimal code changes.

**Syntax Structures and Rules**

Complete general syntax:

```java
collection.stream()
    .filter(predicate)
    .map(function)
    .forEach(consumer);
```

Component breakdown:

- `stream()`: creates a stream.
- `filter`: retains elements matching a predicate.
- `map`: transforms each element.
- `forEach`: terminal operation.

Syntax rules: Intermediate operations are lazy. Terminal operations trigger computation. A stream can be used only once.

Constraints and limitations: Streams do not modify the source. Behavioral parameters must be non-interfering and stateless.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Filter and map**

```java
import java.util.Arrays;
import java.util.List;

public class StreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

        numbers.stream()
               .filter(n -> n % 2 == 0)
               .map(n -> n * n)
               .forEach(System.out::println);
    }
}
```

Expected Output:

```
4
16
36
```

**Why the code produces this result:** The stream filters even numbers (2, 4, 6), then maps each to its square (4, 16, 36), and prints them.

**Example 2: Parallel stream**

```java
import java.util.Arrays;
import java.util.List;

public class ParallelStreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        int sum = numbers.parallelStream()
                         .mapToInt(Integer::intValue)
                         .sum();
        System.out.println("Sum: " + sum);
    }
}
```

Expected Output:

```
Sum: 15
```

**Real-World Cases with Explanation**

Streams are used in data processing pipelines, such as filtering a list of transactions, computing aggregates, or transforming data for reporting.

**References Links**

- Stream (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html
- Stream (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html

### 8. Spliterator Interface

#### Core Definitions

**Core Definition:** An object for traversing and partitioning elements of a source, designed for efficient parallel traversal.

**Technical Definition:** `public interface Spliterator<T>` is an object for traversing and partitioning elements of a source. A `Spliterator` may traverse elements individually (`tryAdvance()`) or sequentially in bulk (`forEachRemaining()`). It may also partition off some of its elements (`trySplit()`) as another `Spliterator` for parallel processing.

**Beginner-Friendly Explanation:** A `Spliterator` is like an iterator that can split itself into two parts, so multiple threads can process different parts of the data simultaneously.


#### 8.1 `tryAdvance()` and `trySplit()`

**Definitions**

- **Core Definition:** Methods for single-element traversal and partitioning.
- **Technical Definition:** `boolean tryAdvance(Consumer<? super T> action)` performs the action on the next element if one exists, returning `true`; otherwise returns `false`. `Spliterator<T> trySplit()` partitions off some elements as another `Spliterator`.
- **Beginner-Friendly Explanation:** `tryAdvance` processes one element. `trySplit` splits the spliterator into two, so parallel processing is possible.

**Purposes**

- To traverse elements one by one.
- To partition elements for parallel processing.
- To provide efficient parallel traversal.

**Syntax Structures and Rules**

Complete general syntax:

```java
Spliterator<T> spliterator = collection.spliterator();
spliterator.tryAdvance(element -> { /* action */ });
Spliterator<T> secondHalf = spliterator.trySplit();
```

Component breakdown:

- `tryAdvance`: processes the next element.
- `trySplit`: splits the spliterator.

Syntax rules: `trySplit` returns `null` if the spliterator cannot be split.

Constraints and limitations: Spliterators are designed for single-use.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic `tryAdvance`**

```java
import java.util.Arrays;
import java.util.List;
import java.util.Spliterator;

public class SpliteratorExample {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("A", "B", "C");
        Spliterator<String> spliterator = list.spliterator();

        spliterator.tryAdvance(s -> System.out.println("First: " + s));
        spliterator.forEachRemaining(s -> System.out.println("Remaining: " + s));
    }
}
```

Expected Output:

```
First: A
Remaining: B
Remaining: C
```

**Real-World Cases with Explanation**

`Spliterator` is used internally by the Stream API for parallel stream processing. It is also used in custom parallel algorithms that need to partition data across threads.

**References Links**

- Spliterator (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Spliterator.html
- Spliterator (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Spliterator.html

### 9. Safe Modification and Fail-Fast vs. Fail-Safe Iterators

#### Core Definitions

**Core Definition:** Safe modification refers to techniques for modifying a collection during iteration without causing `ConcurrentModificationException`.

**Technical Definition:** Fail-fast iterators detect structural modification of the underlying collection (such as adding or removing elements, which changes the modification count) and throw `ConcurrentModificationException` on a best-effort basis. Fail-safe iterators either operate on a snapshot of the collection or use weakly consistent semantics that tolerate concurrent modification. The `Iterator.remove()` method provides a safe way to remove elements during iteration for fail-fast iterators.

**Beginner-Friendly Explanation:** If you try to modify a collection while looping over it with a normal iterator, Java may throw a `ConcurrentModificationException`. To avoid this, you can use `Iterator.remove()`, use a concurrent collection like `CopyOnWriteArrayList`, or use a fail-safe iterator.

#### 9.1 Fail-Fast Iterators

**Definitions**

- **Core Definition:** Iterators that detect concurrent structural modification and throw `ConcurrentModificationException`.
- **Technical Definition:** Fail-fast iterators are implemented by associating a modification count with the collection. If the modification count changes during iteration (other than via the iterator's own `remove()` method), the iterator throws `ConcurrentModificationException`. Fail-fast behavior is not guaranteed in the presence of unsynchronized concurrent modification.
- **Beginner-Friendly Explanation:** Fail-fast iterators are like a referee that blows the whistle immediately if someone changes the game while it is being played.

**Purposes**

- To detect concurrent modification early and fail cleanly.
- To prevent unpredictable behavior from unsynchronized modification.
- To provide well-defined semantics for `Iterator.remove()`.

**Syntax Structures and Rules**

Complete general syntax:

```java
Iterator<E> it = collection.iterator();
while (it.hasNext()) {
    E e = it.next();
    if (condition) {
        it.remove(); // Safe removal
    }
    // collection.add(...) // Would throw ConcurrentModificationException
}
```

Component breakdown:

- `it.remove()`: safe removal during iteration.
- Direct collection modification: unsafe during fail-fast iteration.

Syntax rules: Only `Iterator.remove()` is permitted for structural modification during fail-fast iteration.

Constraints and limitations: Fail-fast behavior is best-effort and cannot be guaranteed in the presence of unsynchronized concurrency.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `ConcurrentModificationException`**

```java
import java.util.ArrayList;
import java.util.List;

public class FailFastExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");
        list.add("C");

        for (String s : list) {
            if (s.equals("B")) {
                list.remove(s); // Modifying during for-each
            }
        }
    }
}
```

Expected Output:

```
Exception in thread "main" java.util.ConcurrentModificationException
```

**Why the code produces this result:** The enhanced `for` loop uses a fail-fast iterator. Directly modifying the list during iteration changes the modification count, causing the iterator to throw `ConcurrentModificationException` on the next `next()` call.

**Example 2: Safe removal with `Iterator.remove()`**

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class FailFastSafeExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");
        list.add("C");

        Iterator<String> it = list.iterator();
        while (it.hasNext()) {
            String s = it.next();
            if (s.equals("B")) {
                it.remove(); // Safe
            }
        }
        System.out.println(list);
    }
}
```

Expected Output:

```
[A, C]
```

**Real-World Cases with Explanation**

Fail-fast iterators are used in single-threaded contexts where accidental concurrent modification should be detected. They are the default for most `java.util` collections.

**References Links**

- ConcurrentModificationException (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ConcurrentModificationException.html
- Iterator (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Iterator.html

#### 9.2 Fail-Safe Iterators

**Definitions**

- **Core Definition:** Iterators that tolerate concurrent modification by operating on a snapshot or using weakly consistent semantics.
- **Technical Definition:** Fail-safe iterators are used by concurrent collections such as `CopyOnWriteArrayList` and `ConcurrentHashMap`. `CopyOnWriteArrayList` uses a snapshot-style iterator that operates on a reference to the array at the time the iterator was created. `ConcurrentHashMap` uses weakly consistent iterators that do not throw `ConcurrentModificationException`.
- **Beginner-Friendly Explanation:** Fail-safe iterators are like a photograph of the collection taken when iteration starts. Changes made after the photograph do not affect the iteration.

**Purposes**

- To allow concurrent modification without throwing exceptions.
- To support thread-safe iteration in concurrent collections.
- To provide weakly consistent semantics for concurrent traversal.

**Syntax Structures and Rules**

Complete general syntax:

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("A");
for (String s : list) {
    list.add("B"); // Safe; iterator sees snapshot
}
```

Component breakdown:

- `CopyOnWriteArrayList`: creates a copy-on-write list.
- Iteration: operates on a snapshot.

Syntax rules: Fail-safe iterators may not reflect the latest state of the collection. `CopyOnWriteArrayList` iterators do not support `remove()`.

Constraints and limitations: Snapshot iterators may be stale. Copy-on-write is expensive for write-heavy workloads.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `CopyOnWriteArrayList` safe iteration**

```java
import java.util.concurrent.CopyOnWriteArrayList;

public class FailSafeExample {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
        list.add("A");
        list.add("B");

        for (String s : list) {
            System.out.println(s);
            list.add("C"); // Safe; iterator uses snapshot
        }
        System.out.println("Final list: " + list);
    }
}
```

Expected Output:

```
A
B
Final list: [A, B, C, C]
```

**Why the code produces this result:** The iterator uses a snapshot of the list at the time of creation, so it only sees “A” and “B”. The additions of “C” are made to the underlying list and are visible in the final list.

**Real-World Cases with Explanation**

`CopyOnWriteArrayList` is used in scenarios where reads vastly outnumber writes, such as maintaining a list of event listeners that is occasionally modified but frequently iterated.

**References Links**

- CopyOnWriteArrayList (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CopyOnWriteArrayList.html
- ConcurrentHashMap (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html

### 10. Enumeration Interface (Legacy)

#### Core Definitions

**Core Definition:** A legacy iteration interface that generates a series of elements, one at a time.

**Technical Definition:** `public interface Enumeration<E>` is a legacy iteration interface. New code should use `Iterator` instead. `Enumeration` declares `boolean hasMoreElements()` and `E nextElement()`. It is used by legacy classes such as `Vector`, `Hashtable`, and `Properties`. Since Java 9, `Enumeration` provides a default `asIterator()` method to convert an `Enumeration` to an `Iterator`.

**Beginner-Friendly Explanation:** `Enumeration` is the old way of iterating. It is still used by some legacy classes, but new code should use `Iterator` instead.

#### 10.1 `hasMoreElements()` and `nextElement()`

**Definitions**

- **Core Definition:** Methods for checking whether more elements exist and retrieving the next element.
- **Technical Definition:** `boolean hasMoreElements()` returns `true` if there are more elements. `E nextElement()` returns the next element. It throws `NoSuchElementException` if there are no more elements.
- **Beginner-Friendly Explanation:** `hasMoreElements` is like `hasNext` and `nextElement` is like `next`, but with longer names.

**Purposes**

- To provide iteration for legacy classes.
- To support conversion to `Iterator` via `asIterator()`.
- To maintain backward compatibility.

**Syntax Structures and Rules**

Complete general syntax:

```java
Enumeration<E> e = vector.elements();
while (e.hasMoreElements()) {
    E element = e.nextElement();
}
```

Component breakdown:

- `hasMoreElements()`: checks for more elements.
- `nextElement()`: retrieves the next element.

Syntax rules: `nextElement()` throws `NoSuchElementException` if no more elements exist.

Constraints and limitations: Does not support element removal. New code should use `Iterator`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Enumeration with `Vector`**

```java
import java.util.Enumeration;
import java.util.Vector;

public class EnumerationExample {
    public static void main(String[] args) {
        Vector<String> vector = new Vector<>();
        vector.add("A");
        vector.add("B");
        vector.add("C");

        Enumeration<String> e = vector.elements();
        while (e.hasMoreElements()) {
            System.out.println(e.nextElement());
        }
    }
}
```

Expected Output:

```
A
B
C
```

**Example 2: Converting `Enumeration` to `Iterator`**

```java
import java.util.Enumeration;
import java.util.Iterator;
import java.util.Vector;

public class EnumerationToIteratorExample {
    public static void main(String[] args) {
        Vector<String> vector = new Vector<>();
        vector.add("X");
        vector.add("Y");

        Enumeration<String> e = vector.elements();
        Iterator<String> it = e.asIterator();
        while (it.hasNext()) {
            System.out.println(it.next());
        }
    }
}
```

Expected Output:

```
X
Y
```

**Real-World Cases with Explanation**

`Enumeration` is encountered when working with legacy codebases that use `Vector`, `Hashtable`, or `Properties`. The `asIterator()` method allows legacy enumerations to be used with modern iterator-based APIs.

**References Links**

- Enumeration (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Enumeration.html
- Enumeration (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Enumeration.html

## Summary Table of Iteration Mechanisms

| Mechanism | External/Internal | Bidirectional | Modification | Parallel | Fail-Fast |
|---|---|---|---|---|---|
| Traditional for | External | No | Index-based | No | N/A |
| Enhanced for | External | No | No | No | Yes (via iterator) |
| Iterator | External | No | `remove()` | No | Yes |
| ListIterator | External | Yes | `add`, `set`, `remove` | No | Yes |
| `forEach()` | Internal | No | No | No | N/A |
| Stream API | Internal | No | No | Yes | N/A |
| Spliterator | External | No | No | Yes | N/A |
| Enumeration | External | No | No | No | No |

## Deprecated, Unsafe, and Version-Specific Notes

- The `Enumeration` interface is legacy and should not be used in new code. Prefer `Iterator`.
- `Iterator.remove()` is the only safe way to remove elements during fail-fast iteration. Directly modifying the collection during iteration causes `ConcurrentModificationException`.
- The enhanced `for` loop does not expose the iterator, so `remove()` cannot be called within a for-each loop. A `ConcurrentModificationException` will be thrown if the collection is modified during a for-each loop.
- `ListIterator` is only available for `List` implementations, not for `Set` or `Queue`.
- Streams are single-use. A stream that has been operated on cannot be reused.
- `Spliterator` is designed for single bulk computation and should not be reused.
- `forEach()` and the Stream API were introduced in Java 8. `Iterator.forEachRemaining()` was introduced in Java 8. `Enumeration.asIterator()` was introduced in Java 9.
- The enhanced `for` loop was introduced in Java 5.

## References

- Iterator (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Iterator.html
- Iterator (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html
- ListIterator (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ListIterator.html
- ListIterator (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/ListIterator.html
- Iterable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Iterable.html
- Iterable (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html
- Stream (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html
- Stream (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html
- Spliterator (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Spliterator.html
- Spliterator (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Spliterator.html
- Enumeration (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Enumeration.html
- Enumeration (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Enumeration.html
- ConcurrentModificationException (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ConcurrentModificationException.html
- CopyOnWriteArrayList (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CopyOnWriteArrayList.html
- The for Statement (Java Language Specification, Java SE 21) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.14
- The Enhanced for Statement (Java Language Specification, Java SE 21) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.14.2