# C++ Parallel Algorithms — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
C++ parallel algorithms are overloads of standard library algorithms that accept an **execution policy** as their first argument, allowing the implementation to execute the algorithm's work in parallel (across threads) and/or in a vectorized manner.

**Technical Definition**
Since C++17, the standard library provides overloads for most algorithms in `<algorithm>` and `<numeric>` that take an execution policy object of type `std::execution::sequenced_policy`, `parallel_policy`, `parallel_unsequenced_policy`, or (since C++20) `unsequenced_policy`. The policy indicates the **kinds of parallelism permitted** (not guaranteed). Implementations are backed by parallel runtimes—GCC and Clang use Intel TBB; MSVC uses the Parallel Patterns Library (PPL) built into the runtime. The **Parallel STL** is the name commonly given to this combined facility.

**Beginner-Friendly Explanation**
Parallel algorithms let you take a standard algorithm like `std::sort` or `std::for_each` and tell the library “you may run this in parallel” simply by adding an extra argument at the front. You do not write threads or mutexes yourself; the runtime handles the work distribution. The catch is that any function you pass to a parallel algorithm must be safe to run concurrently.

### Key Characteristics

- **Single-argument opt-in** – parallelism is requested by prepending an execution policy: `std::sort(std::execution::par, first, last);`.
- **Policy indicates permission, not obligation** – the implementation may ignore the policy and run sequentially.
- **No manual thread management** – the runtime schedules and load-balances work.
- **Concurrency-safe callables required** – predicates, comparators, and operations must not introduce data races.
- **Vectorization-aware policies** – `unseq` and `par_unseq` permit SIMD execution, imposing stricter restrictions.
- **Runtime dependency** – GCC/Clang require Intel TBB (`-ltbb`); MSVC uses built-in PPL.

### Prerequisites

- Solid understanding of standard algorithms (`<algorithm>`, `<numeric>`).
- Familiarity with iterators and callables (lambdas, function objects).
- Knowledge of data races and thread safety.
- A compiler and runtime that support execution policies (C++17 for `seq`/`par`/`par_unseq`; C++20 for `unseq`).

### Related Programming Areas

- **Standard algorithms** – the algorithms being parallelized.
- **Threads and concurrency** – the underlying execution model.
- **Atomics** – safe lock-free operations allowed under strict policies.
- **SIMD** – vectorization permitted by `unseq`/`par_unseq`.
- **Senders/receivers (C++26)** – a more expressive successor model for asynchronous parallelism.

### Core Concepts / Features

1. Parallel algorithms (the facility)
2. Execution policies (`seq`, `par`, `par_unseq`, `unseq`)
3. Parallel STL (implementations and backends)
4. Parallel algorithms best practices

---

## 1. Parallel Algorithms

### Definitions

**Core Definition**
A parallel algorithm is a standard algorithm overload that accepts an execution policy and may execute its work concurrently across threads and/or vector lanes.

**Technical Definition**
An algorithm overload of the form `algorithm(ExecutionPolicy&& policy, ...)` participates in the parallel algorithms facility. The policy argument selects the overload and communicates the permissible execution strategy. The algorithm's semantics (result, complexity guarantees) are identical to the non-policy overload; only the allowed execution strategy changes.

**Beginner-Friendly Explanation**
A parallel algorithm is just a normal algorithm that you have given permission to “go faster” by using multiple cores or vector instructions.

### Purposes

- To parallelize standard algorithms with minimal code changes.
- To exploit multi-core CPUs without writing thread-management code.
- To enable vectorization for data-parallel workloads.
- To provide a portable, standard interface across implementations.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <algorithm>
#include <execution>

