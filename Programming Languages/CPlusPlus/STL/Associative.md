# C++ Associative, Unordered & Flat Containers — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Associative containers are standard library class templates that store elements in a sorted or hashed arrangement, enabling efficient lookup by key. Unordered containers use hash tables for average O(1) access, while ordered associative containers use balanced binary search trees for guaranteed O(log n) access. C++23 introduces flat containers that combine sorted-vector storage with associative interfaces for cache-optimized performance.

**Technical Definition**
The C++ standard library provides eight node-based associative containers: `std::set`, `std::map`, `std::multiset`, `std::multimap` (ordered, typically implemented as red-black trees) and `std::unordered_set`, `std::unordered_map`, `std::unordered_multiset`, `std::unordered_multimap` (unordered, implemented as hash tables with separate chaining). C++23 adds `std::flat_set`, `std::flat_map`, `std::flat_multiset`, and `std::flat_multimap` as container adaptors backed by sorted contiguous storage (typically `std::vector`). All node-based associative containers support the `.extract()` API (C++17) for transferring nodes between containers without copying or moving the stored elements.

**Beginner-Friendly Explanation**
Imagine you have a phone book. You want to look up a person's number quickly. An associative container is like a well-organized phone book: you look up by name (the key) and get the number (the value). Ordered containers keep everything sorted alphabetically, so lookups are fast but inserting a new name requires rearranging. Unordered containers use a "magic index" (hash function) that tells you exactly which page to look on — usually instant, but sometimes two names land on the same page and you have to check a few. Flat containers are like a sorted index card box: everything is packed tightly together, so flipping through is very fast, but adding a new card in the middle means shuffling all the cards after it.

---

### Key Characteristics

- **Key-based access**: All associative containers store elements indexed by a key.
- **Unique vs. equivalent keys**: `set`/`map`/`unordered_set`/`unordered_map` store at most one element per key; the `multi` variants allow equivalent keys.
- **Ordered vs. unordered**: Ordered containers maintain sorted order (by `Compare`); unordered containers organize elements into buckets based on hash values.
- **Node-based vs. flat**: Node-based containers (`set`, `map`, unordered variants) allocate each element in a separate node; flat containers store elements contiguously.
- **Complexity guarantees**: Ordered: O(log n) for search, insert, erase. Unordered: average O(1), worst-case O(n). Flat: O(log n) lookup, O(n) insert/erase due to shifting.
- **Iterator stability**: Node-based containers provide stable iterators (except for erased elements); flat containers invalidate iterators on insert/erase.
- **Node extraction** (C++17): `.extract()` transfers ownership of a node between containers without copying the key or value.

---

### Prerequisites

- Basic C++ syntax: templates, iterators, `std::pair`, and functors.
- Understanding of hash functions and binary search trees at a conceptual level.
- Familiarity with `std::vector` (for flat containers).
- (For unordered containers) Knowledge of `std::hash` specializations.
- (For C++17 node handles) A compiler supporting C++17 or later.
- (For C++23 flat containers) A compiler supporting C++23 or later.

---

### Related Programming Areas

- **Database indexing**: B-trees and hash indexes are conceptually similar to ordered and unordered containers.
- **Cache-efficient programming**: Flat containers exploit CPU cache lines better than node-based containers.
- **Generic programming**: Associative containers are used as template parameters in generic algorithms.
- **Algorithm design**: Choosing between ordered, unordered, and flat containers affects algorithmic complexity.
- **Memory management**: Node-based containers allocate per element; flat containers allocate in bulk.

---

### Core Concepts / Features

1. **Node-Based Associative Containers** — Red-black tree mechanics under the hood of `std::set`, `std::map`, and their multi variants.
2. **Hash-Based Unordered Containers** — Bucket architectures, load factors, hash collisions, and custom hash functions.
3. **Modern Flat Containers (C++23)** — `std::flat_map`, `std::flat_set`, and their multi-variants.
4. **Node Manipulation** — Extracting and splicing nodes safely without copying keys or values using the `.extract()` API.

---

## 1. Node-Based Associative Containers

### Definitions

**Core Definition**
Node-based associative containers store each element in a separately allocated node, linked together in a balanced binary search tree (typically a red-black tree), providing guaranteed O(log n) search, insertion, and deletion.

**Technical Definition**
`std::set<Key, Compare, Allocator>`, `std::map<Key, T, Compare, Allocator>`, `std::multiset`, and `std::multimap` are ordered associative containers. The standard requires logarithmic complexity for search, removal, and insertion. In practice, implementations use red-black trees — a self-balancing binary search tree where each node carries a color bit (red or black) and the tree is kept balanced through rotations and recoloring. All standard library implementations (libstdc++, libc++, MSVC STL) use red-black trees with a sentinel/header node that doubles as the `end()` iterator.

