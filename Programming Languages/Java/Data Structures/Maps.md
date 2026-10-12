# Java Maps: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A `Map` is an object that maps keys to values, where each key can map to at most one value. A map cannot contain duplicate keys.

**Technical Definition:** Per the Java API specification, "An object that maps keys to values. A map cannot contain duplicate keys; each key can map to at most one value". The `Map` interface takes the place of the `Dictionary` class, which was a totally abstract class rather than an interface. The `Map` interface provides three collection views, which allow a map's contents to be viewed as a set of keys, a collection of values, or a set of key-value mappings. The order of a map is defined as the order in which the iterators on the map's collection views return their elements; some implementations, like `TreeMap`, make specific guarantees as to their order, while others, like `HashMap`, do not.

**Beginner-Friendly Explanation:** A `Map` is like a dictionary or a phone book. You look up a word (the key) and find its definition (the value). Each key is unique — you cannot have two entries for the same word. Java provides many different types of maps for different needs: some are fast and unordered, some keep keys sorted, some remember insertion order, and some are designed for safe use by multiple threads.

### Key Characteristics

- **Key-Value Pairs:** Maps store associations between keys and values.
- **Unique Keys:** A map cannot contain duplicate keys; each key maps to at most one value.
- **Three Collection Views:** The `Map` interface provides `keySet()`, `values()`, and `entrySet()` for viewing the map's contents.
- **No Duplicate Keys:** Adding a value with an existing key overwrites the previous value.
- **Mutable Object Warning:** "Great care must be exercised if mutable objects are used as map keys. The behavior of a map is not specified if the value of an object is changed in a manner that affects `equals` comparisons while the object is a key in the map".
- **Two Hierarchies:** `Map` is separate from the `Collection` hierarchy; it does not extend `Collection`.

### Prerequisites

- Basic understanding of Java classes, objects, and interfaces
- Familiarity with generics (`Map<String, Integer>`, etc.)
- Knowledge of the `equals()` and `hashCode()` contracts (critical for correct Map behaviour)
- Awareness of the `java.util` and `java.util.concurrent` packages

### Related Programming Areas

- **Collections Framework:** `Map` is a core interface in the Java Collections Framework.
- **Stream API:** Maps integrate with Java 8 Streams for functional-style processing.
- **Concurrency:** `ConcurrentHashMap` and `ConcurrentSkipListMap` provide thread-safe map operations.
- **Caching:** `LinkedHashMap` and `WeakHashMap` are used to implement caches.
- **Database Access:** Maps model rows from database queries and configuration properties.

---

## Core Concepts / Features

---

## 1. Map Interface

### Definitions

**Core Definition:** The `Map` interface is the abstract data type that defines the contract for objects that map keys to values.

**Technical Definition:** `Map<K,V>` is a generic interface in the `java.util` package. It is not a subtype of `Collection`. It provides methods for adding, removing, and querying key-value mappings. All general-purpose map implementation classes should provide two "standard" constructors: a void (no arguments) constructor, and a constructor with a single argument of type `Map`, which creates a new map with the same key-value mappings as its argument.

**Beginner-Friendly Explanation:** The `Map` interface is the "rulebook" for all map types. It says what you can do with a map — put items in, get items out, check if a key exists, and so on.

### Purposes

- To define a standard contract for key-value data structures
- To enable polymorphic use of different map implementations
- To provide efficient key-based lookup
- To support three different views of the map's contents

### Syntax Rules and Structure

```java
public interface Map<K, V> {
    // Basic operations
    V put(K key, V value);
    V get(Object key);
    V remove(Object key);
    boolean containsKey(Object key);
    boolean containsValue(Object value);
    int size();
    boolean isEmpty();
    void clear();

    // Collection views
    Set<K> keySet();
    Collection<V> values();
    Set<Map.Entry<K, V>> entrySet();

    // Default methods (Java 8+)
    default V getOrDefault(Object key, V defaultValue) { ... }
    default void forEach(BiConsumer<? super K, ? super V> action) { ... }
    default V putIfAbsent(K key, V value) { ... }
    default V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction) { ... }
    default V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) { ... }
    default V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) { ... }
    default V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction) { ... }
    default V replace(K key, V value) { ... }
    default boolean replace(K key, V oldValue, V newValue) { ... }
    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) { ... }
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `put(K, V)` | Associates a value with a key. |
| `get(Object)` | Returns the value for a key, or `null`. |
| `remove(Object)` | Removes the mapping for a key. |
| `containsKey(Object)` | Checks if a key exists. |
| `containsValue(Object)` | Checks if a value exists. |
| `keySet()` | Returns a `Set` of all keys. |
| `values()` | Returns a `Collection` of all values. |
| `entrySet()` | Returns a `Set` of key-value pairs. |
| `getOrDefault(K, V)` | Returns the value or a default if the key is absent. |
| `putIfAbsent(K, V)` | Puts the value only if the key is absent. |
| `computeIfAbsent(K, Function)` | Computes a value if the key is absent. |
| `computeIfPresent(K, BiFunction)` | Computes a new value if the key is present. |
| `compute(K, BiFunction)` | Computes a new value for the key. |
| `merge(K, V, BiFunction)` | Merges the value with the existing value. |
| `replace(K, V)` | Replaces the value for a key if present. |
| `replaceAll(BiFunction)` | Replaces all values using a function. |

**Syntax Rules:**

- `Map` is a generic interface (`Map<K, V>`).
- `Map` does not extend `Collection`.
- Keys must be unique; values can be duplicated.

**Constraints and Limitations:**

- Mutable keys can cause unpredictable behaviour if their `hashCode` or `equals` changes.
- Some implementations prohibit `null` keys or values.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Map Operations**

```java
// File: MapInterfaceDemo.java
import java.util.*;

