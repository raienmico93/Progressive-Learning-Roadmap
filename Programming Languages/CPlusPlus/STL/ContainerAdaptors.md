# C++ Container Adaptors & Custom Priorities — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Container adaptors are standard library class templates that wrap an existing sequence container and restrict its interface to provide a specific abstract data type — `std::stack` (LIFO), `std::queue` (FIFO), or `std::priority_queue` (heap-ordered). They do not own the underlying storage directly; instead, they delegate operations to an encapsulated container object.

**Technical Definition**
A container adaptor is a class template that takes a sequence container type as a template parameter and provides a limited set of member functions that map to operations on that underlying container. `std::stack<T, Container = std::deque<T>>` provides `push`, `pop`, `top`, `empty`, and `size`, delegating to `back`, `push_back`, and `pop_back` of the underlying container. `std::queue<T, Container = std::deque<T>>` provides `push`, `pop`, `front`, `back`, `empty`, and `size`, delegating to `push_back`, `pop_front`, `front`, and `back`. `std::priority_queue<T, Container = std::vector<T>, Compare = std::less<T>>` maintains a binary heap over the underlying container using the standard heap algorithms `std::make_heap`, `std::push_heap`, and `std::pop_heap`. All three adaptors are defined in `<stack>` and `<queue>`.

**Beginner-Friendly Explanation**
A container adaptor is like a specialised tool built on top of a general-purpose box. Imagine you have a `std::deque` — a flexible container that supports adding and removing at both ends. If you cover up all the openings except the back, you get a stack (last in, first out, like a stack of plates). If you cover up the front for removal and the back for insertion, you get a queue (first in, first out, like a line at a ticket counter). The adaptor doesn't create new storage — it just restricts how you can interact with the container you already have.

---

### Key Characteristics

- **Interface restriction**: Adaptors expose only the operations appropriate for their abstract data type; iterators are not provided.
- **Underlying container flexibility**: The backing container can be customised via a template parameter, subject to operation requirements.
- **Default backings**: `std::stack` and `std::queue` default to `std::deque`; `std::priority_queue` defaults to `std::vector`.
- **Heap-based priority**: `std::priority_queue` maintains a binary heap and uses a `Compare` predicate (default `std::less`) to determine priority.
- **Zero runtime abstraction overhead**: The adaptor layer is thin and fully inlined by the compiler; there is no virtual dispatch or type-erasure cost.
- **No iterator support**: Unlike sequence containers, adaptors deliberately omit `begin()`/`end()` to enforce their restricted interface.

---

### Prerequisites

- Familiarity with sequence containers (`std::deque`, `std::vector`, `std::list`) and their member functions.
- Understanding of iterators and template parameters.
- Basic knowledge of comparison predicates and strict weak ordering.
- (For `std::priority_queue` custom comparators) Familiarity with function objects and lambdas.
- A compiler supporting C++11 or later for `emplace` and lambda-based comparators.

---

### Related Programming Areas

- **Algorithm design**: LIFO/FIFO/heap structures underpin parsing, scheduling, and graph algorithms.
- **Operating systems**: Stack-based call management, queue-based process scheduling, priority-based interrupt handling.
- **Data structures and algorithms**: Heap sort, Dijkstra's algorithm, and Huffman coding rely on priority queues.
- **Template metaprogramming**: Adaptors demonstrate the power of compile-time container substitution.
- **Performance engineering**: Choosing the right backing container affects cache locality and allocation patterns.

---

### Core Concepts / Features

1. **Underlying Backings** — Restricting container interfaces using `std::stack` and `std::queue`.
2. **Priority Queues** — Heap-based sorting mechanisms in `std::priority_queue` with custom comparison predicates.
3. **Performance Overheads** — Recognizing when container adaptors introduce virtual layout overheads vs. compile-time stripping.

---

## 1. Underlying Backings

### Definitions

