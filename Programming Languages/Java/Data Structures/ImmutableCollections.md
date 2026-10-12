# Java Immutable Collections: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Immutable collections are collections whose internal structure cannot be modified after creation — elements cannot be added, removed, or replaced. Java 9 introduced static factory methods (`List.of()`, `Set.of()`, `Map.of()`, `Map.ofEntries()`) and copy methods (`List.copyOf()`, `Set.copyOf()`, `Map.copyOf()`) on the collection interfaces to create compact, unmodifiable collection instances conveniently.

**Technical Definition:** Per JEP 269, "Define library APIs to make it convenient to create instances of collections and maps with small numbers of elements, so as to ease the pain of not having collection literals in the Java programming language." The static factory methods on `List`, `Set`, and `Map` interfaces create compact, unmodifiable collection instances. A collection is considered unmodifiable if elements cannot be added, removed, or replaced. The `copyOf` family of methods creates unmodifiable snapshots of existing collections, isolating the returned collection from subsequent changes to the original.

**Beginner-Friendly Explanation:** Immutable collections are like sealed glass jars — once you put items in and seal the jar, you cannot add, remove, or replace anything inside. Java 9 made it easy to create these "sealed jars" with simple factory methods like `List.of("a", "b", "c")`, replacing the verbose pre-Java 9 approach of creating a mutable collection and wrapping it with `Collections.unmodifiableList()`. These collections are safer to share, especially in multi-threaded programs, because no one can accidentally modify them.

### Key Characteristics

- **Structurally Immutable:** Elements cannot be added, removed, or replaced after creation; attempts throw `UnsupportedOperationException`.
- **Null-Hostile:** Null values are not permitted in lists, sets, or maps; passing `null` throws `NullPointerException`.
- **Duplicate Rejection:** `Set.of()` and `Map.of()` reject duplicate elements/keys at creation time, throwing `IllegalArgumentException`.
- **Compact and Efficient:** These collections are implemented with space-efficient internal classes, storing data in fields for small sizes rather than arrays or hash tables.
- **Randomized Iteration Order:** Iteration order for `Set` and `Map` elements is randomized and may differ between JVM runs, helping identify code that depends on iteration order.
- **Serializable:** Instances are serializable if all elements are serializable.
- **Snapshot Semantics:** `copyOf()` creates an isolated copy if the source is mutable; if the source is already unmodifiable, it returns the same reference without copying.

### Prerequisites

- Basic understanding of Java classes, objects, and interfaces
- Familiarity with the Java Collections Framework (`List`, `Set`, `Map`)
- Knowledge of generics (`List<String>`, `Map<K, V>`)
- Awareness of the `java.util` package and the `java.util.stream.Collectors` class
- Java 9 or later for factory methods; Java 10 or later for `copyOf()` methods

### Related Programming Areas

- **Functional Programming:** Immutable collections integrate naturally with streams and lambdas.
- **Concurrency:** Immutable collections are inherently thread-safe if their elements are immutable, eliminating the need for synchronization.
- **API Design:** Returning immutable collections from methods protects internal state from external modification.
- **Defensive Programming:** `copyOf()` provides a safe way to create snapshots of mutable collections.
- **Stream API:** Collectors like `toUnmodifiableList()` create unmodifiable collections from stream pipelines.

---

## Core Concepts / Features

---

## 1. List.of()

### Definitions

**Core Definition:** `List.of()` is a static factory method on the `List` interface that creates an immutable list containing the specified elements.

**Technical Definition:** The `List.of` static factory methods provide a convenient way to create unmodifiable lists. A list is an ordered collection in which duplicate elements are allowed. Null values are not allowed. The methods provide fixed-argument overloads for up to 10 elements and a varargs form supporting an arbitrary number of elements or an array.

**Beginner-Friendly Explanation:** `List.of()` is like writing a shopping list on a stone tablet — once you write it, you cannot erase or add anything. You provide the items when you create it, and that's the final list.

### Purposes

- To create a small, immutable list in a single expression without verbose wrapping code
- To replace the pre-Java 9 pattern of `Collections.unmodifiableList(Arrays.asList(...))`
- To provide a safe, thread-safe list that can be shared freely
- To enable static field initialization of collection constants

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Empty list
static <E> List<E> of()

