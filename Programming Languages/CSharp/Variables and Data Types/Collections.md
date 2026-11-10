# C# Collections — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A collection in C# is a managed container that groups multiple elements into a single logical unit. Unlike arrays, whose size is fixed at creation, most collections can grow, shrink, and reorganize their contents dynamically. Collections are the primary tool for storing, retrieving, and manipulating groups of related data in .NET applications.

**Technical Definition**
The .NET type system organizes collections across several namespaces: `System.Collections` (non-generic, legacy), `System.Collections.Generic` (generic, type-safe, preferred), `System.Collections.Concurrent` (thread-safe), `System.Collections.Immutable` (persistent, immutable), and `System.Collections.ObjectModel` (read-only wrappers). Generic collections accept a type parameter at construction, provide compile-time type safety, and avoid boxing/unboxing of value types. All standard generic collections implement a hierarchy of interfaces rooted at `IEnumerable<T>`, with `ICollection<T>`, `IList<T>`, `IDictionary<TKey,TValue>`, and `ISet<T>` adding progressively richer contracts.

**Beginner-Friendly Explanation**
A collection is a **smart box** for holding many items. Unlike a plain array (a fixed row of lockers), a collection can expand when you add items and shrink when you remove them. Different collections are optimized for different jobs: a `List<T>` is a numbered list, a `Dictionary` is a lookup table, a `Queue` is a line, a `Stack` is a pile, and a `HashSet` is a bag that refuses duplicates.

---

### Key Characteristics

- **Dynamic sizing** — Most collections grow and shrink automatically; the internal storage is resized as needed.
- **Generic by default** — Modern .NET code uses generic collections (`List<T>`, `Dictionary<TKey,TValue>`, etc.) which are type-safe and avoid boxing.
- **Rich interface hierarchy** — Collections implement contracts such as `IEnumerable<T>`, `ICollection<T>`, `IList<T>`, `IDictionary<TKey,TValue>`, and `ISet<T>`, enabling polymorphic consumption.
- **Multiple performance profiles** — Different collections trade off insertion speed, lookup speed, ordering, and memory. Choosing the wrong type can silently degrade performance.
- **Thread-safety and immutability options** — `System.Collections.Concurrent` and `System.Collections.Immutable` provide specialized semantics for concurrent and immutable scenarios.

---

### Prerequisites

- Understanding of C# generics (`<T>`, `<TKey,TValue>`).
- Familiarity with arrays and their limitations.
- Basic knowledge of interfaces and polymorphism.
- Awareness of time complexity (Big-O) for reasoning about performance.
- Understanding of value types vs. reference types and boxing.

---

### Related Programming Areas

- **LINQ** — All generic collections implement `IEnumerable<T>`, making them queryable with LINQ.
- **Algorithms and data structures** — Collections are production-ready implementations of classic structures (lists, maps, sets, queues).
- **Concurrency** — Concurrent collections and immutable collections are essential for thread-safe programming.
- **API design** — Exposing `IReadOnlyCollection<T>` or `IReadOnlyList<T>` communicates intent and prevents mutation by callers.
- **Serialization** — Collections are the standard containers for JSON arrays, XML lists, and database result sets.

---

### Core Concepts / Features

1. Collection types (overarching concept)
2. `List<T>`
3. `Dictionary<TKey, TValue>`
4. `HashSet<T>`
5. `Queue<T>`
6. `Stack<T>`
7. `LinkedList<T>`
8. `SortedList<TKey, TValue>`
9. `SortedDictionary<TKey, TValue>`
10. `SortedSet<T>`
11. `ReadOnlyCollection<T>`
12. `ImmutableArray<T>`
13. `ImmutableList<T>`
14. `ImmutableDictionary<TKey, TValue>`
15. `ConcurrentDictionary<TKey, TValue>`
16. Collection interfaces (`IEnumerable<T>`, `ICollection<T>`, `IList<T>`, `IDictionary<TKey,TValue>`, `ISet<T>`, `IReadOnlyCollection<T>`, `IReadOnlyList<T>`, `IReadOnlyDictionary<TKey,TValue>`)
17. Collection initialization
18. Collection expressions (C# 12)
19. Collection best practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Collection Types (Overarching Concept)

### Definitions

**Core Definition**
Collection types are the families of classes in .NET that represent groups of objects, each designed for a specific access pattern and performance profile.

**Technical Definition**
The .NET class library provides three broad families of collections: **generic** collections (`System.Collections.Generic`), **concurrent** collections (`System.Collections.Concurrent`), and **immutable** collections (`System.Collections.Immutable`). Generic collections are type-safe and preferred for most scenarios. Concurrent collections provide lock-free or fine-grained synchronization for multi-threaded access. Immutable collections are persistent data structures that never change after creation; operations return new collections, making them inherently thread-safe.

**Beginner-Friendly Explanation**
Think of collection types as different **storage containers** in a warehouse. A `List<T>` is a flexible shelf; a `Dictionary` is an index card system; a `HashSet` is a bag that rejects duplicates; a `Queue` is a conveyor belt; a `Stack` is a pile of plates. Choosing the right container makes the job easy; choosing the wrong one makes it slow or impossible.

---

### Purposes

- To store and manipulate groups of data with dynamic sizing.
- To provide type-safe, generic containers that avoid boxing.
- To offer specialized access patterns (indexed, keyed, FIFO, LIFO, set-based).
- To support concurrent and immutable scenarios safely.
- To serve as the backing store for LINQ queries and serialization.

---

### Syntax Rules and Structure

```
// Generic collection declaration
List<T>   list   =   new   List<T>()   ;
Dictionary<TKey, TValue>   map   =   new   Dictionary<TKey, TValue>()   ;
HashSet<T>   set   =   new   HashSet<T>()   ;
Queue<T>   queue   =   new   Queue<T>()   ;
Stack<T>   stack   =   new   Stack<T>()   ;
```

**Component breakdown**

- `List<T>` — Indexed, resizable list.
- `Dictionary<TKey, TValue>` — Key-value map with O(1) lookup.
- `HashSet<T>` — Unordered set with O(1) membership.
- `Queue<T>` — FIFO collection.
- `Stack<T>` — LIFO collection.

#### Syntax Rules

- Generic collections require a type argument at construction.
- All standard generic collections implement `IEnumerable<T>` and can be used with `foreach`.
- Most collections have a `Count` property and mutation methods (`Add`, `Remove`, `Clear`).
- Collection initializers and collection expressions provide concise syntax.

#### Constraints and Limitations

- Non-generic collections (`ArrayList`, `Hashtable`) box value types and are legacy.
- Not all collections are thread-safe; use concurrent or immutable variants for multi-threaded access.
- Some collections (e.g., `SortedList`) have O(n) insertion in the worst case.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Choosing a Collection

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var list = new List<int> { 10, 20, 30 };
        var map = new Dictionary<string, int> { ["Alice"] = 30, ["Bob"] = 25 };
        var set = new HashSet<int> { 1, 2, 2, 3 };   // duplicate 2 ignored
        var queue = new Queue<string>();
        queue.Enqueue("first");
        queue.Enqueue("second");
        var stack = new Stack<string>();
        stack.Push("bottom");
        stack.Push("top");

        Console.WriteLine(list[1]);            // 20
        Console.WriteLine(map["Alice"]);       // 30
        Console.WriteLine(set.Count);          // 3
        Console.WriteLine(queue.Dequeue());     // first
        Console.WriteLine(stack.Pop());         // top
    }
}
```

**Expected Output**

```
20
30
3
first
top
```

**Why this output occurs**

- `List<T>` provides O(1) indexed access.
- `Dictionary<TKey,TValue>` provides O(1) key lookup.
- `HashSet<T>` discards duplicates.
- `Queue<T>` dequeues in FIFO order.
- `Stack<T>` pops in LIFO order.

---

#### Example 2: Iterating with `IEnumerable<T>`

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        IEnumerable<string> items = new List<string> { "a", "b", "c" };
        foreach (string item in items)
        {
            Console.WriteLine(item);
        }
    }
}
```