**Core Definition**
Underlying backing (or backing container) refers to the sequence container that a container adaptor wraps and delegates its operations to. The adaptor's behaviour is determined by the capabilities and performance characteristics of this backing container.

**Technical Definition**
`std::stack<T, Container>` requires `Container` to satisfy the SequenceContainer requirements and provide `back()`, `push_back()`, and `pop_back()`. `std::queue<T, Container>` additionally requires `front()` and `pop_front()`. The standard containers `std::deque` and `std::list` satisfy all requirements for both. `std::vector` satisfies the requirements for `std::stack` but not for `std::queue` (no `pop_front()`). The default backing for both `stack` and `queue` is `std::deque<T>`.

**Beginner-Friendly Explanation**
The backing container is the "engine" under the hood of the adaptor. A `std::stack` built on a `std::deque` grows efficiently without reallocation, while a `std::stack` built on a `std::vector` has better cache locality but may pay for reallocation when it grows. The adaptor itself adds no storage — it just forwards your operations to the container you chose.

---

### Sub-Feature: `std::stack` (LIFO)

#### Definitions

**Core Definition**
`std::stack` is a container adaptor that provides last-in, first-out (LIFO) semantics: elements are added and removed from the same end, called the top.

**Technical Definition**
`template<class T, class Container = std::deque<T>> class stack;`. It pushes and pops from the back of the underlying container. Only `back()`, `push_back()`, and `pop_back()` are required from `Container`. Since C++26, all member functions are `constexpr`. The adaptor does not provide iterators.

**Beginner-Friendly Explanation**
A stack is like a stack of plates — you put a new plate on top and take the top plate off. You can't reach the plates in the middle. This is useful for undo operations, expression evaluation, and backtracking.

#### Purposes (all begin with "To")

- **To** provide LIFO semantics for algorithms that require last-in, first-out processing.
- **To** restrict the interface of a sequence container to prevent accidental access to non-top elements.
- **To** allow the backing container to be swapped (e.g., `std::vector` for cache locality) without changing the algorithm's logic.
- **To** serve as the implementation backbone for recursive algorithms, expression parsers, and undo systems.
- **To** eliminate iterator-based access that could violate the stack abstraction.

#### Syntax Rules and Structure

```cpp
template<class T, class Container = std::deque<T>>
class stack;
```

**Component breakdown:**
- `T` — the type of stored elements.
- `Container` — the underlying sequence container (default `std::deque<T>`).

**Common operations:**
- `s.push(x)` / `s.emplace(args...)` — adds an element at the top (amortised O(1)).
- `s.pop()` — removes the top element (O(1)); does not return the value.
- `s.top()` — returns a reference to the top element (O(1)).
- `s.empty()` — checks if the stack is empty.
- `s.size()` — returns the number of elements.

**Constraints and limitations:**
- No iterators — cannot traverse the stack without popping.
- `pop()` returns `void`; to retrieve and remove, call `top()` then `pop()`.
- The backing container must provide `back()`, `push_back()`, and `pop_back()`.
- `std::vector` is a valid backing for `stack` but may reallocate on growth.

#### Annotated Code Example

```cpp
// File: stack_example.cpp
// Compile with: g++ -std=c++17 stack_example.cpp -o stack_example

#include <stack>
#include <iostream>
#include <vector>

int main() {
    // Step 1: Default stack (backed by std::deque)
    std::stack<int> s;

    // Step 2: Push elements
    s.push(10);
    s.push(20);
    s.push(30);

    // Step 3: Access top (LIFO — last pushed is on top)
    std::cout << "Top: " << s.top() << std::endl;  // 30

    // Step 4: Pop and print
    std::cout << "Popping: ";
    while (!s.empty()) {
        std::cout << s.top() << " ";
        s.pop();
    }
    std::cout << std::endl;

    // Step 5: Stack backed by std::vector (better cache locality)
    std::stack<int, std::vector<int>> vs;
    vs.push(1);
    vs.push(2);
    std::cout << "Vector-backed top: " << vs.top() << std::endl;

    return 0;
}
```

