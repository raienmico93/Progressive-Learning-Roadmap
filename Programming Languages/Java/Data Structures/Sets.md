# Java Sets: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A `Set` is a collection that contains no duplicate elements, modeling the mathematical set abstraction.

**Technical Definition:** Per the Java API specification, "A collection that contains no duplicate elements. More formally, sets contain no pair of elements e1 and e2 such that e1.equals(e2), and at most one null element. As implied by its name, this interface models the mathematical set abstraction". The `Set` interface extends `Collection<E>` and places additional stipulations on the contracts of `add`, `equals`, and `hashCode`. Some set implementations prohibit null elements, and some have restrictions on the types of their elements.

**Beginner-Friendly Explanation:** A `Set` is like a bag of unique items. You can put things in, take things out, and check if something is inside — but you cannot have two of the same item. If you try to add a duplicate, it is silently ignored. This is perfect for tracking things like unique visitors to a website, tags on a blog post, or a deck of cards.

### Key Characteristics

- **No Duplicates:** Sets contain no pair of elements `e1` and `e2` such that `e1.equals(e2)`.
- **At Most One Null:** Most sets permit at most one `null` element, though some (like `TreeSet`, `EnumSet`, and `ConcurrentSkipListSet`) prohibit nulls entirely.
- **Mathematical Set Abstraction:** Sets model the mathematical concept of a set, supporting operations like union, intersection, and difference.
- **Mutable Object Warning:** Great care must be exercised if mutable objects are used as set elements. The behaviour of a set is not specified if the value of an object is changed in a manner that affects `equals` comparisons while it is in the set.
- **Unmodifiable Sets:** `Set.of()` and `Set.copyOf()` provide convenient ways to create unmodifiable sets that disallow nulls and reject duplicates at creation time.

### Prerequisites

- Basic understanding of Java classes, objects, and interfaces
- Familiarity with generics (`Set<String>`, `Set<Integer>`, etc.)
- Knowledge of the `Collection` interface and the for-each loop
- Awareness of the `java.util` and `java.util.concurrent` packages
- Understanding of `equals()` and `hashCode()` contracts (critical for correct Set behaviour)

### Related Programming Areas

- **Collections Framework:** `Set` is a core interface in the Java Collections Framework.
- **Stream API:** Sets integrate seamlessly with Java 8 Streams for functional-style processing.
- **Concurrency:** `CopyOnWriteArraySet` and `ConcurrentSkipListSet` provide thread-safe set operations.
- **Algorithms:** Sets are used in algorithms for deduplication, membership testing, and set operations.
- **Database Operations:** Sets model DISTINCT query results and unique constraints.

---

## Core Concepts / Features

---

## 1. Set Interface

### Definitions

**Core Definition:** The `Set` interface is the abstract data type that defines the contract for collections that contain no duplicate elements.

**Technical Definition:** `Set<E>` extends `Collection<E>` and is a member of the Java Collections Framework. It places additional stipulations on constructors (must create a set with no duplicates) and on the `add`, `equals`, and `hashCode` methods. Set implementations may prohibit null elements, and attempting to add an ineligible element throws an unchecked exception, typically `NullPointerException` or `ClassCastException`.

**Beginner-Friendly Explanation:** The `Set` interface is the rulebook for all set types. It says: "You can add items, remove items, check if an item exists, and combine sets — but you cannot have duplicates."

### Purposes

- To define a standard contract for collections with no duplicate elements
- To enable polymorphic use of different set implementations
- To support mathematical set operations (union, intersection, difference)
- To provide efficient membership testing

### Syntax Rules and Structure

```java
public interface Set<E> extends Collection<E> {
    // Inherits all Collection methods
    // Key methods:
    boolean add(E e);              // Adds if not already present
    boolean remove(Object o);      // Removes if present
    boolean contains(Object o);    // Checks membership
    boolean addAll(Collection<? extends E> c);   // Union
    boolean retainAll(Collection<?> c);          // Intersection
    boolean removeAll(Collection<?> c);          // Difference
    boolean containsAll(Collection<?> c);        // Subset check
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `add(E e)` | Adds the element if not already present. |
| `remove(Object o)` | Removes the element if present. |
| `contains(Object o)` | Returns `true` if the element is present. |
| `addAll(Collection)` | Adds all elements (union). |
| `retainAll(Collection)` | Keeps only elements in both sets (intersection). |
| `removeAll(Collection)` | Removes all elements in the given collection (difference). |
| `containsAll(Collection)` | Returns `true` if the set contains all elements (subset check). |

**Syntax Rules:**

- `Set` is a generic interface (`Set<E>`).
- `Set` extends `Collection`, so all `Collection` methods are available.
- Duplicate elements are silently ignored by `add()`.

**Constraints and Limitations:**

- Set makes no guarantees about iteration order (except `LinkedHashSet`, `TreeSet`, and `EnumSet`).
- Mutable elements can cause issues if their hash code changes after insertion.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Set Operations**

```java
// File: SetInterfaceDemo.java
import java.util.*;

