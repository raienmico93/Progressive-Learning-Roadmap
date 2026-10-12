# Java Collections Utility Class: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `Collections` utility class is a final class consisting exclusively of static methods that operate on or return collections, providing polymorphic algorithms, wrapper factories, and other utility methods.

**Technical Definition:** The `java.util.Collections` class consists exclusively of static methods that operate on or return collections. It contains polymorphic algorithms that operate on collections, "wrappers", which return a new collection backed by a specified collection, and a few other odds and ends. The methods of this class all throw a `NullPointerException` if the collections or class objects provided to them are `null`. The "destructive" algorithms contained in this class, that is, the algorithms that modify the collection on which they operate, are specified to throw `UnsupportedOperationException` if the collection does not support the appropriate mutation primitive(s), such as the `set` method.

**Beginner-Friendly Explanation:** The `Collections` class is a toolbox of static helper methods for working with collections. It's like a Swiss Army knife for lists, sets, and maps — it can sort, search, shuffle, reverse, fill, copy, and wrap your collections, among many other operations. You never create an instance of `Collections`; you just call its static methods directly, like `Collections.sort(myList)`.

### Key Characteristics

- **Utility Class:** Cannot be instantiated (it has a private constructor); all methods are static.
- **Polymorphic Algorithms:** Sorting, searching, shuffling, and other algorithms work on any collection that implements the appropriate interface.
- **Wrapper Factories:** Provides unmodifiable, synchronized, and type-checked views of collections.
- **Factory Methods:** Provides immutable empty collections and singleton collections.
- **Destructive Operations:** Some methods modify the collection in place; these throw `UnsupportedOperationException` if the collection does not support mutation.
- **Null-Hostile:** All methods throw `NullPointerException` if provided with `null` collections or class objects.

### Prerequisites

- Basic understanding of Java classes, objects, and interfaces
- Familiarity with the Java Collections Framework (`List`, `Set`, `Map`, etc.)
- Knowledge of generics and the `Comparable`/`Comparator` interfaces
- Awareness of the `java.util` package

### Related Programming Areas

- **Collections Framework:** The `Collections` class is a core member of the Java Collections Framework.
- **Algorithms:** Provides reusable sorting, searching, and shuffling algorithms.
- **Thread Safety:** Synchronization wrappers enable thread-safe access to collections.
- **API Design:** Unmodifiable wrappers protect internal state when returning collections from APIs.
- **Functional Programming:** Some methods (`sort`, `replaceAll`, `addAll`) integrate with lambdas.

---

## Core Concepts / Features

---

## 1. Collections

### Definitions

**Core Definition:** `Collections` is a final utility class in the `java.util` package that provides static methods for operating on or returning collections.

**Technical Definition:** `public class Collections extends Object`. "This class consists exclusively of static methods that operate on or return collections. It contains polymorphic algorithms that operate on collections, 'wrappers', which return a new collection backed by a specified collection, and a few other odds and ends". The class is declared `final` and has a private constructor, so it cannot be instantiated or subclassed. It was introduced in JDK 1.2.

**Beginner-Friendly Explanation:** `Collections` is like a toolbox. You don't build a toolbox — you just open it and use the tools inside. Each tool (static method) helps you do something useful with your collections.

### Purposes

- To provide reusable algorithms (sorting, searching, shuffling) for collections
- To create unmodifiable and synchronized wrappers for collections
- To provide factory methods for empty and singleton collections
- To simplify common collection tasks and reduce boilerplate code

### Syntax Rules and Structure

```java
// All methods are static — call them directly on the class
Collections.methodName(arguments);
```

**Key Categories of Methods:**

| Category | Examples |
|----------|----------|
| Algorithms | `sort()`, `binarySearch()`, `shuffle()`, `reverse()`, `rotate()`, `swap()`, `fill()`, `copy()` |
| Aggregates | `min()`, `max()`, `frequency()`, `disjoint()` |
| Bulk Operations | `addAll()`, `replaceAll()` |
| Wrappers | `unmodifiableList()`, `synchronizedList()`, `checkedList()` |
| Factories | `emptyList()`, `singletonList()`, `nCopies()` |

**Syntax Rules:**

- `Collections` is `final` and cannot be instantiated.
- All methods are `static`.
- Methods throw `NullPointerException` if provided `null` arguments.
- Destructive algorithms throw `UnsupportedOperationException` if the collection does not support mutation.

**Constraints and Limitations:**

- The class does not provide a method for every possible collection operation.
- Some methods are only applicable to `List` implementations.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Collections Utility Usage**

```java
// File: CollectionsUtilityDemo.java
import java.util.*;

public class CollectionsUtilityDemo {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 9, 3));

        // Sorting
        Collections.sort(numbers);
        System.out.println("Sorted: " + numbers); // [1, 2, 3, 5, 8, 9]

        // Binary search (on sorted list)
        int index = Collections.binarySearch(numbers, 5);
        System.out.println("Index of 5: " + index); // 3

        // Min, max
        System.out.println("Min: " + Collections.min(numbers)); // 1
        System.out.println("Max: " + Collections.max(numbers)); // 9

        // Reverse
        Collections.reverse(numbers);
        System.out.println("Reversed: " + numbers); // [9, 8, 5, 3, 2, 1]

        // Shuffle
        Collections.shuffle(numbers);
        System.out.println("Shuffled: " + numbers); // random order

        // Unmodifiable wrapper
        List<Integer> immutable = Collections.unmodifiableList(numbers);
        // immutable.add(100); // UnsupportedOperationException
    }
}
```

