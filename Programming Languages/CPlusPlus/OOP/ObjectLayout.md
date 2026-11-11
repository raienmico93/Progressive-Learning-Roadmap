# Object Layout — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Object layout is the way a C++ compiler arranges an object's data members (and base class subobjects) in memory, including their order, alignment, padding, and any hidden implementation data (such as vtables).

**Technical Definition**
The C++ standard does **not** fully specify object layout; it specifies requirements (member ordering within access sections, alignment, standard-layout types, trivially copyable types) but leaves the exact layout to the implementation (governed by the platform ABI, e.g., the Itanium C++ ABI or the Microsoft ABI). Layout includes: non-static data members arranged in declaration order within each access section, padding inserted to satisfy alignment requirements, base class subobjects (with the **empty base optimization** for empty bases), and hidden data such as the vptr for polymorphic classes. Object size (`sizeof`) reflects this layout, including padding, and affects copying, moving, and ABI compatibility.

**Beginner-Friendly Explanation**
Object layout is **how an object is packed into memory**. Imagine packing a suitcase: you have items (data members) of different sizes, and you must arrange them so that each item is properly placed (alignment), sometimes adding filler (padding) to keep things tidy. The total size of the suitcase is the object's size. The compiler decides the exact arrangement, but C++ gives you some guarantees (members in declaration order within an access section, alignment requirements) and some tools (standard-layout types, `alignas`, `offsetof`) to reason about and control layout. Understanding layout matters for memory usage, performance (cache efficiency), serialization, and interfacing with C or hardware.

### Key Characteristics

- **Implementation-defined**: The exact layout is determined by the compiler/ABI, not fully by the standard.
- **Declaration order**: Within each access section, non-static data members are laid out in declaration order.
- **Padding**: Inserted to satisfy alignment requirements; increases object size.
- **Alignment**: Each member must be at an address that is a multiple of its alignment (`alignof`).
- **Empty base optimization (EBO)**: Empty base classes may occupy zero bytes.
- **Hidden data**: Polymorphic classes have a vptr; virtual/multiple inheritance adds more hidden data.
- **Standard-layout**: A category of types with predictable, C-compatible layout.
- **`sizeof` and `alignof`**: Reflect the layout; `offsetof` gives member offsets (for standard-layout types).

### Prerequisites

- Understanding of classes, data members, and inheritance.
- Familiarity with `sizeof`, `alignof`, and memory alignment.
- Knowledge of virtual functions and vtables (for polymorphic layout).
- Basic awareness of the platform ABI.

### Related Programming Areas

- **Memory management** – allocation size and alignment.
- **Serialization** – layout affects binary compatibility.
- **C interoperability** – standard-layout types have C-compatible layout.
- **Performance** – padding, cache lines, and false sharing.
- **ABI compatibility** – layout changes break binary compatibility.
- **Multiple inheritance and virtual inheritance** – complex layouts.

### Core Concepts / Features

1. Object Layout
2. Memory Layout
3. Data Member Ordering
4. Padding
5. Alignment
6. Empty Base Optimization (EBO)
7. Object Size
8. Object Copy
9. Object Move
10. Object Best Practices

---

## 1. Object Layout

### Definitions

**Core Definition**
Object layout is the arrangement of an object's data members, base class subobjects, and hidden implementation data in memory.

**Technical Definition**
Object layout is determined by the compiler according to the platform ABI and the standard's requirements. For a non-polymorphic, non-virtual class, the layout typically consists of non-static data members in declaration order (within each access section), with padding between them to satisfy alignment, and possibly padding at the end to make the object's size a multiple of its alignment. For polymorphic classes, a vptr is added (typically at the beginning). For classes with base classes, base subobjects are laid out (usually before derived members), with empty base optimization and virtual base pointers in more complex cases.

**Beginner-Friendly Explanation**
Object layout is the **blueprint of how an object sits in memory**: which member comes first, where the padding goes, where the vtable pointer lives, and how big the object is.

### Purposes

- To understand memory usage and cache behavior.
- To reason about binary compatibility and serialization.
- To optimize object size and performance.
- To interface with C and hardware.
- To diagnose alignment and padding issues.

### Syntax Rules and Structure

