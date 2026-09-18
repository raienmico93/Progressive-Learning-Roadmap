# Java List — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The Java `List` interface is an ordered collection (also known as a sequence) that provides precise control over where each element is inserted, allows access by integer index, and permits duplicate elements.

**Technical Definition:** `java.util.List<E>` extends `Collection<E>` and `Iterable<E>`. It places additional stipulations on the contracts of `iterator`, `add`, `remove`, `equals`, and `hashCode` beyond those specified in the `Collection` interface. Lists are zero-based, meaning the first element is at index 0.

**Beginner-Friendly Explanation:** Think of a Java `List` as a dynamic, resizable array that keeps its items in the exact order you put them. Unlike a mathematical set, you can store the same value multiple times (e.g., two "Apple" entries), and you can retrieve or modify any item by its position number (index).

### Key Characteristics

- **Ordered:** Elements maintain insertion order; iteration returns elements in that order.
- **Allows Duplicates:** Unlike `Set`, a `List` can contain multiple equal elements.
- **Positional Access:** Elements can be accessed, inserted, and removed by integer index.
- **Zero-Based Indexing:** The first element is at index 0.
- **Allows Nulls (typically):** Most implementations permit `null` elements, though some (e.g., `List.of()`) do not.
- **ListIterator Support:** A bidirectional iterator allowing element insertion and replacement during traversal.
- **Part of Java Collections Framework:** Integrates with the broader `java.util` collection hierarchy.

### Prerequisites

To effectively use `List` in Java, you should understand:

- Basic Java syntax, classes, and objects.
- The `Collection` interface and generics (`<E>`).
- The difference between an interface and a concrete class.
- Basic iteration constructs (`for`, `while`).
- Familiarity with `java.util` package imports.

### Related Programming Areas

| Area | Explanation |
|------|-------------|
| **Collections Framework** | `List` is one of the core interfaces in `java.util`, alongside `Set`, `Map`, and `Queue`. |
| **Generics** | Type parameters (`List<String>`) provide compile-time type safety. |
| **Concurrency** | `CopyOnWriteArrayList` provides thread-safe list operations. |
| **Functional Programming** | `forEach`, `removeIf`, and `Stream` integration enable functional-style operations. |
| **Data Structures & Algorithms** | Lists underpin searching, sorting, and indexing algorithms. |

---

## Core Concepts / Key Features

---

## 1. List Interface Core Characteristics

### Core Definitions

- **Core Definition:** The `List` interface defines an ordered collection with positional access, duplicate tolerance, and indexed operations.
- **Technical Definition:** `public interface List<E> extends Collection<E>` — adds positional access methods (`get`, `set`, `add(int, E)`, `remove(int)`), search methods (`indexOf`, `lastIndexOf`), range-view (`subList`), and a specialized `ListIterator`.
- **Beginner-Friendly Explanation:** The `List` interface is the "contract" that all list types in Java must follow. It says: "You must let me add items in a specific order, get items by position, search for items, and allow duplicates."

### Sub-feature 1.1: Ordered Collection

#### Definitions

- **Core Definition:** A `List` maintains the insertion order of its elements.
- **Technical Definition:** Iteration over a `List` returns elements in the same order they were inserted, unless explicitly modified by positional operations (e.g., `add(int, E)`, `set`).
- **Beginner-Friendly Explanation:** If you add "Alice", then "Bob", then "Charlie", the list will always give them back in that exact order unless you deliberately change their positions.

#### Purposes

- To preserve the sequence of user-supplied data (e.g., log entries, transaction history).
- To enable deterministic iteration where order matters.
- To support algorithms that depend on positional relationships (e.g., sorting, searching).

#### Syntax Structures and Rules

**General Syntax:**
```
List<Type> listName = new ArrayList<>();
listName.add(element1);
listName.add(element2);
```

**Component Breakdown:**
- `List<Type>` — the interface and its generic type parameter.
- `new ArrayList<>()` — a concrete implementation.
- `.add(element)` — appends to the end.

**Syntax Rules:**
- The type parameter must be a reference type (not a primitive).
- The order of `add()` calls determines the iteration order.

**Constraints and Limitations:**
- Some implementations (e.g., `PriorityQueue`) do not preserve insertion order; `List` implementations do.
- `List.of()` creates an immutable list where order is fixed at creation.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class OrderedListDemo {
    public static void main(String[] args) {
        // Create an ordered list of strings
        List<String> fruits = new ArrayList<>();
        
        // Add elements in a specific order
        fruits.add("Apple");   // index 0
        fruits.add("Banana");  // index 1
        fruits.add("Cherry");  // index 2
        
        // Iterate and print to demonstrate order preservation
        for (String fruit : fruits) {
            System.out.println(fruit);
        }
    }
}
```

**Expected Output:**
```
Apple
Banana
Cherry
```

**Why This Output:** The `ArrayList` stores elements in the exact order they are added. The enhanced for-loop iterates from index 0 to index 2, printing each element in insertion order.

#### Real-World Case

**Scenario:** A shopping cart application. Items are added to the cart in the order the user selects them. When displaying the cart, the order is preserved so the user sees the sequence of their selections. If the user adds "Milk", then "Bread", then "Eggs", the cart displays them in that order.

---

### Sub-feature 1.2: Allows Duplicates

#### Definitions

- **Core Definition:** A `List` can contain multiple elements that are equal according to `equals()`.
- **Technical Definition:** Unlike `Set`, which uses `equals()` to enforce uniqueness, `List` places no restriction on duplicate elements. Elements `e1` and `e2` may satisfy `e1.equals(e2)`.
- **Beginner-Friendly Explanation:** You can put the same value into a list multiple times. If you add "Apple" twice, the list will contain two "Apple" entries.

#### Purposes

- To represent data where repetition is meaningful (e.g., survey responses, multi-set semantics).
- To allow multiple occurrences of the same value at different positions.
- To support algorithms that count or group repeated values.

#### Syntax Structures and Rules

**General Syntax:**
```
List<Type> list = new ArrayList<>();
list.add(element);   // can be called multiple times with the same value
```

**Component Breakdown:**
- `.add(element)` — appends regardless of whether an equal element already exists.

**Syntax Rules:**
- The `add()` method always returns `true` (for `List`, unlike `Set` where it may return `false`).
- Duplicate detection is the caller's responsibility if needed.

**Constraints and Limitations:**
- `List.of()` and `Collections.unmodifiableList()` still allow duplicates, but the list itself is immutable.
- Searching methods like `indexOf()` return the first occurrence.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class DuplicateDemo {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        
        // Add duplicates intentionally
        names.add("Alice");
        names.add("Bob");
        names.add("Alice");   // duplicate
        names.add("Alice");   // another duplicate
        
        System.out.println("List size: " + names.size());
        System.out.println("First occurrence of 'Alice' at index: " + names.indexOf("Alice"));
        System.out.println("Last occurrence of 'Alice' at index: " + names.lastIndexOf("Alice"));
        
        // Print all elements
        for (String name : names) {
            System.out.println(name);
        }
    }
}
```

**Expected Output:**
```
List size: 4
First occurrence of 'Alice' at index: 0
Last occurrence of 'Alice' at index: 3
Alice
Bob
Alice
Alice
```

**Why This Output:** The list stores all four elements including duplicates. `indexOf` finds the first "Alice" at index 0, while `lastIndexOf` finds the last at index 3.

#### Real-World Case

**Scenario:** An attendance tracking system where a student may check in multiple times per day (e.g., for different sessions). Each check-in is recorded as a separate entry in a `List<AttendanceRecord>`, allowing duplicates for the same student.

---

### Sub-feature 1.3: Positional Access

#### Definitions

- **Core Definition:** Elements in a `List` can be accessed, inserted, and removed by their integer index.
- **Technical Definition:** The `List` interface provides `get(int index)`, `set(int index, E element)`, `add(int index, E element)`, and `remove(int index)` for positional manipulation.
- **Beginner-Friendly Explanation:** Every item in a list has a numbered position, starting from 0. You can say "give me the item at position 3" or "put this new item at position 1".

#### Purposes

- To enable random access to elements by position.
- To support insertion and removal at arbitrary positions.
- To facilitate algorithms that operate on indexed data.

#### Syntax Structures and Rules

**General Syntax:**
```
E element = list.get(index);              // retrieve
list.set(index, newElement);              // replace
list.add(index, element);                 // insert
E removed = list.remove(index);           // remove
```

**Component Breakdown:**
- `index` — an `int` from `0` to `size()-1`.
- All operations throw `IndexOutOfBoundsException` if index is invalid.

