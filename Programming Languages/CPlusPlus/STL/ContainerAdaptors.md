# C++ Container Adaptors — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A container adaptor is a class template that provides a restricted, specialised interface on top of an existing sequence container, implementing a classic data structure such as a stack, queue, or priority queue.

**Technical Definition**
The C++ Standard Library defines three container adaptors: `std::stack`, `std::queue`, and `std::priority_queue`. Each adaptor takes a `Container` template parameter denoting the type of the underlying sequence container it adapts. The adaptor does not provide iterators, does not support direct element access beyond its specified interface, and enforces a specific access discipline (LIFO, FIFO, or priority-based). The underlying container must satisfy the `SequenceContainer` requirements, plus a small set of additional operations specific to each adaptor.

**Beginner-Friendly Explanation**
A container adaptor is like a specialised “shell” around a normal container. You give it a `vector`, `deque`, or `list`, and it hides most of that container’s functions, exposing only the operations that make sense for a stack (last in, first out), a queue (first in, first out), or a priority queue (highest priority first). This restriction prevents accidental misuse and makes the intent of the code clear.

### Key Characteristics

- **Restricted interface** – only a specific set of member functions is provided; iterators are deliberately absent.
- **Underlying container** – the adaptor stores its elements in a sequence container that can be customised via a template parameter.
- **No iteration** – you cannot traverse the elements; you can only access the element(s) designated by the adaptor’s discipline.
- **Efficient operations** – `stack` and `queue` provide O(1) insertion, removal, and access; `priority_queue` provides O(1) `top` and O(log n) insertion/removal.
- **Value semantics** – copying an adaptor copies its underlying container and all elements.

### Prerequisites

- Basic C++ syntax (variables, types, control flow).
- Familiarity with sequence containers (`std::vector`, `std::deque`, `std::list`).
- Understanding of templates and function objects (for `priority_queue` comparators).
- Awareness of Big-O notation.

### Related Programming Areas

- **Sequence containers** – adaptors are built on top of `deque`, `vector`, or `list`.
- **Algorithms** – `std::make_heap`, `std::push_heap`, and `std::pop_heap` underlie `priority_queue`.
- **Graph algorithms** – BFS uses `queue`, DFS uses `stack`, Dijkstra uses `priority_queue`.
- **Expression evaluation** – stacks are used for parsing and evaluating expressions.

### Core Concepts / Features

1. `std::stack`
2. `std::queue`
3. `std::priority_queue`
4. Adaptor operations
5. Adaptor best practices

---

## 1. `std::stack`

### Definitions

**Core Definition**
`std::stack` is a container adaptor that provides the functionality of a stack — specifically, a LIFO (last-in, first-out) data structure.

**Technical Definition**
Declared in `<stack>`:
```cpp
template<class T, class Container = std::deque<T>>
class stack;
```
The class template acts as a wrapper to the underlying container; only a specific set of functions is provided. The stack pushes and pops elements from the **back** of the underlying container, which is called the *top* of the stack. The underlying container must satisfy `SequenceContainer` and provide `back()`, `push_back()`, and `pop_back()`. All member functions are `constexpr` since C++26.

**Beginner-Friendly Explanation**
A stack is like a pile of plates: you add a plate to the top, and you remove the most recently added plate first. You can only see or touch the top plate — the ones underneath are hidden.

### Purposes

- To model LIFO behaviour with a clear, restricted interface.
- To prevent accidental access to elements other than the top.
- To allow the underlying container to be customised for performance or memory characteristics.
- To support algorithms and parsing tasks that naturally require last-in-first-out access.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <stack>
std::stack<T> s;                        // default: deque<T>
std::stack<T, Container> s;             // custom underlying container
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of elements stored. Must be the same as `Container::value_type`. |
| `Container` | The underlying sequence container; default `std::deque<T>`. |

**Member Function Complexity**

| Operation | Complexity | Notes |
|-----------|------------|-------|
| `push` / `emplace` | O(1) | Inserts at the top |
| `pop` | O(1) | Removes the top element |
| `top` | O(1) | Accesses the top element |
| `empty` | O(1) | Checks whether the stack is empty |
| `size` | O(1) | Returns the number of elements |

