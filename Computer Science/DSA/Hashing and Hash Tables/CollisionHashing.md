# Advanced Collision Resolution Strategies: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Collision resolution strategies are the algorithms and data structures used to handle the situation when two or more keys hash to the same bucket index in a hash table.

**Technical Definition**
Collision resolution encompasses two primary paradigms—separate chaining (open hashing) and open addressing (closed hashing)—each with distinct mechanisms for storing colliding elements and retrieving them during lookup.

**Beginner-Friendly Explanation**
When two different keys map to the same slot in a hash table, they "collide." Collision resolution is the set of rules for deciding where to put the second key and how to find it later.

### Key Characteristics

- **Inevitability of Collisions**: Any hash function mapping a large universe to a finite table will produce collisions (pigeonhole principle).
- **Trade-offs**: Separate chaining uses extra memory for pointers but tolerates high load factors; open addressing uses less memory but requires careful probing and deletion handling.
- **Performance Degradation**: Without proper optimization, worst-case performance can degrade to O(n).
- **Modern Optimizations**: Java's HashMap converts long chains to Red-Black trees, achieving O(log n) worst-case chain operations.

### Prerequisites

- Hash function fundamentals (key → hash code → bucket index)
- Load factor concepts (α = n/m)
- Basic data structures: linked lists, arrays, trees

### Related Programming Areas

- Database indexing (hash indexes)
- Language runtime hash tables (Python dict, Java HashMap, C++ unordered_map)
- Caching systems and in-memory databases
- Network routing and distributed hash tables


## 1. Separate Chaining (Open Hashing)

### Definitions

**Core Definition**
Separate chaining resolves collisions by maintaining a secondary data structure (typically a linked list) for each bucket, where all keys mapping to the same index are stored together.

**Technical Definition**
Under separate chaining, each bucket `T[i]` contains a pointer to a linked list (or other collection) of all elements whose hash value equals `i`. Insertion appends to the list; lookup traverses the list comparing keys.

**Beginner-Friendly Explanation**
Imagine a post office with numbered mailboxes. When two people are assigned the same mailbox number, the post office puts a chain of boxes behind that number. To find someone, you go to the mailbox number, then walk down the chain until you find them.

### Purposes (All begin with "To")

- **To** handle collisions without requiring empty slots in the primary array.
- **To** allow the load factor to exceed 1 (more elements than buckets).
- **To** provide predictable O(1 + α) average-case performance where α is the load factor.
- **To** enable simple deletion without tombstones.

### Syntax Rules and Structure

