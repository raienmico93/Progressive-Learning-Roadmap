# C++ Sets and Maps — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`std::set`, `std::multiset`, `std::map`, and `std::multimap` are ordered associative containers in the C++ Standard Library that store elements in a sorted order determined by a key comparison function, providing logarithmic-time search, insertion, and removal.

**Technical Definition**
These four class templates satisfy the *AssociativeContainer* named requirement and are declared in the headers `<set>` and `<map>`. They are typically implemented as self-balancing binary search trees—most commonly red–black trees—although the C++ standard does not mandate a specific tree structure, only the complexity guarantees. The containers differ along two axes: (1) whether each element consists of a key alone (`set`/`multiset`) or a key–value pair (`map`/`multimap`), and (2) whether keys must be unique (`set`/`map`) or may be duplicated (`multiset`/`multimap`).

**Beginner-Friendly Explanation**
Think of an ordered associative container as a self-sorting dictionary. You insert items, and the container automatically keeps them in order based on their keys. `set` is like a sorted list of unique items; `multiset` allows duplicates. `map` is like a dictionary where each word (key) has a definition (value); `multimap` is a dictionary that allows multiple definitions per word.

### Key Characteristics

- **Sorted order** – elements are always iterated in non-descending order of keys according to the `Compare` function (default `std::less<Key>`).
- **Logarithmic complexity** – search, insertion, and removal are O(log n).
- **Unique vs. equivalent keys** – `set` and `map` allow at most one element per key; `multiset` and `multimap` allow multiple equivalent keys.
- **Iterator stability** – insertion and erasure do not invalidate iterators or references to other elements.
- **No random access** – iterators are bidirectional, not random-access.
- **Red–black tree backing** – the standard does not mandate a specific tree, but red–black trees are the near-universal choice because they satisfy the complexity requirements with a good balance of insertion/erasure performance.

### Prerequisites

- Basic C++ syntax (variables, types, control flow).
- Understanding of templates and class templates.
- Familiarity with iterators and the range-based `for` loop.
- Awareness of Big-O notation and algorithmic complexity.
- Basic knowledge of binary search trees (helpful but not required).

### Related Programming Areas

- **STL algorithms** – many algorithms (`std::find`, `std::for_each`) work with associative container iterators.
- **Unordered associative containers** – `std::unordered_set` and `std::unordered_map` provide hash-based alternatives without ordering.
- **Generic programming** – these containers are templates parameterised on key type, comparison function, and allocator.
- **Database indexing** – the sorted structure mirrors the concept of a B-tree or red–black tree index.
- **Text processing** – frequency counting, dictionary implementations, symbol tables.

### Core Concepts / Features

1. `std::set`
2. `std::multiset`
3. `std::map`
4. `std::multimap`
5. Ordered containers and the red–black tree
6. Set operations
7. Map operations
8. Set best practices
9. Map best practices

---

## 1. `std::set`

### Definitions

**Core Definition**
`std::set` is an associative container that contains a sorted set of unique objects of type `Key`.

**Technical Definition**
The template is declared in `<set>` as:
```cpp
template<class Key, class Compare = std::less<Key>, class Allocator = std::allocator<Key>>
class set;
```
It satisfies *Container*, *AllocatorAwareContainer*, *AssociativeContainer*, and *ReversibleContainer*. Search, insertion, and removal have logarithmic complexity. Sets are usually implemented as red–black trees. Uniqueness is determined by the equivalence relation: two keys `a` and `b` are equivalent if `!comp(a, b) && !comp(b, a)`.

**Beginner-Friendly Explanation**
`std::set` is a sorted collection of unique items. If you try to insert a duplicate, the set simply ignores it.

### Purposes