**Expected Output:**
```
Sorted: [1, 2, 3, 5, 8, 9]
Index of 5: 3
Min: 1
Max: 9
Reversed: [9, 8, 5, 3, 2, 1]
Shuffled: [3, 9, 2, 1, 8, 5] (order varies)
```

### Real-World Cases

- **Sorting User Data:** Sorting a list of users by name, age, or registration date.
- **Searching:** Binary search on sorted lists for fast lookups.
- **Shuffling:** Randomising a deck of cards or a playlist.
- **Protecting API State:** Returning unmodifiable views from getters.

---

## 2. Collections.sort()

### Definitions

**Core Definition:** `Collections.sort()` sorts the specified list into ascending order according to the natural ordering of its elements, or according to a specified `Comparator`.

**Technical Definition:** `static <T extends Comparable<? super T>> void sort(List<T> list)` sorts the list into ascending order according to natural ordering. `static <T> void sort(List<T> list, Comparator<? super T> c)` sorts according to the order induced by the comparator. The sort is guaranteed to be stable: equal elements will not be reordered. The specified list must be modifiable, but need not be resizable. This implementation dumps the list into an array, sorts the array, and iterates over the list resetting each element from the corresponding position in the array.

**Beginner-Friendly Explanation:** `sort()` arranges the items in a list in order — either the natural order (like numbers smallest to largest, or strings A to Z) or a custom order you specify.

### Purposes

- To arrange list elements in a specific order
- To prepare data for binary search
- To improve data presentation and readability

### Syntax Rules and Structure

```java
// Natural ordering
static <T extends Comparable<? super T>> void sort(List<T> list)

// Custom ordering
static <T> void sort(List<T> list, Comparator<? super T> c)
```

**Syntax Rules:**

- Elements must implement `Comparable` (for natural ordering) or a `Comparator` must be provided.
- The list must be modifiable.
- The sort is stable.
- The sort is O(n log n).

**Constraints and Limitations:**

- Cannot sort unmodifiable lists (throws `UnsupportedOperationException`).
- Cannot sort lists whose elements are not mutually comparable.

### Annotated Code Example

```java
// File: SortDemo.java
import java.util.*;

public class SortDemo {
    public static void main(String[] args) {
        // Natural ordering
        List<String> names = new ArrayList<>(Arrays.asList("Charlie", "Alice", "Bob"));
        Collections.sort(names);
        System.out.println("Natural: " + names); // [Alice, Bob, Charlie]

        // Custom ordering (by length)
        List<String> words = new ArrayList<>(Arrays.asList("Java", "is", "versatile", "and", "powerful"));
        Collections.sort(words, Comparator.comparingInt(String::length));
        System.out.println("By length: " + words); // [is, and, Java, powerful, versatile]
    }
}
```

**Expected Output:**
```
Natural: [Alice, Bob, Charlie]
By length: [is, and, Java, powerful, versatile]
```

### Real-World Cases

- **E-commerce:** Sorting products by price or popularity.
- **Contact Lists:** Sorting contacts alphabetically.
- **Log Files:** Sorting log entries by timestamp.

---

## 3. Collections.binarySearch()

### Definitions

**Core Definition:** `Collections.binarySearch()` searches a sorted list for a specified object using the binary search algorithm.

**Technical Definition:** `static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key)` searches the list for the key using natural ordering. `static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c)` uses the comparator. Returns the index of the search key, if it is contained in the list; otherwise, returns `(-(insertion point) - 1)`. The list must be sorted into ascending order according to the same ordering prior to making this call.

**Beginner-Friendly Explanation:** `binarySearch()` finds the position of an item in a **sorted** list very quickly. It's like looking up a word in a dictionary — you open to the middle, decide whether to go left or right, and repeat.

### Purposes

- To find the index of an element in a sorted list efficiently (O(log n))
- To determine where an element should be inserted to maintain sorted order

### Syntax Rules and Structure

```java
// Natural ordering
static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key)

// Custom ordering
static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c)
```

**Syntax Rules:**

- The list must be sorted in ascending order before calling.
- If the list is not sorted, the result is undefined.
- Returns the index of the key if found; otherwise returns `-(insertion point) - 1`.

**Constraints and Limitations:**

- Uses `equals()` to determine equality, not `compareTo()`.
- Not suitable for unsorted lists.

### Annotated Code Example

```java
// File: BinarySearchDemo.java
import java.util.*;

public class BinarySearchDemo {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 3, 5, 7, 9, 11, 13));

        int index = Collections.binarySearch(numbers, 7);
        System.out.println("Index of 7: " + index); // 3

        int notFound = Collections.binarySearch(numbers, 4);
        System.out.println("Index of 4: " + notFound); // -4 (insertion point 3, so -3-1 = -4)
    }
}
```

**Expected Output:**
```
Index of 7: 3
Index of 4: -4
```