**Beginner-Friendly Explanation**
A red-black tree is a binary tree that automatically keeps itself balanced. Each element is a "node" with pointers to its left and right children. The tree maintains five rules (listed below) that guarantee the height is never more than twice the minimum possible height, so searching always takes at most O(log n) steps. This is why `std::map` and `std::set` are reliable for lookups even with millions of elements.

---

### Sub-Feature: Red-Black Tree Mechanics

#### Definitions

**Core Definition**
A red-black tree is a self-balancing binary search tree where each node is colored red or black, and specific coloring rules ensure the tree remains approximately balanced.

**Technical Definition**
A red-black tree satisfies five properties:
1. Every node is either red or black.
2. The root is black.
3. All leaves (NIL/sentinel nodes) are black.
4. Every red node has two black children (no two consecutive red nodes).
5. Every path from a node to its descendant leaves contains the same number of black nodes.

These properties guarantee the longest path from root to leaf is at most twice the shortest path, keeping operations O(log n). Implementation-wise, the tree uses a sentinel "header" node: its `parent` pointer points to the root, its `left` pointer points to the smallest element, and its `right` pointer points to the largest element. The header node serves as the `end()` iterator. For a `map`, the payload is `std::pair<const Key, T>`; for a `set`, the payload is `Key`.

**Beginner-Friendly Explanation**
Think of a red-black tree as a tree where you paint some nodes red and some black. The coloring rules force the tree to stay "bushy" rather than becoming a long, thin chain. If you always ensure no two reds are adjacent and every path has the same number of blacks, you can't have one branch that's much longer than another. This is what makes lookups fast.

#### Purposes (all begin with "To")

- **To** provide guaranteed O(log n) search, insertion, and deletion for ordered data.
- **To** maintain elements in sorted order for in-order traversal.
- **To** offer stable iterators and references (except for erased elements).
- **To** enable range queries (e.g., all elements between `a` and `b`) efficiently.
- **To** provide the foundation for `std::set`, `std::map`, `std::multiset`, and `std::multimap`.

#### Syntax Rules and Structure

```cpp
// std::set — unique keys, key == value
template<class Key, class Compare = std::less<Key>,
         class Allocator = std::allocator<Key>>
class set;

// std::map — unique keys, key → mapped value
template<class Key, class T, class Compare = std::less<Key>,
         class Allocator = std::allocator<std::pair<const Key, T>>>
class map;

// std::multiset — equivalent keys allowed
template<class Key, class Compare = std::less<Key>,
         class Allocator = std::allocator<Key>>
class multiset;

// std::multimap — equivalent keys allowed
template<class Key, class T, class Compare = std::less<Key>,
         class Allocator = std::allocator<std::pair<const Key, T>>>
class multimap;
```

**Component breakdown:**
- `Key` — the key type.
- `T` — the mapped value type (for `map`/`multimap` only).
- `Compare` — the comparison function object (default `std::less<Key>`).
- `Allocator` — the allocator type.

**Common operations:**
- `m.insert(x)` — inserts element (O(log n)).
- `m.erase(key)` — removes element by key (O(log n)).
- `m.find(key)` — finds element (O(log n)).
- `m.lower_bound(key)` / `m.upper_bound(key)` — range queries.
- `m.count(key)` — counts elements with given key.

**Constraints and limitations:**
- Elements are stored in nodes; each node typically has 3 pointers + color bit + payload (~40–64 bytes overhead per element).
- Iterators are bidirectional, not random-access.
- No direct index-based access; elements are accessed by key or iterator.
- The specific tree balancing algorithm (red-black vs. AVL) is implementation-defined, though red-black is universal in practice.

#### Annotated Code Example

```cpp
// File: set_map_example.cpp
// Compile with: g++ -std=c++17 set_map_example.cpp -o set_map_example

#include <set>
#include <map>
#include <iostream>
#include <string>

int main() {
    // Step 1: Create a std::set (sorted unique keys)
    std::set<int> s = {50, 20, 40, 10, 30};

    // Step 2: Insert and check
    auto [it, inserted] = s.insert(25);
    std::cout << "Inserted 25: " << (inserted ? "yes" : "no") << std::endl;

    // Step 3: In-order traversal (always sorted)
    std::cout << "Set (sorted): ";
    for (int v : s) {
        std::cout << v << " ";
    }
    std::cout << std::endl;

    // Step 4: Lookup
    auto found = s.find(40);
    std::cout << "Found 40: " << (found != s.end() ? "yes" : "no") << std::endl;

    // Step 5: Create a std::map (key → value)
    std::map<std::string, int> ages;
    ages["Alice"] = 30;
    ages["Bob"] = 25;
    ages["Charlie"] = 35;

    // Step 6: Iterate in key order (alphabetical)
    std::cout << "Map (sorted by key):" << std::endl;
    for (const auto& [name, age] : ages) {
        std::cout << "  " << name << ": " << age << std::endl;
    }

    // Step 7: Range query with lower_bound / upper_bound
    auto lo = ages.lower_bound("B");
    auto hi = ages.upper_bound("C");
    std::cout << "Names in [B, C): ";
    for (auto it2 = lo; it2 != hi; ++it2) {
        std::cout << it2->first << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
Inserted 25: yes
Set (sorted): 10 20 25 30 40 50
Found 40: yes
Map (sorted by key):
  Alice: 30
  Bob: 25
  Charlie: 35
Names in [B, C): Bob
```

