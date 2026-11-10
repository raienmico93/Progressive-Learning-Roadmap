# C# Generic Collections — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Generic collections are **type-safe, reusable data structures** in the `System.Collections.Generic` namespace that store and manage groups of elements of a specific type `T`. Unlike their non-generic counterparts (such as `ArrayList` and `Hashtable`), generic collections enforce type correctness **at compile time**, eliminating the need for casting and reducing runtime errors. They are the standard collection types in modern .NET and form the backbone of LINQ, data access, and everyday application development.

**Technical Definition**
Generic collections are constructed with a **type parameter** (e.g., `List<T>`, `Dictionary<TKey,TValue>`). Because the type is known at compile time, the compiler and JIT can generate specialised, efficient code for each closed generic type, avoiding the boxing and unboxing overhead that plagues non-generic collections storing value types as `object`. Generic collections implement a rich set of interfaces—`IEnumerable<T>`, `ICollection<T>`, `IList<T>`, `IDictionary<TKey,TValue>`, and `ISet<T>`—enabling polymorphic consumption and LINQ querying. The core generic collections include **`List<T>`** (ordered, index-based list), **`Dictionary<TKey,TValue>`** (key-value hash table), **`HashSet<T>`** (unordered set of unique elements), **`Queue<T>`** (first-in, first-out), **`Stack<T>`** (last-in, first-out), and **`LinkedList<T>`** (doubly linked list). Each is optimised for a distinct access pattern, and choosing the right one is essential for correctness and performance.

**Beginner-Friendly Explanation**
Think of generic collections as **specialised containers** for your data. A `List<T>` is a flexible numbered shelf; a `Dictionary` is a filing cabinet indexed by keys; a `HashSet` is a bag that rejects duplicates; a `Queue` is a line at a ticket counter; a `Stack` is a pile of plates; and a `LinkedList` is a chain of connected nodes. Because they are **generic**, you tell them exactly what type they hold—`List<string>`, `Dictionary<int, Customer>`—and the compiler ensures that only that type goes in and comes out. No casting, no surprises, better performance.

---

### Key Characteristics

- **Compile-time type safety** — The compiler rejects attempts to add elements of the wrong type.
- **No boxing for value types** — Generic collections store value types directly, avoiding heap allocations and unboxing costs.
- **Rich interface hierarchy** — All generic collections implement `IEnumerable<T>`; many implement `ICollection<T>`, `IList<T>`, `IDictionary<TKey,TValue>`, or `ISet<T>`.
- **Performance-optimised** — Each collection is designed for a specific access pattern (indexed access, key lookup, FIFO, LIFO, etc.).
- **LINQ-enabled** — Because they implement `IEnumerable<T>`, generic collections can be queried with LINQ.
- **Dynamic sizing** — Most generic collections grow and shrink automatically as elements are added or removed.

---

### Prerequisites

- Understanding of C# generics (`<T>`, type parameters).
- Familiarity with arrays and basic collection concepts.
- Knowledge of interfaces and polymorphism.
- Awareness of time complexity (Big-O) for choosing the right collection.
- Basic understanding of value types vs. reference types and boxing.

---

### Related Programming Areas

- **LINQ** — Generic collections are the primary data source for LINQ queries.
- **Data access** — ORMs and repositories return generic collections.
- **Concurrency** — `ConcurrentDictionary<TKey,TValue>` and other concurrent collections extend the generic model to multithreaded scenarios.
- **Immutability** — `ImmutableList<T>`, `ImmutableDictionary<TKey,TValue>` provide immutable alternatives.
- **Algorithm design** — The choice of collection determines the complexity of operations.

---

### Core Concepts / Features

1. `List<T>`
2. `Dictionary<TKey, TValue>`
3. `HashSet<T>`
4. `Queue<T>`
5. `Stack<T>`
6. `LinkedList<T>`
7. Generic Collection Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. `List<T>`

### Definitions

**Core Definition**
`List<T>` is a **resizable, index-based, strongly typed list** that stores elements of type `T` in order and provides methods for searching, sorting, and manipulating the list.

