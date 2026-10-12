# Java Lists: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A `List` in Java is an ordered collection (also known as a sequence) that provides precise control over where each element is inserted, allows duplicate elements, and supports positional (index-based) access.

**Technical Definition:** Per the Java API specification, "An ordered collection (also known as a sequence). The user of this interface has precise control over where in the list each element is inserted. The user can access elements by their integer index (position in the list), and search for elements in the list. Unlike sets, lists typically allow duplicate elements". The `List` interface extends `Collection<E>` and places additional stipulations on the contracts of the `iterator`, `add`, `remove`, `equals`, and `hashCode` methods. Lists are zero-based, meaning the first element is at index 0.

**Beginner-Friendly Explanation:** A `List` is like a numbered shopping list. Items are kept in the order you add them, you can have the same item multiple times, and you can ask for the item at position 3 (for example). Unlike a `Set` (which is like a bag of unique items), a `List` cares about order and allows duplicates.

### Key Characteristics

- **Ordered:** Elements are stored in the order they were inserted.
- **Index-Based Access:** Elements can be accessed, inserted, and removed by their integer position.
- **Duplicates Allowed:** The same element can appear multiple times in a list.
- **Zero-Based Indexing:** The first element is at index 0, the second at index 1, and so on.
- **Rich API:** The `List` interface provides methods for positional access, search, iteration, and range operations.
- **Multiple Implementations:** Common implementations include `ArrayList` (array-backed), `LinkedList` (linked-list-backed), and `CopyOnWriteArrayList` (thread-safe).

### Prerequisites

- Basic understanding of Java classes, objects, and interfaces
- Familiarity with generics (`List<String>`, `List<Integer>`, etc.)
- Knowledge of the `Collection` interface and the for-each loop
- Awareness of the `java.util` package

### Related Programming Areas

- **Collections Framework:** `List` is a core interface in the Java Collections Framework.
- **Stream API:** Lists integrate seamlessly with Java 8 Streams for functional-style processing.
- **Concurrency:** `CopyOnWriteArrayList` provides thread-safe list operations.
- **Algorithms:** The `Collections` utility class provides sorting, searching, and shuffling algorithms for lists.
- **GUI Programming:** Lists model items in list boxes, combo boxes, and tables.

---

## Core Concepts / Features

---

## 1. List Interface

### Definitions

**Core Definition:** The `List` interface is the abstract data type that defines the contract for ordered collections in Java.

**Technical Definition:** `List<E>` extends `Collection<E>` and adds positional access, search, iteration, and range-view operations. It provides four methods for positional (indexed) access: `get(int index)`, `set(int index, E element)`, `add(int index, E element)`, and `remove(int index)`. It also provides a special iterator, `ListIterator`, that allows bidirectional traversal and element modification.

**Beginner-Friendly Explanation:** The `List` interface is the "rulebook" that all list types follow. It says what you can do with a list — add items, get items by position, remove items, search for items, and so on.

### Purposes

- To define a standard contract for ordered collections
- To enable polymorphic use of different list implementations
- To provide positional access and search operations
- To support bidirectional iteration and range operations

### Syntax Rules and Structure

```java
public interface List<E> extends Collection<E> {
    // Positional access
    E get(int index);
    E set(int index, E element);
    void add(int index, E element);
    E remove(int index);

    // Search
    int indexOf(Object o);
    int lastIndexOf(Object o);

    // Iteration
    ListIterator<E> listIterator();
    ListIterator<E> listIterator(int index);

    // Range-view
    List<E> subList(int fromIndex, int toIndex);
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `get(int index)` | Returns the element at the specified position. |
| `set(int index, E element)` | Replaces the element at the position. |
| `add(int index, E element)` | Inserts an element at the position. |
| `remove(int index)` | Removes and returns the element at the position. |
| `indexOf(Object o)` | Returns the first index of the element, or `-1`. |
| `lastIndexOf(Object o)` | Returns the last index of the element, or `-1`. |
| `listIterator()` | Returns a bidirectional iterator. |
| `subList(int from, int to)` | Returns a view of the list between indices. |

**Syntax Rules:**

- Indices are zero-based.
- `List` is a generic interface (`List<E>`).
- `List` extends `Collection`, so all `Collection` methods are available.

**Constraints and Limitations:**

- Positional access may be O(1) for `ArrayList` but O(n) for `LinkedList`.
- Search operations may perform costly linear searches in many implementations.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic List Operations**

```java
// File: ListInterfaceDemo.java
import java.util.*;