public class MapInterfaceDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();

        // Adding key-value pairs
        map.put("Alice", 30);
        map.put("Bob", 25);
        map.put("Charlie", 35);
        map.put("Alice", 31); // Overwrites previous value
        System.out.println("Map: " + map);

        // Retrieving values
        System.out.println("Alice's age: " + map.get("Alice"));
        System.out.println("Contains Bob? " + map.containsKey("Bob"));

        // Collection views
        System.out.println("Keys: " + map.keySet());
        System.out.println("Values: " + map.values());

        // Iterating over entries
        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " -> " + entry.getValue());
        }
    }
}
```

**Expected Output:**
```
Map: {Alice=31, Bob=25, Charlie=35}
Alice's age: 31
Contains Bob? true
Keys: [Alice, Bob, Charlie]
Values: [31, 25, 35]
Alice -> 31
Bob -> 25
Charlie -> 35
```

**Step-by-Step Explanation:**

1. `map.put("Alice", 30)` adds a mapping.
2. `map.put("Alice", 31)` overwrites the previous value.
3. `map.get("Alice")` returns 31.
4. `map.keySet()` returns a `Set` of keys.
5. `map.entrySet()` allows iteration over key-value pairs.

### Real-World Cases

- **Configuration:** A `Map<String, String>` for application settings.
- **Cache:** A `Map<String, Object>` for caching expensive results.
- **Database Index:** A `Map<ID, Entity>` for fast entity lookup.

---

## 2. HashMap

### Definitions

**Core Definition:** `HashMap` is the most commonly used `Map` implementation, backed by a hash table, offering constant-time performance for basic operations.

**Technical Definition:** `HashMap<K,V>` extends `AbstractMap<K,V>` and implements `Map<K,V>`, `Cloneable`, and `Serializable`. It is "the standard general-purpose Map implementation. It stores its entries in a hash table and provides constant-time performance for the basic operations (`get` and `put`), assuming the hash function disperses the elements properly among the buckets". It permits one `null` key and multiple `null` values. It makes no guarantees about iteration order.

**Beginner-Friendly Explanation:** `HashMap` is the default, fastest map. It uses a hash table to store entries, which makes adding, removing, and looking up values very fast. It does not keep entries in any particular order.

### Purposes

- To provide fast, constant-time performance for basic map operations
- To serve as the default general-purpose map implementation
- To store key-value pairs without concern for ordering

### Syntax Rules and Structure

```java
public class HashMap<K,V> extends AbstractMap<K,V>
        implements Map<K,V>, Cloneable, Serializable {
    public HashMap();
    public HashMap(int initialCapacity);
    public HashMap(int initialCapacity, float loadFactor);
    public HashMap(Map<? extends K, ? extends V> m);
}
```

**Component Breakdown:**

| Constructor | Description |
|-------------|-------------|
| `HashMap()` | Creates an empty map with default capacity (16) and load factor (0.75). |
| `HashMap(int initialCapacity)` | Creates an empty map with specified initial capacity. |
| `HashMap(Map<? extends K, ? extends V> m)` | Creates a map with the same mappings as the given map. |

**Syntax Rules:**

- `HashMap` is generic (`HashMap<K, V>`).
- It permits one `null` key and multiple `null` values.
- It is not synchronized; for concurrent access, use `ConcurrentHashMap`.

**Constraints and Limitations:**

- Makes no guarantees about iteration order.
- Iteration performance depends on initial capacity — do not set it too high.

### Annotated Code Example

```java
// File: HashMapDemo.java
import java.util.*;

public class HashMapDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();

        map.put("Apple", 10);
        map.put("Banana", 20);
        map.put("Cherry", 30);
        map.put(null, 99); // null key is permitted

        System.out.println("Map: " + map);
        System.out.println("Size: " + map.size());
        System.out.println("Contains Apple? " + map.containsKey("Apple"));
        System.out.println("Value for null: " + map.get(null));

        // Iteration (order is unspecified)
        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " = " + entry.getValue());
        }
    }
}
```

**Expected Output (order may vary):**
```
Map: {null=99, Apple=10, Banana=20, Cherry=30}
Size: 4
Contains Apple? true
Value for null: 99
null = 99
Apple = 10
Banana = 20
Cherry = 30
```

**Step-by-Step Explanation:**

1. `map.put("Apple", 10)` adds a mapping.
2. `map.put(null, 99)` adds a `null` key (HashMap permits it).
3. `map.containsKey("Apple")` returns `true`.
4. Iteration order is unspecified.

### Real-World Cases

- **General-Purpose Storage:** The default choice for most map needs.
- **Caching:** Storing computed values keyed by input parameters.
- **Counting:** Counting occurrences of words or events.

---

## 3. LinkedHashMap

### Definitions

**Core Definition:** `LinkedHashMap` is a hash table and linked list implementation of the `Map` interface with predictable iteration order.

**Technical Definition:** `LinkedHashMap<K,V>` extends `HashMap<K,V>` and maintains a doubly-linked list running through all of its entries. This linked list defines the encounter order (iteration order), which is normally the order in which keys were inserted into the map (insertion-order). It also provides the `removeEldestEntry` method, which may be overridden to impose a policy for removing stale mappings automatically when new mappings are added to the map. This makes it very easy to implement a custom cache.

**Beginner-Friendly Explanation:** `LinkedHashMap` is like `HashMap` but it remembers the order you added entries. When you iterate over it, entries come out in the order they were inserted.

### Purposes

- To provide a map with predictable, insertion-ordered iteration
- To implement LRU (Least Recently Used) caches using the `removeEldestEntry` hook
- To combine the performance of `HashMap` with deterministic iteration

### Syntax Rules and Structure

```java
public class LinkedHashMap<K,V> extends HashMap<K,V>
        implements Map<K,V> {
    public LinkedHashMap();
    public LinkedHashMap(int initialCapacity);
    public LinkedHashMap(int initialCapacity, float loadFactor);
    public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder);
    public LinkedHashMap(Map<? extends K, ? extends V> m);
}
```

**Syntax Rules:**

- `LinkedHashMap` extends `HashMap`.
- It permits one `null` key and multiple `null` values.
- Iteration order is insertion order (or access order if `accessOrder` is `true`).
- `removeEldestEntry` can be overridden to implement a cache eviction policy.

**Constraints and Limitations:**

- Slightly slower than `HashMap` due to the added linked list.
- Iteration is proportional to the map's size, not its capacity.

### Annotated Code Example

```java
// File: LinkedHashMapDemo.java
import java.util.*;