// Fixed-argument forms (1 to 10 elements)
static <E> List<E> of(E e1)
static <E> List<E> of(E e1, E e2)
// ... up to 10 elements

// Varargs form (arbitrary number of elements or an array)
static <E> List<E> of(E... elements)
```

**Component Breakdown:**

| Form | Description |
|------|-------------|
| `List.of()` | Returns an empty immutable list. |
| `List.of(e1)` | Returns an immutable list containing one element. |
| `List.of(e1, e2)` | Returns an immutable list containing two elements. |
| `List.of(elements...)` | Returns an immutable list containing an arbitrary number of elements. |

**Syntax Rules:**

- Elements are stored in the order provided.
- Duplicate elements are allowed in lists.
- `null` elements are not permitted; passing `null` throws `NullPointerException`.
- The returned list is structurally immutable.

**Constraints and Limitations:**

- Cannot add, remove, or replace elements.
- No null values allowed.
- Iteration order is deterministic (insertion order) for lists.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ListOfDemo.java
import java.util.*;

public class ListOfDemo {
    public static void main(String[] args) {
        // Step 1: Create an empty immutable list
        List<String> empty = List.of();
        System.out.println("Empty: " + empty); // []

        // Step 2: Create a list with elements
        List<String> fruits = List.of("Apple", "Banana", "Cherry");
        System.out.println("Fruits: " + fruits); // [Apple, Banana, Cherry]

        // Step 3: Duplicates are allowed
        List<String> withDuplicates = List.of("A", "B", "A");
        System.out.println("With duplicates: " + withDuplicates); // [A, B, A]

        // Step 4: Attempt to modify throws UnsupportedOperationException
        try {
            fruits.add("Date");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify immutable list");
        }

        // Step 5: Null elements are rejected
        try {
            List.of("A", null);
        } catch (NullPointerException e) {
            System.out.println("Null elements not allowed");
        }

        // Step 6: Varargs form with an array
        String[] array = {"X", "Y", "Z"};
        List<String> fromArray = List.of(array);
        System.out.println("From array: " + fromArray); // [X, Y, Z]
    }
}
```

**Expected Output:**
```
Empty: []
Fruits: [Apple, Banana, Cherry]
With duplicates: [A, B, A]
Cannot modify immutable list
Null elements not allowed
From array: [X, Y, Z]
```

**Step-by-Step Explanation:**

1. `List.of()` creates an empty immutable list.
2. `List.of("Apple", "Banana", "Cherry")` creates a list with three elements in the specified order.
3. Duplicates are permitted because lists allow duplicate elements.
4. `fruits.add("Date")` throws `UnsupportedOperationException` because the list is structurally immutable.
5. `List.of("A", null)` throws `NullPointerException` because null elements are not allowed.
6. `List.of(array)` uses the varargs form to create a list from an array.

### Real-World Cases

- **Configuration Constants:** Defining a fixed list of supported locales or valid status codes.
- **API Responses:** Returning a read-only list of items from a method.
- **Test Data:** Creating immutable test fixtures that cannot be accidentally modified.

---

## 2. Set.of()

### Definitions

**Core Definition:** `Set.of()` is a static factory method on the `Set` interface that creates an immutable set containing the specified elements.

**Technical Definition:** The `Set.of` static factory methods provide a convenient way to create unmodifiable sets. A set is a collection that does not contain duplicate elements. If a duplicate entry is detected, an `IllegalArgumentException` is thrown. Null values are not allowed.

**Beginner-Friendly Explanation:** `Set.of()` is like a bag of unique items — you cannot have two of the same thing, and once the bag is sealed, you cannot add or remove anything.

### Purposes

- To create a small, immutable set in a single expression
- To enforce uniqueness of elements at creation time
- To provide a thread-safe set that can be shared freely

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Empty set
static <E> Set<E> of()

// Fixed-argument forms (1 to 10 elements)
static <E> Set<E> of(E e1)
static <E> Set<E> of(E e1, E e2)
// ... up to 10 elements