**Why this output:** The `std::set` stores elements in sorted order (10, 20, 25, 30, 40, 50). `insert(25)` succeeds and returns `{iterator, true}`. The `std::map` iterates in alphabetical order of keys because it is a red-black tree sorted by `std::less<std::string>`. `lower_bound("B")` finds the first key not less than "B" (Bob), and `upper_bound("C")` finds the first key greater than "C" (Charlie), so the range `[B, C)` contains only Bob.

#### Real-World Cases with Explanation

**Case 1 — Configuration management**: A `std::map<std::string, std::string>` stores configuration key-value pairs, automatically sorted for deterministic iteration and fast lookup by key.

**Case 2 — Symbol tables in compilers**: A `std::map<std::string, SymbolInfo>` maps identifier names to their semantic information, providing O(log n) lookup during compilation.

**Case 3 — Range queries in databases**: Ordered containers support `lower_bound`/`upper_bound`, enabling efficient "find all records with key between X and Y" queries.

**Case 4 — Ordered output**: When iterating a `std::map`, elements come out in sorted order without additional sorting, useful for reports and serialization.

---

## 2. Hash-Based Unordered Containers

### Definitions

**Core Definition**
Unordered associative containers store elements in buckets based on hash values, providing average O(1) access. They sacrifice ordering for speed and do not maintain any sorted sequence.

**Technical Definition**
`std::unordered_set`, `std::unordered_map`, `std::unordered_multiset`, and `std::unordered_multimap` are parameterized by `Key`, a `Hash` function object, and an equality predicate `Pred`. Elements are organized into buckets; keys with the same hash code appear in the same bucket (separate chaining). The container automatically increases the number of buckets as elements are added to keep the average elements per bucket below `max_load_factor()`. A hash function takes a `Key` and returns a `std::size_t` in `[0, max_size_t)`. If two keys are equal, the hash function must return the same value for both.

**Beginner-Friendly Explanation**
An unordered container is like a set of filing cabinets. Each drawer (bucket) is labeled with a number. To file a document (element), you compute a "magic number" from its name (hash function) and put it in the drawer with that number. To find a document, you compute the same magic number and look only in that drawer. If two documents have the same magic number, they share a drawer (collision) and you check each one. The "load factor" is how full the drawers are on average — when they get too full, the container adds more drawers (rehash) to keep lookups fast.

---

### Sub-Feature: Bucket Architecture and Load Factor

#### Definitions

**Core Definition**
A bucket is a slot in the hash table that holds a chain of elements whose keys hash to the same bucket index. The load factor is the average number of elements per bucket.

**Technical Definition**
`load_factor() = size() / bucket_count()`. The container stores a `max_load_factor()` (default 1.0). When an insertion would cause `load_factor()` to exceed `max_load_factor()`, the container triggers a rehash: it allocates a larger bucket array (typically ~2× the current size), recomputes the bucket index for every element, and redistributes them. Rehashing invalidates iterators but does not invalidate pointers or references to elements.

**Beginner-Friendly Explanation**
The load factor is like the average number of people per room. If rooms are too crowded (high load factor), finding a specific person takes longer. The container's solution is to add more rooms (rehash) when the average occupancy exceeds a threshold. You can prevent unnecessary rehashes by calling `reserve(n)` if you know how many elements you'll insert.

#### Purposes (all begin with "To")

- **To** provide average O(1) lookup, insertion, and deletion for unique keys.
- **To** avoid the O(log n) overhead of tree-based containers when ordering is not needed.
- **To** allow custom hash functions for user-defined key types.
- **To** support bucket-level introspection for performance tuning.
- **To** offer `reserve()` and `rehash()` for controlling memory/performance trade-offs.

#### Syntax Rules and Structure

```cpp
// std::unordered_map
template<class Key, class T,
         class Hash = std::hash<Key>,
         class KeyEqual = std::equal_to<Key>,
         class Allocator = std::allocator<std::pair<const Key, T>>>
class unordered_map;
```