public class LinkedHashMapDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new LinkedHashMap<>();

        map.put("Cherry", 30);
        map.put("Apple", 10);
        map.put("Banana", 20);
        map.put("Apple", 11); // Overwrites value, order unchanged

        // Iteration order is insertion order
        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " = " + entry.getValue());
        }
        System.out.println("Map: " + map);
    }
}
```

**Expected Output:**
```
Cherry = 30
Apple = 11
Banana = 20
Map: {Cherry=30, Apple=11, Banana=20}
```

**Step-by-Step Explanation:**

1. `map.put("Cherry", 30)` adds "Cherry" first.
2. `map.put("Apple", 10)` adds "Apple" second.
3. `map.put("Banana", 20)` adds "Banana" third.
4. Re-putting "Apple" overwrites the value but does not change the order.
5. Iteration follows insertion order: Cherry, Apple, Banana.

### Real-World Cases

- **LRU Caches:** Implementing a cache with automatic eviction of the least recently used entry.
- **Maintaining Order:** When you need to remember the order in which entries were added.

---

## 4. TreeMap

### Definitions

**Core Definition:** `TreeMap` is a `NavigableMap` implementation based on a Red-Black tree, storing entries in sorted order by key.

**Technical Definition:** `TreeMap<K,V>` extends `AbstractMap<K,V>` and implements `NavigableMap<K,V>`, `Cloneable`, and `Serializable`. The map is sorted according to the natural ordering of its keys, or by a `Comparator` provided at map creation time. It provides guaranteed `log(n)` time cost for the `containsKey`, `get`, `put`, and `remove` operations. It does **not** permit `null` keys (for natural ordering) but permits `null` values.

**Beginner-Friendly Explanation:** `TreeMap` keeps its keys sorted automatically. When you iterate over it, entries come out in sorted order (by key).

### Purposes

- To maintain entries in sorted key order
- To provide navigation methods (firstKey, lastKey, headMap, tailMap, subMap)
- To implement ordered map operations

### Syntax Rules and Structure

```java
public class TreeMap<K,V> extends AbstractMap<K,V>
        implements NavigableMap<K,V>, Cloneable, Serializable {
    public TreeMap();
    public TreeMap(Comparator<? super K> comparator);
    public TreeMap(Map<? extends K, ? extends V> m);
    public TreeMap(SortedMap<K, ? extends V> m);
}
```

**Syntax Rules:**

- Keys must implement `Comparable` (for natural ordering) or a `Comparator` must be provided.
- The ordering must be consistent with `equals` for correct `Map` behaviour.
- It does **not** permit `null` keys (with natural ordering); `null` values are permitted.

**Constraints and Limitations:**

- `log(n)` time for basic operations (slower than `HashMap`'s constant time).
- Not synchronized; use `Collections.synchronizedSortedMap()` for concurrent access.

### Annotated Code Example

```java
// File: TreeMapDemo.java
import java.util.*;