- To maintain a sorted collection of unique keys.
- To provide O(log n) membership testing.
- To support ordered range queries via `lower_bound` and `upper_bound`.
- To eliminate duplicates automatically.
- To serve as a foundation for mathematical set operations.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <set>
std::set<Key> s;                        // empty
std::set<Key> s = {a, b, c};           // initializer-list
std::set<Key> s(other);                 // copy
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Key` | The type of elements stored. |
| `Compare` | Strict weak ordering function object (default `std::less<Key>`). |
| `Allocator` | Optional allocator type. |

**Member Function Complexity**

| Operation | Complexity |
|-----------|------------|
| `insert` | O(log n) |
| `erase` | O(log n) |
| `find` | O(log n) |
| `count` | O(log n) (returns 0 or 1) |
| `lower_bound` / `upper_bound` | O(log n) |
| `begin` / `end` | O(1) |

**Syntax Rules**
- Keys are `const` through iterators; elements cannot be modified in place.
- The comparison function must impose a strict weak ordering.
- `set` does not provide `operator[]` or `at()`; access is via iterators.

**Constraints and Limitations**
- No random access; iteration is bidirectional.
- Insertion may invalidate iterators, but references to existing elements remain valid.
- Requires a valid comparison function; an invalid ordering results in undefined behaviour.

### Annotated Code Examples

**Example 1: Uniqueness and sorted order**

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> s = {3, 1, 2, 3, 1};    // duplicates ignored
    for (int x : s) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`set` stores only unique keys and iterates them in ascending order.

**Example 2: Range queries with `lower_bound` and `upper_bound`**

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> s = {10, 20, 30, 40, 50};
    auto lo = s.lower_bound(25);           // first element >= 25
    auto hi = s.upper_bound(40);           // first element > 40

    for (auto it = lo; it != hi; ++it)
        std::cout << *it << ' ';
}
```

**Expected Output**
```
30 40
```

**Why this output?**  
`lower_bound(25)` points to `30`; `upper_bound(40)` points to `50`. The range `[30, 40]` is printed.

**Example 3: Erasing by key**

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> s = {1, 2, 3, 4, 5};
    s.erase(3);                            // O(log n)
    for (int x : s) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 4 5
```

**Why this output?**  
`erase(3)` removes the element with value `3`.

### Real-World Cases

- **Deduplication** – storing unique identifiers or tokens.
- **Sorted iteration** – processing elements in order without an explicit sort.
- **Membership testing** – checking whether a value belongs to a set.
- **Mathematical sets** – union, intersection, difference via STL algorithms.

---

## 2. `std::multiset`

### Definitions

**Core Definition**
`std::multiset` is an associative container that contains a sorted set of objects of type `Key`, allowing multiple keys with equivalent values.

**Technical Definition**
Declared in `<set>`:
```cpp
template<class Key, class Compare = std::less<Key>, class Allocator = std::allocator<Key>>
class multiset;
```
Unlike `set`, multiple elements with equivalent keys are permitted. The order of equivalent elements is the order of insertion and does not change (since C++11). All member functions are `constexpr` since C++26.

**Beginner-Friendly Explanation**
`std::multiset` is a sorted bag: it keeps duplicates and orders them alongside everything else.

### Purposes

- To maintain a sorted collection that permits duplicate keys.
- To count occurrences of values without a separate counter.
- To support range queries that include multiple equivalent elements.
- To model multisets in mathematics.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <set>
std::multiset<Key> ms;
```

**Component Breakdown**

| Operation | Complexity |
|-----------|------------|
| `insert` | O(log n) |
| `erase` | O(log n) |
| `find` | O(log n) (returns any equivalent element) |
| `count` | O(log n + k), where k is the number of matches |
| `equal_range` | O(log n) |

**Syntax Rules**
- `count(key)` returns the number of elements equivalent to `key`.
- `equal_range(key)` returns a pair of iterators delimiting all equivalent elements.
- There is no `operator[]`; access is via iterators.

**Constraints and Limitations**
- Duplicates increase memory usage compared with `set`.
- `erase(key)` removes all elements with that key; use `erase(iterator)` to remove one.

### Annotated Code Examples

**Example 1: Duplicates preserved**

```cpp
#include <set>
#include <iostream>

int main() {
    std::multiset<int> ms = {3, 1, 2, 3, 1};
    for (int x : ms) std::cout << x << ' ';
}
```

**Expected Output**
```
1 1 2 3 3
```

**Why this output?**  
`multiset` keeps all duplicate keys, sorted.