**Expected Output:**
```
Top: 30
Popping: 30 20 10
Vector-backed top: 2
```

**Why this output:** `std::stack` pushes and pops from the back of the underlying container. The last element pushed (`30`) is the top. Popping removes elements in reverse order of insertion (LIFO). The vector-backed stack behaves identically; only the underlying storage strategy differs.

#### Real-World Cases with Explanation

**Case 1 — Expression evaluation**: Compilers and calculators use stacks to evaluate postfix expressions and to match parentheses during parsing.

**Case 2 — Undo/redo systems**: Each action is pushed onto an undo stack; undoing pops the most recent action.

**Case 3 — Depth-first search**: Graph traversal uses a stack (explicit or via recursion) to track the path.

**Case 4 — Function call management**: The call stack of a running program is conceptually a stack of activation records.

---

### Sub-Feature: `std::queue` (FIFO)

#### Definitions

**Core Definition**
`std::queue` is a container adaptor that provides first-in, first-out (FIFO) semantics: elements are inserted at the back and removed from the front.

**Technical Definition**
`template<class T, class Container = std::deque<T>> class queue;`. It requires `front()`, `back()`, `push_back()`, and `pop_front()` from `Container`. `std::deque` and `std::list` satisfy these requirements; `std::vector` does not (no `pop_front()`). Since C++26, all member functions are `constexpr`. No iterators are provided.

**Beginner-Friendly Explanation**
A queue is like a line at a supermarket — the first person in line is the first to be served. New people join at the back. You can only see the person at the front and the person at the back, not those in the middle.

#### Purposes (all begin with "To")

- **To** provide FIFO semantics for algorithms that require first-in, first-out processing.
- **To** restrict a container's interface so that insertion and removal happen at opposite ends.
- **To** support breadth-first traversal of trees and graphs.
- **To** model real-world waiting lines in simulations and scheduling systems.
- **To** allow the backing container to be swapped (e.g., `std::list`) without changing the algorithm.

#### Syntax Rules and Structure

```cpp
template<class T, class Container = std::deque<T>>
class queue;
```

**Component breakdown:**
- `T` — element type.
- `Container` — underlying sequence container (default `std::deque<T>`).

**Common operations:**
- `q.push(x)` / `q.emplace(args...)` — adds at the back (amortised O(1)).
- `q.pop()` — removes the front element (O(1)); returns `void`.
- `q.front()` — reference to the front element.
- `q.back()` — reference to the back element.
- `q.empty()` / `q.size()` — capacity checks.

**Constraints and limitations:**
- No iterators.
- `pop()` returns `void`; retrieve via `front()` before popping.
- The backing container must support `pop_front()`; `std::vector` is not valid.
- No random access to middle elements.

#### Annotated Code Example

```cpp
// File: queue_example.cpp
// Compile with: g++ -std=c++17 queue_example.cpp -o queue_example

#include <queue>
#include <iostream>

int main() {
    // Step 1: Default queue (backed by std::deque)
    std::queue<std::string> q;

    // Step 2: Enqueue elements
    q.push("Alice");
    q.push("Bob");
    q.push("Charlie");

    // Step 3: Inspect front and back
    std::cout << "Front: " << q.front() << std::endl;  // Alice
    std::cout << "Back:  " << q.back()  << std::endl;  // Charlie

    // Step 4: Dequeue and print (FIFO order)
    std::cout << "Dequeuing: ";
    while (!q.empty()) {
        std::cout << q.front() << " ";
        q.pop();
    }
    std::cout << std::endl;

    // Step 5: Queue backed by std::list
    std::queue<int, std::list<int>> lq;
    lq.push(100);
    lq.push(200);
    std::cout << "List-backed front: " << lq.front() << std::endl;

    return 0;
}
```

**Expected Output:**
```
Front: Alice
Back:  Charlie
Dequeuing: Alice Bob Charlie
List-backed front: 100
```