public class SetInterfaceDemo {
    public static void main(String[] args) {
        Set<String> set = new HashSet<>();

        // Adding elements
        set.add("Apple");
        set.add("Banana");
        set.add("Cherry");
        set.add("Apple"); // Duplicate — ignored
        System.out.println("Set: " + set);
        System.out.println("Size: " + set.size());

        // Membership testing
        System.out.println("Contains Banana? " + set.contains("Banana"));

        // Removing elements
        set.remove("Banana");
        System.out.println("After removal: " + set);

        // Set operations
        Set<String> other = new HashSet<>(Arrays.asList("Banana", "Date"));

        // Union
        Set<String> union = new HashSet<>(set);
        union.addAll(other);
        System.out.println("Union: " + union);

        // Intersection
        Set<String> intersection = new HashSet<>(set);
        intersection.retainAll(other);
        System.out.println("Intersection: " + intersection);

        // Difference
        Set<String> difference = new HashSet<>(set);
        difference.removeAll(other);
        System.out.println("Difference: " + difference);
    }
}
```

**Expected Output:**
```
Set: [Apple, Banana, Cherry]
Size: 3
Contains Banana? true
After removal: [Apple, Cherry]
Union: [Apple, Banana, Cherry, Date]
Intersection: [Banana]
Difference: [Apple, Cherry]
```

**Step-by-Step Explanation:**

1. `set.add("Apple")` adds "Apple".
2. Adding "Apple" again is ignored because `Set` does not allow duplicates.
3. `set.contains("Banana")` returns `true`.
4. `set.remove("Banana")` removes "Banana".
5. `union.addAll(other)` combines both sets.
6. `intersection.retainAll(other)` keeps only elements present in both.
7. `difference.removeAll(other)` removes elements present in `other`.

### Real-World Cases

- **Unique Tags:** A `Set<String>` stores unique tags on a blog post.
- **IP Blacklist:** A `Set<String>` for fast membership checks.
- **Duplicate Removal:** Converting a `List` to a `Set` to remove duplicates.
- **Set Operations:** Finding common interests between users.

---

## 2. HashSet

### Definitions

**Core Definition:** `HashSet` is the most commonly used `Set` implementation, backed by a hash table (actually a `HashMap` instance).

**Technical Definition:** "This class implements the Set interface, backed by a hash table (actually a HashMap instance). It makes no guarantees as to the iteration order of the set; in particular, it does not guarantee that the order will remain constant over time. This class permits the null element. This class offers constant time performance for the basic operations (add, remove, contains and size), assuming the hash function disperses the elements properly among the buckets".

**Beginner-Friendly Explanation:** `HashSet` is the default, fastest set. It uses a hash table to store items, which makes adding, removing, and checking items very fast. It does not keep items in any particular order.

### Purposes

- To provide fast, constant-time performance for basic set operations
- To serve as the default general-purpose set implementation
- To store unique elements without concern for ordering

### Syntax Rules and Structure

```java
public class HashSet<E> extends AbstractSet<E>
        implements Set<E>, Cloneable, Serializable {
    public HashSet();
    public HashSet(int initialCapacity);
    public HashSet(int initialCapacity, float loadFactor);
    public HashSet(Collection<? extends E> c);
}
```

**Component Breakdown:**

| Constructor | Description |
|-------------|-------------|
| `HashSet()` | Creates an empty set with default capacity (16) and load factor (0.75). |
| `HashSet(int initialCapacity)` | Creates an empty set with specified initial capacity. |
| `HashSet(Collection<? extends E> c)` | Creates a set containing the elements of the collection. |

**Syntax Rules:**

- `HashSet` is generic (`HashSet<E>`).
- It permits the `null` element.
- It is not synchronized; for concurrent access, wrap with `Collections.synchronizedSet()`.

**Constraints and Limitations:**

- Makes no guarantees about iteration order.
- Iteration performance depends on initial capacity — do not set it too high.
- Fail-fast iterators throw `ConcurrentModificationException` if modified during iteration.

### Annotated Code Example

```java
// File: HashSetDemo.java
import java.util.*;