**Syntax Rules:**
- Indices are zero-based.
- `add(index, element)` shifts subsequent elements to the right.
- `remove(index)` shifts subsequent elements to the left.
- For `LinkedList`, positional access is O(n); for `ArrayList`, it is O(1).

**Constraints and Limitations:**
- Performance varies by implementation: `ArrayList` offers constant-time positional access; `LinkedList` requires linear traversal.
- Some immutable lists (`List.of()`) throw `UnsupportedOperationException` on positional modification.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class PositionalAccessDemo {
    public static void main(String[] args) {
        List<String> colors = new ArrayList<>();
        colors.add("Red");      // index 0
        colors.add("Green");    // index 1
        colors.add("Blue");     // index 2
        
        // Retrieve by index
        String secondColor = colors.get(1);
        System.out.println("Element at index 1: " + secondColor);
        
        // Replace by index
        colors.set(1, "Yellow");
        System.out.println("After set(1, 'Yellow'): " + colors);
        
        // Insert at index
        colors.add(1, "Purple");
        System.out.println("After add(1, 'Purple'): " + colors);
        
        // Remove by index
        String removed = colors.remove(2);
        System.out.println("Removed element: " + removed);
        System.out.println("Final list: " + colors);
    }
}
```

**Expected Output:**
```
Element at index 1: Green
After set(1, 'Yellow'): [Red, Yellow, Blue]
After add(1, 'Purple'): [Red, Purple, Yellow, Blue]
Removed element: Yellow
Final list: [Red, Purple, Blue]
```

**Why This Output:** `get(1)` retrieves "Green". `set(1, "Yellow")` replaces "Green" with "Yellow". `add(1, "Purple")` inserts at index 1, shifting "Yellow" and "Blue" right. `remove(2)` removes "Yellow" (now at index 2), shifting "Blue" left.

#### Real-World Case

**Scenario:** A playlist editor where a user can drag songs to reorder them. Each song has an index position. The user can insert a new song at a specific position, remove a song by its position, or replace a song at a given index.

---

### Sub-feature 1.4: Relationship to the Collection Interface

#### Definitions

- **Core Definition:** `List` extends the `Collection` interface, inheriting basic collection operations while adding list-specific behaviors.
- **Technical Definition:** `public interface List<E> extends Collection<E>` — `List` inherits `size()`, `isEmpty()`, `contains()`, `iterator()`, `toArray()`, `add()`, `remove()`, `containsAll()`, `addAll()`, `removeAll()`, `retainAll()`, and `clear()`.
- **Beginner-Friendly Explanation:** `Collection` is the parent "contract" for all container types. `List` is a specialized version that adds ordering and indexing on top of what `Collection` already provides.

#### Purposes

- To provide a common API for all collection types.
- To allow polymorphic treatment of lists as collections.
- To inherit bulk operations and basic query methods.

#### Syntax Structures and Rules

**General Syntax:**
```
Collection<String> collection = new ArrayList<>();
List<String> list = (List<String>) collection;   // downcast (unsafe)
```

**Syntax Rules:**
- A `List` can be used wherever a `Collection` is expected.
- Not all `Collection` methods are meaningful for all `List` implementations (e.g., `remove(int)` is list-specific).

**Constraints and Limitations:**
- `Collection` does not have `get(int)` or `set(int, E)`.
- Some `Collection` methods (e.g., `remove(Object)`) behave differently in `List` because of overloading with `remove(int)`.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

public class CollectionRelationshipDemo {
    public static void main(String[] args) {
        // Treat a List as a Collection
        Collection<String> collection = new ArrayList<>();
        collection.add("A");
        collection.add("B");
        collection.add("C");
        
        System.out.println("Collection size: " + collection.size());
        System.out.println("Contains 'B': " + collection.contains("B"));
        
        // Downcast to List to access positional methods
        if (collection instanceof List) {
            List<String> list = (List<String>) collection;
            System.out.println("Element at index 0: " + list.get(0));
        }
    }
}
```

**Expected Output:**
```
Collection size: 3
Contains 'B': true
Element at index 0: A
```

**Why This Output:** The `ArrayList` is created as a `Collection`, so only `Collection` methods are accessible. After downcasting to `List`, positional methods like `get(0)` become available.

#### Real-World Case

**Scenario:** A method that accepts any `Collection<String>` for processing (e.g., printing all elements) can also receive a `List<String>` because `List` is a subtype of `Collection`. This enables code reuse across different collection types.

#### References