std::algorithm_name(std::execution::policy, first, last, ...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `policy` | An execution policy object (`seq`, `par`, `par_unseq`, `unseq`). |
| `first`, `last` | Iterators defining the range. |
| Additional arguments | Predicates, comparators, output iterators, etc. |

**Constraints and Limitations**
- The policy overload is selected by overload resolution; omitting the policy selects the sequential version.
- The implementation may ignore the policy and execute sequentially.
- Only algorithms with policy overloads participate (most of `<algorithm>` and `<numeric>`, but not all).

### Annotated Code Example

```cpp
#include <algorithm>
#include <execution>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v(1'000'000, 1);
    std::for_each(std::execution::par, v.begin(), v.end(),
                  [](int& x) { x *= 2; });
    std::cout << v[0] << '\n';             // 2
}
```

**Expected Output**
```
2
```

**Why this output?**  
The parallel `for_each` doubles every element; the result is the same as sequential execution, but the work may be distributed across threads.

---

## 2. Execution Policies

### Definitions

**Core Definition**
An execution policy is an object that specifies the kind of parallelism an algorithm is permitted to use.

**Technical Definition**
`<execution>` defines four policy types and their inline `constexpr` instances: `sequenced_policy` (`seq`, C++17), `parallel_policy` (`par`, C++17), `parallel_unsequenced_policy` (`par_unseq`, C++17), and `unsequenced_policy` (`unseq`, C++20). The policy type is used for overload disambiguation; the instance communicates the execution strategy.

**Beginner-Friendly Explanation**
An execution policy is a “permission slip” that tells the algorithm how much parallelism it may use—none, threads, threads plus vectorization, or vectorization alone.

### 2.1 `std::execution::seq`

#### Definitions

**Core Definition**
`seq` permits only sequential, in-order execution on the calling thread.

**Technical Definition**
`std::execution::seq` is an instance of `sequenced_policy`. It requires the algorithm to execute sequentially in the calling thread, in element order. It is the baseline against which parallel policies are compared.

#### Purposes

- To explicitly request sequential execution.
- To provide a baseline for benchmarking.
- To disable parallelism without changing the algorithm call site.

#### Syntax Rules and Structure

**General Syntax**
```cpp
std::sort(std::execution::seq, v.begin(), v.end());
```

**Constraints and Limitations**
- No parallelism; identical to the non-policy overload.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <execution>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};
    std::sort(std::execution::seq, v.begin(), v.end());
    for (int x : v) std::cout << x << ' '; // 1 1 3 4 5
}
```

**Expected Output**
```
1 1 3 4 5
```

**Why this output?**  
`seq` sorts sequentially; the result is the same as the default.

---

### 2.2 `std::execution::par`

#### Definitions

**Core Definition**
`par` permits parallel execution across multiple threads, but does **not** permit vectorization.

**Technical Definition**
`std::execution::par` is an instance of `parallel_policy`. It allows the algorithm to execute across multiple threads. The element access functions (predicates, comparators, operations) must not cause data races. Vectorization is **not** permitted, so mutexes and other synchronization primitives may be used (though they may serialize execution).

#### Purposes

- To parallelize CPU-bound work across cores.
- To allow the use of synchronization primitives in callables.
- To avoid the stricter restrictions of `par_unseq`.

#### Syntax Rules and Structure

**General Syntax**
```cpp
std::for_each(std::execution::par, v.begin(), v.end(), f);
```

**Constraints and Limitations**
- Callables must be free of data races.
- Vectorization is not permitted.
- Mutexes may be used, but may reduce parallel speedup.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <execution>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v(1000, 1);
    std::for_each(std::execution::par, v.begin(), v.end(),
                  [](int& x) { x = x * x; });
    std::cout << v[0] << '\n';             // 1
}
```

**Expected Output**
```
1
```

**Why this output?**  
`par` executes the lambda across threads; `1 * 1` is `1`.

---

### 2.3 `std::execution::par_unseq`

#### Definitions

**Core Definition**
`par_unseq` permits parallel execution across threads **and** vectorization (SIMD) within each thread.

**Technical Definition**
`std::execution::par_unseq` is an instance of `parallel_unsequenced_policy`. It permits both multi-threaded execution and vectorized (interleaved) execution within a thread. This **forbids** operations that are unsafe under vectorization: mutexes cannot be used, memory allocation/deallocation is not allowed, and only lock-free atomic specializations may be used.

#### Purposes

- To enable the highest level of parallelism (threads + SIMD).
- To accelerate data-parallel loops.
- To exploit both multi-core and vector hardware.

#### Syntax Rules and Structure

**General Syntax**
```cpp
std::for_each(std::execution::par_unseq, v.begin(), v.end(), f);
```