**Syntax Rules**
- `top()` returns a reference to the top element; calling it on an empty stack is undefined behaviour.
- `pop()` removes the top element but does not return it; call `top()` first if the value is needed.
- `push` inserts a copy (or moves) an existing object; `emplace` constructs the element directly in place.
- There are no iterators, no `begin()`/`end()`, and no direct access to elements below the top.

**Constraints and Limitations**
- No traversal or iteration over elements.
- Only the top element is accessible.
- The underlying container must support `back()`, `push_back()`, and `pop_back()`; `std::vector`, `std::deque`, and `std::list` all qualify.

### Annotated Code Examples

**Example 1: Basic stack operations**

```cpp
#include <stack>
#include <iostream>

int main() {
    std::stack<int> s;

    s.push(1);                             // bottom
    s.push(2);
    s.push(3);                             // top

    std::cout << s.top() << '\n';          // 3
    s.pop();                               // removes 3
    std::cout << s.top() << '\n';          // 2
}
```

**Expected Output**
```
3
2
```

**Why this output?**  
`push` adds elements in order; `top` returns the most recently pushed element (LIFO). After `pop`, the next element becomes the top.

**Example 2: Stack with `vector` as the underlying container**

```cpp
#include <stack>
#include <vector>
#include <iostream>

int main() {
    std::stack<int, std::vector<int>> s;

    s.push(10);
    s.push(20);

    std::cout << s.top() << '\n';          // 20
    std::cout << s.size() << '\n';         // 2
}
```

**Expected Output**
```
20
2
```

**Why this output?**  
The underlying container is `std::vector<int>`, but the interface remains LIFO. `top` still returns the last pushed element.

**Example 3: `emplace` for in-place construction**

```cpp
#include <stack>
#include <iostream>
#include <string>

int main() {
    std::stack<std::string> s;

    s.emplace("hello");                    // construct in place
    s.emplace("world");

    std::cout << s.top() << '\n';          // world
}
```

**Expected Output**
```
world
```

**Why this output?**  
`emplace` forwards arguments to the element’s constructor, constructing the string directly in the stack’s storage.

### Real-World Cases

- **Expression evaluation** – parsing and evaluating postfix expressions.
- **Undo/redo systems** – storing states for reversal.
- **Depth-first search (DFS)** – iterative graph traversal.
- **Syntax parsing** – matching parentheses and tags.

---

## 2. `std::queue`

### Definitions

**Core Definition**
`std::queue` is a container adaptor that provides the functionality of a queue — specifically, a FIFO (first-in, first-out) data structure.

**Technical Definition**
Declared in `<queue>`:
```cpp
template<class T, class Container = std::deque<T>>
class queue;
```
The queue pushes elements on the **back** of the underlying container and pops them from the **front**. The underlying container must satisfy `SequenceContainer` and provide `back()`, `front()`, `push_back()`, and `pop_front()`. Standard containers `std::deque` and `std::list` satisfy these requirements.

**Beginner-Friendly Explanation**
A queue is like a line at a ticket counter: the first person to join the line is the first to be served. You add people to the back of the line and serve them from the front.

### Purposes

- To model FIFO behaviour with a clear, restricted interface.
- To prevent access to elements other than the front and back.
- To support breadth-first search and task scheduling.
- To allow the underlying container to be customised.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <queue>
std::queue<T> q;                        // default: deque<T>
std::queue<T, Container> q;             // custom underlying container
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of elements stored. |
| `Container` | The underlying sequence container; default `std::deque<T>`. |

**Member Function Complexity**

| Operation | Complexity | Notes |
|-----------|------------|-------|
| `push` / `emplace` | O(1) | Inserts at the back |
| `pop` | O(1) | Removes the front element |
| `front` | O(1) | Accesses the front element |
| `back` | O(1) | Accesses the back element |
| `empty` / `size` | O(1) | |

**Syntax Rules**
- `front()` returns the oldest element; `back()` returns the newest.
- `pop()` removes the front element without returning it.
- `push` adds at the back; `emplace` constructs in place at the back.
- No iterators are provided; `begin()`/`end()` are not available.

**Constraints and Limitations**
- No traversal or iteration.
- Only the front and back elements are accessible.
- The underlying container must support `pop_front()`; `std::vector` cannot be used because it lacks `pop_front()`.

### Annotated Code Examples