**Technical Definition**
`System.Collections.Generic.List<T>` represents a strongly typed list of objects that can be accessed by index. It is backed by a dynamically resized array and implements `IList<T>`, `ICollection<T>`, `IEnumerable<T>`, `IReadOnlyList<T>`, and `IReadOnlyCollection<T>`. Indexed access is O(1); appending is amortised O(1); insertion and removal in the middle are O(n). `List<T>` provides methods such as `Add`, `Insert`, `Remove`, `RemoveAt`, `IndexOf`, `Sort`, `Reverse`, `Find`, `FindAll`, and `ForEach`.

**Beginner-Friendly Explanation**
`List<T>` is a **flexible numbered list**. You can add items to the end, insert them in the middle, remove them, and access any item by its index. It is the go-to collection for ordered data that changes size.

---

### Purposes

- To store an **ordered, indexable sequence** of elements.
- To provide **fast access by position** (O(1)).
- To support **dynamic resizing** without manual array management.
- To serve as a **general-purpose replacement** for fixed-size arrays.
- To support **LINQ queries** through `IEnumerable<T>`.

---

### Syntax Rules and Structure

```
List<T> list = new List<T>();
List<T> list2 = new List<T> { item1, item2, ... };
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

- Insertion and removal in the middle are O(n).
- Not thread-safe; use `ConcurrentBag<T>` or synchronisation for concurrent access.
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

- `Add` appends, `Insert` places an element at a specific index, and `Remove` deletes the first matching element.

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

- List<T> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1

---

## 2. `Dictionary<TKey, TValue>`

### Definitions

**Core Definition**
`Dictionary<TKey, TValue>` is a **hash-based collection of key-value pairs** that provides fast lookup by unique key.

**Technical Definition**
`System.Collections.Generic.Dictionary<TKey,TValue>` represents a collection of keys and values. It implements `IDictionary<TKey,TValue>`, `ICollection<KeyValuePair<TKey,TValue>>`, `IEnumerable<KeyValuePair<TKey,TValue>>`, `IReadOnlyDictionary<TKey,TValue>`, and `IReadOnlyCollection<KeyValuePair<TKey,TValue>>`. Lookup, insertion, and removal are O(1) on average. Keys must be unique and non-null (for reference-type keys); values may be any type. The indexer `dict[key]` returns the value or throws `KeyNotFoundException` if the key is missing; `TryGetValue` provides a safe alternative.

**Beginner-Friendly Explanation**
A `Dictionary` is a **lookup table**. You give it a key (like a name), and it instantly returns the associated value (like a phone number). Each key can appear only once.

---

### Purposes

- To store data as **key-value pairs** for fast lookup.
- To map **identifiers to objects** (e.g., ID → entity).
- To implement **caches and indexes**.
- To provide **O(1) average-time access** by key.
- To support **configuration and settings storage**.

---

### Syntax Rules and Structure

```
Dictionary<TKey, TValue> map = new Dictionary<TKey, TValue>();
map[key] = value;
TValue v = map[key];
```

**Component breakdown**

- `TKey` — The key type (must be non-null).
- `TValue` — The value type.
- `map[key]` — Indexer for get/set.
- `Add(key, value)` — Adds a new pair; throws if the key exists.
- `TryGetValue(key, out value)` — Safe lookup.

#### Syntax Rules

- Keys must be unique; adding a duplicate key with `Add` throws `ArgumentException`.
- The indexer `map[key]` throws `KeyNotFoundException` if the key is missing.
- `TryGetValue` returns `bool` and avoids exceptions.
- `ContainsKey` checks for key presence.

#### Constraints and Limitations

- Not thread-safe; use `ConcurrentDictionary<TKey,TValue>` for concurrent access.
- Iteration order is not guaranteed.
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

- The indexer retrieves the value for `"Alice"`; `ContainsKey` checks for `"Bob"`; `TryGetValue` returns `false` for a missing key without throwing.

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

- Dictionary<TKey,TValue> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2

---

## 3. `HashSet<T>`

### Definitions

**Core Definition**
`HashSet<T>` is an **unordered collection of unique elements** that provides high-performance set operations such as union, intersection, and difference.

**Technical Definition**
`System.Collections.Generic.HashSet<T>` represents a set of values. It implements `ISet<T>`, `ICollection<T>`, `IEnumerable<T>`, and `IReadOnlyCollection<T>`. `Add`, `Remove`, and `Contains` are O(1) on average. The `Add` method returns `bool` (`false` if the element already exists). Set operations are available through `UnionWith`, `IntersectWith`, `ExceptWith`, and `SymmetricExceptWith`.

**Beginner-Friendly Explanation**
A `HashSet<T>` is a **bag that refuses duplicates**. If you try to add the same item twice, the second add is silently ignored. It is excellent for checking membership quickly.

---

### Purposes

- To store a collection of **unique elements**.
- To perform **fast membership tests** (O(1)).
- To implement **mathematical set operations**.
- To **deduplicate data**.
- To track **seen items** efficiently.

---

### Syntax Rules and Structure

```
HashSet<T> set = new HashSet<T>();
set.Add(item);
bool exists = set.Contains(item);
```

**Component breakdown**

- `HashSet<T>` — The set type.
- `Add` — Returns `true` if added; `false` if already present.
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

- HashSet<T> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.hashset-1

---

## 4. `Queue<T>`

### Definitions

**Core Definition**
`Queue<T>` is a **first-in, first-out (FIFO) collection** where elements are dequeued in the order they were enqueued.

**Technical Definition**
`System.Collections.Generic.Queue<T>` represents a first-in, first-out collection of objects. It implements `IEnumerable<T>`, `IReadOnlyCollection<T>`, and `ICollection`. `Enqueue` adds to the rear in amortised O(1); `Dequeue` removes from the front in O(1); `Peek` returns the front element without removing it. `TryDequeue` (since .NET Core 2.0) provides a safe alternative to `Dequeue` on an empty queue.

**Beginner-Friendly Explanation**
A `Queue<T>` is a **line at a checkout counter**. The first person in line is the first served. You add to the back and remove from the front.

---

### Purposes

- To process items in **arrival order**.
- To implement **breadth-first search**.
- To buffer work items for **asynchronous processing**.
- To model real-world queues (print jobs, messages).

---

### Syntax Rules and Structure

```
Queue<T> q = new Queue<T>();
q.Enqueue(item);
T front = q.Dequeue();
T peek = q.Peek();
```

**Component breakdown**

- `Enqueue` — Adds to the rear.
- `Dequeue` — Removes and returns the front.
- `Peek` — Returns the front without removing.

#### Syntax Rules

- `Dequeue` on an empty queue throws `InvalidOperationException`.
- `TryDequeue` provides a safe alternative.
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

- Queue<T> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.queue-1

---

## 5. `Stack<T>`

### Definitions

**Core Definition**
`Stack<T>` is a **last-in, first-out (LIFO) collection** where the most recently added element is removed first.

**Technical Definition**
`System.Collections.Generic.Stack<T>` represents a variable-size last-in-first-out collection of instances of the same specified type. It implements `IEnumerable<T>`, `IReadOnlyCollection<T>`, and `ICollection`. `Push` adds to the top in amortised O(1); `Pop` removes and returns the top in O(1); `Peek` returns the top without removing it. `TryPop` (since .NET Core 2.0) provides a safe alternative.

**Beginner-Friendly Explanation**
A `Stack<T>` is a **pile of plates**. You put a plate on top, and you take the top plate off first. The last plate placed is the first one removed.

---

### Purposes

- To implement **undo/redo** functionality.
- To evaluate expressions (e.g., postfix notation).
- To traverse trees or graphs in **depth-first order**.
- To manage nested scopes or states.

---

### Syntax Rules and Structure

```
Stack<T> s = new Stack<T>();
s.Push(item);
T top = s.Pop();
T peek = s.Peek();
```

**Component breakdown**

- `Push` — Adds to the top.
- `Pop` — Removes and returns the top.
- `Peek` — Returns the top without removing.

#### Syntax Rules

- `Pop` on an empty stack throws `InvalidOperationException`.
- `TryPop` provides a safe alternative.

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

- Stack<T> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.stack-1

---

## 6. `LinkedList<T>`

### Definitions

**Core Definition**
`LinkedList<T>` is a **doubly linked list** that stores elements in individual nodes, each pointing to the previous and next node, providing O(1) insertion and removal at known positions.

**Technical Definition**
`System.Collections.Generic.LinkedList<T>` represents a doubly linked list. It implements `ICollection<T>`, `IEnumerable<T>`, and `IReadOnlyCollection<T>`. Insertion and removal at a known `LinkedListNode<T>` are O(1). Searching by value is O(n) via `Find`. The class exposes `AddFirst`, `AddLast`, `AddBefore`, `AddAfter`, `Remove`, and `RemoveFirst`/`RemoveLast`. It does not support indexed access.

**Beginner-Friendly Explanation**
A `LinkedList<T>` is a **chain of nodes**. Each node knows its neighbours, so inserting or removing a node is fast if you already have a reference to it. But finding a specific value requires walking the chain.

---

### Purposes

- To support **frequent insertion and removal** in the middle of a sequence.
- To implement **LRU caches** and adjacency lists.
- To avoid the **shifting cost** of array-based lists.
- To provide **stable references** to nodes across modifications.

---

### Syntax Rules and Structure

```
LinkedList<T> list = new LinkedList<T>();
LinkedListNode<T> node = list.AddFirst(item);
list.AddAfter(node, newItem);
list.Remove(node);
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