**Restrictions (Strict)**
- No `std::mutex` or other locking primitives.
- No dynamic memory allocation (`new`, `malloc`).
- Only lock-free `std::atomic` specializations.
- No operations that rely on sequential ordering or thread-local state.

**Constraints and Limitations**
- Violating the restrictions leads to undefined behaviour.
- Not all algorithms benefit from vectorization.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <execution>
#include <vector>
#include <iostream>

int main() {
    std::vector<float> v(10000, 2.0f);
    std::for_each(std::execution::par_unseq, v.begin(), v.end(),
                  [](float& x) { x = x * x; });   // safe: no locks, no allocation
    std::cout << v[0] << '\n';             // 4
}
```

**Expected Output**
```
4
```

**Why this output?**  
The lambda is vectorization-safe (no mutexes, no allocation); `2.0 * 2.0` is `4.0`.

---

### 2.4 `std::execution::unseq` (C++20)

#### Definitions

**Core Definition**
`unseq` permits vectorized (SIMD) execution on the **calling thread**, without multi-threading.

**Technical Definition**
`std::execution::unseq` (C++20) is an instance of `unsequenced_policy`. It allows the algorithm to be vectorized—executed on a single thread using SIMD instructions—but does **not** permit multi-threading. It imposes the same restrictions as `par_unseq` (no mutexes, no allocation, only lock-free atomics).

#### Purposes

- To enable vectorization without thread overhead.
- To accelerate small-to-medium data-parallel workloads.
- To provide a single-threaded SIMD execution policy.

#### Syntax Rules and Structure

**General Syntax**
```cpp
std::for_each(std::execution::unseq, v.begin(), v.end(), f);
```

**Constraints and Limitations**
- Same restrictions as `par_unseq`.
- Executes on the calling thread only.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <execution>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v(1000, 5);
    std::for_each(std::execution::unseq, v.begin(), v.end(),
                  [](int& x) { x += 1; });
    std::cout << v[0] << '\n';             // 6
}
```

**Expected Output**
```
6
```

**Why this output?**  
`unseq` vectorizes the addition on the calling thread; `5 + 1 = 6`.

---

### Execution Policy Comparison

| Policy | Threads | Vectorization | Mutexes Allowed? | Allocation Allowed? | Standard |
|--------|---------|---------------|------------------|---------------------|----------|
| `seq` | No | No | Yes | Yes | C++17 |
| `par` | Yes | No | Yes | Yes | C++17 |
| `par_unseq` | Yes | Yes | **No** | **No** | C++17 |
| `unseq` | No | Yes | **No** | **No** | C++20 |

---

## 3. Parallel STL

### Definitions

**Core Definition**
Parallel STL is the implementation of the C++ standard algorithms with execution policy support, typically backed by a parallel runtime.

**Technical Definition**
Parallel STL implementations provide the policy overloads. GCC’s `libstdc++` and LLVM’s `libc++` use Intel TBB (Threading Building Blocks) as the threading backend; MSVC uses its built-in Parallel Patterns Library (PPL). The implementation may use OpenMP SIMD directives for vectorization.

**Beginner-Friendly Explanation**
Parallel STL is the “engine” behind the policies. On GCC and Clang, it uses TBB, so you must link with `-ltbb`. On MSVC, it is built in.

### Purposes

- To provide the runtime support for execution policies.
- To map standard algorithm calls to parallel work-stealing schedules.
- To enable vectorization where permitted.

### Compiler and Backend Support

| Compiler | Backend | Required Flags |
|----------|---------|----------------|
| GCC | Intel TBB | `-ltbb` |
| Clang | Intel TBB | `-ltbb` |
| MSVC | Built-in PPL | None |

### Annotated Code Example

```cpp
// Compile with: g++ -std=c++17 -O3 -ltbb program.cpp
#include <algorithm>
#include <execution>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v(1'000'000);
    std::iota(v.begin(), v.end(), 0);
    std::sort(std::execution::par, v.begin(), v.end(), std::greater<>{});
    std::cout << v[0] << '\n';             // 999999
}
```

**Expected Output**
```
999999
```

**Why this output?**  
`std::sort` with `par` sorts the vector in descending order; the first element is the largest value.

---

## 4. Parallel Algorithms Best Practices