**Component breakdown:**
- `Key` — key type.
- `T` — mapped value type.
- `Hash` — hash function object (default `std::hash<Key>`).
- `KeyEqual` — equality predicate (default `std::equal_to<Key>`).
- `Allocator` — allocator type.

**Common operations:**
- `um.insert({k, v})` — insert (average O(1)).
- `um.find(k)` — find (average O(1)).
- `um.erase(k)` — erase (average O(1)).
- `um.reserve(n)` — pre-allocate buckets for n elements.
- `um.rehash(n)` — set bucket count to at least n.
- `um.load_factor()` / `um.max_load_factor()` — query load factor.
- `um.bucket_count()` — number of buckets.
- `um.bucket(k)` — which bucket key k hashes to.

**Constraints and limitations:**
- No ordering; iteration order is unspecified and may change after rehash.
- Worst-case lookup is O(n) if all keys collide.
- The default `std::hash` for integers is often the identity function, making sequential integers vulnerable to adversarial collision attacks.
- Custom key types require `std::hash` specialization or a custom hash functor.
- Rehashing invalidates all iterators (but not pointers/references to elements).

#### Annotated Code Example

```cpp
// File: unordered_map_example.cpp
// Compile with: g++ -std=c++17 unordered_map_example.cpp -o unordered_map_example

#include <unordered_map>
#include <iostream>
#include <string>

int main() {
    // Step 1: Create an unordered_map
    std::unordered_map<std::string, int> um;

    // Step 2: Reserve to avoid rehashing
    um.reserve(4);
    std::cout << "Initial bucket_count: " << um.bucket_count() << std::endl;

    // Step 3: Insert elements
    um["apple"] = 1;
    um["banana"] = 2;
    um["cherry"] = 3;
    um["date"] = 4;

    // Step 4: Query load factor and bucket info
    std::cout << "Size: " << um.size() << std::endl;
    std::cout << "Bucket count: " << um.bucket_count() << std::endl;
    std::cout << "Load factor: " << um.load_factor() << std::endl;
    std::cout << "Max load factor: " << um.max_load_factor() << std::endl;

    // Step 5: Lookup
    auto it = um.find("cherry");
    if (it != um.end()) {
        std::cout << "Found cherry: " << it->second << std::endl;
    }

    // Step 6: Show which bucket a key belongs to
    std::cout << "Bucket for 'apple': " << um.bucket("apple") << std::endl;

    // Step 7: Iterate (order is unspecified)
    std::cout << "Elements:" << std::endl;
    for (const auto& [key, val] : um) {
        std::cout << "  " << key << " -> " << val
                  << " (bucket " << um.bucket(key) << ")" << std::endl;
    }

    return 0;
}
```

**Expected Output (GCC/libstdc++):**
```
Initial bucket_count: 11
Size: 4
Bucket count: 11
Load factor: 0.363636
Max load factor: 1
Found cherry: 3
Bucket for 'apple': 7
Elements:
  date -> 4 (bucket 2)
  banana -> 2 (bucket 5)
  cherry -> 3 (bucket 6)
  apple -> 1 (bucket 7)
```

**Why this output:** `reserve(4)` ensures at least 4 buckets are available (libstdc++ chooses 11). After inserting 4 elements, the load factor is 4/11 ≈ 0.36, well below the max of 1.0. The iteration order is implementation-specific and depends on hash values and bucket indices.

---

### Sub-Feature: Custom Hash Functions

#### Definitions

**Core Definition**
A custom hash function is a user-provided function object that computes a hash value for a user-defined key type, enabling its use with unordered containers.

**Technical Definition**
The `Hash` template parameter must be a function object (or function pointer) callable as `Hash(key)` returning `std::size_t`. The standard library provides `std::hash` specializations for fundamental types, `std::string`, and smart pointers. For user-defined types, you must either specialize `std::hash<MyType>` or provide a custom functor as the `Hash` template argument. If `Pred(a, b)` returns true, then `Hash(a)` must equal `Hash(b)`.

**Beginner-Friendly Explanation**
If you want to use your own class as a key in an unordered_map, the library doesn't know how to "hash" it. You need to write a function that takes your object and produces a number. A good hash function produces different numbers for different objects (to avoid collisions) and the same number for equal objects (required by the standard).

#### Purposes (all begin with "To")

- **To** enable unordered containers to work with user-defined key types.
- **To** control the quality of hashing to minimize collisions.
- **To** avoid the identity hash for integers that makes adversarial attacks possible.
- **To** combine multiple fields of a composite key into a single hash value.
- **To** optimize hash computation for domain-specific key patterns.

#### Syntax Rules and Structure

