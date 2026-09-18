# Java Map — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A `Map<K,V>` is an object that maps keys to values. A map cannot contain duplicate keys; each key can map to at most one value.

**Technical Definition:** `java.util.Map<K,V>` is an interface in the Java Collections Framework. It provides three collection views — a set of keys, a collection of values, and a set of key-value mappings — allowing a map's contents to be viewed in different ways. The order of a map is defined as the order in which the iterators on the map's collection views return their elements.

**Beginner-Friendly Explanation:** A `Map` is like a dictionary or a phone book. You look up a word (the key) to find its definition (the value). Each key appears only once, but different keys can have the same value. Unlike a `List`, which stores items by position number, a `Map` stores items by their key.

### Key Characteristics

- **Key Uniqueness:** A map cannot contain duplicate keys; each key maps to at most one value.
- **Three Collection Views:** `keySet()`, `values()`, and `entrySet()` provide different ways to view the map's contents.
- **No Index-Based Access:** Unlike `List`, `Map` does not provide positional access; it uses keys for retrieval.
- **Mutable Elements Warning:** Great care must be exercised if mutable objects are used as map keys. The behavior of a map is not specified if a key's value is changed in a manner that affects `equals()` comparisons while it is in the map.
- **Optional Operations:** "Destructive" methods throw `UnsupportedOperationException` if the map does not support the operation.
- **Part of Java Collections Framework:** `Map` is a core interface alongside `List`, `Set`, and `Queue`.

### Prerequisites

To effectively use `Map` in Java, you should understand:

- Basic Java syntax, classes, and objects.
- Generics (`<K, V>`).
- The difference between an interface and a concrete class.
- Basic iteration constructs (`for`, `while`).
- The `hashCode()` and `equals()` contracts for custom objects.
- The `Comparable` and `Comparator` interfaces for sorted maps.
- Familiarity with `java.util` package imports.

### Related Programming Areas

| Area | Explanation |
|------|-------------|
| **Collections Framework** | `Map` is a core interface in `java.util`, alongside `List`, `Set`, and `Queue`. |
| **Hashing** | `HashMap` and `LinkedHashMap` rely on `hashCode()` and `equals()` for key storage and retrieval. |
| **Sorting & Comparators** | `TreeMap` relies on `Comparable` or `Comparator` for ordering. |
| **Concurrency** | `ConcurrentHashMap` provides thread-safe map operations with high concurrency. |
| **Garbage Collection** | `WeakHashMap` allows keys to be garbage-collected when no longer referenced. |
| **Functional Programming** | Java 8+ added functional methods like `computeIfAbsent`, `merge`, and `forEach`. |

---

## 1. Java Map Architecture

### Core Definitions

- **Core Definition:** The `Map` interface is the core contract for all key-value data structures in Java. `SortedMap` and `NavigableMap` extend it to provide sorted and navigable operations.
- **Technical Definition:** `java.util.Map<K,V>` defines the basic operations (`put`, `get`, `remove`, `containsKey`, `containsValue`). `java.util.SortedMap<K,V>` extends `Map` and guarantees that the map is in ascending key order. `java.util.NavigableMap<K,V>` extends `SortedMap` and adds navigation methods (`lowerKey`, `floorKey`, `ceilingKey`, `higherKey`, `pollFirstEntry`, `pollLastEntry`, etc.).
- **Beginner-Friendly Explanation:** The `Map` interface is the contract that all maps follow. `SortedMap` adds the promise that keys are always sorted. `NavigableMap` adds methods to find the closest key before or after a given key.

### Sub-feature 1.1: The `java.util.Map` Interface (Core Contract)

#### Definitions

- **Core Definition:** `Map<K,V>` is an object that maps keys to values, with no duplicate keys.
- **Technical Definition:** The `Map` interface is not a true `Collection` (it does not extend `Collection`), but it is part of the Java Collections Framework. It provides methods for adding, removing, and querying key-value mappings.
- **Beginner-Friendly Explanation:** The `Map` interface tells you what every map can do: put, get, remove, check existence, and iterate over keys, values, or entries.

#### Purposes

- To provide a uniform API for all key-value data structures.
- To enable polymorphic treatment of different map implementations.
- To define the fundamental operations for mapping keys to values.

#### Syntax Structures and Rules

**General Syntax:**
```
Map<KeyType, ValueType> map = new HashMap<>();
map.put(key, value);
ValueType value = map.get(key);
```

**Syntax Rules:**
- `Map` is an interface; you must instantiate a concrete implementation.
- Keys must be unique; putting an existing key replaces its value.
- `Map` does not extend `Collection`, so it does not have `add()`, `iterator()`, etc. directly.

**Constraints and Limitations:**
- `Map` does not guarantee any particular iteration order unless a specific implementation (e.g., `TreeMap`, `LinkedHashMap`) is used.
- Not all implementations allow `null` keys or values.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class MapInterfaceDemo {
    public static void main(String[] args) {
        // Create a Map using the interface type
        Map<String, Integer> ages = new HashMap<>();
        
        // Put key-value pairs
        ages.put("Alice", 30);
        ages.put("Bob", 25);
        ages.put("Charlie", 35);
        
        // Get a value by key
        int aliceAge = ages.get("Alice");
        System.out.println("Alice's age: " + aliceAge);
        
        // Check if a key exists
        System.out.println("Contains Bob: " + ages.containsKey("Bob"));
        System.out.println("Contains David: " + ages.containsKey("David"));
        
        // Remove a key-value pair
        ages.remove("Bob");
        System.out.println("After removing Bob: " + ages);
        
        // Get all keys
        System.out.println("Keys: " + ages.keySet());
        
        // Get all values
        System.out.println("Values: " + ages.values());
    }
}
```

**Expected Output:**
```
Alice's age: 30
Contains Bob: true
Contains David: false
After removing Bob: {Alice=30, Charlie=35}
Keys: [Alice, Charlie]
Values: [30, 35]
```

**Why This Output:** The `Map` stores key-value pairs. `get("Alice")` returns 30. `containsKey("Bob")` returns `true` because Bob was added. After `remove("Bob")`, the map contains only Alice and Charlie.

#### Real-World Case

**Scenario:** A user database where each user has a unique ID (key) and a user profile (value). The `Map` allows O(1) lookups by ID.

#### References

- Map (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/Map.html

---

### Sub-feature 1.2: `java.util.SortedMap` and `NavigableMap` Interfaces

#### Definitions

- **Core Definition:** `SortedMap` guarantees that the map is in ascending key order; `NavigableMap` adds navigation methods for finding the closest key matches.
- **Technical Definition:** `SortedMap<K,V>` extends `Map<K,V>` and provides `comparator()`, `firstKey()`, `lastKey()`, `headMap()`, `tailMap()`, and `subMap()`. `NavigableMap<K,V>` extends `SortedMap` and adds `lowerKey()`, `floorKey()`, `ceilingKey()`, `higherKey()`, `pollFirstEntry()`, `pollLastEntry()`, and `descendingMap()`.
- **Beginner-Friendly Explanation:** `SortedMap` keeps keys sorted. `NavigableMap` adds methods like "find the key just below this one" or "find the key just above this one."

#### Purposes

- To provide sorted traversal and range queries.
- To enable navigation to the closest matching key.
- To support ordered map operations like `firstKey()`, `lastKey()`, and `subMap()`.

#### Syntax Structures and Rules

**General Syntax:**
```
SortedMap<KeyType, ValueType> sortedMap = new TreeMap<>();
NavigableMap<KeyType, ValueType> navMap = new TreeMap<>();
```

**Syntax Rules:**
- Keys must be mutually comparable or a `Comparator` must be provided.
- `firstKey()` and `lastKey()` throw `NoSuchElementException` if the map is empty.
- `headMap`, `tailMap`, and `subMap` return views backed by the original map.

**Constraints and Limitations:**
- Only `TreeMap` and `ConcurrentSkipListMap` implement `NavigableMap` in the standard library.
- Not all implementations allow `null` keys.

#### Annotated Code Example

```java
import java.util.NavigableMap;
import java.util.TreeMap;