**Expected Output**

```
a
b
c
```

**Why this output occurs**

- `List<T>` implements `IEnumerable<T>`, so it can be iterated with `foreach`.

---

### Real-World Cases

- **Inventory systems** — `List<Product>` for catalogues, `Dictionary<int, Product>` for lookup by ID.
- **Task scheduling** — `Queue<Job>` for FIFO processing, `Stack<Command>` for undo/redo.
- **Data deduplication** — `HashSet<string>` for unique tags or identifiers.
- **Thread-safe caches** — `ConcurrentDictionary<string, object>` for shared lookups.

---

### References

- Collections and Data Structures - .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/
- Commonly Used Collection Types - .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/commonly-used-collection-types
- System.Collections.Generic Namespace - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic

---

## 2. `List<T>`

### Definitions

**Core Definition**
`List<T>` is a resizable, indexed, strongly typed collection of objects of type `T`.

**Technical Definition**
`System.Collections.Generic.List<T>` is backed by a dynamically resized array. It provides O(1) indexed access, amortized O(1) insertion at the end (O(n) when the internal array must be resized), O(n) insertion/removal in the middle, and O(n) search with `IndexOf`. It implements `IList<T>`, `ICollection<T>`, and `IEnumerable<T>`. The `Capacity` property can be set to pre-allocate storage and reduce resizing.

**Beginner-Friendly Explanation**
`List<T>` is a **flexible numbered list**. You can add items to the end, insert them in the middle, remove them, and access any item by its index. It is the go-to collection for ordered data that changes size.

---

### Purposes

- To store an ordered, indexable sequence of elements.
- To provide fast access by position.
- To support dynamic resizing without manual array management.
- To serve as a general-purpose replacement for fixed-size arrays.
- To support LINQ queries through `IEnumerable<T>`.

---

### Syntax Rules and Structure

```
List<T>   list   =   new   List<T>()   ;
List<T>   list2   =   new   List<T>   {   item1, item2, ...   }   ;
```

**Component breakdown**

- `List<T>` — The generic list type.
- `new List<T>()` — Creates an empty list with default capacity.
- `{ ... }` — Collection initializer.

#### Syntax Rules

- Elements are accessed by zero-based index (`list[0]`).
- `Add` appends to the end; `Insert` adds at a specific index.
- `Remove` removes the first matching element; `RemoveAt` removes by index.
- `Count` is the number of elements; `Capacity` is the allocated storage size.

#### Constraints and Limitations

- Insertion in the middle is O(n) because subsequent elements must shift.
- Not thread-safe; use locks or concurrent collections for shared access.
- Accessing an invalid index throws `ArgumentOutOfRangeException`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Operations

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var list = new List<string> { "apple", "banana" };
        list.Add("cherry");
        list.Insert(1, "blueberry");

        Console.WriteLine(list[0]);   // apple
        Console.WriteLine(list[1]);   // blueberry
        Console.WriteLine(list.Count); // 4

        list.Remove("banana");
        Console.WriteLine(list.Count); // 3
    }
}
```

**Expected Output**

```
apple
blueberry
4
3
```

**Why this output occurs**

- `Add` appends, `Insert` places an element at a specific index.
- `Remove` deletes the first matching element.

---

#### Example 2: Pre-allocating Capacity

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var list = new List<int>(capacity: 1000);
        Console.WriteLine(list.Capacity); // 1000
        Console.WriteLine(list.Count);    // 0
    }
}
```

**Expected Output**

```
1000
0
```

**Why this output occurs**

- `Capacity` is set at construction, but `Count` remains `0` until elements are added.

---

### Real-World Cases

- **UI lists** — Items in a dropdown or data grid.
- **Batch processing** — Accumulating results from a loop.
- **Game inventories** — A player’s items in order.

---

### References

- List<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1

---

## 3. `Dictionary<TKey, TValue>`

### Definitions

**Core Definition**
`Dictionary<TKey, TValue>` is a hash-based collection that stores key-value pairs and provides fast lookup by unique key.

**Technical Definition**
`System.Collections.Generic.Dictionary<TKey, TValue>` implements a hash table. Keys must be unique and non-null; values may be any type. Lookup, insertion, and removal are O(1) on average (O(n) in the worst case if many keys collide). It implements `IDictionary<TKey, TValue>`, `ICollection<KeyValuePair<TKey, TValue>>`, and `IEnumerable<KeyValuePair<TKey, TValue>>`.

**Beginner-Friendly Explanation**
A `Dictionary` is a **lookup table**. You give it a key (like a name), and it instantly returns the associated value (like a phone number). Each key can appear only once.

---

### Purposes

- To store data as key-value pairs for fast lookup.
- To map identifiers to objects (e.g., ID → entity).
- To implement caches and indexes.
- To support configuration and settings storage.
- To provide O(1) average-time access by key.

---

### Syntax Rules and Structure

```
Dictionary<TKey, TValue>   map   =   new   Dictionary<TKey, TValue>()   ;
map[key]   =   value   ;
TValue   v   =   map[key]   ;
```

**Component breakdown**

- `TKey` — The key type (must be non-null).
- `TValue` — The value type.
- `map[key]` — Indexer for get/set.
- `Add(key, value)` — Adds a new pair; throws if the key exists.
- `TryGetValue(key, out value)` — Safe lookup.

#### Syntax Rules

- Keys must be unique; adding a duplicate key with `Add` throws `ArgumentException`.
- The indexer `map[key]` returns the value or throws `KeyNotFoundException` if the key is missing.
- `TryGetValue` returns `bool` and avoids exceptions.
- `ContainsKey` checks for key presence.

#### Constraints and Limitations