public class HashSetDemo {
    public static void main(String[] args) {
        Set<String> set = new HashSet<>();

        // Adding elements
        set.add("Apple");
        set.add("Banana");
        set.add("Cherry");
        set.add("Apple"); // Duplicate — ignored
        set.add(null);    // null is permitted

        System.out.println("Set: " + set);
        System.out.println("Size: " + set.size());
        System.out.println("Contains Banana? " + set.contains("Banana"));

        // Iteration (order is unspecified)
        for (String s : set) {
            System.out.println(s);
        }
    }
}
```

**Expected Output (order may vary):**
```
Set: [null, Apple, Banana, Cherry]
Size: 4
Contains Banana? true
null
Apple
Banana
Cherry
```

**Step-by-Step Explanation:**

1. `set.add("Apple")` adds "Apple".
2. Adding "Apple" again is ignored.
3. `set.add(null)` adds the `null` element (HashSet permits it).
4. The output shows the set contents and confirms membership testing.

### Real-World Cases

- **General-Purpose Storage:** The default choice for most set needs.
- **Membership Testing:** Fast `contains()` checks for whitelists/blacklists.
- **Deduplication:** Removing duplicates from a list.

---

## 3. LinkedHashSet

### Definitions

**Core Definition:** `LinkedHashSet` is a hash table and linked list implementation of the `Set` interface with predictable iteration order.

**Technical Definition:** "Hash table and linked list implementation of the Set interface, with well-defined encounter order. This implementation differs from HashSet in that it maintains a doubly-linked list running through all of its entries. This linked list defines the encounter order (iteration order), which is the order in which elements were inserted into the set (insertion-order). The least recently inserted element (the eldest) is first, and the youngest element is last".

**Beginner-Friendly Explanation:** `LinkedHashSet` is like `HashSet` but it remembers the order you added items. When you iterate over it, items come out in the order they were inserted.

### Purposes

- To provide a set with predictable, insertion-ordered iteration
- To combine the performance of `HashSet` with deterministic iteration
- To produce copies of sets that preserve order

### Syntax Rules and Structure

```java
public class LinkedHashSet<E> extends HashSet<E>
        implements Set<E>, Cloneable, Serializable {
    public LinkedHashSet();
    public LinkedHashSet(int initialCapacity);
    public LinkedHashSet(int initialCapacity, float loadFactor);
    public LinkedHashSet(Collection<? extends E> c);
}
```

**Syntax Rules:**

- `LinkedHashSet` extends `HashSet` and adds a linked list.
- It permits null elements.
- Iteration order is insertion order (first inserted to last inserted).
- Re-inserting an element does not change its position in the order.

**Constraints and Limitations:**

- Slightly slower than `HashSet` due to the added linked list.
- Iteration is proportional to the set's size, not its capacity.

### Annotated Code Example

```java
// File: LinkedHashSetDemo.java
import java.util.*;