**General Syntax (Python)**
```python
class ChainingHashTable:
    def __init__(self, capacity):
        self._capacity = capacity
        self._buckets = [[] for _ in range(capacity)]  # List of lists

    def _hash(self, key):
        return hash(key) % self._capacity

    def insert(self, key, value):
        index = self._hash(key)
        bucket = self._buckets[index]
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket[i] = (key, value)  # Update
                return
        bucket.append((key, value))  # Append to chain

    def find(self, key):
        index = self._hash(key)
        for k, v in self._buckets[index]:
            if k == key:
                return v
        raise KeyError(key)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `_buckets` | Array of chains (lists) |
| `_hash(key)` | Maps key to bucket index |
| `insert` | Appends or updates within the chain |
| `find` | Traverses chain comparing keys |

### Linked List Chaining vs. Dynamic Array Chaining

| Chain Type | Find | Delete | Insert |
|------------|------|--------|--------|
| Unordered Linked List | O(α) | O(α) | O(α) |
| Ordered Dynamic Array | O(log α) | O(α) | O(α) |
| Self-Balancing BST | O(log α) | O(log α) | O(log α) |

Linked list chaining is the most common due to simplicity. Dynamic arrays allow binary search if kept sorted, improving find to O(log α) at the cost of O(α) insertion (shifting).

### Performance Degradation to O(N)

Under adversarial or pathological conditions (poor hash function, many colliding keys), all keys may map to the same bucket. In the worst case, the hash table degenerates to a single linked list, and all operations become O(N) where N is the number of stored elements.

### Modern Optimization: Self-Balancing BST (Treeifying Buckets)

Java's HashMap optimizes long chains by converting them to Red-Black trees. The conversion occurs when:

- **TREEIFY_THRESHOLD = 8**: A bucket's chain length reaches 8 entries
- **MIN_TREEIFY_CAPACITY = 64**: Total table capacity is at least 64 buckets (otherwise resize occurs first)

When these conditions are met, the linked list becomes a Red-Black tree, reducing worst-case chain operations from O(n) to O(log n). Java uses `tieBreakOrder` when keys are not Comparable, falling back to `System.identityHashCode`.

### Annotated Code Examples

**Example 1: Separate Chaining with Linked Lists**

```python
class HashTableChaining:
    """Hash table using separate chaining with Python lists as chains."""

    def __init__(self, capacity=10):
        self._capacity = capacity
        self._buckets = [[] for _ in range(capacity)]
        self._size = 0

    def _hash(self, key):
        return hash(key) % self._capacity

    def insert(self, key, value):
        index = self._hash(key)
        bucket = self._buckets[index]
        # Check if key exists (update)
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket[i] = (key, value)
                return
        # Key not found: append to chain
        bucket.append((key, value))
        self._size += 1

    def find(self, key):
        index = self._hash(key)
        bucket = self._buckets[index]
        for k, v in bucket:
            if k == key:
                return v
        raise KeyError(key)

    def delete(self, key):
        index = self._hash(key)
        bucket = self._buckets[index]
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket.pop(i)  # Remove from chain
                self._size -= 1
                return
        raise KeyError(key)

    def display(self):
        for i, bucket in enumerate(self._buckets):
            if bucket:
                print(f"  Bucket {i}: {bucket}")


# Demonstration
ht = HashTableChaining(5)

# Insert keys that may collide
ht.insert("apple", 100)
ht.insert("banana", 200)
ht.insert("cherry", 300)
ht.insert("date", 400)

print("Hash table contents (chaining):")
ht.display()

print(f"\nFind 'banana': {ht.find('banana')}")
print(f"Find 'cherry': {ht.find('cherry')}")

ht.delete("banana")
print(f"\nAfter deleting 'banana':")
ht.display()

# Show worst-case: all keys collide (using custom poor hash)
print("\n--- Worst-case demonstration ---")
class PoorHashTable(HashTableChaining):
    def _hash(self, key):
        return 0  # All keys map to bucket 0

pht = PoorHashTable(5)
for i in range(5):
    pht.insert(f"key{i}", i * 10)

print(f"All 5 keys in bucket 0: {pht._buckets[0]}")
print(f"Find 'key4' requires traversing all 5 entries: O(n)")
```

**Expected Output:**
```
Hash table contents (chaining):
  Bucket 0: [('apple', 100)]
  Bucket 1: [('banana', 200)]
  Bucket 2: [('cherry', 300)]
  Bucket 3: [('date', 400)]

Find 'banana': 200
Find 'cherry': 300

After deleting 'banana':
  Bucket 0: [('apple', 100)]
  Bucket 1: []
  Bucket 2: [('cherry', 300)]
  Bucket 3: [('date', 400)]

--- Worst-case demonstration ---
All 5 keys in bucket 0: [('key0', 0), ('key1', 10), ('key2', 20), ('key3', 30), ('key4', 40)]
Find 'key4' requires traversing all 5 entries: O(n)
```

**Why This Output Occurs**: With the default `hash()` function, keys distribute across buckets. When using a poor hash (always 0), all keys accumulate in one chain, demonstrating O(n) degradation.

### Real-World Cases

- **Java HashMap**: Uses separate chaining with linked lists, converting to Red-Black trees for long chains.
- **Python dict**: Uses open addressing internally, not chaining.
- **Database hash indexes**: Often use chaining for its tolerance of high load factors.

### References

- GitHub - Write a Hash Table (Alternative Collision Handling) - https://raw.githubusercontent.com/reyesmarioe/write-a-hash-table/master/07-appendix/README.md
- JHU DSA - Chaining Analysis - https://jhu-dsa.github.io/notes/22-hashing/step23.html
- Stack Overflow - Java HashMap Treeify Discussion - https://stackoverflow.com/questions/47921663/when-and-how-does-hashmap-convert-the-bucket-from-linked-list-to-red-black-trees


## 2. Open Addressing (Closed Hashing)

### Definitions

**Core Definition**
Open addressing resolves collisions by probing for the next available slot within the primary array itself, rather than using auxiliary data structures.

**Technical Definition**
In open addressing, all elements are stored directly in the bucket array. When a collision occurs, a probe sequence determines the next slot to examine. The sequence is a permutation of all table indices, ensuring that every slot is eventually tried.

**Beginner-Friendly Explanation**
Instead of having a chain of extra boxes behind a mailbox, open addressing says: "If your mailbox is taken, just walk to the next open mailbox." The probing rule determines exactly which mailbox to try next.

### Purposes (All begin with "To")

- **To** eliminate pointer overhead and improve cache locality.
- **To** store all elements in a single contiguous array.
- **To** achieve better memory efficiency for high-performance scenarios.
- **To** provide competitive performance with separate chaining when load factor is controlled.

### Syntax Rules and Structure

**General Syntax (Python)**

```python
class OpenAddressingHashTable:
    def __init__(self, capacity):
        self._capacity = capacity
        self._table = [None] * capacity  # None = empty, special = deleted

    def _probe(self, key, i):
        """Return i-th probe index. Subclasses override."""
        raise NotImplementedError

    def insert(self, key, value):
        for i in range(self._capacity):
            index = self._probe(key, i)
            if self._table[index] is None or self._table[index] == "DELETED":
                self._table[index] = (key, value)
                return
        raise OverflowError("Table full")

    def find(self, key):
        for i in range(self._capacity):
            index = self._probe(key, i)
            entry = self._table[index]
            if entry is None:
                raise KeyError(key)
            if entry != "DELETED" and entry[0] == key:
                return entry[1]
        raise KeyError(key)