- `AddFirst`, `AddAfter`, and `AddLast` build the list; `Remove(node)` removes the node in O(1).

---

### Real-World Cases

- **LRU caches** — Move accessed nodes to the front.
- **Adjacency lists** — Graph representations with frequent edge updates.
- **Music playlists** — Insert and remove tracks efficiently.

---

### References

- LinkedList<T> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.linkedlist-1

---

## 7. Generic Collection Best Practices

### Definitions

**Core Definition**
Generic collection best practices are the guidelines for **choosing the right collection type**, using it efficiently, and avoiding common pitfalls such as boxing, unnecessary copying, and incorrect thread assumptions.

**Technical Definition**
Best practices for generic collections derive from Microsoft's guidance on collections and data structures. Key recommendations include: **use generic collections over non-generic collections** because they are type-safe at compile time and typically offer better performance; **choose the collection that matches the access pattern** (indexed access → `List<T>`; key lookup → `Dictionary<TKey,TValue>`; unique elements → `HashSet<T>`; FIFO → `Queue<T>`; LIFO → `Stack<T>`; frequent middle insertion/removal → `LinkedList<T>`); **pre-size collections** when the capacity is known (`new List<T>(capacity)`); **use `TryGetValue` instead of `ContainsKey` + indexer** for dictionary lookups; **avoid modifying a collection inside a `foreach` loop**; and **use concurrent or immutable collections for shared state**.