### Real-World Cases

- **Searching Sorted Data:** Finding a student's record by ID in a sorted list.
- **Insertion Point:** Determining where to insert a new element to keep a list sorted.

---

## 4. Collections.reverse()

### Definitions

**Core Definition:** `Collections.reverse()` reverses the order of the elements in the specified list.

**Technical Definition:** `static void reverse(List<?> list)` reverses the order of the elements in the specified list. This method runs in linear time.

**Beginner-Friendly Explanation:** `reverse()` flips the list — the first element becomes last, the last becomes first, and so on.

### Purposes

- To reverse the order of elements in a list
- To display data in reverse order
- To implement undo/redo functionality

### Syntax Rules and Structure

```java
static void reverse(List<?> list)
```

**Syntax Rules:**

- The list must be modifiable.
- Runs in linear time.

**Constraints and Limitations:**

- Throws `UnsupportedOperationException` for unmodifiable lists.

### Annotated Code Example

```java
// File: ReverseDemo.java
import java.util.*;

public class ReverseDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));
        Collections.reverse(list);
        System.out.println("Reversed: " + list); // [D, C, B, A]
    }
}
```

**Expected Output:**
```
Reversed: [D, C, B, A]
```

### Real-World Cases

- **Displaying Recent First:** Reversing a list of log entries to show newest first.
- **Undo History:** Reversing the order of operations.

---

## 5. Collections.shuffle()

### Definitions

**Core Definition:** `Collections.shuffle()` randomly permutes the elements in the specified list.

**Technical Definition:** `static void shuffle(List<?> list)` randomly permutes the list using a default source of randomness. `static void shuffle(List<?> list, Random rnd)` uses the specified source of randomness. This implementation traverses the list backwards, from the last element up to the second, repeatedly swapping a randomly selected element into the "current position".

**Beginner-Friendly Explanation:** `shuffle()` mixes up the items in a list randomly, like shuffling a deck of cards.

### Purposes

- To randomise the order of elements in a list
- To implement card games, lottery draws, or randomised playlists
- To eliminate ordering bias in experiments or simulations

### Syntax Rules and Structure

```java
static void shuffle(List<?> list)
static void shuffle(List<?> list, Random rnd)
```

**Syntax Rules:**

- The list must be modifiable.
- Runs in linear time.

**Constraints and Limitations:**

- Throws `UnsupportedOperationException` for unmodifiable lists.

### Annotated Code Example

```java
// File: ShuffleDemo.java
import java.util.*;

public class ShuffleDemo {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        Collections.shuffle(numbers);
        System.out.println("Shuffled: " + numbers); // random order

        // With a specific Random for reproducibility
        Collections.shuffle(numbers, new Random(42));
        System.out.println("Shuffled with seed: " + numbers);
    }
}
```

**Expected Output:**
```
Shuffled: [4, 2, 5, 1, 3] (varies)
Shuffled with seed: [3, 5, 1, 4, 2] (varies)
```

### Real-World Cases

- **Card Games:** Shuffling a deck of cards.
- **Lottery:** Drawing random numbers.
- **Playlists:** Randomising song order.

---

## 6. Collections.swap()

### Definitions

**Core Definition:** `Collections.swap()` swaps the elements at two specified positions in a list.

**Technical Definition:** `static void swap(List<?> list, int i, int j)` swaps the elements at the specified positions in the specified list. If `i == j`, the invocation leaves the list unchanged.

**Beginner-Friendly Explanation:** `swap()` exchanges the items at two positions in a list. It's like swapping the positions of two books on a shelf.

### Purposes

- To exchange two elements in a list
- To implement sorting algorithms (e.g., bubble sort)
- To reorder elements by position

### Syntax Rules and Structure

```java
static void swap(List<?> list, int i, int j)
```

**Syntax Rules:**

- The list must be modifiable.
- `i` and `j` must be valid indices (0 ≤ index < size).
- If `i == j`, the list is unchanged.

**Constraints and Limitations:**

- Throws `IndexOutOfBoundsException` for invalid indices.

### Annotated Code Example

```java
// File: SwapDemo.java
import java.util.*;

public class SwapDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));
        Collections.swap(list, 0, 3);
        System.out.println("After swap: " + list); // [D, B, C, A]
    }
}
```

**Expected Output:**
```
After swap: [D, B, C, A]
```

### Real-World Cases

- **Sorting:** Swapping elements during custom sort implementations.
- **Game Boards:** Swapping tiles or pieces.
- **List Reordering:** Moving an element to a specific position.

---

## 7. Collections.fill()

### Definitions

**Core Definition:** `Collections.fill()` replaces all elements in a list with the specified element.

**Technical Definition:** `static <T> void fill(List<? super T> list, T obj)` replaces all of the elements of the specified list with the specified element. This method runs in linear time.

**Beginner-Friendly Explanation:** `fill()` is like painting every item in a list the same colour — it replaces all elements with the same value.

### Purposes

- To initialise a list with a default value
- To reset all elements to a single value
- To prepare a list for further operations

### Syntax Rules and Structure

```java
static <T> void fill(List<? super T> list, T obj)
```

**Syntax Rules:**