// Varargs form
static <E> Set<E> of(E... elements)
```

**Component Breakdown:**

| Form | Description |
|------|-------------|
| `Set.of()` | Returns an empty immutable set. |
| `Set.of(e1)` | Returns an immutable set containing one element. |
| `Set.of(e1, e2)` | Returns an immutable set containing two elements. |
| `Set.of(elements...)` | Returns an immutable set containing an arbitrary number of elements. |

**Syntax Rules:**

- Duplicate elements cause `IllegalArgumentException` at creation time.
- `null` elements are not permitted.
- The returned set is structurally immutable.

**Constraints and Limitations:**

- Cannot add or remove elements.
- No null values allowed.
- Iteration order is randomized and may differ between JVM runs.

### Annotated Code Example

```java
// File: SetOfDemo.java
import java.util.*;

public class SetOfDemo {
    public static void main(String[] args) {
        // Step 1: Create an immutable set
        Set<String> colors = Set.of("Red", "Green", "Blue");
        System.out.println("Colors: " + colors);

        // Step 2: Duplicate elements throw IllegalArgumentException
        try {
            Set.of("A", "B", "A");
        } catch (IllegalArgumentException e) {
            System.out.println("Duplicate elements not allowed");
        }

        // Step 3: Null elements throw NullPointerException
        try {
            Set.of("A", null);
        } catch (NullPointerException e) {
            System.out.println("Null elements not allowed");
        }

        // Step 4: Cannot modify
        try {
            colors.add("Yellow");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify immutable set");
        }
    }
}
```

**Expected Output:**
```
Colors: [Green, Blue, Red] (order varies)
Duplicate elements not allowed
Null elements not allowed
Cannot modify immutable set
```

**Step-by-Step Explanation:**

1. `Set.of("Red", "Green", "Blue")` creates an immutable set. The iteration order is randomized.
2. `Set.of("A", "B", "A")` throws `IllegalArgumentException` because "A" is duplicated.
3. `Set.of("A", null)` throws `NullPointerException`.
4. `colors.add("Yellow")` throws `UnsupportedOperationException`.

### Real-World Cases

- **Enum-like Constants:** Defining a fixed set of valid operation codes.
- **Permissions:** Representing a user's immutable set of permissions.
- **Unique Tags:** Creating a set of unique tags for content categorization.

---

## 3. Map.of()

### Definitions

**Core Definition:** `Map.of()` is a static factory method on the `Map` interface that creates an immutable map containing the specified key-value pairs.

**Technical Definition:** The `Map.of` static factory methods provide a convenient way to create unmodifiable maps. A Map cannot contain duplicate keys. If a duplicate key is detected, an `IllegalArgumentException` is thrown. Each key is associated with one value. Null cannot be used for either Map keys or values. The fixed-argument form supports up to 10 key-value pairs.

**Beginner-Friendly Explanation:** `Map.of()` is like a sealed dictionary — you provide all the word-definition pairs at once, and then it's locked. You cannot add, remove, or change any entries.

### Purposes

- To create a small, immutable map in a single expression
- To replace the verbose pre-Java 9 pattern of building a map and wrapping it
- To provide a thread-safe map for constant lookup tables

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Empty map
static <K,V> Map<K,V> of()

// Fixed-argument forms (1 to 10 key-value pairs)
static <K,V> Map<K,V> of(K k1, V v1)
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2)
// ... up to 10 key-value pairs
```

**Component Breakdown:**

| Form | Description |
|------|-------------|
| `Map.of()` | Returns an empty immutable map. |
| `Map.of(k1, v1)` | Returns an immutable map with one mapping. |
| `Map.of(k1, v1, k2, v2)` | Returns an immutable map with two mappings. |

**Syntax Rules:**

- Keys and values must be provided in alternating key-value pairs.
- Duplicate keys throw `IllegalArgumentException`.
- `null` keys or values are not permitted.
- The returned map is structurally immutable.

**Constraints and Limitations:**

- Cannot add, remove, or replace mappings.
- No null keys or values allowed.
- Iteration order is randomized.

### Annotated Code Example