```cpp
// Option 1: Specialize std::hash
namespace std {
    template<> struct hash<MyType> {
        std::size_t operator()(const MyType& obj) const noexcept {
            return /* compute hash */;
        }
    };
}

// Option 2: Custom functor as template argument
struct MyHash {
    std::size_t operator()(const MyType& obj) const noexcept {
        return /* compute hash */;
    }
};
std::unordered_map<MyType, Value, MyHash> um;
```

**Constraints and limitations:**
- A hash function must never throw (the container assumes it is nothrow-callable in many operations).
- A hash function must be deterministic: for the same input, it must return the same output every time.
- Equal keys must produce equal hashes; unequal keys *should* produce different hashes (but collisions are allowed).
- The hash function must not modify the object.
- Specializing `std::hash` for user types must be done in the `std` namespace and is allowed for user-defined types only.

#### Annotated Code Example

```cpp
// File: custom_hash_example.cpp
// Compile with: g++ -std=c++17 custom_hash_example.cpp -o custom_hash_example

#include <unordered_map>
#include <iostream>
#include <string>

// Step 1: A user-defined key type
struct Point {
    int x, y;
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

// Step 2: Custom hash functor
struct PointHash {
    std::size_t operator()(const Point& p) const noexcept {
        // Combine x and y with a mixing constant to reduce collisions
        std::size_t hx = std::hash<int>{}(p.x);
        std::size_t hy = std::hash<int>{}(p.y);
        return hx ^ (hy << 1);  // simple combine
    }
};

// Step 3: Required: specialize std::hash OR use functor as template arg
// Here we use the functor as a template argument.

int main() {
    // Step 4: Create unordered_map with custom hash
    std::unordered_map<Point, std::string, PointHash> grid;

    // Step 5: Insert points
    grid[{0, 0}] = "origin";
    grid[{1, 2}] = "a";
    grid[{2, 1}] = "b";
    grid[{3, 4}] = "c";

    // Step 6: Lookup
    auto it = grid.find({1, 2});
    if (it != grid.end()) {
        std::cout << "Found ({1,2}): " << it->second << std::endl;
    }

    // Step 7: Show buckets
    std::cout << "Bucket count: " << grid.bucket_count() << std::endl;
    for (const auto& [pt, label] : grid) {
        std::cout << "  (" << pt.x << "," << pt.y << ") -> " << label
                  << " (bucket " << grid.bucket(pt) << ")" << std::endl;
    }

    return 0;
}
```

**Expected Output (GCC/libstdc++):**
```
Found ({1,2}): a
Bucket count: 13
  (1,2) -> a (bucket 5)
  (0,0) -> origin (bucket 0)
  (3,4) -> c (bucket 9)
  (2,1) -> b (bucket 3)
```

**Why this output:** The `PointHash` functor combines `x` and `y` into a single hash value. The container uses this hash to assign each point to a bucket. The iteration order reflects the bucket indices, which depend on the hash values and the bucket count (13).

#### Real-World Cases with Explanation

**Case 1 — Spatial hashing**: A game engine stores entities in an `unordered_map<GridCoord, EntityList>` using a hash that combines grid coordinates, enabling fast spatial queries.

**Case 2 — Network address lookup**: A firewall uses `unordered_map<IPv4Address, Rule>` with a custom hash that processes the 32-bit address for fast packet classification.

**Case 3 — Composite keys**: A database index uses `unordered_map<std::pair<UserId, Timestamp>, Record>` with a hash that combines both fields, allowing O(1) lookup by the composite key.

**Case 4 — Adversarial hardening**: When keys come from untrusted input, a custom hash (e.g., SipHash or a salted hash) prevents denial-of-service attacks that exploit hash collisions.

---

## 3. Modern Flat Containers (C++23)

### Definitions

**Core Definition**
Flat containers are associative containers (C++23) backed by sorted contiguous storage, typically `std::vector`, offering the same interface as `std::map`/`std::set` with superior cache locality.

**Technical Definition**
`std::flat_map`, `std::flat_set`, `std::flat_multimap`, and `std::flat_multiset` (headers `<flat_map>`, `<flat_set>`) are container adaptors. `flat_map` maintains two parallel sorted vectors: one for keys and one for mapped values. `flat_set` maintains a single sorted vector of keys. Lookups use binary search (O(log n)) on contiguous memory; insertions and erasures are O(n) due to element shifting. Construction supports `std::sorted_unique` and `std::sorted_equivalent` tags to skip internal sorting when the input is already ordered. The `keys()` and `values()` accessors provide direct access to the underlying containers.

**Beginner-Friendly Explanation**
A flat_map is like a sorted spreadsheet. All keys are in one column, all values in the next. To find a key, you do a binary search on the key column — very fast because the data is packed tightly in memory (cache-friendly). But inserting a new row means shifting all rows below it down — slow if you insert in the middle. So flat_map is perfect for "build once, read many times" scenarios.