- Not thread-safe; use `ConcurrentDictionary` for concurrent access.
- Iteration order is not guaranteed (implementation-dependent).
- Keys must have consistent `Equals`/`GetHashCode` implementations.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Dictionary Operations

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var ages = new Dictionary<string, int>();
        ages["Alice"] = 30;
        ages["Bob"] = 25;

        Console.WriteLine(ages["Alice"]);   // 30
        Console.WriteLine(ages.ContainsKey("Bob")); // True

        if (ages.TryGetValue("Carol", out int carolAge))
            Console.WriteLine(carolAge);
        else
            Console.WriteLine("Carol not found"); // this branch
    }
}
```

**Expected Output**

```
30
True
Carol not found
```

**Why this output occurs**

- The indexer retrieves the value for `"Alice"`.
- `ContainsKey` checks for `"Bob"`.
- `TryGetValue` returns `false` for a missing key without throwing.

---

#### Example 2: Iterating Key-Value Pairs

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var map = new Dictionary<string, int> { ["a"] = 1, ["b"] = 2 };
        foreach (KeyValuePair<string, int> pair in map)
        {
            Console.WriteLine($"{pair.Key} = {pair.Value}");
        }
    }
}
```

**Expected Output (order may vary)**

```
a = 1
b = 2
```

**Why this output occurs**

- `Dictionary` implements `IEnumerable<KeyValuePair<TKey, TValue>>`, enabling `foreach`.

---

### Real-World Cases

- **User directories** — Username → User object.
- **Caches** — URL → Cached response.
- **Configuration** — Setting name → Setting value.
- **Counting** — Word → Frequency.

---

### References

- Dictionary<TKey,TValue> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2

---

## 4. `HashSet<T>`

### Definitions

**Core Definition**
`HashSet<T>` is an unordered collection that contains no duplicate elements and provides high-performance set operations.

**Technical Definition**
`System.Collections.Generic.HashSet<T>` is backed by a hash table. It provides O(1) average-time `Add`, `Remove`, and `Contains` operations. It implements `ISet<T>`, `ICollection<T>`, and `IEnumerable<T>`. Set operations such as union, intersection, and difference are available through methods like `UnionWith`, `IntersectWith`, and `ExceptWith`.

**Beginner-Friendly Explanation**
A `HashSet<T>` is a **bag that refuses duplicates**. If you try to add the same item twice, the second add is silently ignored. It is excellent for checking membership quickly.

---

### Purposes

- To store a collection of unique elements.
- To perform fast membership tests.
- To implement mathematical set operations.
- To deduplicate data.
- To track seen items efficiently.

---

### Syntax Rules and Structure

```
HashSet<T>   set   =   new   HashSet<T>()   ;
set.Add(item)   ;
bool   exists   =   set.Contains(item)   ;
```

**Component breakdown**

- `HashSet<T>` — The set type.
- `Add` — Returns `true` if the element was added; `false` if it already existed.
- `Contains` — O(1) membership test.

#### Syntax Rules

- Elements must have consistent `Equals`/`GetHashCode`.
- Order is not guaranteed.
- `Add` returns `bool`, unlike `List<T>.Add` which returns `void`.

#### Constraints and Limitations

- Not thread-safe.
- No indexed access.
- Iteration order is implementation-dependent.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Deduplication

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var set = new HashSet<int>();
        Console.WriteLine(set.Add(1));   // True
        Console.WriteLine(set.Add(2));   // True
        Console.WriteLine(set.Add(1));   // False (already present)

        Console.WriteLine(set.Count);    // 2
    }
}
```

**Expected Output**

```
True
True
False
2
```

**Why this output occurs**

- `Add` returns `false` when the element is already in the set.

---

#### Example 2: Set Operations

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var a = new HashSet<int> { 1, 2, 3 };
        var b = new HashSet<int> { 3, 4, 5 };

        a.IntersectWith(b);   // a becomes { 3 }
        Console.WriteLine(string.Join(", ", a)); // 3
    }
}
```

**Expected Output**

```
3
```

**Why this output occurs**

- `IntersectWith` modifies the set to contain only elements present in both sets.

---

### Real-World Cases

- **Tag systems** — Unique tags on a post.
- **Visited nodes** — Graph traversal.
- **Duplicate detection** — Finding repeated values in a stream.

---

### References

- HashSet<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.hashset-1

---

## 5. `Queue<T>`

### Definitions

**Core Definition**
`Queue<T>` is a first-in, first-out (FIFO) collection where elements are dequeued in the order they were enqueued.

**Technical Definition**
`System.Collections.Generic.Queue<T>` is a circular buffer. `Enqueue` adds to the rear in amortized O(1); `Dequeue` removes from the front in O(1). `Peek` returns the front element without removing it. It implements `IEnumerable<T>` and `ICollection`.

**Beginner-Friendly Explanation**
A `Queue<T>` is a **line at a checkout counter**. The first person in line is the first served. You add to the back and remove from the front.

---

### Purposes

- To process items in arrival order.
- To implement breadth-first search.
- To buffer work items for asynchronous processing.
- To model real-world queues (print jobs, messages).

---

### Syntax Rules and Structure

```
Queue<T>   q   =   new   Queue<T>()   ;
q.Enqueue(item)   ;
T   front   =   q.Dequeue()   ;
T   peek   =   q.Peek()   ;
```

**Component breakdown**

- `Enqueue` — Adds to the rear.
- `Dequeue` — Removes and returns the front.
- `Peek` — Returns the front without removing.

#### Syntax Rules

- `Dequeue` on an empty queue throws `InvalidOperationException`.
- `TryDequeue` (since .NET Core 2.0) provides a safe alternative.
- `Count` returns the number of elements.

#### Constraints and Limitations

- No random access.
- Not thread-safe; use `ConcurrentQueue<T>` for concurrent scenarios.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: FIFO Behaviour

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var q = new Queue<string>();
        q.Enqueue("first");
        q.Enqueue("second");
        q.Enqueue("third");

        Console.WriteLine(q.Dequeue()); // first
        Console.WriteLine(q.Dequeue()); // second
    }
}
```

**Expected Output**

```
first
second
```

**Why this output occurs**

- `Dequeue` removes elements in the order they were enqueued.

---

### Real-World Cases

- **Print spoolers** — Jobs processed in submission order.
- **BFS algorithms** — Nodes visited level by level.
- **Message queues** — Messages consumed in order.

---

### References

- Queue<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.queue-1

---

## 6. `Stack<T>`

### Definitions

**Core Definition**
`Stack<T>` is a last-in, first-out (LIFO) collection where the most recently added element is removed first.

**Technical Definition**
`System.Collections.Generic.Stack<T>` is backed by a dynamically sized array. `Push` adds to the top in amortized O(1); `Pop` removes and returns the top in O(1). `Peek` returns the top without removing it. It implements `IEnumerable<T>`.

**Beginner-Friendly Explanation**
A `Stack<T>` is a **pile of plates**. You put a plate on top, and you take the top plate off first. The last plate placed is the first one removed.

---

### Purposes

- To implement undo/redo functionality.
- To evaluate expressions (e.g., postfix notation).
- To traverse trees or graphs in depth-first order.
- To manage nested scopes or states.

---

### Syntax Rules and Structure

```
Stack<T>   s   =   new   Stack<T>()   ;
s.Push(item)   ;
T   top   =   s.Pop()   ;
T   peek   =   s.Peek()   ;
```

**Component breakdown**

- `Push` — Adds to the top.
- `Pop` — Removes and returns the top.
- `Peek` — Returns the top without removing.

#### Syntax Rules

- `Pop` on an empty stack throws `InvalidOperationException`.
- `TryPop` (since .NET Core 2.0) provides a safe alternative.

#### Constraints and Limitations

- No random access.
- Not thread-safe; use `ConcurrentStack<T>` for concurrent scenarios.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: LIFO Behaviour

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var s = new Stack<int>();
        s.Push(1);
        s.Push(2);
        s.Push(3);

        Console.WriteLine(s.Pop()); // 3
        Console.WriteLine(s.Pop()); // 2
    }
}
```