### Definitions

**Core Definition**
Parallel algorithms best practices are guidelines for using execution policies correctly, safely, and efficiently, ensuring data-race-free callables and measuring before committing.

**Technical Definition**
These practices are informed by the C++ Core Guidelines (CP.par: Parallelism; CP.1: Assume multi-threaded use; CP.2: Avoid data races) and community consensus: start with `seq`, measure, then try `par`; only use `par_unseq`/`unseq` when the callable is vectorization-safe; avoid parallelizing small workloads; and never assume speedup without benchmarking.

**Beginner-Friendly Explanation**
Best practices for parallel algorithms are simple: start sequential, measure, then add `par`; only use `par_unseq` or `unseq` if your lambda is safe for vectorization (no locks, no allocation); and don’t parallelize tiny loops.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **CP.par** | Prefer parallel algorithms when parallelism is appropriate; otherwise sequential. |
| **CP.1 / CP.2** | Assume multi-threaded use; avoid data races in callables. |
| **Start with `seq`** | Establish a baseline and a correctness reference. |
| **Try `par` before `par_unseq`** | `par` is safer; use `par_unseq` only when justified. |
| **`par_unseq`/`unseq`: no locks, no allocation** | Mutexes and `new` are forbidden. |
| **Benchmark before committing** | Parallel overhead can outweigh gains for small inputs (typically < ~10K elements). |
| **Avoid data races** | Callables must not share mutable state without synchronization. |
| **Use lock-free atomics with `par_unseq`** | Only lock-free specializations are permitted. |
| **Do not assume speedup** | Policy is a permission; the implementation may run sequentially. |
| **Validate with tools** | ThreadSanitizer catches races in parallel callables. |

### Annotated Code Example

```cpp
#include <algorithm>
#include <execution>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {5, 3, 1, 4, 2};

    // Baseline: sequential
    std::sort(std::execution::seq, v.begin(), v.end());

    // Safe parallel: no shared mutable state in the comparator
    std::sort(std::execution::par, v.begin(), v.end(), std::greater<>{});

    for (int x : v) std::cout << x << ' '; // 5 4 3 2 1
}
```

**Expected Output**
```
5 4 3 2 1
```

**Why this output?**  
The sequential sort orders the vector ascending; the parallel sort with `std::greater<>` orders it descending. The comparator is stateless and data-race-free.

### Real-World Cases

- **Data processing** – parallel `for_each` or `transform` over large datasets.
- **Sorting** – parallel `sort` on large arrays.
- **Numerical computation** – parallel `reduce` and `transform_reduce`.
- **Image processing** – parallel pixel operations with `par`.

---

## References

- `std::execution::seq, std::execution::par, std::execution::par_unseq, std::execution::unseq` — cppreference — https://en.cppreference.com/cpp/algorithm/execution_policy_tag
- `<execution>` header — cppreference — https://en.cppreference.com/w/cpp/header/execution
- C++ Core Guidelines — CP: Concurrency and parallelism — https://cpp-core-guidelines-docs.vercel.app/concurrency
- C++ Core Guidelines — CP.par: Parallelism — https://cpp-core-guidelines-docs.vercel.app/concurrency
- Use parallel execution policies (C++17) for algorithm parallelism — Dr-Sergey/learn_cpp — https://github.com/Dr-Sergey/learn_cpp/blob/main/08_Standard_Library_Algorithms/Use_parallel_execution_policies_C++17_for_algorithm_parallelism.md
- Intel’s Parallel STL contribution proposal (TBB backend) — GCC libstdc++ mailing list — https://gcc.gnu.org/pipermail/libstdc++/2017-November/046601.html
- Parallel STL (LLVM) — https://llvm.googlesource.com/parallelstl/
- Algorithms with parallelized versions — Microsoft Learn — https://learn.microsoft.com/en-us/cpp/standard-library/algorithms-parallel
- P0394R4: Hotel Parallelifornia — terminate() for Parallel Algorithms Exception Handling — https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0394r4.html
- P2500R0: C++17 parallel algorithms and P2300 — https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2500r0.html

---

This cheat sheet provides a comprehensive, structured reference for C++ parallel algorithms, covering the facility, the four execution policies, Parallel STL backends, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.