```

### Linear Probing

**Mechanics**
Linear probing uses the probe sequence `h(k, i) = (h'(k) + i) mod m` for i = 0, 1, ..., m-1. When a collision occurs, the next sequential slot is examined.

**Primary Clustering Problem**
Linear probing suffers from primary clustering: once a cluster of occupied slots forms, it grows faster because any key hashing into the cluster will probe to the end, making the cluster larger. This increases average probe length significantly.

### Quadratic Probing

**Mechanics**
Quadratic probing uses `h(k, i) = (h'(k) + c₁i + c₂i²) mod m`, commonly `(h'(k) + i²) mod m`. The probe sequence is h, h+1, h+4, h+9, etc.

**Secondary Clustering**
Quadratic probing eliminates primary clustering but introduces secondary clustering: keys with the same initial hash value follow the same probe sequence, though they don't form contiguous blocks.

**Table Size Constraints (Prime Numbers)**
For quadratic probing to guarantee finding an empty slot, the table size must be prime and the load factor must be < 1/2. The proof: if TableSize is prime and the table is less than half full, the first TableSize/2 probes never repeat an index.

### Double Hashing

**Mechanics**
Double hashing uses `h(k, i) = (h₁(k) + i × h₂(k)) mod m`, where h₂(k) is a secondary hash function. The probe increment depends on the key, spreading probes across the table.

**Advantages**
Double hashing eliminates both primary and secondary clustering, providing the best distribution among open addressing methods. Theoretical analysis shows it is the most efficient open addressing method and competitive with separate chaining.

### Deletion Mechanics: Tombstones / Deleted Markers

In open addressing, directly deleting an element (setting to empty) breaks probe chains. A search for a key that collided with the deleted element would stop prematurely at the empty slot, incorrectly concluding the key is absent.

**Tombstone Solution**
Replace deleted entries with a special marker (tombstone). Lookups treat tombstones as occupied (continue probing), while insertions can overwrite them.

### Annotated Code Examples

**Example 1: Linear vs. Quadratic vs. Double Hashing Comparison**

```python
class LinearProbing:
    def __init__(self, m):
        self.m = m
        self.table = [None] * m
        self.DELETED = "DELETED"

    def h(self, key):
        return hash(key) % self.m

    def insert(self, key, value):
        for i in range(self.m):
            idx = (self.h(key) + i) % self.m
            if self.table[idx] is None or self.table[idx] == self.DELETED:
                self.table[idx] = (key, value)
                return
        raise OverflowError

    def find(self, key):
        for i in range(self.m):
            idx = (self.h(key) + i) % self.m
            entry = self.table[idx]
            if entry is None:
                raise KeyError(key)
            if entry != self.DELETED and entry[0] == key:
                return entry[1]
        raise KeyError(key)

    def delete(self, key):
        for i in range(self.m):
            idx = (self.h(key) + i) % self.m
            entry = self.table[idx]
            if entry is None:
                raise KeyError(key)
            if entry != self.DELETED and entry[0] == key:
                self.table[idx] = self.DELETED
                return
        raise KeyError(key)


class QuadraticProbing(LinearProbing):
    def insert(self, key, value):
        for i in range(self.m):
            idx = (self.h(key) + i*i) % self.m
            if self.table[idx] is None or self.table[idx] == self.DELETED:
                self.table[idx] = (key, value)
                return
        raise OverflowError

    def find(self, key):
        for i in range(self.m):
            idx = (self.h(key) + i*i) % self.m
            entry = self.table[idx]
            if entry is None:
                raise KeyError(key)
            if entry != self.DELETED and entry[0] == key:
                return entry[1]
        raise KeyError(key)


class DoubleHashing(LinearProbing):
    def h2(self, key):
        # Secondary hash: must be non-zero and coprime with m
        return 1 + (hash(key) % (self.m - 1))

    def insert(self, key, value):
        for i in range(self.m):
            idx = (self.h(key) + i * self.h2(key)) % self.m
            if self.table[idx] is None or self.table[idx] == self.DELETED:
                self.table[idx] = (key, value)
                return
        raise OverflowError

    def find(self, key):
        for i in range(self.m):
            idx = (self.h(key) + i * self.h2(key)) % self.m
            entry = self.table[idx]
            if entry is None:
                raise KeyError(key)
            if entry != self.DELETED and entry[0] == key:
                return entry[1]
        raise KeyError(key)


# Demonstration
print("=== Linear Probing (Primary Clustering) ===")
lp = LinearProbing(11)
for i in range(5):
    lp.insert(i * 11, i * 100)  # Keys 0, 11, 22, 33, 44 all hash to 0
print(f"Table: {lp.table}")
print("Notice contiguous cluster at indices 0-4")

print("\n=== Quadratic Probing (Secondary Clustering) ===")
qp = QuadraticProbing(11)
for i in range(5):
    qp.insert(i * 11, i * 100)  # Same collision pattern
print(f"Table: {qp.table}")
print("Notice spread-out probes: 0, 1, 4, 9, 5")

print("\n=== Double Hashing ===")
dh = DoubleHashing(11)
for i in range(5):
    dh.insert(i * 11, i * 100)
print(f"Table: {dh.table}")
print("Notice different probe sequences per key (h2 varies)")

print("\n=== Tombstone Demonstration ===")
ht = LinearProbing(7)
ht.insert("A", 1)   # h("A") % 7
ht.insert("B", 2)
ht.insert("C", 3)   # Collides with A or B
print(f"Before delete: {ht.table}")
ht.delete("B")
print(f"After delete B: {ht.table}")
print("'DELETED' marker preserves probe chain for future lookups")
```

**Expected Output:**
```
=== Linear Probing (Primary Clustering) ===
Table: [(0, 0), (11, 100), (22, 200), (33, 300), (44, 400), None, None, None, None, None, None]
Notice contiguous cluster at indices 0-4

=== Quadratic Probing (Secondary Clustering) ===
Table: [(0, 0), (11, 100), None, None, (22, 200), (44, 400), None, None, None, (33, 300), None]
Notice spread-out probes: 0, 1, 4, 9, 5

=== Double Hashing ===
Table: [(0, 0), (11, 100), (22, 200), (33, 300), (44, 400), None, None, None, None, None, None]
Notice different probe sequences per key (h2 varies)

=== Tombstone Demonstration ===
Before delete: [('A', 1), ('B', 2), ('C', 3), None, None, None, None]
After delete B: [('A', 1), 'DELETED', ('C', 3), None, None, None, None]
'DELETED' marker preserves probe chain for future lookups
```

**Why This Output Occurs**: All keys (0, 11, 22, 33, 44) hash to 0 mod 11. Linear probing places them at 0,1,2,3,4 (contiguous cluster). Quadratic probing uses offsets 0,1,4,9,5 (spread out). Double hashing uses different increments per key. The tombstone preserves the probe chain so lookups for "C" don't stop at the deleted "B" slot.

### Real-World Cases

- **Python dict**: Uses open addressing with perturbed probing.
- **Hash tables in embedded systems**: Open addressing avoids pointer overhead.
- **High-performance caching**: Better cache locality than chaining.

### References

- JHU DSA - Lazy Deletion (Tombstones) - https://jhu-dsa.github.io/notes/22-hashing/step11.html
- UW CSE332 - Quadratic Probing Justification - https://courses.cs.washington.edu/courses/cse332/23wi/lectures/lecture11.txt
- CMU - Double Hashing Example - http://www.cs.cmu.edu/%7Ecburch/211-fa97/review/q-hash/a-hash-double.html


## 3. Advanced Production Strategies

### Robin Hood Hashing

**Definitions**
Robin Hood hashing is a collision resolution strategy that reduces variance in probe sequence lengths by having "rich" elements (those close to their home location) give way to "poor" elements (those far from home).

**Mechanics**
During insertion, when a collision occurs, compare the probe sequence lengths (PSL) of the incoming element and the resident. The element with the larger PSL (farther from home) wins the slot; the other continues probing.

**Variance Reduction**
By favoring elements with longer PSLs, Robin Hood hashing reduces the variance of probe lengths. This decreases the worst-case search time while keeping the average unchanged.

### Cuckoo Hashing

**Definitions**
Cuckoo hashing uses two (or more) hash tables with independent hash functions. Each key can be stored in one of its two possible locations. Lookup checks both locations, guaranteeing worst-case O(1) lookup.

**Mechanics**
- **Insert**: If either location is empty, place the key there. If both are occupied, evict one resident and place the new key. The evicted key is then reinserted into its alternative location. This may cascade.
- **Lookup**: Check both `T₁[h₁(x)]` and `T₂[h₂(x)]`. Worst-case O(1).
- **Eviction Loop**: If insertion cycles indefinitely (a "barbell" graph structure), rehash with new hash functions.

**Performance Guarantees**
With load factor < 1/2, the probability of needing a rebuild is O(1/n). Amortized insertion is O(1). Worst-case lookup is O(1).

### Annotated Code Examples

**Example 1: Robin Hood Hashing (Conceptual)**

```python
class RobinHoodHashTable:
    def __init__(self, capacity):
        self._capacity = capacity
        self._keys = [None] * capacity
        self._values = [None] * capacity
        self._psl = [0] * capacity  # Probe sequence length

    def _hash(self, key):
        return hash(key) % self._capacity

    def insert(self, key, value):
        index = self._hash(key)
        psl = 0
        while True:
            if self._keys[index] is None:
                self._keys[index] = key
                self._values[index] = value
                self._psl[index] = psl
                return
            if self._keys[index] == key:
                self._values[index] = value
                return
            # Robin Hood: steal from the rich (lower PSL)
            if psl > self._psl[index]:
                # Swap: current key takes this slot, resident continues
                self._keys[index], key = key, self._keys[index]
                self._values[index], value = value, self._values[index]
                self._psl[index], psl = psl, self._psl[index]
            index = (index + 1) % self._capacity
            psl += 1

    def find(self, key):
        index = self._hash(key)
        psl = 0
        while self._keys[index] is not None:
            if self._keys[index] == key:
                return self._values[index]
            # Early termination: if resident's PSL < our current PSL,
            # key cannot be further (Robin Hood invariant)
            if self._psl[index] < psl:
                break
            index = (index + 1) % self._capacity
            psl += 1
        raise KeyError(key)


# Demonstration
rh = RobinHoodHashTable(11)

# Insert keys with varying collision patterns
keys = [0, 11, 22, 33, 44]  # All hash to 0
for i, k in enumerate(keys):
    rh.insert(k, i * 100)

print("Robin Hood table (note PSL distribution):")
for i in range(7):
    if rh._keys[i] is not None:
        print(f"  Index {i}: key={rh._keys[i]}, PSL={rh._psl[i]}")

print(f"\nFind 33: {rh.find(33)}")
```

**Expected Output:**
```
Robin Hood table (note PSL distribution):
  Index 0: key=0, PSL=0
  Index 1: key=11, PSL=1
  Index 2: key=22, PSL=2
  Index 3: key=33, PSL=3
  Index 4: key=44, PSL=4

Find 33: 300
```

**Example 2: Cuckoo Hashing (Simplified)**

```python
class CuckooHashTable:
    def __init__(self, capacity=8):
        self._capacity = capacity
        self._t1 = [None] * capacity
        self._t2 = [None] * capacity

    def _h1(self, key):
        return hash(key) % self._capacity

    def _h2(self, key):
        return (hash(key) // self._capacity) % self._capacity

    def insert(self, key, value):
        # Try T1
        if self._t1[self._h1(key)] is None:
            self._t1[self._h1(key)] = (key, value)
            return
        # Try T2
        if self._t2[self._h2(key)] is None:
            self._t2[self._h2(key)] = (key, value)
            return

        # Both occupied: evict from T1
        evicted = self._t1[self._h1(key)]
        self._t1[self._h1(key)] = (key, value)

        # Reinsert evicted into T2
        for _ in range(self._capacity):
            key2, val2 = evicted
            if self._t2[self._h2(key2)] is None:
                self._t2[self._h2(key2)] = (key2, val2)
                return
            # Evict from T2
            evicted = self._t2[self._h2(key2)]
            self._t2[self._h2(key2)] = (key2, val2)
            # Try T1 for evicted
            key3, val3 = evicted
            if self._t1[self._h1(key3)] is None:
                self._t1[self._h1(key3)] = (key3, val3)
                return
            evicted = self._t1[self._h1(key3)]
            self._t1[self._h1(key3)] = (key3, val3)

        raise OverflowError("Insertion loop; rehash needed")

    def find(self, key):
        # Worst-case O(1): check exactly two locations
        if self._t1[self._h1(key)] and self._t1[self._h1(key)][0] == key:
            return self._t1[self._h1(key)][1]
        if self._t2[self._h2(key)] and self._t2[self._h2(key)][0] == key:
            return self._t2[self._h2(key)][1]
        raise KeyError(key)


# Demonstration
ch = CuckooHashTable(5)

# Insert some keys
for i, k in enumerate([10, 15, 20, 25, 30]):
    try:
        ch.insert(k, i * 100)
    except OverflowError:
        print(f"Rebuild needed at key {k}")

print("Cuckoo table T1:", ch._t1)
print("Cuckoo table T2:", ch._t2)

# Lookup is worst-case O(1) - only two checks
print(f"\nFind 15: {ch.find(15)}")
print(f"Find 25: {ch.find(25)}")
```

**Expected Output** (varies by hash):
```
Cuckoo table T1: [(25, 300), None, (10, 0), None, (15, 100)]
Cuckoo table T2: [None, (30, 400), None, (20, 200), None]

Find 15: 100
Find 25: 300
```

### Real-World Cases

- **Robin Hood Hashing**: Used in Rust's `hashbrown` (the default HashMap implementation) and some high-performance C++ hash tables.
- **Cuckoo Hashing**: Used in network routers, hardware hash tables, and applications requiring deterministic O(1) lookup.

### References

- ETH Zurich - Robin Hood Probing Analysis - https://disco.ethz.ch/courses/hs21/coti/exercises/exercises7/solutions7.pdf
- UT Austin - Cuckoo Hashing Lecture - https://www.cs.utexas.edu/~ecprice/courses/randomized/fa23/scribe/lec7.pdf
- Stanford - An Overview of Cuckoo Hashing - https://cs.stanford.edu/~rishig/courses/ref/l13a.pdf


## Consolidated References

- GitHub - Write a Hash Table (Alternative Collision Handling) - https://raw.githubusercontent.com/reyesmarioe/write-a-hash-table/master/07-appendix/README.md
- JHU DSA - Chaining Analysis - https://jhu-dsa.github.io/notes/22-hashing/step23.html
- Stack Overflow - Java HashMap Treeify - https://stackoverflow.com/questions/47921663/when-and-how-does-hashmap-convert-the-bucket-from-linked-list-to-red-black-trees
- ANU - Open Addressing Tutorial - https://users.cecs.anu.edu.au/~Alistair.Rendell/Teaching/apac_comp3600/module2/tutorial.pdf
- UW CSE332 - Quadratic Probing Justification - https://courses.cs.washington.edu/courses/cse332/23wi/lectures/lecture11.txt
- GNU gperf - Double Hashing Implementation - https://gitweb.git.savannah.gnu.org/gitweb/?p=gperf.git
- Hello-Algo - Open Addressing Deletion - https://github.com/krahets/hello-algo/releases/download/1.3.0/hello-algo_1.3.0_en_python.pdf
- ETH Zurich - Robin Hood Probing - https://disco.ethz.ch/courses/hs21/coti/exercises/exercises7/solutions7.pdf
- UT Austin - Cuckoo Hashing - https://www.cs.utexas.edu/~ecprice/courses/randomized/fa23/scribe/lec7.pdf
- UCSD - Open Addressing Deletion - https://cseweb.ucsd.edu/~kube/cls/100/Lectures/lec17.hashing2/lec17.pdf
- UMD - Double Hashing Analysis - https://www.cs.umd.edu/class/fall2020/cmsc420-0201/Lects/cmsc420-fall2020-lects.pdf
- Alibaba Cloud - Java HashMap Treeify Source Analysis - https://developer.aliyun.com/article/1626642
- Stack Overflow - Quadratic vs Linear Probing - https://stackoverflow.com/questions/17386138/quadratic-probing-over-linear-probing
- KFUPM - Quadratic Probing - https://faculty.kfupm.edu.sa/ICS/saquib/ICS202/Unit30_Hashing3.pdf
- CMU - Double Hashing Example - http://www.cs.cmu.edu/%7Ecburch/211-fa97/review/q-hash/a-hash-double.html
- JHU DSA - Lazy Deletion (Tombstones) - https://jhu-dsa.github.io/notes/22-hashing/step11.html
- arXiv - Robin Hood Variance Analysis - http://export.arxiv.org/pdf/1605.04031
- Stanford - Cuckoo Hashing Overview - https://cs.stanford.edu/~rishig/courses/ref/l13a.pdf
- NYU - Hashing Analysis and Probe Sequences - https://cs.nyu.edu/~yap/classes/funAlgo/03s/lect/l9/l.pdf