**Why this output:** Elements are inserted at the back (`push`) and removed from the front (`pop`). The first element inserted (`Alice`) is the first removed (FIFO). The list-backed queue behaves identically.

#### Real-World Cases with Explanation

**Case 1 — Print spoolers**: Print jobs are queued in arrival order and processed sequentially.

**Case 2 — Breadth-first search**: BFS uses a queue to visit nodes level by level.

**Case 3 — Message queues**: Inter-process communication often uses FIFO queues for message delivery.

**Case 4 — Task scheduling**: Operating systems use queues to manage ready processes.

---

## 2. Priority Queues

### Definitions

**Core Definition**
`std::priority_queue` is a container adaptor that provides constant-time access to the largest (by default) element, with logarithmic-time insertion and extraction, using a binary heap.

**Technical Definition**
`template<class T, class Container = std::vector<T>, class Compare = std::less<typename Container::value_type>> class priority_queue;`. The adaptor requires `Container` to satisfy SequenceContainer and provide random-access iterators (LegacyRandomAccessIterator), plus `front()`, `push_back()`, and `pop_back()`. It maintains a heap by calling `std::make_heap` on construction and `std::push_heap` / `std::pop_heap` on modification. The `Compare` parameter defines a strict weak ordering; the element that is "greatest" according to `Compare` is at the top. The default `std::less` makes it a max-heap; `std::greater` makes it a min-heap.

**Beginner-Friendly Explanation**
A priority queue is like a hospital emergency room — patients are not treated in arrival order; the most critical case is seen first. Internally, it uses a binary heap stored in a contiguous array. The "top" element is always the one with the highest priority. You can insert new elements and remove the top, but you cannot access elements in the middle.

---

### Sub-Feature: Heap-Based Sorting Mechanisms

#### Definitions

**Core Definition**
The heap is a binary tree stored in an array where every parent is "greater" than its children according to a comparison predicate, enabling efficient retrieval of the maximum (or minimum) element.

**Technical Definition**
`std::priority_queue` maintains an implicit binary heap in the underlying container. The heap property is: for all `i`, `comp(c[i], c[parent(i)])` is `false` (i.e., the parent is not "less than" the child). In a max-heap with `std::less`, the root is the largest element. Insertion uses `std::push_heap` (percolate up), and extraction uses `std::pop_heap` (swap root with last, then percolate down). Both operations are O(log n). Top access is O(1).

**Beginner-Friendly Explanation**
A heap is like a tournament bracket where the winner (largest element) is always at the top. When you add a new contestant, they climb up the bracket until they find their level. When the winner leaves, the last contestant takes their place and sinks down until the bracket is valid again. All of this happens in an array, not a tree of pointers, so it is cache-friendly.

#### Purposes (all begin with "To")

- **To** provide O(1) access to the highest-priority element.
- **To** support O(log n) insertion and removal of the highest-priority element.
- **To** implement algorithms such as Dijkstra's shortest path, Huffman coding, and A* search.
- **To** maintain a dynamically changing set of elements ordered by priority.
- **To** allow custom priority definitions via a `Compare` predicate.

#### Syntax Rules and Structure

```cpp
template<class T, class Container = std::vector<T>,
         class Compare = std::less<typename Container::value_type>>
class priority_queue;
```

**Component breakdown:**
- `T` — element type.
- `Container` — underlying container (default `std::vector<T>`; must support random access).
- `Compare` — comparison predicate (default `std::less<T>`, giving a max-heap).

**Common operations:**
- `pq.push(x)` / `pq.emplace(args...)` — inserts and reheapifies (O(log n)).
- `pq.pop()` — removes the top element (O(log n)).
- `pq.top()` — returns the top element (O(1)).
- `pq.empty()` / `pq.size()` — capacity checks.