public class TreeMapDemo {
    public static void main(String[] args) {
        TreeMap<Integer, String> map = new TreeMap<>();

        map.put(5, "Five");
        map.put(2, "Two");
        map.put(8, "Eight");
        map.put(1, "One");
        map.put(9, "Nine");

        System.out.println("Map (sorted by key): " + map);
        System.out.println("First key: " + map.firstKey());
        System.out.println("Last key: " + map.lastKey());
        System.out.println("HeadMap (< 5): " + map.headMap(5));
        System.out.println("TailMap (>= 5): " + map.tailMap(5));
    }
}
```

**Expected Output:**
```
Map (sorted by key): {1=One, 2=Two, 5=Five, 8=Eight, 9=Nine}
First key: 1
Last key: 9
HeadMap (< 5): {1=One, 2=Two}
TailMap (>= 5): {5=Five, 8=Eight, 9=Nine}
```

**Step-by-Step Explanation:**

1. Entries are inserted in arbitrary order.
2. The `TreeMap` stores them sorted by key: 1, 2, 5, 8, 9.
3. `firstKey()` returns 1, `lastKey()` returns 9.
4. `headMap(5)` returns entries with keys less than 5.
5. `tailMap(5)` returns entries with keys greater than or equal to 5.

### Real-World Cases

- **Sorted Data:** Maintaining a sorted list of scores, prices, or names.
- **Range Queries:** Finding all entries within a key range.
- **Leaderboards:** Keeping high scores sorted by key.

---

## 5. Hashtable (legacy)

### Definitions

**Core Definition:** `Hashtable` is a legacy, synchronized implementation of the `Map` interface that predates the Java Collections Framework.

**Technical Definition:** `Hashtable<K,V>` extends `Dictionary<K,V>` and implements `Map<K,V>`, `Cloneable`, and `Serializable`. "As of the Java 2 platform v1.2, this class was retrofitted to implement the `Map` interface, making it a member of the Java Collections Framework". "Unlike the new collection implementations, `Hashtable` is synchronized. If a thread-safe implementation is not needed, it is recommended to use `HashMap` in place of `Hashtable`". It does **not** permit `null` keys or values.

**Beginner-Friendly Explanation:** `Hashtable` is an older version of `HashMap` that is thread-safe. However, the synchronization is often unnecessary and slows things down. It's better to use `HashMap` and synchronize only when needed.

### Purposes

- **Legacy.** To provide a thread-safe map implementation (now largely replaced by `ConcurrentHashMap`)
- To maintain backward compatibility with pre-Collections code

### Syntax Rules and Structure

```java
public class Hashtable<K,V> extends Dictionary<K,V>
        implements Map<K,V>, Cloneable, Serializable {
    public Hashtable();
    public Hashtable(int initialCapacity);
    public Hashtable(int initialCapacity, float loadFactor);
    public Hashtable(Map<? extends K, ? extends V> t);
}
```

**Syntax Rules:**

- `Hashtable` is synchronized, so all methods are thread-safe.
- It does **not** permit `null` keys or values.
- It is largely obsolete; use `HashMap` or `ConcurrentHashMap` instead.

**Constraints and Limitations:**

- Synchronization overhead makes it slower than `HashMap`.
- It has many legacy methods not in the `Map` interface.

### Annotated Code Example

```java
// File: HashtableDemo.java
import java.util.*;

public class HashtableDemo {
    public static void main(String[] args) {
        Hashtable<String, Integer> table = new Hashtable<>();
        table.put("Apple", 10);
        table.put("Banana", 20);
        System.out.println("Hashtable: " + table);
        System.out.println("Size: " + table.size());

        // null key or value throws NullPointerException
        try {
            table.put(null, 30);
        } catch (NullPointerException e) {
            System.out.println("null key not permitted");
        }
    }
}
```

**Expected Output:**
```
Hashtable: {Apple=10, Banana=20}
Size: 2
null key not permitted
```

**Step-by-Step Explanation:**

1. `Hashtable` is instantiated like `HashMap`.
2. `put()` is synchronized, so it is thread-safe but slower.
3. Adding a `null` key throws `NullPointerException`.

### Real-World Cases

- **Legacy Code:** Maintaining old codebases that use `Hashtable`.
- **Rare Thread-Safety Needs:** When a synchronized map is required and `ConcurrentHashMap` is not suitable.

---

## 6. EnumMap

### Definitions

**Core Definition:** `EnumMap` is a specialized `Map` implementation for use with enum keys, represented internally as an array.

**Technical Definition:** `EnumMap<K extends Enum<K>, V>` extends `AbstractMap<K,V>` and implements `Serializable`. "`EnumMap`, which is internally implemented as an array, is a high-performance Map implementation for use with enum keys. This implementation combines the richness and safety of the Map interface with a speed approaching that of an array. If you want to map an enum to a value, you should always use an `EnumMap` in preference to an array". It does **not** permit `null` keys (but permits `null` values).

**Beginner-Friendly Explanation:** `EnumMap` is a highly efficient map designed specifically for enum keys. It uses an array behind the scenes, making it extremely fast and compact.

### Purposes

- To provide a highly efficient map implementation for enum keys
- To serve as a typesafe alternative to array-based mappings
- To enable fast bulk operations on enum-keyed maps

### Syntax Rules and Structure

```java
public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K,V>
        implements Serializable, Cloneable {
    public EnumMap(Class<K> keyType);
    public EnumMap(EnumMap<K, ? extends V> m);
    public EnumMap(Map<K, ? extends V> m);
}
```

**Syntax Rules:**

- The key type must be a single enum type.
- `EnumMap` does **not** permit `null` keys; `null` values are permitted.
- It maintains the natural order of the enum constants.

**Constraints and Limitations:**

- Not synchronized; use `Collections.synchronizedMap()` for concurrent access.

### Annotated Code Example

```java
// File: EnumMapDemo.java
import java.util.*;

enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }

public class EnumMapDemo {
    public static void main(String[] args) {
        EnumMap<Day, String> map = new EnumMap<>(Day.class);

        map.put(Day.MONDAY, "Work");
        map.put(Day.SATURDAY, "Relax");
        map.put(Day.SUNDAY, "Rest");

        System.out.println("Map: " + map);
        System.out.println("Monday: " + map.get(Day.MONDAY));
        System.out.println("Size: " + map.size());
    }
}
```

**Expected Output:**
```
Map: {MONDAY=Work, SATURDAY=Relax, SUNDAY=Rest}
Monday: Work
Size: 3
```

**Step-by-Step Explanation:**

1. `EnumMap` is created with `Day.class` as the key type.
2. Entries are added for specific enum constants.
3. Iteration follows the natural order of the enum constants.

### Real-World Cases

- **Permissions:** Modeling user permissions as an enum-keyed map.
- **Configuration:** Mapping configuration options (enums) to values.
- **State Machines:** Mapping states (enums) to handlers.

---

## 7. WeakHashMap

### Definitions

**Core Definition:** `WeakHashMap` is a `Map` implementation that stores only weak references to its keys, allowing entries to be garbage-collected when their keys are no longer referenced.

**Technical Definition:** `WeakHashMap<K,V>` extends `AbstractMap<K,V>` and implements `Map<K,V>`. "`WeakHashMap` is an implementation of the Map interface that stores only weak references to its keys. Storing only weak references allows a key-value pair to be garbage-collected when its key is no longer referenced outside of the `WeakHashMap`. This class provides the easiest way to harness the power of weak references. It is useful for implementing 'registry-like' data structures, where the utility of an entry vanishes when its key is no longer reachable by any thread". It permits `null` keys and `null` values.

**Beginner-Friendly Explanation:** `WeakHashMap` is a map that does not prevent its keys from being garbage-collected. If nothing else is using a key, the entry is removed automatically. It's useful for caches where you don't want the cache to keep objects alive.

### Purposes

- To implement registry-like data structures where entries should vanish when keys are no longer referenced
- To provide a map that does not prevent key garbage collection
- To enable memory-sensitive caching

### Syntax Rules and Structure

```java
public class WeakHashMap<K,V> extends AbstractMap<K,V>
        implements Map<K,V> {
    public WeakHashMap();
    public WeakHashMap(int initialCapacity);
    public WeakHashMap(int initialCapacity, float loadFactor);
    public WeakHashMap(Map<? extends K, ? extends V> m);
}
```

**Syntax Rules:**

- Entries are automatically removed when their keys are no longer strongly referenced.
- It permits `null` keys and `null` values.
- Not synchronized.

**Constraints and Limitations:**

- The behaviour depends on the garbage collector, so entries may be removed at unpredictable times.
- `size()` may return a smaller value than expected due to asynchronous removal.

### Annotated Code Example

```java
// File: WeakHashMapDemo.java
import java.util.*;

public class WeakHashMapDemo {
    public static void main(String[] args) throws InterruptedException {
        Map<Object, String> map = new WeakHashMap<>();
        Object key = new Object();
        map.put(key, "Value");

        System.out.println("Before GC: " + map);
        key = null; // Remove strong reference
        System.gc();
        Thread.sleep(100);
        System.out.println("After GC: " + map);
    }
}
```

**Expected Output:**
```
Before GC: {java.lang.Object@...=Value}
After GC: {}
```

**Step-by-Step Explanation:**

1. A `WeakHashMap` is created with a strong reference to a key.
2. After `key = null`, the key becomes weakly reachable.
3. `System.gc()` suggests garbage collection.
4. The entry is removed automatically.

### Real-World Cases

- **Caching:** Caching metadata about objects without preventing their collection.
- **Registry:** Maintaining a registry of objects that should not be kept alive by the registry itself.

---

## 8. IdentityHashMap

### Definitions

**Core Definition:** `IdentityHashMap` is an identity-based `Map` implementation that uses reference equality (`==`) instead of `equals()` for key comparison.

**Technical Definition:** `IdentityHashMap<K,V>` extends `AbstractMap<K,V>` and implements `Map<K,V>`, `Serializable`, and `Cloneable`. "This class is useful for topology-preserving object graph transformations, such as serialization or deep-copying. To perform such transformations, you need to maintain an identity-based 'node table' that keeps track of which objects have already been seen. Identity-based maps are also used to maintain object-to-meta-information mappings in dynamic debuggers and similar systems. Finally, identity-based maps are useful in thwarting 'spoof attacks' that are a result of intentionally perverse `equals` methods because `IdentityHashMap` never invokes the `equals` method on its keys. An added benefit of this implementation is that it is fast". It permits `null` keys and `null` values.

**Beginner-Friendly Explanation:** `IdentityHashMap` compares keys by reference (same object) rather than by value (equal contents). It's useful when you need to distinguish between two objects that happen to be equal.

### Purposes

- To provide a map that uses reference equality for keys
- To support topology-preserving object graph transformations
- To maintain identity-based mappings for debugging and meta-information

### Syntax Rules and Structure

```java
public class IdentityHashMap<K,V> extends AbstractMap<K,V>
        implements Map<K,V>, Serializable, Cloneable {
    public IdentityHashMap();
    public IdentityHashMap(int expectedMaxSize);
    public IdentityHashMap(Map<? extends K, ? extends V> m);
}
```

**Syntax Rules:**

- Uses `==` (reference equality) instead of `equals()` for key comparison.
- Permits `null` keys and `null` values.
- Not synchronized.

**Constraints and Limitations:**

- Violates the general `Map` contract (which expects `equals()`).
- Not suitable for general-purpose use.

### Annotated Code Example

```java
// File: IdentityHashMapDemo.java
import java.util.*;