- List (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/List.html
- The List Interface (Oracle Tutorial) — https://docs.oracle.com/javase/tutorial/collections/interfaces/list.html
- Java List Interface (Baeldung) — https://www.baeldung.com/java-list-interface

---

## 2. Core Implementations

### Core Definitions

- **Core Definition:** Java provides several concrete classes that implement the `List` interface, each with different performance characteristics.
- **Technical Definition:** The main `List` implementations are `ArrayList` (resizable array), `LinkedList` (doubly-linked list), `Vector` (legacy synchronized array), `Stack` (legacy LIFO), and `CopyOnWriteArrayList` (thread-safe copy-on-write).
- **Beginner-Friendly Explanation:** The `List` interface is just a blueprint; you need to choose a concrete class that fits your needs. `ArrayList` is the general-purpose default; `LinkedList` is good for frequent insertions/deletions at the beginning; `CopyOnWriteArrayList` is for multi-threaded scenarios.

### Sub-feature 2.1: ArrayList

#### Definitions

- **Core Definition:** `ArrayList` is a resizable-array implementation of the `List` interface.
- **Technical Definition:** Backed by a dynamically growing array. Offers O(1) random access, amortized O(1) appending, and O(n) insertion/removal at arbitrary positions (due to element shifting).
- **Beginner-Friendly Explanation:** `ArrayList` is like a smart array that grows automatically when you add more items. It's the most commonly used list because it's fast for most operations.

#### Purposes

- To provide fast random access by index.
- To serve as the default general-purpose list implementation.
- To minimize memory overhead compared to linked structures.

#### Syntax Structures and Rules

**General Syntax:**
```
List<Type> list = new ArrayList<>();          // default initial capacity 10
List<Type> list = new ArrayList<>(capacity);  // specify initial capacity
List<Type> list = new ArrayList<>(collection); // copy from another collection
```

**Syntax Rules:**
- Initial capacity does not limit maximum size; the array grows automatically.
- Adding beyond capacity triggers a resize operation (O(n) amortized).
- Not thread-safe; use `Collections.synchronizedList` or `CopyOnWriteArrayList` for concurrency.

**Constraints and Limitations:**
- Insertion/removal in the middle is O(n) due to array shifting.
- Not synchronized; concurrent modification from multiple threads is unsafe.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class ArrayListDemo {
    public static void main(String[] args) {
        // Create an ArrayList with default capacity
        List<Integer> numbers = new ArrayList<>();
        
        // Append elements (fast: amortized O(1))
        numbers.add(10);
        numbers.add(20);
        numbers.add(30);
        numbers.add(40);
        numbers.add(50);
        System.out.println("After appends: " + numbers);
        
        // Random access by index (fast: O(1))
        int third = numbers.get(2);
        System.out.println("Element at index 2: " + third);
        
        // Insert at beginning (slow: O(n) due to shifting)
        numbers.add(0, 5);
        System.out.println("After add(0, 5): " + numbers);
        
        // Remove from middle (slow: O(n))
        numbers.remove(3);
        System.out.println("After remove(3): " + numbers);
        
        // Check size
        System.out.println("Size: " + numbers.size());
    }
}
```

**Expected Output:**
```
After appends: [10, 20, 30, 40, 50]
Element at index 2: 30
After add(0, 5): [5, 10, 20, 30, 40, 50]
After remove(3): [5, 10, 20, 40, 50]
Size: 5
```

**Why This Output:** The `ArrayList` stores elements in an internal array. `get(2)` retrieves the third element (30). `add(0, 5)` inserts at the beginning, shifting all elements right. `remove(3)` removes the element at index 3 (originally 30), shifting subsequent elements left.

#### Real-World Case

**Scenario:** A to-do list application. Tasks are stored in an `ArrayList` because users mostly append new tasks and access them by index when displaying a numbered list. Occasional insertions (e.g., inserting a high-priority task at the top) are acceptable despite the O(n) cost.

#### References

- ArrayList (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html
- List Implementations (Oracle Tutorial) — https://docs.oracle.com/javase/tutorial/collections/implementations/list.html

---

### Sub-feature 2.2: LinkedList

#### Definitions

- **Core Definition:** `LinkedList` is a doubly-linked list implementation of the `List` and `Queue` interfaces.
- **Technical Definition:** Each element is stored in a node containing references to the previous and next nodes. Offers O(1) insertion/removal at the beginning and end, but O(n) positional access.
- **Beginner-Friendly Explanation:** `LinkedList` is like a chain of connected boxes. Each box knows only about the box before and after it. Adding or removing at the ends is very fast, but finding a box in the middle requires walking the chain from the start.

#### Purposes

- To provide fast insertion and removal at the beginning and end.
- To implement stacks and queues efficiently.
- To reduce memory overhead when elements are frequently added/removed from the ends.

#### Syntax Structures and Rules

**General Syntax:**
```
List<Type> list = new LinkedList<>();
LinkedList<Type> ll = new LinkedList<>();
ll.addFirst(element);   // O(1)
ll.addLast(element);    // O(1)
ll.getFirst();          // O(1)
ll.getLast();           // O(1)
ll.removeFirst();       // O(1)
ll.removeLast();        // O(1)
```

**Syntax Rules:**
- Implements both `List` and `Deque` (double-ended queue).
- Positional access (`get(int)`) is O(n).
- Ideal when you frequently add/remove at the beginning.

**Constraints and Limitations:**
- Higher memory overhead per element (node objects with two references).
- Poor cache locality compared to `ArrayList`; often slower in practice even for operations that are theoretically O(1).

#### Annotated Code Example

```java
import java.util.LinkedList;

public class LinkedListDemo {
    public static void main(String[] args) {
        LinkedList<String> queue = new LinkedList<>();
        
        // Add to the end (O(1))
        queue.addLast("Task 1");
        queue.addLast("Task 2");
        queue.addLast("Task 3");
        System.out.println("Queue: " + queue);
        
        // Add to the front (O(1))
        queue.addFirst("Urgent Task");
        System.out.println("After addFirst: " + queue);
        
        // Access first and last (O(1))
        System.out.println("First: " + queue.getFirst());
        System.out.println("Last: " + queue.getLast());
        
        // Remove from front (O(1))
        String removed = queue.removeFirst();
        System.out.println("Removed: " + removed);
        System.out.println("Final queue: " + queue);
    }
}
```

**Expected Output:**
```
Queue: [Task 1, Task 2, Task 3]
After addFirst: [Urgent Task, Task 1, Task 2, Task 3]
First: Urgent Task
Last: Task 3
Removed: Urgent Task
Final queue: [Task 1, Task 2, Task 3]
```

**Why This Output:** `addLast` appends to the end, `addFirst` inserts at the beginning. `getFirst` and `getLast` access the ends in O(1). `removeFirst` removes the head node and returns its element.

#### Real-World Case

**Scenario:** A browser's back/forward navigation history. Each page visit is added to a `LinkedList`. Going back removes the last element; going forward adds a new page. The O(1) operations at the ends make this efficient.

#### References

- LinkedList (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html

---

### Sub-feature 2.3: Vector & Stack (Legacy)

#### Definitions

- **Core Definition:** `Vector` is a synchronized resizable-array implementation; `Stack` extends `Vector` to provide LIFO operations.
- **Technical Definition:** `Vector` methods are synchronized, making it thread-safe but slower than `ArrayList` in single-threaded contexts. `Stack` adds `push()`, `pop()`, and `peek()`.
- **Beginner-Friendly Explanation:** `Vector` is like an old, thread-safe version of `ArrayList`. `Stack` is a `Vector` that adds "last-in-first-out" behavior (like a stack of plates). These are legacy classes and generally not recommended for new code.

#### Purposes

- To provide synchronized list operations without external wrapping.
- To support legacy code that depends on `Vector` or `Stack`.
- To implement LIFO data structures.

#### Syntax Structures and Rules

**General Syntax:**
```
Vector<Type> vector = new Vector<>();
Stack<Type> stack = new Stack<>();
stack.push(element);    // add to top
stack.pop();            // remove from top
stack.peek();           // view top without removing
```

**Syntax Rules:**
- `Vector` methods are `synchronized`; individual operations are thread-safe.
- `Stack` is a subclass of `Vector`; it inherits all `Vector` methods.
- `Stack` is not a true LIFO if `Vector` methods are used directly (e.g., `add(0, element)`).

**Constraints and Limitations:**
- **Deprecated/legacy:** Oracle recommends using `ArrayList` or `Collections.synchronizedList` instead of `Vector`.
- `Stack` is considered a design flaw (inherits from `Vector`); use `Deque` (e.g., `ArrayDeque`) for stack behavior.
- Synchronization per-method does not guarantee thread safety for compound operations.

#### Annotated Code Example

```java
import java.util.Stack;

public class StackDemo {
    public static void main(String[] args) {
        Stack<String> stack = new Stack<>();
        
        // Push elements onto the stack
        stack.push("Bottom");
        stack.push("Middle");
        stack.push("Top");
        System.out.println("Stack: " + stack);
        
        // Peek at the top
        System.out.println("Peek: " + stack.peek());
        
        // Pop elements (LIFO order)
        while (!stack.isEmpty()) {
            System.out.println("Popped: " + stack.pop());
        }
        System.out.println("Stack after pops: " + stack);
    }
}
```

**Expected Output:**
```
Stack: [Bottom, Middle, Top]
Peek: Top
Popped: Top
Popped: Middle
Popped: Bottom
Stack after pops: []
```

**Why This Output:** `push` adds to the top of the stack (end of the vector). `peek` returns "Top" without removing it. `pop` removes and returns elements in LIFO order: Top, then Middle, then Bottom.

#### Real-World Case

**Scenario:** Maintaining a legacy banking application that uses `Vector` for thread-safe transaction lists. New development should migrate to `CopyOnWriteArrayList` or external synchronization with `ArrayList`.

#### References

- Vector (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/Vector.html
- Stack (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/Stack.html

---

### Sub-feature 2.4: CopyOnWriteArrayList

#### Definitions

- **Core Definition:** `CopyOnWriteArrayList` is a thread-safe `List` implementation where all mutative operations create a fresh copy of the underlying array.
- **Technical Definition:** Read operations are lock-free and never throw `ConcurrentModificationException`; write operations are synchronized and expensive because they copy the entire array.
- **Beginner-Friendly Explanation:** Every time you modify the list, it creates an entirely new copy. This makes reads very fast and safe, but writes are slow and memory-intensive.

#### Purposes

- To provide thread-safe iteration without synchronization.
- To support scenarios with frequent reads and infrequent writes.
- To eliminate `ConcurrentModificationException` during iteration.

#### Syntax Structures and Rules

**General Syntax:**
```
List<Type> list = new CopyOnWriteArrayList<>();
```

**Syntax Rules:**
- Iterators operate on a snapshot of the array at the time of creation.
- Iterator's `remove()` is not supported (throws `UnsupportedOperationException`).
- `add`, `set`, `remove` are synchronized and copy the array.

**Constraints and Limitations:**
- **Not suitable for write-heavy workloads** — each write copies the entire array (O(n)).
- Memory usage can be high during writes (two arrays exist temporarily).
- Iterators do not reflect changes made after iterator creation.

#### Annotated Code Example

```java
import java.util.Iterator;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class CopyOnWriteDemo {
    public static void main(String[] args) {
        List<String> list = new CopyOnWriteArrayList<>();
        list.add("A");
        list.add("B");
        list.add("C");
        
        // Get an iterator (snapshot)
        Iterator<String> it = list.iterator();
        
        // Modify the list while iterating (safe: iterator uses snapshot)
        list.add("D");
        
        // Iterate over the snapshot (does not include "D")
        System.out.println("Iterating snapshot:");
        while (it.hasNext()) {
            System.out.println(it.next());
        }
        
        // Print current list
        System.out.println("Current list: " + list);
    }
}
```

**Expected Output:**
```
Iterating snapshot:
A
B
C
Current list: [A, B, C, D]
```

**Why This Output:** The iterator was obtained before "D" was added, so it iterates over the snapshot `[A, B, C]`. The current list contains `[A, B, C, D]` because the iterator does not reflect post-creation modifications.

#### Real-World Case

**Scenario:** An event-handler registry in a GUI application. Listeners are added or removed rarely, but events are dispatched frequently. `CopyOnWriteArrayList` allows event dispatch to iterate over the listener list without synchronization while safely handling occasional listener registration changes.

#### References

- CopyOnWriteArrayList (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CopyOnWriteArrayList.html

---

## 3. List Traversal (Iteration)

### Core Definitions

- **Core Definition:** Traversal is the process of visiting each element in a `List` sequentially.
- **Technical Definition:** Java provides multiple iteration mechanisms: index-based for loops, enhanced for-each loops, `Iterator`, `ListIterator`, `forEach()` with lambdas, and `Stream`.
- **Beginner-Friendly Explanation:** Traversal means "going through the list item by item." Java gives you several ways to do this, each suited to different situations.

### Sub-feature 3.1: Standard for Loop (Index-Based)

#### Definitions

- **Core Definition:** A traditional `for` loop that uses an index to access each element via `get(index)`.
- **Technical Definition:** `for (int i = 0; i < list.size(); i++) { E element = list.get(i); }`.
- **Beginner-Friendly Explanation:** A counter-based loop that goes from 0 to `size()-1`, retrieving each element by its index.

#### Purposes

- To enable index-aware processing (e.g., accessing neighboring elements).
- To allow modification of elements via `set(index, element)`.
- To provide maximum control over iteration.

#### Syntax Structures and Rules

**General Syntax:**
```
for (int i = 0; i < list.size(); i++) {
    Type element = list.get(i);
    // process element
}
```

**Syntax Rules:**
- The loop variable `i` is the index.
- `list.get(i)` is O(1) for `ArrayList`, O(n) for `LinkedList`.
- Avoid calling `list.size()` in the condition if the list is modified; cache it if necessary.

**Constraints and Limitations:**
- Inefficient for `LinkedList` due to O(n) `get()` calls.
- Not suitable for concurrent modification (may throw `ConcurrentModificationException` or produce incorrect results).

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class ForLoopDemo {
    public static void main(String[] args) {
        List<String> cities = new ArrayList<>();
        cities.add("New York");
        cities.add("London");
        cities.add("Tokyo");
        
        // Index-based for loop
        for (int i = 0; i < cities.size(); i++) {
            System.out.println("City " + i + ": " + cities.get(i));
        }
        
        // Modify elements via set()
        for (int i = 0; i < cities.size(); i++) {
            cities.set(i, cities.get(i).toUpperCase());
        }
        System.out.println("Uppercased: " + cities);
    }
}
```

**Expected Output:**
```
City 0: New York
City 1: London
City 2: Tokyo
Uppercased: [NEW YORK, LONDON, TOKYO]
```

**Why This Output:** The first loop uses `i` as the index to retrieve and print each city. The second loop uses `set(i, ...)` to replace each element with its uppercase version.

#### Real-World Case

**Scenario:** A game board represented as a `List<Tile>`. The game logic needs to check each tile's index to determine adjacency (e.g., tile at index `i` is adjacent to tiles at `i-1` and `i+1`).

---

### Sub-feature 3.2: Enhanced for-each Loop

#### Definitions

- **Core Definition:** A simplified loop that iterates over all elements without explicit index management.
- **Technical Definition:** `for (Type element : list) { ... }` — compiles to an iterator-based loop internally.
- **Beginner-Friendly Explanation:** A clean, readable loop that says "for each element in the list, do this."

#### Purposes

- To provide clean, readable iteration when the index is not needed.
- To avoid off-by-one errors.
- To work uniformly across all `Iterable` types.

#### Syntax Structures and Rules

**General Syntax:**
```
for (Type element : collection) {
    // process element
}
```

**Syntax Rules:**
- The loop variable `element` is assigned each element in turn.
- Cannot modify the list structurally during iteration (may throw `ConcurrentModificationException`).
- Does not expose the index.

**Constraints and Limitations:**
- Cannot access the index directly.
- Cannot remove elements safely during iteration (use `Iterator.remove()`).
- For `LinkedList`, it uses an iterator internally and is efficient.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class ForEachDemo {
    public static void main(String[] args) {
        List<Integer> scores = new ArrayList<>();
        scores.add(85);
        scores.add(92);
        scores.add(78);
        scores.add(95);
        
        int sum = 0;
        // For-each loop: clean and simple
        for (int score : scores) {
            System.out.println("Score: " + score);
            sum += score;
        }
        
        double average = (double) sum / scores.size();
        System.out.println("Sum: " + sum);
        System.out.println("Average: " + average);
    }
}
```

**Expected Output:**
```
Score: 85
Score: 92
Score: 78
Score: 95
Sum: 350
Average: 87.5
```

**Why This Output:** The for-each loop iterates over each `Integer` in the list, printing it and accumulating the sum. The average is calculated by dividing the sum by the size.

#### Real-World Case

**Scenario:** Calculating the total price of items in a shopping cart. The for-each loop cleanly iterates over each item and adds its price to a running total, without needing to know each item's index.

---

### Sub-feature 3.3: Iterator (Forward-Only)

#### Definitions

- **Core Definition:** An `Iterator` provides a way to traverse a collection and safely remove elements during iteration.
- **Technical Definition:** `Iterator<E>` has `hasNext()`, `next()`, and `remove()`. It is forward-only.
- **Beginner-Friendly Explanation:** An iterator is like a bookmark that moves through the list one item at a time. It also lets you remove the current item safely.

#### Purposes

- To traverse a collection without exposing its underlying structure.
- To safely remove elements during iteration via `remove()`.
- To provide a uniform traversal mechanism for all collections.

#### Syntax Structures and Rules

**General Syntax:**
```
Iterator<Type> it = list.iterator();
while (it.hasNext()) {
    Type element = it.next();
    if (condition) {
        it.remove();  // safe removal
    }
}
```

**Syntax Rules:**
- `next()` must be called before `remove()`.
- `remove()` can only be called once per `next()` call.
- `ConcurrentModificationException` is thrown if the list is structurally modified outside the iterator.

**Constraints and Limitations:**
- Forward-only; cannot traverse backward.
- Cannot replace elements during iteration.
- `remove()` removes the last element returned by `next()`.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorDemo {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("Alice");
        names.add("Bob");
        names.add("Charlie");
        names.add("David");
        
        // Remove names starting with 'C' using Iterator
        Iterator<String> it = names.iterator();
        while (it.hasNext()) {
            String name = it.next();
            if (name.startsWith("C")) {
                it.remove();  // safe removal during iteration
                System.out.println("Removed: " + name);
            }
        }
        
        System.out.println("Remaining: " + names);
    }
}
```

**Expected Output:**
```
Removed: Charlie
Remaining: [Alice, Bob, David]
```

**Why This Output:** The iterator traverses the list. When it encounters "Charlie" (which starts with "C"), it calls `remove()` to safely delete the current element. The list is structurally modified only through the iterator, so no `ConcurrentModificationException` is thrown.

#### Real-World Case

**Scenario:** Filtering a list of email addresses to remove invalid ones. The `Iterator` allows safe removal of invalid entries while iterating through the list, avoiding the pitfalls of modifying a list during a for-each loop.

#### References

- Iterator (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html

---

### Sub-feature 3.4: ListIterator (Bidirectional)

#### Definitions

- **Core Definition:** A `ListIterator` extends `Iterator` with bidirectional traversal, element replacement, and insertion.
- **Technical Definition:** `ListIterator<E>` adds `hasPrevious()`, `previous()`, `nextIndex()`, `previousIndex()`, `set(E)`, and `add(E)`.
- **Beginner-Friendly Explanation:** A `ListIterator` is a super-powered iterator that can go forward and backward, replace elements, and insert new ones during iteration.

#### Purposes

- To traverse a list in both directions.
- To modify elements during iteration via `set()`.
- To insert elements during iteration via `add()`.

#### Syntax Structures and Rules

**General Syntax:**
```
ListIterator<Type> lit = list.listIterator();
while (lit.hasNext()) {
    Type element = lit.next();
    lit.set(newValue);       // replace
    lit.add(newElement);     // insert
}
while (lit.hasPrevious()) {
    Type element = lit.previous();
}
```

**Syntax Rules:**
- `set()` can only be called after `next()` or `previous()`.
- `add()` inserts before the element that would be returned by `next()`.
- `remove()` removes the last element returned by `next()` or `previous()`.

**Constraints and Limitations:**
- Not all `List` implementations support `set()` and `add()` (e.g., immutable lists throw `UnsupportedOperationException`).
- `CopyOnWriteArrayList` does not support `ListIterator` modification.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;

public class ListIteratorDemo {
    public static void main(String[] args) {
        List<String> items = new ArrayList<>();
        items.add("A");
        items.add("B");
        items.add("C");
        
        // Forward traversal with replacement
        ListIterator<String> lit = items.listIterator();
        while (lit.hasNext()) {
            String item = lit.next();
            lit.set(item + "!");  // replace each element
        }
        System.out.println("After replacement: " + items);
        
        // Backward traversal
        System.out.println("Backward traversal:");
        while (lit.hasPrevious()) {
            System.out.println(lit.previous());
        }
    }
}
```

**Expected Output:**
```
After replacement: [A!, B!, C!]
Backward traversal:
C!
B!
A!
```

**Why This Output:** The forward pass replaces each element with itself plus "!". After the forward pass, the iterator is at the end of the list. The backward pass then traverses from the end to the beginning, printing each element.

#### Real-World Case

**Scenario:** A text editor's undo/redo functionality. The undo operation traverses the edit history backward, while redo traverses forward. `ListIterator`'s bidirectional traversal makes this natural.

#### References

- ListIterator (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/ListIterator.html

---

### Sub-feature 3.5: forEach() Method with Lambda Expressions / Streams

#### Definitions

- **Core Definition:** `Iterable.forEach(Consumer)` executes a given action for each element; `Stream` provides a functional pipeline for processing collections.
- **Technical Definition:** `default void forEach(Consumer<? super T> action)` — added in Java 8 to the `Iterable` interface. `Stream` is obtained via `list.stream()`.
- **Beginner-Friendly Explanation:** `forEach` lets you pass a function (lambda) that gets applied to every element. Streams let you chain operations like filter, map, and collect.

#### Purposes

- To enable functional-style iteration with concise syntax.
- To separate iteration from the action performed.
- To leverage parallel processing with `parallelStream()`.

#### Syntax Structures and Rules

**General Syntax:**
```
list.forEach(element -> { /* action */ });
list.stream().filter(...).map(...).collect(...);
```

**Syntax Rules:**
- Lambda parameter type is inferred.
- `forEach` order is not guaranteed for parallel streams.
- Use `forEachOrdered` for ordered parallel processing.

**Constraints and Limitations:**
- Cannot use `break` or `continue` inside `forEach`.
- Cannot modify the list structurally inside `forEach` (may throw `ConcurrentModificationException`).
- Streams are single-use; a stream cannot be reused after a terminal operation.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

public class ForEachStreamDemo {
    public static void main(String[] args) {
        List<String> words = new ArrayList<>();
        words.add("apple");
        words.add("banana");
        words.add("cherry");
        words.add("avocado");
        
        // forEach with lambda
        System.out.println("All words:");
        words.forEach(w -> System.out.println("  " + w));
        
        // Stream: filter, map, collect
        List<String> upperAWords = words.stream()
            .filter(w -> w.startsWith("a"))
            .map(String::toUpperCase)
            .collect(Collectors.toList());
        System.out.println("Words starting with 'a', uppercased: " + upperAWords);
    }
}
```

**Expected Output:**
```
All words:
  apple
  banana
  cherry
  avocado
Words starting with 'a', uppercased: [APPLE, AVOCADO]
```

**Why This Output:** `forEach` applies the lambda to each word, printing it. The stream pipeline filters words starting with "a", maps them to uppercase, and collects the results into a new list.

#### Real-World Case

**Scenario:** Processing a list of sensor readings. Streams can filter anomalous readings, map them to a normalized format, and collect the results for analysis, all in a single readable pipeline.

#### References

- Iterable.forEach (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html#forEach-java.util.function.Consumer-
- Stream (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html

---

## 4. List Insertion (Addition)

### Core Definitions

- **Core Definition:** Insertion operations add elements to a `List`.
- **Technical Definition:** The `List` interface provides `add(E e)` (append), `add(int index, E element)` (positional insert), and `addAll(Collection<? extends E> c)` (bulk append).
- **Beginner-Friendly Explanation:** Insertion means adding new items to the list — either at the end, at a specific position, or in bulk.

### Sub-feature 4.1: Appending Elements — `add(E e)`

#### Definitions

- **Core Definition:** Appends the specified element to the end of the list.
- **Technical Definition:** `boolean add(E e)` — returns `true` if the list changed as a result.
- **Beginner-Friendly Explanation:** Adds a new item to the very end of the list.

#### Purposes

- To add elements in insertion order.
- To build a list incrementally.
- To provide the most common list operation.

#### Syntax Structures and Rules

**General Syntax:**
```
list.add(element);
```

**Syntax Rules:**
- Always returns `true` for `List` (unlike `Set`).
- Amortized O(1) for `ArrayList`; O(1) for `LinkedList`.
- Throws `UnsupportedOperationException` for immutable lists.

**Constraints and Limitations:**
- Cannot add `null` to lists created by `List.of()`.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class AddDemo {
    public static void main(String[] args) {
        List<String> tasks = new ArrayList<>();
        
        boolean added1 = tasks.add("Buy groceries");
        boolean added2 = tasks.add("Walk the dog");
        boolean added3 = tasks.add("Read a book");
        
        System.out.println("Added 1: " + added1);
        System.out.println("Added 2: " + added2);
        System.out.println("Added 3: " + added3);
        System.out.println("Tasks: " + tasks);
    }
}
```

**Expected Output:**
```
Added 1: true
Added 2: true
Added 3: true
Tasks: [Buy groceries, Walk the dog, Read a book]
```

**Why This Output:** Each `add()` call returns `true` because the list is modifiable and each addition changes the list. The elements are stored in insertion order.

#### Real-World Case

**Scenario:** Logging application events. Each event is appended to an `ArrayList<LogEntry>` as it occurs, preserving chronological order.

---

### Sub-feature 4.2: Positional Insertion — `add(int index, E element)`

#### Definitions

- **Core Definition:** Inserts the specified element at the specified position, shifting subsequent elements to the right.
- **Technical Definition:** `void add(int index, E element)` — index must be between 0 and `size()`.
- **Beginner-Friendly Explanation:** Inserts a new item at a specific position, pushing the existing items at that position and after it one spot to the right.

#### Purposes

- To insert elements at arbitrary positions.
- To maintain sorted order by inserting at the correct index.
- To prioritize elements by placing them at the front.

#### Syntax Structures and Rules

**General Syntax:**
```
list.add(index, element);
```

**Syntax Rules:**
- `index` must satisfy `0 <= index <= size()`.
- Throws `IndexOutOfBoundsException` if index is out of range.
- O(n) for `ArrayList` (shifts elements); O(n) for `LinkedList` (traverses to index).

**Constraints and Limitations:**
- Expensive for `ArrayList` when inserting near the front.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class PositionalAddDemo {
    public static void main(String[] args) {
        List<String> priorities = new ArrayList<>();
        priorities.add("Low");
        priorities.add("Medium");
        priorities.add("Low");
        System.out.println("Before: " + priorities);
        
        // Insert "High" at the beginning
        priorities.add(0, "High");
        System.out.println("After add(0, 'High'): " + priorities);
        
        // Insert "Critical" at index 1
        priorities.add(1, "Critical");
        System.out.println("After add(1, 'Critical'): " + priorities);
    }
}
```

**Expected Output:**
```
Before: [Low, Medium, Low]
After add(0, 'High'): [High, Low, Medium, Low]
After add(1, 'Critical'): [High, Critical, Low, Medium, Low]
```

**Why This Output:** `add(0, "High")` inserts at the beginning, shifting all existing elements right. `add(1, "Critical")` inserts at index 1, shifting "Low", "Medium", and "Low" further right.

#### Real-World Case

**Scenario:** A task manager where users can insert a new task at a specific position in their priority list.

---

### Sub-feature 4.3: Bulk Insertion — `addAll(Collection<? extends E> c)`

#### Definitions

- **Core Definition:** Appends all elements from a specified collection to the end of the list.
- **Technical Definition:** `boolean addAll(Collection<? extends E> c)` — returns `true` if the list changed.
- **Beginner-Friendly Explanation:** Adds all items from another collection to the end of the list.

#### Purposes

- To concatenate two lists.
- To copy elements from one collection to another.
- To efficiently add multiple elements at once.

#### Syntax Structures and Rules

**General Syntax:**
```
list.addAll(collection);
list.addAll(index, collection);   // insert at position
```

**Syntax Rules:**
- The order of added elements follows the collection's iterator.
- Returns `true` if the list changed (i.e., the collection was non-empty).

**Constraints and Limitations:**
- Does not deduplicate; duplicates are allowed.
- If the collection is the list itself, behavior is undefined.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class AddAllDemo {
    public static void main(String[] args) {
        List<String> list1 = new ArrayList<>();
        list1.add("A");
        list1.add("B");
        
        List<String> list2 = new ArrayList<>();
        list2.add("C");
        list2.add("D");
        list2.add("E");
        
        // Append all of list2 to list1
        boolean changed = list1.addAll(list2);
        System.out.println("Changed: " + changed);
        System.out.println("Combined: " + list1);
        
        // Insert list2 at index 1 of a new list
        List<String> list3 = new ArrayList<>();
        list3.add("X");
        list3.add("Y");
        list3.addAll(1, list2);
        System.out.println("Inserted at index 1: " + list3);
    }
}
```

**Expected Output:**
```
Changed: true
Combined: [A, B, C, D, E]
Inserted at index 1: [X, C, D, E, Y]
```

**Why This Output:** `addAll(list2)` appends C, D, E to the end of list1. `addAll(1, list2)` inserts C, D, E starting at index 1 of list3, shifting "Y" to the end.

#### Real-World Case

**Scenario:** Merging two lists of customer orders. One list contains online orders, another contains in-store orders. `addAll` combines them into a single list for processing.

#### References

- The List Interface (Oracle Tutorial) — https://docs.oracle.com/javase/tutorial/collections/interfaces/list.html

---

## 5. List Removal

### Core Definitions

- **Core Definition:** Removal operations delete elements from a `List`.
- **Technical Definition:** The `List` interface provides `remove(int index)`, `remove(Object o)`, `removeAll()`, `retainAll()`, `clear()`, and `removeIf(Predicate)`.
- **Beginner-Friendly Explanation:** Removal means taking items out of the list — either by position, by value, by condition, or all at once.

### Sub-feature 5.1: Removal by Index — `remove(int index)`

#### Definitions

- **Core Definition:** Removes the element at the specified position and returns it.
- **Technical Definition:** `E remove(int index)` — throws `IndexOutOfBoundsException` if index is invalid.
- **Beginner-Friendly Explanation:** Removes the item at a specific position and tells you what was removed.

#### Purposes

- To remove an element when its position is known.
- To retrieve and remove an element in one operation.
- To support stack/queue-like behavior.

#### Syntax Structures and Rules

**General Syntax:**
```
E removed = list.remove(index);
```

**Syntax Rules:**
- Index must be `0 <= index < size()`.
- Subsequent elements shift left.
- O(n) for `ArrayList`; O(n) for `LinkedList` (traversal).

**Constraints and Limitations:**
- Overloaded with `remove(Object)`; with an `Integer` argument, Java may select `remove(int)` or `remove(Object)` depending on boxing.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class RemoveByIndexDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");
        list.add("C");
        list.add("D");
        System.out.println("Before: " + list);
        
        String removed = list.remove(2);  // remove "C"
        System.out.println("Removed: " + removed);
        System.out.println("After: " + list);
    }
}
```

**Expected Output:**
```
Before: [A, B, C, D]
Removed: C
After: [A, B, D]
```

**Why This Output:** `remove(2)` removes the element at index 2 ("C") and returns it. "D" shifts from index 3 to index 2.

#### Real-World Case

**Scenario:** A music playlist where the user clicks "remove" on the third song. The application calls `remove(2)` to delete it by position.

---

### Sub-feature 5.2: Removal by Object/Value — `remove(Object o)`

#### Definitions

- **Core Definition:** Removes the first occurrence of the specified element, if present.
- **Technical Definition:** `boolean remove(Object o)` — returns `true` if an element was removed.
- **Beginner-Friendly Explanation:** Removes the first item that matches the given value.

#### Purposes

- To remove a specific value from the list.
- To delete an element when its position is unknown.
- To support value-based filtering.

#### Syntax Structures and Rules

**General Syntax:**
```
boolean removed = list.remove(object);
```

**Syntax Rules:**
- Uses `equals()` to determine matching.
- Returns `false` if the element is not found.
- O(n) for both `ArrayList` and `LinkedList` (linear search).

**Constraints and Limitations:**
- Must override `equals()` for custom objects to work correctly.
- Ambiguity with `remove(int)` when passing `Integer` objects.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class RemoveByObjectDemo {
    public static void main(String[] args) {
        List<String> fruits = new ArrayList<>();
        fruits.add("Apple");
        fruits.add("Banana");
        fruits.add("Apple");
        fruits.add("Cherry");
        System.out.println("Before: " + fruits);
        
        boolean removed1 = fruits.remove("Apple");   // removes first "Apple"
        System.out.println("Removed 'Apple': " + removed1);
        System.out.println("After: " + fruits);
        
        boolean removed2 = fruits.remove("Grape");   // not present
        System.out.println("Removed 'Grape': " + removed2);
    }
}
```

**Expected Output:**
```
Before: [Apple, Banana, Apple, Cherry]
Removed 'Apple': true
After: [Banana, Apple, Cherry]
Removed 'Grape': false
```

**Why This Output:** `remove("Apple")` removes the first occurrence (index 0). The second "Apple" at index 1 (now index 1) remains. `remove("Grape")` returns `false` because "Grape" is not in the list.

#### Real-World Case

**Scenario:** A contact list where the user deletes a contact by name. The application calls `remove("John Doe")` to remove the first matching contact.

---

### Sub-feature 5.3: Bulk Removal — `removeAll()`, `retainAll()`, `clear()`

#### Definitions

- **Core Definition:** Bulk removal operations remove multiple elements at once.
- **Technical Definition:**
  - `removeAll(Collection<?> c)` — removes all elements contained in the specified collection.
  - `retainAll(Collection<?> c)` — retains only elements contained in the specified collection.
  - `clear()` — removes all elements.
- **Beginner-Friendly Explanation:** `removeAll` deletes everything that's in another collection; `retainAll` keeps only what's in another collection; `clear` empties the entire list.

#### Purposes

- To perform set-like operations on lists.
- To filter lists based on membership.
- To reset a list to empty.

#### Syntax Structures and Rules

**General Syntax:**
```
list.removeAll(collection);
list.retainAll(collection);
list.clear();
```

**Syntax Rules:**
- All three return `boolean` indicating whether the list changed (except `clear()` returns `void`).
- `removeAll` and `retainAll` use `equals()` for comparison.

**Constraints and Limitations:**
- `retainAll` is the complement of `removeAll`.
- Performance is O(n*m) in the worst case.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class BulkRemovalDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");
        list.add("C");
        list.add("D");
        list.add("E");
        
        List<String> toRemove = new ArrayList<>();
        toRemove.add("B");
        toRemove.add("D");
        
        // removeAll
        List<String> copy1 = new ArrayList<>(list);
        copy1.removeAll(toRemove);
        System.out.println("After removeAll: " + copy1);
        
        // retainAll (keep only B and D)
        List<String> copy2 = new ArrayList<>(list);
        copy2.retainAll(toRemove);
        System.out.println("After retainAll: " + copy2);
        
        // clear
        List<String> copy3 = new ArrayList<>(list);
        copy3.clear();
        System.out.println("After clear: " + copy3);
    }
}
```

**Expected Output:**
```
After removeAll: [A, C, E]
After retainAll: [B, D]
After clear: []
```

**Why This Output:** `removeAll(toRemove)` removes "B" and "D". `retainAll(toRemove)` keeps only "B" and "D". `clear()` removes all elements, resulting in an empty list.

#### Real-World Case

**Scenario:** An inventory system that removes all discontinued products (`removeAll`) or retains only products in a specific category (`retainAll`).

---

### Sub-feature 5.4: Conditional Removal — `removeIf(Predicate<? super E> filter)`

#### Definitions

- **Core Definition:** Removes all elements that satisfy the given predicate.
- **Technical Definition:** `default boolean removeIf(Predicate<? super E> filter)` — added in Java 8.
- **Beginner-Friendly Explanation:** Removes every item that matches a condition you specify.

#### Purposes

- To filter a list in place using a lambda expression.
- To remove elements based on complex conditions.
- To avoid manual iterator-based removal.

#### Syntax Structures and Rules

**General Syntax:**
```
list.removeIf(element -> condition);
```

**Syntax Rules:**
- Uses an internal iterator to safely remove matching elements.
- Returns `true` if any elements were removed.
- Throws `NullPointerException` if the predicate is `null`.

**Constraints and Limitations:**
- Only available in Java 8 and later.
- Some implementations (`CopyOnWriteArrayList`) have optimized overrides.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class RemoveIfDemo {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>();
        for (int i = 1; i <= 10; i++) {
            numbers.add(i);
        }
        System.out.println("Original: " + numbers);
        
        // Remove all even numbers
        boolean removed = numbers.removeIf(n -> n % 2 == 0);
        System.out.println("Removed evens: " + removed);
        System.out.println("After removeIf: " + numbers);
        
        // Remove numbers greater than 7
        numbers.removeIf(n -> n > 7);
        System.out.println("After removing > 7: " + numbers);
    }
}
```

**Expected Output:**
```
Original: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
Removed evens: true
After removeIf: [1, 3, 5, 7, 9]
After removing > 7: [1, 3, 5, 7]
```

**Why This Output:** `removeIf(n -> n % 2 == 0)` removes all even numbers (2, 4, 6, 8, 10). Then `removeIf(n -> n > 7)` removes 9, leaving `[1, 3, 5, 7]`.

#### Real-World Case

**Scenario:** An email client removing all messages marked as spam. `removeIf(msg -> msg.isSpam())` cleans the inbox in one line.

#### References

- How To Use remove() Methods for Java List and ArrayList (DigitalOcean) — https://www.digitalocean.com/community/tutorials/java-list-remove-methods-arraylist-remove

---

## 6. List Searching & Retrieval

### Core Definitions

- **Core Definition:** Searching operations locate elements within a `List` and retrieve their positions or existence.
- **Technical Definition:** The `List` interface provides `get(int index)`, `indexOf(Object o)`, `lastIndexOf(Object o)`, `contains(Object o)`, and `containsAll(Collection<?> c)`.
- **Beginner-Friendly Explanation:** Searching means looking for an item in the list and finding out where it is or whether it exists.

### Sub-feature 6.1: Positional Retrieval — `get(int index)`

#### Definitions

- **Core Definition:** Returns the element at the specified position.
- **Technical Definition:** `E get(int index)` — throws `IndexOutOfBoundsException` if index is invalid.
- **Beginner-Friendly Explanation:** Gets the item at a specific position number.

#### Purposes

- To retrieve an element when its index is known.
- To access elements randomly.
- To support algorithms that require indexed access.

#### Syntax Structures and Rules

**General Syntax:**
```
E element = list.get(index);
```

**Syntax Rules:**
- Index must be `0 <= index < size()`.
- O(1) for `ArrayList`; O(n) for `LinkedList`.

**Constraints and Limitations:**
- Not supported by immutable lists for mutation, but `get()` works on all lists.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class GetDemo {
    public static void main(String[] args) {
        List<String> planets = new ArrayList<>();
        planets.add("Mercury");
        planets.add("Venus");
        planets.add("Earth");
        planets.add("Mars");
        
        // Access by index
        System.out.println("First planet: " + planets.get(0));
        System.out.println("Third planet: " + planets.get(2));
        System.out.println("Last planet: " + planets.get(planets.size() - 1));
    }
}
```

**Expected Output:**
```
First planet: Mercury
Third planet: Earth
Last planet: Mars
```

**Why This Output:** `get(0)` returns the first element, `get(2)` returns the third, and `get(size()-1)` returns the last element.

#### Real-World Case

**Scenario:** A leaderboard where the top 3 players are displayed by accessing indices 0, 1, and 2.

---

### Sub-feature 6.2: Finding Indices — `indexOf()` and `lastIndexOf()`

#### Definitions

- **Core Definition:** `indexOf` returns the first occurrence index; `lastIndexOf` returns the last occurrence index.
- **Technical Definition:** `int indexOf(Object o)` and `int lastIndexOf(Object o)` — return `-1` if not found.
- **Beginner-Friendly Explanation:** `indexOf` finds the first position of a value; `lastIndexOf` finds the last position.

#### Purposes

- To locate the position of an element.
- To distinguish between first and last occurrences of duplicates.
- To check existence by comparing the result to `-1`.

#### Syntax Structures and Rules

**General Syntax:**
```
int first = list.indexOf(object);
int last = list.lastIndexOf(object);
```

**Syntax Rules:**
- Uses `equals()` for comparison.
- Returns `-1` if not found.
- O(n) linear search.

**Constraints and Limitations:**
- Requires proper `equals()` implementation for custom objects.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class IndexOfDemo {
    public static void main(String[] args) {
        List<String> items = new ArrayList<>();
        items.add("X");
        items.add("Y");
        items.add("Z");
        items.add("Y");
        items.add("X");
        
        System.out.println("First 'Y': " + items.indexOf("Y"));
        System.out.println("Last 'Y': " + items.lastIndexOf("Y"));
        System.out.println("First 'X': " + items.indexOf("X"));
        System.out.println("Last 'X': " + items.lastIndexOf("X"));
        System.out.println("'W' index: " + items.indexOf("W"));
    }
}
```

**Expected Output:**
```
First 'Y': 1
Last 'Y': 3
First 'X': 0
Last 'X': 4
'W' index: -1
```

**Why This Output:** "Y" first appears at index 1 and last at index 3. "X" first appears at index 0 and last at index 4. "W" is not present, so `-1` is returned.

#### Real-World Case

**Scenario:** A text editor finding the first and last occurrences of a search term in a document.

---

### Sub-feature 6.3: Checking Existence — `contains()` and `containsAll()`

#### Definitions

- **Core Definition:** `contains` checks if a single element exists; `containsAll` checks if all elements of another collection exist.
- **Technical Definition:** `boolean contains(Object o)` and `boolean containsAll(Collection<?> c)`.
- **Beginner-Friendly Explanation:** `contains` asks "Is this item in the list?" `containsAll` asks "Are all these items in the list?"

#### Purposes

- To verify membership without needing the index.
- To validate that a list contains required elements.
- To support conditional logic based on list contents.

#### Syntax Structures and Rules

**General Syntax:**
```
boolean has = list.contains(element);
boolean hasAll = list.containsAll(collection);
```

**Syntax Rules:**
- Uses `equals()` for comparison.
- Returns `false` if the element/collection is not present.
- O(n) for `contains`; O(n*m) for `containsAll`.

**Constraints and Limitations:**
- Requires proper `equals()` implementation for custom objects.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class ContainsDemo {
    public static void main(String[] args) {
        List<String> inventory = new ArrayList<>();
        inventory.add("Sword");
        inventory.add("Shield");
        inventory.add("Potion");
        
        System.out.println("Has Sword: " + inventory.contains("Sword"));
        System.out.println("Has Bow: " + inventory.contains("Bow"));
        
        List<String> required = new ArrayList<>();
        required.add("Sword");
        required.add("Shield");
        
        System.out.println("Has all required: " + inventory.containsAll(required));
        
        required.add("Bow");
        System.out.println("Has all required (with Bow): " + inventory.containsAll(required));
    }
}
```

**Expected Output:**
```
Has Sword: true
Has Bow: false
Has all required: true
Has all required (with Bow): false
```

**Why This Output:** The inventory contains "Sword" but not "Bow". `containsAll` returns `true` for `[Sword, Shield]` because both are present, and `false` for `[Sword, Shield, Bow]` because "Bow" is missing.

#### Real-World Case

**Scenario:** A recipe app checking if the user's pantry contains all required ingredients for a recipe.

#### References

- How to Find an Element in a List with Java (Baeldung) — https://www.baeldung.com/find-list-element-java

---

## 7. Advanced & Utility Operations

### Core Definitions

- **Core Definition:** Advanced operations include sorting, sublist views, array conversion, and immutability.
- **Technical Definition:** `List.sort(Comparator)`, `subList(int, int)`, `toArray()`, `List.of()`, and `Collections.unmodifiableList()`.
- **Beginner-Friendly Explanation:** These operations help you sort lists, work with portions of lists, convert to arrays, and create read-only lists.

### Sub-feature 7.1: Sorting — `List.sort()`, `Collections.sort()`, Comparator

#### Definitions

- **Core Definition:** Sorting rearranges elements into a specified order.
- **Technical Definition:** `default void sort(Comparator<? super E> c)` — added in Java 8. `Collections.sort(List<T>)` is the legacy version.
- **Beginner-Friendly Explanation:** Sorting puts the list items in order — alphabetical, numerical, or any custom order you define.

#### Purposes

- To arrange data in ascending or descending order.
- To enable efficient searching (binary search requires sorted data).
- To present data in a user-friendly order.

#### Syntax Structures and Rules

**General Syntax:**
```
list.sort(Comparator.naturalOrder());
list.sort(Comparator.reverseOrder());
list.sort((a, b) -> a.compareTo(b));
Collections.sort(list);   // legacy
```

**Syntax Rules:**
- `List.sort()` is preferred over `Collections.sort()` since Java 8.
- The list is modified in place.
- Uses a stable sort (TimSort).

**Constraints and Limitations:**
- Elements must be mutually comparable or a `Comparator` must be provided.
- `Collections.sort` historically failed with `CopyOnWriteArrayList` because it uses `ListIterator.set`; modern overrides fix this.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class SortDemo {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("Charlie");
        names.add("Alice");
        names.add("Bob");
        System.out.println("Original: " + names);
        
        // Natural order (alphabetical)
        names.sort(Comparator.naturalOrder());
        System.out.println("Sorted: " + names);
        
        // Reverse order
        names.sort(Comparator.reverseOrder());
        System.out.println("Reverse: " + names);
        
        // Custom comparator: by length
        names.sort(Comparator.comparingInt(String::length));
        System.out.println("By length: " + names);
    }
}
```

**Expected Output:**
```
Original: [Charlie, Alice, Bob]
Sorted: [Alice, Bob, Charlie]
Reverse: [Charlie, Bob, Alice]
By length: [Bob, Alice, Charlie]
```

**Why This Output:** Natural order sorts alphabetically. Reverse order sorts descending. The custom comparator sorts by string length: "Bob" (3), "Alice" (5), "Charlie" (7).

#### Real-World Case

**Scenario:** An e-commerce site sorting products by price (low to high), name (A-Z), or rating (high to low) using different comparators.

#### References

- List.sort (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/List.html#sort-java.util.Comparator-

---

### Sub-feature 7.2: Sublists — `subList(int fromIndex, int toIndex)`

#### Definitions

- **Core Definition:** Returns a view of the portion of the list between `fromIndex` (inclusive) and `toIndex` (exclusive).
- **Technical Definition:** `List<E> subList(int fromIndex, int toIndex)` — returns a backed view; changes to the sublist are reflected in the original list.
- **Beginner-Friendly Explanation:** `subList` gives you a "window" into a portion of the list. Changes made through the window appear in the original list.

#### Purposes

- To operate on a range of elements without copying.
- To sort or modify a portion of the list.
- To extract a subset for processing.

#### Syntax Structures and Rules

**General Syntax:**
```
List<Type> sub = list.subList(fromIndex, toIndex);
```

**Syntax Rules:**
- `fromIndex` is inclusive; `toIndex` is exclusive.
- The returned list is a view (backed by the original).
- Structural modifications to the original outside the sublist invalidate the sublist.

**Constraints and Limitations:**
- The sublist is not a snapshot; it is a live view.
- Concurrent modification outside the sublist may throw `ConcurrentModificationException`.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;

public class SubListDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");
        list.add("C");
        list.add("D");
        list.add("E");
        System.out.println("Original: " + list);
        
        // Get sublist from index 1 (inclusive) to 4 (exclusive)
        List<String> sub = list.subList(1, 4);
        System.out.println("Sublist [1,4): " + sub);
        
        // Modify sublist (affects original)
        sub.set(0, "B-modified");
        System.out.println("After sub.set(0, ...): " + list);
        
        // Sort sublist
        sub.sort(null);
        System.out.println("After sub.sort(): " + list);
    }
}
```

**Expected Output:**
```
Original: [A, B, C, D, E]
Sublist [1,4): [B, C, D]
After sub.set(0, ...): [A, B-modified, C, D, E]
After sub.sort(): [A, B-modified, C, D, E]
```

**Why This Output:** `subList(1, 4)` returns elements at indices 1, 2, 3. Modifying `sub` changes the original list. Sorting the sublist sorts only that range in the original list.

#### Real-World Case

**Scenario:** Paginating a list of search results. `subList(page * pageSize, min((page+1) * pageSize, total))` extracts the current page's results.

---

### Sub-feature 7.3: Array Conversions — `toArray()`, `toArray(T[] a)`

#### Definitions

- **Core Definition:** Converts the list to an array.
- **Technical Definition:** `Object[] toArray()` and `<T> T[] toArray(T[] a)`.
- **Beginner-Friendly Explanation:** Turns the list into a plain array, either as `Object[]` or a typed array.

#### Purposes

- To interoperate with APIs that require arrays.
- To create a fixed-size snapshot of the list.
- To enable array-based algorithms.

#### Syntax Structures and Rules

**General Syntax:**
```
Object[] arr = list.toArray();
String[] arr = list.toArray(new String[0]);
```

**Syntax Rules:**
- `toArray()` returns `Object[]`.
- `toArray(T[] a)` returns an array of the same runtime type as `a`.
- If `a` is large enough, it is used; otherwise, a new array is allocated.

**Constraints and Limitations:**
- The returned array is a copy; modifying it does not affect the list.
- `toArray()` cannot be cast directly to `String[]`.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class ToArrayDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Apple");
        list.add("Banana");
        list.add("Cherry");
        
        // Object[] array
        Object[] objArr = list.toArray();
        System.out.println("Object array: " + Arrays.toString(objArr));
        
        // Typed array (preferred)
        String[] strArr = list.toArray(new String[0]);
        System.out.println("String array: " + Arrays.toString(strArr));
        
        // Typed array with pre-sized array
        String[] strArr2 = list.toArray(new String[list.size()]);
        System.out.println("Pre-sized array: " + Arrays.toString(strArr2));
    }
}
```

**Expected Output:**
```
Object array: [Apple, Banana, Cherry]
String array: [Apple, Banana, Cherry]
Pre-sized array: [Apple, Banana, Cherry]
```

**Why This Output:** All three methods produce an array containing the same elements in the same order. The typed versions return `String[]`, which is more useful than `Object[]`.

#### Real-World Case

**Scenario:** Passing list data to a legacy method that only accepts arrays, such as `String[]` for a database batch insert.

---

### Sub-feature 7.4: Immutability — `List.of()`, `Collections.unmodifiableList()`

#### Definitions

- **Core Definition:** Immutable lists cannot be modified after creation.
- **Technical Definition:** `List.of(E... elements)` creates a truly immutable list (Java 9+). `Collections.unmodifiableList(List)` returns an unmodifiable view of an existing list.
- **Beginner-Friendly Explanation:** `List.of()` creates a read-only list that can never be changed. `Collections.unmodifiableList()` wraps an existing list to prevent modification.

#### Purposes

- To create thread-safe, read-only data.
- To enforce immutability in APIs.
- To prevent accidental modification of critical data.

#### Syntax Structures and Rules

**General Syntax:**
```
List<Type> immutable = List.of(e1, e2, e3);
List<Type> unmodifiable = Collections.unmodifiableList(original);
```

**Syntax Rules:**
- `List.of()` disallows `null` elements; throws `NullPointerException`.
- `List.of()` throws `UnsupportedOperationException` on any mutator method.
- `Collections.unmodifiableList()` is a view; changes to the backing list are visible through the unmodifiable view.

**Constraints and Limitations:**
- `List.of()` is truly immutable; `Collections.unmodifiableList()` is a wrapper that reflects backing list changes.
- `List.of()` does not allow `null`; `Collections.unmodifiableList()` allows `null` if the backing list does.

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ImmutabilityDemo {
    public static void main(String[] args) {
        // List.of() - truly immutable
        List<String> immutable = List.of("A", "B", "C");
        System.out.println("Immutable: " + immutable);
        
        try {
            immutable.add("D");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify List.of(): " + e.getClass().getSimpleName());
        }
        
        // Collections.unmodifiableList() - view of a modifiable list
        List<String> mutable = new ArrayList<>();
        mutable.add("X");
        mutable.add("Y");
        List<String> unmodifiable = Collections.unmodifiableList(mutable);
        System.out.println("Unmodifiable view: " + unmodifiable);
        
        // Modify backing list (visible through view)
        mutable.add("Z");
        System.out.println("After backing list change: " + unmodifiable);
        
        try {
            unmodifiable.add("W");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify unmodifiable view: " + e.getClass().getSimpleName());
        }
    }
}
```