- The list must be modifiable.
- Runs in linear time.

**Constraints and Limitations:**

- Throws `UnsupportedOperationException` for unmodifiable lists.

### Annotated Code Example

```java
// File: FillDemo.java
import java.util.*;

public class FillDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
        Collections.fill(list, "X");
        System.out.println("Filled: " + list); // [X, X, X]
    }
}
```

**Expected Output:**
```
Filled: [X, X, X]
```

### Real-World Cases

- **Initialisation:** Creating a list of default values.
- **Reset:** Clearing a board game or grid.

---

## 8. Collections.copy()

### Definitions

**Core Definition:** `Collections.copy()` copies all elements from one list into another.

**Technical Definition:** `static <T> void copy(List<? super T> dest, List<? extends T> src)` copies all of the elements from one list into another. After the operation, the index of each copied element in the destination list will be identical to its index in the source list. The destination list must be at least as long as the source list. If it is longer, the remaining elements in the destination list are unaffected.

**Beginner-Friendly Explanation:** `copy()` copies the contents of one list into another list, replacing the destination's elements.

### Purposes

- To duplicate the contents of a list into another
- To back up a list
- To transfer data between lists

### Syntax Rules and Structure

```java
static <T> void copy(List<? super T> dest, List<? extends T> src)
```

**Syntax Rules:**

- The destination list must be at least as long as the source list.
- The destination list must be modifiable.
- Runs in linear time.

**Constraints and Limitations:**

- Throws `IndexOutOfBoundsException` if the destination list is too small.

### Annotated Code Example

```java
// File: CopyDemo.java
import java.util.*;

public class CopyDemo {
    public static void main(String[] args) {
        List<String> src = Arrays.asList("A", "B", "C");
        List<String> dest = new ArrayList<>(Arrays.asList("X", "Y", "Z", "W"));

        Collections.copy(dest, src);
        System.out.println("After copy: " + dest); // [A, B, C, W]
    }
}
```

**Expected Output:**
```
After copy: [A, B, C, W]
```

### Real-World Cases

- **Backup:** Copying a list before modifying it.
- **Batch Processing:** Transferring data between buffers.

---

## 9. Collections.min() / Collections.max()

### Definitions

**Core Definition:** `Collections.min()` and `Collections.max()` return the minimum or maximum element of a collection according to natural ordering or a specified comparator.

**Technical Definition:** `static <T extends Object & Comparable<? super T>> T min(Collection<? extends T> coll)` returns the minimum element according to natural ordering. `static <T> T min(Collection<? extends T> coll, Comparator<? super T> comp)` uses the comparator. Similarly for `max()`. These methods iterate over the entire collection.

**Beginner-Friendly Explanation:** `min()` finds the smallest element, and `max()` finds the largest, according to a comparison rule.

### Purposes

- To find the smallest or largest element in a collection
- To determine extremes in data sets
- To implement leaderboards or rankings

### Syntax Rules and Structure

```java
static <T extends Object & Comparable<? super T>> T min(Collection<? extends T> coll)
static <T> T min(Collection<? extends T> coll, Comparator<? super T> comp)
static <T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll)
static <T> T max(Collection<? extends T> coll, Comparator<? super T> comp)
```

**Syntax Rules:**

- The collection must be non-empty.
- Elements must be mutually comparable.
- Runs in linear time.

**Constraints and Limitations:**

- Throws `NoSuchElementException` if the collection is empty.

### Annotated Code Example

```java
// File: MinMaxDemo.java
import java.util.*;

public class MinMaxDemo {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9);

        System.out.println("Min: " + Collections.min(numbers)); // 1
        System.out.println("Max: " + Collections.max(numbers)); // 9

        // With custom comparator (reverse order)
        System.out.println("Min (reverse): " + Collections.min(numbers, Comparator.reverseOrder())); // 9
    }
}
```

**Expected Output:**
```
Min: 1
Max: 9
Min (reverse): 9
```

### Real-World Cases

- **Leaderboards:** Finding the highest score.
- **Data Analysis:** Finding the minimum or maximum value in a dataset.
- **Scheduling:** Finding the earliest or latest date.

---

## 10. Collections.frequency()

### Definitions

**Core Definition:** `Collections.frequency()` returns the number of elements in a collection equal to a specified object.

**Technical Definition:** `static int frequency(Collection<?> c, Object o)` returns the number of elements in the specified collection equal to the specified object. More formally, returns the number of elements `e` in the collection such that `Objects.equals(o, e)`.

**Beginner-Friendly Explanation:** `frequency()` counts how many times a specific item appears in a collection.

### Purposes

- To count occurrences of an element
- To analyse data distributions
- To detect duplicates

### Syntax Rules and Structure

```java
static int frequency(Collection<?> c, Object o)
```

**Syntax Rules:**

- Uses `Objects.equals()` for comparison.
- Runs in linear time.

**Constraints and Limitations:**

- None significant.

### Annotated Code Example

```java
// File: FrequencyDemo.java
import java.util.*;

public class FrequencyDemo {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("apple", "banana", "apple", "cherry", "apple");
        System.out.println("Frequency of apple: " + Collections.frequency(words, "apple")); // 3
        System.out.println("Frequency of banana: " + Collections.frequency(words, "banana")); // 1
    }
}
```