**Beginner-Friendly Explanation**
Best practices for generic collections are simple: **pick the right container for the job**, **pre-size it if you know how much it will hold**, **use safe lookup methods**, and **do not change a collection while you are looping over it**.

---

### Purposes

- To choose the **optimal collection** for each scenario.
- To avoid **performance pitfalls** (boxing, unnecessary allocations, wrong complexity).
- To ensure **thread safety** where needed.
- To write **readable, maintainable** collection code.
- To align with **.NET conventions**.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer generic collections over non-generic
var list = new List<int>();   // not ArrayList

// Choose the right collection
// Indexed access → List<T>
// Key lookup → Dictionary<TKey,TValue>
// Unique elements → HashSet<T>
// FIFO → Queue<T>
// LIFO → Stack<T>
// Frequent middle insertion/removal → LinkedList<T>

// Pre-size when capacity is known
var list = new List<int>(capacity: 1000);

// Use TryGetValue instead of ContainsKey + indexer
if (dict.TryGetValue(key, out var value)) { ... }

// Do not modify a collection inside foreach
// Use ConcurrentDictionary for shared state
```

**Component breakdown**

- **Generic over non-generic** — Type safety and no boxing.
- **Right collection** — Match access pattern to performance profile.
- **Pre-sizing** — Avoid repeated resizing.
- **Safe lookup** — `TryGetValue` avoids exceptions.
- **No mutation in `foreach`** — Use a snapshot or index-based loop.
- **Concurrency** — `ConcurrentDictionary`, `ConcurrentQueue`, `ConcurrentStack`.

#### Rules and Constraints

- **Do** use generic collections over non-generic ones.
- **Do** choose the collection that matches the access pattern.
- **Do** pre-size collections when the capacity is known.
- **Do** use `TryGetValue` for dictionary lookups.
- **Do not** modify a collection inside a `foreach` loop.
- **Do not** use `List<T>.Contains` for frequent membership tests; use `HashSet<T>`.
- **Do not** use `LinkedList<T>` when indexed access is needed.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Choosing the Right Collection

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        // Indexed access: List<T>
        var list = new List<string> { "a", "b", "c" };
        Console.WriteLine(list[1]);   // b

        // Key lookup: Dictionary<TKey,TValue>
        var dict = new Dictionary<int, string> { [1] = "one" };
        Console.WriteLine(dict[1]);   // one

        // Unique elements: HashSet<T>
        var set = new HashSet<int> { 1, 2, 2, 3 };
        Console.WriteLine(set.Count); // 3
    }
}
```