public class LinkedHashSetDemo {
    public static void main(String[] args) {
        Set<String> set = new LinkedHashSet<>();

        set.add("Cherry");
        set.add("Apple");
        set.add("Banana");
        set.add("Apple"); // Duplicate — ignored, order unchanged

        // Iteration order is insertion order
        for (String s : set) {
            System.out.println(s);
        }
        System.out.println("Set: " + set);
    }
}
```

**Expected Output:**
```
Cherry
Apple
Banana
Set: [Cherry, Apple, Banana]
```

**Step-by-Step Explanation:**

1. `set.add("Cherry")` adds "Cherry" first.
2. `set.add("Apple")` adds "Apple" second.
3. `set.add("Banana")` adds "Banana" third.
4. Re-adding "Apple" is ignored and does not change its position.
5. Iteration follows insertion order: Cherry, Apple, Banana.

### Real-World Cases

- **Maintaining Order:** When you need to remember the order in which elements were added.
- **Copying Sets:** Preserving the order of an input set when copying it.
- **Caching:** Storing recently accessed items in order.

---

## 4. TreeSet

### Definitions

**Core Definition:** `TreeSet` is a `NavigableSet` implementation based on a `TreeMap` that stores elements in sorted order.

**Technical Definition:** "A NavigableSet implementation based on a TreeMap. The elements are ordered using their natural ordering, or by a Comparator provided at set creation time, depending on which constructor is used. This implementation provides guaranteed log(n) time cost for the basic operations (add, remove and contains)".

**Beginner-Friendly Explanation:** `TreeSet` keeps its elements sorted automatically. When you iterate over it, items come out in sorted order (either natural order or the order you specify with a comparator).

### Purposes

- To maintain elements in sorted order
- To provide navigation methods (first, last, headSet, tailSet, subSet)
- To implement ordered set operations

### Syntax Rules and Structure

```java
public class TreeSet<E> extends AbstractSet<E>
        implements NavigableSet<E>, Cloneable, Serializable {
    public TreeSet();
    public TreeSet(Comparator<? super E> comparator);
    public TreeSet(Collection<? extends E> c);
    public TreeSet(SortedSet<E> s);
}
```

**Syntax Rules:**

- Elements must implement `Comparable` (for natural ordering) or a `Comparator` must be provided.
- The ordering must be consistent with `equals` for correct `Set` behaviour.
- It does **not** permit null elements (a `TreeSet` with natural ordering throws `NullPointerException` when adding null).

**Constraints and Limitations:**

- Log(n) time for basic operations (slower than HashSet's constant time).
- Not synchronized; use `Collections.synchronizedSortedSet()` for concurrent access.
- Fail-fast iterators.

### Annotated Code Example

```java
// File: TreeSetDemo.java
import java.util.*;

public class TreeSetDemo {
    public static void main(String[] args) {
        TreeSet<Integer> set = new TreeSet<>();

        set.add(5);
        set.add(2);
        set.add(8);
        set.add(1);
        set.add(9);
        set.add(2); // Duplicate — ignored

        System.out.println("Set (sorted): " + set);
        System.out.println("First: " + set.first());
        System.out.println("Last: " + set.last());
        System.out.println("HeadSet (less than 5): " + set.headSet(5));
        System.out.println("TailSet (>= 5): " + set.tailSet(5));
        System.out.println("SubSet (2 to 8): " + set.subSet(2, 8));
    }
}
```

**Expected Output:**
```
Set (sorted): [1, 2, 5, 8, 9]
First: 1
Last: 9
HeadSet (less than 5): [1, 2]
TailSet (>= 5): [5, 8, 9]
SubSet (2 to 8): [2, 5]
```

**Step-by-Step Explanation:**

1. Elements are inserted: 5, 2, 8, 1, 9, 2.
2. The duplicate 2 is ignored.
3. The set stores elements in sorted order: [1, 2, 5, 8, 9].
4. `first()` returns 1, `last()` returns 9.
5. `headSet(5)` returns elements less than 5.
6. `tailSet(5)` returns elements greater than or equal to 5.
7. `subSet(2, 8)` returns elements from 2 (inclusive) to 8 (exclusive).

### Real-World Cases

- **Sorted Data:** Maintaining a sorted list of scores, prices, or names.
- **Range Queries:** Finding all elements within a range.
- **Leaderboards:** Keeping high scores sorted.

---

## 5. EnumSet

### Definitions

**Core Definition:** `EnumSet` is a specialized `Set` implementation for use with enum types, represented internally as bit vectors.

**Technical Definition:** "A specialized Set implementation for use with enum types. All of the elements in an enum set must come from a single enum type that is specified, explicitly or implicitly, when the set is created. Enum sets are represented internally as bit vectors. This representation is extremely compact and efficient".

**Beginner-Friendly Explanation:** `EnumSet` is a highly efficient set designed specifically for enum values. It uses bit manipulation behind the scenes, making it extremely fast and compact.

### Purposes

- To provide a highly efficient set implementation for enum types
- To serve as a typesafe alternative to traditional int-based "bit flags".
- To enable fast bulk operations on enum sets

### Syntax Rules and Structure

```java
public abstract class EnumSet<E extends Enum<E>> extends AbstractSet<E>
        implements Cloneable, Serializable {
    // Static factory methods:
    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType);
    public static <E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType);
    public static <E extends Enum<E>> EnumSet<E> of(E e);
    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2);
    public static <E extends Enum<E>> EnumSet<E> range(E from, E to);
    public static <E extends Enum<E>> EnumSet<E> copyOf(Collection<E> c);
}
```

**Syntax Rules:**

- `EnumSet` is abstract; instances are created via static factory methods.
- All elements must come from a single enum type.
- `EnumSet` does **not** permit null elements; adding null throws `NullPointerException`.
- The iterator traverses elements in their natural order (the order in which enum constants are declared).

**Constraints and Limitations:**

- Not synchronized; use `Collections.synchronizedSet()` for concurrent access.
- The iterator is weakly consistent and never throws `ConcurrentModificationException`.

### Annotated Code Example

```java
// File: EnumSetDemo.java
import java.util.*;

enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }

public class EnumSetDemo {
    public static void main(String[] args) {
        // Create an EnumSet with specific elements
        EnumSet<Day> weekdays = EnumSet.of(Day.MONDAY, Day.TUESDAY, Day.WEDNESDAY,
                                            Day.THURSDAY, Day.FRIDAY);
        System.out.println("Weekdays: " + weekdays);

        // Create an EnumSet with all elements
        EnumSet<Day> allDays = EnumSet.allOf(Day.class);
        System.out.println("All days: " + allDays);

        // Create an EnumSet with a range
        EnumSet<Day> weekend = EnumSet.range(Day.SATURDAY, Day.SUNDAY);
        System.out.println("Weekend: " + weekend);

        // Complement (all days not in weekdays)
        EnumSet<Day> weekend2 = EnumSet.complementOf(weekdays);
        System.out.println("Complement of weekdays: " + weekend2);
    }
}
```

**Expected Output:**
```
Weekdays: [MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY]
All days: [MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY]
Weekend: [SATURDAY, SUNDAY]
Complement of weekdays: [SATURDAY, SUNDAY]
```

**Step-by-Step Explanation:**

1. `EnumSet.of(...)` creates a set with the specified enum constants.
2. `EnumSet.allOf(Day.class)` creates a set with all constants.
3. `EnumSet.range(Day.SATURDAY, Day.SUNDAY)` creates a set with a range of constants.
4. `EnumSet.complementOf(weekdays)` creates a set with all constants not in `weekdays`.

### Real-World Cases

- **Permissions:** Modeling user permissions as an enum set (READ, WRITE, EXECUTE).
- **Flags:** Replacing int-based bit flags with type-safe enum sets.
- **Scheduling:** Modeling which days of the week a task runs.

---

## 6. CopyOnWriteArraySet

### Definitions

**Core Definition:** `CopyOnWriteArraySet` is a thread-safe set implementation backed by an internal `CopyOnWriteArrayList`.

**Technical Definition:** "A Set that uses an internal CopyOnWriteArrayList for all of its operations. Thus, it shares the same basic properties: It is best suited for applications in which set sizes generally stay small, read-only operations vastly outnumber mutative operations, and you need to prevent interference among threads during traversal. It is thread-safe. Mutative operations (add, set, remove, etc.) are expensive since they usually entail copying the entire underlying array. Iterators do not support the mutative remove operation. Traversal via iterators is fast and cannot encounter interference from other threads".

**Beginner-Friendly Explanation:** `CopyOnWriteArraySet` is a thread-safe set that makes a fresh copy of its internal array every time you modify it. This makes it very safe for reading from multiple threads at the same time, but slow if you change it often.

### Purposes

- To provide thread-safe set operations without explicit synchronization
- To enable safe iteration without `ConcurrentModificationException`
- To support scenarios where reads vastly outnumber writes

### Syntax Rules and Structure

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements Serializable {
    public CopyOnWriteArraySet();
    public CopyOnWriteArraySet(Collection<? extends E> c);
}
```

**Syntax Rules:**

- All mutative operations create a new copy of the underlying array.
- Iterators are snapshot-based and never throw `ConcurrentModificationException`.
- Iterators do not support `remove()`.

**Constraints and Limitations:**

- Mutations are expensive because the entire array is copied.
- Best suited for read-heavy, write-rare scenarios.

### Annotated Code Example

```java
// File: CopyOnWriteArraySetDemo.java
import java.util.concurrent.*;

public class CopyOnWriteArraySetDemo {
    public static void main(String[] args) {
        CopyOnWriteArraySet<String> set = new CopyOnWriteArraySet<>();
        set.add("Apple");
        set.add("Banana");
        set.add("Cherry");
        set.add("Apple"); // Duplicate — ignored

        System.out.println("Set: " + set);

        // Safe iteration even if another thread modifies the set
        for (String s : set) {
            System.out.println(s);
        }
    }
}
```