**Typical Layout (non-polymorphic)**
```
+------------------+
| member1          |
+------------------+
| padding          |
+------------------+
| member2          |
+------------------+
| ...              |
+------------------+
```

**Typical Layout (polymorphic)**
```
+------------------+
| vptr             |
+------------------+
| member1          |
+------------------+
| ...              |
+------------------+
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Data members | In declaration order (within access sections). |
| Padding | Inserted for alignment. |
| vptr | Hidden pointer for polymorphic classes. |
| Base subobjects | Laid out before derived members (typically). |

**Syntax Rules**
- Members are laid out in declaration order within each access section (the standard guarantees this for standard-layout types).
- Padding is inserted to satisfy alignment.
- The object's size is a multiple of its alignment.
- The vptr (if any) is implementation-defined but typically first.

**Constraints and Limitations**
- The exact layout is implementation-defined.
- Access sections may reorder members (pre-C++11 rules; C++11 guarantees order within a section).
- Virtual and multiple inheritance complicate layout.

### Annotated Code Examples

**Example 1: Observing layout with `offsetof`**

```cpp
#include <cstddef>
#include <iostream>

struct Point {
    int x;
    int y;
};

int main() {
    std::cout << "sizeof(Point) = " << sizeof(Point) << '\n';
    std::cout << "offsetof(x) = " << offsetof(Point, x) << '\n';
    std::cout << "offsetof(y) = " << offsetof(Point, y) << '\n';
}
```

**Expected Output** (typical, 4-byte `int`)
```
sizeof(Point) = 8
offsetof(x) = 0
offsetof(y) = 4
```

**Why this output?**  
`Point` has two `int` members laid out consecutively; `x` is at offset `0`, `y` at `4`, and the total size is `8`.

### Real-World Cases

- **Serialization** – layout determines binary format.
- **C interoperability** – standard-layout types match C structs.
- **Performance** – member ordering affects cache usage.
- **Memory debugging** – interpreting raw memory dumps.

---

## 2. Memory Layout

### Definitions

**Core Definition**
Memory layout is the broader arrangement of an object's bytes in memory, including data members, padding, hidden data, and any base class subobjects.

**Technical Definition**
Memory layout encompasses object layout plus the surrounding memory context: alignment of the object itself, placement within arrays or containers, and the effects of storage duration. It includes the layout of nested objects (members, bases) and the impact of inheritance (single, multiple, virtual). The memory layout of a type is described by its size, alignment, and the offsets of its members and base subobjects.

**Beginner-Friendly Explanation**
Memory layout is the **full picture** of how an object's bytes are arranged, including where it sits in memory and how its parts are positioned relative to each other.

### Purposes

- To understand memory usage and cache behavior.
- To reason about binary compatibility and serialization.
- To optimize data structures for performance.
- To interface with hardware and C APIs.

### Syntax Rules and Structure

**Layout Components**

| Component | Description |
|-----------|-------------|
| Data members | Fields of the object. |
| Padding | Alignment filler. |
| vptr | Polymorphic dispatch pointer. |
| Base subobjects | Inherited state. |
| Virtual base pointers | For virtual inheritance. |

**Syntax Rules**
- Layout is implementation-defined.
- `sizeof(T)` gives the object's size.
- `alignof(T)` gives its alignment.
- `offsetof(T, member)` gives a member's offset (standard-layout types only).
- `std::is_standard_layout_v<T>` indicates predictable layout.

**Constraints and Limitations**
- Non-standard-layout types have no portable layout guarantees.
- Virtual and multiple inheritance add hidden data.
- Bit-fields have implementation-defined layout.

### Annotated Code Examples

**Example 1: Layout with inheritance**

```cpp
#include <iostream>

struct Base {
    int a;
};

struct Derived : Base {
    int b;
};