```java
// File: MapOfDemo.java
import java.util.*;

public class MapOfDemo {
    public static void main(String[] args) {
        // Step 1: Create an immutable map
        Map<String, Integer> ages = Map.of(
            "Alice", 30,
            "Bob", 25,
            "Charlie", 35
        );
        System.out.println("Ages: " + ages);

        // Step 2: Retrieve a value
        System.out.println("Alice's age: " + ages.get("Alice")); // 30

        // Step 3: Duplicate keys throw IllegalArgumentException
        try {
            Map.of("A", 1, "B", 2, "A", 3);
        } catch (IllegalArgumentException e) {
            System.out.println("Duplicate keys not allowed");
        }

        // Step 4: Null key/value throws NullPointerException
        try {
            Map.of("A", null);
        } catch (NullPointerException e) {
            System.out.println("Null values not allowed");
        }

        // Step 5: Cannot modify
        try {
            ages.put("Diana", 28);
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify immutable map");
        }
    }
}
```

**Expected Output:**
```
Ages: {Bob=25, Alice=30, Charlie=35} (order varies)
Alice's age: 30
Duplicate keys not allowed
Null values not allowed
Cannot modify immutable map
```

**Step-by-Step Explanation:**

1. `Map.of("Alice", 30, "Bob", 25, "Charlie", 35)` creates an immutable map.
2. `ages.get("Alice")` returns `30`.
3. `Map.of("A", 1, "B", 2, "A", 3)` throws `IllegalArgumentException` because "A" is duplicated.
4. `Map.of("A", null)` throws `NullPointerException`.
5. `ages.put("Diana", 28)` throws `UnsupportedOperationException`.

### Real-World Cases

- **Configuration Maps:** Defining a fixed set of configuration key-value pairs.
- **HTTP Status Codes:** Creating an immutable map of status codes to descriptions.
- **Country Codes:** Mapping country codes to country names.

---

## 4. Map.ofEntries()

### Definitions

**Core Definition:** `Map.ofEntries()` is a static factory method that creates an immutable map from a variable number of `Map.Entry` objects, supporting arbitrary numbers of key-value pairs.

**Technical Definition:** If you have more than 10 key-value pairs, then create the map entries using the `Map.entry` method, and pass those objects to the `Map.ofEntries` method. The varargs form supports an arbitrary number of `Entry` objects or an array.

**Beginner-Friendly Explanation:** `Map.ofEntries()` is like `Map.of()` but for when you have more than 10 pairs. You use `Map.entry(key, value)` for each pair and then pass them all to `Map.ofEntries()`.

### Purposes

- To create immutable maps with more than 10 key-value pairs
- To improve readability when many entries are involved
- To enable building maps from computed or collected entries

### Syntax Rules and Structure

**Complete General Syntax:**

```java
static <K,V> Map<K,V> ofEntries(Map.Entry<? extends K, ? extends V>... entries)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Map.entry(K, V)` | Creates a `Map.Entry` object. |
| `entries` | A varargs array of `Map.Entry` objects. |

**Syntax Rules:**

- Each entry must be created with `Map.entry()`.
- Duplicate keys throw `IllegalArgumentException`.
- `null` keys or values are not permitted.
- The returned map is structurally immutable.

**Constraints and Limitations:**

- Cannot add, remove, or replace mappings.
- No null keys or values allowed.

### Annotated Code Example

```java
// File: MapOfEntriesDemo.java
import java.util.*;
import static java.util.Map.entry;

public class MapOfEntriesDemo {
    public static void main(String[] args) {
        // Step 1: Create a map with more than 10 entries
        Map<Integer, String> friendMap = Map.ofEntries(
            entry(1, "Tom"),
            entry(2, "Dick"),
            entry(3, "Harry"),
            entry(4, "Alice"),
            entry(5, "Bob"),
            entry(6, "Charlie"),
            entry(7, "Diana"),
            entry(8, "Eve"),
            entry(9, "Frank"),
            entry(10, "Grace"),
            entry(11, "Henry"),
            entry(12, "Ivy")
        );

        System.out.println("Map size: " + friendMap.size()); // 12
        System.out.println("Friend 5: " + friendMap.get(5)); // Bob
    }
}
```

**Expected Output:**
```
Map size: 12
Friend 5: Bob
```

**Step-by-Step Explanation:**

1. `Map.entry(1, "Tom")` creates a `Map.Entry` object for each key-value pair.
2. `Map.ofEntries(...)` accepts the variable number of entries and creates an immutable map.
3. The map can contain more than 10 entries, unlike `Map.of()`.

### Real-World Cases

- **Large Constant Maps:** Defining mappings with many entries, such as error codes.
- **Test Fixtures:** Creating maps with many test data entries.
- **Configuration Loading:** Building maps from configuration file entries.

---

## 5. List.copyOf()

### Definitions

**Core Definition:** `List.copyOf()` creates an unmodifiable `List` containing the elements of the given `Collection`, in its iteration order.

**Technical Definition:** `List.copyOf(Collection<? extends E> coll)` returns an unmodifiable List containing the elements of the given Collection, in its iteration order. The given Collection must not be null, and it must not contain any null elements. If the given Collection is subsequently modified, the returned List will not reflect such modifications. If the given Collection is already an unmodifiable List, the `copyOf` method simply returns a reference to the original collection.

**Beginner-Friendly Explanation:** `List.copyOf()` is like taking a photograph of a list — the photo is a frozen snapshot that does not change even if the original list changes.

### Purposes

- To create an unmodifiable snapshot of a mutable collection in one step
- To isolate the returned collection from subsequent changes to the original
- To avoid the verbosity of `List.of(array)` when working with existing collections

### Syntax Rules and Structure

```java
static <E> List<E> copyOf(Collection<? extends E> coll)
```

**Syntax Rules:**

- The source collection must not be `null`.
- The source collection must not contain `null` elements.
- If the source is already unmodifiable, the same reference may be returned.
- If the source is mutable, a copy is made.

**Constraints and Limitations:**

- Cannot copy collections containing `null`.
- The returned list is structurally immutable.

### Annotated Code Example

```java
// File: ListCopyOfDemo.java
import java.util.*;

public class ListCopyOfDemo {
    public static void main(String[] args) {
        // Step 1: Create a mutable list
        List<String> mutableList = new ArrayList<>(Arrays.asList("A", "B", "C"));

        // Step 2: Create an unmodifiable snapshot
        List<String> snapshot = List.copyOf(mutableList);
        System.out.println("Snapshot: " + snapshot); // [A, B, C]

        // Step 3: Modify the original — snapshot is unaffected
        mutableList.add("D");
        System.out.println("Original after modification: " + mutableList); // [A, B, C, D]
        System.out.println("Snapshot after modification: " + snapshot); // [A, B, C]

        // Step 4: Cannot modify the snapshot
        try {
            snapshot.add("E");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify snapshot");
        }

        // Step 5: Null elements throw NullPointerException
        try {
            List.copyOf(Arrays.asList("A", null));
        } catch (NullPointerException e) {
            System.out.println("Null elements not allowed");
        }
    }
}
```

**Expected Output:**
```
Snapshot: [A, B, C]
Original after modification: [A, B, C, D]
Snapshot after modification: [A, B, C]
Cannot modify snapshot
Null elements not allowed
```

**Step-by-Step Explanation:**

1. `List.copyOf(mutableList)` creates an unmodifiable copy of the list.
2. Adding "D" to the original does not affect the snapshot.
3. Attempting to modify the snapshot throws `UnsupportedOperationException`.
4. Passing a collection with `null` throws `NullPointerException`.

### Real-World Cases

- **Defensive Copying:** Storing an immutable snapshot of a list passed to a constructor.
- **API Returns:** Returning an unmodifiable copy of an internal list.
- **Thread Safety:** Creating a thread-safe snapshot of a mutable collection.

---

## 6. Set.copyOf()

### Definitions

**Core Definition:** `Set.copyOf()` creates an unmodifiable `Set` containing the elements of the given `Collection`, removing duplicates.

**Technical Definition:** Because the parameter of `Set.copyOf` is `Collection`, you can create an unmodifiable `Set` that contains the elements of a `List`. If you use `Set.copyOf` to create a `Set` from a `List`, and the `List` contains duplicate elements, an exception is not thrown. Instead, an arbitrary one of the duplicate elements is included in the resulting `Set`.

**Beginner-Friendly Explanation:** `Set.copyOf()` is like taking a photograph of a collection and filtering out duplicates — the resulting photo contains only unique items.

### Purposes

- To create an unmodifiable set from any collection
- To remove duplicates while creating an immutable set
- To isolate the returned set from changes to the source

### Syntax Rules and Structure

```java
static <E> Set<E> copyOf(Collection<? extends E> coll)
```

**Syntax Rules:**

- The source collection must not be `null`.
- The source collection must not contain `null` elements.
- Duplicates are silently removed (one arbitrary duplicate is kept).
- If the source is already an unmodifiable `Set`, the same reference may be returned.

**Constraints and Limitations:**

- Cannot copy collections containing `null`.
- Iteration order is randomized.

### Annotated Code Example

```java
// File: SetCopyOfDemo.java
import java.util.*;

public class SetCopyOfDemo {
    public static void main(String[] args) {
        // Step 1: Create a list with duplicates
        List<String> listWithDuplicates = Arrays.asList("A", "B", "A", "C", "B");

        // Step 2: Create an unmodifiable set (duplicates removed)
        Set<String> set = Set.copyOf(listWithDuplicates);
        System.out.println("Set: " + set); // [A, B, C] (order varies)
        System.out.println("Size: " + set.size()); // 3

        // Step 3: Null elements throw NullPointerException
        try {
            Set.copyOf(Arrays.asList("A", null));
        } catch (NullPointerException e) {
            System.out.println("Null elements not allowed");
        }
    }
}
```

**Expected Output:**
```
Set: [A, B, C] (order varies)
Size: 3
Null elements not allowed
```

### Real-World Cases

- **Deduplication:** Creating a unique set from a list of items.
- **API Returns:** Returning an unmodifiable set of unique values.
- **Caching:** Storing an immutable snapshot of unique keys.

---

## 7. Map.copyOf()

### Definitions

**Core Definition:** `Map.copyOf()` creates an unmodifiable `Map` containing the same mappings as the given `Map`.

**Technical Definition:** `Map.copyOf(Map<? extends K, ? extends V> map)` returns an unmodifiable Map containing the same mappings as the given Map. The given Map must not be null, and it must not contain any null keys or values. If the given Map is subsequently modified, the returned Map will not reflect such modifications. If the given Map is already an unmodifiable Map, the copyOf method simply returns a reference to the original.

**Beginner-Friendly Explanation:** `Map.copyOf()` is like taking a photograph of a map — the photo captures all the key-value pairs at that moment and does not change even if the original map changes.

### Purposes

- To create an unmodifiable snapshot of a mutable map
- To isolate the returned map from subsequent changes to the original
- To provide a convenient alternative to `Collections.unmodifiableMap(new HashMap<>(map))`

### Syntax Rules and Structure

```java
static <K,V> Map<K,V> copyOf(Map<? extends K, ? extends V> map)
```

**Syntax Rules:**

- The source map must not be `null`.
- The source map must not contain `null` keys or values.
- If the source is already unmodifiable, the same reference may be returned.
- If the source is mutable, a copy is made.

**Constraints and Limitations:**

- Cannot copy maps containing `null`.
- Iteration order is randomized.

### Annotated Code Example

```java
// File: MapCopyOfDemo.java
import java.util.*;

public class MapCopyOfDemo {
    public static void main(String[] args) {
        // Step 1: Create a mutable map
        Map<String, Integer> mutableMap = new HashMap<>();
        mutableMap.put("A", 1);
        mutableMap.put("B", 2);
        mutableMap.put("C", 3);

        // Step 2: Create an unmodifiable snapshot
        Map<String, Integer> snapshot = Map.copyOf(mutableMap);
        System.out.println("Snapshot: " + snapshot);

        // Step 3: Modify the original — snapshot unaffected
        mutableMap.put("D", 4);
        System.out.println("Original: " + mutableMap);
        System.out.println("Snapshot: " + snapshot); // still has 3 entries

        // Step 4: Cannot modify snapshot
        try {
            snapshot.put("E", 5);
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify snapshot");
        }
    }
}
```

**Expected Output:**
```
Snapshot: {A=1, B=2, C=3} (order varies)
Original: {A=1, B=2, C=3, D=4}
Snapshot: {A=1, B=2, C=3}
Cannot modify snapshot
```

### Real-World Cases

- **Configuration Snapshots:** Creating an immutable snapshot of a configuration map.
- **API Returns:** Returning an unmodifiable copy of an internal map.
- **Caching:** Storing an immutable snapshot of cached key-value pairs.

---

## 8. Immutability

### Definitions

**Core Definition:** Immutability means that once a collection is created, its structure cannot be changed — no elements can be added, removed, or replaced.

**Technical Definition:** A collection is considered unmodifiable if elements cannot be added, removed, or replaced. However, an unmodifiable collection is only immutable if the elements contained in the collection are immutable. To be considered thread-safe, collections created using the static factory methods and `toUnmodifiable` collectors must contain only immutable elements.

**Beginner-Friendly Explanation:** An immutable collection is like a sealed jar. You can look at what's inside, but you cannot add, remove, or swap anything. However, if the items inside the jar are themselves mutable (like a jar of small boxes you can open), you can still change the contents of those items.

### Purposes

- To prevent accidental modification of shared collections
- To enable safe concurrent access without synchronization (when elements are also immutable)
- To provide space efficiency benefits and thread safety
- To support defensive programming by isolating internal state from external modification

### Syntax Rules and Structure

**Key Concepts:**

| Term | Meaning |
|------|---------|
| Structurally immutable | Cannot add, remove, or replace elements. |
| Deeply immutable | Elements themselves are also immutable. |
| Thread-safe | Safe for concurrent access without synchronization (requires immutable elements). |
| Unmodifiable view | A read-only wrapper around a mutable collection (the original can still change). |
| Immutable copy | An independent, unmodifiable snapshot of a collection. |

**Syntax Rules:**

- Static factory methods (`List.of()`, `Set.of()`, `Map.of()`) create immutable collections.
- `copyOf()` methods create immutable copies of existing collections.
- Unmodifiable wrappers (`Collections.unmodifiableList()`) create views, not copies.
- The `toUnmodifiable` collectors create unmodifiable collections from streams.

**Constraints and Limitations:**

- Immutable collections cannot contain `null` elements.
- If elements are mutable, the collection is not deeply immutable.
- Attempts to modify throw `UnsupportedOperationException`.

### Annotated Code Example

```java
// File: ImmutabilityDemo.java
import java.util.*;

public class ImmutabilityDemo {
    public static void main(String[] args) {
        // Structurally immutable but elements are mutable
        List<StringBuilder> list = new ArrayList<>();
        list.add(new StringBuilder("Hello"));

        List<List<StringBuilder>> immutable = List.of(list);
        // immutable.add(...) throws UnsupportedOperationException
        // But we can still modify the StringBuilder inside
        immutable.get(0).get(0).append(" World");
        System.out.println(immutable.get(0).get(0)); // Hello World

        // Deeply immutable (elements are immutable)
        List<String> deeplyImmutable = List.of("A", "B", "C");
        // No modification possible at any level
    }
}
```

**Expected Output:**
```
Hello World
```

### Real-World Cases

- **Concurrency:** Sharing immutable collections across threads without locks.
- **API Design:** Returning immutable collections to prevent external modification.
- **Caching:** Storing immutable snapshots of data.

---

## 9. Immutable Collection Best Practices

### Definitions

**Core Definition:** Best practices for immutable collections are guidelines for choosing when and how to use immutable collections effectively in Java applications.

**Technical Definition:** Best practices include using the Java 9+ factory methods for small collections, using `copyOf()` for creating snapshots of existing collections, preferring immutable collections for constants and configuration data, and being aware of the distinction between unmodifiable views and immutable copies. "An unmodifiable collection provides space efficiency benefits and prevents the collection from accidentally being modified, which might cause the program to work incorrectly".

**Beginner-Friendly Explanation:** Best practices help you decide when to use immutable collections and when to use mutable ones, and how to use the factory methods correctly.

### Key Best Practices

| Practice | Description |
|----------|-------------|
| **Use `List.of()`, `Set.of()`, `Map.of()` for small collections** | Cleanest and fastest way to create immutable collections. |
| **Use `copyOf()` for snapshots** | Creates an isolated, unmodifiable copy of an existing collection in one step. |
| **Use `Map.ofEntries()` for more than 10 pairs** | Provides a readable way to create larger immutable maps. |
| **Prefer immutable collections for constants** | Collections initialized from constants or configuration data are ideal candidates. |
| **Use mutable collections for frequently modified data** | Modifications are performed in-place, avoiding expensive copies. |
| **Avoid null elements** | Immutable collections are null-hostile; use `Optional` or sentinel values instead. |
| **Be aware of randomized iteration order** | Do not depend on iteration order for `Set.of()` or `Map.of()`. |
| **Prefer immutable collections for thread safety** | If elements are immutable, no synchronization is needed. |
| **Use `toUnmodifiable` collectors for streams** | Guarantees the returned collection is unmodifiable. |
| **Use `Collections.unmodifiableList()` for views** | When you want a read-only view backed by a mutable collection. |