**Constraints and limitations:**
- No iterators; cannot traverse the heap.
- `pop()` returns `void`; retrieve via `top()` first.
- The `Compare` must define a strict weak ordering; violating this causes undefined behaviour.
- `std::vector<bool>` is a valid backing container but is a specialised bit-packed template.

#### Annotated Code Example

```cpp
// File: priority_queue_basic.cpp
// Compile with: g++ -std=c++17 priority_queue_basic.cpp -o priority_queue_basic

#include <queue>
#include <iostream>

int main() {
    // Step 1: Default max-heap
    std::priority_queue<int> max_pq;
    max_pq.push(30);
    max_pq.push(10);
    max_pq.push(50);
    max_pq.push(20);

    std::cout << "Max-heap order: ";
    while (!max_pq.empty()) {
        std::cout << max_pq.top() << " ";
        max_pq.pop();
    }
    std::cout << std::endl;

    // Step 2: Min-heap using std::greater
    std::priority_queue<int, std::vector<int>, std::greater<int>> min_pq;
    min_pq.push(30);
    min_pq.push(10);
    min_pq.push(50);
    min_pq.push(20);

    std::cout << "Min-heap order: ";
    while (!min_pq.empty()) {
        std::cout << min_pq.top() << " ";
        min_pq.pop();
    }
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
Max-heap order: 50 30 20 10
Min-heap order: 10 20 30 50
```

**Why this output:** With `std::less` (default), the heap puts the largest element at the top. With `std::greater`, the comparison is reversed, so the smallest element is at the top. Popping removes the top and reheapifies the remaining elements.

---

### Sub-Feature: Custom Comparison Predicates

#### Definitions

**Core Definition**
A custom comparison predicate is a user-provided function object or lambda that defines the priority ordering for a `std::priority_queue`.

**Technical Definition**
The `Compare` template parameter must be a type that is callable as `comp(a, b)` returning `bool`, and it must define a strict weak ordering. The element for which `comp(other, top)` is `false` for all other elements is the top. For a max-heap with a custom "priority" notion (e.g., smaller numbers mean higher priority), the predicate should return `true` when the first argument has lower priority than the second. The predicate is stored as a member and default-constructed unless a comparator instance is passed to the constructor.

**Beginner-Friendly Explanation**
Sometimes "largest number" is not the right priority. Maybe you want the smallest number first (min-heap), or the patient with the highest severity score, or the task with the earliest deadline. A custom comparator lets you define exactly what "higher priority" means.

#### Purposes (all begin with "To")

- **To** reverse the default ordering (min-heap instead of max-heap).
- **To** define priority based on a member of a struct or class (e.g., deadline, severity).
- **To** implement priority queues where "higher priority" means a smaller numeric value.
- **To** support lambda-based comparators for context-dependent priority logic.
- **To** use stateful comparators that capture external data.

#### Syntax Rules and Structure

```cpp
// Using a function object
struct MyCompare {
    bool operator()(const T& a, const T& b) const {
        return a.priority > b.priority;  // smaller value = higher priority
    }
};
std::priority_queue<T, std::vector<T>, MyCompare> pq;

// Using a lambda (must pass instance to constructor)
auto cmp = [](const T& a, const T& b) { return a.deadline > b.deadline; };
std::priority_queue<T, std::vector<T>, decltype(cmp)> pq(cmp);
```

**Constraints and limitations:**
- The comparator must define a strict weak ordering.
- A lambda comparator cannot be default-constructed, so it must be passed to the constructor.
- The comparator must not modify the elements it compares.
- Stateful comparators must be copy-constructible.

#### Annotated Code Example