public class ListInterfaceDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();

        // Adding elements
        list.add("Apple");
        list.add("Banana");
        list.add("Cherry");
        list.add(1, "Apricot"); // Insert at index 1
        System.out.println("List: " + list);

        // Positional access
        System.out.println("Element at index 2: " + list.get(2));

        // Search
        System.out.println("Index of Cherry: " + list.indexOf("Cherry"));
        System.out.println("Last index of Apple: " + list.lastIndexOf("Apple"));

        // Sub-list
        List<String> sub = list.subList(1, 3);
        System.out.println("Sub-list: " + sub);

        // Duplicates allowed
        list.add("Apple");
        System.out.println("With duplicate: " + list);
    }
}
```

**Expected Output:**
```
List: [Apple, Apricot, Banana, Cherry]
Element at index 2: Banana
Index of Cherry: 3
Last index of Apple: 0
Sub-list: [Apricot, Banana]
With duplicate: [Apple, Apricot, Banana, Cherry, Apple]
```

**Step-by-Step Explanation:**

1. `list.add("Apple")` appends an element.
2. `list.add(1, "Apricot")` inserts at index 1, shifting elements to the right.
3. `list.get(2)` retrieves the element at index 2 (zero-based).
4. `list.indexOf("Cherry")` returns the first index of "Cherry".
5. `list.subList(1, 3)` returns a view from index 1 (inclusive) to 3 (exclusive).
6. `list.add("Apple")` demonstrates that duplicates are allowed.

### Real-World Cases

- **Playlist:** A `List<Song>` maintains the order of songs.
- **To-Do List:** A `List<Task>` where order matters.
- **Undo History:** A `List<Command>` for undo/redo operations.

---

## 2. ArrayList

### Definitions

**Core Definition:** `ArrayList` is a resizable-array implementation of the `List` interface that provides fast random access and is the most commonly used list implementation.

**Technical Definition:** `ArrayList<E>` extends `AbstractList<E>` and implements `List<E>`, `RandomAccess`, `Cloneable`, and `Serializable`. It is backed by a dynamically resizable array. "Most of the time, you'll probably use ArrayList, which offers constant-time positional access and is just plain fast. It does not have to allocate a node object for each element in the List, and it can take advantage of System.arraycopy when it has to move multiple elements at the same time".

**Beginner-Friendly Explanation:** `ArrayList` is like a stretchy array. You can add items and it grows automatically. It's very fast when you want to get the item at a specific position.

### Purposes

- To provide fast, O(1) positional access
- To serve as the default general-purpose list implementation
- To offer a resizable alternative to arrays
- To enable efficient bulk operations using `System.arraycopy`

### Syntax Rules and Structure

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, Serializable {
    // Constructors
    public ArrayList();
    public ArrayList(int initialCapacity);
    public ArrayList(Collection<? extends E> c);

    // Key methods (inherited from List)
    public E get(int index);
    public boolean add(E e);
    public void add(int index, E element);
    public E remove(int index);
    public int size();
}
```

**Component Breakdown:**

| Constructor | Description |
|-------------|-------------|
| `ArrayList()` | Creates an empty list with default capacity (10). |
| `ArrayList(int initialCapacity)` | Creates an empty list with the specified capacity. |
| `ArrayList(Collection<? extends E> c)` | Creates a list containing the elements of the collection. |

**Syntax Rules:**