**Expected Output:**
```
Frequency of apple: 3
Frequency of banana: 1
```

### Real-World Cases

- **Text Analysis:** Counting word occurrences.
- **Inventory:** Counting stock items.
- **Voting:** Counting votes for candidates.

---

## 11. Collections.disjoint()

### Definitions

**Core Definition:** `Collections.disjoint()` returns `true` if two collections have no elements in common.

**Technical Definition:** `static boolean disjoint(Collection<?> c1, Collection<?> c2)` returns `true` if the two specified collections have no elements in common. If either collection is empty, returns `true`.

**Beginner-Friendly Explanation:** `disjoint()` checks if two collections share any elements. It's like checking if two people have any common friends.

### Purposes

- To check if two collections are completely different
- To detect overlapping data
- To validate that sets are mutually exclusive

### Syntax Rules and Structure

```java
static boolean disjoint(Collection<?> c1, Collection<?> c2)
```

**Syntax Rules:**

- Returns `true` if no elements are shared.
- Runs in linear time.

**Constraints and Limitations:**

- If either collection is empty, returns `true`.

### Annotated Code Example

```java
// File: DisjointDemo.java
import java.util.*;

public class DisjointDemo {
    public static void main(String[] args) {
        Set<String> set1 = new HashSet<>(Arrays.asList("A", "B", "C"));
        Set<String> set2 = new HashSet<>(Arrays.asList("D", "E", "F"));
        Set<String> set3 = new HashSet<>(Arrays.asList("C", "D", "E"));

        System.out.println("set1 and set2 disjoint: " + Collections.disjoint(set1, set2)); // true
        System.out.println("set1 and set3 disjoint: " + Collections.disjoint(set1, set3)); // false
    }
}
```

**Expected Output:**
```
set1 and set2 disjoint: true
set1 and set3 disjoint: false
```

### Real-World Cases

- **Permissions:** Checking if two roles have overlapping permissions.
- **Scheduling:** Finding free time slots.
- **Data Validation:** Ensuring sets are mutually exclusive.

---

## 12. Collections.addAll()

### Definitions

**Core Definition:** `Collections.addAll()` adds all of the specified elements to the specified collection.

**Technical Definition:** `static <T> boolean addAll(Collection<? super T> c, T... elements)` adds all of the specified elements to the specified collection. The behaviour of this convenience method is identical to that of `c.addAll(Arrays.asList(elements))`, but this method is likely to run significantly faster under most implementations.

**Beginner-Friendly Explanation:** `addAll()` adds a bunch of items to a collection in one go.

### Purposes

- To add multiple elements to a collection efficiently
- To combine collections
- To initialise collections with multiple values

### Syntax Rules and Structure

```java
static <T> boolean addAll(Collection<? super T> c, T... elements)
```

**Syntax Rules:**

- Returns `true` if the collection changed as a result.
- Runs in linear time.

**Constraints and Limitations:**

- Throws `UnsupportedOperationException` for unmodifiable collections.

### Annotated Code Example

```java
// File: AddAllDemo.java
import java.util.*;

public class AddAllDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        Collections.addAll(list, "A", "B", "C", "D");
        System.out.println("After addAll: " + list); // [A, B, C, D]
    }
}
```

**Expected Output:**
```
After addAll: [A, B, C, D]
```

### Real-World Cases

- **Bulk Initialisation:** Adding initial data to a collection.
- **Merging Data:** Combining multiple collections.

---

## 13. Collections.unmodifiableList() / unmodifiableSet() / unmodifiableMap()

### Definitions

**Core Definition:** These methods return unmodifiable views of the specified collections, preventing any modifications to the underlying collection through the view.

**Technical Definition:** `static <T> List<T> unmodifiableList(List<? extends T> list)` returns an unmodifiable view of the specified list. `unmodifiableSet()` and `unmodifiableMap()` are similar. Attempts to modify the returned collection, whether directly or via its iterator, result in an `UnsupportedOperationException`. The returned collection acts as a read-only view; the underlying collection can still be modified through the original reference.

**Beginner-Friendly Explanation:** These methods create a "read-only" wrapper around a collection. You can look at it, but you cannot change it — any attempt to modify it throws an exception.

### Purposes

- To protect internal collections from external modification
- To return read-only views from APIs
- To enforce immutability of collection contents

### Syntax Rules and Structure

```java
static <T> List<T> unmodifiableList(List<? extends T> list)
static <T> Set<T> unmodifiableSet(Set<? extends T> s)
static <K,V> Map<K,V> unmodifiableMap(Map<? extends K, ? extends V> m)
```

**Syntax Rules:**

- The returned collection is a view backed by the original.
- Any modification attempt throws `UnsupportedOperationException`.
- The original collection can still be modified elsewhere.

**Constraints and Limitations:**

- The view is read-only, but the original is still mutable.
- Does not make a copy; it wraps.

### Annotated Code Example