---

### Sub-Feature: `std::flat_map` and `std::flat_set`

#### Definitions

**Core Definition**
`std::flat_map` is a sorted associative container backed by two parallel sorted vectors (keys and values). `std::flat_set` is backed by a single sorted vector of keys.

**Technical Definition**
`template<class Key, class T, class Compare = std::less<Key>, class KeyContainer = std::vector<Key>, class MappedContainer = std::vector<T>> class flat_map;`. `template<class Key, class Compare = std::less<Key>, class KeyContainer = std::vector<Key>> class flat_set;`. Both provide random-access iterators (unlike `std::map`/`std::set`, which provide bidirectional iterators). The `keys()` member returns a reference to the key container; `values()` returns a reference to the mapped container (for `flat_map`).

**Beginner-Friendly Explanation**
`flat_map` is like a dictionary where the pages are actually just two long lists glued together — one list of words, one list of definitions, perfectly aligned. You can flip to any word quickly (binary search), but adding a new word in the middle means re-gluing everything after it.

#### Purposes (all begin with "To")

- **To** provide cache-friendly associative lookup for read-heavy workloads.
- **To** reduce memory overhead by eliminating per-node pointers.
- **To** offer the same interface as `std::map`/`std::set` for easy migration.
- **To** enable sorted iteration over contiguous memory.
- **To** support construction from pre-sorted data without redundant sorting.

#### Syntax Rules and Structure

```cpp
// std::flat_map
template<class Key, class T, class Compare = std::less<Key>,
         class KeyContainer = std::vector<Key>,
         class MappedContainer = std::vector<T>>
class flat_map;

// std::flat_set
template<class Key, class Compare = std::less<Key>,
         class KeyContainer = std::vector<Key>>
class flat_set;
```

**Common operations:**
- `fm.find(key)` — binary search on keys vector (O(log n)).
- `fm.insert({k, v})` — insert and shift elements (O(n)).
- `fm.erase(key)` — erase and shift elements (O(n)).
- `fm.keys()` — access the underlying keys container.
- `fm.values()` — access the underlying values container.

**Construction tags:**
- `std::sorted_unique` — keys are already sorted and unique; skip sort.
- `std::sorted_equivalent` — keys are already sorted; equivalent keys allowed (multi variants).

**Constraints and limitations:**
- Insert/erase are O(n) due to element shifting — not suitable for write-heavy workloads.
- All iterators are invalidated on insert or erase.
- Pointers and references to elements remain valid only until the next modification.
- Requires C++23 or later.
- No stable iterator guarantee (unlike `std::map`).

#### Annotated Code Example

```cpp
// File: flat_map_example.cpp
// Compile with: g++ -std=c++23 flat_map_example.cpp -o flat_map_example

#include <flat_map>
#include <iostream>
#include <vector>
#include <string>

int main() {
    // Step 1: Construct with sorted_unique tag (no redundant sort)
    std::vector<int> keys = {1, 3, 5, 7, 9};
    std::vector<std::string> vals = {"one", "three", "five", "seven", "nine"};

    std::flat_map<int, std::string> fm(
        std::sorted_unique,
        std::move(keys),
        std::move(vals)
    );

    // Step 2: Lookup (binary search on contiguous memory)
    auto it = fm.find(5);
    if (it != fm.end()) {
        std::cout << "Found 5: " << it->second << std::endl;
    }

    // Step 3: Insert (O(n) — shifts elements)
    fm.insert({4, "four"});
    std::cout << "After insert 4:" << std::endl;
    for (const auto& [k, v] : fm) {
        std::cout << "  " << k << " -> " << v << std::endl;
    }

    // Step 4: Access underlying containers
    std::cout << "Keys vector size: " << fm.keys().size() << std::endl;
    std::cout << "Values vector size: " << fm.values().size() << std::endl;

    // Step 5: Random access via iterators
    auto begin = fm.begin();
    std::cout << "First element: " << begin->first << " -> "
              << begin->second << std::endl;

    return 0;
}
```

**Expected Output:**
```
Found 5: five
After insert 4:
  1 -> one
  3 -> three
  4 -> four
  5 -> five
  7 -> seven
  9 -> nine
Keys vector size: 6
Values vector size: 6
First element: 1 -> one
```

**Why this output:** The `sorted_unique` tag tells the constructor that the input is already sorted, so no internal sort is performed. `find(5)` does a binary search on the keys vector. `insert({4, "four"})` shifts elements to maintain sorted order. `keys()` and `values()` return references to the underlying vectors.

#### Real-World Cases with Explanation

**Case 1 — Configuration tables**: A `flat_map<std::string, ConfigValue>` is built once at startup and read thousands of times during execution — ideal for flat_map's read-heavy profile.