- `ArrayList` is generic (`ArrayList<E>`).
- It implements `RandomAccess`, so index-based access is fast.
- Initial capacity can be specified to avoid resizing.

**Constraints and Limitations:**

- Insertion and removal at arbitrary positions are O(n) because elements must be shifted.
- Not thread-safe; use `Collections.synchronizedList()` or `CopyOnWriteArrayList` for concurrent access.

### Annotated Code Example

```java
// File: ArrayListDemo.java
import java.util.*;

public class ArrayListDemo {
    public static void main(String[] args) {
        // Default constructor
        List<String> list = new ArrayList<>();

        list.add("Apple");
        list.add("Banana");
        list.add("Cherry");
        System.out.println("List    : " + list);


        // With initial capacity
        List<Integer> numbers = new ArrayList<>(100);
        for (int i = 1; i <= 5; i++) {
            numbers.add(i);
        }
        System.out.println("Numbers : " + numbers);


        // From another collection
        List<String> copy = new ArrayList<>(list);
        System.out.println("Copy    : " + copy);
    }
}
```

**Expected Output:**
```
List    : [Apple, Banana, Cherry]
Numbers : [1, 2, 3, 4, 5]
Copy    : [Apple, Banana, Cherry]
```

**Step-by-Step Explanation:**

1. `new ArrayList<>()` creates an empty list with default capacity.
2. `new ArrayList<>(100)` creates a list with initial capacity 100, avoiding resizing for up to 100 elements.
3. `new ArrayList<>(list)` creates a copy of the existing list.

### Real-World Cases

- **General-Purpose Storage:** The default choice for most list needs.
- **Random Access:** Scenarios requiring frequent `get(int index)` calls.
- **Caching:** Storing recently accessed items.

---

## 3. LinkedList

### Definitions

**Core Definition:** `LinkedList` is a doubly-linked list implementation of the `List` and `Deque` interfaces that provides efficient insertion and removal at both ends.

**Technical Definition:** `LinkedList<E>` extends `AbstractSequentialList<E>` and implements `List<E>`, `Deque<E>`, `Cloneable`, and `Serializable`. "If you frequently add elements to the beginning of the List or iterate over the List to delete elements from its interior, you should consider using LinkedList. These operations require constant-time in a LinkedList and linear-time in an ArrayList. But you pay a big price in performance. Positional access requires linear-time in a LinkedList and constant-time in an ArrayList".

**Beginner-Friendly Explanation:** `LinkedList` is like a chain of linked nodes. Each node points to the next and previous node. It's fast when you need to add or remove items from the beginning or end, but slow when you need to find the item at position 100.

### Purposes

- To provide efficient O(1) insertion and removal at both ends
- To serve as a double-ended queue (`Deque`)
- To implement stacks and queues
- To offer a list implementation with no capacity tuning parameters

### Syntax Rules and Structure

```java
public class LinkedList<E> extends AbstractSequentialList<E>
        implements List<E>, Deque<E>, Cloneable, Serializable {
    // Constructors
    public LinkedList();
    public LinkedList(Collection<? extends E> c);

    // Additional methods from Deque
    public void addFirst(E e);
    public void addLast(E e);
    public E removeFirst();
    public E removeLast();
    public E getFirst();
    public E getLast();
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `addFirst(E e)` | Adds an element at the front. |
| `addLast(E e)` | Adds an element at the end. |
| `removeFirst()` | Removes and returns the first element. |
| `removeLast()` | Removes and returns the last element. |
| `getFirst()` | Returns the first element without removing. |
| `getLast()` | Returns the last element without removing. |

**Syntax Rules:**

- `LinkedList` is generic (`LinkedList<E>`).
- It implements both `List` and `Deque`.
- It has no initial capacity tuning parameter.

**Constraints and Limitations:**

- Positional access is O(n) because it requires traversing the list.
- Iteration is slower than `ArrayList` due to pointer chasing and CPU cache misses.
- `LinkedList` almost never outperforms `ArrayList` or `ArrayDeque` in practice.

### Annotated Code Example

```java
// File: LinkedListDemo.java
import java.util.*;

public class LinkedListDemo {
    public static void main(String[] args) {
        LinkedList<String> list = new LinkedList<>();

        // Adding at both ends
        list.addFirst("First");
        list.addLast("Last");
        list.add("Middle"); // Same as addLast
        System.out.println("List: " + list);

        // Accessing ends
        System.out.println("First: " + list.getFirst());
        System.out.println("Last: " + list.getLast());

        // Removing from ends
        list.removeFirst();
        list.removeLast();
        System.out.println("After removals: " + list);
    }
}
```

**Expected Output:**
```
List: [First, Middle, Last]
First: First
Last: Last
After removals: [Middle]
```

**Step-by-Step Explanation:**

1. `addFirst("First")` inserts at the front.
2. `addLast("Last")` appends at the end.
3. `add("Middle")` appends (same as `addLast`).
4. `getFirst()` and `getLast()` retrieve the ends without removing.
5. `removeFirst()` and `removeLast()` remove from the ends.

### Real-World Cases

- **Stack Implementation:** Use `LinkedList` as a LIFO stack (though `ArrayDeque` is preferred).
- **Queue Implementation:** Use `LinkedList` as a FIFO queue (though `ArrayDeque` is preferred).
- **Undo/Redo:** Store operations at both ends.

---

## 4. Vector (legacy)

### Definitions

**Core Definition:** `Vector` is a legacy, synchronized implementation of the `List` interface that predates the Java Collections Framework.

**Technical Definition:** `Vector<E>` extends `AbstractList<E>` and implements `List<E>`, `RandomAccess`, `Cloneable`, and `Serializable`. "Vector performs synchronization that is usually unnecessary; prefer ArrayList. If a synchronized collection is necessary, use Collections.synchronizedList or a data structure from java.util.concurrent".

**Beginner-Friendly Explanation:** `Vector` is an older version of `ArrayList` that is thread-safe. However, the synchronization is often unnecessary and slows things down. It's better to use `ArrayList` and synchronize only when needed.

### Purposes

- **Legacy.** To provide a thread-safe list implementation (now largely replaced by `Collections.synchronizedList()` and concurrent collections)
- To maintain backward compatibility with pre-Collections code

### Syntax Rules and Structure

```java
public class Vector<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, Serializable {
    public Vector();
    public Vector(int initialCapacity);
    public Vector(int initialCapacity, int capacityIncrement);
    public Vector(Collection<? extends E> c);
}
```

**Syntax Rules:**

- `Vector` is synchronized, so all methods are thread-safe.
- It is largely obsolete; use `ArrayList` with `Collections.synchronizedList()` if synchronization is needed.

**Constraints and Limitations:**

- Synchronization overhead makes it slower than `ArrayList`.
- It has many legacy methods not in the `List` interface.

### Annotated Code Example

```java
// File: VectorDemo.java
import java.util.*;

public class VectorDemo {
    public static void main(String[] args) {
        Vector<String> vector = new Vector<>();
        vector.add("Apple");
        vector.add("Banana");
        System.out.println("Vector: " + vector);
        System.out.println("Size: " + vector.size());
    }
}
```

**Expected Output:**
```
Vector: [Apple, Banana]
Size: 2
```

**Step-by-Step Explanation:**

1. `Vector` is instantiated like `ArrayList`.
2. `add()` is synchronized, so it is thread-safe but slower.
3. The output shows the vector contents.

### Real-World Cases

- **Legacy Code:** Maintaining old codebases that use `Vector`.
- **Rare Thread-Safety Needs:** When a synchronized list is required and `Collections.synchronizedList()` is not practical.

---

## 5. Stack (legacy)

### Definitions

**Core Definition:** `Stack` is a legacy, synchronized implementation of a last-in, first-out (LIFO) stack that extends `Vector`.

**Technical Definition:** `Stack<E>` extends `Vector<E>` and provides `push()`, `pop()`, and `peek()` operations. "Stack is a nonstandard class that predates the Java Collections Framework; prefer ArrayDeque. If a synchronized collection is necessary, use a data structure from java.util.concurrent".

**Beginner-Friendly Explanation:** `Stack` is like a stack of plates — you add plates on top and remove them from the top. It's an older class; `ArrayDeque` is now recommended instead.

### Purposes

- **Legacy.** To provide a LIFO stack data structure
- To maintain backward compatibility

### Syntax Rules and Structure

```java
public class Stack<E> extends Vector<E> {
    public Stack();
    public E push(E item);
    public E pop();
    public E peek();
    public boolean empty();
    public int search(Object o);
}
```

**Syntax Rules:**

- `push()` adds to the top of the stack.
- `pop()` removes and returns the top element.
- `peek()` returns the top element without removing.
- `empty()` returns `true` if the stack is empty.

**Constraints and Limitations:**

- Extends `Vector`, so it carries synchronization overhead.
- Prefer `ArrayDeque` for stack operations.

### Annotated Code Example

```java
// File: StackDemo.java
import java.util.*;

public class StackDemo {
    public static void main(String[] args) {
        Stack<String> stack = new Stack<>();
        stack.push("First");
        stack.push("Second");
        stack.push("Third");
        System.out.println("Stack: " + stack);
        System.out.println("Peek: " + stack.peek());
        System.out.println("Pop: " + stack.pop());
        System.out.println("After pop: " + stack);
    }
}
```

**Expected Output:**
```
Stack: [First, Second, Third]
Peek: Third
Pop: Third
After pop: [First, Second]
```

**Step-by-Step Explanation:**

1. `push("First")` adds to the top of the stack.
2. `peek()` returns "Third" without removing.
3. `pop()` removes and returns "Third".
4. The remaining stack is `[First, Second]`.

### Real-World Cases

- **Legacy Code:** Old applications that use `Stack`.
- **Expression Evaluation:** Parsing arithmetic expressions (though `ArrayDeque` is preferred).

---

## 6. CopyOnWriteArrayList

### Definitions

**Core Definition:** `CopyOnWriteArrayList` is a thread-safe variant of `ArrayList` in which all mutative operations are implemented by making a fresh copy of the underlying array.

**Technical Definition:** "A thread-safe variant of ArrayList in which all mutative operations (add, set, and so on) are implemented by making a fresh copy of the underlying array. This is ordinarily too costly, but may be more efficient than alternatives when traversal operations vastly outnumber mutations, and is useful when you cannot or don't want to synchronize traversals, yet need to preclude interference among concurrent threads".

**Beginner-Friendly Explanation:** `CopyOnWriteArrayList` is like a list that makes a photocopy every time you change it. This makes it very safe for reading from multiple threads at the same time, but slow if you change it often.

### Purposes

- To provide thread-safe list operations without explicit synchronization
- To enable safe iteration without `ConcurrentModificationException`
- To support scenarios where reads vastly outnumber writes

### Syntax Rules and Structure

```java
public class CopyOnWriteArrayList<E>
        implements List<E>, RandomAccess, Cloneable, Serializable {
    public CopyOnWriteArrayList();
    public CopyOnWriteArrayList(Collection<? extends E> c);
    public CopyOnWriteArrayList(E[] toCopyIn);
}
```

**Syntax Rules:**

- All mutative operations create a new copy of the underlying array.
- Iterators are snapshot-based and never throw `ConcurrentModificationException`.
- Element-changing operations on iterators (`remove`, `set`, `add`) are not supported.

**Constraints and Limitations:**

- Mutations are expensive because the entire array is copied.
- Best suited for read-heavy, write-rare scenarios.

### Annotated Code Example

```java
// File: CopyOnWriteArrayListDemo.java
import java.util.concurrent.*;

public class CopyOnWriteArrayListDemo {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
        list.add("Apple");
        list.add("Banana");
        list.add("Cherry");

        // Iteration is safe even if another thread modifies the list
        for (String item : list) {
            System.out.println(item);
            // list.add("Date"); // Would not affect this iteration
        }
        System.out.println("List: " + list);
    }
}
```

**Expected Output:**
```
Apple
Banana
Cherry
List: [Apple, Banana, Cherry]
```

**Step-by-Step Explanation:**

1. `CopyOnWriteArrayList` is created and populated.
2. The for-each loop iterates over a snapshot of the list.
3. Even if another thread modifies the list during iteration, the iterator sees the original snapshot.
4. The output shows the list contents.

### Real-World Cases

- **Event Listeners:** Maintaining a list of listeners that are iterated frequently but modified rarely.
- **Configuration Lists:** Read-mostly lists of settings or properties.
- **Caching:** Lists of cached items that are occasionally refreshed.

---

## 7. List Operations

### Definitions

**Core Definition:** List operations are the methods provided by the `List` interface and its implementations for manipulating and accessing list elements.

**Technical Definition:** The `List` interface provides methods for positional access (`get`, `set`, `add`, `remove`), search (`indexOf`, `lastIndexOf`), iteration (`iterator`, `listIterator`), range operations (`subList`), and bulk operations (`addAll`, `removeAll`, `retainAll`). Java 8 added default methods `sort`, `replaceAll`, and `forEach`.

**Beginner-Friendly Explanation:** List operations are the actions you can perform on a list — adding items, getting items, removing items, searching, sorting, and more.

### add()

**Definition:** Appends or inserts an element into the list.

**Syntax:**
```java
boolean add(E e);              // Appends to the end
void add(int index, E element); // Inserts at index
```

**Example:**
```java
List<String> list = new ArrayList<>();
list.add("Apple");        // [Apple]
list.add(0, "Banana");    // [Banana, Apple]
```

### get()

**Definition:** Returns the element at the specified position.

**Syntax:**
```java
E get(int index);
```

**Example:**
```java
String first = list.get(0);
```

### set()

**Definition:** Replaces the element at the specified position.

**Syntax:**
```java
E set(int index, E element);
```

**Example:**
```java
list.set(0, "Cherry"); // Replaces element at index 0
```

### remove()

**Definition:** Removes the first occurrence of an element or the element at a specified index.

**Syntax:**
```java
boolean remove(Object o);   // Removes first occurrence
E remove(int index);        // Removes at index
```

**Example:**
```java
list.remove("Apple");   // Removes first "Apple"
list.remove(0);         // Removes element at index 0
```

### indexOf() / lastIndexOf()

**Definition:** Returns the first or last index of the specified element, or `-1` if not found.

**Syntax:**
```java
int indexOf(Object o);
int lastIndexOf(Object o);
```

**Example:**
```java
int first = list.indexOf("Apple");
int last = list.lastIndexOf("Apple");
```

### contains()

**Definition:** Returns `true` if the list contains the specified element.

**Syntax:**
```java
boolean contains(Object o);
```

**Example:**
```java
boolean hasApple = list.contains("Apple");
```

### size() / isEmpty()

**Definition:** Returns the number of elements or checks if the list is empty.

**Syntax:**
```java
int size();
boolean isEmpty();
```

**Example:**
```java
int count = list.size();
boolean empty = list.isEmpty();
```

### clear()

**Definition:** Removes all elements from the list.

**Syntax:**
```java
void clear();
```

**Example:**
```java
list.clear(); // List is now empty
```

### subList()

**Definition:** Returns a view of the portion of the list between `fromIndex` (inclusive) and `toIndex` (exclusive).

**Syntax:**
```java
List<E> subList(int fromIndex, int toIndex);
```

**Example:**
```java
List<String> sub = list.subList(1, 3); // Elements at indices 1 and 2
```

### sort()

**Definition:** Sorts the list according to the order induced by the specified `Comparator`.

**Syntax:**
```java
default void sort(Comparator<? super E> c);
```

**Example:**
```java
list.sort(String::compareTo); // Natural order
```

### replaceAll()

**Definition:** Replaces each element with the result of applying the operator.

**Syntax:**
```java
default void replaceAll(UnaryOperator<E> operator);
```

**Example:**
```java
list.replaceAll(s -> s.toUpperCase()); // Uppercase all elements
```

### forEach()

**Definition:** Performs the given action for each element.

**Syntax:**
```java
default void forEach(Consumer<? super E> action);
```

**Example:**
```java
list.forEach(System.out::println); // Print each element
```

### iterator() / listIterator()

**Definition:** Returns an iterator or list iterator for traversing the list.

**Syntax:**
```java
Iterator<E> iterator();
ListIterator<E> listIterator();
ListIterator<E> listIterator(int index);
```

**Example:**
```java
ListIterator<String> it = list.listIterator();
while (it.hasNext()) {
    String s = it.next();
    if (s.equals("Banana")) it.set("Mango"); // Replace
}
```

### toArray()

**Definition:** Returns an array containing all elements in the list.

**Syntax:**
```java
Object[] toArray();
<T> T[] toArray(T[] a);
```

**Example:**
```java
String[] arr = list.toArray(new String[0]);
```

### stream() / parallelStream()

**Definition:** Returns a sequential or parallel stream with the list as its source.

**Syntax:**
```java
default Stream<E> stream();
default Stream<E> parallelStream();
```

**Example:**
```java
list.stream().filter(s -> s.startsWith("A")).forEach(System.out::println);
```

---

## 8. List Implementations Comparison

| Feature | ArrayList | LinkedList | Vector | Stack | CopyOnWriteArrayList |
|---------|-----------|------------|--------|-------|---------------------|
| **Backing Structure** | Resizable array | Doubly-linked list | Resizable array | Extends Vector | Copy-on-write array |
| **Thread-Safe** | No | No | Yes (synchronized) | Yes (synchronized) | Yes (copy-on-write) |
| **Random Access** | O(1) | O(n) | O(1) | O(1) | O(1) |
| **Insert/Remove at End** | O(1) amortized | O(1) | O(1) amortized | O(1) | O(n) |
| **Insert/Remove at Index** | O(n) | O(n) | O(n) | O(n) | O(n) |
| **Iteration** | Fast | Slow | Fast | Fast | Snapshot-based |
| **Duplicate Elements** | Yes | Yes | Yes | Yes | Yes |
| **Null Elements** | Yes | Yes | Yes | Yes | Yes |
| **Recommended** | Default choice | Rarely; prefer ArrayList or ArrayDeque | Legacy; prefer ArrayList | Legacy; prefer ArrayDeque | Read-heavy, write-rare |
| **Since** | 1.2 | 1.2 | 1.0 | 1.0 | 1.5 |

**Key Takeaways:**

- `ArrayList` is usually the best choice for general-purpose lists.
- `LinkedList` almost never outperforms `ArrayList` or `ArrayDeque`; prefer them instead.
- `Vector` and `Stack` are legacy classes; prefer `ArrayList` and `ArrayDeque` respectively.
- `CopyOnWriteArrayList` is best for read-heavy, write-rare concurrent scenarios.

---

## 9. List Best Practices

### Definitions

**Core Definition:** List best practices are guidelines for choosing and using list implementations effectively.

**Technical Definition:** Best practices include programming to the `List` interface, choosing the right implementation for the use case, specifying initial capacity for `ArrayList`, using `List.of()` for immutable lists, and avoiding legacy classes like `Vector` and `Stack`.

**Beginner-Friendly Explanation:** Best practices help you write list code that is fast, safe, and easy to maintain.

### Key Best Practices

| Practice | Description |
|----------|-------------|
| **Program to interfaces** | Declare variables as `List`, not `ArrayList`. |
| **Choose the right implementation** | Use `ArrayList` for general use; `CopyOnWriteArrayList` for read-heavy concurrent use. |
| **Specify initial capacity** | For `ArrayList`, set an initial capacity to avoid resizing. |
| **Use `List.of()` for immutable lists** | Java 9+ provides `List.of()` for unmodifiable lists. |
| **Prefer `ArrayDeque` over `Stack`** | `Stack` is legacy; `ArrayDeque` is faster. |
| **Prefer `ArrayList` over `Vector`** | `Vector` is synchronized unnecessarily; use `Collections.synchronizedList()` if needed. |
| **Use `isEmpty()` instead of `size() == 0`** | `isEmpty()` is clearer and may be more efficient. |
| **Use `ListIterator` for bidirectional traversal** | `ListIterator` supports both directions and element modification. |
| **Be careful with `subList()`** | `subList()` returns a view, not a copy; changes to the view affect the original. |
| **Use streams for bulk operations** | `stream()`, `filter()`, `map()`, `collect()` for functional-style processing. |

### Annotated Code Example

```java
// File: ListBestPractices.java
import java.util.*;