```cpp
// File: priority_queue_custom.cpp
// Compile with: g++ -std=c++17 priority_queue_custom.cpp -o priority_queue_custom

#include <queue>
#include <iostream>
#include <string>
#include <vector>

// Step 1: A task with a deadline (smaller deadline = higher priority)
struct Task {
    std::string name;
    int deadline;  // smaller = more urgent
};

// Step 2: Custom comparator — urgent tasks first (min-heap on deadline)
struct UrgentFirst {
    bool operator()(const Task& a, const Task& b) const {
        return a.deadline > b.deadline;  // "greater" deadline = lower priority
    }
};

int main() {
    // Step 3: Priority queue with custom comparator
    std::priority_queue<Task, std::vector<Task>, UrgentFirst> pq;

    pq.push({"Write report", 5});
    pq.push({"Fix bug", 1});
    pq.push({"Attend meeting", 3});
    pq.push({"Review PR", 2});

    // Step 4: Process tasks in priority order
    std::cout << "Task order (most urgent first):" << std::endl;
    while (!pq.empty()) {
        const Task& t = pq.top();
        std::cout << "  " << t.name << " (deadline: " << t.deadline << ")"
                  << std::endl;
        pq.pop();
    }

    // Step 5: Lambda-based comparator (capturing a threshold)
    int threshold = 2;
    auto lambda_cmp = [threshold](const Task& a, const Task& b) {
        // Tasks at or below threshold are "urgent" and come first
        bool a_urgent = a.deadline <= threshold;
        bool b_urgent = b.deadline <= threshold;
        if (a_urgent != b_urgent) return !a_urgent;  // urgent first
        return a.deadline > b.deadline;  // among same urgency, smaller first
    };

    std::priority_queue<Task, std::vector<Task>, decltype(lambda_cmp)>
        pq2(lambda_cmp);

    pq2.push({"Write report", 5});
    pq2.push({"Fix bug", 1});
    pq2.push({"Attend meeting", 3});
    pq2.push({"Review PR", 2});

    std::cout << "\nLambda-ordered tasks:" << std::endl;
    while (!pq2.empty()) {
        const Task& t = pq2.top();
        std::cout << "  " << t.name << " (deadline: " << t.deadline << ")"
                  << std::endl;
        pq2.pop();
    }

    return 0;
}
```

**Expected Output:**
```
Task order (most urgent first):
  Fix bug (deadline: 1)
  Review PR (deadline: 2)
  Attend meeting (deadline: 3)
  Write report (deadline: 5)

Lambda-ordered tasks:
  Fix bug (deadline: 1)
  Review PR (deadline: 2)
  Attend meeting (deadline: 3)
  Write report (deadline: 5)
```

**Why this output:** The `UrgentFirst` comparator returns `true` when `a.deadline > b.deadline`, meaning a task with a larger deadline is "less than" a task with a smaller deadline. Therefore, the smallest deadline is at the top. The lambda comparator adds a threshold: tasks with `deadline <= 2` are treated as urgent and come before non-urgent tasks, regardless of their relative deadlines.

#### Real-World Cases with Explanation

**Case 1 — Operating system schedulers**: Real-time schedulers use priority queues to run the highest-priority task first, often with custom comparators for deadline-based ordering.

**Case 2 — Dijkstra's algorithm**: A min-heap priority queue (with `std::greater`) stores vertices by tentative distance, always extracting the closest unvisited vertex.

**Case 3 — Huffman coding**: A min-heap priority queue combines the two least frequent characters at each step to build an optimal prefix code.

**Case 4 — Event-driven simulation**: Events are stored in a priority queue ordered by simulation time; the next event to occur is always at the top.

---

## 3. Performance Overheads

### Definitions

**Core Definition**
Performance overhead in the context of container adaptors refers to any runtime cost introduced by the adaptor layer itself, as opposed to the cost of the underlying container operations. In practice, this overhead is effectively zero due to inlining and compile-time resolution.

**Technical Definition**
Container adaptors are class templates that contain an instance of the backing container as a member. All adaptor member functions are defined inline in the header and forward directly to the underlying container's member functions. There is no virtual dispatch, no type erasure, and no runtime indirection. The compiler resolves the adaptor's operations at compile time and typically inlines them entirely. Any perceived overhead is actually attributable to the backing container (e.g., `std::deque`'s block management vs. `std::vector`'s reallocation) rather than to the adaptor itself.