**Case 2 — Translation tables**: A `flat_map<int, std::string>` mapping error codes to messages, built once and queried repeatedly.

**Case 3 — Small to medium lookup tables**: For maps with < ~100K elements, flat_map often outperforms std::map due to cache effects, even with O(n) insertion.

**Case 4 — Memory-constrained systems**: Flat containers eliminate per-node pointer overhead, reducing memory usage by 3–4× compared to node-based containers.

---

## 4. Node Manipulation — `.extract()` and Splicing

### Definitions

**Core Definition**
`std::map::extract` (C++17) is a member function that unlinks a node from an associative container and returns a `node_handle` that owns the node, allowing the node to be modified and re-inserted without copying or moving the stored element.

**Technical Definition**
`node_type extract(const_iterator pos)` and `node_type extract(const Key& k)` are available on all node-based associative containers (`map`, `set`, `multimap`, `multiset`, and their `unordered_` variants). The returned `node_handle` is a move-only type that owns the node's memory. It provides `key()` (non-const for `map`/`set`) and `mapped()` (for `map`) accessors. When a `node_handle` is inserted into a compatible container, the node is spliced in via pointer adjustments only — no allocation, no copy, no move, no constructor or destructor call for the stored value. `extract` is the only standard way to modify a `map` key in-place.

**Beginner-Friendly Explanation**
Normally, if you want to change a key in a `std::map`, you have to erase the element and insert a new one, which copies or moves the value and allocates a new node. `extract()` lets you "unhook" a node from the tree, change its key directly, and "re-hook" it into the same or another container — all without touching the stored value. It's like taking a book off a shelf, changing its cover, and putting it back without rewriting the pages.

---

### Sub-Feature: `.extract()` and `node_handle`

#### Definitions

**Core Definition**
A `node_handle` is a move-only object that owns a single node extracted from an associative container, providing access to the stored key and value and allowing re-insertion into a compatible container.

**Technical Definition**
`node_type` is the `node_handle` type for each container (e.g., `std::map<int, std::string>::node_type`). It is move-constructible and move-assignable but not copyable. It provides:
- `explicit operator bool()` — true if non-empty.
- `bool empty() const` — true if the handle owns no node.
- `key_type& key() const` — mutable reference to the key (only for `map`/`set`; not for `multimap`/`multiset`? Actually, `key()` is available and non-const for `map` and `set`; for `multimap` and `multiset`, `key()` is const because equivalent keys must maintain relative order).
- `mapped_type& mapped() const` — reference to the mapped value (for `map`/`unordered_map`).
- `allocator_type get_allocator() const` — the allocator of the extracted node.

**Beginner-Friendly Explanation**
A `node_handle` is like a "claim ticket" for a node. While you hold the ticket, the node belongs to you — it's not in any container. You can change its key (for `map`), inspect its value, and then insert it into a container. The container takes ownership of the node back.

#### Purposes (all begin with "To")

- **To** modify a `map` key in-place without erasing and re-inserting.
- **To** transfer nodes between containers without copying or moving the stored value.
- **To** avoid heap allocation when moving elements between compatible containers.
- **To** provide exception safety: if re-insertion fails, the node is still owned by the handle.
- **To** enable "splicing" — O(1) (for `map`/`set`) or O(1) (for `unordered`, amortized) transfer of nodes.

#### Syntax Rules and Structure

```cpp
// Extract by iterator (always succeeds)
node_type extract(const_iterator pos);

// Extract by key (returns empty handle if not found)
node_type extract(const Key& k);

// For map: modify key and re-insert
auto nh = m.extract(key);
nh.key() = new_key;
m.insert(std::move(nh));
```

**Constraints and limitations:**
- `extract` invalidates only iterators to the extracted element; pointers and references to the element remain valid while owned by the handle (but accessing them is UB if the element is accessed through the handle).
- `node_handle` is move-only; it cannot be copied.
- Re-insertion may fail (e.g., if the key already exists in the target container); the handle remains valid and owns the node.
- The node's allocator must match the target container's allocator for splice to succeed.
- `extract` is not available on flat containers (C++23) because they are not node-based.

#### Annotated Code Example