**Example 1: Basic queue operations**

```cpp
#include <queue>
#include <iostream>

int main() {
    std::queue<int> q;

    q.push(1);                             // front
    q.push(2);
    q.push(3);                             // back

    std::cout << q.front() << '\n';        // 1
    std::cout << q.back() << '\n';         // 3

    q.pop();                               // removes 1
    std::cout << q.front() << '\n';        // 2
}
```

**Expected Output**
```
1
3
2
```

**Why this output?**  
`push` adds at the back; `front` returns the oldest element; `pop` removes the front element, revealing the next one.

**Example 2: Queue with `list` as the underlying container**

```cpp
#include <queue>
#include <list>
#include <iostream>

int main() {
    std::queue<int, std::list<int>> q;

    q.push(10);
    q.push(20);

    std::cout << q.front() << '\n';        // 10
    std::cout << q.back() << '\n';         // 20
}
```

**Expected Output**
```
10
20
```

**Why this output?**  
`std::list` satisfies the queue’s requirements, and the FIFO interface is preserved.

**Example 3: Using `emplace`**

```cpp
#include <queue>
#include <iostream>
#include <string>

int main() {
    std::queue<std::string> q;

    q.emplace("first");
    q.emplace("second");

    std::cout << q.front() << '\n';        // first
}
```

**Expected Output**
```
first
```

**Why this output?**  
`emplace` constructs the string directly at the back of the queue.

### Real-World Cases

- **Breadth-first search (BFS)** – exploring graph levels in order.
- **Printer spooling** – processing jobs in arrival order.
- **Message queues** – asynchronous communication between components.
- **Task scheduling** – fair, first-come-first-served execution.

---

## 3. `std::priority_queue`

### Definitions

**Core Definition**
`std::priority_queue` is a container adaptor that provides constant-time access to the largest (by default) element, at the cost of logarithmic-time insertion and extraction.

**Technical Definition**
Declared in `<queue>`:
```cpp
template<class T, class Container = std::vector<T>,
         class Compare = std::less<typename Container::value_type>>
class priority_queue;
```
The underlying container must satisfy `SequenceContainer` and provide random-access iterators, plus `front()`, `push_back()`, and `pop_back()`. The `Compare` parameter imposes a strict weak ordering; by default, `std::less<T>` makes the largest element the top. Using `std::greater<T>` makes the smallest element the top.

**Beginner-Friendly Explanation**
A priority queue is like a hospital emergency room: the most urgent case is treated first, regardless of arrival order. By default, the “largest” element has the highest priority.

### Purposes

- To model priority-based access with O(1) `top` and O(log n) insertion/removal.
- To support algorithms that repeatedly need the maximum (or minimum) element.
- To provide a safer alternative to manually managing a heap.
- To allow custom priority ordering via a comparator.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <queue>
std::priority_queue<T> pq;                          // max-heap
std::priority_queue<T, Container, Compare> pq;       // custom
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of elements stored. |
| `Container` | Underlying container; default `std::vector<T>`. |
| `Compare` | Strict weak ordering; default `std::less<T>` (max-heap). |

**Member Function Complexity**

| Operation | Complexity | Notes |
|-----------|------------|-------|
| `top` | O(1) | Accesses the highest-priority element |
| `push` / `emplace` | O(log n) | Inserts and restores heap order |
| `pop` | O(log n) | Removes the top element |
| `empty` / `size` | O(1) | |

**Syntax Rules**
- `top()` returns a const reference to the highest-priority element.
- `pop()` removes the top element but does not return it.
- The comparator must provide a strict weak ordering; an invalid comparator leads to undefined behaviour.
- With `std::greater<T>`, the smallest element becomes the top (min-heap).

**Constraints and Limitations**
- No iteration or direct access to elements other than the top.
- The underlying container must support random access; `std::list` cannot be used.
- Building a priority queue from a range is O(n), but individual insertions are O(log n).

### Annotated Code Examples

**Example 1: Max-heap (default)**

```cpp
#include <queue>
#include <iostream>

int main() {
    std::priority_queue<int> pq;

    pq.push(3);
    pq.push(1);
    pq.push(4);

    std::cout << pq.top() << '\n';         // 4 (largest)
    pq.pop();
    std::cout << pq.top() << '\n';         // 3
}
```