**Example 2: Counting occurrences**

```cpp
#include <set>
#include <iostream>

int main() {
    std::multiset<int> ms = {1, 2, 2, 3, 2};
    std::cout << ms.count(2) << '\n';      // 3
}
```

**Expected Output**
```
3
```

**Why this output?**  
`count(2)` returns the number of elements equivalent to `2`.

### Real-World Cases

- **Frequency counting** – tallying how many times each value appears.
- **Event logs** – storing multiple events with the same timestamp.
- **Inventory systems** – tracking multiple items with the same identifier.

---

## 3. `std::map`

### Definitions

**Core Definition**
`std::map` is a sorted associative container that contains key–value pairs with unique keys.

**Technical Definition**
Declared in `<map>`:
```cpp
template<class Key, class T, class Compare = std::less<Key>,
         class Allocator = std::allocator<std::pair<const Key, T>>>
class map;
```
Keys are sorted by `Compare`. Search, removal, and insertion have logarithmic complexity. Maps are usually implemented as red–black trees. Iteration follows ascending key order.

**Beginner-Friendly Explanation**
`std::map` is a dictionary that keeps its entries sorted by key. You can look up a value by its key, and iteration always visits keys in order.

### Purposes

- To associate values with unique keys.
- To provide O(log n) key-based lookup.
- To maintain sorted order by key.
- To support range queries over keys.
- To implement frequency maps and symbol tables.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <map>
std::map<Key, T> m;
m[key] = value;                          // inserts or updates
m.insert({key, value});                  // inserts only if key absent
m.at(key);                               // throws if key absent
m.find(key);                             // returns iterator or end()
```

**Component Breakdown**

| Operation | Complexity | Notes |
|-----------|------------|-------|
| `operator[]` | O(log n) | Inserts a default-constructed value if key is absent. |
| `at` | O(log n) | Throws `std::out_of_range` if key is absent. |
| `insert` | O(log n) | Does not overwrite existing key. |
| `erase` | O(log n) | |
| `find` | O(log n) | |
| `lower_bound` / `upper_bound` | O(log n) | |

**Syntax Rules**
- `operator[]` requires `T` to be default-constructible.
- Keys are `const`; values can be modified through iterators.
- `insert` returns a pair of iterator and `bool` indicating whether insertion occurred.

**Constraints and Limitations**
- `operator[]` silently inserts a default-constructed value when the key is absent; this is a common source of bugs when a lookup was intended.
- Iterators are bidirectional, not random-access.
- The comparison function must impose a strict weak ordering.

### Annotated Code Examples

**Example 1: Dictionary basics**

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::map<std::string, int> ages;
    ages["Alice"] = 30;
    ages["Bob"] = 25;
    ages["Alice"] = 31;                    // updates existing key

    for (const auto& [name, age] : ages)
        std::cout << name << ": " << age << '\n';
}
```

**Expected Output**
```
Alice: 31
Bob: 25
```

**Why this output?**  
Keys are sorted alphabetically; `ages["Alice"] = 31` updates the existing value.

**Example 2: `insert` vs. `operator[]`**

```cpp
#include <map>
#include <iostream>

int main() {
    std::map<int, std::string> m;
    m[1] = "one";                          // inserts
    m.insert({2, "two"});                  // inserts
    m.insert({1, "ONE"});                  // ignored: key 1 exists

    for (const auto& [k, v] : m)
        std::cout << k << ": " << v << '\n';
}
```

**Expected Output**
```
1: one
2: two
```

**Why this output?**  
`insert` does not overwrite an existing key; `operator[]` does.

**Example 3: `at` for safe lookup**

```cpp
#include <map>
#include <iostream>
#include <stdexcept>

int main() {
    std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
    try {
        std::cout << m.at(1) << '\n';      // "one"
        std::cout << m.at(5) << '\n';      // throws
    } catch (const std::out_of_range&) {
        std::cout << "Key not found\n";
    }
}
```

**Expected Output**
```
one
Key not found
```

**Why this output?**  
`at` throws when the key is absent, whereas `operator[]` would insert a default value.

### Real-World Cases

- **Dictionaries** – word-to-definition mappings.
- **Configuration stores** – key–value settings.
- **Symbol tables** – compilers and interpreters.
- **Frequency maps** – `std::map<std::string, int>` word counts.

---

## 4. `std::multimap`

### Definitions

**Core Definition**
`std::multimap` is an associative container that contains a sorted list of key–value pairs, permitting multiple entries with the same key.

**Technical Definition**
Declared in `<map>`:
```cpp
template<class Key, class T, class Compare = std::less<Key>,
         class Allocator = std::allocator<std::pair<const Key, T>>>
class multimap;
```
Search, insertion, and removal have logarithmic complexity. The order of key–value pairs with equivalent keys is the order of insertion and does not change (since C++11).

**Beginner-Friendly Explanation**
`std::multimap` is a sorted dictionary that allows several entries for the same word.

### Purposes

- To store multiple values for the same key.
- To maintain sorted order by key while allowing duplicates.
- To support range queries over equivalent keys via `equal_range`.
- To model one-to-many relationships.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <map>
std::multimap<Key, T> mm;
mm.insert({key, value});
auto range = mm.equal_range(key);
```

**Component Breakdown**

| Operation | Complexity |
|-----------|------------|
| `insert` | O(log n) |
| `erase` | O(log n) |
| `find` | O(log n) (returns any equivalent element) |
| `count` | O(log n + k) |
| `equal_range` | O(log n) |

**Syntax Rules**
- `multimap` does not provide `operator[]` or `at()` because keys are not unique.
- Use `equal_range` to access all values associated with a key.
- Equivalent elements are visited in insertion order (since C++11).

**Constraints and Limitations**
- No direct subscript access.
- Removing a key requires `erase(key)` (removes all) or iteration with `erase(iterator)`.

### Annotated Code Examples

**Example 1: Multiple values per key**

```cpp
#include <map>
#include <iostream>

int main() {
    std::multimap<char, int> mm;
    mm.insert({'a', 1});
    mm.insert({'a', 2});
    mm.insert({'b', 3});

    auto range = mm.equal_range('a');
    for (auto it = range.first; it != range.second; ++it)
        std::cout << it->second << ' ';
}
```

**Expected Output**
```
1 2
```

**Why this output?**  
`equal_range('a')` returns the range of all values associated with key `'a'`.

**Example 2: Sorted iteration**

```cpp
#include <map>
#include <iostream>