```cpp
// File: extract_example.cpp
// Compile with: g++ -std=c++17 extract_example.cpp -o extract_example

#include <map>
#include <iostream>
#include <string>

int main() {
    // Step 1: Create a map
    std::map<int, std::string> m = {
        {1, "one"}, {2, "two"}, {3, "three"}
    };

    // Step 2: Extract by key — returns node_handle
    auto nh = m.extract(2);

    // Step 3: Check the handle
    if (!nh.empty()) {
        std::cout << "Extracted key: " << nh.key()
                  << ", value: " << nh.mapped() << std::endl;
    }

    // Step 4: Modify the key (only possible with extract!)
    nh.key() = 20;

    // Step 5: Re-insert with new key
    m.insert(std::move(nh));

    // Step 6: Show the map
    std::cout << "Map after key change:" << std::endl;
    for (const auto& [k, v] : m) {
        std::cout << "  " << k << " -> " << v << std::endl;
    }

    // Step 7: Transfer between maps with merge()
    std::map<int, std::string> a = {{10, "ten"}, {30, "thirty"}};
    std::map<int, std::string> b = {{20, "twenty"}, {40, "forty"}};
    a.merge(b);

    std::cout << "After merge:" << std::endl;
    std::cout << "  a: ";
    for (const auto& [k, v] : a) std::cout << "(" << k << "," << v << ") ";
    std::cout << std::endl;
    std::cout << "  b: ";
    for (const auto& [k, v] : b) std::cout << "(" << k << "," << v << ") ";
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
Extracted key: 2, value: two
Map after key change:
  1 -> one
  3 -> three
  20 -> two
After merge:
  a: (10,ten) (20,twenty) (30,thirty) (40,forty)
  b:
```

**Why this output:** `extract(2)` removes the node containing `{2, "two"}` from the map and returns a `node_handle`. `nh.key() = 20` changes the key directly. `m.insert(std::move(nh))` re-inserts the node with the new key. `merge(b)` transfers all nodes from `b` to `a` that don't have equivalent keys in `a` (none conflict here), leaving `b` empty.

#### Real-World Cases with Explanation

**Case 1 — Updating map keys**: A configuration manager needs to rename a key without losing the associated value. `extract` allows changing the key in-place.

**Case 2 — Moving nodes between containers**: A cache eviction policy moves entries from a "hot" map to a "cold" map without copying the potentially large values.

**Case 3 — Exception-safe key modification**: The extract-modify-reinsert pattern is exception-safe: while the node is held in the handle, the data is fully owned and cannot be lost even if re-insertion throws.

**Case 4 — Merging maps**: `merge()` transfers all nodes from one container to another, skipping keys that already exist in the target. This is useful for combining configuration or cache maps.

**Case 5 — Multimap slicing**: Extracting a range of elements from a `multimap` and inserting them into another `multimap` without copying.

---

## Summary Table

| Container | Storage | Lookup | Insert | Erase | Iterators | Cache Locality | Memory Overhead |
|---|---|---|---|---|---|---|---|
| `std::set` / `std::map` | Red-black tree (nodes) | O(log n) | O(log n) | O(log n) | Bidirectional | Poor | ~3–4 pointers/node |
| `std::unordered_set` / `std::unordered_map` | Hash table (buckets) | O(1) avg, O(n) worst | O(1) avg | O(1) avg | Forward | Moderate | ~1–2 pointers/node |
| `std::flat_set` / `std::flat_map` | Sorted vector | O(log n) | O(n) | O(n) | Random-access | Excellent | Near-zero |
| `std::multiset` / `std::multimap` | Red-black tree | O(log n) | O(log n) | O(log n) | Bidirectional | Poor | ~3–4 pointers/node |
| `std::unordered_multiset` / `std::unordered_multimap` | Hash table | O(1) avg | O(1) avg | O(1) avg | Forward | Moderate | ~1–2 pointers/node |
| `std::flat_multiset` / `std::flat_multimap` | Sorted vector | O(log n) | O(n) | O(n) | Random-access | Excellent | Near-zero |

---

## References

- std::set — cppreference.com - https://en.cppreference.com/w/cpp/container/set
- std::map — cppreference.com - https://en.cppreference.com/w/cpp/container/map
- std::unordered_map — cppreference.com - https://en.cppreference.com/w/cpp/container/unordered_map
- std::flat_map — cppreference.com - https://en.cppreference.com/w/cpp/container/flat_map
- std::flat_set — cppreference.com - https://en.cppreference.com/w/cpp/container/flat_set
- Node handle — cppreference.com - https://en.cppreference.com/w/cpp/container/node_handle
- std::map::extract — cppreference.com - https://en.cppreference.com/w/cpp/container/map/extract
- Inside STL: The map, set, multimap, and multiset — Microsoft DevBlogs - https://devblogs.microsoft.com/oldnewthing/20230807-00/?p=108562
- Unordered associative containers — cppreference.com - https://en.cppreference.com/w/cpp/named_req/UnorderedAssociativeContainer
- std::hash — cppreference.com - https://en.cppreference.com/w/cpp/utility/hash
- C++23 flat_map and flat_set — P0429R9 - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p0429r9.pdf
- Splicing Maps and Sets (Revision 1) — N3645 - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3645.pdf
- Understand unordered_map load factor, rehashing, and bucket management — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp
- Use node-based containers with merge and extract (C++17) — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp