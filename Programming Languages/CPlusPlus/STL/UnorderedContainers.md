# C++ Unordered Containers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Unordered associative containers are C++ Standard Library containers that store elements in a hash table, providing average constant-time search, insertion, and removal without maintaining any sorted order.

**Technical Definition**
`std::unordered_set`, `std::unordered_multiset`, `std::unordered_map`, and `std::unordered_multimap` (all since C++11) satisfy the *UnorderedAssociativeContainer* named requirement. They are parameterized by `Key`, a `Hash` function object (default `std::hash<Key>`), a `KeyEqual` binary predicate (default `std::equal_to<Key>`), and an `Allocator`. Elements are organized into buckets: keys with the same hash value end up in the same bucket. Average complexity for search, insertion, and removal is constant; worst case is linear.

**Beginner-Friendly Explanation**
Think of an unordered container as a set of labeled drawers (buckets). A hash function tells you which drawer a given key belongs in, so you can go straight to the right drawer instead of searching through every item. This is why lookups are usually very fast—but there is no “sorted order,” so you cannot iterate in any predictable sequence.

### Key Characteristics

- **Hash-based organization** – elements are placed into buckets according to their hash value.
- **Average O(1) operations** – search, insertion, and removal are average constant time; worst case is O(n) .
- **No ordering** – iteration order is unspecified and may change after rehashing.
- **Unique vs. equivalent keys** – `unordered_set`/`unordered_map` allow at most one element per key; the `multi` variants allow multiple equivalent keys (which are always adjacent on iteration).
- **Iterator stability** – insertion does not invalidate references; rehashing invalidates all iterators but not references.
- **Customizable** – hash function, key equality, and bucket count can be supplied.

### Prerequisites

- Basic C++ syntax (variables, types, control flow).
- Understanding of templates and function objects.
- Familiarity with iterators and the range-based `for` loop.
- Awareness of Big-O notation.
- Basic knowledge of hash tables (helpful but not required).

### Related Programming Areas

- **Ordered associative containers** – `std::set`, `std::map` provide sorted alternatives with O(log n) operations.
- **STL algorithms** – many algorithms work with unordered container iterators, though ordering assumptions must be avoided.
- **Generic programming** – hash and equality customization for user-defined types.
- **Performance engineering** – load factor tuning and rehashing control.

### Core Concepts / Features

1. `std::unordered_set`
2. `std::unordered_multiset`
3. `std::unordered_map`
4. `std::unordered_multimap`
5. Hash functions
6. Bucket interface
7. Load factor
8. Rehashing
9. Unordered container best practices

---

## 1. `std::unordered_set`

### Definitions

**Core Definition**
`std::unordered_set` is an associative container that contains a set of unique objects of type `Key`, organized into buckets using a hash function.

**Technical Definition**
Declared in `<unordered_set>`:
```cpp
template<class Key, class Hash = std::hash<Key>,
         class KeyEqual = std::equal_to<Key>,
         class Allocator = std::allocator<Key>>
class unordered_set;
```
Search, insertion, and removal have average constant-time complexity. Internally, elements are not sorted but organized into buckets; which bucket an element goes into depends entirely on the hash of its key.

**Beginner-Friendly Explanation**
`std::unordered_set` is a fast “bag of unique items.” It does not keep items sorted, but it can tell you very quickly whether an item is present.

### Purposes

- To provide average O(1) membership testing.
- To deduplicate keys without requiring sorting.
- To support fast insertion and removal.
- To serve as a hash-based collection of unique keys.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <unordered_set>
std::unordered_set<Key> us;
std::unordered_set<Key> us = {a, b, c};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Key` | The type of elements stored. |
| `Hash` | Hash function object (default `std::hash<Key>`). |
| `KeyEqual` | Equality predicate (default `std::equal_to<Key>`). |

**Member Function Complexity**

| Operation | Average | Worst Case |
|-----------|---------|------------|
| `insert` | O(1) | O(n) |
| `erase` | O(1) | O(n) |
| `find` | O(1) | O(n) |
| `count` | O(1) | O(n) |

**Syntax Rules**
- Uniqueness is determined by `KeyEqual`: two keys are equivalent if `KeyEqual(a, b)` returns `true`.
- If two keys are equivalent, `Hash` must return the same value for both.
- Elements are `const` through iterators.

**Constraints and Limitations**
- Iteration order is unspecified and may change after rehashing.
- Worst-case O(n) if the hash function distributes poorly.
- No random access; iterators are forward iterators.

### Annotated Code Examples

**Example 1: Uniqueness and fast lookup**

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_set<int> us = {3, 1, 2, 3, 1};
    std::cout << us.size() << '\n';        // 3 (duplicates removed)
    if (us.find(2) != us.end())
        std::cout << "2 found\n";
}
```

**Expected Output** (iteration order may vary)
```
3
2 found
```

**Why this output?**  
Duplicates are ignored; `find(2)` succeeds because `2` is present.

**Example 2: `count` for membership**

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_set<int> us = {10, 20, 30};
    std::cout << us.count(20) << '\n';     // 1
    std::cout << us.count(99) << '\n';     // 0
}
```

**Expected Output**
```
1
0
```

**Why this output?**  
For `unordered_set`, `count` returns 0 or 1.

### Real-World Cases

- **Deduplication** – removing duplicate records from a data stream.
- **Visited-set tracking** – graph algorithms.
- **Caches** – fast membership checks.

---

## 2. `std::unordered_multiset`

### Definitions

**Core Definition**
`std::unordered_multiset` is an unordered associative container that contains a set of objects of type `Key`, allowing multiple keys with equivalent values.

**Technical Definition**
Declared in `<unordered_set>`:
```cpp
template<class Key, class Hash = std::hash<Key>,
         class KeyEqual = std::equal_to<Key>,
         class Allocator = std::allocator<Key>>
class unordered_multiset;
```
Unlike `unordered_set`, multiple elements with equivalent keys are permitted. Equivalent keys are always adjacent on iteration.

**Beginner-Friendly Explanation**
`std::unordered_multiset` is a fast “bag” that keeps duplicates.

### Purposes

- To maintain a hash-based collection that permits duplicate keys.
- To count occurrences without a separate counter.
- To support fast access to groups of equivalent elements.
- To model multisets with constant-time average lookup.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <unordered_set>
std::unordered_multiset<Key> ums;
```

**Component Breakdown**

| Operation | Average Complexity |
|-----------|-------------------|
| `insert` | O(1) |
| `erase` | O(1) |
| `count` | O(1) average; O(n) worst |
| `equal_range` | O(1) average |

**Syntax Rules**
- `count(key)` returns the number of elements equivalent to `key`.
- `equal_range(key)` returns a pair of iterators delimiting all equivalent elements.
- No `operator[]`.

**Constraints and Limitations**
- Duplicates increase memory usage.
- `erase(key)` removes all elements with that key.

### Annotated Code Examples

**Example 1: Duplicates preserved**

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_multiset<int> ums = {3, 1, 2, 3, 1};
    std::cout << ums.count(1) << '\n';     // 2
    std::cout << ums.count(3) << '\n';     // 2
}
```

**Expected Output**
```
2
2
```

**Why this output?**  
Both `1` and `3` appear twice; `count` returns 2 for each.

### Real-World Cases

- **Frequency counting** – tallying occurrences of values.
- **Event logs** – multiple events with the same identifier.

---

## 3. `std::unordered_map`

### Definitions

**Core Definition**
`std::unordered_map` is an unordered associative container that contains key–value pairs with unique keys.

**Technical Definition**
Declared in `<unordered_map>`:
```cpp
template<class Key, class T, class Hash = std::hash<Key>,
         class KeyEqual = std::equal_to<Key>,
         class Allocator = std::allocator<std::pair<const Key, T>>>
class unordered_map;
```
Search, insertion, and removal have average constant-time complexity. The mapped type `T` is associated with the key `Key`.

**Beginner-Friendly Explanation**
`std::unordered_map` is a fast hash-based dictionary: you give it a key, and it retrieves the associated value almost instantly.

### Purposes

- To provide average O(1) key–value lookup.
- To implement associative arrays without ordering.
- To support high-performance counting and indexing.
- To serve as a hash-based dictionary.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <unordered_map>
std::unordered_map<Key, T> um;
um[key] = value;
```

**Component Breakdown**

| Operation | Average Complexity | Notes |
|-----------|-------------------|-------|
| `operator[]` | O(1) | Inserts default value if key absent. |
| `at` | O(1) | Throws if key absent. |
| `insert` | O(1) | Does not overwrite. |
| `find` | O(1) | |
| `erase` | O(1) | |

**Syntax Rules**
- `operator[]` requires `T` to be default-constructible.
- `insert` does not overwrite existing keys.
- `at` throws `std::out_of_range` if the key is absent.

**Constraints and Limitations**
- `operator[]` silently inserts a default-constructed value for a missing key—a common source of bugs when a pure lookup was intended.
- Iteration order is unspecified.

### Annotated Code Examples

**Example 1: Fast dictionary lookup**

```cpp
#include <unordered_map>
#include <iostream>
#include <string>

int main() {
    std::unordered_map<std::string, int> um;
    um["Alice"] = 30;
    um["Bob"] = 25;
    std::cout << um["Alice"] << '\n';      // 30
}
```

**Expected Output**
```
30
```

**Why this output?**  
`operator[]` retrieves the value associated with `"Alice"`.

**Example 2: `at` vs. `operator[]`**

```cpp
#include <unordered_map>
#include <iostream>
#include <stdexcept>

int main() {
    std::unordered_map<int, std::string> um = {{1, "one"}};
    try {
        std::cout << um.at(5) << '\n';     // throws
    } catch (const std::out_of_range&) {
        std::cout << "5 not found\n";
    }
    std::cout << um.size() << '\n';        // still 1
}
```

**Expected Output**
```
5 not found
1
```

**Why this output?**  
`at` throws without inserting; `operator[]` would have inserted a default value and increased the size.

### Real-World Cases

- **Caches** – memoization tables.
- **Frequency counting** – `unordered_map<std::string, int>`.
- **Symbol tables** – fast identifier lookup.

---

## 4. `std::unordered_multimap`

### Definitions

**Core Definition**
`std::unordered_multimap` is an unordered associative container that contains key–value pairs, permitting multiple entries with the same key.

**Technical Definition**
Declared in `<unordered_map>`:
```cpp
template<class Key, class T, class Hash = std::hash<Key>,
         class KeyEqual = std::equal_to<Key>,
         class Allocator = std::allocator<std::pair<const Key, T>>>
class unordered_multimap;
```
Multiple elements with equivalent keys are allowed and are adjacent on iteration.

**Beginner-Friendly Explanation**
`std::unordered_multimap` is a fast dictionary that allows several entries for the same key.

### Purposes

- To store multiple values for the same key with hash-based speed.
- To model one-to-many relationships.
- To support fast access to groups of equivalent keys via `equal_range`.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <unordered_map>
std::unordered_multimap<Key, T> umm;
umm.insert({key, value});
auto range = umm.equal_range(key);
```

**Component Breakdown**

| Operation | Average Complexity |
|-----------|-------------------|
| `insert` | O(1) |
| `erase` | O(1) |
| `find` | O(1) |
| `equal_range` | O(1) |

**Syntax Rules**
- No `operator[]` or `at`.
- Use `equal_range` to access all values for a key.

**Constraints and Limitations**
- Equivalent keys are adjacent but unordered relative to each other.

### Annotated Code Examples

**Example 1: Multiple values per key**

```cpp
#include <unordered_map>
#include <iostream>

int main() {
    std::unordered_multimap<char, int> umm;
    umm.insert({'a', 1});
    umm.insert({'a', 2});
    umm.insert({'b', 3});

    auto range = umm.equal_range('a');
    for (auto it = range.first; it != range.second; ++it)
        std::cout << it->second << ' ';
}
```

**Expected Output** (order may vary)
```
1 2
```

**Why this output?**  
`equal_range('a')` returns all values associated with `'a'`.

### Real-World Cases

- **One-to-many relationships** – multiple phone numbers per contact.
- **DNS caches** – multiple IP addresses per hostname.

---

## 5. Hash Functions

### Definitions

**Core Definition**
A hash function maps a key to a `std::size_t` value, which determines the bucket in which the key is stored.

**Technical Definition**
`std::hash<Key>` is a function object template defined in `<functional>`. Its `operator()` accepts a `Key` and returns a `std::size_t` hash value. The unordered containers use specializations of `std::hash` as their default hash function. Requirements: (1) it must not throw; (2) for two equivalent keys `k1` and `k2`, `std::hash<Key>()(k1) == std::hash<Key>()(k2)`; (3) for different keys, the probability of collision should be very small. Hash functions are only required to produce the same result for the same input within a single execution of a program; this permits salted hashes that defend against collision DoS attacks (since C++14).

**Beginner-Friendly Explanation**
A hash function is a “fingerprint maker” for keys. It turns a key into a number so the container knows which drawer (bucket) to put it in. Good hash functions spread keys evenly across drawers; bad ones pile everything into one drawer and destroy performance.

### Purposes

- To distribute keys evenly across buckets.
- To enable average O(1) lookup.
- To provide a customizable mechanism for user-defined types.
- To defend against hash-collision attacks via salted hashing.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::hash<Key>{}(key);          // returns std::size_t
```

**Component Breakdown**

| Requirement | Description |
|-------------|-------------|
| Signature | `std::size_t operator()(const Key&) const` |
| No exceptions | Must not throw |
| Equality consistency | Equal keys ⇒ equal hashes |
| Low collision probability | Different keys should rarely collide |

**Syntax Rules**
- For a user-defined type, you may specialize `std::hash<Key>` in namespace `std` or provide a custom hash functor as a template argument.
- `std::hash<const char*>` hashes the pointer value, **not** the string contents—use `std::hash<std::string>` or a custom hasher for C strings.
- Since C++20, if both `Hash::is_transparent` and `KeyEqual::is_transparent` are valid, heterogeneous lookup is enabled: `find`, `contains`, `count`, `equal_range`, and `bucket` accept arguments of types other than `Key` without constructing a `Key`.

**Constraints and Limitations**
- The standard does not require cryptographic quality; `std::hash` is designed for container use, not security.
- Implementations may use identity hashing for integers.
- A poor hash function causes worst-case O(n) behavior.

### Annotated Code Examples

**Example 1: Custom hash specialization**

```cpp
#include <unordered_set>
#include <iostream>

struct Point {
    int x, y;
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

namespace std {
    template<>
    struct hash<Point> {
        std::size_t operator()(const Point& p) const {
            // Combine hashes; simple but effective for this example
            return std::hash<int>{}(p.x) ^ (std::hash<int>{}(p.y) << 1);
        }
    };
}

int main() {
    std::unordered_set<Point> pts;
    pts.insert({1, 2});
    pts.insert({1, 2});                    // duplicate ignored
    std::cout << pts.size() << '\n';       // 1
}
```

**Expected Output**
```
1
```

**Why this output?**  
The custom hash specialization allows `Point` to be used in `unordered_set`; the `==` operator makes duplicates equivalent.

**Example 2: Transparent hashing (C++20)**

```cpp
#include <unordered_set>
#include <string>
#include <string_view>
#include <iostream>

struct string_hash {
    using is_transparent = void;            // enables heterogeneous lookup
    std::size_t operator()(std::string_view sv) const {
        return std::hash<std::string_view>{}(sv);
    }
};

int main() {
    std::unordered_set<std::string, string_hash, std::equal_to<>> s;
    s.insert("hello");
    std::cout << s.contains(std::string_view{"hello"}) << '\n'; // 1
}
```

**Expected Output**
```
1
```

**Why this output?**  
With `is_transparent`, `contains` accepts a `std::string_view` without constructing a temporary `std::string`.

### Real-World Cases

- **User-defined key types** – points, IDs, composite keys.
- **Heterogeneous lookup** – avoiding temporary string allocations.
- **Security-sensitive applications** – salted hashes prevent collision DoS.

---

## 6. Bucket Interface

### Definitions

**Core Definition**
The bucket interface provides functions to inspect and navigate the internal bucket structure of an unordered container.

**Technical Definition**
Each unordered container exposes: `bucket_count()` (number of buckets), `max_bucket_count()` (implementation limit), `bucket_size(n)` (number of elements in bucket `n`), and `bucket(key)` (index of the bucket where `key` would be found). Iterators `begin(n)` and `end(n)` traverse a single bucket.

**Beginner-Friendly Explanation**
Buckets are the “drawers” of the hash table. The bucket interface lets you peek inside: how many drawers there are, how many items each drawer holds, and which drawer a key belongs to.

### Purposes

- To diagnose hash function quality (bucket size distribution).
- To traverse individual buckets.
- To reason about rehashing and load factor.
- To tune initial bucket count for performance.

### Syntax Rules and Structure

**General Syntax**
```cpp
us.bucket_count();          // number of buckets
us.bucket_size(n);          // elements in bucket n
us.bucket(key);             // bucket index for key
us.begin(n);                // iterator to first element in bucket n
us.end(n);                  // iterator past last element in bucket n
```

**Component Breakdown**

| Function | Description | Complexity |
|----------|-------------|------------|
| `bucket_count()` | Number of buckets | O(1) |
| `max_bucket_count()` | Maximum possible buckets | O(1) |
| `bucket_size(n)` | Elements in bucket `n` | O(bucket_size(n)) |
| `bucket(key)` | Bucket index for `key` | O(1) |

**Syntax Rules**
- `bucket(key)` returns a value in `[0, bucket_count())`.
- The return value is valid only while `bucket_count()` is unchanged; `rehash()` invalidates it.
- `begin(n)` and `end(n)` iterate a single bucket, not the whole container.

**Constraints and Limitations**
- The number of buckets is implementation-managed and may change on insertion.
- Bucket sizes are not directly controllable except via rehashing.

### Annotated Code Examples

**Example 1: Inspecting bucket distribution**

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_set<int> us = {1, 2, 3, 4, 5, 6, 7, 8};
    std::cout << "buckets: " << us.bucket_count() << '\n';
    for (std::size_t i = 0; i < us.bucket_count(); ++i) {
        if (us.bucket_size(i) > 0)
            std::cout << "bucket " << i << ": "
                      << us.bucket_size(i) << " element(s)\n";
    }
}
```

**Expected Output** (bucket counts and distribution are implementation-dependent)
```
buckets: 13
bucket 1: 1 element(s)
bucket 2: 1 element(s)
...
```

**Why this output?**  
`bucket_count()` reports the number of buckets; `bucket_size(i)` shows how many elements are in each non-empty bucket.

### Real-World Cases

- **Hash quality diagnosis** – a skewed bucket distribution indicates a poor hash function.
- **Performance tuning** – choosing an initial `bucket_count` to avoid early rehashes.

---

## 7. Load Factor

### Definitions

**Core Definition**
The load factor is the average number of elements per bucket: `size() / bucket_count()`.

**Technical Definition**
`load_factor()` returns the current average number of elements per bucket. `max_load_factor()` gets or sets the maximum allowed load factor (default is implementation-defined, often 1.0). The container automatically increases the number of buckets if the load factor would exceed this threshold upon insertion.

**Beginner-Friendly Explanation**
The load factor tells you how “full” the hash table is. If each bucket holds many items, lookups slow down. `max_load_factor` is the limit: when the table gets too full, it automatically grows more buckets.

### Purposes

- To control when rehashing occurs.
- To balance memory usage against lookup speed.
- To tune performance for specific workloads.
- To prevent excessive bucket chains.

### Syntax Rules and Structure

**General Syntax**
```cpp
float lf  = um.load_factor();            // current load factor
float mlf = um.max_load_factor();        // current maximum
um.max_load_factor(0.75f);              // set maximum
```

**Component Breakdown**

| Function | Description | Complexity |
|----------|-------------|------------|
| `load_factor()` | Average elements per bucket | O(1) |
| `max_load_factor()` | Get/set maximum load factor | O(1) |

**Syntax Rules**
- `load_factor()` = `size() / bucket_count()`.
- If insertion would make `size() > max_load_factor() * bucket_count()`, rehashing occurs before insertion.
- Setting `max_load_factor` does not itself trigger rehashing; it affects future insertions.

**Constraints and Limitations**
- A lower `max_load_factor` means more buckets, more memory, and faster lookups.
- A higher `max_load_factor` means fewer buckets, less memory, and slower lookups (longer chains).
- The default maximum load factor is implementation-defined.

### Annotated Code Examples

**Example 1: Observing load factor**

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_set<int> us;
    std::cout << us.load_factor() << '\n'; // 0 (empty)
    for (int i = 0; i < 100; ++i) us.insert(i);
    std::cout << us.load_factor() << '\n'; // ~size/bucket_count
}
```