**Expected Output:**
```
Set: [Apple, Banana, Cherry]
Apple
Banana
Cherry
```

**Step-by-Step Explanation:**

1. `CopyOnWriteArraySet` is created and populated.
2. Adding "Apple" again is ignored.
3. The for-each loop iterates over a snapshot of the set.
4. The output shows the set contents.

### Real-World Cases

- **Event Listeners:** Maintaining a set of listeners that are iterated frequently but modified rarely.
- **Configuration Sets:** Read-mostly sets of settings or properties.
- **Caching:** Sets of cached items that are occasionally refreshed.

---

## 7. ConcurrentSkipListSet

### Definitions

**Core Definition:** `ConcurrentSkipListSet` is a scalable concurrent `NavigableSet` implementation based on a `ConcurrentSkipListMap`.

**Technical Definition:** "A scalable concurrent NavigableSet implementation based on a ConcurrentSkipListMap. The elements of the set are kept sorted according to their natural ordering, or by a Comparator provided at set creation time, depending on which constructor is used. This implementation provides expected average log(n) time cost for the contains, add, and remove operations and their variants. Insertion, removal, and access operations safely execute concurrently by multiple threads".

**Beginner-Friendly Explanation:** `ConcurrentSkipListSet` is a thread-safe set that keeps its elements sorted. It is designed for concurrent access by multiple threads without requiring external synchronization.

### Purposes

- To provide a thread-safe, sorted set implementation
- To enable concurrent sorted set operations
- To support scalable, concurrent navigation

### Syntax Rules and Structure

```java
public class ConcurrentSkipListSet<E> extends AbstractSet<E>
        implements NavigableSet<E>, Cloneable, Serializable {
    public ConcurrentSkipListSet();
    public ConcurrentSkipListSet(Comparator<? super E> comparator);
    public ConcurrentSkipListSet(Collection<? extends E> c);
    public ConcurrentSkipListSet(SortedSet<E> s);
}
```

**Syntax Rules:**

- Elements are sorted by natural ordering or a `Comparator`.
- `size()` is not a constant-time operation and may report inaccurate results during concurrent modification.
- Does **not** permit null elements.
- Iterators are weakly consistent and never throw `ConcurrentModificationException`.

**Constraints and Limitations:**

- Bulk operations are not guaranteed to be performed atomically.
- `size()` requires traversal.

### Annotated Code Example

```java
// File: ConcurrentSkipListSetDemo.java
import java.util.concurrent.*;

public class ConcurrentSkipListSetDemo {
    public static void main(String[] args) {
        ConcurrentSkipListSet<Integer> set = new ConcurrentSkipListSet<>();

        set.add(5);
        set.add(2);
        set.add(8);
        set.add(1);
        set.add(9);

        System.out.println("Set (sorted): " + set);
        System.out.println("First: " + set.first());
        System.out.println("Last: " + set.last());
        System.out.println("HeadSet (< 5): " + set.headSet(5));
    }
}
```

**Expected Output:**
```
Set (sorted): [1, 2, 5, 8, 9]
First: 1
Last: 9
HeadSet (< 5): [1, 2]
```

**Step-by-Step Explanation:**

1. Elements are inserted: 5, 2, 8, 1, 9.
2. The set stores them in sorted order: [1, 2, 5, 8, 9].
3. `first()` returns 1, `last()` returns 9.
4. `headSet(5)` returns elements less than 5.

### Real-World Cases

- **Concurrent Sorted Data:** Maintaining a sorted set of scores in a multi-threaded environment.
- **Priority Queues:** Implementing concurrent priority-based processing.
- **Leaderboards:** Concurrent sorted high scores.

---

## 8. Set Operations

### Definitions

**Core Definition:** Set operations are the methods provided by the `Set` interface and its implementations for manipulating and querying set elements.

**Technical Definition:** The `Set` interface provides methods for adding elements (`add`, `addAll`), removing elements (`remove`, `removeAll`, `retainAll`, `clear`), querying membership (`contains`, `containsAll`), and traversing elements (`forEach`, `iterator`, `stream`).