**Expected Output**

```
3
2
```

**Why this output occurs**

- `Pop` removes the most recently pushed element first.

---

### Real-World Cases

- **Undo/redo** — Commands pushed onto a stack.
- **Expression evaluation** — Operands and operators on a stack.
- **Depth-first search** — Nodes pushed and popped.

---

### References

- Stack<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.stack-1

---

## 7. `LinkedList<T>`

### Definitions

**Core Definition**
`LinkedList<T>` is a doubly linked list that provides O(1) insertion and removal at known nodes.

**Technical Definition**
`System.Collections.Generic.LinkedList<T>` stores elements in `LinkedListNode<T>` objects linked by `Next` and `Previous` references. Insertion and removal at a known node are O(1). Searching for a value is O(n). It implements `ICollection<T>` and `IEnumerable<T>`.

**Beginner-Friendly Explanation**
A `LinkedList<T>` is a **chain of nodes**. Each node knows its neighbours, so inserting or removing a node is fast if you already have a reference to it. But finding a specific value requires walking the chain.

---

### Purposes

- To support frequent insertion and removal in the middle of a sequence.
- To implement LRU caches and adjacency lists.
- To avoid the shifting cost of array-based lists.
- To provide stable references to nodes across modifications.

---

### Syntax Rules and Structure

```
LinkedList<T>   list   =   new   LinkedList<T>()   ;
LinkedListNode<T>   node   =   list.AddFirst(item)   ;
list.AddAfter(node,   newItem)   ;
list.Remove(node)   ;
```

**Component breakdown**

- `AddFirst` / `AddLast` — O(1) insertion at ends.
- `AddAfter` / `AddBefore` — O(1) insertion relative to a node.
- `Remove(node)` — O(1) removal of a known node.

#### Syntax Rules

- Nodes expose `Value`, `Next`, and `Previous`.
- Searching by value is O(n) via `Find`.
- Does not implement `IList<T>`; no indexed access.

#### Constraints and Limitations

- Higher memory overhead per element (node objects).
- Poor cache locality compared to arrays.
- Not thread-safe.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Inserting and Removing Nodes

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var list = new LinkedList<int>();
        var node = list.AddFirst(10);
        list.AddAfter(node, 20);
        list.AddLast(30);

        Console.WriteLine(string.Join(", ", list)); // 10, 20, 30

        list.Remove(node);
        Console.WriteLine(string.Join(", ", list)); // 20, 30
    }
}
```

**Expected Output**

```
10, 20, 30
20, 30
```

**Why this output occurs**

- `AddFirst`, `AddAfter`, and `AddLast` build the list.
- `Remove(node)` removes the node in O(1).

---

### Real-World Cases

- **LRU caches** — Move accessed nodes to the front.
- **Adjacency lists** — Graph representations with frequent edge updates.
- **Music playlists** — Insert and remove tracks efficiently.

---

### References

- LinkedList<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.linkedlist-1

---

## 8. `SortedList<TKey, TValue>`

### Definitions

**Core Definition**
`SortedList<TKey, TValue>` is a collection of key-value pairs sorted by key, with both key and index access.

**Technical Definition**
`System.Collections.Generic.SortedList<TKey, TValue>` is backed by a sorted array of keys and a parallel array of values. Insertion is O(n) for unsorted data and O(log n) when appending at the end; lookup by key is O(log n) via binary search; lookup by index is O(1). It implements `IDictionary<TKey, TValue>` and `IReadOnlyDictionary<TKey, TValue>`.

**Beginner-Friendly Explanation**
A `SortedList` is like a **phone book** sorted by name. You can look up a name quickly, and you can also access entries by their position in the sorted order.

---

### Purposes

- To maintain key-value pairs in sorted key order.
- To support both key-based and index-based access.
- To iterate in sorted order.
- To use less memory than `SortedDictionary` in some scenarios.

---

### Syntax Rules and Structure

```
SortedList<TKey, TValue>   list   =   new   SortedList<TKey, TValue>()   ;
list.Add(key,   value)   ;
TValue   v   =   list[key]   ;
KeyValuePair<TKey, TValue>   pair   =   list.ElementAt(index)   ;
```

**Component breakdown**

- `Add` — Inserts in sorted order (O(n) for unsorted data).
- `list[key]` — O(log n) binary search.
- `ElementAt(index)` — O(1) access by sorted position.

#### Syntax Rules

- Keys must be unique.
- Keys are sorted according to `IComparer<TKey>`.
- Supports both key and index access.

#### Constraints and Limitations

- Insertion is O(n) in the worst case.
- Not thread-safe.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Sorted Order and Index Access

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var list = new SortedList<string, int>();
        list.Add("banana", 2);
        list.Add("apple", 1);
        list.Add("cherry", 3);

        Console.WriteLine(list.Keys[0]);   // apple
        Console.WriteLine(list["banana"]); // 2
    }
}
```

**Expected Output**

```
apple
2
```

**Why this output occurs**

- Keys are sorted alphabetically.
- Index `0` corresponds to `"apple"`.

---

### Real-World Cases

- **Leaderboards** — Sorted player scores with index access.
- **Configuration** — Sorted settings with key lookup.

---

### References

- SortedList<TKey,TValue> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.sortedlist-2

---

## 9. `SortedDictionary<TKey, TValue>`

### Definitions

**Core Definition**
`SortedDictionary<TKey, TValue>` is a binary-search-tree-based dictionary that maintains keys in sorted order.

**Technical Definition**
`System.Collections.Generic.SortedDictionary<TKey, TValue>` uses a red-black tree. Insertion, removal, and lookup are O(log n). It implements `IDictionary<TKey, TValue>` and provides sorted iteration by key.

**Beginner-Friendly Explanation**
A `SortedDictionary` is a **balanced tree of key-value pairs**. It keeps everything in sorted order and provides fast lookup, but it does not support index-based access.

---

### Purposes

- To maintain a sorted dictionary with O(log n) operations.
- To iterate keys in sorted order.
- To avoid the O(n) insertion cost of `SortedList`.

---

### Syntax Rules and Structure

```
SortedDictionary<TKey, TValue>   dict   =   new   SortedDictionary<TKey, TValue>()   ;
dict.Add(key,   value)   ;
TValue   v   =   dict[key]   ;
```

**Component breakdown**

- `Add` — O(log n) insertion.
- `dict[key]` — O(log n) lookup.