**Expected Output** (values are implementation-dependent)
```
0
0.8
```

**Why this output?**  
`load_factor` starts at 0 and approaches `max_load_factor` as elements are inserted.

**Example 2: Tuning `max_load_factor`**

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_set<int> us;
    us.max_load_factor(0.5f);              // more buckets, faster lookups
    for (int i = 0; i < 100; ++i) us.insert(i);
    std::cout << us.bucket_count() << '\n';
}
```

**Expected Output** (implementation-dependent)
```
211
```

**Why this output?**  
With a lower maximum load factor, the container allocates more buckets to keep `load_factor ≤ 0.5`.

### Real-World Cases

- **Latency-sensitive systems** – lower load factors reduce chain lengths and lookup times.
- **Memory-constrained systems** – higher load factors save memory at the cost of speed.
- **Pre-sizing tables** – setting the load factor before bulk insertion.

---

## 8. Rehashing

### Definitions

**Core Definition**
Rehashing is the process of changing the number of buckets and redistributing all elements into the new buckets.

**Technical Definition**
`rehash(count)` sets the number of buckets to at least `count` and at least `size() / max_load_factor()`, then redistributes all elements. Rehashing occurs automatically on insertion when the new size would exceed `max_load_factor() * bucket_count()`. If rehashing occurs, all iterators are invalidated, but references and pointers to elements remain valid.

**Beginner-Friendly Explanation**
When the hash table gets too crowded, it builds a bigger set of drawers and moves every item into the new drawers according to their hashes. This is called rehashing. It invalidates iterators but keeps your element references valid.

### Purposes

- To keep the load factor under control.
- To restore average O(1) performance after growth.
- To allow manual control of bucket count via `rehash()` and `reserve()`.
- To force a rehash after temporarily disabling automatic rehashing.

### Syntax Rules and Structure

**General Syntax**
```cpp
um.rehash(count);                       // at least count buckets
um.reserve(count);                      // buckets for at least count elements
```

**Component Breakdown**

| Function | Effect | Complexity |
|----------|--------|------------|
| `rehash(count)` | Set bucket count to ≥ `max(count, size()/max_load_factor())` | Average O(n), worst O(n²) |
| `reserve(count)` | `rehash(ceil(count / max_load_factor()))` | Average O(n) |

**Syntax Rules**
- `rehash(0)` forces an unconditional rehash.
- Rehashing invalidates all iterators but **not** references or pointers.
- `reserve(n)` is equivalent to `rehash(ceil(n / max_load_factor()))` and is the idiomatic way to pre-size a container.

**Constraints and Limitations**
- Rehashing is expensive; avoid unnecessary calls.
- After rehashing, `bucket()` return values from before are invalid.

### Annotated Code Examples

**Example 1: `reserve` to avoid rehashing**

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_set<int> us;
    us.reserve(1000);                      // pre-allocate buckets
    for (int i = 0; i < 1000; ++i) us.insert(i);
    std::cout << us.bucket_count() << '\n';
}
```

**Expected Output** (implementation-dependent, but at least 1000)
```
1069
```

**Why this output?**  
`reserve(1000)` ensures enough buckets for at least 1000 elements without exceeding `max_load_factor`.