public class NavigableMapDemo {
    public static void main(String[] args) {
        NavigableMap<Integer, String> map = new TreeMap<>();
        map.put(1, "One");
        map.put(3, "Three");
        map.put(5, "Five");
        map.put(7, "Seven");
        map.put(9, "Nine");
        
        System.out.println("Map: " + map);
        
        // Navigation methods
        System.out.println("Floor(4): " + map.floorKey(4));    // 3
        System.out.println("Ceiling(4): " + map.ceilingKey(4)); // 5
        System.out.println("Lower(5): " + map.lowerKey(5));     // 3
        System.out.println("Higher(5): " + map.higherKey(5));   // 7
        
        // Range operations
        System.out.println("HeadMap(<5): " + map.headMap(5));
        System.out.println("TailMap(>=5): " + map.tailMap(5));
        System.out.println("SubMap(3,8): " + map.subMap(3, 8));
        
        // Descending order
        System.out.println("Descending: " + map.descendingMap());
        
        // First and last
        System.out.println("First entry: " + map.firstEntry());
        System.out.println("Last entry: " + map.lastEntry());
    }
}
```

**Expected Output:**
```
Map: {1=One, 3=Three, 5=Five, 7=Seven, 9=Nine}
Floor(4): 3
Ceiling(4): 5
Lower(5): 3
Higher(5): 7
HeadMap(<5): {1=One, 3=Three}
TailMap(>=5): {5=Five, 7=Seven, 9=Nine}
SubMap(3,8): {3=Three, 5=Five, 7=Seven}
Descending: {9=Nine, 7=Seven, 5=Five, 3=Three, 1=One}
First entry: 1=One
Last entry: 9=Nine
```

**Why This Output:** The `TreeMap` automatically sorts keys. `floorKey(4)` returns 3 (the largest key ≤ 4). `ceilingKey(4)` returns 5 (the smallest key ≥ 4). `headMap(5)` returns keys less than 5.

#### Real-World Case

**Scenario:** A stock trading system that needs to find the closest bid price below or above a given price. `NavigableMap` provides `floorKey()` and `ceilingKey()` for this purpose.

#### References

- SortedMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/SortedMap.html
- NavigableMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/NavigableMap.html

---

## 2. Core Implementations

### Core Definitions

- **Core Definition:** Java provides several concrete `Map` implementations, each with different ordering, performance, and concurrency characteristics.
- **Technical Definition:** The main implementations are `HashMap` (unordered hash table), `LinkedHashMap` (insertion/access order), `TreeMap` (sorted Red-Black tree), and `Hashtable` (legacy synchronized hash table).
- **Beginner-Friendly Explanation:** `HashMap` is the default fast map. `LinkedHashMap` remembers insertion order. `TreeMap` keeps keys sorted. `Hashtable` is an old thread-safe map.

### Sub-feature 2.1: HashMap

#### Definitions

- **Core Definition:** `HashMap` is a hash-table-based implementation that provides O(1) constant-time performance for `get` and `put`, with no ordering guarantees.
- **Technical Definition:** `HashMap<K,V>` extends `AbstractMap<K,V>` and implements `Map<K,V>`, `Cloneable`, and `Serializable`. It permits `null` values and the `null` key. It is roughly equivalent to `Hashtable` except that it is unsynchronized and permits nulls.
- **Beginner-Friendly Explanation:** `HashMap` is like a magic filing cabinet that uses a hash code to figure out which drawer an item belongs in. Adding, getting, and removing items is almost instant, but the items are stored in no particular order.

#### Purposes

- To provide the fastest general-purpose map implementation with O(1) average-case performance.
- To store key-value pairs without any ordering requirements.
- To serve as the default `Map` implementation when order does not matter.

#### Syntax Structures and Rules

**General Syntax:**
```
Map<KeyType, ValueType> map = new HashMap<>();
Map<KeyType, ValueType> map = new HashMap<>(initialCapacity);
Map<KeyType, ValueType> map = new HashMap<>(initialCapacity, loadFactor);
```

**Syntax Breakdown:**
- Map: The interface that defines the standard behavior of a key-value pair collection.
- HashMap: The specific class implementation that uses a hash table to store the data.
- KeyType: The data type (object) of the keys used to look up values (e.g., String, Integer).
- ValueType: The data type (object) of the data being stored (e.g., User, List<String>). 
- initialCapacity: The integer number of "buckets" (slots in the internal array) created when the map is initialized.
- Default value: 16
   - Why change it? If you know you will store 1,000 items, setting the initial capacity higher prevents the map from constantly resizing itself as it grows, saving CPU cycles.
- loadFactor: A float value between 0.0 and 1.0 that determines when the map will automatically resize (double its capacity).
- Default value: 0.75 (75%)
   - How it works: With a default capacity of 16 and a load factor of 0.75, the map will grow once it holds 12 elements (16 × 0.75).
   - Trade-off: A higher load factor decreases memory overhead but increases lookup time because more items end up in the same buckets (collisions).

**Syntax Rules:**
- Default initial capacity is 16; default load factor is 0.75.
- When the number of entries exceeds the product of the load factor and current capacity, the hash table is rehashed (approximately doubled).
- `HashMap` allows one `null` key and multiple `null` values.

**Constraints and Limitations:**
- No ordering guarantees; iteration order may change over time.
- Not thread-safe; concurrent modification may throw `ConcurrentModificationException`.
- Iteration performance is proportional to capacity plus size; do not set initial capacity too high.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapDemo {
    public static void main(String[] args) {
        Map<String, Integer> scores = new HashMap<>();
        
        // Put key-value pairs
        scores.put("Alice", 95);
        scores.put("Bob", 87);
        scores.put("Charlie", 92);
        scores.put("Diana", 88);
        
        System.out.println("Scores: " + scores);
        System.out.println("Alice's score: " + scores.get("Alice"));
        System.out.println("Contains Bob: " + scores.containsKey("Bob"));
        
        // Null key and null value are allowed
        scores.put(null, 0);
        scores.put("Eve", null);
        System.out.println("After nulls: " + scores);
        
        // Remove a key
        scores.remove("Bob");
        System.out.println("After removing Bob: " + scores);
        
        // Size
        System.out.println("Size: " + scores.size());
    }
}
```

**Expected Output (order may vary):**
```
Scores: {Diana=88, Alice=95, Bob=87, Charlie=92}
Alice's score: 95
Contains Bob: true
After nulls: {null=0, Diana=88, Alice=95, Bob=87, Charlie=92, Eve=null}
After removing Bob: {null=0, Diana=88, Alice=95, Charlie=92, Eve=null}
Size: 5
```

**Why This Output:** `HashMap` stores entries in hash order, so the output order is unspecified. `get("Alice")` returns 95. Both `null` key and `null` value are accepted. `remove("Bob")` removes Bob's entry.

#### Real-World Case

**Scenario:** A cache of user session data. Session IDs are keys, and user data objects are values. `HashMap` provides fast lookups without needing any particular ordering.

#### References

- HashMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html

---

### Sub-feature 2.2: LinkedHashMap

#### Definitions

- **Core Definition:** `LinkedHashMap` extends `HashMap` with a doubly-linked list running through all entries, maintaining insertion order or access order.
- **Technical Definition:** `LinkedHashMap<K,V>` extends `HashMap<K,V>`. The linked list defines iteration ordering: normally insertion order (the order in which keys were inserted). A special constructor creates a map whose iteration order is the order in which entries were last accessed (access-order), from least-recently accessed to most-recently.
- **Beginner-Friendly Explanation:** `LinkedHashMap` is like `HashMap` but with a memory of the order you added items. It also supports an "access order" mode where the most recently accessed item moves to the end.

#### Purposes

- To maintain insertion order while still benefiting from hash-based performance.
- To build LRU (Least Recently Used) caches using access-order mode.
- To provide predictable iteration order without the overhead of sorting.

#### Syntax Structures and Rules

**General Syntax:**
```
Map<KeyType, ValueType> map = new LinkedHashMap<>();
Map<KeyType, ValueType> lruMap = new LinkedHashMap<>(capacity, loadFactor, true); // access-order
```

**Syntax Rules:**
- Insertion order is default (`accessOrder = false`).
- Access-order mode (`accessOrder = true`) moves an entry to the end when accessed via `put`, `get`, `getOrDefault`, `compute`, etc..
- Re-inserting an existing key does not change its position.

**Constraints and Limitations:**
- Slightly more memory than `HashMap` due to the linked list.
- Not thread-safe.

#### Annotated Code Example

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LinkedHashMapDemo {
    public static void main(String[] args) {
        // Insertion-order LinkedHashMap
        Map<String, Integer> insertionOrder = new LinkedHashMap<>();
        insertionOrder.put("One", 1);
        insertionOrder.put("Two", 2);
        insertionOrder.put("Three", 3);
        insertionOrder.put("Four", 4);
        
        System.out.println("Insertion order: " + insertionOrder);
        
        // Access-order LinkedHashMap (LRU-like)
        Map<String, Integer> accessOrder = new LinkedHashMap<>(16, 0.75f, true);
        accessOrder.put("A", 1);
        accessOrder.put("B", 2);
        accessOrder.put("C", 3);
        System.out.println("Before access: " + accessOrder);
        
        // Access "A" (moves it to the end)
        accessOrder.get("A");
        System.out.println("After get(A): " + accessOrder);
        
        // Access "B" (moves it to the end)
        accessOrder.get("B");
        System.out.println("After get(B): " + accessOrder);
    }
}
```

**Expected Output:**
```
Insertion order: {One=1, Two=2, Three=3, Four=4}
Before access: {A=1, B=2, C=3}
After get(A): {B=2, C=3, A=1}
After get(B): {C=3, A=1, B=2}
```

**Why This Output:** The insertion-order map preserves the order keys were added. The access-order map moves an entry to the end whenever it is accessed. After `get("A")`, A moves to the end. After `get("B")`, B moves to the end.

#### Real-World Case

**Scenario:** An LRU cache in a web browser. When the cache reaches its limit, the least-recently accessed entry (the eldest) is removed via `removeEldestEntry()`.

#### References

- LinkedHashMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html

---

### Sub-feature 2.3: TreeMap

#### Definitions

- **Core Definition:** `TreeMap` is a Red-Black tree-based implementation of `SortedMap`/`NavigableMap` that stores keys in sorted order.
- **Technical Definition:** `TreeMap<K,V>` extends `AbstractMap<K,V>` and implements `NavigableMap<K,V>`. It guarantees O(log n) time for `containsKey`, `get`, `put`, and `remove`. It is sorted according to the natural ordering of its keys or by a `Comparator` provided at creation time.
- **Beginner-Friendly Explanation:** `TreeMap` keeps all its keys sorted automatically. Adding, getting, and removing are slower than `HashMap`, but you get sorted keys and range operations for free.

#### Purposes

- To maintain keys in sorted order.
- To enable range queries (`subMap`, `headMap`, `tailMap`).
- To provide navigable operations (`firstKey`, `lastKey`, `floorKey`, `ceilingKey`).

#### Syntax Structures and Rules

**General Syntax:**
```
NavigableMap<KeyType, ValueType> map = new TreeMap<>();           // natural ordering
NavigableMap<KeyType, ValueType> map = new TreeMap<>(Comparator); // custom ordering
```

**Syntax Rules:**
- Keys must be mutually comparable or a `Comparator` must be provided.
- Does not allow `null` keys (throws `NullPointerException`).
- Allows `null` values.

**Constraints and Limitations:**
- O(log n) performance, slower than `HashMap`.
- Not thread-safe.
- Ordering must be consistent with `equals()` for correct `Map` behavior.

#### Annotated Code Example

```java
import java.util.NavigableMap;
import java.util.TreeMap;