int main() {
    std::multimap<int, char> mm = {{2, 'b'}, {1, 'a'}, {2, 'c'}};
    for (const auto& [k, v] : mm)
        std::cout << k << v << ' ';
}
```

**Expected Output**
```
1a 2b 2c
```

**Why this output?**  
Keys are sorted; equivalent keys retain insertion order.

### Real-World Cases

- **One-to-many relationships** – a student enrolled in multiple courses.
- **Log aggregation** – multiple log entries per timestamp.
- **DNS caches** – multiple IP addresses per hostname.

---

## 5. Ordered Containers and the Red–Black Tree

### Definitions

**Core Definition**
Ordered containers are associative containers that maintain their elements in a sorted order determined by a key comparison function. The red–black tree is a self-balancing binary search tree commonly used to implement these containers.

**Technical Definition**
A red–black tree is a binary search tree with one extra bit of storage per node (its colour, red or black) and a set of invariants that guarantee the tree remains approximately balanced: the root is black; every leaf (NIL) is black; red nodes have black children; every path from a node to its descendant leaves contains the same number of black nodes. These invariants guarantee O(log n) height. The C++ standard does not mandate red–black trees, but their complexity guarantees—combined with the requirement for stable iterators and logarithmic operations—effectively narrow the practical choices to AVL trees and red–black trees; in practice, red–black trees are the universal choice.

**Beginner-Friendly Explanation**
A red–black tree is a cleverly balanced binary search tree. It uses colours (red and black) to keep itself roughly balanced, so search, insert, and delete all take logarithmic time. It is the engine behind `std::set` and `std::map`.

### Purposes

- To provide guaranteed O(log n) search, insertion, and removal.
- To maintain sorted order for iteration and range queries.
- To keep iterators stable across insertions and erasures.
- To serve as the underlying data structure for ordered associative containers.

### Syntax Rules and Structure (Conceptual)

**Conceptual Node Layout**
```
struct node_base {
    node_base* parent;
    node_base* left;
    node_base* right;
    bool color;          // red or black
    // payload: Key (set) or pair<const Key, T> (map)
};
```

A tree object typically holds a header/sentinel node and a `size` member. The sentinel’s `parent` points to the root, `left` points to the smallest element, and `right` points to the largest element. This sentinel also serves as the `end()` iterator.

**Constraints and Limitations**
- The exact layout is implementation-defined.
- Red–black trees have higher constant factors than hash tables for pure lookup.
- They are not cache-friendly compared with contiguous containers.

### Annotated Code Example (Conceptual)

```cpp
// Conceptual illustration, not actual std::set internals
// Each insertion triggers rebalancing via rotations and recolouring.
// The result: height = O(log n), guaranteeing logarithmic operations.
```

**Expected Output**  
(No output; conceptual illustration.)

### Real-World Cases

- **Standard library implementation** – all four ordered associative containers.
- **Database indexes** – B-tree variants are common, but red–black trees appear in in-memory indexes.
- **Scheduler queues** – ordered task management.

---

## 6. Set Operations

### Definitions

**Core Definition**
Set operations are the member functions and algorithms used to insert, remove, search, and traverse elements in `std::set` and `std::multiset`.

**Technical Definition**
The core operations include `insert`, `emplace`, `erase`, `find`, `count`, `lower_bound`, `upper_bound`, `equal_range`, `clear`, and iteration via `begin`/`end`. All logarithmic operations are guaranteed by the underlying balanced tree.

**Beginner-Friendly Explanation**
These are the commands you use to work with sets: adding items, removing them, checking whether something is present, and walking through the sorted elements.

### Key Operations

| Operation | Description | Complexity |
|-----------|-------------|------------|
| `insert(value)` | Insert a value (ignored if equivalent key exists in `set`) | O(log n) |
| `emplace(args...)` | Construct in place | O(log n) |
| `erase(key)` | Remove all elements equivalent to `key` | O(log n) |
| `erase(iterator)` | Remove the element at the iterator | Amortised O(1) |
| `find(key)` | Return iterator to element, or `end()` | O(log n) |
| `count(key)` | Number of elements equivalent to `key` | O(log n + k) |
| `lower_bound(key)` | First element not less than `key` | O(log n) |
| `upper_bound(key)` | First element greater than `key` | O(log n) |
| `equal_range(key)` | Pair of iterators delimiting equivalent elements | O(log n) |
| `clear()` | Remove all elements | O(n) |

**Syntax Rules**
- `insert` returns `pair<iterator, bool>` for `set`; for `multiset` it returns an iterator.
- `erase(key)` removes all equivalent elements; use `erase(iterator)` to remove one.
- `count` for `set` returns 0 or 1; for `multiset` it may return any non-negative integer.

**Constraints and Limitations**
- Elements in a `set` cannot be modified through iterators; they are effectively `const`.
- Erasing invalidates only iterators to the erased element.

### Annotated Code Example

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> s = {10, 20, 30, 40};

    s.insert(25);                          // O(log n)
    s.emplace(35);                         // O(log n)

    if (s.find(20) != s.end())
        std::cout << "20 found\n";

    s.erase(30);                           // O(log n)

    for (int x : s) std::cout << x << ' ';
}
```

**Expected Output**
```
20 found
10 20 25 35 40
```

**Why this output?**  
`insert` and `emplace` add elements in sorted order; `find` confirms `20` is present; `erase(30)` removes it.

---

## 7. Map Operations

### Definitions

**Core Definition**
Map operations are the member functions used to access, insert, update, and remove key–value pairs in `std::map` and `std::multimap`.

**Technical Definition**
The core operations include `operator[]`, `at`, `insert`, `emplace`, `try_emplace` (C++17), `erase`, `find`, `count`, `lower_bound`, `upper_bound`, and `equal_range`.