public class ListBestPractices {
    public static void main(String[] args) {
        // GOOD: Program to interface
        List<String> list = new ArrayList<>(20); // Initial capacity

        // GOOD: Use List.of() for immutable lists
        List<String> immutable = List.of("A", "B", "C");
        // immutable.add("D"); // UnsupportedOperationException

        // GOOD: Use isEmpty()
        if (!list.isEmpty()) {
            System.out.println("List has elements");
        }

        // GOOD: Use ListIterator for bidirectional traversal
        list.add("Apple");
        list.add("Banana");
        list.add("Cherry");

        ListIterator<String> it = list.listIterator(list.size());
        while (it.hasPrevious()) {
            System.out.println(it.previous()); // Reverse order
        }

        // GOOD: Use streams for bulk operations
        list.stream()
            .filter(s -> s.startsWith("A"))
            .forEach(System.out::println); // Prints "Apple"
    }
}
```

**Expected Output:**
```
Cherry
Banana
Apple
Apple
```

**Step-by-Step Explanation:**

1. `new ArrayList<>(20)` specifies an initial capacity to avoid resizing.
2. `List.of("A", "B", "C")` creates an immutable list.
3. `list.isEmpty()` is used instead of `list.size() == 0`.
4. `list.listIterator(list.size())` creates a `ListIterator` at the end for reverse traversal.
5. `list.stream().filter(...)` demonstrates functional-style processing.

### Real-World Cases

- **API Design:** Returning `List` from methods instead of `ArrayList`.
- **Performance Tuning:** Specifying initial capacity for large lists.
- **Immutable APIs:** Using `List.of()` to protect internal state.
- **Concurrent Systems:** Using `CopyOnWriteArrayList` for event listener lists.

---

## References Links

- List (Java SE 17 & JDK 17) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/List.html
- List (Java Platform SE 8) — Oracle API Documentation - https://docs.oracle.com/javase/8/docs/api/java/util/List.html
- List Implementations — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/collections/implementations/list.html
- ListIterator (Java SE 26) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/ListIterator.html
- CopyOnWriteArrayList (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CopyOnWriteArrayList.html
- Creating Unmodifiable Lists, Sets, and Maps — Oracle Core Libraries - https://docs.oracle.com/en/java/javase/17/core/creating-unmodifiable-lists-sets-and-maps.html
- JdkObsolete — Error Prone Bug Patterns - https://github.com/google/error-prone/blob/master/docs/bugpattern/JdkObsolete.md
- Some Java List Benchmarks — Stuart Marks (Oracle) - https://stuartmarks.wordpress.com/2015/12/18/some-java-list-benchmarks/
- Choosing the Right Implementation Between ArrayList and LinkedList — Dev.java - https://dev.java/learn/api/collections-framework/implementations/
- Extending Collection with List — Dev.java - https://dev.java/learn/api/collections-framework/lists/