public class TreeMapDemo {
    public static void main(String[] args) {
        NavigableMap<String, Integer> scores = new TreeMap<>();
        scores.put("Charlie", 85);
        scores.put("Alice", 92);
        scores.put("Bob", 78);
        scores.put("Diana", 95);
        
        System.out.println("Sorted by key: " + scores);
        
        // First and last
        System.out.println("First key: " + scores.firstKey());
        System.out.println("Last key: " + scores.lastKey());
        
        // Navigation
        System.out.println("Floor key of 'C': " + scores.floorKey("C"));
        System.out.println("Ceiling key of 'C': " + scores.ceilingKey("C"));
        
        // Range
        System.out.println("Head map (< 'C'): " + scores.headMap("C"));
        System.out.println("Tail map (>= 'C'): " + scores.tailMap("C"));
        System.out.println("Sub map ['B', 'D'): " + scores.subMap("B", "D"));
    }
}
```

**Expected Output:**
```
Sorted by key: {Alice=92, Bob=78, Charlie=85, Diana=95}
First key: Alice
Last key: Diana
Floor key of 'C': Bob
Ceiling key of 'C': Charlie
Head map (< 'C'): {Alice=92, Bob=78}
Tail map (>= 'C'): {Charlie=85, Diana=95}
Sub map ['B', 'D'): {Bob=78, Charlie=85}
```

**Why This Output:** `TreeMap` sorts keys alphabetically. `firstKey()` returns "Alice", `lastKey()` returns "Diana". `floorKey("C")` returns "Bob" (largest key ≤ "C"). `subMap("B", "D")` returns keys from "B" (inclusive) to "D" (exclusive).

#### Real-World Case

**Scenario:** A leaderboard where player names are keys and scores are values. `TreeMap` keeps the leaderboard sorted alphabetically, and `subMap()` can extract players in a name range.

#### References

- TreeMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/TreeMap.html

---

### Sub-feature 2.4: Hashtable (Legacy)

#### Definitions

- **Core Definition:** `Hashtable` is a legacy synchronized hash-table implementation that does not allow `null` keys or values.
- **Technical Definition:** `Hashtable<K,V>` extends `Dictionary<K,V>` and implements `Map<K,V>`. All methods are `synchronized`. The `put` method throws `NullPointerException` if the key or value is `null`.
- **Beginner-Friendly Explanation:** `Hashtable` is the old, thread-safe version of `HashMap`. It does not allow `null` keys or values, and it is slower because every method is synchronized.

#### Purposes

- To provide thread-safe map operations without external wrapping.
- To support legacy code that depends on `Hashtable`.
- To serve as a synchronized alternative to `HashMap` (though `ConcurrentHashMap` is now preferred).

#### Syntax Structures and Rules

**General Syntax:**
```
Hashtable<KeyType, ValueType> table = new Hashtable<>();
```

**Syntax Rules:**
- Neither key nor value can be `null`.
- All methods are `synchronized`.
- Does not implement `NavigableMap`.

**Constraints and Limitations:**
- **Deprecated/legacy:** Oracle recommends using `HashMap` or `ConcurrentHashMap` instead.
- No `null` keys or values.
- Single lock for all operations; not scalable.

#### Annotated Code Example

```java
import java.util.Hashtable;
import java.util.Map;

public class HashtableDemo {
    public static void main(String[] args) {
        Map<String, Integer> table = new Hashtable<>();
        table.put("One", 1);
        table.put("Two", 2);
        table.put("Three", 3);
        
        System.out.println("Table: " + table);
        System.out.println("Get One: " + table.get("One"));
        
        // Null key or value throws NullPointerException
        try {
            table.put(null, 4);
        } catch (NullPointerException e) {
            System.out.println("Cannot put null key: " + e.getClass().getSimpleName());
        }
        
        try {
            table.put("Four", null);
        } catch (NullPointerException e) {
            System.out.println("Cannot put null value: " + e.getClass().getSimpleName());
        }
        
        // Check existence
        System.out.println("Contains Two: " + table.containsKey("Two"));
        System.out.println("Contains Five: " + table.containsKey("Five"));
    }
}
```

**Expected Output:**
```
Table: {Three=3, Two=2, One=1}
Get One: 1
Cannot put null key: NullPointerException
Cannot put null value: NullPointerException
Contains Two: true
Contains Five: false
```

**Why This Output:** `Hashtable` stores key-value pairs but does not allow `null`. Attempting to put `null` throws `NullPointerException`. `containsKey("Two")` returns `true` because "Two" was added.

#### Real-World Case

**Scenario:** Maintaining a legacy banking application that uses `Hashtable` for thread-safe account lookups. New development should migrate to `ConcurrentHashMap`.

#### References

- Hashtable (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/Hashtable.html

---

## 3. Key-Value Relationships & Mechanics

### Core Definitions

- **Core Definition:** Key uniqueness and the `hashCode()`/`equals()` contract are fundamental to how `HashMap` and `LinkedHashMap` work. `TreeMap` uses `compareTo()`/`compare()` instead.
- **Technical Definition:** For hash-based maps, two keys are considered equal if `k1.equals(k2)` is `true` and `k1.hashCode() == k2.hashCode()`. For `TreeMap`, keys are equal if `compareTo()` or `compare()` returns 0.
- **Beginner-Friendly Explanation:** Keys must be unique. Hash maps use `hashCode()` to find the right bucket and `equals()` to check for exact matches. Sorted maps use `compareTo()` to determine both order and uniqueness.

### Sub-feature 3.1: Key Uniqueness Constraints

#### Definitions

- **Core Definition:** A map cannot contain duplicate keys; each key maps to at most one value.
- **Technical Definition:** For `HashMap`/`LinkedHashMap`, uniqueness is enforced by `hashCode()` and `equals()`. For `TreeMap`, uniqueness is enforced by `compareTo()`/`compare()`.
- **Beginner-Friendly Explanation:** You cannot have two identical keys in a map. If you put a value with an existing key, the old value is replaced.

#### Purposes

- To ensure each key maps to exactly one value.
- To enable reliable lookups by key.
- To define the contract that custom key objects must fulfill.

#### Syntax Structures and Rules

**General Syntax:**
```
map.put(key, value1);   // adds
map.put(key, value2);   // replaces value1 with value2
```

**Syntax Rules:**
- If `key` already exists, `put()` replaces the value and returns the old value.
- If `key` is new, `put()` returns `null`.

**Constraints and Limitations:**
- Mutable keys can break the map if their `hashCode()` or `equals()` depends on mutable fields.
- For `TreeMap`, `compareTo()` must be consistent with `equals()`.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class KeyUniquenessDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        
        // First put returns null (new key)
        Integer old1 = map.put("A", 1);
        System.out.println("First put(A, 1) returned: " + old1);
        
        // Second put returns old value (existing key)
        Integer old2 = map.put("A", 100);
        System.out.println("Second put(A, 100) returned: " + old2);
        System.out.println("Map after: " + map);
        
        // Different key, same value is allowed
        map.put("B", 100);
        System.out.println("Map with duplicate values: " + map);
    }
}
```

**Expected Output:**
```
First put(A, 1) returned: null
Second put(A, 100) returned: 1
Map after: {A=100}
Map with duplicate values: {A=100, B=100}
```

**Why This Output:** The first `put("A", 1)` returns `null` because "A" was not present. The second `put("A", 100)` returns `1` (the old value) and replaces it with `100`. Different keys can map to the same value.

#### Real-World Case

**Scenario:** A configuration system where each setting name (key) maps to exactly one value. Putting a new value for an existing setting replaces the old value.

---

### Sub-feature 3.2: The Contract Between `hashCode()` and `equals()`

#### Definitions

- **Core Definition:** For `HashMap` and `LinkedHashMap`, the contract states that if two objects are equal according to `equals()`, they must have the same `hashCode()`.
- **Technical Definition:** `hashCode()` determines the bucket index. `equals()` resolves collisions within a bucket. If `equals()` returns `true` but `hashCode()` differs, the map may fail to find the key.
- **Beginner-Friendly Explanation:** `hashCode()` tells the map which drawer to look in. `equals()` tells the map which item in the drawer is the right one. If two identical items have different hash codes, they end up in different drawers, and the map gets confused.