```java
// File: UnmodifiableDemo.java
import java.util.*;

public class UnmodifiableDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
        List<String> immutable = Collections.unmodifiableList(list);

        System.out.println("Immutable view: " + immutable); // [A, B, C]

        try {
            immutable.add("D");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify unmodifiable list");
        }

        // Original list can still be modified
        list.add("D");
        System.out.println("Original after modification: " + list); // [A, B, C, D]
        System.out.println("Immutable view reflects change: " + immutable); // [A, B, C, D]
    }
}
```

**Expected Output:**
```
Immutable view: [A, B, C]
Cannot modify unmodifiable list
Original after modification: [A, B, C, D]
Immutable view reflects change: [A, B, C, D]
```

### Real-World Cases

- **API Design:** Returning unmodifiable collections from getters.
- **Defensive Programming:** Protecting internal state from external modification.
- **Read-Only Views:** Providing read-only access to shared data.

---

## 14. Collections.synchronizedList() / synchronizedSet() / synchronizedMap()

### Definitions

**Core Definition:** These methods return synchronized (thread-safe) views of the specified collections.

**Technical Definition:** `static <T> List<T> synchronizedList(List<T> list)` returns a synchronized (thread-safe) list backed by the specified list. `synchronizedSet()` and `synchronizedMap()` are similar. To guarantee serial access, it is critical that all access to the backing collection is accomplished through the returned collection. It is imperative that the user manually synchronize on the returned collection when iterating over it.

**Beginner-Friendly Explanation:** These methods wrap a collection so that only one thread can access it at a time, preventing race conditions in multi-threaded programs.

### Purposes

- To provide thread-safe access to collections
- To protect against concurrent modification in multi-threaded environments
- To serve as a legacy alternative to concurrent collections

### Syntax Rules and Structure

```java
static <T> List<T> synchronizedList(List<T> list)
static <T> Set<T> synchronizedSet(Set<T> s)
static <K,V> Map<K,V> synchronizedMap(Map<K,V> m)
```

**Syntax Rules:**

- The returned collection is thread-safe for individual operations.
- Manual synchronization is required for iteration.
- The original collection should not be accessed directly.

**Constraints and Limitations:**

- Compound operations (e.g., check-then-act) are not atomic.
- Iteration requires manual synchronization.

### Annotated Code Example

```java
// File: SynchronizedDemo.java
import java.util.*;

public class SynchronizedDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        List<String> syncList = Collections.synchronizedList(list);

        // Thread-safe individual operations
        syncList.add("A");
        syncList.add("B");

        // Manual synchronization for iteration
        synchronized (syncList) {
            for (String item : syncList) {
                System.out.println(item);
            }
        }
    }
}
```

**Expected Output:**
```
A
B
```

### Real-World Cases

- **Multi-Threaded Applications:** Sharing a list between threads.
- **Legacy Code:** Thread-safe collections in older Java code.
- **Simple Synchronization:** When full concurrent collections are overkill.

---

## 15. Collections.emptyList() / emptySet() / emptyMap()

### Definitions

**Core Definition:** These methods return immutable empty collections.

**Technical Definition:** `static <T> List<T> emptyList()` returns an empty immutable list. `emptySet()` and `emptyMap()` are similar. These methods are useful when you want to return an empty collection from a method without allocating a new one each time.

**Beginner-Friendly Explanation:** These methods give you an empty collection that you cannot modify. They are useful for returning "nothing" from a method.

### Purposes

- To return an empty collection without allocating a new one
- To avoid `null` returns from methods
- To provide a shared, immutable empty collection

### Syntax Rules and Structure

```java
static <T> List<T> emptyList()
static <T> Set<T> emptySet()
static <K,V> Map<K,V> emptyMap()
```

**Syntax Rules:**

- The returned collections are immutable.
- Attempts to modify throw `UnsupportedOperationException`.
- The same instance may be returned for each call.

**Constraints and Limitations:**

- Cannot add elements to the returned collections.

### Annotated Code Example

```java
// File: EmptyDemo.java
import java.util.*;

public class EmptyDemo {
    public static List<String> getNames(boolean hasNames) {
        if (!hasNames) {
            return Collections.emptyList();
        }
        return Arrays.asList("Alice", "Bob");
    }

    public static void main(String[] args) {
        System.out.println(getNames(false)); // []
        System.out.println(getNames(true));  // [Alice, Bob]
    }
}
```

**Expected Output:**
```
[]
[Alice, Bob]
```

### Real-World Cases

- **API Design:** Returning empty collections instead of `null`.
- **Default Values:** Providing default empty collections.

---

## 16. Collections.singletonList() / singleton() / singletonMap()

### Definitions

**Core Definition:** These methods return immutable collections containing a single element.

**Technical Definition:** `static <T> List<T> singletonList(T o)` returns an immutable list containing only the specified object. `singleton(T o)` returns an immutable set containing only the specified object. `singletonMap(K key, V value)` returns an immutable map containing only the specified key-value pair.

**Beginner-Friendly Explanation:** These methods create a collection with exactly one item, which cannot be changed.

### Purposes

- To create a single-element collection
- To return a single value as a collection from an API
- To provide a convenient way to create immutable single-element collections

### Syntax Rules and Structure

```java
static <T> List<T> singletonList(T o)
static <T> Set<T> singleton(T o)
static <K,V> Map<K,V> singletonMap(K key, V value)
```