**Beginner-Friendly Explanation**
These are the commands for working with dictionaries: looking up a value by key, adding or updating entries, and removing entries.

### Key Operations

| Operation | Description | Complexity | Notes |
|-----------|-------------|------------|-------|
| `operator[]` | Access or insert default-constructed value | O(log n) | Inserts if key absent. |
| `at(key)` | Access with bounds checking | O(log n) | Throws `std::out_of_range` if absent. |
| `insert({k, v})` | Insert only if key absent | O(log n) | Does not overwrite. |
| `insert_or_assign(k, v)` (C++17) | Insert or overwrite | O(log n) | |
| `try_emplace(k, args...)` (C++17) | Construct value only if key absent | O(log n) | Avoids unnecessary construction. |
| `erase(key)` | Remove elements with key | O(log n) | |
| `find(key)` | Iterator to element or `end()` | O(log n) | |
| `count(key)` | Number of elements with key | O(log n + k) | |
| `equal_range(key)` | Range of equivalent keys | O(log n) | Essential for `multimap`. |

**Syntax Rules**
- `operator[]` requires `T` to be default-constructible and silently inserts if the key is missing.
- `at` is the safe lookup when absence is an error.
- `try_emplace` avoids constructing the mapped value if the key already exists, which is important for expensive types.
- `multimap` does not provide `operator[]` or `at`.

**Constraints and Limitations**
- `operator[]` is a common source of accidental insertions; prefer `find` or `at` for pure lookups.
- `insert` does not overwrite existing keys; use `insert_or_assign` or `operator[]` to update.

### Annotated Code Example

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::map<std::string, int> m;

    m["apple"] = 3;                        // insert
    m["banana"] = 5;                       // insert
    m.insert({"cherry", 2});               // insert only
    m.insert({"apple", 99});               // ignored

    auto [it, inserted] = m.try_emplace("date", 7); // construct only if absent

    for (const auto& [k, v] : m)
        std::cout << k << ": " << v << '\n';
}
```

**Expected Output**
```
apple: 3
banana: 5
cherry: 2
date: 7
```

**Why this output?**  
`operator[]` inserts or updates; `insert` does not overwrite `"apple"`; `try_emplace` inserts `"date"` because it is absent.

---

## 8. Set Best Practices

### Definitions

**Core Definition**
Set best practices are guidelines for using `std::set` and `std::multiset` correctly, safely, and efficiently.

**Technical Definition**
These practices are grounded in the C++ Core Guidelines, which recommend preferring `vector` by default unless there is a specific reason to use another container (SL.con.2). For sets, the specific reasons are ordered iteration, logarithmic membership testing, and automatic deduplication.

**Beginner-Friendly Explanation**
Use a set when you need sorted unique items and fast “is it there?” checks. Do not use a set when a vector or an unordered set would do the job better.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Use `set` for sorted unique keys** | When order matters and duplicates must be eliminated. |
| **Use `multiset` for sorted duplicates** | When you need to count or iterate over repeated keys. |
| **Prefer `find` over `count` for membership** | `count` may traverse all equivalent elements in a `multiset`. |
| **Use `lower_bound` / `upper_bound` for ranges** | Avoid linear scans for range queries. |
| **Do not modify elements through iterators** | Elements are `const`; modifying them would break the ordering. |
| **Consider `unordered_set` for pure lookup** | If ordering is not needed, a hash set is usually faster. |
| **Reserve memory if possible** | `set` has no `reserve`, but be aware of allocation overhead. |
| **Provide a strict weak ordering** | An invalid `Compare` leads to undefined behaviour. |

### Annotated Code Example

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> s = {5, 3, 1, 4, 2};

    // Use find, not count, for simple membership
    if (s.find(3) != s.end())
        std::cout << "3 is in the set\n";

    // Range query with lower_bound / upper_bound
    auto lo = s.lower_bound(2);
    auto hi = s.upper_bound(4);
    for (auto it = lo; it != hi; ++it)
        std::cout << *it << ' ';
}
```

**Expected Output**
```
3 is in the set
2 3 4
```