**Beginner-Friendly Explanation**
You might worry that wrapping a container in an adaptor adds a layer of "slowness." It doesn't. The adaptor's functions are so simple that the compiler replaces them entirely with direct calls to the underlying container. If you write `s.push(5)` on a `std::stack`, the compiler sees it as `c.push_back(5)` on the deque — no extra function call, no extra memory. The only performance differences come from which backing container you choose.

---

### Sub-Feature: Compile-Time Stripping vs. Virtual Layout Overheads

#### Definitions

**Core Definition**
Compile-time stripping (or zero-overhead abstraction) means the compiler eliminates the adaptor layer entirely, leaving only the underlying container's operations. Virtual layout overhead refers to the cost of runtime polymorphism (vtables, virtual dispatch), which adaptors do not incur.

**Technical Definition**
Since container adaptors are templates, their member functions are instantiated for each backing container type and are candidates for inlining. The resulting machine code is identical to what you would get by calling the backing container's methods directly. There is no vtable, no indirect call, and no type-erased wrapper. In contrast, a hypothetical runtime-polymorphic container adaptor (e.g., one holding a `std::unique_ptr<ContainerBase>`) would incur virtual dispatch overhead for every operation, which can be significant on superscalar architectures.

**Beginner-Friendly Explanation**
Adaptors are like a thin coat of paint on a car — the paint does not make the car heavier or slower. The engine (backing container) determines performance. If you instead built a "generic container" using virtual functions (runtime polymorphism), that would be like adding a heavy trailer — every operation would pay the cost of an indirect call.

#### Purposes (all begin with "To")

- **To** reassure developers that container adaptors introduce no runtime penalty compared to using the backing container directly.
- **To** demonstrate why template-based adaptors are preferable to runtime-polymorphic alternatives.
- **To** identify where the real performance costs lie (in the backing container, not the adaptor).
- **To** guide the choice of backing container for a given performance profile.
- **To** understand the trade-offs between compile-time flexibility (templates) and runtime flexibility (virtual dispatch).

#### Syntax Rules and Structure

There is no special syntax for achieving zero overhead — it is a property of the template design. The relevant rule is:

> Always choose the backing container based on the performance characteristics you need; the adaptor itself adds nothing.

**Backing container performance comparison for `std::stack`:**

| Backing | Push | Pop | Memory | Cache | Growth |
|---|---|---|---|---|---|
| `std::vector` | Amortised O(1) | O(1) | Minimal | Excellent | Reallocation copies all elements |
| `std::deque` | Amortised O(1) | O(1) | Block overhead | Good | No reallocation of existing elements |
| `std::list` | O(1) | O(1) | High (node pointers) | Poor | No reallocation |

**Constraints and limitations:**
- The zero-overhead property depends on the compiler's ability to inline; with optimisations disabled (`-O0`), there may be a small function-call cost.
- The adaptor cannot eliminate the backing container's inherent costs (e.g., `std::deque`'s block allocation).
- If you need runtime polymorphism (e.g., choosing the backing container at runtime), you must use virtual dispatch, which does introduce overhead.

#### Annotated Code Example

```cpp
// File: zero_overhead.cpp
// Compile with: g++ -O2 -std=c++17 zero_overhead.cpp -o zero_overhead
// Verify with: objdump -d zero_overhead | grep -A20 "main"

#include <stack>
#include <vector>
#include <deque>
#include <iostream>

// Step 1: A function using std::stack (adaptor)
void use_stack(std::stack<int>& s, int n) {
    for (int i = 0; i < n; ++i) {
        s.push(i);
    }
    while (!s.empty()) {
        s.pop();
    }
}

// Step 2: The same logic using std::vector directly (no adaptor)
void use_vector(std::vector<int>& v, int n) {
    for (int i = 0; i < n; ++i) {
        v.push_back(i);
    }
    while (!v.empty()) {
        v.pop_back();
    }
}

int main() {
    // Step 3: Both should generate nearly identical machine code
    std::stack<int, std::vector<int>> s;
    std::vector<int> v;

    use_stack(s, 1000000);
    use_vector(v, 1000000);

    std::cout << "Both operations completed." << std::endl;
    return 0;
}
```