**Expected Output**
```
4
3
```

**Why this output?**  
The default comparator `std::less<int>` makes the largest element the top. After `pop`, the next largest element becomes the top.

**Example 2: Min-heap with `std::greater`**

```cpp
#include <queue>
#include <iostream>
#include <functional>

int main() {
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq;

    pq.push(3);
    pq.push(1);
    pq.push(4);

    std::cout << pq.top() << '\n';         // 1 (smallest)
}
```

**Expected Output**
```
1
```

**Why this output?**  
`std::greater<int>` reverses the ordering, so the smallest element has the highest priority.

**Example 3: Custom comparator for a struct**

```cpp
#include <queue>
#include <iostream>
#include <string>

struct Task {
    int priority;
    std::string name;
};

struct CompareTask {
    bool operator()(const Task& a, const Task& b) const {
        return a.priority < b.priority;    // lower number = lower priority
    }
};

int main() {
    std::priority_queue<Task, std::vector<Task>, CompareTask> pq;

    pq.push({2, "email"});
    pq.push({5, "emergency"});
    pq.push({1, "laundry"});

    std::cout << pq.top().name << '\n';    // emergency
}
```

**Expected Output**
```
emergency
```

**Why this output?**  
The custom comparator defines the priority ordering; the task with the highest `priority` value is at the top.

### Real-World Cases

- **Dijkstra’s shortest-path algorithm** – repeatedly extracting the nearest unvisited node.
- **Huffman coding** – merging the two lowest-frequency nodes.
- **Task schedulers** – executing the highest-priority task next.
- **Event simulation** – processing events in order of scheduled time.

---

## 4. Adaptor Operations

### Definitions

**Core Definition**
Adaptor operations are the member functions shared by all three container adaptors for inserting, removing, accessing, and querying elements within the constraints of their respective disciplines.

**Technical Definition**
All three adaptors provide a common core: `empty()`, `size()`, `push()`, `pop()`, `emplace()`, `swap()`, and assignment. `stack` additionally provides `top()`; `queue` provides `front()` and `back()`; `priority_queue` provides `top()`. None provide iterators or `begin()`/`end()`.

**Beginner-Friendly Explanation**
Even though stacks, queues, and priority queues behave differently, they share many of the same commands: you can check if they are empty, see how many items they hold, add an item, and remove an item. The difference is *which* item is added and removed, and how you access it.

### Common Interface

| Operation | `stack` | `queue` | `priority_queue` |
|-----------|---------|---------|------------------|
| `empty()` | ✓ | ✓ | ✓ |
| `size()` | ✓ | ✓ | ✓ |
| `push(value)` | ✓ (top) | ✓ (back) | ✓ (heap) |
| `emplace(args...)` | ✓ | ✓ | ✓ |
| `pop()` | ✓ (top) | ✓ (front) | ✓ (top) |
| `top()` | ✓ | — | ✓ |
| `front()` | — | ✓ | — |
| `back()` | — | ✓ | — |
| `swap(other)` | ✓ | ✓ | ✓ |

**Syntax Rules**
- `pop()` on all adaptors returns `void`; retrieve the element first via `top()` or `front()`.
- `push` and `emplace` add to the discipline-appropriate end: top for stack, back for queue, heap for priority_queue.
- `swap` exchanges the underlying containers and is generally O(1).

**Constraints and Limitations**
- No iterators: range-based `for` cannot be used directly on adaptors.
- No `clear()` member function in the standard (before C++23 proposals); to clear, assign an empty adaptor or pop all elements.
- The underlying container is protected, not public; you cannot modify it directly without subclassing.

### Annotated Code Example

```cpp
#include <stack>
#include <queue>
#include <iostream>

int main() {
    std::stack<int> s;
    s.push(1);
    s.push(2);
    s.emplace(3);
    std::cout << s.top() << ' ' << s.size() << '\n'; // 3 3

    std::queue<int> q;
    q.push(1);
    q.push(2);
    q.emplace(3);
    std::cout << q.front() << ' ' << q.back() << ' ' << q.size() << '\n'; // 1 3 3

    std::priority_queue<int> pq;
    pq.push(1);
    pq.push(3);
    pq.emplace(2);
    std::cout << pq.top() << ' ' << pq.size() << '\n'; // 3 3
}
```

**Expected Output**
```
3 3
1 3 3
3 3
```

**Why this output?**  
Each adaptor applies its own insertion/access discipline while sharing the `push`, `emplace`, `size`, and `pop` interface.

---

## 5. Adaptor Best Practices

### Definitions

**Core Definition**
Adaptor best practices are guidelines for choosing the right adaptor, using it safely, and understanding its performance and interface constraints.

**Technical Definition**
These practices are grounded in the C++ Core Guidelines (SL.con.2: prefer `vector` by default unless there is a reason to use another container) and community consensus: use adaptors to express intent clearly, choose the underlying container deliberately, and avoid operations that the adaptor does not support.

**Beginner-Friendly Explanation**
Best practices for adaptors are simple: use a stack when you need last-in-first-out, a queue for first-in-first-out, and a priority queue for highest-priority-first. Don’t try to iterate over an adaptor or access elements it doesn’t expose. Choose the underlying container based on your performance needs.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Use the adaptor that matches the discipline** | `stack` for LIFO, `queue` for FIFO, `priority_queue` for priority. |
| **Choose the underlying container deliberately** | `deque` is the default for `stack`/`queue`; `vector` for `priority_queue`. `vector` is viable for `stack` but not `queue` (lacks `pop_front`). |
| **Check `empty()` before `top()` / `front()`** | Calling `top()` or `front()` on an empty adaptor is undefined behaviour. |
| **Prefer `emplace` over `push`** | Constructs elements in place, avoiding temporaries. |
| **Do not rely on iteration** | Adaptors intentionally omit iterators; use the designated access function. |
| **Use `priority_queue` with a custom comparator for min-heaps** | Default is max-heap; `std::greater<T>` gives min-heap. |
| **Understand the cost model** | `stack`/`queue` are O(1) for all core operations; `priority_queue` is O(log n) for push/pop. |
| **Consider `deque` for `stack` with large elements** | `deque` never copies existing elements on growth, unlike `vector`. |
| **Avoid subclassing to expose the underlying container** | Protected member `c` is an implementation detail; use composition if you need more control. |

### Annotated Code Example

```cpp
#include <stack>
#include <queue>
#include <iostream>
#include <functional>

int main() {
    // Safe stack usage
    std::stack<int> s;
    s.push(10);
    s.push(20);
    if (!s.empty()) {
        std::cout << s.top() << '\n';      // 20
        s.pop();
    }

    // Min-heap priority queue
    std::priority_queue<int, std::vector<int>, std::greater<int>> min_pq;
    min_pq.push(3);
    min_pq.push(1);
    min_pq.push(4);
    if (!min_pq.empty()) {
        std::cout << min_pq.top() << '\n'; // 1
    }
}
```

**Expected Output**
```
20
1
```

**Why this output?**  
The empty check prevents undefined behaviour; the custom comparator makes the smallest element the top of the priority queue.

### Real-World Cases

- **Code reviews** – confirming that the adaptor matches the intended discipline.
- **Performance tuning** – choosing `deque` or `vector` as the underlying container based on element size and insertion patterns.
- **API design** – exposing only adaptor operations to prevent misuse.

---

## References

- `std::stack` — cppreference – https://en.cppreference.com/w/cpp/container/stack
- `std::queue` — cppreference – https://en.cppreference.com/w/cpp/container/queue
- `std::priority_queue` — cppreference – https://en.cppreference.com/w/cpp/container/priority_queue
- Containers library — cppreference – https://en.cppreference.com/w/cpp/container
- SequenceContainer named requirement — cppreference – https://en.cppreference.com/w/cpp/named_req/SequenceContainer
- C++ Core Guidelines — SL.con: Containers – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-stdlib
- Stack (abstract data type) — Wikipedia – https://en.wikipedia.org/wiki/Stack_(abstract_data_type)
- Queue (abstract data type) — Wikipedia – https://en.wikipedia.org/wiki/Queue_(abstract_data_type)
- Container adaptors — cppreference (Containers library overview) – https://en.cppreference.com/w/cpp/container#Container_adaptors

---

This cheat sheet provides a comprehensive, structured reference for C++ container adaptors, covering `std::stack`, `std::queue`, `std::priority_queue`, their operations, underlying container requirements, performance characteristics, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.