**Why this output?**  
`find` confirms membership; `lower_bound(2)` points to `2`, and `upper_bound(4)` points to `5`, so `2 3 4` is printed.

---

## 9. Map Best Practices

### Definitions

**Core Definition**
Map best practices are guidelines for using `std::map` and `std::multimap` correctly, safely, and efficiently.

**Technical Definition**
These practices focus on avoiding accidental insertions via `operator[]`, choosing the right insertion function (`insert`, `try_emplace`, `insert_or_assign`), and understanding the performance and semantic differences between lookup and modification.

**Beginner-Friendly Explanation**
Use a map when you need sorted key–value pairs. Be careful with `operator[]`—it can silently insert a default value when you only meant to look something up.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Avoid `operator[]` for pure lookup** | It inserts a default-constructed value if the key is absent. |
| **Use `at()` for checked lookup** | Throws if the key is absent; safer than `operator[]` for read-only access. |
| **Use `try_emplace` (C++17)** | Constructs the mapped value only if the key is absent; avoids unnecessary temporaries. |
| **Use `insert_or_assign` (C++17) for update-or-insert** | Clearer intent than `operator[]` when you want to overwrite. |
| **Use `equal_range` for `multimap`** | `multimap` has no `operator[]`; `equal_range` is the canonical access pattern. |
| **Prefer `find` over `count` for membership** | `count` on a `multimap` may traverse all equivalent elements. |
| **Consider `unordered_map` for pure lookup** | If ordering is not needed, a hash map is usually faster. |
| **Be aware of key `const`-ness** | Keys cannot be modified through iterators; use `extract` (C++17) if you need to change a key. |

### Annotated Code Example

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::map<std::string, int> m = {{"a", 1}, {"b", 2}};

    // Safe lookup with at()
    try {
        std::cout << m.at("a") << '\n';    // 1
        std::cout << m.at("z") << '\n';    // throws
    } catch (const std::out_of_range&) {
        std::cout << "z not found\n";
    }

    // try_emplace only constructs if absent
    m.try_emplace("c", 3);
    m.try_emplace("a", 99);                // ignored: "a" exists

    for (const auto& [k, v] : m)
        std::cout << k << ':' << v << ' ';
}
```

**Expected Output**
```
1
z not found
a:1 b:2 c:3
```

**Why this output?**  
`at` performs a checked lookup; `try_emplace` inserts `"c"` but does not overwrite `"a"`.

---

## References

- `std::set` — cppreference – https://en.cppreference.com/w/cpp/container/set
- `std::multiset` — cppreference – https://en.cppreference.com/w/cpp/container/multiset
- `std::map` — cppreference – https://en.cppreference.com/w/cpp/container/map
- `std::multimap` — cppreference – https://en.cppreference.com/w/cpp/container/multimap
- Associative containers (C++) — Wikipedia – https://en.wikipedia.org/wiki/Associative_containers_(C%2B%2B)
- Inside STL: The map, set, multimap, and multiset — The Old New Thing (Microsoft) – https://devblogs.microsoft.com/oldnewthing/20230807-00/?p=108562
- C++ Core Guidelines — SL.con: Containers – https://cpp-core-guidelines-docs.vercel.app/stdlib
- C++ Core Guidelines — SL.con.2: Prefer using STL `vector` by default – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#slcon2-prefer-using-stl-vector-by-default-unless-you-have-a-reason-to-use-a-different-container
- How to Look up Values in a Map — Sandor Dargo (ISO C++) – https://isocpp.org/blog/2025/10/how-to-look-up-values-in-a-map
- `std::map::operator[]` — cppreference – https://en.cppreference.com/w/cpp/container/map/operator_at
- Ordered Containers — Red-Black Tree (cpp_mastery) – https://raw.githubusercontent.com/nimbustan/cpp_mastery/refs/heads/master/EN/module_04_advanced/06_associative_containers.cpp

---

This cheat sheet provides a comprehensive, structured reference for C++ sets and maps, covering all four ordered associative containers, the red–black tree foundation, set and map operations, performance characteristics, best practices, annotated code examples with expected outputs, and authoritative references.