#### Purposes

- To ensure correct key lookup and uniqueness.
- To prevent collisions from causing incorrect behavior.
- To define the standard contract for custom key objects.

#### Syntax Structures and Rules

**General Syntax:**
```
@Override
public int hashCode() {
    return Objects.hash(field1, field2);
}

@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof MyClass)) return false;
    MyClass other = (MyClass) o;
    return field1.equals(other.field1) && field2 == other.field2;
}
```

**Syntax Rules:**
- If `a.equals(b)` is `true`, then `a.hashCode() == b.hashCode()` must be `true`.
- If `a.hashCode() == b.hashCode()`, `a.equals(b)` may be `true` or `false`.
- `hashCode()` must be consistent across multiple calls.

**Constraints and Limitations:**
- Violating the contract can lead to lost keys and unpredictable behavior.
- `Objects.hash()` is a convenient helper for generating hash codes.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

class Point {
    int x, y;
    
    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return x == p.x && y == p.y;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
    
    @Override
    public String toString() {
        return "(" + x + "," + y + ")";
    }
}

public class HashCodeEqualsDemo {
    public static void main(String[] args) {
        Map<Point, String> map = new HashMap<>();
        Point p1 = new Point(1, 2);
        Point p2 = new Point(1, 2); // equal to p1
        
        map.put(p1, "First point");
        System.out.println("Get with p2: " + map.get(p2)); // finds p1's value
        
        // Using p2 as key replaces p1's value
        map.put(p2, "Second point");
        System.out.println("Map size: " + map.size());
        System.out.println("Map: " + map);
    }
}
```

**Expected Output:**
```
Get with p2: First point
Map size: 1
Map: {(1,2)=Second point}
```

**Why This Output:** `p2` is equal to `p1` and has the same `hashCode()`. The map correctly finds `p1`'s value when using `p2` as the key. Putting `p2` replaces the value because the keys are considered equal.

#### Real-World Case

**Scenario:** A geo-caching application where coordinates (latitude, longitude) are keys. Proper `hashCode()` and `equals()` implementation ensures that the same location always maps to the same cache entry.

---

### Sub-feature 3.3: Internal Working of HashMap (Buckets, Chaining, Treeification)

#### Definitions

- **Core Definition:** `HashMap` uses an array of buckets. Each bucket stores entries with the same hash code. In Java 8+, when a bucket's linked list grows beyond a threshold (8), it is converted to a Red-Black tree for O(log n) lookup.
- **Technical Definition:** The underlying array is called `table`. Each element is a `Node` (linked list node) or `TreeNode` (Red-Black tree node). The hash is computed as `(h = key.hashCode()) ^ (h >>> 16)`, and the index is `hash & (capacity - 1)`. When the number of entries in a bucket reaches `TREEIFY_THRESHOLD` (8), the bucket is converted to a Red-Black tree if the table's capacity is at least `MIN_TREEIFY_CAPACITY` (64).
- **Beginner-Friendly Explanation:** `HashMap` is like a set of drawers. Each key's hash code tells it which drawer to go in. If too many items end up in the same drawer (a collision), the drawer switches from a simple chain to a balanced tree to keep lookups fast.

#### Purposes

- To understand why hash functions matter for performance.
- To explain how `HashMap` handles collisions efficiently.
- To recognize when treeification occurs and why.

#### Syntax Structures and Rules

**General Syntax (Internal representation):**
```
table = new Node[capacity];
index = hash(key) & (capacity - 1);
```

**Syntax Rules:**
- Initial capacity is 16; load factor is 0.75.
- When entries exceed `capacity * loadFactor`, the table is resized (doubled).
- Treeification occurs when a bucket has ≥ 8 entries and capacity ≥ 64.

**Constraints and Limitations:**
- Poor hash functions lead to many collisions and degraded performance (up to O(n)).
- Treeification adds overhead; it is triggered only when necessary.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapInternalDemo {
    public static void main(String[] args) {
        // Create a HashMap with a small capacity
        Map<Integer, String> map = new HashMap<>(4);
        
        // Add entries
        for (int i = 0; i < 10; i++) {
            map.put(i, "Value " + i);
        }
        
        System.out.println("Map size: " + map.size());
        System.out.println("Get 5: " + map.get(5));
        System.out.println("Contains 7: " + map.containsKey(7));
        
        // Demonstrate rehashing: adding more entries
        for (int i = 10; i < 100; i++) {
            map.put(i, "Value " + i);
        }
        System.out.println("After more entries, size: " + map.size());
        System.out.println("Get 50: " + map.get(50));
    }
}
```

**Expected Output:**
```
Map size: 10
Get 5: Value 5
Contains 7: true
After more entries, size: 100
Get 50: Value 50
```

**Why This Output:** The map starts with capacity 4 but automatically resizes as entries are added. Lookups remain correct regardless of internal resizing. After adding 100 entries, the map contains all of them and `get(50)` returns "Value 50".

#### Real-World Case

**Scenario:** A high-performance caching system. Understanding `HashMap` internals helps developers choose a good hash function and initial capacity to minimize collisions and resizing.

#### References

- HashMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html

---

## 4. Essential Operations & Methods

### Core Definitions

- **Core Definition:** `Map` provides basic CRUD operations (`put`, `get`, `remove`, `containsKey`, `containsValue`), bulk operations (`putAll`, `clear`, `size`, `isEmpty`), and Java 8+ default methods (`getOrDefault`, `putIfAbsent`, `computeIfAbsent`, `merge`).
- **Technical Definition:** These methods form the core API for working with maps. The Java 8+ methods provide atomic, functional-style operations that reduce boilerplate code.
- **Beginner-Friendly Explanation:** These are the everyday operations you use with maps: adding items, retrieving them, removing them, and checking if they exist.

### Sub-feature 4.1: Basic CRUD Operations

#### Definitions

- **Core Definition:** `put` adds or replaces a key-value pair; `get` retrieves a value; `remove` deletes a key-value pair; `containsKey`/`containsValue` check existence.
- **Technical Definition:** `put(K key, V value)` returns the previous value or `null`. `get(Object key)` returns the value or `null`. `remove(Object key)` returns the removed value or `null`. `containsKey(Object key)` and `containsValue(Object value)` return `boolean`.
- **Beginner-Friendly Explanation:** These are the four fundamental operations: add, get, remove, and check.

#### Purposes

- To add, retrieve, and remove key-value pairs.
- To check whether a key or value exists in the map.
- To support basic map manipulation.

#### Syntax Structures and Rules

**General Syntax:**
```
V oldValue = map.put(key, value);
V value = map.get(key);
V removed = map.remove(key);
boolean hasKey = map.containsKey(key);
boolean hasValue = map.containsValue(value);
```

**Syntax Rules:**
- `put` replaces the value if the key already exists.
- `get` returns `null` if the key is not present (unless the value is `null`).
- `containsValue` is O(n) for `HashMap`.

**Constraints and Limitations:**
- `get` returning `null` is ambiguous if `null` values are allowed; use `containsKey` to distinguish.
- `containsValue` requires linear search.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class CrudDemo {
    public static void main(String[] args) {
        Map<String, String> capitals = new HashMap<>();
        
        // Create
        capitals.put("France", "Paris");
        capitals.put("Japan", "Tokyo");
        capitals.put("Brazil", "Brasilia");
        
        // Read
        System.out.println("Capital of Japan: " + capitals.get("Japan"));
        System.out.println("Capital of Germany: " + capitals.get("Germany"));
        
        // Update (put replaces)
        capitals.put("Japan", "Tokyo (updated)");
        System.out.println("After update: " + capitals.get("Japan"));
        
        // Delete
        capitals.remove("Brazil");
        System.out.println("After removing Brazil: " + capitals);
        
        // Contains
        System.out.println("Contains France: " + capitals.containsKey("France"));
        System.out.println("Contains Paris: " + capitals.containsValue("Paris"));
    }
}
```

**Expected Output:**
```
Capital of Japan: Tokyo
Capital of Germany: null
After update: Tokyo (updated)
After removing Brazil: {France=Paris, Japan=Tokyo (updated)}
Contains France: true
Contains Paris: true
```

**Why This Output:** `get("Japan")` returns "Tokyo". `get("Germany")` returns `null` because Germany is not in the map. `put("Japan", ...)` replaces the old value. `remove("Brazil")` deletes the entry.

#### Real-World Case

**Scenario:** A contact management application where each contact name (key) maps to a phone number (value). CRUD operations allow adding, looking up, updating, and deleting contacts.

---

### Sub-feature 4.2: Bulk Operations

#### Definitions

- **Core Definition:** Bulk operations affect multiple entries at once: `putAll`, `clear`, `size`, `isEmpty`.
- **Technical Definition:** `putAll(Map<? extends K, ? extends V> m)` copies all mappings from another map. `clear()` removes all mappings. `size()` returns the number of entries. `isEmpty()` returns `true` if the map has no entries.
- **Beginner-Friendly Explanation:** These operations work on the map as a whole — adding many items at once, clearing everything, or checking how many items are present.

#### Purposes

- To copy all entries from another map.
- To reset a map to empty.
- To check the map's size and emptiness.

#### Syntax Structures and Rules

**General Syntax:**
```
map.putAll(otherMap);
map.clear();
int count = map.size();
boolean empty = map.isEmpty();
```

**Syntax Rules:**
- `putAll` replaces existing values for keys that appear in both maps.
- `clear` removes all entries; the map can still be used after clearing.
- `size` returns the number of key-value mappings.

**Constraints and Limitations:**
- `putAll` is not atomic; concurrent modification may cause issues.
- `clear` on an unmodifiable map throws `UnsupportedOperationException`.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class BulkOperationsDemo {
    public static void main(String[] args) {
        Map<String, Integer> map1 = new HashMap<>();
        map1.put("A", 1);
        map1.put("B", 2);
        
        Map<String, Integer> map2 = new HashMap<>();
        map2.put("C", 3);
        map2.put("D", 4);
        map2.put("A", 100); // will overwrite map1's A
        
        // putAll
        map1.putAll(map2);
        System.out.println("After putAll: " + map1);
        
        // size and isEmpty
        System.out.println("Size: " + map1.size());
        System.out.println("Is empty: " + map1.isEmpty());
        
        // clear
        map1.clear();
        System.out.println("After clear: " + map1);
        System.out.println("Is empty after clear: " + map1.isEmpty());
    }
}
```