**Expected Output:**
```
Immutable: [A, B, C]
Cannot modify List.of(): UnsupportedOperationException
Unmodifiable view: [X, Y]
After backing list change: [X, Y, Z]
Cannot modify unmodifiable view: UnsupportedOperationException
```

**Why This Output:** `List.of("A", "B", "C")` creates a truly immutable list; any `add` call throws `UnsupportedOperationException`. `Collections.unmodifiableList(mutable)` creates a view; adding "Z" to the backing list is visible through the view, but calling `add` on the view itself throws `UnsupportedOperationException`.

#### Real-World Case

**Scenario:** Defining a list of configuration constants that must never be modified at runtime. `List.of()` ensures the constants remain fixed.

#### References

- Unmodifiable Lists (Oracle Java 11 Documentation) — https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html
- Java List Interface (Baeldung) — https://www.baeldung.com/java-list-interface

---

## Consolidated Reference Links

| Name | Link |
|------|------|
| List (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/List.html |
| The List Interface (Oracle Tutorial) | https://docs.oracle.com/javase/tutorial/collections/interfaces/list.html |
| List Implementations (Oracle Tutorial) | https://docs.oracle.com/javase/tutorial/collections/implementations/list.html |
| ArrayList (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html |
| LinkedList (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html |
| Vector (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/Vector.html |
| Stack (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/Stack.html |
| CopyOnWriteArrayList (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CopyOnWriteArrayList.html |
| Iterator (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html |
| ListIterator (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/ListIterator.html |
| Iterable.forEach (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html#forEach-java.util.function.Consumer- |
| Stream (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html |
| Java List Interface (Baeldung) | https://www.baeldung.com/java-list-interface |
| How to Find an Element in a List with Java (Baeldung) | https://www.baeldung.com/find-list-element-java |
| How To Use remove() Methods (DigitalOcean) | https://www.digitalocean.com/community/tutorials/java-list-remove-methods-arraylist-remove |
| Java Language Specification (Chapter 14) | https://docs.oracle.com/javase/specs/jls/se7/html/jls-14.html |