**Syntax Rules:**

- The returned collections are immutable.
- Attempts to modify throw `UnsupportedOperationException`.

**Constraints and Limitations:**

- Cannot add or remove elements.

### Annotated Code Example

```java
// File: SingletonDemo.java
import java.util.*;

public class SingletonDemo {
    public static void main(String[] args) {
        List<String> singleList = Collections.singletonList("Only");
        Set<String> singleSet = Collections.singleton("Only");
        Map<String, Integer> singleMap = Collections.singletonMap("Only", 1);

        System.out.println("List: " + singleList); // [Only]
        System.out.println("Set: " + singleSet);   // [Only]
        System.out.println("Map: " + singleMap);   // {Only=1}
    }
}
```

**Expected Output:**
```
List: [Only]
Set: [Only]
Map: {Only=1}
```

### Real-World Cases

- **API Design:** Returning a single element as a collection.
- **Configuration:** Creating a map with one entry.

---

## 17. Collections.nCopies()

### Definitions

**Core Definition:** `Collections.nCopies()` returns an immutable list consisting of `n` copies of a specified object.

**Technical Definition:** `static <T> List<T> nCopies(int n, T o)` returns an immutable list consisting of `n` copies of the specified object. The newly allocated data object is tiny (it contains a single reference to the data object). This method is useful when you need a list of a certain size filled with the same element.

**Beginner-Friendly Explanation:** `nCopies()` creates a list with the same item repeated `n` times.

### Purposes

- To create a list with repeated elements
- To initialise a list with a default value
- To avoid manually adding the same element multiple times

### Syntax Rules and Structure

```java
static <T> List<T> nCopies(int n, T o)
```

**Syntax Rules:**

- `n` must be non-negative.
- Returns an immutable list.
- Uses a single reference internally for efficiency.

**Constraints and Limitations:**

- Throws `IllegalArgumentException` if `n < 0`.

### Annotated Code Example

```java
// File: NCopiesDemo.java
import java.util.*;

public class NCopiesDemo {
    public static void main(String[] args) {
        List<String> list = Collections.nCopies(3, "Hello");
        System.out.println("nCopies: " + list); // [Hello, Hello, Hello]
    }
}
```

**Expected Output:**
```
nCopies: [Hello, Hello, Hello]
```

### Real-World Cases

- **Initialisation:** Creating a list of default values.
- **Padding:** Filling a list with a placeholder value.

---

## 18. Collections.rotate()

### Definitions

**Core Definition:** `Collections.rotate()` rotates the elements in a list by a specified distance.

**Technical Definition:** `static void rotate(List<?> list, int distance)` rotates the elements in the specified list by the specified distance. After calling this method, the element at index `i` will be the element previously at index `(i - distance) mod list.size()`, for all values of `i`. This method has no effect on the size of the list.

**Beginner-Friendly Explanation:** `rotate()` shifts all elements in a list by a certain number of positions, wrapping around the ends.

### Purposes

- To rotate the elements of a list
- To implement circular buffers
- To shift data by a fixed amount

### Syntax Rules and Structure

```java
static void rotate(List<?> list, int distance)
```

**Syntax Rules:**

- `distance` can be positive or negative.
- Positive distance rotates right; negative rotates left.
- The list must be modifiable.

**Constraints and Limitations:**

- Throws `UnsupportedOperationException` for unmodifiable lists.

### Annotated Code Example

```java
// File: RotateDemo.java
import java.util.*;

public class RotateDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D", "E"));

        Collections.rotate(list, 2);
        System.out.println("Rotated right by 2: " + list); // [D, E, A, B, C]

        Collections.rotate(list, -1);
        System.out.println("Rotated left by 1: " + list); // [E, A, B, C, D]
    }
}
```

**Expected Output:**
```
Rotated right by 2: [D, E, A, B, C]
Rotated left by 1: [E, A, B, C, D]
```

### Real-World Cases

- **Circular Buffers:** Rotating data in a fixed-size buffer.
- **Shift Scheduling:** Rotating duty rosters.
- **Data Rotation:** Shifting elements by a fixed amount.

---

## 19. Collections.replaceAll()

### Definitions

**Core Definition:** `Collections.replaceAll()` replaces all occurrences of one specified value in a list with another.

**Technical Definition:** `static <T> boolean replaceAll(List<T> list, T oldVal, T newVal)` replaces all occurrences of the specified value `oldVal` in the list with `newVal`. Returns `true` if any replacements were made. More formally, replaces with `newVal` each element `e` in the list such that `Objects.equals(oldVal, e)`.

**Beginner-Friendly Explanation:** `replaceAll()` finds every occurrence of one value in a list and replaces it with another value.

### Purposes

- To replace all occurrences of a value in a list
- To update data in bulk
- To standardise values in a collection

### Syntax Rules and Structure

```java
static <T> boolean replaceAll(List<T> list, T oldVal, T newVal)
```

**Syntax Rules:**

- Uses `Objects.equals()` for comparison.
- Returns `true` if any replacements occurred.
- The list must be modifiable.

**Constraints and Limitations:**

- Throws `UnsupportedOperationException` for unmodifiable lists.

### Annotated Code Example