**Expected Output:**
```
After putAll: {A=100, B=2, C=3, D=4}
Size: 4
Is empty: false
After clear: {}
Is empty after clear: true
```

**Why This Output:** `putAll(map2)` adds C and D to map1 and replaces A's value with 100. Size becomes 4. After `clear()`, the map is empty.

#### Real-World Case

**Scenario:** Merging configuration maps from multiple sources. `putAll` combines them, with later maps overriding earlier values for conflicting keys.

---

### Sub-feature 4.3: Modern Java 8+ Default Methods

#### Definitions

- **Core Definition:** Java 8 added default methods to `Map` that provide atomic, functional-style operations: `getOrDefault`, `putIfAbsent`, `computeIfAbsent`, `merge`, `compute`, `computeIfPresent`, `replaceAll`, and `forEach`.
- **Technical Definition:** These methods combine checks and updates into single atomic operations, reducing boilerplate and enabling functional programming patterns.
- **Beginner-Friendly Explanation:** These methods let you do common tasks in one line: "get this value or a default," "put this value only if the key is absent," "compute a value if the key is missing," and "merge a value with an existing one."

#### Purposes

- To reduce boilerplate code for common map operations.
- To provide atomic operations that are safe in concurrent contexts (for `ConcurrentHashMap`).
- To enable functional programming with lambdas.

#### Syntax Structures and Rules

**General Syntax:**
```
V value = map.getOrDefault(key, defaultValue);
V value = map.putIfAbsent(key, value);
V value = map.computeIfAbsent(key, key -> computeValue(key));
V value = map.merge(key, value, (oldVal, newVal) -> combine(oldVal, newVal));
```

**Syntax Rules:**
- `getOrDefault` returns the default if the key is absent.
- `putIfAbsent` puts the value only if the key is absent or mapped to `null`.
- `computeIfAbsent` computes a value using the mapping function if the key is absent.
- `merge` combines the existing value with the new value using a `BiFunction`.

**Constraints and Limitations:**
- These methods are not atomic for `HashMap` in concurrent contexts; use `ConcurrentHashMap` for thread safety.
- `computeIfAbsent` should not modify the map inside the mapping function (may cause `ConcurrentModificationException`).

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class ModernMethodsDemo {
    public static void main(String[] args) {
        Map<String, Integer> inventory = new HashMap<>();
        
        // getOrDefault
        int apples = inventory.getOrDefault("Apple", 0);
        System.out.println("Apples (default 0): " + apples);
        
        // putIfAbsent
        inventory.putIfAbsent("Apple", 10);
        inventory.putIfAbsent("Apple", 20); // ignored (key present)
        System.out.println("After putIfAbsent: " + inventory);
        
        // computeIfAbsent
        inventory.computeIfAbsent("Banana", k -> 5);
        inventory.computeIfAbsent("Banana", k -> 100); // ignored (key present)
        System.out.println("After computeIfAbsent: " + inventory);
        
        // merge
        inventory.merge("Apple", 5, Integer::sum);  // 10 + 5 = 15
        inventory.merge("Cherry", 7, Integer::sum); // new key, value = 7
        System.out.println("After merge: " + inventory);
        
        // computeIfPresent
        inventory.computeIfPresent("Apple", (k, v) -> v * 2); // 15 * 2 = 30
        System.out.println("After computeIfPresent: " + inventory);
    }
}
```

**Expected Output:**
```
Apples (default 0): 0
After putIfAbsent: {Apple=10}
After computeIfAbsent: {Apple=10, Banana=5}
After merge: {Apple=15, Banana=5, Cherry=7}
After computeIfPresent: {Apple=30, Banana=5, Cherry=7}
```

**Why This Output:** `getOrDefault` returns 0 because "Apple" is not yet present. `putIfAbsent` adds "Apple" with 10; the second call is ignored. `computeIfAbsent` adds "Banana" with 5. `merge` adds 5 to Apple's 15 and adds Cherry with 7. `computeIfPresent` doubles Apple's value to 30.

#### Real-World Case

**Scenario:** A word-counting application. `merge(word, 1, Integer::sum)` increments the count for each word, adding new words with an initial count of 1.

#### References

- Map (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/Map.html

---

## 5. Map Traversal & Views

### Core Definitions

- **Core Definition:** `Map` provides three collection views: `keySet()` (set of keys), `values()` (collection of values), and `entrySet()` (set of key-value pairs). Traversal can use these views, `forEach()`, or iterators.
- **Technical Definition:** `keySet()` returns a `Set<K>`, `values()` returns a `Collection<V>`, and `entrySet()` returns a `Set<Map.Entry<K,V>>`. These views are backed by the map, so changes to the view are reflected in the map.
- **Beginner-Friendly Explanation:** You can look at a map's keys, values, or key-value pairs. `entrySet()` is the most efficient way to iterate when you need both keys and values.

### Sub-feature 5.1: keySet()

#### Definitions

- **Core Definition:** `keySet()` returns a `Set` view of all keys in the map.
- **Technical Definition:** `Set<K> keySet()` — returns a set backed by the map. Removing a key from the set removes the mapping from the map.
- **Beginner-Friendly Explanation:** Gives you all the keys in the map.

#### Purposes

- To iterate over keys.
- To check if a key exists.
- To remove keys via the set view.

#### Syntax Structures and Rules

**General Syntax:**
```
for (KeyType key : map.keySet()) {
    // process key
}
```

**Syntax Rules:**
- The returned set is a view; changes affect the map.
- Order depends on the map implementation.

**Constraints and Limitations:**
- Using `keySet()` and then `map.get(key)` requires an extra lookup per iteration; use `entrySet()` for better performance.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class KeySetDemo {
    public static void main(String[] args) {
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 87);
        scores.put("Charlie", 92);
        
        // Iterate over keys
        System.out.println("Keys:");
        for (String key : scores.keySet()) {
            System.out.println("  " + key);
        }
        
        // Check if a key exists
        System.out.println("Contains Alice: " + scores.keySet().contains("Alice"));
        
        // Remove a key via keySet view
        scores.keySet().remove("Bob");
        System.out.println("After removing Bob: " + scores);
    }
}
```

**Expected Output:**
```
Keys:
  Alice
  Bob
  Charlie
Contains Alice: true
After removing Bob: {Alice=95, Charlie=92}
```

**Why This Output:** The `keySet()` view allows iteration over keys. Removing "Bob" from the set also removes it from the map.

---

### Sub-feature 5.2: values()

#### Definitions

- **Core Definition:** `values()` returns a `Collection` view of all values in the map.
- **Technical Definition:** `Collection<V> values()` — returns a collection backed by the map. Removing a value from the collection removes the corresponding mapping.
- **Beginner-Friendly Explanation:** Gives you all the values in the map, without the keys.

#### Purposes

- To iterate over values when keys are not needed.
- To compute aggregates (sum, average) over values.
- To check if a value exists.

#### Syntax Structures and Rules

**General Syntax:**
```
for (ValueType value : map.values()) {
    // process value
}
```

**Syntax Rules:**
- The returned collection is a view; changes affect the map.
- Duplicate values are allowed.