**Expected Output:**
```
Both operations completed.
```

**Why this output:** The program runs both versions without error. At `-O2`, the generated assembly for `use_stack` and `use_vector` is effectively identical — the adaptor's `push` and `pop` are inlined to the vector's `push_back` and `pop_back`. There is no extra function call or memory access introduced by the adaptor.

#### Real-World Cases with Explanation

**Case 1 — High-frequency trading**: Low-latency systems use `std::stack` and `std::queue` backed by pre-allocated vectors to avoid dynamic allocation; the adaptor layer adds no measurable latency.

**Case 2 — Game engines**: Command queues (`std::queue`) and undo stacks (`std::stack`) are used per-frame; the zero-overhead adaptor ensures no frame-time impact.

**Case 3 — Embedded systems**: Memory-constrained devices choose `std::vector` as the backing for `std::stack` to avoid `std::deque`'s block-management overhead, while the adaptor itself uses no extra memory.

**Case 4 — Real-time audio processing**: Priority queues for event scheduling use custom comparators and `std::vector` backing; the adaptor's inlined operations keep jitter below microseconds.

---

## Summary Table

| Adaptor | Semantics | Default Backing | Required Backing Operations | Access | Insert | Remove | Iterators |
|---|---|---|---|---|---|---|---|
| `std::stack` | LIFO | `std::deque` | `back`, `push_back`, `pop_back` | `top()` O(1) | `push` amortised O(1) | `pop` O(1) | No |
| `std::queue` | FIFO | `std::deque` | `front`, `back`, `push_back`, `pop_front` | `front()`, `back()` O(1) | `push` amortised O(1) | `pop` O(1) | No |
| `std::priority_queue` | Heap (max by default) | `std::vector` | `front`, `push_back`, `pop_back`, random access | `top()` O(1) | `push` O(log n) | `pop` O(log n) | No |

---

## References

- std::stack — cppreference.com - https://en.cppreference.com/w/cpp/container/stack
- std::queue — cppreference.com - https://en.cppreference.com/w/cpp/container/queue
- std::priority_queue — cppreference.com - https://en.cppreference.com/w/cpp/container/priority_queue
- Container adaptors — cppreference.com - https://en.cppreference.com/w/cpp/container#Container_adaptors
- priority_queue Class — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/standard-library/priority-queue-class
- std::make_heap — cppreference.com - https://en.cppreference.com/w/cpp/algorithm/make_heap
- std::push_heap — cppreference.com - https://en.cppreference.com/w/cpp/algorithm/push_heap
- std::pop_heap — cppreference.com - https://en.cppreference.com/w/cpp/algorithm/pop_heap
- N3337: Working Draft, Standard for Programming Language C++ — ISO/IEC 14882:2011 - https://github.com/cplusplus/draft/raw/master/papers/n3337.pdf
- Boost Mailing List: Re: [boost] [gsoc] heaps & queues — Andrew Sutton - https://listarchives.boost.org/Archives/boost/2010/05/166171.php
- Add pop_value methods to container adaptors — P3136R0, Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3136r0.html
- Issue 2194: Impossible container requirements for adaptor types — Open-Std - https://isocpp.open-std.org/jtc1/sc22/wg21/docs/lwg-active.html#2194
- std::greater — cppreference.com - https://en.cppreference.com/w/cpp/utility/functional/greater
- Why does std::stack use std::deque by default? — Stack Overflow - https://stackoverflow.com/questions/102459/why-does-stdstack-use-stddeque-by-default
- C++ priority_queue with lambda comparator — Stack Overflow - https://stackoverflow.com/questions/16111337/declaring-a-priority-queue-in-c-with-a-custom-comparator