int main() {
    std::cout << "sizeof(Base) = " << sizeof(Base) << '\n';
    std::cout << "sizeof(Derived) = " << sizeof(Derived) << '\n';
}
```

**Expected Output** (typical, 4-byte `int`)
```
sizeof(Base) = 4
sizeof(Derived) = 8
```

**Why this output?**  
`Derived` contains `Base`'s `a` (4 bytes) plus its own `b` (4 bytes), totaling `8`.

### Real-World Cases

- **ABI compatibility** – layout changes break binary compatibility.
- **Serialization** – layout determines binary format.
- **Performance** – layout affects cache behavior.
- **Debugging** – interpreting memory dumps.

---

## 3. Data Member Ordering

### Definitions

**Core Definition**
Data member ordering is the sequence in which non-static data members are placed in memory, typically in declaration order.

**Technical Definition**
Within each access section (`public`, `private`, `protected`), non-static data members are laid out in declaration order. The standard guarantees this for standard-layout types; for non-standard-layout types, the order across access sections is implementation-defined (though in practice, compilers usually preserve declaration order within each section). Reordering members can reduce padding and object size.

**Beginner-Friendly Explanation**
Data members are usually placed in the order you declare them. You can **reorder them** to reduce padding and make objects smaller.

### Purposes

- To understand the layout order.
- To optimize object size by reordering members.
- To ensure predictable layout for serialization.
- To avoid padding waste.

### Syntax Rules and Structure

**Declaration Order**
```cpp
struct A {
    int x;    // offset 0
    int y;    // offset 4
};            // size 8
```

**Reordering to Reduce Padding**
```cpp
struct B {
    char c;   // offset 0
    int i;    // offset 4 (padding at 1-3)
    char d;   // offset 8
};            // size 12 (padding at 9-11)

struct C {
    int i;    // offset 0
    char c;   // offset 4
    char d;   // offset 5
};            // size 8 (padding at 6-7)
```

**Syntax Rules**
- Members are laid out in declaration order (within access sections).
- Reordering can reduce padding.
- The standard guarantees order for standard-layout types.

**Constraints and Limitations**
- Reordering may break serialization or ABI compatibility.
- Access sections may affect order in non-standard-layout types.

### Annotated Code Examples

**Example 1: Reducing padding by reordering**

```cpp
#include <iostream>

struct Bad {
    char c;
    int i;
    char d;
};

struct Good {
    int i;
    char c;
    char d;
};

int main() {
    std::cout << "sizeof(Bad) = " << sizeof(Bad) << '\n';
    std::cout << "sizeof(Good) = " << sizeof(Good) << '\n';
}
```

**Expected Output** (typical, 4-byte `int`)
```
sizeof(Bad) = 12
sizeof(Good) = 8
```

**Why this output?**  
`Bad` has padding after `c` (3 bytes) and at the end (3 bytes), totaling `12`. `Good` orders members to minimize padding, totaling `8`.

### Real-World Cases

- **Memory-constrained systems** – reducing object size.
- **Cache efficiency** – grouping frequently accessed members.
- **Serialization** – predictable layout.

---

## 4. Padding

### Definitions

**Core Definition**
Padding is the extra bytes inserted between data members (or at the end of an object) to satisfy alignment requirements, increasing the object's size.

**Technical Definition**
Padding is added by the compiler so that each member is placed at an address that is a multiple of its alignment. Padding may appear **between members** (internal padding) and **at the end of the object** (tail padding) to make the object's size a multiple of its alignment (so arrays of the object are properly aligned). Padding bytes have unspecified values and should not be read.

**Beginner-Friendly Explanation**
Padding is the **filler** the compiler inserts to keep members properly aligned. It makes objects bigger but ensures efficient and correct access.

### Purposes

- To satisfy alignment requirements.
- To ensure proper alignment in arrays.
- To understand object size.
- To optimize memory usage by minimizing padding.

### Syntax Rules and Structure

**Padding Example**
```
struct X {
    char c;    // offset 0
    // 3 bytes padding
    int i;     // offset 4
    char d;    // offset 8
    // 3 bytes padding (tail)
};              // size 12
```

**Component Breakdown**

| Padding Type | Description |
|--------------|-------------|
| Internal padding | Between members for alignment. |
| Tail padding | At the end to make size a multiple of alignment. |

**Syntax Rules**
- Padding is inserted for alignment.
- Padding bytes are unspecified.
- Reordering members can reduce padding.
- `sizeof` includes padding.

**Constraints and Limitations**
- Padding bytes should not be read (they may be indeterminate).
- `memcmp` on structs with padding is unreliable.
- Serialization must account for padding.

### Annotated Code Examples

**Example 1: Observing padding**

```cpp
#include <cstddef>
#include <iostream>