```java
// File: ReplaceAllDemo.java
import java.util.*;

public class ReplaceAllDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "A", "C", "A"));
        boolean replaced = Collections.replaceAll(list, "A", "X");
        System.out.println("Replaced: " + replaced); // true
        System.out.println("List: " + list); // [X, B, X, C, X]
    }
}
```

**Expected Output:**
```
Replaced: true
List: [X, B, X, C, X]
```

### Real-World Cases

- **Data Cleaning:** Replacing null values with defaults.
- **Standardisation:** Replacing abbreviations with full terms.
- **Bulk Updates:** Updating all occurrences of a value.

---

## 20. Collections Best Practices

### Definitions

**Core Definition:** Best practices for using the `Collections` utility class are guidelines for writing efficient, safe, and maintainable code when working with collections.

**Technical Definition:** Best practices include programming to interfaces, using the most general type for parameters and the most specific for return values, preferring `Collections` factory methods for empty and singleton collections, using unmodifiable wrappers to protect internal state, and being aware of the performance implications of each method. "Never use an implementation type because this defeats the purpose of an interface-based Collections Framework, which is to allow collections to be manipulated without regard to implementation details". "Use the least-specific type that makes sense".

**Beginner-Friendly Explanation:** Best practices help you use the `Collections` toolbox correctly — choosing the right tool for the job and avoiding common mistakes.

### Key Best Practices

| Practice | Description |
|----------|-------------|
| **Program to interfaces** | Use `List`, `Set`, `Map` as parameter and variable types. |
| **Use the least-specific type for parameters** | Accept `Collection` instead of `List` when possible. |
| **Return the most specific type** | Return `List` or `SortedMap` when the caller needs those capabilities. |
| **Use `Collections.emptyList()` instead of `null`** | Avoid `null` returns; return an empty collection. |
| **Use `Collections.unmodifiableList()` to protect internal state** | Return read-only views from getters. |
| **Use `Collections.synchronizedList()` for thread safety** | Wrap collections when multiple threads access them. |
| **Prefer `List.of()` over `Collections.unmodifiableList()`** | Java 9+ provides immutable factory methods. |
| **Use `Collections.sort()` on modifiable lists** | Do not attempt to sort unmodifiable lists. |
| **Use `Collections.binarySearch()` on sorted lists** | The list must be sorted before searching. |
| **Be aware of `UnsupportedOperationException`** | Destructive algorithms throw when the collection is unmodifiable. |
| **Use `Collections.addAll()` for bulk additions** | More efficient than adding elements one by one. |
| **Use `Collections.frequency()` for counting** | Convenient for counting occurrences. |

### Annotated Code Example

```java
// File: CollectionsBestPractices.java
import java.util.*;

public class CollectionsBestPractices {

    // GOOD: Accept the most general type
    public static int sumAll(Collection<Integer> numbers) {
        int total = 0;
        for (int n : numbers) total += n;
        return total;
    }

    // GOOD: Return an unmodifiable view
    public static List<String> getNames() {
        List<String> names = new ArrayList<>(Arrays.asList("Alice", "Bob"));
        return Collections.unmodifiableList(names);
    }

    // GOOD: Use emptyList() instead of null
    public static List<String> getEmpty() {
        return Collections.emptyList();
    }

    public static void main(String[] args) {
        // GOOD: Use Collections.addAll()
        List<String> list = new ArrayList<>();
        Collections.addAll(list, "A", "B", "C");

        // GOOD: Use Collections.sort()
        Collections.sort(list);

        // GOOD: Use Collections.frequency()
        System.out.println("Frequency of B: " + Collections.frequency(list, "B"));

        // GOOD: Use Collections.unmodifiableList()
        List<String> immutable = getNames();
        // immutable.add("C"); // UnsupportedOperationException

        System.out.println("Names: " + immutable);
    }
}
```

**Expected Output:**
```
Frequency of B: 1
Names: [Alice, Bob]
```

### Real-World Cases

- **API Design:** Returning unmodifiable collections from getters.
- **Thread Safety:** Wrapping collections with `synchronizedList()`.
- **Data Processing:** Using `Collections.sort()`, `binarySearch()`, and `frequency()`.

---

## References Links

- Collections (Java SE 21 & JDK 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collections.html
- Collections (Java Platform SE 11) — Oracle API Documentation - https://docs.oracle.com/javase/jp/11/docs/api/java.base/java/util/Collections.html
- Lesson: Algorithms — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/collections/algorithms/index.html
- API Design — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/collections/interoperability/api-design.html
- Class Collections (OpenJDK) - https://cr.openjdk.org/~iris/se/15/latestSpec/apidiffs/java.base/java/util/Collections-report.html
- Collections Utility Class — Sip of Java (Inside.java) - https://inside.java/2023/06/11/sip080/
- Java Collections Framework: Complete Guide, Patterns, and Best Practices — Cleverence - https://www.cleverence.com/articles/oracle-documentation/java-collections-framework-4721/
- Unmodifiable Wrappers — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/collections/implementations/wrapper.html
- Synchronization Wrappers — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/collections/implementations/wrapper.html
- Java Collections Utility Methods — Baeldung - https://www.baeldung.com/java-collections-utility-class