# Java Sets — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A `Set` is a collection that contains no duplicate elements. More formally, sets contain no pair of elements `e1` and `e2` such that `e1.equals(e2)`, and at most one `null` element. As implied by its name, this interface models the mathematical set abstraction.

**Technical Definition:** `java.util.Set<E>` extends `Collection<E>` and places additional stipulations on the contracts of all constructors and on the contracts of the `add`, `equals`, and `hashCode` methods. All constructors must create a set that contains no duplicate elements. The `Set` interface is a member of the Java Collections Framework.

**Beginner-Friendly Explanation:** Think of a Java `Set` as a bag that automatically rejects duplicate items. Unlike a `List`, which is like a numbered shelf where you can place identical items in specific slots, a `Set` is more like a guest list at an exclusive event — each name can appear only once, and there are no assigned seat numbers.

### Key Characteristics

- **No Duplicates:** A `Set` cannot contain two equal elements, as determined by `equals()` (and `hashCode()` for hash-based sets) or `compareTo()`/`compare()` for sorted sets.
- **At Most One Null:** Most `Set` implementations allow a single `null` element, but `TreeSet` does not.
- **No Index-Based Access:** Unlike `List`, `Set` does not provide `get(int index)` or `set(int index, E)`.
- **Mathematical Set Abstraction:** Models the mathematical concept of a set, supporting operations like union, intersection, and difference.
- **Part of Java Collections Framework:** `Set` extends `Collection` and integrates with the broader `java.util` hierarchy.
- **Mutable Elements Warning:** Great care must be exercised if mutable objects are used as set elements. The behavior of a set is not specified if the value of an object is changed in a manner that affects `equals` comparisons while the object is an element in the set.

### Prerequisites

To effectively use `Set` in Java, you should understand:

- Basic Java syntax, classes, and objects.
- The `Collection` interface and generics (`<E>`).
- The difference between an interface and a concrete class.
- Basic iteration constructs (`for`, `while`).
- The concepts of `hashCode()` and `equals()` for custom objects.
- Familiarity with `java.util` package imports.

### Related Programming Areas

| Area | Explanation |
|------|-------------|
| **Collections Framework** | `Set` is one of the core interfaces in `java.util`, alongside `List`, `Map`, and `Queue`. |
| **Generics** | Type parameters (`Set<String>`) provide compile-time type safety. |
| **Concurrency** | `ConcurrentSkipListSet` and `CopyOnWriteArraySet` provide thread-safe set operations. |
| **Hashing** | `HashSet` and `LinkedHashSet` rely on `hashCode()` and `equals()` for uniqueness and performance. |
| **Sorting & Comparators** | `TreeSet` relies on `Comparable` or `Comparator` for ordering and uniqueness. |
| **Mathematical Set Theory** | Set operations (union, intersection, difference) map directly to mathematical set concepts. |

---

## Core Concepts / Key Features

---

## 1. Core Set Implementations

### Core Definitions

- **Core Definition:** Java provides several concrete classes that implement the `Set` interface, each with different ordering and performance characteristics.
- **Technical Definition:** The three general-purpose `Set` implementations are `HashSet`, `LinkedHashSet`, and `TreeSet`. Each is backed by a corresponding `Map` implementation (`HashMap`, `LinkedHashMap`, and `TreeMap` respectively).
- **Beginner-Friendly Explanation:** The `Set` interface is just a contract; you need to choose a concrete class. `HashSet` is the default fast option; `LinkedHashSet` keeps insertion order; `TreeSet` keeps elements sorted.

### Sub-feature 1.1: HashSet

#### Definitions

- **Core Definition:** `HashSet` is backed by a `HashMap` and offers O(1) constant-time performance for basic operations (`add`, `remove`, `contains`), with no ordering guarantees.
- **Technical Definition:** `public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, Serializable`. Internally, it wraps a `HashMap` instance that acts as a delegate. The `HashMap` stores elements as keys with a shared dummy value.
- **Beginner-Friendly Explanation:** `HashSet` is like a bucket with many compartments. It uses a hash code to quickly figure out which compartment an item belongs in, so adding, removing, or checking for an item is almost instant — but the items are scattered in no particular order.

#### Purposes

- To provide the fastest general-purpose set implementation with O(1) average-case performance.
- To store unique elements without any ordering requirements.
- To serve as the default `Set` implementation when order does not matter.

#### Syntax Structures and Rules

**General Syntax:**
```
Set<Type> set = new HashSet<>();
Set<Type> set = new HashSet<>(initialCapacity);
Set<Type> set = new HashSet<>(initialCapacity, loadFactor);
Set<Type> set = new HashSet<>(collection);
```

**Component Breakdown:**
- `new HashSet<>()` — creates an empty set with default initial capacity 16 and load factor 0.75.
- `initialCapacity` — the initial number of buckets; if the set grows beyond `capacity × loadFactor`, it is rehashed.
- `collection` — creates a set containing the unique elements of the given collection.

**Syntax Rules:**
- The type parameter must be a reference type.
- `add()` returns `true` if the element was not already present, `false` otherwise.
- `HashSet` allows one `null` element.

**Constraints and Limitations:**
- No ordering guarantees; iteration order may change over time.
- Not thread-safe; concurrent modification may throw `ConcurrentModificationException`.
- Iteration performance is linear in the sum of the number of entries and the number of buckets (capacity); choosing an initial capacity that's too high can waste both space and time.

#### Annotated Code Example

```java
import java.util.HashSet;
import java.util.Set;

public class HashSetDemo {
    public static void main(String[] args) {
        // Create a HashSet with default capacity
        Set<String> fruits = new HashSet<>();
        
        // Add elements (duplicates are silently ignored)
        boolean added1 = fruits.add("Apple");     // true (new)
        boolean added2 = fruits.add("Banana");    // true (new)
        boolean added3 = fruits.add("Apple");     // false (duplicate)
        boolean added4 = fruits.add("Cherry");    // true (new)
        
        System.out.println("Added Apple: " + added1);
        System.out.println("Added Banana: " + added2);
        System.out.println("Added Apple again: " + added3);
        System.out.println("Added Cherry: " + added4);
        System.out.println("Set size: " + fruits.size());
        System.out.println("Contains Banana: " + fruits.contains("Banana"));
        System.out.println("Contains Grape: " + fruits.contains("Grape"));
        
        // Iterate (order is not guaranteed)
        System.out.println("Iterating over set:");
        for (String fruit : fruits) {
            System.out.println("  " + fruit);
        }
    }
}
```

**Expected Output (order may vary):**
```
Added Apple: true
Added Banana: true
Added Apple again: false
Added Cherry: true
Set size: 3
Contains Banana: true
Contains Grape: false
Iterating over set:
  Apple
  Banana
  Cherry
```