struct X {
    char c;
    int i;
    char d;
};

int main() {
    std::cout << "sizeof(X) = " << sizeof(X) << '\n';
    std::cout << "offsetof(i) = " << offsetof(X, i) << '\n';
    std::cout << "offsetof(d) = " << offsetof(X, d) << '\n';
}
```

**Expected Output** (typical)
```
sizeof(X) = 12
offsetof(i) = 4
offsetof(d) = 8
```

**Why this output?**  
`i` is at offset `4` (3 bytes of padding after `c`), `d` at offset `8`, and the total size is `12` (3 bytes of tail padding).

### Real-World Cases

- **Serialization** – padding must be handled.
- **Memory optimization** – reordering to reduce padding.
- **Debugging** – interpreting memory dumps.

---

## 5. Alignment

### Definitions

**Core Definition**
Alignment is the requirement that an object's address be a multiple of its alignment, ensuring correct and efficient access.

**Technical Definition**
Every type has an alignment requirement (`alignof(T)`), which is the number of bytes between consecutive valid addresses for objects of that type. The compiler ensures members are aligned by inserting padding. Over-aligned types (alignment greater than the default maximum) require special allocation (`alignas`, aligned `operator new`). Misaligned access can cause crashes, performance penalties, or undefined behavior.

**Beginner-Friendly Explanation**
Alignment means placing objects at **nice round addresses** that are multiples of their size. For example, a 4-byte `int` should be at an address divisible by 4. The compiler inserts padding to make this happen.

### Purposes

- To ensure correct and efficient memory access.
- To satisfy hardware requirements.
- To enable SIMD and cache-line alignment.
- To support over-aligned types.

### Syntax Rules and Structure

**Alignment Facilities**
```cpp
alignof(T);              // query alignment
alignas(16) T x;         // specify alignment (C++11)
std::align(...);         // adjust pointer within buffer
```

**Component Breakdown**

| Facility | Purpose |
|----------|---------|
| `alignof(T)` | Query alignment requirement. |
| `alignas(N)` | Specify alignment. |
| Aligned allocation | Support over-aligned types. |

**Syntax Rules**
- Each type has a natural alignment.
- `alignas` can increase (not decrease) alignment.
- `alignof` returns the alignment in bytes.
- Over-aligned types require aligned allocation.

**Constraints and Limitations**
- Misaligned access is undefined behavior (on some platforms).
- `malloc` returns memory aligned for any fundamental type, but not over-aligned types.
- `alignas` cannot reduce alignment.

### Annotated Code Examples

**Example 1: Alignment and `alignas`**

```cpp
#include <iostream>

struct alignas(16) Vec4 {
    float x, y, z, w;
};

int main() {
    std::cout << "alignof(Vec4) = " << alignof(Vec4) << '\n';
    std::cout << "sizeof(Vec4) = " << sizeof(Vec4) << '\n';
}
```

**Expected Output**
```
alignof(Vec4) = 16
sizeof(Vec4) = 16
```

**Why this output?**  
`alignas(16)` forces 16-byte alignment; the size is padded to a multiple of the alignment.

### Real-World Cases

- **SIMD** – 16/32-byte alignment.
- **Cache lines** – 64-byte alignment to avoid false sharing.
- **Hardware registers** – fixed-address access.
- **Serialization** – ensuring consistent layout.

---

## 6. Empty Base Optimization (EBO)

### Definitions

**Core Definition**
Empty Base Optimization (EBO) is a compiler optimization that allows an **empty base class** to occupy **zero bytes** in a derived object.

**Technical Definition**
When a class derives from an empty base class (a class with no non-static data members, no virtual functions, and no virtual bases), the compiler may allocate zero bytes for the base subobject, provided it doesn't conflict with other requirements (e.g., two empty bases of the same type must have distinct addresses). EBO reduces object size and is widely used in the standard library (e.g., `std::allocator` as a base). Since C++20, `[[no_unique_address]]` extends similar optimization to members.

**Beginner-Friendly Explanation**
EBO means **empty base classes take up no space** in derived objects. It's a free optimization that makes types like allocators and policy classes zero-cost.

### Purposes

- To reduce object size for empty base classes.
- To enable zero-cost policy classes.
- To optimize standard library types.
- To support the empty base optimization idiom.

### Syntax Rules and Structure

**EBO Example**
```cpp
struct Empty {};