#### Syntax Rules

- Keys are sorted by `IComparer<TKey>`.
- No index-based access.
- Keys must be unique.

#### Constraints and Limitations

- O(log n) operations (slower than `Dictionary` for lookup).
- Not thread-safe.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Sorted Iteration

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var dict = new SortedDictionary<int, string>();
        dict[3] = "three";
        dict[1] = "one";
        dict[2] = "two";

        foreach (var pair in dict)
        {
            Console.WriteLine($"{pair.Key}: {pair.Value}");
        }
    }
}
```

**Expected Output**

```
1: one
2: two
3: three
```

**Why this output occurs**

- `SortedDictionary` iterates in ascending key order.

---

### Real-World Cases

- **Range queries** — Efficiently finding keys within a range.
- **Ordered logs** — Timestamped entries maintained in order.

---

### References

- SortedDictionary<TKey,TValue> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.sorteddictionary-2

---

## 10. `SortedSet<T>`

### Definitions

**Core Definition**
`SortedSet<T>` is a set that maintains its elements in sorted order.

**Technical Definition**
`System.Collections.Generic.SortedSet<T>` is backed by a red-black tree. `Add`, `Remove`, and `Contains` are O(log n). It implements `ISet<T>` and provides sorted iteration. It supports set operations such as union, intersection, and difference.

**Beginner-Friendly Explanation**
A `SortedSet<T>` is a **sorted bag of unique items**. It combines the uniqueness of a set with the ordering of a sorted list.

---

### Purposes

- To maintain a unique, sorted collection.
- To perform set operations while preserving order.
- To support range queries on unique elements.

---

### Syntax Rules and Structure

```
SortedSet<T>   set   =   new   SortedSet<T>()   ;
set.Add(item)   ;
bool   exists   =   set.Contains(item)   ;
```

**Component breakdown**

- `Add` — O(log n) insertion; ignores duplicates.
- `Contains` — O(log n) membership test.

#### Syntax Rules

- Elements are sorted by `IComparer<T>`.
- Duplicates are ignored.
- Supports `GetViewBetween` for range queries.

#### Constraints and Limitations

- O(log n) operations (slower than `HashSet` for membership).
- Not thread-safe.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Sorted Unique Elements

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var set = new SortedSet<int> { 3, 1, 2, 1 };
        Console.WriteLine(string.Join(", ", set)); // 1, 2, 3
    }
}
```

**Expected Output**

```
1, 2, 3
```

**Why this output occurs**

- Duplicates are removed and elements are sorted.

---

### Real-World Cases

- **Leaderboards** — Unique players sorted by score.
- **Time-series data** — Unique timestamps in order.

---

### References

- SortedSet<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.sortedset-1

---

## 11. `ReadOnlyCollection<T>`

### Definitions

**Core Definition**
`ReadOnlyCollection<T>` is a wrapper that provides a read-only view of an existing collection.

**Technical Definition**
`System.Collections.ObjectModel.ReadOnlyCollection<T>` wraps an `IList<T>` and exposes only read operations. It does not copy the underlying collection; changes to the original collection are visible through the wrapper. It implements `IList<T>`, `ICollection<T>`, and `IEnumerable<T>`, but mutation methods throw `NotSupportedException`.

**Beginner-Friendly Explanation**
A `ReadOnlyCollection<T>` is a **glass window** onto a collection. You can see and read the contents, but you cannot change them through the window.

---

### Purposes

- To expose a collection without allowing mutation.
- To protect internal state in API design.
- To provide a read-only view without copying.
- To satisfy `IList<T>` contracts where mutation is not desired.

---

### Syntax Rules and Structure

```
ReadOnlyCollection<T>   ro   =   new   ReadOnlyCollection<T>(list)   ;
```

**Component breakdown**

- Constructor — Accepts an `IList<T>`.
- `Count`, indexer, `Contains`, `IndexOf` — Read operations.

#### Syntax Rules

- Mutation methods throw `NotSupportedException`.
- The wrapper reflects changes to the underlying collection.
- Use `AsReadOnly()` on `List<T>` for a convenient wrapper.

#### Constraints and Limitations

- Not a deep copy; underlying collection must not be modified concurrently.
- Does not prevent mutation of the underlying collection through other references.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Read-Only View

```csharp
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;

class Program
{
    static void Main()
    {
        var list = new List<int> { 1, 2, 3 };
        var ro = new ReadOnlyCollection<int>(list);

        Console.WriteLine(ro[0]);   // 1
        // ro[0] = 99;              // NotSupportedException
    }
}
```

**Expected Output**

```
1
```

**Why this output occurs**

- The indexer reads from the underlying list.
- Assignment throws `NotSupportedException`.

---

### Real-World Cases

- **API returns** — Exposing internal lists without mutation.
- **Immutable facades** — Providing read-only views of mutable data.

---

### References

- ReadOnlyCollection<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.objectmodel.readonlycollection-1

---

## 12. `ImmutableArray<T>`

### Definitions

**Core Definition**
`ImmutableArray<T>` is an immutable array whose contents cannot change after creation.

**Technical Definition**
`System.Collections.Immutable.ImmutableArray<T>` is a struct that wraps an array and guarantees immutability. It implements `IReadOnlyList<T>`, `IList<T>`, and `IEnumerable<T>`. Operations that would modify the array return a new `ImmutableArray<T>`. It is more memory-efficient than `ImmutableList<T>` for small collections.

**Beginner-Friendly Explanation**
An `ImmutableArray<T>` is a **frozen array**. Once created, it never changes. If you need a modified version, you create a new one.

---

### Purposes

- To provide a thread-safe, immutable array.
- To avoid defensive copies in API design.
- To support persistent data structures.
- To enable safe sharing across threads.

---

### Syntax Rules and Structure

```
ImmutableArray<T>   arr   =   ImmutableArray.Create(item1, item2)   ;
ImmutableArray<T>   empty   =   ImmutableArray<T>.Empty   ;
```

**Component breakdown**

- `ImmutableArray.Create` — Factory method.
- `Empty` — Cached empty array.

#### Syntax Rules

- All mutation operations return new instances.
- Implements `IReadOnlyList<T>`.
- Cannot be modified in place.

#### Constraints and Limitations

- `ImmutableArray<T>` is a struct; default values may be uninitialized.
- Operations that create new arrays allocate.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Creating and Reading

```csharp
using System;
using System.Collections.Immutable;

class Program
{
    static void Main()
    {
        var arr = ImmutableArray.Create(1, 2, 3);
        Console.WriteLine(arr[1]);   // 2
    }
}
```

**Expected Output**

```
2
```

**Why this output occurs**

- `ImmutableArray.Create` builds an immutable array.
- Indexed access works like a regular array.

---

### Real-World Cases

- **Configuration** — Immutable lists of settings.
- **Functional programming** — Persistent data structures.

---

### References

- ImmutableArray<T> Struct - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.immutable.immutablearray-1

---

## 13. `ImmutableList<T>`

### Definitions