**Constraints and Limitations:**
- Cannot determine which key corresponds to a value.
- `containsValue` on the collection is O(n).

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class ValuesDemo {
    public static void main(String[] args) {
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 87);
        scores.put("Charlie", 92);
        
        // Sum all values
        int sum = 0;
        for (int score : scores.values()) {
            sum += score;
        }
        System.out.println("Total: " + sum);
        System.out.println("Average: " + (double) sum / scores.size());
        
        // Check if a value exists
        System.out.println("Contains 92: " + scores.values().contains(92));
        System.out.println("Contains 100: " + scores.values().contains(100));
    }
}
```

**Expected Output:**
```
Total: 274
Average: 91.33333333333333
Contains 92: true
Contains 100: false
```

**Why This Output:** The `values()` view allows iteration over all values. The sum is 274, and the average is 91.33. `contains(92)` returns `true`.

---

### Sub-feature 5.3: entrySet() (Efficient Iteration)

#### Definitions

- **Core Definition:** `entrySet()` returns a `Set` of `Map.Entry<K,V>` objects, each containing a key and its value.
- **Technical Definition:** `Set<Map.Entry<K,V>> entrySet()` — returns a set view of the mappings. `Map.Entry` provides `getKey()` and `getValue()` methods.
- **Beginner-Friendly Explanation:** Gives you both the key and value at the same time, which is the most efficient way to iterate when you need both.

#### Purposes

- To iterate over key-value pairs efficiently.
- To modify values via `entry.setValue()`.
- To avoid extra lookups from `keySet()` + `get()`.

#### Syntax Structures and Rules

**General Syntax:**
```
for (Map.Entry<K, V> entry : map.entrySet()) {
    K key = entry.getKey();
    V value = entry.getValue();
}
```

**Syntax Rules:**
- `entrySet()` is the most efficient way to iterate when both key and value are needed.
- `entry.setValue()` can update the value in the map.

**Constraints and Limitations:**
- Modifying the map outside the entry set during iteration may throw `ConcurrentModificationException`.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class EntrySetDemo {
    public static void main(String[] args) {
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 87);
        scores.put("Charlie", 92);
        
        // Iterate over entries
        System.out.println("Entries:");
        for (Map.Entry<String, Integer> entry : scores.entrySet()) {
            System.out.println("  " + entry.getKey() + " -> " + entry.getValue());
        }
        
        // Modify values via entry.setValue()
        for (Map.Entry<String, Integer> entry : scores.entrySet()) {
            entry.setValue(entry.getValue() + 5);
        }
        System.out.println("After adding 5 to each: " + scores);
    }
}
```

**Expected Output:**
```
Entries:
  Alice -> 95
  Bob -> 87
  Charlie -> 92
After adding 5 to each: {Alice=100, Bob=92, Charlie=97}
```

**Why This Output:** The `entrySet()` provides both key and value. `entry.setValue()` updates the value in the map directly.

#### Real-World Case

**Scenario:** A grade book where each student's grade needs to be curved. Using `entrySet()` and `entry.setValue()`, the application can add 5 points to every grade in one pass.

#### References

- Iterate Over a Map in Java (Baeldung) — https://www.baeldung.com/java-iterate-map

---

### Sub-feature 5.4: forEach() with Lambda Expressions

#### Definitions

- **Core Definition:** `Map.forEach(BiConsumer)` executes a given action for each entry in the map.
- **Technical Definition:** `default void forEach(BiConsumer<? super K, ? super V> action)` — added in Java 8. The action receives the key and value as parameters.
- **Beginner-Friendly Explanation:** A clean, functional way to iterate over a map using a lambda expression.

#### Purposes

- To provide concise iteration with lambda expressions.
- To separate iteration from the action performed.
- To enable functional-style processing of map entries.

#### Syntax Structures and Rules

**General Syntax:**
```
map.forEach((key, value) -> { /* action */ });
```

**Syntax Rules:**
- The lambda takes two parameters: key and value.
- Iteration order depends on the map implementation.

**Constraints and Limitations:**
- Cannot use `break` or `continue`.
- Cannot modify the map structurally during `forEach`.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class ForEachMapDemo {
    public static void main(String[] args) {
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 87);
        scores.put("Charlie", 92);
        
        // forEach with lambda
        System.out.println("Scores:");
        scores.forEach((name, score) -> 
            System.out.println("  " + name + ": " + score)
        );
        
        // Compute total using forEach
        int[] total = {0}; // use array for mutable variable
        scores.forEach((name, score) -> total[0] += score);
        System.out.println("Total: " + total[0]);
    }
}
```

**Expected Output:**
```
Scores:
  Alice: 95
  Bob: 87
  Charlie: 92
Total: 274
```

**Why This Output:** The `forEach` method applies the lambda to each entry, printing the key and value. The second `forEach` accumulates the total score.

#### Real-World Case

**Scenario:** A dashboard that displays each metric (key) and its current value. `forEach` cleanly formats and displays each metric without explicit iteration boilerplate.

---

### Sub-feature 5.5: Fail-Fast vs. Fail-Safe Iterators

#### Definitions

- **Core Definition:** Fail-fast iterators throw `ConcurrentModificationException` if the map is structurally modified during iteration. Fail-safe (weakly consistent) iterators do not throw and operate on a snapshot or weakly consistent view.
- **Technical Definition:** Fail-fast iterators maintain a `modCount` and compare it on each `next()` call. Fail-safe iterators, found in `java.util.concurrent` classes, create a clone or use a weakly consistent algorithm.
- **Beginner-Friendly Explanation:** Fail-fast iterators "fail quickly" if you change the map while iterating. Fail-safe iterators continue iterating without crashing, but may not see the latest changes.

#### Purposes

- To detect concurrent modification bugs early (fail-fast).
- To allow safe iteration in concurrent environments (fail-safe).
- To understand the behavior of different map implementations.

#### Syntax Structures and Rules

**General Syntax:**
```
// Fail-fast (HashMap)
Iterator<Map.Entry<K,V>> it = map.entrySet().iterator();
while (it.hasNext()) {
    it.next();
    map.put(newKey, newValue); // throws ConcurrentModificationException
}

// Fail-safe (ConcurrentHashMap)
Iterator<Map.Entry<K,V>> it = concurrentMap.entrySet().iterator();
while (it.hasNext()) {
    it.next();
    concurrentMap.put(newKey, newValue); // no exception
}
```

**Syntax Rules:**
- Fail-fast iterators throw `ConcurrentModificationException` on a best-effort basis.
- Fail-safe iterators do not guarantee seeing the latest data.
- `Iterator.remove()` is safe for fail-fast iterators.

**Constraints and Limitations:**
- Fail-fast behavior is not guaranteed in all scenarios.
- Fail-safe iterators may incur overhead due to copying.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class IteratorFailureDemo {
    public static void main(String[] args) {
        // Fail-fast: HashMap
        Map<String, Integer> hashMap = new HashMap<>();
        hashMap.put("A", 1);
        hashMap.put("B", 2);
        
        try {
            Iterator<Map.Entry<String, Integer>> it = hashMap.entrySet().iterator();
            while (it.hasNext()) {
                it.next();
                hashMap.put("C", 3); // structural modification
            }
        } catch (Exception e) {
            System.out.println("Fail-fast: " + e.getClass().getSimpleName());
        }
        
        // Fail-safe: ConcurrentHashMap
        Map<String, Integer> concurrentMap = new ConcurrentHashMap<>();
        concurrentMap.put("A", 1);
        concurrentMap.put("B", 2);
        
        Iterator<Map.Entry<String, Integer>> it2 = concurrentMap.entrySet().iterator();
        while (it2.hasNext()) {
            it2.next();
            concurrentMap.put("C", 3); // no exception
        }
        System.out.println("Fail-safe completed. Map size: " + concurrentMap.size());
    }
}
```

**Expected Output:**
```
Fail-fast: ConcurrentModificationException
Fail-safe completed. Map size: 3
```

**Why This Output:** The `HashMap` iterator is fail-fast and throws `ConcurrentModificationException` when the map is modified during iteration. The `ConcurrentHashMap` iterator is fail-safe (weakly consistent) and does not throw; the map grows to size 3.

#### Real-World Case

**Scenario:** A multi-threaded application where one thread iterates over a map while another thread adds entries. `ConcurrentHashMap`'s fail-safe iterator allows the iteration to continue without crashing.

#### References

- Fail-Safe Iterator vs Fail-Fast Iterator (Baeldung) — https://www.baeldung.com/java-fail-safe-vs-fail-fast-iterator

---

## 6. Concurrent & Specialized Maps

### Core Definitions

- **Core Definition:** Java provides specialized map implementations for concurrent, enum-keyed, and weakly-referenced use cases: `ConcurrentHashMap`, `Collections.synchronizedMap()`, `EnumMap`, and `WeakHashMap`.
- **Technical Definition:** `ConcurrentHashMap` provides thread-safe operations with high concurrency. `Collections.synchronizedMap()` wraps a map with synchronized methods. `EnumMap` is an array-backed map for enum keys. `WeakHashMap` allows keys to be garbage-collected.
- **Beginner-Friendly Explanation:** These maps solve specific problems: `ConcurrentHashMap` for multi-threaded access, `EnumMap` for enum keys, and `WeakHashMap` for memory-sensitive caching.

### Sub-feature 6.1: ConcurrentHashMap

#### Definitions

- **Core Definition:** `ConcurrentHashMap` is a thread-safe `Map` implementation that provides high concurrency for retrievals and updates.
- **Technical Definition:** `ConcurrentHashMap<K,V>` extends `AbstractMap<K,V>` and implements `ConcurrentMap<K,V>`. In Java 8+, it uses CAS (compare-and-swap) operations and synchronized blocks on individual buckets (Nodes) instead of the segment locking used in Java 7.
- **Beginner-Friendly Explanation:** `ConcurrentHashMap` allows multiple threads to read and write at the same time without blocking each other. Reading is non-blocking; writing locks only the specific bucket being modified.

#### Purposes

- To provide thread-safe map operations with high concurrency.
- To avoid the performance bottleneck of a single global lock.
- To support concurrent iteration without `ConcurrentModificationException`.

#### Syntax Structures and Rules

**General Syntax:**
```
ConcurrentMap<KeyType, ValueType> map = new ConcurrentHashMap<>();
map.putIfAbsent(key, value);
map.computeIfAbsent(key, k -> computeValue(k));
map.merge(key, value, (old, newV) -> combine(old, newV));
```