public class IdentityHashMapDemo {
    public static void main(String[] args) {
        Map<String, String> map = new IdentityHashMap<>();

        String a = new String("key");
        String b = new String("key");

        map.put(a, "Value A");
        map.put(b, "Value B");

        System.out.println("Size: " + map.size()); // 2
        System.out.println("Get a: " + map.get(a)); // Value A
        System.out.println("Get b: " + map.get(b)); // Value B
    }
}
```

**Expected Output:**
```
Size: 2
Get a: Value A
Get b: Value B
```

**Step-by-Step Explanation:**

1. Two distinct `String` objects with the same content are created.
2. `a` and `b` are different objects (different references).
3. `IdentityHashMap` treats them as different keys.

### Real-World Cases

- **Serialization:** Tracking which objects have been serialized.
- **Deep Copy:** Tracking which objects have been copied.
- **Debugging:** Maintaining object-to-metadata mappings.

---

## 9. ConcurrentHashMap

### Definitions

**Core Definition:** `ConcurrentHashMap` is a thread-safe `Map` implementation designed for high concurrency, allowing concurrent reads and writes without locking the entire table.

**Technical Definition:** `ConcurrentHashMap<K,V>` extends `AbstractMap<K,V>` and implements `ConcurrentMap<K,V>`, `Serializable`. "A hash table supporting full concurrency of retrievals and adjustable expected concurrency for updates. This class obeys the same functional specification as `Hashtable`, and includes versions of methods corresponding to each method of `Hashtable`. However, even though all operations are thread-safe, retrieval operations do not entail locking, and there is not any support for locking the entire table in a way that prevents all access". "Like `Hashtable` but unlike `HashMap`, this class does NOT allow `null` to be used as a key or value".

**Beginner-Friendly Explanation:** `ConcurrentHashMap` is a thread-safe map that allows multiple threads to read and write simultaneously without blocking each other. It's the go-to choice for concurrent map operations.

### Purposes

- To provide a high-performance, thread-safe map
- To support concurrent reads and writes without global locking
- To serve as a drop-in replacement for `Hashtable`

### Syntax Rules and Structure

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
        implements ConcurrentMap<K,V>, Serializable {
    public ConcurrentHashMap();
    public ConcurrentHashMap(int initialCapacity);
    public ConcurrentHashMap(int initialCapacity, float loadFactor);
    public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel);
    public ConcurrentHashMap(Map<? extends K, ? extends V> m);
}
```

**Syntax Rules:**

- Does **not** permit `null` keys or values.
- Retrieval operations do not block.
- Iterators are weakly consistent and never throw `ConcurrentModificationException`.

**Constraints and Limitations:**

- Bulk operations are not atomic.
- `size()` may be inaccurate during concurrent modification.

### Annotated Code Example

```java
// File: ConcurrentHashMapDemo.java
import java.util.concurrent.*;

public class ConcurrentHashMapDemo {
    public static void main(String[] args) throws InterruptedException {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

        // Multiple threads can safely update the map
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) map.merge("count", 1, Integer::sum);
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) map.merge("count", 1, Integer::sum);
        });

        t1.start(); t2.start();
        t1.join(); t2.join();

        System.out.println("Count: " + map.get("count")); // 2000
    }
}
```

**Expected Output:**
```
Count: 2000
```

**Step-by-Step Explanation:**

1. Two threads update the same key concurrently.
2. `merge()` is atomic, so no updates are lost.
3. The final count is 2000.

### Real-World Cases

- **Caching:** Thread-safe caches shared across threads.
- **Counters:** Concurrent counting of events or requests.
- **Session Management:** Storing user sessions in a web application.

---

## 10. ConcurrentSkipListMap

### Definitions

**Core Definition:** `ConcurrentSkipListMap` is a scalable concurrent `NavigableMap` implementation based on a skip list, providing sorted key order with thread-safe operations.

**Technical Definition:** `ConcurrentSkipListMap<K,V>` extends `AbstractMap<K,V>` and implements `ConcurrentNavigableMap<K,V>`, `Cloneable`, `Serializable`. "A scalable concurrent `ConcurrentNavigableMap` implementation. The map is sorted according to the natural ordering of its keys, or by a `Comparator` provided at map creation time, depending on which constructor is used. This class implements a concurrent variant of SkipLists providing expected average `log(n)` time cost for the `containsKey`, `get`, `put`, and `remove` operations and their variants. Insertion, removal, update, and access operations safely execute concurrently by multiple threads". It does **not** permit `null` keys or values.

**Beginner-Friendly Explanation:** `ConcurrentSkipListMap` is a thread-safe map that keeps its keys sorted. It's designed for concurrent access by multiple threads without requiring external synchronization.

### Purposes

- To provide a thread-safe, sorted map implementation
- To enable concurrent sorted map operations
- To support scalable, concurrent navigation

### Syntax Rules and Structure

```java
public class ConcurrentSkipListMap<K,V> extends AbstractMap<K,V>
        implements ConcurrentNavigableMap<K,V>, Cloneable, Serializable {
    public ConcurrentSkipListMap();
    public ConcurrentSkipListMap(Comparator<? super K> comparator);
    public ConcurrentSkipListMap(Map<? extends K, ? extends V> m);
    public ConcurrentSkipListMap(SortedMap<K, ? extends V> m);
}
```

**Syntax Rules:**

- Keys are sorted by natural ordering or a `Comparator`.
- Does **not** permit `null` keys or values.
- Iterators are weakly consistent.

**Constraints and Limitations:**

- Bulk operations are not guaranteed to be atomic.
- `size()` requires traversal.

### Annotated Code Example

```java
// File: ConcurrentSkipListMapDemo.java
import java.util.concurrent.*;

public class ConcurrentSkipListMapDemo {
    public static void main(String[] args) {
        ConcurrentSkipListMap<Integer, String> map = new ConcurrentSkipListMap<>();

        map.put(5, "Five");
        map.put(2, "Two");
        map.put(8, "Eight");
        map.put(1, "One");

        System.out.println("Map (sorted): " + map);
        System.out.println("First key: " + map.firstKey());
        System.out.println("Last key: " + map.lastKey());
    }
}
```

**Expected Output:**
```
Map (sorted): {1=One, 2=Two, 5=Five, 8=Eight}
First key: 1
Last key: 8
```

**Step-by-Step Explanation:**

1. Entries are inserted in arbitrary order.
2. The map stores them sorted by key: 1, 2, 5, 8.
3. `firstKey()` returns 1, `lastKey()` returns 8.

### Real-World Cases