**Why This Output:** The `HashSet` rejects the second "Apple" because `equals()` returns `true`, and `add()` returns `false`. The set size remains 3. `contains("Banana")` returns `true` because "Banana" was added. The iteration order is unspecified and may differ between runs.

#### Real-World Case

**Scenario:** Deduplicating a list of email addresses before sending a newsletter. A `HashSet` quickly eliminates duplicates without caring about order, ensuring each recipient receives only one email.

#### References

- HashSet (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html
- Set Implementations (Oracle Tutorial) — https://docs.oracle.com/javase/tutorial/collections/implementations/set.html

---

### Sub-feature 1.2: LinkedHashSet

#### Definitions

- **Core Definition:** `LinkedHashSet` extends `HashSet` with a doubly-linked list running through all elements, maintaining insertion order while keeping near-O(1) performance.
- **Technical Definition:** `public class LinkedHashSet<E> extends HashSet<E> implements Set<E>, Cloneable, Serializable`. The underlying `LinkedHashMap` maintains a linked list of entries in insertion order.
- **Beginner-Friendly Explanation:** `LinkedHashSet` is like `HashSet` but with a memory of the order in which you added items. When you iterate, you get them back in the same order you put them in.

#### Purposes

- To maintain insertion order while still benefiting from hash-based performance.
- To provide predictable iteration order without the overhead of sorting.
- To spare clients from the unspecified, generally chaotic ordering provided by `HashSet` without incurring the increased cost associated with `TreeSet`.

#### Syntax Structures and Rules

**General Syntax:**
```
Set<Type> set = new LinkedHashSet<>();
Set<Type> set = new LinkedHashSet<>(initialCapacity);
Set<Type> set = new LinkedHashSet<>(collection);
```

**Syntax Rules:**
- Insertion order is defined as the order in which elements were inserted into the set (least recently inserted to most recently).
- Re-inserting an existing element does not change its position.
- Allows one `null` element.

**Constraints and Limitations:**
- Slightly more memory than `HashSet` due to the linked list.
- Not thread-safe.
- Iteration time is not affected by capacity (unlike `HashSet`).

#### Annotated Code Example

```java
import java.util.LinkedHashSet;
import java.util.Set;

public class LinkedHashSetDemo {
    public static void main(String[] args) {
        Set<String> steps = new LinkedHashSet<>();
        
        // Add steps in a specific order
        steps.add("Preheat oven");
        steps.add("Mix ingredients");
        steps.add("Pour into pan");
        steps.add("Bake for 30 minutes");
        steps.add("Mix ingredients");  // duplicate: ignored
        
        System.out.println("Recipe steps (in insertion order):");
        for (String step : steps) {
            System.out.println("  " + step);
        }
        System.out.println("Size: " + steps.size());
    }
}
```

**Expected Output:**
```
Recipe steps (in insertion order):
  Preheat oven
  Mix ingredients
  Pour into pan
  Bake for 30 minutes
Size: 4
```

**Why This Output:** The `LinkedHashSet` maintains insertion order: "Preheat oven", "Mix ingredients", "Pour into pan", "Bake for 30 minutes". The duplicate "Mix ingredients" is ignored and does not change its original position. Size remains 4.

#### Real-World Case

**Scenario:** A recipe application displaying steps in the order the chef entered them. `LinkedHashSet` ensures steps are shown in the correct sequence while preventing accidental duplicate steps.

#### References

- LinkedHashSet (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashSet.html

---

### Sub-feature 1.3: TreeSet

#### Definitions

- **Core Definition:** `TreeSet` is backed by a `TreeMap` (Red-Black tree) and stores elements in sorted, ascending order (natural ordering or a custom `Comparator`).
- **Technical Definition:** `public class TreeSet<E> extends AbstractSet<E> implements NavigableSet<E>, Cloneable, Serializable`. It provides guaranteed O(log n) time for basic operations (`add`, `remove`, `contains`). It implements `NavigableSet`, which extends `SortedSet`, and enables range operations like `subSet()`, `headSet()`, and `tailSet()`.
- **Beginner-Friendly Explanation:** `TreeSet` keeps all its items sorted automatically. Whenever you add a new item, it finds the correct position to maintain order. This makes it slower than `HashSet` for adding and removing, but it allows you to easily get sorted views and ranges of elements.

#### Purposes

- To maintain elements in sorted order automatically.
- To enable range-based operations (`subSet`, `headSet`, `tailSet`).
- To provide navigable methods like `first()`, `last()`, `higher()`, `lower()`, `ceiling()`, and `floor()`.
- To support sorted iteration in both ascending and descending order.

#### Syntax Structures and Rules

**General Syntax:**
```
Set<Type> set = new TreeSet<>();                          // natural ordering
Set<Type> set = new TreeSet<>(Comparator);                // custom ordering
Set<Type> set = new TreeSet<>(SortedSet);                 // copy from SortedSet
NavigableSet<Type> nav = new TreeSet<>();
```

**Syntax Rules:**
- Elements must be mutually comparable (implement `Comparable`) or a `Comparator` must be provided.
- `add()` throws `NullPointerException` if the element is `null` and natural ordering is used, or if the comparator does not permit `null` elements.
- `add()` throws `ClassCastException` if the element cannot be compared to existing elements.

**Constraints and Limitations:**
- Does not allow `null` elements (throws `NullPointerException`).
- Slower than `HashSet` and `LinkedHashSet` for basic operations (O(log n) vs O(1)).
- Not thread-safe.

#### Annotated Code Example

```java
import java.util.NavigableSet;
import java.util.TreeSet;

public class TreeSetDemo {
    public static void main(String[] args) {
        NavigableSet<Integer> scores = new TreeSet<>();
        
        // Add elements in random order
        scores.add(85);
        scores.add(92);
        scores.add(78);
        scores.add(95);
        scores.add(88);
        
        System.out.println("Sorted set: " + scores);
        System.out.println("First (lowest): " + scores.first());
        System.out.println("Last (highest): " + scores.last());
        System.out.println("Higher than 88: " + scores.higher(88));
        System.out.println("Lower than 88: " + scores.lower(88));
        System.out.println("Ceiling of 90: " + scores.ceiling(90));
        System.out.println("Floor of 90: " + scores.floor(90));
        
        // Range operations
        System.out.println("Head set (< 90): " + scores.headSet(90));
        System.out.println("Tail set (>= 90): " + scores.tailSet(90));
        System.out.println("Sub set [80, 93): " + scores.subSet(80, 93));
        
        // Descending iteration
        System.out.println("Descending: " + scores.descendingSet());
    }
}
```

**Expected Output:**
```
Sorted set: [78, 85, 88, 92, 95]
First (lowest): 78
Last (highest): 95
Higher than 88: 92
Lower than 88: 85
Ceiling of 90: 92
Floor of 90: 88
Head set (< 90): [78, 85, 88]
Tail set (>= 90): [92, 95]
Sub set [80, 93): [85, 88, 92]
Descending: [95, 92, 88, 85, 78]
```

**Why This Output:** The `TreeSet` automatically sorts the elements: 78, 85, 88, 92, 95. `first()` returns 78, `last()` returns 95. `higher(88)` returns the smallest element strictly greater than 88 (92). `headSet(90)` returns elements less than 90. `subSet(80, 93)` returns elements in [80, 93).

#### Real-World Case

**Scenario:** A leaderboard where player scores are always displayed in sorted order. `TreeSet` ensures the leaderboard is automatically sorted, and `subSet()` can extract players within a certain score range.

#### References

- TreeSet (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html
- NavigableSet (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/NavigableSet.html

---

## 2. Key Mechanics

### Core Definitions

- **Core Definition:** Key mechanics govern how `Set` implementations enforce uniqueness, order elements, handle `null`, and choose between hashing and sorting.
- **Technical Definition:** These mechanics include uniqueness enforcement (`equals`/`hashCode` vs `compareTo`/`compare`), ordering and iteration behavior, `null` element support, and the trade-off between hashing and sorting.
- **Beginner-Friendly Explanation:** These are the "rules of the game" for sets — how they decide what's a duplicate, what order they keep things in, whether they allow `null`, and why they perform differently.

### Sub-feature 2.1: Uniqueness — `equals()`/`hashCode()` vs `compareTo()`/`compare()`

#### Definitions

- **Core Definition:** `Set` implementations enforce uniqueness either through `equals()` and `hashCode()` (hash-based sets) or through `compareTo()`/`compare()` (sorted sets).
- **Technical Definition:** For `HashSet` and `LinkedHashSet`, duplicate detection uses both `equals()` and `hashCode()`. For `TreeSet`, a sorted set performs all element comparisons using its `compareTo()` (or `compare()`) method; an element is considered a duplicate if the comparison returns 0.
- **Beginner-Friendly Explanation:** `HashSet` and `LinkedHashSet` ask "Are these two items equal?" using `equals()`. `TreeSet` asks "How do these two items compare?" using `compareTo()` — if they compare as equal (return 0), the second one is rejected.

#### Purposes

- To ensure no duplicate elements are stored in the set.
- To define the contract that custom objects must fulfill to be used in sets.
- To enable correct set operations (union, intersection, etc.).

#### Syntax Structures and Rules

**General Syntax:**
```
// For HashSet / LinkedHashSet: override equals() and hashCode()
@Override
public boolean equals(Object o) { ... }

@Override
public int hashCode() { ... }

// For TreeSet: implement Comparable or provide Comparator
@Override
public int compareTo(T other) { ... }
```

**Syntax Rules:**
- For `HashSet`/`LinkedHashSet`: if `e1.equals(e2)` is `true`, then `e1.hashCode() == e2.hashCode()` must be `true`.
- For `TreeSet`: if `compareTo()` returns 0, the elements are considered duplicates regardless of `equals()`.
- Inconsistent `equals()` and `compareTo()` can lead to surprising behavior in `TreeSet`.

**Constraints and Limitations:**
- Mutable objects used as set elements can break the set if their `hashCode()` or `compareTo()` depends on mutable fields.
- `TreeSet` does not use `equals()` at all; it relies entirely on `compareTo()` or `compare()`.

#### Annotated Code Example

```java
import java.util.HashSet;
import java.util.Set;
import java.util.TreeSet;

class Person implements Comparable<Person> {
    String name;
    int age;
    
    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
        Person p = (Person) o;
        return age == p.age && name.equals(p.name);
    }
    
    @Override
    public int hashCode() {
        return 31 * name.hashCode() + age;
    }
    
    @Override
    public int compareTo(Person other) {
        // Compare by age only (inconsistent with equals)
        return Integer.compare(this.age, other.age);
    }
    
    @Override
    public String toString() {
        return name + "(" + age + ")";
    }
}

public class UniquenessDemo {
    public static void main(String[] args) {
        Person alice25 = new Person("Alice", 25);
        Person bob30 = new Person("Bob", 30);
        Person alice25b = new Person("Alice", 25);  // equal to alice25
        Person charlie25 = new Person("Charlie", 25); // same age as alice25
        
        // HashSet: uses equals/hashCode
        Set<Person> hashSet = new HashSet<>();
        hashSet.add(alice25);
        hashSet.add(bob30);
        hashSet.add(alice25b);      // duplicate (equals)
        hashSet.add(charlie25);     // not duplicate (different name)
        System.out.println("HashSet size: " + hashSet.size());  // 3
        
        // TreeSet: uses compareTo
        Set<Person> treeSet = new TreeSet<>();
        treeSet.add(alice25);
        treeSet.add(bob30);
        treeSet.add(alice25b);      // duplicate (compareTo = 0)
        treeSet.add(charlie25);     // duplicate (compareTo = 0, same age)
        System.out.println("TreeSet size: " + treeSet.size());  // 2
        System.out.println("TreeSet: " + treeSet);
    }
}
```

**Expected Output:**
```
HashSet size: 3
TreeSet size: 2
TreeSet: [Alice(25), Bob(30)]
```

**Why This Output:** In `HashSet`, `alice25b` is a duplicate of `alice25` (equals), but `charlie25` is not (different name). So size is 3. In `TreeSet`, `compareTo` compares only by age, so `alice25`, `alice25b`, and `charlie25` all compare as equal (age 25). Only the first one is kept, resulting in size 2.

#### Real-World Case

**Scenario:** A university enrollment system. If students are considered unique by their student ID, the `compareTo()` method should use the ID, not the name. Using `HashSet` with `equals()`/`hashCode()` based on ID is also valid. Consistency between `equals()` and `compareTo()` is crucial to avoid subtle bugs.

#### References

- Set (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/Set.html
- Unique object identification of Java Set — https://stackoverflow.com/questions/16947875/unique-object-identification-of-java-set

---

### Sub-feature 2.2: Ordering & Iteration

#### Definitions

- **Core Definition:** `Set` implementations differ in their iteration order: `HashSet` is unordered, `LinkedHashSet` maintains insertion order, and `TreeSet` maintains sorted order.
- **Technical Definition:** `HashSet` makes no guarantees about iteration order; it can change over time. `LinkedHashSet` orders elements by insertion order (least recently inserted to most recently). `TreeSet` orders elements by their natural ordering or a `Comparator`.
- **Beginner-Friendly Explanation:** When you iterate over a `HashSet`, the order is unpredictable. `LinkedHashSet` gives you items back in the order you added them. `TreeSet` gives them back in sorted order.

#### Purposes

- To choose the appropriate `Set` implementation based on ordering needs.
- To ensure reproducible iteration order when required.
- To enable sorted traversal and range queries.

#### Syntax Structures and Rules

**General Syntax:**
```
// HashSet: no ordering
Set<String> hashSet = new HashSet<>();

// LinkedHashSet: insertion order
Set<String> linkedSet = new LinkedHashSet<>();

// TreeSet: sorted order
NavigableSet<String> treeSet = new TreeSet<>();
```

**Syntax Rules:**
- `HashSet` iteration order is unspecified and should not be relied upon.
- `LinkedHashSet` preserves insertion order; re-inserting an existing element does not change its position.
- `TreeSet` preserves sorted order; iteration is always in ascending order (or descending via `descendingSet()`).

**Constraints and Limitations:**
- If you need reproducible order without sorting, use `LinkedHashSet`.
- If you need sorted order, use `TreeSet`.
- `HashSet` is the fastest but provides no ordering.

#### Annotated Code Example

```java
import java.util.HashSet;
import java.util.LinkedHashSet;
import java.util.Set;
import java.util.TreeSet;

public class OrderingDemo {
    public static void main(String[] args) {
        // HashSet: unordered
        Set<String> hashSet = new HashSet<>();
        hashSet.add("Banana");
        hashSet.add("Apple");
        hashSet.add("Cherry");
        hashSet.add("Date");
        System.out.println("HashSet: " + hashSet);
        
        // LinkedHashSet: insertion order
        Set<String> linkedSet = new LinkedHashSet<>();
        linkedSet.add("Banana");
        linkedSet.add("Apple");
        linkedSet.add("Cherry");
        linkedSet.add("Date");
        System.out.println("LinkedHashSet: " + linkedSet);
        
        // TreeSet: sorted order
        Set<String> treeSet = new TreeSet<>();
        treeSet.add("Banana");
        treeSet.add("Apple");
        treeSet.add("Cherry");
        treeSet.add("Date");
        System.out.println("TreeSet: " + treeSet);
    }
}
```

**Expected Output:**
```
HashSet: [Apple, Cherry, Banana, Date]  (order may vary)
LinkedHashSet: [Banana, Apple, Cherry, Date]
TreeSet: [Apple, Banana, Cherry, Date]
```

**Why This Output:** `HashSet` produces an arbitrary order based on hash codes. `LinkedHashSet` preserves the insertion order: Banana, Apple, Cherry, Date. `TreeSet` sorts alphabetically: Apple, Banana, Cherry, Date.

#### Real-World Case

**Scenario:** A browser's autocomplete suggestions. `LinkedHashSet` could store recently typed queries in insertion order, while `TreeSet` could store them alphabetically for a sorted dropdown.

#### References

- Extending Collection with Set, SortedSet and NavigableSet (Dev.java) — https://dev.java/learn/api/collections-framework/sets/
- The Set Interface (Oracle Tutorial) — https://docs.oracle.com/javase/tutorial/collections/interfaces/set.html

---

### Sub-feature 2.3: Null Element Support

#### Definitions

- **Core Definition:** `HashSet` and `LinkedHashSet` allow one `null` element; `TreeSet` does not allow `null` elements.
- **Technical Definition:** `HashSet` and `LinkedHashSet` permit a single `null` element. `TreeSet` throws `NullPointerException` when `null` is added because it needs to compare elements using `compareTo()` or `compare()`.
- **Beginner-Friendly Explanation:** `HashSet` and `LinkedHashSet` can handle a `null` item without any problem. `TreeSet` cannot, because it needs to compare every item to maintain sorted order, and comparing `null` to anything causes an error.

#### Purposes

- To understand which `Set` implementations can store `null`.
- To avoid `NullPointerException` when using `TreeSet`.
- To decide which `Set` implementation to use based on `null` handling needs.

#### Syntax Structures and Rules

**General Syntax:**
```
// HashSet / LinkedHashSet: null allowed
Set<String> hashSet = new HashSet<>();
hashSet.add(null);  // OK

// TreeSet: null not allowed
Set<String> treeSet = new TreeSet<>();
treeSet.add(null);  // throws NullPointerException
```

**Syntax Rules:**
- `HashSet` and `LinkedHashSet` permit at most one `null` element.
- `TreeSet` throws `NullPointerException` if `null` is added and natural ordering is used, or if the comparator does not permit `null` elements.
- Even with a custom comparator, adding `null` to `TreeSet` is generally problematic.

**Constraints and Limitations:**
- `TreeSet` cannot store `null` because it relies on comparison to maintain order.
- If you need a sorted set that allows `null`, you would need a custom comparator that handles `null`, but this is rarely recommended.

#### Annotated Code Example

```java
import java.util.HashSet;
import java.util.LinkedHashSet;
import java.util.Set;
import java.util.TreeSet;

public class NullSupportDemo {
    public static void main(String[] args) {
        // HashSet: null allowed
        Set<String> hashSet = new HashSet<>();
        hashSet.add("A");
        hashSet.add(null);
        hashSet.add("B");
        System.out.println("HashSet with null: " + hashSet);
        
        // LinkedHashSet: null allowed
        Set<String> linkedSet = new LinkedHashSet<>();
        linkedSet.add("A");
        linkedSet.add(null);
        linkedSet.add("B");
        System.out.println("LinkedHashSet with null: " + linkedSet);
        
        // TreeSet: null NOT allowed
        Set<String> treeSet = new TreeSet<>();
        treeSet.add("A");
        treeSet.add("B");
        try {
            treeSet.add(null);  // throws NullPointerException
        } catch (NullPointerException e) {
            System.out.println("TreeSet cannot accept null: " + e.getClass().getSimpleName());
        }
        System.out.println("TreeSet: " + treeSet);
    }
}
```

**Expected Output:**
```
HashSet with null: [null, A, B]
LinkedHashSet with null: [A, null, B]
TreeSet cannot accept null: NullPointerException
TreeSet: [A, B]
```

**Why This Output:** `HashSet` and `LinkedHashSet` successfully store the `null` element. `TreeSet` throws `NullPointerException` when `null` is added because it cannot compare `null` with "A" or "B".

#### Real-World Case

**Scenario:** A configuration system where a `Set` stores optional settings. If `null` is a valid "unset" value, `HashSet` or `LinkedHashSet` is appropriate. If the settings must be sorted, use `TreeSet` but ensure `null` is never added.

#### References

- TreeSet (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html
- Why adding null in HashSet does not throw Exception — https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/

---

### Sub-feature 2.4: Hashing vs. Sorting

#### Definitions

- **Core Definition:** Hashing (used by `HashSet`/`LinkedHashSet`) uses an element's hash code to determine a bucket location, providing O(1) performance. Sorting (used by `TreeSet`) uses element comparison to build a tree, providing O(log n) performance but keeping elements sorted.
- **Technical Definition:** `HashSet` and `LinkedHashSet` use hash-based storage, where the `hashCode()` determines the bucket, and `equals()` resolves collisions. `TreeSet` uses a Red-Black tree, where `compareTo()` or `compare()` determines the position of each element.
- **Beginner-Friendly Explanation:** Hashing is like having a magic filing cabinet that tells you exactly which drawer to look in — it's very fast but doesn't keep anything in order. Sorting is like arranging books on a shelf by title — it takes a bit longer to put each book in the right place, but you can always find them in order.

#### Purposes

- To choose the right performance/ordering trade-off for your application.
- To understand why `HashSet` is faster but unordered, while `TreeSet` is slower but sorted.
- To make informed decisions about which `Set` implementation to use.

#### Syntax Structures and Rules

**General Syntax:**
```
// Hashing: O(1) average
Set<Type> hashSet = new HashSet<>();

// Sorting: O(log n)
Set<Type> treeSet = new TreeSet<>();
```

**Syntax Rules:**
- `HashSet` requires `hashCode()` and `equals()` to be properly implemented for custom objects.
- `TreeSet` requires `compareTo()` or a `Comparator` to be properly implemented.
- `HashSet` performance depends on a good hash function; poor hash functions can degrade to O(n).
- `TreeSet` performance is guaranteed O(log n) regardless of element values.

**Constraints and Limitations:**
- `HashSet` does not support range queries; `TreeSet` does.
- `TreeSet` requires elements to be mutually comparable.
- `HashSet` iteration order is unpredictable; `TreeSet` iteration is sorted.

#### Annotated Code Example

```java
import java.util.HashSet;
import java.util.Set;
import java.util.TreeSet;

public class HashingVsSortingDemo {
    public static void main(String[] args) {
        int n = 1_000_000;
        
        // Hashing: HashSet
        long startHash = System.nanoTime();
        Set<Integer> hashSet = new HashSet<>();
        for (int i = 0; i < n; i++) {
            hashSet.add(i);
        }
        long endHash = System.nanoTime();
        System.out.println("HashSet add " + n + " elements: " + (endHash - startHash) / 1_000_000 + " ms");
        
        // Sorting: TreeSet
        long startTree = System.nanoTime();
        Set<Integer> treeSet = new TreeSet<>();
        for (int i = 0; i < n; i++) {
            treeSet.add(i);
        }
        long endTree = System.nanoTime();
        System.out.println("TreeSet add " + n + " elements: " + (endTree - startTree) / 1_000_000 + " ms");
        
        // Contains check
        long startContainsHash = System.nanoTime();
        boolean hashContains = hashSet.contains(n - 1);
        long endContainsHash = System.nanoTime();
        System.out.println("HashSet contains: " + hashContains + " (" + (endContainsHash - startContainsHash) + " ns)");
        
        long startContainsTree = System.nanoTime();
        boolean treeContains = treeSet.contains(n - 1);
        long endContainsTree = System.nanoTime();
        System.out.println("TreeSet contains: " + treeContains + " (" + (endContainsTree - startContainsTree) + " ns)");
    }
}
```

**Expected Output (approximate):**
```
HashSet add 1000000 elements: 120 ms
TreeSet add 1000000 elements: 850 ms
HashSet contains: true (5000 ns)
TreeSet contains: true (12000 ns)
```

**Why This Output:** `HashSet` is much faster than `TreeSet` (constant-time versus log-time for most operations). Adding 1 million elements to `HashSet` is significantly faster than to `TreeSet` because hashing avoids the overhead of tree balancing. `contains()` is also faster in `HashSet` for the same reason.

#### Real-World Case

**Scenario:** A real-time analytics system that tracks unique visitor IDs. If only membership checks are needed, `HashSet` is ideal. If the system also needs to find all visitors with IDs in a certain range, `TreeSet` is necessary despite the performance cost.

#### References

- Set Implementations (Oracle Tutorial) — https://docs.oracle.com/javase/tutorial/collections/implementations/set.html
- Java Set 去重性能实测 — https://www.ctyun.cn

---

## 3. Set Operations (Union, Intersection, Difference)

### Core Definitions

- **Core Definition:** `Set` supports mathematical set operations: union (`addAll`), intersection (`retainAll`), and difference (`removeAll`).
- **Technical Definition:** `s1.addAll(s2)` transforms `s1` into the union of `s1` and `s2`. `s1.retainAll(s2)` transforms `s1` into the intersection of `s1` and `s2`. `s1.removeAll(s2)` transforms `s1` into the (asymmetric) set difference of `s1` and `s2`.
- **Beginner-Friendly Explanation:** Union means "combine both sets"; intersection means "keep only what's in both"; difference means "keep only what's in the first set but not the second."

### Sub-feature 3.1: Union — `addAll()`

#### Definitions

- **Core Definition:** The union of two sets contains all elements that appear in either set.
- **Technical Definition:** `boolean addAll(Collection<? extends E> c)` — adds all elements from `c` to the set, ignoring duplicates.
- **Beginner-Friendly Explanation:** Combines two sets into one, removing any duplicates.

#### Purposes

- To merge two sets into one.
- To combine data from multiple sources while eliminating duplicates.
- To implement the mathematical union operation.

#### Syntax Structures and Rules

**General Syntax:**
```
set1.addAll(set2);  // set1 becomes the union
```

**Syntax Rules:**
- Modifies `set1` in place.
- Returns `true` if `set1` changed as a result.
- Elements from `set2` that are already in `set1` are ignored.

**Constraints and Limitations:**
- If `set1` and `set2` are the same set, behavior is undefined.
- Not thread-safe.

#### Annotated Code Example

```java
import java.util.HashSet;
import java.util.Set;

public class UnionDemo {
    public static void main(String[] args) {
        Set<String> set1 = new HashSet<>();
        set1.add("A");
        set1.add("B");
        set1.add("C");
        
        Set<String> set2 = new HashSet<>();
        set2.add("C");
        set2.add("D");
        set2.add("E");
        
        System.out.println("Set1: " + set1);
        System.out.println("Set2: " + set2);
        
        // Union: set1 becomes A, B, C, D, E
        set1.addAll(set2);
        System.out.println("Union: " + set1);
    }
}
```

**Expected Output:**
```
Set1: [A, B, C]
Set2: [C, D, E]
Union: [A, B, C, D, E]
```

**Why This Output:** `addAll(set2)` adds D and E to `set1`. C is already present, so it is not added again. The result is the union of both sets.

#### Real-World Case

**Scenario:** Combining two mailing lists — one for newsletter subscribers and one for product updates — into a single deduplicated list.

---

### Sub-feature 3.2: Intersection — `retainAll()`

#### Definitions

- **Core Definition:** The intersection of two sets contains only elements that appear in both sets.
- **Technical Definition:** `boolean retainAll(Collection<?> c)` — retains only the elements in the set that are contained in `c`.
- **Beginner-Friendly Explanation:** Keeps only the items that both sets have in common.

#### Purposes

- To find common elements between two sets.
- To filter a set based on membership in another collection.
- To implement the mathematical intersection operation.

#### Syntax Structures and Rules

**General Syntax:**
```
set1.retainAll(set2);  // set1 becomes the intersection
```

**Syntax Rules:**
- Modifies `set1` in place.
- Returns `true` if `set1` changed as a result.
- Elements in `set1` that are not in `set2` are removed.

**Constraints and Limitations:**
- Uses `equals()` for comparison.
- Not thread-safe.

#### Annotated Code Example

```java
import java.util.HashSet;
import java.util.Set;

public class IntersectionDemo {
    public static void main(String[] args) {
        Set<String> set1 = new HashSet<>();
        set1.add("A");
        set1.add("B");
        set1.add("C");
        
        Set<String> set2 = new HashSet<>();
        set2.add("B");
        set2.add("C");
        set2.add("D");
        
        System.out.println("Set1: " + set1);
        System.out.println("Set2: " + set2);
        
        // Intersection: set1 becomes B, C
        set1.retainAll(set2);
        System.out.println("Intersection: " + set1);
    }
}
```

**Expected Output:**
```
Set1: [A, B, C]
Set2: [B, C, D]
Intersection: [B, C]
```

**Why This Output:** `retainAll(set2)` removes A from `set1` because A is not in `set2`. B and C remain because they are in both sets.

#### Real-World Case

**Scenario:** Finding users who have both purchased a product and subscribed to a newsletter. The intersection of the "purchasers" set and the "subscribers" set gives the target audience.

---

### Sub-feature 3.3: Difference — `removeAll()`

#### Definitions

- **Core Definition:** The difference of two sets contains elements that are in the first set but not in the second.
- **Technical Definition:** `boolean removeAll(Collection<?> c)` — removes all elements from the set that are contained in `c`.
- **Beginner-Friendly Explanation:** Removes from the first set everything that appears in the second set.

#### Purposes

- To subtract one set from another.
- To filter out unwanted elements.
- To implement the mathematical difference operation.

#### Syntax Structures and Rules

**General Syntax:**
```
set1.removeAll(set2);  // set1 becomes set1 - set2
```

**Syntax Rules:**
- Modifies `set1` in place.
- Returns `true` if `set1` changed as a result.
- Elements in `set1` that are also in `set2` are removed.

**Constraints and Limitations:**
- The operation is not symmetric: `set1.removeAll(set2)` is not the same as `set2.removeAll(set1)`.
- Not thread-safe.

#### Annotated Code Example

```java
import java.util.HashSet;
import java.util.Set;

public class DifferenceDemo {
    public static void main(String[] args) {
        Set<String> set1 = new HashSet<>();
        set1.add("A");
        set1.add("B");
        set1.add("C");
        set1.add("D");
        
        Set<String> set2 = new HashSet<>();
        set2.add("B");
        set2.add("D");
        set2.add("E");
        
        System.out.println("Set1: " + set1);
        System.out.println("Set2: " + set2);
        
        // Difference: set1 becomes A, C (elements in set1 not in set2)
        set1.removeAll(set2);
        System.out.println("Difference (set1 - set2): " + set1);
    }
}
```

**Expected Output:**
```
Set1: [A, B, C, D]
Set2: [B, D, E]
Difference (set1 - set2): [A, C]
```

**Why This Output:** `removeAll(set2)` removes B and D from `set1` because they are in `set2`. A and C remain because they are not in `set2`.

#### Real-World Case

**Scenario:** A user's "unread" set minus the "deleted" set gives the messages that are unread and not deleted.

#### References

- Set Operations in Java (Baeldung) — https://www.baeldung.com/java-set-operations

---

## 4. Concurrency & Thread Safety

### Core Definitions

- **Core Definition:** None of the standard `Set` implementations (`HashSet`, `LinkedHashSet`, `TreeSet`) are thread-safe. For concurrent environments, use `Collections.synchronizedSet()` or concurrent alternatives like `ConcurrentSkipListSet` or `CopyOnWriteArraySet`.
- **Technical Definition:** `Collections.synchronizedSet(Set<T>)` returns a synchronized (thread-safe) set backed by the specified set. `ConcurrentSkipListSet` is a concurrent `SortedSet` implementation with O(log n) operations. `CopyOnWriteArraySet` is a thread-safe set backed by a copy-on-write array.
- **Beginner-Friendly Explanation:** Regular sets are not safe to use from multiple threads at the same time. If you need a set that can handle multiple threads, you have several options: wrap it with `Collections.synchronizedSet()`, or use `ConcurrentSkipListSet` (sorted) or `CopyOnWriteArraySet` (for small sets with frequent reads).

### Sub-feature 4.1: Collections.synchronizedSet()

#### Definitions

- **Core Definition:** Returns a synchronized (thread-safe) set backed by the specified set.
- **Technical Definition:** `static <T> Set<T> synchronizedSet(Set<T> s)` — wraps a synchronized block around each method of the original set.
- **Beginner-Friendly Explanation:** Wraps a regular set so that every method call is synchronized, making it safe for multiple threads.

#### Purposes

- To make an existing set thread-safe with minimal changes.
- To provide a simple synchronization mechanism for low-concurrency scenarios.
- To ensure all changes are immediately visible to other threads.

#### Syntax Structures and Rules

**General Syntax:**
```
Set<Type> syncSet = Collections.synchronizedSet(new HashSet<>());
```

**Syntax Rules:**
- Must access the set only through the returned synchronized wrapper.
- Iteration must be manually synchronized on the wrapper object.
- Individual method calls are atomic, but compound operations are not.

**Constraints and Limitations:**
- **Iteration requires external synchronization:** `synchronized (syncSet) { for (Type e : syncSet) { ... } }`.
- No concurrency if multiple threads use the set; one thread blocks another.
- Performance is similar to the original set with synchronization overhead.

#### Annotated Code Example

```java
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;

public class SynchronizedSetDemo {
    public static void main(String[] args) throws InterruptedException {
        Set<Integer> syncSet = Collections.synchronizedSet(new HashSet<>());
        
        // Multiple threads adding to the set
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                syncSet.add(i);
            }
        });
        
        Thread t2 = new Thread(() -> {
            for (int i = 500; i < 1500; i++) {
                syncSet.add(i);
            }
        });
        
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        
        System.out.println("Set size: " + syncSet.size());
        
        // Safe iteration with external synchronization
        synchronized (syncSet) {
            System.out.println("Contains 999: " + syncSet.contains(999));
            System.out.println("Contains 1499: " + syncSet.contains(1499));
        }
    }
}
```

**Expected Output:**
```
Set size: 1500
Contains 999: true
Contains 1499: true
```

**Why This Output:** The synchronized wrapper ensures that concurrent `add()` calls are thread-safe. The set contains unique values from 0 to 1499, totaling 1500 elements. The synchronized block ensures safe iteration.

#### Real-World Case

**Scenario:** A low-concurrency logging system where multiple threads add log messages to a shared set of unique error codes.

#### References

- Collections.synchronizedSet (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedSet-java.util.Set-

---

### Sub-feature 4.2: ConcurrentSkipListSet

#### Definitions

- **Core Definition:** `ConcurrentSkipListSet` is a scalable concurrent `NavigableSet` implementation based on a skip list.
- **Technical Definition:** `public class ConcurrentSkipListSet<E> extends AbstractSet<E> implements NavigableSet<E>, Cloneable, Serializable`. Most basic operations are O(log n). It allows concurrent adding/removing and reading/iteration, where iteration may or may not reflect changes since the iterator was created.
- **Beginner-Friendly Explanation:** A thread-safe sorted set. Multiple threads can add, remove, and iterate simultaneously without blocking each other. It's ideal for high-concurrency scenarios where sorting is needed.

#### Purposes

- To provide a thread-safe sorted set for high-concurrency environments.
- To support concurrent iteration without blocking.
- To enable range operations in a concurrent setting.

#### Syntax Structures and Rules

**General Syntax:**
```
ConcurrentSkipListSet<Type> set = new ConcurrentSkipListSet<>();
ConcurrentSkipListSet<Type> set = new ConcurrentSkipListSet<>(Comparator);
```

**Syntax Rules:**
- Elements must be mutually comparable or a `Comparator` must be provided.
- Does not allow `null` elements.
- Iterators are weakly consistent: they may or may not reflect changes made after creation.
- Bulk operations are not atomic.

**Constraints and Limitations:**
- O(log n) performance, slower than `HashSet` (O(1)).
- Does not allow `null`.
- Bulk operations (e.g., `addAll`) are not atomic.

#### Annotated Code Example

```java
import java.util.concurrent.ConcurrentSkipListSet;

public class ConcurrentSkipListSetDemo {
    public static void main(String[] args) throws InterruptedException {
        ConcurrentSkipListSet<Integer> set = new ConcurrentSkipListSet<>();
        
        // Multiple threads adding concurrently
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i += 2) {
                set.add(i);
            }
        });
        
        Thread t2 = new Thread(() -> {
            for (int i = 1; i < 1000; i += 2) {
                set.add(i);
            }
        });
        
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        
        System.out.println("Size: " + set.size());
        System.out.println("First: " + set.first());
        System.out.println("Last: " + set.last());
        System.out.println("Subset [100, 105): " + set.subSet(100, 105));
    }
}
```

**Expected Output:**
```
Size: 1000
First: 0
Last: 999
Subset [100, 105): [100, 101, 102, 103, 104]
```

**Why This Output:** The two threads concurrently add even and odd numbers from 0 to 999. The set contains all 1000 unique numbers. `first()` returns 0, `last()` returns 999. `subSet(100, 105)` returns the elements in [100, 105).

#### Real-World Case

**Scenario:** A real-time bidding system where multiple threads add and remove bid amounts. The sorted set allows quick retrieval of the highest and lowest bids while handling concurrent modifications.

#### References

- ConcurrentSkipListSet (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentSkipListSet.html

---

### Sub-feature 4.3: CopyOnWriteArraySet

#### Definitions

- **Core Definition:** `CopyOnWriteArraySet` is a thread-safe set backed by a copy-on-write array. All mutative operations create a fresh copy of the underlying array.
- **Technical Definition:** `public class CopyOnWriteArraySet<E> extends AbstractSet<E> implements Set<E>, Serializable`. Internally uses a `CopyOnWriteArrayList`. Read operations are lock-free and never throw `ConcurrentModificationException`; write operations are synchronized and expensive because they copy the entire array.
- **Beginner-Friendly Explanation:** Every time you modify the set, it creates an entirely new copy. This makes reads very fast and safe, but writes are slow and memory-intensive. Best for small sets that are read often and changed rarely.

#### Purposes

- To provide thread-safe iteration without synchronization.
- To support scenarios with frequent reads and infrequent writes.
- To eliminate `ConcurrentModificationException` during iteration.

#### Syntax Structures and Rules

**General Syntax:**
```
Set<Type> set = new CopyOnWriteArraySet<>();
```

**Syntax Rules:**
- Iterators operate on a snapshot of the array at the time of creation.
- Iterator's `remove()` is not supported.
- `add`, `set`, `remove` are synchronized and copy the array.

**Constraints and Limitations:**
- **Not suitable for write-heavy workloads** — each write copies the entire array (O(n)).
- Memory usage can be high during writes (two arrays exist temporarily).
- Iterators do not reflect changes made after iterator creation.
- Best for small sets (e.g., listener registries).

#### Annotated Code Example

```java
import java.util.Iterator;
import java.util.Set;
import java.util.concurrent.CopyOnWriteArraySet;

public class CopyOnWriteArraySetDemo {
    public static void main(String[] args) {
        Set<String> listeners = new CopyOnWriteArraySet<>();
        listeners.add("Listener A");
        listeners.add("Listener B");
        listeners.add("Listener C");
        
        // Get an iterator (snapshot)
        Iterator<String> it = listeners.iterator();
        
        // Modify the set while iterating (safe: iterator uses snapshot)
        listeners.add("Listener D");
        listeners.remove("Listener B");
        
        // Iterate over the snapshot (does not include D, still includes B)
        System.out.println("Iterating snapshot:");
        while (it.hasNext()) {
            System.out.println("  " + it.next());
        }
        
        // Print current set
        System.out.println("Current set: " + listeners);
    }
}
```

**Expected Output:**
```
Iterating snapshot:
  Listener A
  Listener B
  Listener C
Current set: [Listener A, Listener C, Listener D]
```

**Why This Output:** The iterator was obtained before "Listener D" was added and "Listener B" was removed. It iterates over the snapshot `[Listener A, Listener B, Listener C]`. The current set contains `[Listener A, Listener C, Listener D]` because the iterator does not reflect post-creation modifications.

#### Real-World Case

**Scenario:** An event-handler registry in a GUI application. Listeners are added or removed rarely, but events are dispatched frequently. `CopyOnWriteArraySet` allows event dispatch to iterate over the listener set without synchronization while safely handling occasional listener registration changes.

#### References

- CopyOnWriteArraySet (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CopyOnWriteArraySet.html

---

## 5. Immutable Sets (Java 9+)

### Core Definitions

- **Core Definition:** `Set.of()` and `Set.copyOf()` create unmodifiable sets that cannot be changed after creation.
- **Technical Definition:** `Set.of(E... elements)` creates a truly immutable set (Java 9+). `Set.copyOf(Collection<? extends E> coll)` creates an unmodifiable copy of an existing collection (Java 10+). These sets throw `UnsupportedOperationException` on any mutator method, disallow `null` elements, and reject duplicate elements at creation time.
- **Beginner-Friendly Explanation:** `Set.of()` creates a read-only set that can never be changed. `Set.copyOf()` creates a read-only copy of an existing collection.

### Sub-feature 5.1: Set.of()

#### Definitions

- **Core Definition:** Creates an immutable set containing the specified elements.
- **Technical Definition:** `static <E> Set<E> of(E... elements)` — returns a set containing the given elements. Throws `IllegalArgumentException` on duplicate elements and `NullPointerException` on `null` elements.
- **Beginner-Friendly Explanation:** A quick way to create a small, unchangeable set.

#### Purposes

- To create small, fixed sets of constants.
- To enforce immutability in APIs.
- To provide a convenient factory method for sets.

#### Syntax Structures and Rules

**General Syntax:**
```
Set<Type> set = Set.of(e1, e2, e3);
Set<Type> empty = Set.of();
```

**Syntax Rules:**
- Disallows `null` elements.
- Rejects duplicate elements at creation time.
- Throws `UnsupportedOperationException` on any mutator method.
- Iteration order is unspecified and may change.

**Constraints and Limitations:**
- Cannot contain duplicates.
- Cannot contain `null`.
- Not suitable for large sets (overloaded variants for 0–10 elements).

#### Annotated Code Example

```java
import java.util.Set;

public class SetOfDemo {
    public static void main(String[] args) {
        // Immutable set of colors
        Set<String> colors = Set.of("Red", "Green", "Blue");
        System.out.println("Colors: " + colors);
        
        // Try to modify (throws UnsupportedOperationException)
        try {
            colors.add("Yellow");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify Set.of(): " + e.getClass().getSimpleName());
        }
        
        // Try to create with null (throws NullPointerException)
        try {
            Set.of("A", null, "B");
        } catch (NullPointerException e) {
            System.out.println("Cannot include null: " + e.getClass().getSimpleName());
        }
        
        // Try to create with duplicate (throws IllegalArgumentException)
        try {
            Set.of("A", "B", "A");
        } catch (IllegalArgumentException e) {
            System.out.println("Cannot include duplicates: " + e.getClass().getSimpleName());
        }
    }
}
```

**Expected Output:**
```
Colors: [Red, Green, Blue]
Cannot modify Set.of(): UnsupportedOperationException
Cannot include null: NullPointerException
Cannot include duplicates: IllegalArgumentException
```

**Why This Output:** `Set.of("Red", "Green", "Blue")` creates an immutable set. Calling `add()` throws `UnsupportedOperationException`. Creating a set with `null` throws `NullPointerException`. Creating a set with duplicates throws `IllegalArgumentException`.

#### Real-World Case

**Scenario:** Defining a set of valid configuration options that must never change at runtime.

---

### Sub-feature 5.2: Set.copyOf()

#### Definitions

- **Core Definition:** Creates an unmodifiable set containing the elements of the given collection.
- **Technical Definition:** `static <E> Set<E> copyOf(Collection<? extends E> coll)` — returns an unmodifiable set containing the elements of `coll`. If `coll` is already an unmodifiable set, it may be returned directly.
- **Beginner-Friendly Explanation:** Makes a read-only copy of an existing collection, removing duplicates.

#### Purposes

- To create a defensive, unmodifiable copy of a mutable collection.
- To pass a collection to an API without allowing modification.
- To deduplicate a collection into an unmodifiable set.

#### Syntax Structures and Rules

**General Syntax:**
```
Set<Type> immutable = Set.copyOf(existingCollection);
```

**Syntax Rules:**
- Disallows `null` elements.
- The returned set is unmodifiable.
- If the collection is already an unmodifiable set, it may be returned as-is.

**Constraints and Limitations:**
- The original collection's iteration order is not preserved.
- The copy is shallow (elements themselves are not copied).

#### Annotated Code Example

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Set;

public class SetCopyOfDemo {
    public static void main(String[] args) {
        List<String> mutableList = new ArrayList<>();
        mutableList.add("A");
        mutableList.add("B");
        mutableList.add("A");  // duplicate
        mutableList.add("C");
        
        System.out.println("Original list: " + mutableList);
        
        // Create unmodifiable set (duplicates removed)
        Set<String> immutableSet = Set.copyOf(mutableList);
        System.out.println("Immutable set: " + immutableSet);
        System.out.println("Size: " + immutableSet.size());
        
        // Try to modify
        try {
            immutableSet.add("D");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify: " + e.getClass().getSimpleName());
        }
    }
}
```

**Expected Output:**
```
Original list: [A, B, A, C]
Immutable set: [A, B, C]
Size: 3
Cannot modify: UnsupportedOperationException
```

**Why This Output:** `Set.copyOf(mutableList)` creates an unmodifiable set containing the unique elements A, B, C. The duplicate A is removed. Attempting to add "D" throws `UnsupportedOperationException`.

#### Real-World Case

**Scenario:** A method that returns a set of configuration keys. Returning `Set.copyOf(internalSet)` prevents callers from modifying the internal set.

#### References

- Set (Java Platform SE 8) — https://docs.oracle.com/javase/8/docs/api/java/util/Set.html
- Unmodifiable Sets (OpenJDK) — https://cr.openjdk.org/~smarks/

---

## Consolidated Reference Links

| Name | Link |
|------|------|
| Set (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/Set.html |
| HashSet (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html |
| LinkedHashSet (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashSet.html |
| TreeSet (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html |
| NavigableSet (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/NavigableSet.html |
| SortedSet (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/SortedSet.html |
| ConcurrentSkipListSet (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentSkipListSet.html |
| CopyOnWriteArraySet (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CopyOnWriteArraySet.html |
| Collections.synchronizedSet (Java Platform SE 8) | https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedSet-java.util.Set- |
| Set Implementations (Oracle Tutorial) | https://docs.oracle.com/javase/tutorial/collections/implementations/set.html |
| The Set Interface (Oracle Tutorial) | https://docs.oracle.com/javase/tutorial/collections/interfaces/set.html |
| Extending Collection with Set, SortedSet and NavigableSet (Dev.java) | https://dev.java/learn/api/collections-framework/sets/ |
| Set vs List in Java (Baeldung) | https://www.baeldung.com/java-set-vs-list |
| Set Operations in Java (Baeldung) | https://www.baeldung.com/java-set-operations |
| Different types of thread-safe Sets in Java (Stack Overflow) | https://stackoverflow.com/questions/6720396/different-types-of-thread-safe-sets-in-java |
| Unique object identification of Java Set (Stack Overflow) | https://stackoverflow.com/questions/16947875/unique-object-identification-of-java-set |
| Java Language Specification (Chapter 14) | https://docs.oracle.com/javase/specs/jls/se7/html/jls-14.html |