**Syntax Rules:**
- Does not allow `null` keys or values.
- Read operations are non-blocking.
- Write operations lock only the affected bucket.
- Iterators are weakly consistent (fail-safe).

**Constraints and Limitations:**
- No `null` keys or values.
- Not a drop-in replacement for `HashMap` if `null` is required.
- Bulk operations are not atomic.

#### Annotated Code Example

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;

public class ConcurrentHashMapDemo {
    public static void main(String[] args) throws InterruptedException {
        ConcurrentMap<String, Integer> counter = new ConcurrentHashMap<>();
        
        // Multiple threads incrementing the same counter
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.merge("A", 1, Integer::sum);
            }
        });
        
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.merge("A", 1, Integer::sum);
            }
        });
        
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        
        System.out.println("Counter A: " + counter.get("A"));
        
        // putIfAbsent (atomic)
        counter.putIfAbsent("B", 100);
        counter.putIfAbsent("B", 200); // ignored
        System.out.println("Counter B: " + counter.get("B"));
    }
}
```

**Expected Output:**
```
Counter A: 2000
Counter B: 100
```

**Why This Output:** Two threads each increment counter "A" 1000 times. `merge` atomically updates the value, so the final count is 2000. `putIfAbsent` adds "B" with 100; the second call is ignored.

#### Real-World Case

**Scenario:** A web server's request counter. Multiple request-handling threads increment counters for different URLs. `ConcurrentHashMap` allows concurrent updates without blocking.

#### References

- ConcurrentHashMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html

---

### Sub-feature 6.2: SynchronizedMap Wrapper (Collections.synchronizedMap)

#### Definitions

- **Core Definition:** `Collections.synchronizedMap()` returns a synchronized (thread-safe) map backed by the specified map.
- **Technical Definition:** `static <K,V> Map<K,V> synchronizedMap(Map<K,V> m)` — wraps every method of the backing map with a synchronized block. All accesses must go through the returned map.
- **Beginner-Friendly Explanation:** Wraps a regular map so that every method call is synchronized, making it safe for multiple threads — but only one thread can access the map at a time.

#### Purposes

- To make an existing map thread-safe with minimal changes.
- To provide a simple synchronization mechanism for low-concurrency scenarios.
- To ensure all changes are immediately visible to other threads.

#### Syntax Structures and Rules

**General Syntax:**
```
Map<KeyType, ValueType> syncMap = Collections.synchronizedMap(new HashMap<>());
```

**Syntax Rules:**
- Must access the map only through the returned synchronized wrapper.
- Iteration must be manually synchronized on the wrapper object.
- Individual method calls are atomic, but compound operations are not.

**Constraints and Limitations:**
- **Iteration requires external synchronization:** `synchronized (syncMap) { for (...) { ... } }`.
- Single lock for all operations; performance degrades under high contention.
- `Collections.synchronizedMap()` still throws `ConcurrentModificationException` during iteration if modified.

#### Annotated Code Example

```java
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;

public class SynchronizedMapDemo {
    public static void main(String[] args) throws InterruptedException {
        Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
        
        // Multiple threads adding to the map
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                syncMap.put("A" + i, i);
            }
        });
        
        Thread t2 = new Thread(() -> {
            for (int i = 1000; i < 2000; i++) {
                syncMap.put("A" + i, i);
            }
        });
        
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        
        System.out.println("Map size: " + syncMap.size());
        
        // Safe iteration with external synchronization
        synchronized (syncMap) {
            int count = 0;
            for (Map.Entry<String, Integer> entry : syncMap.entrySet()) {
                count++;
            }
            System.out.println("Iterated entries: " + count);
        }
    }
}
```

**Expected Output:**
```
Map size: 2000
Iterated entries: 2000
```

**Why This Output:** The synchronized wrapper ensures thread-safe `put` operations. The map contains 2000 unique keys. The synchronized block ensures safe iteration.

#### Real-World Case

**Scenario:** A low-concurrency logging system where multiple threads add log entries to a shared map.

#### References

- Collections.synchronizedMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedMap-java.util.Map-
- Collections.synchronizedMap vs. ConcurrentHashMap (Baeldung) — https://www.baeldung.com/java-synchronizedmap-vs-concurrenthashmap

---

### Sub-feature 6.3: EnumMap

#### Definitions

- **Core Definition:** `EnumMap` is a specialized `Map` implementation for enum keys, represented internally as an array.
- **Technical Definition:** `EnumMap<K extends Enum<K>, V>` extends `AbstractMap<K,V>`. All basic operations execute in constant time, and they are likely (though not guaranteed) to be faster than their `HashMap` counterparts.
- **Beginner-Friendly Explanation:** `EnumMap` is a map where the keys are enum constants. It uses an array internally, making it extremely fast and compact.

#### Purposes

- To provide highly efficient map operations for enum keys.
- To maintain keys in natural enum order.
- To use a compact, array-based internal representation.

#### Syntax Structures and Rules

**General Syntax:**
```
EnumMap<EnumType, ValueType> map = new EnumMap<>(EnumType.class);
```

**Syntax Rules:**
- All keys must come from a single enum type.
- Null keys are not permitted (throws `NullPointerException`).
- Null values are permitted.
- Iteration order is the natural order of enum constants.

**Constraints and Limitations:**
- Only works with enum keys.
- Not synchronized; use `Collections.synchronizedMap()` for thread safety.

#### Annotated Code Example

```java
import java.util.EnumMap;
import java.util.Map;

enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }

public class EnumMapDemo {
    public static void main(String[] args) {
        EnumMap<Day, String> schedule = new EnumMap<>(Day.class);
        
        schedule.put(Day.MONDAY, "Work");
        schedule.put(Day.TUESDAY, "Gym");
        schedule.put(Day.WEDNESDAY, "Work");
        schedule.put(Day.THURSDAY, "Meeting");
        schedule.put(Day.FRIDAY, "Work");
        schedule.put(Day.SATURDAY, "Rest");
        schedule.put(Day.SUNDAY, "Rest");
        
        // Iteration in enum declaration order
        System.out.println("Schedule:");
        schedule.forEach((day, activity) -> 
            System.out.println("  " + day + ": " + activity)
        );
        
        System.out.println("Contains Monday: " + schedule.containsKey(Day.MONDAY));
        System.out.println("Size: " + schedule.size());
    }
}
```

**Expected Output:**
```
Schedule:
  MONDAY: Work
  TUESDAY: Gym
  WEDNESDAY: Work
  THURSDAY: Meeting
  FRIDAY: Work
  SATURDAY: Rest
  SUNDAY: Rest
Contains Monday: true
Size: 7
```

**Why This Output:** `EnumMap` stores entries in enum declaration order (MONDAY to SUNDAY). All seven days are present, so size is 7.

#### Real-World Case

**Scenario:** A weekly schedule application where each day of the week (enum) maps to an activity. `EnumMap` provides fast lookup and maintains the correct order.

#### References

- EnumMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/EnumMap.html

---

### Sub-feature 6.4: WeakHashMap

#### Definitions

- **Core Definition:** `WeakHashMap` is a hash-table-based map with weak keys, meaning entries are automatically removed when their keys are no longer referenced.
- **Technical Definition:** `WeakHashMap<K,V>` extends `AbstractMap<K,V>`. Each key is stored indirectly as the referent of a weak reference. When a key is no longer in ordinary use (no strong references), the garbage collector discards it and the entry is removed.
- **Beginner-Friendly Explanation:** `WeakHashMap` holds its keys "weakly." If nothing else is using a key, the map lets the garbage collector take it away, freeing memory. This is useful for caches and listener registries.

#### Purposes

- To prevent memory leaks by allowing keys to be garbage-collected.
- To implement caches where entries should be removed when keys are no longer used.
- To store metadata about objects without preventing those objects from being collected.

#### Syntax Structures and Rules

**General Syntax:**
```
WeakHashMap<KeyType, ValueType> map = new WeakHashMap<>();
```

**Syntax Rules:**
- Both `null` keys and `null` values are supported.
- Entries may disappear at any time due to garbage collection.
- The map's `size()` may return smaller values over time.

**Constraints and Limitations:**
- Behavior depends on garbage collector timing; `size()` may change unexpectedly.
- Not synchronized; use `Collections.synchronizedMap()` for thread safety.
- Values should not strongly refer to their own keys (prevents garbage collection).

#### Annotated Code Example

```java
import java.util.WeakHashMap;