**Core Definition**
`ImmutableList<T>` is an immutable, persistent list that supports efficient structural sharing.

**Technical Definition**
`System.Collections.Immutable.ImmutableList<T>` is a binary tree-based immutable list. Add, remove, and update operations return new lists while sharing most of the underlying structure. It implements `IImmutableList<T>` and `IReadOnlyList<T>`.

**Beginner-Friendly Explanation**
An `ImmutableList<T>` is a **list you can never change**. Every “modification” produces a new list, but the runtime shares most of the data for efficiency.

---

### Purposes

- To provide immutable list semantics.
- To support functional-style transformations.
- To enable safe concurrent reads without locks.

---

### Syntax Rules and Structure

```
ImmutableList<T>   list   =   ImmutableList.Create<T>()   ;
ImmutableList<T>   list2   =   list.Add(item)   ;
```

**Component breakdown**

- `Create` — Factory method.
- `Add` — Returns a new list with the item added.

#### Syntax Rules

- All mutations return new instances.
- Structural sharing minimizes copying.

#### Constraints and Limitations

- Higher memory overhead than `ImmutableArray` for small collections.
- Not ideal for very large, frequently modified data.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Persistent Add

```csharp
using System;
using System.Collections.Immutable;

class Program
{
    static void Main()
    {
        var list1 = ImmutableList.Create(1, 2);
        var list2 = list1.Add(3);

        Console.WriteLine(list1.Count);   // 2
        Console.WriteLine(list2.Count);   // 3
    }
}
```

**Expected Output**

```
2
3
```

**Why this output occurs**

- `Add` returns a new list; the original is unchanged.

---

### Real-World Cases

- **Event sourcing** — Immutable sequences of events.
- **State management** — Snapshots that never change.

---

### References

- ImmutableList<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.immutable.immutablelist-1

---

## 14. `ImmutableDictionary<TKey, TValue>`

### Definitions

**Core Definition**
`ImmutableDictionary<TKey, TValue>` is an immutable key-value dictionary that returns new dictionaries on modification.

**Technical Definition**
`System.Collections.Immutable.ImmutableDictionary<TKey, TValue>` is a persistent hash-based dictionary. Add, remove, and update operations return new dictionaries with structural sharing. It implements `IImmutableDictionary<TKey, TValue>` and `IReadOnlyDictionary<TKey, TValue>`.

**Beginner-Friendly Explanation**
An `ImmutableDictionary` is a **dictionary that never changes**. If you need to add or remove a key, you get a new dictionary.

---

### Purposes

- To provide immutable key-value storage.
- To support safe concurrent reads.
- To enable persistent configuration and caches.

---

### Syntax Rules and Structure

```
ImmutableDictionary<TKey, TValue>   dict   =   ImmutableDictionary<TKey, TValue>.Empty   ;
var   newDict   =   dict.Add(key,   value)   ;
```

**Component breakdown**

- `Empty` — The empty dictionary.
- `Add` — Returns a new dictionary.

#### Syntax Rules

- Mutations return new instances.
- Keys must be unique.

#### Constraints and Limitations

- O(log n) operations in the persistent tree.
- Slower than mutable `Dictionary` for frequent updates.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Immutable Add

```csharp
using System;
using System.Collections.Immutable;

class Program
{
    static void Main()
    {
        var d1 = ImmutableDictionary<string, int>.Empty;
        var d2 = d1.Add("a", 1);

        Console.WriteLine(d1.Count);   // 0
        Console.WriteLine(d2.Count);   // 1
    }
}
```

**Expected Output**

```
0
1
```

**Why this output occurs**

- `Add` returns a new dictionary; the original remains empty.

---

### Real-World Cases

- **Configuration snapshots** — Immutable settings at a point in time.
- **State trees** — Persistent application state.

---

### References

- ImmutableDictionary<TKey,TValue> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.immutable.immutabledictionary-2

---

## 15. `ConcurrentDictionary<TKey, TValue>`

### Definitions

**Core Definition**
`ConcurrentDictionary<TKey, TValue>` is a thread-safe dictionary designed for concurrent reads and writes.

**Technical Definition**
`System.Collections.Concurrent.ConcurrentDictionary<TKey, TValue>` uses fine-grained locking and lock-free reads to allow multiple threads to access and modify the dictionary safely. It provides atomic operations such as `GetOrAdd`, `AddOrUpdate`, and `TryUpdate`.

**Beginner-Friendly Explanation**
A `ConcurrentDictionary` is a **dictionary safe for many people to use at once**. Multiple threads can read and write without corrupting the data.

---

### Purposes

- To provide thread-safe key-value storage.
- To support atomic add-or-update operations.
- To enable concurrent caches and shared state.

---

### Syntax Rules and Structure

```
ConcurrentDictionary<TKey, TValue>   dict   =   new   ConcurrentDictionary<TKey, TValue>()   ;
dict.TryAdd(key,   value)   ;
TValue   v   =   dict.GetOrAdd(key,   factory)   ;
```

**Component breakdown**

- `TryAdd` — Atomic add; returns `false` if the key exists.
- `GetOrAdd` — Returns existing value or adds a new one atomically.
- `AddOrUpdate` — Atomically updates or adds.

#### Syntax Rules

- Operations are thread-safe without external locks.
- `GetOrAdd` may invoke the factory multiple times under contention.
- Implements `IDictionary<TKey, TValue>`.

#### Constraints and Limitations

- Higher overhead than `Dictionary` for single-threaded use.
- Enumeration is not a snapshot; it reflects concurrent modifications.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Atomic Add

```csharp
using System;
using System.Collections.Concurrent;

class Program
{
    static void Main()
    {
        var dict = new ConcurrentDictionary<string, int>();
        Console.WriteLine(dict.TryAdd("a", 1));   // True
        Console.WriteLine(dict.TryAdd("a", 2));   // False
        Console.WriteLine(dict["a"]);             // 1
    }
}
```

**Expected Output**

```
True
False
1
```

**Why this output occurs**

- `TryAdd` returns `false` when the key already exists.

---

### Real-World Cases

- **Shared caches** — Multiple threads reading and writing cached values.
- **Counters and metrics** — Atomic increments by key.

---

### References

- ConcurrentDictionary<TKey,TValue> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentdictionary-2

---

## 16. Collection Interfaces

### Definitions

**Core Definition**
Collection interfaces define contracts that collections implement, enabling polymorphic consumption and decoupling of producers from consumers.

**Technical Definition**
The generic collection interfaces form a hierarchy: `IEnumerable<T>` → `ICollection<T>` → `IList<T>`, `IDictionary<TKey,TValue>`, and `ISet<T>`. Read-only variants (`IReadOnlyCollection<T>`, `IReadOnlyList<T>`, `IReadOnlyDictionary<TKey,TValue>`) provide mutation-free views. `IEnumerable<T>` exposes an enumerator for simple iteration; `ICollection<T>` adds `Count`, `Add`, `Remove`, `Contains`, and `Clear`; `IList<T>` adds indexed access.