### Annotated Code Example

```java
// File: BestPracticesImmutable.java
import java.util.*;
import static java.util.Map.entry;
import java.util.stream.*;

public class BestPracticesImmutable {
    // GOOD: Immutable constant
    public static final List<String> SUPPORTED_LOCALES =
        List.of("en-US", "fr-FR", "de-DE", "ja-JP");

    // GOOD: Immutable map with more than 10 entries
    public static final Map<Integer, String> STATUS_CODES = Map.ofEntries(
        entry(200, "OK"), entry(201, "Created"), entry(400, "Bad Request"),
        entry(401, "Unauthorized"), entry(403, "Forbidden"),
        entry(404, "Not Found"), entry(500, "Internal Server Error"),
        entry(502, "Bad Gateway"), entry(503, "Service Unavailable")
    );

    // GOOD: copyOf for defensive copying
    public static List<String> getSnapshot(Collection<String> source) {
        return List.copyOf(source);
    }

    // GOOD: toUnmodifiable collector from stream
    public static Set<String> getUniqueWords(List<String> lines) {
        return lines.stream()
            .flatMap(line -> Arrays.stream(line.split("\\s+")))
            .collect(Collectors.toUnmodifiableSet());
    }

    public static void main(String[] args) {
        System.out.println("Locales: " + SUPPORTED_LOCALES);
        System.out.println("404: " + STATUS_CODES.get(404));
        System.out.println("Snapshot: " + getSnapshot(Arrays.asList("X", "Y", "Z")));
    }
}
```

**Expected Output:**
```
Locales: [en-US, fr-FR, de-DE, ja-JP]
404: Not Found
Snapshot: [X, Y, Z]
```

### Real-World Cases

- **API Design:** Returning immutable collections from getters to protect internal state.
- **Configuration Management:** Storing application configuration as immutable maps.
- **Concurrent Systems:** Sharing immutable collections across threads without synchronization.
- **Stream Processing:** Collecting stream results into unmodifiable collections.

---

## Comparison Table: Pre-Java 9 vs. Java 9+ Immutable Collections

| Aspect | Pre-Java 9 | Java 9+ |
|--------|-----------|---------|
| **Small List** | `Collections.unmodifiableList(Arrays.asList("a","b"))` | `List.of("a","b")` |
| **Small Set** | `Collections.unmodifiableSet(new HashSet<>(Arrays.asList("a","b")))` | `Set.of("a","b")` |
| **Small Map** | Build map, then wrap | `Map.of("k1", v1, "k2", v2)` |
| **Large Map** | Build map, then wrap | `Map.ofEntries(entry(k1,v1), ...)` |
| **Snapshot** | `Collections.unmodifiableList(new ArrayList<>(list))` | `List.copyOf(list)` |
| **Null handling** | Allowed if underlying collection allows | Disallowed |
| **Duplicate handling** | Varies by collection type | `Set.of`/`Map.of` reject duplicates |
| **Iteration order** | Deterministic (for List) | Randomized (for Set/Map) |
| **Verbosity** | High | Low |

---

## References Links

- JEP 269: Convenience Factory Methods for Collections — OpenJDK - https://openjdk.org/jeps/269
- Creating Unmodifiable Lists, Sets, and Maps — Oracle Core Libraries - https://docs.oracle.com/en/java/javase/11/core/creating-unmodifiable-lists-sets-and-maps.html
- Unmodifiable List Static Factory Methods — Oracle Core Libraries Developer Guide - https://docs.oracle.com/javase/jp/18/core/java-core-libraries-developer-guide.pdf
- List (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html
- Set (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Set.html
- Map (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Map.html
- JEP 186: Collection Literals — OpenJDK - https://openjdk.org/jeps/186
- Java Immutability: Create Immutable Lists, Sets, and Maps (Best Practices) — Cleverence - https://www.cleverence.com/articles/oracle-documentation/java-immutability-4721/
- Creating and Processing Data with the Collections Factory Methods — Dev.java - https://dev.java/learn/api/collections-framework/creating/