**Beginner-Friendly Explanation:** Set operations are the actions you can perform on a set — adding items, removing items, checking membership, combining sets, and more.

### add()

**Definition:** Adds an element to the set if it is not already present.

**Syntax:**
```java
boolean add(E e);
```

**Example:**
```java
Set<String> set = new HashSet<>();
set.add("Apple"); // Returns true
set.add("Apple"); // Returns false (already present)
```

### remove()

**Definition:** Removes the specified element from the set if it is present.

**Syntax:**
```java
boolean remove(Object o);
```

**Example:**
```java
set.remove("Apple"); // Returns true if removed
```

### contains()

**Definition:** Returns `true` if the set contains the specified element.

**Syntax:**
```java
boolean contains(Object o);
```

**Example:**
```java
boolean hasApple = set.contains("Apple");
```

### size() / isEmpty()

**Definition:** Returns the number of elements or checks if the set is empty.

**Syntax:**
```java
int size();
boolean isEmpty();
```

**Example:**
```java
int count = set.size();
boolean empty = set.isEmpty();
```

### clear()

**Definition:** Removes all elements from the set.

**Syntax:**
```java
void clear();
```

**Example:**
```java
set.clear(); // Set is now empty
```

### addAll()

**Definition:** Adds all elements from the specified collection that are not already present (union).

**Syntax:**
```java
boolean addAll(Collection<? extends E> c);
```

**Example:**
```java
Set<String> set1 = new HashSet<>(Arrays.asList("A", "B"));
Set<String> set2 = new HashSet<>(Arrays.asList("B", "C"));
set1.addAll(set2); // set1 = [A, B, C]
```

### retainAll()

**Definition:** Retains only the elements that are contained in the specified collection (intersection).

**Syntax:**
```java
boolean retainAll(Collection<?> c);
```

**Example:**
```java
set1.retainAll(set2); // set1 = [B]
```

### removeAll()

**Definition:** Removes all elements that are contained in the specified collection (difference).

**Syntax:**
```java
boolean removeAll(Collection<?> c);
```

**Example:**
```java
set1.removeAll(set2); // set1 = [A]
```

### containsAll()

**Definition:** Returns `true` if the set contains all elements of the specified collection (subset check).

**Syntax:**
```java
boolean containsAll(Collection<?> c);
```

**Example:**
```java
boolean isSubset = set1.containsAll(set2);
```

### forEach()

**Definition:** Performs the given action for each element.

**Syntax:**
```java
default void forEach(Consumer<? super E> action);
```

**Example:**
```java
set.forEach(System.out::println);
```

### iterator()

**Definition:** Returns an iterator over the elements in the set.

**Syntax:**
```java
Iterator<E> iterator();
```

**Example:**
```java
Iterator<String> it = set.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}
```

### stream()

**Definition:** Returns a sequential stream with the set as its source.

**Syntax:**
```java
default Stream<E> stream();
```

**Example:**
```java
set.stream().filter(s -> s.startsWith("A")).forEach(System.out::println);
```

---

## 9. Set Implementations Comparison

| Feature | HashSet | LinkedHashSet | TreeSet | EnumSet | CopyOnWriteArraySet | ConcurrentSkipListSet |
|---------|---------|---------------|---------|---------|---------------------|----------------------|
| **Backing Structure** | Hash table | Hash table + linked list | Red-black tree | Bit vector | Copy-on-write array | Skip list |
| **Thread-Safe** | No | No | No | No | Yes | Yes |
| **Ordering** | None | Insertion order | Sorted order | Enum declaration order | Insertion order | Sorted order |
| **Null Elements** | Yes (one) | Yes (one) | No | No | Yes (one) | No |
| **Basic Operations** | O(1) | O(1) | O(log n) | O(1) | O(n) for writes, O(1) for reads | O(log n) |
| **Iteration** | O(size + capacity) | O(size) | O(size) | O(size) | Snapshot-based | O(size) |
| **Duplicates** | No | No | No | No | No | No |
| **Recommended** | Default choice | When order matters | When sorted order needed | For enum types | Read-heavy, write-rare concurrent | Concurrent sorted |
| **Since** | 1.2 | 1.4 | 1.2 | 1.5 | 1.5 | 1.6 |

**Key Takeaways:**