**Beginner-Friendly Explanation**
Interfaces are **contracts**. `IEnumerable<T>` says “you can loop over me.” `ICollection<T>` says “you can count and modify me.” `IList<T>` says “you can access me by index.” `IReadOnlyList<T>` says “you can read me by index, but not change me.”

---

### Purposes

- To decouple code from concrete collection types.
- To express intent (e.g., read-only vs. mutable).
- To enable LINQ and `foreach` over any collection.
- To support polymorphism in APIs.
- To provide a uniform programming model across collections.

---

### Syntax Rules and Structure

```
IEnumerable<T>   seq   =   list   ;
ICollection<T>   coll   =   list   ;
IList<T>   indexed   =   list   ;
IReadOnlyList<T>   ro   =   list   ;
```

**Component breakdown**

- `IEnumerable<T>` — Iteration only.
- `ICollection<T>` — Count and mutation.
- `IList<T>` — Indexed access.
- `IReadOnlyList<T>` — Indexed read-only access.

#### Syntax Rules

- All generic collections implement `IEnumerable<T>`.
- `IList<T>` extends `ICollection<T>` and `IEnumerable<T>`.
- `IReadOnlyList<T>` extends `IReadOnlyCollection<T>` and `IEnumerable<T>`.
- `IDictionary<TKey,TValue>` extends `ICollection<KeyValuePair<TKey,TValue>>`.

#### Constraints and Limitations

- Interface dispatch may introduce allocations in hot loops (use generic constraints or concrete types for performance).
- `IEnumerable<T>` can be lazily evaluated; multiple enumerations may re-execute the sequence.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Polymorphic Consumption

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void PrintAll(IEnumerable<int> items)
    {
        foreach (int item in items)
            Console.WriteLine(item);
    }

    static void Main()
    {
        var list = new List<int> { 1, 2, 3 };
        var set = new HashSet<int> { 4, 5 };
        PrintAll(list);
        PrintAll(set);
    }
}
```

**Expected Output**

```
1
2
3
4
5
```

**Why this output occurs**

- `PrintAll` accepts any `IEnumerable<int>`, so both `List<int>` and `HashSet<int>` work.

---

#### Example 2: Read-Only Interfaces

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void PrintCount(IReadOnlyCollection<int> items)
    {
        Console.WriteLine(items.Count);
    }

    static void Main()
    {
        PrintCount(new List<int> { 1, 2, 3 });   // 3
    }
}
```

**Expected Output**

```
3
```

**Why this output occurs**

- `IReadOnlyCollection<T>` exposes `Count` without mutation methods.

---

### Real-World Cases

- **API design** — Accepting `IEnumerable<T>` for streaming, `IReadOnlyList<T>` for indexed reads, and `IList<T>` for mutation.
- **LINQ** — Querying any `IEnumerable<T>`.
- **Testing** — Mocking interfaces instead of concrete collections.

---

### References

- IEnumerable<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1
- ICollection<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.icollection-1
- IList<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ilist-1
- IDictionary<TKey,TValue> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.idictionary-2
- ISet<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.iset-1
- IReadOnlyCollection<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ireadonlycollection-1
- IReadOnlyList<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ireadonlylist-1
- IReadOnlyDictionary<TKey,TValue> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ireadonlydictionary-2

---

## 17. Collection Initialization

### Definitions

**Core Definition**
Collection initialization is the syntax for creating a collection and populating it with elements in a single expression.

**Technical Definition**
C# provides **collection initializers** (`new List<int> { 1, 2, 3 }`) and **object initializers** for collections that implement `IEnumerable` and have an `Add` method. Index initializers (`new Dictionary<int, string> { [1] = "one" }`) are also supported. These are compiled into calls to `Add` or the indexer.

**Beginner-Friendly Explanation**
Collection initialization is **building the container and filling it at the same time**. Instead of creating an empty list and calling `Add` repeatedly, you write the items directly in braces.

---

### Purposes

- To create and populate collections concisely.
- To reduce boilerplate code.
- To improve readability of test data and configuration.
- To support immutable and read-only collections in initializers.

---

### Syntax Rules and Structure

```
List<T>   list   =   new   List<T>   {   item1, item2   }   ;
Dictionary<TKey, TValue>   map   =   new   Dictionary<TKey, TValue>   {   [key] = value   }   ;
```

**Component breakdown**

- `{ ... }` — Collection initializer.
- `[key] = value` — Index initializer for dictionaries.

#### Syntax Rules

- The collection must implement `IEnumerable` and have an accessible `Add` method.
- Index initializers require an indexer.
- Initializers can be nested for jagged or complex structures.

#### Constraints and Limitations

- Not all collections support collection initializers (e.g., those without `Add`).
- Read-only collections may require constructor arguments instead.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: List and Dictionary Initializers

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var list = new List<int> { 1, 2, 3 };
        var map = new Dictionary<string, int> { ["a"] = 1, ["b"] = 2 };

        Console.WriteLine(list.Count);   // 3
        Console.WriteLine(map["a"]);     // 1
    }
}
```

**Expected Output**

```
3
1
```

**Why this output occurs**

- The initializer calls `Add` for each element.

---

### Real-World Cases

- **Test data** — Predefined collections for unit tests.
- **Configuration** — Dictionaries of settings.
- **Lookup tables** — Precomputed mappings.

---

### References

- Object and Collection Initializers - C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers

---

## 18. Collection Expressions (C# 12)

### Definitions

**Core Definition**
Collection expressions are a C# 12 feature that provides a unified, concise syntax (`[...]`) for creating collections.

**Technical Definition**
Collection expressions use square brackets (`[1, 2, 3]`) to initialize arrays, spans, and collection types that support collection initializers. They can also use the spread operator (`..`) to inline another collection. The compiler lowers collection expressions to efficient construction code, potentially using `CollectionsMarshal` or cached empty instances.

**Beginner-Friendly Explanation**
Collection expressions are a **shorter, cleaner way to write collections**. Instead of `new List<int> { 1, 2, 3 }`, you write `[1, 2, 3]`.

---

### Purposes

- To provide a uniform syntax for all collection types.
- To reduce verbosity in collection creation.
- To support spread elements for combining collections.
- To improve readability in modern C# code.

---

### Syntax Rules and Structure

```
int[]   arr   =   [1, 2, 3]   ;
List<int>   list   =   [1, 2, 3]   ;
int[]   combined   =   [..arr, 4, 5]   ;
```

**Component breakdown**

- `[ ... ]` — Collection expression.
- `..` — Spread operator for inlining another collection.

#### Syntax Rules

- Requires C# 12 / .NET 8+.
- Works for arrays, spans, and types with collection initializers.
- Spread elements must be enumerable.

#### Constraints and Limitations

- Not all collection types support collection expressions.
- Some semantic differences exist compared to traditional initializers (e.g., empty array caching).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Collection Expression

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        int[] arr = [1, 2, 3];
        List<int> list = [4, 5, 6];

        Console.WriteLine(arr.Length);   // 3
        Console.WriteLine(list.Count);   // 3
    }
}
```

**Expected Output**

```
3
3
```