- **Concurrent Sorted Data:** Maintaining a sorted map in a multi-threaded environment.
- **Priority Queues:** Implementing concurrent priority-based processing.
- **Leaderboards:** Concurrent sorted high scores.

---

## 11. Map Operations

### Definitions

**Core Definition:** Map operations are the methods provided by the `Map` interface and its implementations for manipulating and querying key-value mappings.

**Technical Definition:** The `Map` interface provides methods for basic operations (`put`, `get`, `remove`, `containsKey`, `containsValue`, `size`, `isEmpty`, `clear`), collection views (`keySet`, `values`, `entrySet`), and default methods introduced in Java 8 (`getOrDefault`, `putIfAbsent`, `computeIfAbsent`, `computeIfPresent`, `compute`, `merge`, `replace`, `replaceAll`, `forEach`).

**Beginner-Friendly Explanation:** Map operations are the actions you can perform on a map — adding entries, looking up values, removing entries, and iterating over keys, values, or entries.

### put() / get()

**Definition:** `put()` associates a value with a key; `get()` returns the value for a key.

**Syntax:**
```java
V put(K key, V value);
V get(Object key);
```

**Example:**
```java
map.put("Alice", 30);
Integer age = map.get("Alice"); // 30
```

### remove()

**Definition:** Removes the mapping for a key.

**Syntax:**
```java
V remove(Object key);
```

**Example:**
```java
map.remove("Alice");
```

### containsKey() / containsValue()

**Definition:** Checks if the map contains a key or value.

**Syntax:**
```java
boolean containsKey(Object key);
boolean containsValue(Object value);
```

**Example:**
```java
boolean hasAlice = map.containsKey("Alice");
```

### size() / isEmpty() / clear()

**Definition:** Returns the number of entries, checks if the map is empty, or removes all entries.

**Syntax:**
```java
int size();
boolean isEmpty();
void clear();
```

**Example:**
```java
int count = map.size();
boolean empty = map.isEmpty();
map.clear();
```

### keySet() / values() / entrySet()

**Definition:** Returns views of the map's keys, values, or key-value entries.

**Syntax:**
```java
Set<K> keySet();
Collection<V> values();
Set<Map.Entry<K, V>> entrySet();
```

**Example:**
```java
for (String key : map.keySet()) { ... }
for (Integer value : map.values()) { ... }
for (Map.Entry<String, Integer> entry : map.entrySet()) { ... }
```

### forEach()

**Definition:** Performs the given action for each entry.

**Syntax:**
```java
default void forEach(BiConsumer<? super K, ? super V> action);
```

**Example:**
```java
map.forEach((k, v) -> System.out.println(k + " = " + v));
```

### getOrDefault()

**Definition:** Returns the value for a key, or a default value if the key is absent.

**Syntax:**
```java
default V getOrDefault(Object key, V defaultValue);
```

**Example:**
```java
int age = map.getOrDefault("Unknown", 0);
```

### putIfAbsent()

**Definition:** Puts the value only if the key is not already associated with a value.

**Syntax:**
```java
default V putIfAbsent(K key, V value);
```

**Example:**
```java
map.putIfAbsent("Alice", 30);
```

### computeIfAbsent()

**Definition:** If the key is not already associated with a value, computes its value using the given mapping function and enters it into the map.

**Syntax:**
```java
default V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction);
```

**Example:**
```java
map.computeIfAbsent("key", k -> new ArrayList<>()).add("item");
```

### computeIfPresent()

**Definition:** If the value for the key is present, computes a new value using the given function.

**Syntax:**
```java
default V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction);
```

**Example:**
```java
map.computeIfPresent("count", (k, v) -> v + 1);
```

### compute()

**Definition:** Computes a new value for the key using the given function.

**Syntax:**
```java
default V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction);
```

**Example:**
```java
map.compute("count", (k, v) -> (v == null) ? 1 : v + 1);
```

### merge()

**Definition:** Merges the value with the existing value for the key.

**Syntax:**
```java
default V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction);
```

**Example:**
```java
map.merge("count", 1, Integer::sum);
```

### replace() / replaceAll()

**Definition:** Replaces values for keys.

**Syntax:**
```java
default V replace(K key, V value);
default boolean replace(K key, V oldValue, V newValue);
default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function);
```

**Example:**
```java
map.replace("Alice", 31);
map.replaceAll((k, v) -> v + 1);
```

---

## 12. Map Implementations Comparison

| Feature | HashMap | LinkedHashMap | TreeMap | Hashtable | EnumMap | WeakHashMap | IdentityHashMap | ConcurrentHashMap | ConcurrentSkipListMap |
|---------|---------|---------------|---------|-----------|---------|-------------|-----------------|-------------------|----------------------|
| **Thread-Safe** | No | No | No | Yes | No | No | No | Yes | Yes |
| **Ordering** | None | Insertion or access order | Sorted by key | None | Enum declaration order | None | None | None | Sorted by key |
| **Null Keys** | Yes (one) | Yes (one) | No | No | No | Yes (one) | Yes (one) | No | No |
| **Null Values** | Yes | Yes | Yes | No | Yes | Yes | Yes | No | No |
| **Basic Operations** | O(1) | O(1) | O(log n) | O(1) | O(1) | O(1) | O(1) | O(1) | O(log n) |
| **Iteration** | O(capacity + size) | O(size) | O(size) | O(capacity + size) | O(size) | O(size) | O(capacity + size) | O(capacity + size) | O(size) |
| **Since** | 1.2 | 1.4 | 1.2 | 1.0 | 1.5 | 1.2 | 1.4 | 1.5 | 1.6 |
| **Best For** | Default choice | Insertion order | Sorted order | Legacy | Enum keys | Weak keys | Identity keys | Concurrency | Concurrent sorted |

**Key Takeaways:**

- `HashMap` is usually the best default choice for general-purpose maps.
- `LinkedHashMap` is used when insertion order or access order matters.
- `TreeMap` is used when sorted key order is required.
- `ConcurrentHashMap` is the right choice for concurrent access (replaces `Hashtable`).
- `ConcurrentSkipListMap` is best for concurrent sorted map operations.
- `EnumMap` is the most efficient choice for enum keys.
- `WeakHashMap` is used when entries should be removed when keys are no longer referenced.
- `IdentityHashMap` is used when reference equality is required.
- `Hashtable` is a legacy class; prefer `ConcurrentHashMap` or `HashMap`.

---

## 13. Map Best Practices

### Definitions

**Core Definition:** Map best practices are guidelines for choosing and using map implementations effectively.

**Technical Definition:** Best practices include programming to the `Map` interface, choosing the right implementation for the use case, overriding `equals()` and `hashCode()` for custom keys, using the Java 8 default methods to simplify code, and being aware of thread-safety requirements.

**Beginner-Friendly Explanation:** Best practices help you write map code that is fast, safe, and easy to maintain.

### Key Best Practices

| Practice | Description |
|----------|-------------|
| **Program to interfaces** | Declare variables as `Map`, not `HashMap`. |
| **Choose the right implementation** | Use `HashMap` for general use; `LinkedHashMap` for insertion order; `TreeMap` for sorted order; `ConcurrentHashMap` for concurrency. |
| **Override `equals()` and `hashCode()`** | For custom keys, these methods are essential for correct lookup. |
| **Use Java 8 default methods** | Replace `containsKey`/`get` patterns with `getOrDefault`, `computeIfAbsent`, `merge`, etc.. |
| **Prefer `entrySet()` for iteration** | Iterating over `entrySet()` is more efficient than iterating over `keySet()` and calling `get()` for each key. |
| **Specify initial capacity** | For `HashMap`, set an initial capacity to avoid resizing. |
| **Use `Map.of()` for immutable maps** | Java 9+ provides `Map.of()` and `Map.ofEntries()` for unmodifiable maps. |
| **Avoid mutable keys** | Changing a key's `hashCode` or `equals` after insertion breaks the map. |
| **Use `ConcurrentHashMap` for concurrency** | It provides better performance than `Collections.synchronizedMap()` or `Hashtable`. |
| **Use `EnumMap` for enum keys** | It is far more efficient than `HashMap` for enums. |
| **Be aware of `null` handling** | Most concurrent maps prohibit `null`; check the implementation's contract. |

### Annotated Code Example

```java
// File: MapBestPractices.java
import java.util.*;

public class MapBestPractices {
    public static void main(String[] args) {
        // GOOD: Program to interface
        Map<String, Integer> map = new HashMap<>(64); // Initial capacity

        // GOOD: Use getOrDefault()
        int age = map.getOrDefault("Alice", 0);

        // GOOD: Use computeIfAbsent() for lazy initialization
        map.computeIfAbsent("Bob", k -> 25);
        System.out.println("Bob's age: " + map.get("Bob")); // 25

        // GOOD: Use merge() for counting
        map.merge("count", 1, Integer::sum);
        map.merge("count", 1, Integer::sum);
        System.out.println("Count: " + map.get("count")); // 2

        // GOOD: Use Map.of() for immutable maps (Java 9+)
        Map<String, Integer> immutable = Map.of("A", 1, "B", 2);
        // immutable.put("C", 3); // UnsupportedOperationException

        // GOOD: Prefer entrySet() for iteration
        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " = " + entry.getValue());
        }
    }
}
```

**Expected Output:**
```
Bob's age: 25
Count: 2
Bob = 25
count = 2
```

**Step-by-Step Explanation:**

1. `new HashMap<>(64)` specifies an initial capacity.
2. `getOrDefault()` avoids a `containsKey()` check.
3. `computeIfAbsent()` lazily initializes a value.
4. `merge()` simplifies counting.
5. `Map.of()` creates an immutable map.
6. `entrySet()` iteration is efficient.

### Real-World Cases

- **API Design:** Returning `Map` from methods instead of `HashMap`.
- **Performance Tuning:** Specifying initial capacity for large maps.
- **Immutable APIs:** Using `Map.of()` to protect internal state.
- **Concurrent Systems:** Using `ConcurrentHashMap` for thread-safe maps.

---

## References Links

- Map (Java SE 11 & JDK 11) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html
- Map Implementations — The Java Tutorials (Oracle) - https://docs.oracle.com/javase/tutorial/collections/implementations/map.html
- ConcurrentHashMap (Java SE 5) — Oracle API Documentation - https://people.csail.mit.edu/dfhuynh/research/javadoc/jdk1.5.0/java/util/concurrent/ConcurrentHashMap.html
- LinkedHashMap and Special-Purpose Maps — Pearson Education - https://ptgmedia.pearsoncmg.com/images/0321334205/samplechapter/Zakhour_ch11.pdf
- ConcurrentSkipListMap (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentSkipListMap.html
- RFR: 8004518 & 8010122: Default methods on Map — OpenJDK - https://mail.openjdk.org/pipermail/core-libs-dev/2013-April/015903.html
- Simplifiable 'Map' operations — JetBrains Inspectopedia - https://www.jetbrains.com/help/inspectopedia/Java8MapApi.html
- Java ArrayList, Hashmap instead of Vector, Hashtable — Stack Overflow - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/69945682/java-arraylist-hashmap-instead-of-vector-hashtable