- `HashSet` is usually the best default choice for general-purpose sets.
- `LinkedHashSet` is used when insertion order matters.
- `TreeSet` is used when sorted order is required.
- `EnumSet` is the most efficient choice for enum types.
- `CopyOnWriteArraySet` is best for read-heavy, write-rare concurrent scenarios.
- `ConcurrentSkipListSet` is best for concurrent sorted set operations.

---

## 10. Set Best Practices

### Definitions

**Core Definition:** Set best practices are guidelines for choosing and using set implementations effectively.

**Technical Definition:** Best practices include programming to the `Set` interface, choosing the right implementation for the use case, overriding `equals()` and `hashCode()` for custom elements, and avoiding null elements in sets that prohibit them.

**Beginner-Friendly Explanation:** Best practices help you write set code that is fast, safe, and easy to maintain.

### Key Best Practices

| Practice | Description |
|----------|-------------|
| **Program to interfaces** | Declare variables as `Set`, not `HashSet`. |
| **Choose the right implementation** | Use `HashSet` for general use; `LinkedHashSet` for insertion order; `TreeSet` for sorted order; `EnumSet` for enums. |
| **Override `equals()` and `hashCode()`** | For custom objects, these methods are essential for correct duplicate detection and hashing. |
| **Avoid null elements** | Most set implementations do not permit null; check before adding. |
| **Specify initial capacity** | For `HashSet`, set an initial capacity to avoid resizing. |
| **Use `Set.of()` for immutable sets** | Java 9+ provides `Set.of()` for unmodifiable sets. |
| **Use streams for bulk operations** | `stream()`, `filter()`, `map()`, `collect()` for functional-style processing. |
| **Be careful with mutable elements** | Changing an element's hash code after insertion breaks the set. |
| **Use `EnumSet` for enum types** | It is far more efficient than `HashSet` for enums. |

### Annotated Code Example

```java
// File: SetBestPractices.java
import java.util.*;

public class SetBestPractices {
    public static void main(String[] args) {
        // GOOD: Program to interface
        Set<String> set = new HashSet<>(64); // Initial capacity

        // GOOD: Use Set.of() for immutable sets
        Set<String> immutable = Set.of("A", "B", "C");
        // immutable.add("D"); // UnsupportedOperationException

        // GOOD: Override equals() and hashCode() for custom objects
        Set<Person> people = new HashSet<>();
        people.add(new Person("Alice", 30));
        people.add(new Person("Alice", 30)); // Duplicate — ignored
        System.out.println("People: " + people.size()); // 1

        // GOOD: Use EnumSet for enum types
        EnumSet<Day> weekdays = EnumSet.of(Day.MONDAY, Day.TUESDAY, Day.WEDNESDAY);
        System.out.println("Weekdays: " + weekdays);
    }

    static class Person {
        private String name;
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Person person = (Person) o;
            return age == person.age && Objects.equals(name, person.name);
        }

        @Override
        public int hashCode() {
            return Objects.hash(name, age);
        }

        @Override
        public String toString() {
            return name + " (" + age + ")";
        }
    }

    enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }
}
```

**Expected Output:**
```
People: 1
Weekdays: [MONDAY, TUESDAY, WEDNESDAY]
```

**Step-by-Step Explanation:**

1. `new HashSet<>(64)` specifies an initial capacity.
2. `Set.of("A", "B", "C")` creates an immutable set.
3. `Person` overrides `equals()` and `hashCode()`, so duplicate people are correctly identified.
4. `EnumSet.of(...)` creates an efficient enum set.

### Real-World Cases

- **API Design:** Returning `Set` from methods instead of `HashSet`.
- **Performance Tuning:** Specifying initial capacity for large sets.
- **Immutable APIs:** Using `Set.of()` to protect internal state.
- **Concurrent Systems:** Using `CopyOnWriteArraySet` or `ConcurrentSkipListSet` for thread-safe sets.

---

## References Links

- Set (Java SE 11 & JDK 11) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Set.html
- HashSet (Java SE 27) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/27/docs/api/java.base/java/util/HashSet.html
- LinkedHashSet (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedHashSet.html
- TreeSet (Java SE 11) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/TreeSet.html
- EnumSet (Java SE 11) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/EnumSet.html
- CopyOnWriteArraySet (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CopyOnWriteArraySet.html
- ConcurrentSkipListSet (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentSkipListSet.html
- Set Implementations — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/collections/implementations/set.html
- The Set Interface — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/collections/interfaces/set.html
- Set Implementations Comparison — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/collections/implementations/set.html