struct X : Empty {
    int value;
};
// sizeof(X) == sizeof(int) with EBO
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Empty base | Class with no data members. |
| EBO | Zero-byte base subobject. |
| Address uniqueness | Two empty bases of the same type must differ. |

**Syntax Rules**
- The base class must be empty (no non-static data members, no virtual functions, no virtual bases).
- The compiler may apply EBO (it's not guaranteed by the standard, but widely implemented).
- Two empty bases of the same type cannot both be at offset 0.
- `[[no_unique_address]]` (C++20) extends this to members.

**Constraints and Limitations**
- EBO is an optimization, not a guarantee (though widely implemented).
- Multiple empty bases of the same type may prevent EBO.
- The empty base's address must be distinct from other subobjects of the same type.

### Annotated Code Examples

**Example 1: EBO in action**

```cpp
#include <iostream>

struct Empty {};

struct X : Empty {
    int value;
};

int main() {
    std::cout << "sizeof(Empty) = " << sizeof(Empty) << '\n';
    std::cout << "sizeof(X) = " << sizeof(X) << '\n';
}
```

**Expected Output** (typical)
```
sizeof(Empty) = 1
sizeof(X) = 4
```

**Why this output?**  
`Empty` has size `1` (empty classes have size at least 1), but `X` has size `4` thanks to EBO — the empty base occupies zero bytes.

### Real-World Cases

- **Standard library** – allocators, comparators, and policies as empty bases.
- **Policy-based design** – zero-cost policy classes.
- **`[[no_unique_address]]`** – C++20 member optimization.

---

## 7. Object Size

### Definitions

**Core Definition**
Object size is the number of bytes an object occupies in memory, as reported by `sizeof`, including data members, padding, hidden data, and base subobjects.

**Technical Definition**
`sizeof(T)` returns the size of an object of type `T` in bytes, including padding and hidden data (e.g., vptr). The size is always a multiple of the alignment. For arrays, `sizeof(arr) = N * sizeof(T)`. Empty classes have size at least `1`. Object size affects memory usage, copying, and ABI compatibility.

**Beginner-Friendly Explanation**
Object size is **how many bytes an object takes up**. It includes everything: members, padding, and hidden data.

### Purposes

- To understand memory usage.
- To optimize object size.
- To reason about copying and moving costs.
- To ensure ABI compatibility.

### Syntax Rules and Structure

**Size Facilities**
```cpp
sizeof(T);                // size in bytes
sizeof(obj);              // size of object's type
std::is_empty_v<T>;       // check if empty
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `sizeof` | Size in bytes. |
| Alignment | Size is a multiple of alignment. |
| Empty class | Size at least 1. |

**Syntax Rules**
- `sizeof` is a compile-time constant.
- Size includes padding and hidden data.
- Empty classes have size at least 1.
- Arrays: `sizeof(arr) = N * sizeof(T)`.

**Constraints and Limitations**
- `sizeof` does not include dynamically allocated memory.
- Size may vary across platforms/ABIs.

### Annotated Code Examples

**Example 1: Object sizes**

```cpp
#include <iostream>

struct Empty {};
struct WithInt { int x; };
struct WithTwoInts { int x, y; };
struct WithVptr { virtual void f(); int x; };

int main() {
    std::cout << sizeof(Empty) << '\n';
    std::cout << sizeof(WithInt) << '\n';
    std::cout << sizeof(WithTwoInts) << '\n';
    std::cout << sizeof(WithVptr) << '\n';
}
```

**Expected Output** (typical, 64-bit)
```
1
4
8
16
```

**Why this output?**  
`Empty` has size `1`; `WithInt` is `4`; `WithTwoInts` is `8`; `WithVptr` includes a vptr (`8` bytes) plus `int` (`4`) plus padding (`4`), totaling `16`.

### Real-World Cases

- **Memory optimization** – reducing object size.
- **ABI compatibility** – size changes break binary compatibility.
- **Serialization** – size determines format.

---

## 8. Object Copy

### Definitions

**Core Definition**
Object copy is the process of creating a new object as a copy of an existing object, using the copy constructor or copy assignment operator.

**Technical Definition**
Object copy is performed by the copy constructor (`T(const T&)`) or copy assignment (`T& operator=(const T&)`). The compiler-generated versions perform member-wise copy (shallow copy for pointers). Copying includes padding bytes (which may be indeterminate) and hidden data (vptr is copied but points to the same vtable). For polymorphic objects, copying through a base reference slices the object.

**Beginner-Friendly Explanation**
Object copy is **duplicating an object**. It copies all members, including padding (which is why `memcmp` on structs with padding is unreliable). For classes with pointers, the default copy is shallow — you often need a deep copy.

### Purposes

- To create independent copies of objects.
- To support value semantics.
- To pass and return objects by value.
- To store objects in containers.

### Syntax Rules and Structure

**Copy Operations**
```cpp
T(const T& other);             // copy constructor
T& operator=(const T& other);  // copy assignment
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| Copy ctor | Creates a new object from an existing one. |
| Copy assignment | Replaces an existing object's contents. |
| Member-wise | Default behavior. |
| Deep copy | User-defined for owning resources. |

**Syntax Rules**
- The compiler generates copy operations unless suppressed.
- Member-wise copy includes padding (unspecified).
- Pointers are copied shallowly by default.
- Copying through a base reference slices the object.

**Constraints and Limitations**
- Shallow copy of owning pointers causes double deletion.
- `memcmp` on structs with padding is unreliable.
- Copying polymorphic objects by value slices them.

### Annotated Code Examples

**Example 1: Copy and padding**

```cpp
#include <cstring>
#include <iostream>

struct X {
    char c;
    int i;
};

int main() {
    X a{'a', 42};
    X b = a;
    std::cout << b.c << ' ' << b.i << '\n';
    // memcmp may fail due to padding
}
```

**Expected Output**
```
a 42
```

**Why this output?**  
`b = a` copies `c` and `i`. Padding bytes are also copied but their values are unspecified (which is why `memcmp` is unreliable).

### Real-World Cases

- **Value semantics** – copying objects in containers.
- **Serialization** – copying to buffers.
- **Deep copies** – classes owning resources.

---

## 9. Object Move

### Definitions

**Core Definition**
Object move is the process of transferring resources from a source object to a new object, leaving the source in a valid but unspecified state.

**Technical Definition**
Object move is performed by the move constructor (`T(T&&)`) or move assignment (`T& operator=(T&&)`). The compiler-generated versions perform member-wise move (which for pointers is a copy of the pointer value, not a deep move). Move operations are used when the source is a temporary or explicitly moved. Moving from an object leaves it in a valid but unspecified state; it should not be used except to reassign or destroy.

**Beginner-Friendly Explanation**
Object move is **transferring resources** from one object to another. It's faster than copying for resource-owning classes because it steals pointers instead of duplicating data.

### Purposes

- To efficiently transfer resources.
- To support move-only types.
- To enable efficient container operations.
- To return large objects efficiently.

### Syntax Rules and Structure

**Move Operations**
```cpp
T(T&& other) noexcept;              // move constructor
T& operator=(T&& other) noexcept;   // move assignment
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| Move ctor | Transfers resources from an rvalue. |
| Move assignment | Transfers resources into an existing object. |
| `noexcept` | Recommended. |
| Source state | Valid but unspecified. |

**Syntax Rules**
- Move operations take rvalue references.
- They should be `noexcept`.
- The source is left in a valid but unspecified state.
- Move operations are used for temporaries and `std::move` casts.

**Constraints and Limitations**
- Moving from `const` rvalues usually copies.
- Moved-from objects should not be used (except to reassign/destroy).
- Move operations do not apply to types without efficient resource transfer.

### Annotated Code Examples

**Example 1: Move constructor**

```cpp
#include <iostream>
#include <utility>

class Buffer {
    int* data_;
public:
    explicit Buffer(int v) : data_(new int(v)) {}
    ~Buffer() { delete data_; }
    Buffer(Buffer&& other) noexcept : data_(other.data_) {
        other.data_ = nullptr;
        std::cout << "moved\n";
    }
    int value() const { return *data_; }
};

int main() {
    Buffer a(42);
    Buffer b(std::move(a));
    std::cout << b.value() << '\n';
}
```

**Expected Output**
```
moved
42
```

**Why this output?**  
`Buffer b(std::move(a))` transfers `a`'s pointer to `b`, leaving `a` empty.

### Real-World Cases

- **`unique_ptr`** – move-only ownership.
- **`std::vector` reallocation** – moves elements.
- **Returning large objects** – move construction avoids copies.

---

## 10. Object Best Practices

### Definitions

**Core Definition**
Object best practices are guidelines for designing classes and using objects that respect layout, alignment, copying, and moving correctly and efficiently.

**Technical Definition**
These practices are based on the C++ Core Guidelines and community standards. Key principles: order members to minimize padding; use `alignas` for over-aligned types; prefer standard-layout types for C interoperability; avoid `memcmp` on types with padding; use EBO and `[[no_unique_address]]` for empty members; follow the Rule of Zero/Five for copy/move; and be aware of ABI compatibility.

**Beginner-Friendly Explanation**
Best practices for object layout and operations are simple: **order members to reduce padding**, **respect alignment**, **avoid `memcmp` on padded types**, **follow the Rule of Zero/Five**, and **be aware of ABI compatibility**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Order members** | Order members to minimize padding. | Larger members first. |
| **Alignment** | Respect alignment; use `alignas` for over-aligned types. | Aligned allocation. |
| **Standard-layout** | Use standard-layout types for C interoperability. | `is_standard_layout`. |
| **Avoid `memcmp`** | Padding bytes are unspecified. | Don't compare raw bytes. |
| **EBO / `[[no_unique_address]]`** | Use for empty members. | Zero-cost policies. |
| **Rule of Zero/Five** | Follow the appropriate special-member rule. | Rule of Zero preferred. |
| **ABI awareness** | Layout changes break binary compatibility. | Version APIs carefully. |

### Annotated Code Examples

**Example 1: Ordering members to minimize padding**

```cpp
#include <iostream>

struct Bad {
    char c;
    int i;
    char d;
};

struct Good {
    int i;
    char c;
    char d;
};

int main() {
    std::cout << "sizeof(Bad) = " << sizeof(Bad) << '\n';
    std::cout << "sizeof(Good) = " << sizeof(Good) << '\n';
}
```

**Expected Output** (typical)
```
sizeof(Bad) = 12
sizeof(Good) = 8
```

**Why this is best practice?**  
Ordering larger members first reduces padding, saving memory.

### Real-World Cases

- **Memory-constrained systems** – minimizing object size.
- **C interoperability** – standard-layout types.
- **ABI compatibility** – careful layout changes.
- **Serialization** – predictable layout.

---

## References

- Object layout (cppreference) – https://en.cppreference.com/w/cpp/language/objects
- Data members (cppreference) – https://en.cppreference.com/w/cpp/language/data_members
- `alignas` specifier (cppreference) – https://en.cppreference.com/w/cpp/language/alignas
- `alignof` operator (cppreference) – https://en.cppreference.com/w/cpp/language/alignof
- `offsetof` (cppreference) – https://en.cppreference.com/w/cpp/types/offsetof
- Empty base optimization (cppreference) – https://en.cppreference.com/w/cpp/language/ebo
- `[[no_unique_address]]` (cppreference) – https://en.cppreference.com/w/cpp/language/attributes/no_unique_address
- Standard-layout type (cppreference) – https://en.cppreference.com/w/cpp/named_req/StandardLayoutType
- C++ Core Guidelines — C.1: Organize related data into structures – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-org
- C++ Core Guidelines — C.2: Use `class` if the class has an invariant – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-struct
- C++ Core Guidelines — C.20: If you can avoid defining default operations, do – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-zero
- Itanium C++ ABI — Object layout – https://itanium-cxx-abi.github.io/cxx-abi/abi.html#layout
- Microsoft Learn — `alignas` and `alignof` – https://learn.microsoft.com/en-us/cpp/cpp/alignment-cpp-declarations
- Microsoft Learn — `sizeof` – https://learn.microsoft.com/en-us/cpp/cpp/sizeof-operator

---

This cheat sheet provides a complete, structured reference for C++ object layout, covering object layout, memory layout, data member ordering, padding, alignment, empty base optimization, object size, object copy, object move, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.