**Expected Output**

```
b
one
3
```

**Why this output occurs**

- Each collection is used according to its intended access pattern.

---

#### Example 2: Safe Dictionary Lookup

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var ages = new Dictionary<string, int> { ["Alice"] = 30 };

        if (ages.TryGetValue("Bob", out int age))
            Console.WriteLine(age);
        else
            Console.WriteLine("Not found"); // this branch
    }
}
```

**Expected Output**

```
Not found
```

**Why this output occurs**

- `TryGetValue` returns `false` for a missing key without throwing an exception.

---

### Real-World Cases

- **API design** — Returning `IReadOnlyList<T>` or `IReadOnlyDictionary<TKey,TValue>` for immutable views.
- **Caching** — Using `ConcurrentDictionary<TKey,TValue>` for shared caches.
- **Data processing** — Choosing `HashSet<T>` for deduplication and `Queue<T>` for FIFO pipelines.

---

### References

- Collections and Data Structures — .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/
- When to use generic collections — .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/when-to-use-generic-collections
- Select a collection — .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/selecting-a-collection-class

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Non-generic collections (`ArrayList`, `Hashtable`) | **Legacy**; box value types; avoid in new code. |
| `ConcurrentDictionary<TKey,TValue>` | Thread-safe; available since .NET 4.0. |
| Immutable collections (`ImmutableList<T>`, etc.) | Require `System.Collections.Immutable` NuGet package. |
| `List<T>` internal array | **Implementation detail**; use `CollectionsMarshal.AsSpan` for direct access. |
| `Dictionary<TKey,TValue>` iteration order | **Implementation-dependent**; not guaranteed. |
| `Queue<T>.TryDequeue` / `Stack<T>.TryPop` | Available since .NET Core 2.0. |
| `LinkedList<T>` cache locality | **Implementation-dependent**; poor compared to arrays. |

---

## References (Consolidated)

- List<T> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1
- Dictionary<TKey,TValue> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2
- HashSet<T> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.hashset-1
- Queue<T> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.queue-1
- Stack<T> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.stack-1
- LinkedList<T> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.linkedlist-1
- Collections and Data Structures — .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/
- When to use generic collections — .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/when-to-use-generic-collections
- Select a collection — .NET - https://learn.microsoft.com/en-us/dotnet/standard/collections/selecting-a-collection-class
- Implement ordered collections with List<T> — Training - https://learn.microsoft.com/en-us/training/modules/implement-ordered-collections/

---