**Example 2: Forcing a rehash**

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_set<int> us = {1, 2, 3};
    auto buckets_before = us.bucket_count();
    us.rehash(100);                        // force at least 100 buckets
    std::cout << buckets_before << ' ' << us.bucket_count() << '\n';
}
```

**Expected Output** (implementation-dependent)
```
13 103
```

**Why this output?**  
`rehash(100)` changes the bucket count to at least 100, redistributing the elements.

### Real-World Cases

- **Bulk insertion** – `reserve` before inserting a known number of elements.
- **Post-tuning** – temporarily disabling automatic rehashing, then rehashing once.

---

## 9. Unordered Container Best Practices

### Definitions

**Core Definition**
Unordered container best practices are guidelines for choosing and using hash-based containers correctly, safely, and efficiently.

**Technical Definition**
These practices are grounded in the C++ Core Guidelines (SL.con.2: prefer `vector` by default unless there is a reason to use another container) and community consensus: use unordered containers for average O(1) lookup when ordering is unnecessary; provide high-quality hash functions and equality predicates; tune load factor and bucket count for performance; and be aware of rehashing's effect on iterators.

**Beginner-Friendly Explanation**
Use unordered containers when you need speed and don't care about order. Give them a good hash function, reserve space before bulk insertion, and remember that iterators can be invalidated by rehashing—even though references survive.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Use `unordered_*` for pure lookup speed** | When ordering is not required and average O(1) is desired. |
| **Use ordered containers when order matters** | `set`/`map` provide deterministic iteration. |
| **Provide a high-quality hash function** | Poor hashing leads to O(n) chains. |
| **Specialize `std::hash` or pass a custom hasher** | For user-defined types. |
| **Consider `is_transparent` (C++20)** | Enables heterogeneous lookup, avoiding temporary key objects. |
| **Use `reserve()` before bulk insertion** | Prevents repeated rehashing. |
| **Tune `max_load_factor`** | Lower for speed, higher for memory savings. |
| **Avoid `operator[]` for pure lookup in maps** | It silently inserts default values. |
| **Be aware of iterator invalidation** | Rehashing invalidates all iterators; references remain valid. |
| **Do not rely on iteration order** | Order is unspecified and may change after rehashing. |

### Annotated Code Example

```cpp
#include <unordered_map>
#include <iostream>
#include <string>
#include <string_view>

struct sv_hash {
    using is_transparent = void;
    std::size_t operator()(std::string_view sv) const {
        return std::hash<std::string_view>{}(sv);
    }
};

int main() {
    std::unordered_map<std::string, int, sv_hash, std::equal_to<>> m;
    m.reserve(100);                        // avoid rehashing
    m.max_load_factor(0.7f);              // balance speed and memory

    m["apple"] = 3;
    m["banana"] = 5;

    // Heterogeneous lookup: no temporary std::string
    if (m.contains(std::string_view{"apple"}))
        std::cout << "apple: " << m.at("apple") << '\n';
}
```

**Expected Output**
```
apple: 3
```

**Why this output?**  
`reserve` prevents rehashing; `max_load_factor` tunes the bucket growth policy; transparent hashing allows `contains` and `at` to accept `std::string_view` without constructing a `std::string`.

### Real-World Cases

- **Code reviews** – checking for missing `reserve` calls before bulk insertion.
- **Performance profiling** – validating that hash distribution is uniform.
- **API design** – exposing transparent lookup for string-like keys.

---

## References

- Unordered associative containers — cppreference – https://en.cppreference.com/w/cpp/container#Unordered_associative_containers
- `std::unordered_set` — cppreference – https://en.cppreference.com/w/cpp/container/unordered_set
- `std::unordered_multiset` — cppreference – https://en.cppreference.com/w/cpp/container/unordered_multiset
- `std::unordered_map` — cppreference – https://en.cppreference.com/w/cpp/container/unordered_map
- `std::unordered_multimap` — cppreference – https://en.cppreference.com/w/cpp/container/unordered_multimap
- UnorderedAssociativeContainer named requirement — cppreference – https://en.cppreference.com/w/cpp/named_req/UnorderedAssociativeContainer
- `std::hash` — cppreference – https://en.cppreference.com/w/cpp/utility/hash
- Bucket interface (`bucket`) — cppreference – https://en.cppreference.com/w/cpp/container/unordered_set/bucket
- `load_factor` — cppreference – https://en.cppreference.com/w/cpp/container/unordered_map/load_factor
- `max_load_factor` — cppreference – https://en.cppreference.com/w/cpp/container/unordered_map/max_load_factor
- `rehash` — cppreference – https://en.cppreference.com/w/cpp/container/unordered_multiset/rehash
- C++ Core Guidelines — SL.con: Containers – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-stdlib
- Custom hash specializations for user-defined types — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/60fbc72618f87c1b14a95a3c0322776df14a7139/09_Standard_Library_Utilities/Use_stdhash_and_define_custom_hash_specializations.md
- Heterogeneous lookup (transparent hashing) — cppreference – https://en.cppreference.com/w/cpp/container/unordered_map/find

---

This cheat sheet provides a comprehensive, structured reference for C++ unordered containers, covering all four container types, hash functions, the bucket interface, load factor, rehashing, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.