**Why this output occurs**

- Collection expressions create the corresponding collection with the specified elements.

---

#### Example 2: Spread Operator

```csharp
using System;

class Program
{
    static void Main()
    {
        int[] first = [1, 2];
        int[] combined = [..first, 3, 4];

        Console.WriteLine(string.Join(", ", combined)); // 1, 2, 3, 4
    }
}
```

**Expected Output**

```
1, 2, 3, 4
```

**Why this output occurs**

- `..first` spreads the elements of `first` into the new array.

---

### Real-World Cases

- **Modern API design** — Concise collection creation in public APIs.
- **Data pipelines** — Combining sequences with spread.

---

### References

- Collection expressions - C# 12 - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12#collection-expressions
- IDE0028: Use collection initializers or expressions - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0028

---

## 19. Collection Best Practices

### Definitions

**Core Definition**
Collection best practices are the guidelines for choosing, using, and exposing collections in C# to achieve correctness, performance, and maintainability.

**Technical Definition**
Best practices include preferring generic collections over non-generic, choosing the right collection for the access pattern, exposing interfaces (`IEnumerable<T>`, `IReadOnlyList<T>`) rather than concrete types in public APIs, avoiding unnecessary boxing, pre-sizing collections when the size is known, and using concurrent or immutable collections for shared state.

**Beginner-Friendly Explanation**
Best practices are the habits that keep collection code fast and safe: use generic collections, pick the right container for the job, expose read-only interfaces when callers should not mutate, and avoid creating unnecessary copies.

---

### Purposes

- To choose the optimal collection for each scenario.
- To avoid performance pitfalls (boxing, unnecessary allocations, wrong complexity).
- To design APIs that communicate intent through interfaces.
- To ensure thread safety where needed.
- To write readable, maintainable collection code.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer generic collections
var list = new List<int>();   // not ArrayList

// Choose the right collection
// Indexed access → List<T>
// Key lookup → Dictionary<TKey,TValue>
// Unique elements → HashSet<T>
// FIFO → Queue<T>
// LIFO → Stack<T>

// Expose interfaces in public APIs
public IReadOnlyList<string> Names { get; }

// Pre-size collections when known
var list = new List<int>(capacity: 1000);

// Use concurrent collections for shared mutable state
var cache = new ConcurrentDictionary<string, object>();
```

**Component breakdown**

- **Generic over non-generic** — Type safety and no boxing.
- **Right collection** — Match access pattern to performance profile.
- **Interface exposure** — `IEnumerable<T>`, `IReadOnlyList<T>`, `IReadOnlyDictionary<TKey,TValue>`.
- **Pre-sizing** — Avoid repeated resizing.
- **Concurrency** — `ConcurrentDictionary`, `ConcurrentQueue`, etc.

#### Rules and Constraints

- Avoid `System.Collections` non-generic types in new code.
- Use `IReadOnlyList<T>` or `IReadOnlyCollection<T>` for API returns that should not be mutated.
- Use `TryGetValue` instead of `ContainsKey` followed by indexer access.
- Consider `HashSet<T>` for membership tests instead of `List<T>.Contains`.
- Use `StringComparer.Ordinal` or `StringComparer.OrdinalIgnoreCase` for dictionary keys when appropriate.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Interface-Based API Design

```csharp
using System;
using System.Collections.Generic;

public class Inventory
{
    private readonly List<string> _items = new();

    public IReadOnlyList<string> Items => _items.AsReadOnly();

    public void Add(string item) => _items.Add(item);
}

class Program
{
    static void Main()
    {
        var inv = new Inventory();
        inv.Add("apple");
        Console.WriteLine(inv.Items[0]); // apple
    }
}
```

**Expected Output**

```
apple
```

**Why this output occurs**

- `Items` exposes a read-only view; callers cannot mutate the underlying list.

---

#### Example 2: Choosing `HashSet<T>` for Membership

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var allowed = new HashSet<string>(StringComparer.OrdinalIgnoreCase) { "admin", "user" };
        Console.WriteLine(allowed.Contains("ADMIN")); // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `HashSet<T>` provides O(1) membership with case-insensitive comparison.

---

### Real-World Cases

- **API design** — Returning `IReadOnlyList<T>` from repository methods.
- **Caching** — Using `ConcurrentDictionary<TKey,TValue>` for shared caches.
- **Configuration** — `ImmutableDictionary<TKey,TValue>` for settings that never change at runtime.

---

### References

- Collections and Data Structures - .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/
- Selecting a Collection Class - .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/selecting-a-collection-class
- IDE0028: Use collection initializers or expressions - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0028

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Non-generic collections (`ArrayList`, `Hashtable`) | **Legacy**; box value types; avoid in new code. |
| `ImmutableArray<T>` | Requires `System.Collections.Immutable` NuGet package; struct with default-value pitfalls. |
| `ConcurrentDictionary<TKey,TValue>` | Available since .NET 4.0; thread-safe but higher overhead than `Dictionary`. |
| Collection expressions (`[...]`) | Introduced in **C# 12**; version-specific. |
| `SortedList<TKey,TValue>` insertion | **O(n)** for unsorted data; choose `SortedDictionary` for frequent insertions. |
| `IReadOnlyList<T>` / `IReadOnlyDictionary<TKey,TValue>` | Since .NET 4.5; recommended for API returns. |
| `LinkedList<T>` cache locality | **Implementation-dependent**; poor compared to arrays. |
| `Dictionary<TKey,TValue>` iteration order | **Implementation-dependent**; not guaranteed. |

---

## References (Consolidated)

- Collections and Data Structures - .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/
- Commonly Used Collection Types - .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/commonly-used-collection-types
- Selecting a Collection Class - .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/selecting-a-collection-class
- System.Collections.Generic Namespace - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic
- List<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1
- Dictionary<TKey,TValue> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2
- HashSet<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.hashset-1
- Queue<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.queue-1
- Stack<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.stack-1
- LinkedList<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.linkedlist-1
- SortedList<TKey,TValue> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.sortedlist-2
- SortedDictionary<TKey,TValue> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.sorteddictionary-2
- SortedSet<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.sortedset-1
- ReadOnlyCollection<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.objectmodel.readonlycollection-1
- ImmutableArray<T> Struct - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.immutable.immutablearray-1
- ImmutableList<T> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.immutable.immutablelist-1
- ImmutableDictionary<TKey,TValue> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.immutable.immutabledictionary-2
- ConcurrentDictionary<TKey,TValue> Class - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentdictionary-2
- IEnumerable<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1
- ICollection<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.icollection-1
- IList<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ilist-1
- IDictionary<TKey,TValue> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.idictionary-2
- ISet<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.iset-1
- IReadOnlyCollection<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ireadonlycollection-1
- IReadOnlyList<T> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ireadonlylist-1
- IReadOnlyDictionary<TKey,TValue> Interface - .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ireadonlydictionary-2
- Object and Collection Initializers - C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers
- Collection expressions - C# 12 - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12#collection-expressions
- IDE0028: Use collection initializers or expressions - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0028

---