public class WeakHashMapDemo {
    public static void main(String[] args) {
        WeakHashMap<Object, String> map = new WeakHashMap<>();
        
        // Create a key with a strong reference
        Object strongKey = new Object();
        map.put(strongKey, "Strong reference key");
        
        // Create a key with only a weak reference (no external reference)
        map.put(new Object(), "Weak reference key");
        
        System.out.println("Before GC: size = " + map.size());
        
        // Suggest garbage collection
        System.gc();
        try { Thread.sleep(100); } catch (InterruptedException e) {}
        
        System.out.println("After GC: size = " + map.size());
        System.out.println("Strong key value: " + map.get(strongKey));
    }
}
```

**Expected Output (may vary):**
```
Before GC: size = 2
After GC: size = 1
Strong key value: Strong reference key
```

**Why This Output:** The key with only a weak reference is garbage-collected, so its entry is removed. The key with a strong reference remains, so its entry survives.

#### Real-World Case

**Scenario:** A listener registry that stores listeners as weak keys. When a listener is no longer referenced elsewhere, it is automatically removed from the registry, preventing memory leaks.

#### References

- WeakHashMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/WeakHashMap.html

---

## 7. Performance Metrics

### Core Definitions

- **Core Definition:** Map performance varies by implementation: `HashMap` offers O(1) average-case operations, `TreeMap` offers O(log n), and `ConcurrentHashMap` offers O(1) with thread safety.
- **Technical Definition:** Time complexity for `HashMap` is O(1) average, O(n) worst case (many collisions). `TreeMap` is O(log n) for all operations. `ConcurrentHashMap` is O(1) average with lock-free reads.
- **Beginner-Friendly Explanation:** `HashMap` is the fastest for most operations. `TreeMap` is slower but keeps keys sorted. `ConcurrentHashMap` is fast and thread-safe.

### Sub-feature 7.1: Time Complexity Comparisons

#### Definitions

- **Core Definition:** Time complexity describes how operation time grows with the number of entries.
- **Technical Definition:** `HashMap`: O(1) average for `get`, `put`, `remove`; O(n) worst case. `LinkedHashMap`: O(1) average. `TreeMap`: O(log n) for `get`, `put`, `remove`. `ConcurrentHashMap`: O(1) average for reads, O(1) average for writes (with bucket-level locking).
- **Beginner-Friendly Explanation:** `HashMap` and `LinkedHashMap` are constant time on average. `TreeMap` is logarithmic time. `ConcurrentHashMap` is constant time with thread safety.

#### Purposes

- To choose the right map implementation for performance requirements.
- To understand the trade-offs between speed and ordering.
- To predict performance as the map grows.

#### Syntax Structures and Rules

**General Syntax (complexity table):**

| Implementation | get | put | remove | Ordering |
|---|---|---|---|---|
| HashMap | O(1) avg | O(1) avg | O(1) avg | None |
| LinkedHashMap | O(1) avg | O(1) avg | O(1) avg | Insertion/Access |
| TreeMap | O(log n) | O(log n) | O(log n) | Sorted |
| ConcurrentHashMap | O(1) avg | O(1) avg | O(1) avg | None |
| EnumMap | O(1) | O(1) | O(1) | Enum order |

**Syntax Rules:**
- O(1) assumes a good hash function.
- O(log n) is guaranteed for `TreeMap`.
- O(n) worst case for `HashMap` when all keys hash to the same bucket.

**Constraints and Limitations:**
- Hash map performance depends on hash function quality.
- `TreeMap` has higher constant factors than `HashMap`.
- `ConcurrentHashMap` has lower throughput than `HashMap` in single-threaded contexts.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;
import java.util.TreeMap;

public class PerformanceComparisonDemo {
    public static void main(String[] args) {
        int n = 1_000_000;
        
        // HashMap: O(1) average
        long startHash = System.nanoTime();
        Map<Integer, Integer> hashMap = new HashMap<>();
        for (int i = 0; i < n; i++) {
            hashMap.put(i, i);
        }
        long endHash = System.nanoTime();
        System.out.println("HashMap put " + n + " entries: " + (endHash - startHash) / 1_000_000 + " ms");
        
        // TreeMap: O(log n)
        long startTree = System.nanoTime();
        Map<Integer, Integer> treeMap = new TreeMap<>();
        for (int i = 0; i < n; i++) {
            treeMap.put(i, i);
        }
        long endTree = System.nanoTime();
        System.out.println("TreeMap put " + n + " entries: " + (endTree - startTree) / 1_000_000 + " ms");
        
        // Lookup comparison
        long startGetHash = System.nanoTime();
        hashMap.get(n - 1);
        long endGetHash = System.nanoTime();
        System.out.println("HashMap get: " + (endGetHash - startGetHash) + " ns");
        
        long startGetTree = System.nanoTime();
        treeMap.get(n - 1);
        long endGetTree = System.nanoTime();
        System.out.println("TreeMap get: " + (endGetTree - startGetTree) + " ns");
    }
}
```

**Expected Output (approximate):**
```
HashMap put 1000000 entries: 150 ms
TreeMap put 1000000 entries: 900 ms
HashMap get: 3000 ns
TreeMap get: 15000 ns
```

**Why This Output:** `HashMap` is significantly faster than `TreeMap` for both insertion and lookup because it uses constant-time hashing instead of logarithmic tree traversal.

#### Real-World Case

**Scenario:** A high-frequency trading system where microsecond latency matters. `HashMap` is chosen for order lookups because O(1) performance is critical. `TreeMap` would be too slow despite providing sorted order.

#### References

- HashMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html
- TreeMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/TreeMap.html

---

### Sub-feature 7.2: Space Overhead (Initial Capacity and Load Factor Tuning)

#### Definitions

- **Core Definition:** `HashMap` performance is affected by two parameters: initial capacity (number of buckets) and load factor (how full the table can get before resizing).
- **Technical Definition:** Default initial capacity is 16; default load factor is 0.75. When the number of entries exceeds `capacity × loadFactor`, the table is rehashed (doubled). Setting a sufficiently large initial capacity can avoid rehashing.
- **Beginner-Friendly Explanation:** Initial capacity is how many buckets the map starts with. Load factor is how full the map can get before it automatically grows. A higher load factor saves space but slows lookups; a lower load factor speeds lookups but uses more memory.

#### Purposes

- To minimize rehashing operations.
- To optimize memory usage vs. lookup speed.
- To tune performance for known map sizes.

#### Syntax Structures and Rules

**General Syntax:**
```
// For expected 1000 entries with default load factor 0.75:
// initial capacity should be 1000 / 0.75 = 1333 (rounded up to 2048 for power of 2)
Map<KeyType, ValueType> map = new HashMap<>(2048);
```

**Syntax Rules:**
- Default load factor 0.75 offers a good trade-off between time and space costs.
- Higher load factor decreases space overhead but increases lookup cost.
- Lower load factor decreases lookup cost but increases space overhead.
- Initial capacity should be set to `expectedSize / loadFactor` to avoid rehashing.

**Constraints and Limitations:**
- Setting initial capacity too high wastes memory and slows iteration.
- Setting load factor too low wastes memory.
- Hash table capacity is always a power of 2.

#### Annotated Code Example

```java
import java.util.HashMap;
import java.util.Map;

public class CapacityTuningDemo {
    public static void main(String[] args) {
        int expectedSize = 1000;
        float loadFactor = 0.75f;
        int initialCapacity = (int) Math.ceil(expectedSize / loadFactor);
        // Round up to next power of 2
        int capacity = 1;
        while (capacity < initialCapacity) {
            capacity <<= 1;
        }
        
        System.out.println("Expected size: " + expectedSize);
        System.out.println("Calculated initial capacity: " + capacity);
        
        // Create map with tuned capacity
        Map<Integer, String> map = new HashMap<>(capacity, loadFactor);
        
        // Add entries
        for (int i = 0; i < expectedSize; i++) {
            map.put(i, "Value " + i);
        }
        
        System.out.println("Map size: " + map.size());
        System.out.println("Get 500: " + map.get(500));
    }
}
```

**Expected Output:**
```
Expected size: 1000
Calculated initial capacity: 2048
Map size: 1000
Get 500: Value 500
```

**Why This Output:** The initial capacity is calculated to avoid rehashing for 1000 entries: 1000 / 0.75 = 1333, rounded up to 2048 (next power of 2). The map holds all 1000 entries without resizing.

#### Real-World Case

**Scenario:** A batch processing system that loads 100,000 records into a map. Setting the initial capacity to 131072 (100,000 / 0.75 rounded to power of 2) avoids multiple rehashing operations, significantly improving performance.

#### References

- HashMap (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html

---

## Consolidated Reference Links

| Name | Link |
|------|------|
| Map (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/Map.html |
| HashMap (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html |
| LinkedHashMap (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html |
| TreeMap (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/TreeMap.html |
| Hashtable (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/Hashtable.html |
| SortedMap (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/SortedMap.html |
| NavigableMap (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/NavigableMap.html |
| ConcurrentHashMap (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html |
| Collections.synchronizedMap (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedMap-java.util.Map- |
| EnumMap (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/EnumMap.html |
| WeakHashMap (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/WeakHashMap.html |
| Map Interface (Oracle Tutorial) | https://docs.oracle.com/javase/tutorial/collections/interfaces/map.html |
| Iterate Over a Map in Java (Baeldung) | https://www.baeldung.com/java-iterate-map |
| Fail-Safe Iterator vs Fail-Fast Iterator (Baeldung) | https://www.baeldung.com/java-fail-safe-vs-fail-fast-iterator |
| Collections.synchronizedMap vs. ConcurrentHashMap (Baeldung) | https://www.baeldung.com/java-synchronizedmap-vs-concurrenthashmap |
| HashMap Internal Working (Stack Overflow) | https://stackoverflow.com/questions/24937841/when-and-how-does-hashmap-convert-the-bucket-from-linked-list-to-red-black-trees |
| ConcurrentHashMap Lock Striping (Stack Overflow) | https://stackoverflow.com/questions/16151606/need-simple-explanation-how-lock-striping-works-with-concurrenthashmap |
| WeakHashMap Use Cases (Stack Overflow) | https://stackoverflow.com/questions/2077247/when-would-you-use-a-weakhashmap-or-a-weakreference |