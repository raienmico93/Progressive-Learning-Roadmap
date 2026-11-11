# C# Span and Memory — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`Span<T>` and `Memory<T>` are **types that represent contiguous regions of memory** in a type-safe, allocation-free way. They enable zero-copy slicing, parsing, and manipulation of arrays, strings, stack memory, and native buffers. `Span<T>` is a **`ref struct`** designed for synchronous, stack-only operations; `Memory<T>` is a regular struct designed for **storable, asynchronous** scenarios. Together they form the foundation of high-performance memory programming in .NET.

**Technical Definition**
`System.Span<T>` is a `ref struct` that provides a type-safe, memory-safe view over a contiguous region of arbitrary memory. It can wrap arrays, `stackalloc` buffers, strings (`ReadOnlySpan<char>`), or native memory. Because it is a `ref struct`, `Span<T>` is **stack-only**: it cannot be boxed, stored in fields of non-`ref struct` types, or used across `await` boundaries. `System.ReadOnlySpan<T>` is the read-only variant. `System.Memory<T>` and `System.ReadOnlyMemory<T>` are **regular structs** that can be stored in fields and used in `async` methods; `.Span` provides a synchronous `Span<T>` view. Since **C# 7.2**, `stackalloc` can produce a `Span<T>` in safe code. Since **C# 11**, `ref` fields in `ref struct` types and `scoped` modifiers refine lifetime rules.

**Beginner-Friendly Explanation**
`Span<T>` and `Memory<T>` are **windows into existing memory**. Instead of copying data (e.g., creating a new substring or array slice), you create a **view** that points to a portion of the original memory. You can read or write through the view, and you can slice it further without allocating. `Span<T>` is the **fast, stack-only** version—ideal for synchronous parsing, formatting, and buffer manipulation. `Memory<T>` is the **storable, async-friendly** version—ideal for I/O and pipelines where the view must survive across `await` or be held in a field. Both are type-safe and avoid the pitfalls of raw pointers.

---

### Key Characteristics

- **Zero-copy views** — `Span<T>` and `Memory<T>` reference existing memory without copying.
- **Type-safe** — Indexing and slicing are checked; no pointer arithmetic required.
- **`Span<T>` is a `ref struct`** — Stack-only; cannot be boxed, stored in fields, or used across `await`.
- **`Memory<T>` is a regular struct** — Can be stored in fields, captured by lambdas, and used across `await`.
- **`ReadOnlySpan<T>` / `ReadOnlyMemory<T>`** — Read-only variants that prevent mutation.
- **Unified abstraction** — Spans over arrays, strings, `stackalloc`, and native memory share the same API.
- **`stackalloc` returns `Span<T>`** — Since C# 7.2, safe stack allocation.
- **Lifetime rules** — Spans and memories must not outlive the underlying data.
- **Modern C# refinements** — `scoped` (C# 11), `ref` fields, and `ReadOnlySpan<char>` for string parsing.

---

### Prerequisites

- Basic understanding of **arrays**, **strings**, and **value types vs. reference types**.
- Familiarity with **`ref`/`out` parameters** and **`ref` returns/locals** (C# 7+).
- Awareness of **managed heap vs. stack** and garbage collection.
- Knowledge of `async`/`await` (for `Memory<T>` use cases).
- Basic understanding of **`stackalloc`** (helpful but not required).

---

### Related Programming Areas

- **String parsing** — `ReadOnlySpan<char>` avoids substring allocations.
- **Binary protocol parsing** — `Span<byte>` and `ReadOnlySpan<byte>` for zero-copy parsing.
- **High-performance I/O** — `System.IO.Pipelines` and `MemoryPool<T>` use `Memory<T>`.
- **Interop** — `Span<T>` over native memory; `stackalloc` for temporary buffers.
- **Memory optimization** — Allocation reduction, GC pressure reduction.

---

### Core Concepts / Features

1. `Span<T>`
2. `ReadOnlySpan<T>`
3. `Memory<T>`
4. `ReadOnlyMemory<T>`
5. `stackalloc`
6. Span Operations
7. Span Limitations
8. Span Best Practices

---

## 1. `Span<T>`

### Definitions

**Core Definition**
`Span<T>` is a **stack-only, type-safe view over a contiguous region of memory**, enabling allocation-free slicing and manipulation.

**Technical Definition**
`System.Span<T>` is a `ref struct` representing a contiguous region of memory. It can wrap:
- an array (`array.AsSpan()`),
- a `stackalloc` buffer,
- a string (`ReadOnlySpan<char>`),
- native memory (unsafe code).
It exposes a `Length` property, an indexer, and slicing methods (`Slice`, `Slice(int)`). It is **not** heap-allocatable: it cannot be boxed, stored in fields of classes, or used across `await`. It is the foundation for zero-copy parsing and buffer manipulation.

**Beginner-Friendly Explanation**
`Span<T>` is a **window into memory** you can slice and index without copying.

---

### Purposes

- To provide **zero-copy views** over memory.
- To slice arrays, strings, and stack buffers without allocating.
- To unify access to different memory sources (array, stack, native).
- To improve performance by eliminating copies and allocations.

---

### Syntax Rules and Structure

```csharp
Span<int> span = new[] { 1, 2, 3, 4, 5 };
Span<int> slice = span.Slice(1, 3);   // [2, 3, 4]
slice[0] = 20;                         // modifies original array
```

**Component breakdown**

- `Span<T>` — `ref struct` view.
- `.Slice(start, length)` — Sub-span.
- Indexer — Read/write access.
- `.Length` — Element count.

#### Syntax Rules

- `Span<T>` is a `ref struct` (stack-only).
- Use `AsSpan()` to get a span from an array or string.
- Slicing does not allocate; it returns a new view.

#### Constraints and Limitations

- Cannot be stored in a field of a class.
- Cannot be boxed or used across `await`.
- Cannot outlive the underlying memory.

---

### Annotated Code Examples and Expected Outputs

#### Example: Slicing Without Allocation

```csharp
using System;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };
        Span<int> slice = numbers.AsSpan(1, 3);   // [2, 3, 4]
        slice[0] = 20;
        Console.WriteLine(numbers[1]);   // 20 (modified)
    }
}
```

**Expected Output**
```
20
```

**Why this output occurs**
`AsSpan` returns a view over the array; modifying the span modifies the array.

---

### Real-World Cases

- **Parsing binary data** — `Span<byte>` over network buffers.
- **String manipulation** — `Span<char>` for formatting without allocation.
- **Image processing** — `Span<byte>` over pixel buffers.

---

### References

- Span<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.span-1
- Memory<T> and Span<T> usage guidelines — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines

---

## 2. `ReadOnlySpan<T>`

### Definitions

**Core Definition**
`ReadOnlySpan<T>` is a **stack-only, read-only view over contiguous memory**, ideal for parsing and inspection.

**Technical Definition**
`System.ReadOnlySpan<T>` is a `ref struct` that provides read-only access to a contiguous region of memory. It is returned by `string.AsSpan()` and is used for zero-copy string parsing. It supports `Length`, indexing (read-only), `Slice`, `IndexOf`, `StartsWith`, `EndsWith`, and more. It cannot modify the underlying memory.

**Beginner-Friendly Explanation**
`ReadOnlySpan<T>` is a **read-only window** into memory—perfect for parsing strings and buffers without modifying them.

---

### Purposes

- To provide **read-only views** without copying.
- To enable **zero-copy string parsing**.
- To avoid allocations in read-only operations.

---

### Syntax Rules and Structure

```csharp
ReadOnlySpan<char> span = "Hello, World".AsSpan();
ReadOnlySpan<char> hello = span.Slice(0, 5);
```

**Component breakdown**

- `ReadOnlySpan<T>` — Read-only `ref struct`.
- `.Slice(start, length)` — Sub-span.
- Indexer — Read-only access.

#### Syntax Rules

- `ReadOnlySpan<T>` is a `ref struct`.
- Use `AsSpan()` on strings to get `ReadOnlySpan<char>`.
- Slicing is allocation-free.

#### Constraints and Limitations

- Cannot be stored in fields (classes).
- Cannot be used across `await`.
- Cannot modify the underlying memory.

---

### Annotated Code Examples and Expected Outputs

#### Example: Parsing Without Allocation

```csharp
using System;

class Program
{
    static void Main()
    {
        ReadOnlySpan<char> csv = "Alice,30,Seattle".AsSpan();
        int comma = csv.IndexOf(',');
        ReadOnlySpan<char> name = csv.Slice(0, comma);
        Console.WriteLine(name.ToString());   // Alice
    }
}
```

**Expected Output**
```
Alice
```

**Why this output occurs**
Slicing the span does not allocate; `ToString()` is called only for output.

---

### Real-World Cases

- **CSV parsing** — Extracting fields without substrings.
- **URL parsing** — Extracting path segments.
- **Protocol parsing** — Reading headers from buffers.

---

### References

- ReadOnlySpan<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.readonlyspan-1

---

## 3. `Memory<T>`

### Definitions

**Core Definition**
`Memory<T>` is a **storable, heap-friendly view over contiguous memory**, usable in fields and across `await`.

**Technical Definition**
`System.Memory<T>` is a regular struct that represents a contiguous region of memory. Unlike `Span<T>`, `Memory<T>` is **not a `ref struct`**—it can be stored in fields, captured by lambdas, and used across `await`. It supports `Slice`, `Length`, and a `.Span` property for synchronous operations. It is commonly used with `ArrayPool<T>`, `MemoryPool<T>`, and `System.IO.Pipelines`.

**Beginner-Friendly Explanation**
`Memory<T>` is a **storable window** into memory—you can keep it in fields and use it across awaits.

---

### Purposes

- To provide a storable view over memory.
- To support **async operations** (unlike `Span<T>`).
- To integrate with pooled buffers.

---

### Syntax Rules and Structure

```csharp
Memory<byte> memory = new byte[100];
Memory<byte> slice = memory.Slice(10, 50);
Span<byte> span = memory.Span;   // synchronous view
```

**Component breakdown**

- `Memory<T>` — Storable view.
- `.Span` — Gets a `Span<T>` for synchronous work.
- `.Slice(start, length)` — Sub-view.

#### Syntax Rules

- Can be stored in fields and used in async code.
- `.Span` provides a `Span<T>` for synchronous operations.
- `Slice` returns a sub-view.

#### Constraints and Limitations

- Underlying memory must be kept alive while the `Memory<T>` is in use.
- Cannot be used directly for synchronous operations without `.Span`.

---

### Annotated Code Examples and Expected Outputs

#### Example: `Memory<T>` in Async Code

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Memory<byte> memory = new byte[100];
        await ProcessAsync(memory);
        Console.WriteLine(memory.Span[0]);   // 42
    }

    static async Task ProcessAsync(Memory<byte> memory)
    {
        await Task.Delay(100);
        memory.Span[0] = 42;   // safe: Memory<T> is not ref struct
    }
}
```

**Expected Output**
```
42
```

**Why this output occurs**
`Memory<T>` can be used across `await`; the span view is created after the await.

---

### Real-World Cases

- **Async I/O** — Buffered reads/writes.
- **Pipelines** — `System.IO.Pipelines` uses `ReadOnlyMemory<byte>`.
- **Pooled buffers** — `MemoryPool<T>` returns `Memory<T>`.

---

### References

- Memory<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.memory-1

---

## 4. `ReadOnlyMemory<T>`

### Definitions

**Core Definition**
`ReadOnlyMemory<T>` is a **storable, read-only view over contiguous memory**, usable in fields and across `await`.

**Technical Definition**
`System.ReadOnlyMemory<T>` is a regular struct that provides read-only access to a contiguous region of memory. It is the read-only counterpart of `Memory<T>`. It is used in `System.IO.Pipelines` (`ReadOnlySequence<T>` yields `ReadOnlyMemory<T>` slices) and in APIs that accept immutable buffers.

**Beginner-Friendly Explanation**
`ReadOnlyMemory<T>` is a **storable, read-only window** into memory.

---

### Purposes

- To provide **storable read-only views**.
- To support async/streaming APIs with immutable buffers.
- To avoid copying in read-only scenarios.

---

### Syntax Rules and Structure

```csharp
ReadOnlyMemory<byte> memory = new byte[100];
ReadOnlyMemory<byte> slice = memory.Slice(10, 50);
ReadOnlySpan<byte> span = memory.Span;   // synchronous view
```

**Component breakdown**

- `ReadOnlyMemory<T>` — Storable read-only view.
- `.Span` — Gets a `ReadOnlySpan<T>`.
- `.Slice(start, length)` — Sub-view.

#### Syntax Rules

- Can be stored in fields and used in async code.
- `.Span` provides a `ReadOnlySpan<T>`.

#### Constraints and Limitations

- Cannot modify the underlying memory.
- Underlying memory must be kept alive.

---

### Annotated Code Examples and Expected Outputs

#### Example: Read-Only Async Processing

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        ReadOnlyMemory<byte> memory = new byte[] { 1, 2, 3 };
        await ProcessAsync(memory);
    }

    static async Task ProcessAsync(ReadOnlyMemory<byte> memory)
    {
        await Task.Delay(100);
        ReadOnlySpan<byte> span = memory.Span;
        Console.WriteLine(span[0]);   // 1
    }
}
```

**Expected Output**
```
1
```

**Why this output occurs**
`ReadOnlyMemory<T>` is used across `await`; its `.Span` is read-only.

---

### Real-World Cases

- **Pipelines** — `ReadOnlySequence<byte>` yields `ReadOnlyMemory<byte>`.
- **Middleware** — Passing immutable buffers through async layers.

---

### References

- ReadOnlyMemory<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.readonlymemory-1

---

## 5. `stackalloc`

### Definitions

**Core Definition**
`stackalloc` allocates a block of memory on the **stack**, avoiding heap allocation.

**Technical Definition**
`stackalloc T[n]` allocates `n` elements of type `T` on the stack. In safe code (C# 7.2+), `stackalloc` can be assigned to `Span<T>` or `ReadOnlySpan<T>`. In unsafe code, it returns a pointer. Stack memory is automatically reclaimed when the method returns. Because the stack is limited (~1 MB per thread), `stackalloc` should only be used for small, short-lived buffers.

**Beginner-Friendly Explanation**
`stackalloc` is **"allocate on the stack"**—fast, automatic, no GC.

---

### Purposes

- To avoid heap allocation for small buffers.
- To improve performance in hot paths.
- To reduce GC pressure.

---

### Syntax Rules and Structure

```csharp
Span<byte> buffer = stackalloc byte[256];
```

**Component breakdown**

- `stackalloc T[n]` — Allocates `n` elements on the stack.
- Assigned to `Span<T>` (safe code).

#### Syntax Rules

- Use only for small buffers (typically ≤ a few KB).
- Do not use inside loops (stack growth).
- The memory is freed when the method returns.
- Since C# 7.2, `stackalloc` is safe when assigned to `Span<T>`.

#### Constraints and Limitations

- Stack size is limited (~1 MB per thread).
- Stack overflow if too much is allocated.
- Do not return the span from the method.

---

### Annotated Code Examples and Expected Outputs

#### Example: Stack Allocation

```csharp
using System;

class Program
{
    static void Main()
    {
        Span<int> numbers = stackalloc int[5];
        for (int i = 0; i < numbers.Length; i++) numbers[i] = i * i;
        foreach (var n in numbers) Console.Write(n + " ");
        // 0 1 4 9 16
    }
}
```

**Expected Output**
```
0 1 4 9 16
```

**Why this output occurs**
`stackalloc` allocates the buffer on the stack; it is automatically freed when `Main` returns.

---

### Real-World Cases

- **Small temporary buffers** — Formatting, hashing, parsing.
- **Cryptography** — Temporary key material (clear after use).

---

### References

- stackalloc expression — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/stackalloc

---

## 6. Span Operations

### Definitions

**Core Definition**
Span operations are the methods and indexers available on `Span<T>` and `ReadOnlySpan<T>` for slicing, searching, and manipulating memory.

**Technical Definition**
Spans expose a rich API: `Length`, indexer, `Slice(start, length)`, `Slice(start)`, `ToArray()`, `CopyTo()`, `Fill()`, `Clear()`, `IndexOf()`, `LastIndexOf()`, `StartsWith()`, `EndsWith()`, `Contains()`, `BinarySearch()`, `SequenceEqual()`, `Reverse()`, `Sort()`, and more. Many operations are intrinsic and JIT-optimized (e.g., `IndexOf` uses vectorized search). The API is designed to be allocation-free and efficient.

**Beginner-Friendly Explanation**
Spans have **slicing, searching, copying, and filling operations**—all allocation-free.

---

### Purposes

- To slice and manipulate memory without copying.
- To search for values efficiently (vectorized).
- To copy and fill buffers.
- To parse and transform data in place.

---

### Syntax Rules and Structure

```csharp
Span<int> span = new[] { 1, 2, 3, 4, 5 };

// Slicing
Span<int> slice = span.Slice(1, 3);

// Searching
int index = span.IndexOf(3);   // 2

// Copying
Span<int> dest = new int[5];
span.CopyTo(dest);

// Filling
span.Fill(0);

// Sorting
span.Sort();
```

**Component breakdown**

- `.Slice(start, length)` — Sub-span.
- `.IndexOf(value)` — Search.
- `.CopyTo(destination)` — Copy.
- `.Fill(value)` — Fill with a value.
- `.Sort()` — Sort in place.

#### Syntax Rules

- Slicing returns a new view (no allocation).
- `IndexOf` and similar operations are vectorized.
- `CopyTo` requires the destination to be large enough.

#### Constraints and Limitations

- Some operations (e.g., `Sort`) mutate the underlying memory.
- `ToArray()` allocates a new array.

---

### Annotated Code Examples and Expected Outputs

#### Example: Slicing, Searching, and Copying

```csharp
using System;

class Program
{
    static void Main()
    {
        Span<int> span = new[] { 1, 2, 3, 4, 5 };
        Span<int> slice = span.Slice(1, 3);   // [2, 3, 4]
        Console.WriteLine(slice.IndexOf(3));   // 1 (within slice)

        Span<int> dest = new int[3];
        slice.CopyTo(dest);
        Console.WriteLine(string.Join(", ", dest.ToArray()));   // 2, 3, 4
    }
}
```

**Expected Output**
```
1
2, 3, 4
```

**Why this output occurs**
`IndexOf` searches within the slice; `CopyTo` copies the slice to a new span; `ToArray` materializes for output.

---

### Real-World Cases

- **Parsing** — `IndexOf` and `Slice` for tokenization.
- **Binary protocols** — `SequenceEqual` for magic numbers.
- **Buffers** — `Fill` and `Clear` for reuse.

---

### References

- Span<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.span-1

---

## 7. Span Limitations

### Definitions

**Core Definition**
Span limitations are the restrictions imposed by `Span<T>` being a `ref struct` and a view over memory.

**Technical Definition**
`Span<T>` is a `ref struct`, which imposes strict lifetime rules: it can only be allocated on the stack, cannot be boxed, cannot be stored in fields of classes, cannot be captured by lambdas that escape, and cannot be used across `await`. Additionally, a span must not outlive the memory it references—returning a span over a local array or `stackalloc` buffer is invalid. Since C# 11, `scoped` and `ref` fields refine these rules, but the fundamental constraint remains: spans are **stack-only, non-escaping views**.

**Beginner-Friendly Explanation**
Spans are **stack-only and cannot escape**—no fields, no async, no boxing.

---

### Purposes

- To understand why `Span<T>` cannot be used in certain contexts.
- To choose `Memory<T>` when spans are unsuitable.
- To avoid lifetime bugs.

---

### Syntax Rules and Structure (Limitation Matrix)

| Context | `Span<T>` allowed? |
|---|---|
| Local variable | ✔️ |
| Method parameter | ✔️ |
| Return type | ✔️ (with lifetime constraints) |
| Field in `ref struct` | ✔️ (C# 11+ with `ref` fields) |
| Field in `class`/`struct` | ❌ |
| `async` method across `await` | ❌ |
| Lambda that escapes | ❌ |
| Boxing | ❌ |
| `Task.Run` closure | ❌ |

**Component breakdown**

- **Stack-only** — Cannot escape to the heap.
- **No async** — Cannot cross `await`.
- **No fields in classes** — Cannot be stored.
- **No boxing** — Cannot be boxed.

#### Syntax Rules

- Spans cannot be used across `await`; use `Memory<T>`.
- Spans cannot be stored in class fields; use `Memory<T>` or `ReadOnlyMemory<T>`.
- Spans cannot be captured by escaping lambdas.

#### Constraints and Limitations

- Some APIs require `Memory<T>` for async support.
- Spans cannot be used with `LINQ` directly (no `IEnumerable<T>`).

---

### Annotated Code Examples and Expected Outputs

#### Example: `Span<T>` Across `await` (Compile Error)

```csharp
// COMPILE ERROR: Cannot use Span<T> across await
async Task BadAsync()
{
    Span<int> span = stackalloc int[10];
    // await Task.Delay(100);   // CS4007: cannot use span across await
}
```

**Expected Output (conceptual)**
```
Compile-time error CS4007
```

**Why this error occurs**
`Span<T>` is a `ref struct` and cannot be used across `await`.

---

### Real-World Cases

- **Async I/O** — Must use `Memory<T>` instead of `Span<T>`.
- **Class fields** — Must use `Memory<T>` or arrays.
- **LINQ** — Must use arrays or collections, not spans.

---

### References

- Memory<T> and Span<T> usage guidelines — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines

---

## 8. Span Best Practices

### Definitions

**Core Definition**
Span best practices are the guidelines for using `Span<T>`, `ReadOnlySpan<T>`, `Memory<T>`, and `ReadOnlyMemory<T>` correctly, safely, and efficiently.

**Technical Definition**
Best practices include: **prefer `ReadOnlySpan<T>` for read-only APIs**; **use `Span<T>` for synchronous, stack-only operations**; **use `Memory<T>` for async or storable scenarios**; **do not return spans over local or `stackalloc` memory**; **use `stackalloc` only for small buffers**; **use `AsSpan()` and `Slice()` for allocation-free slicing**; **use `ArrayPool<T>` for large or repeated buffers**; **avoid copying when a span suffices**; and **be mindful of lifetime rules**.

**Beginner-Friendly Explanation**
The best practices are: **use spans for sync, memory for async**, **don't return spans of locals**, **don't overuse `stackalloc`**, and **prefer `ReadOnlySpan` for read-only APIs**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Use ReadOnlySpan<T> for read-only parameters
public int CountWords(ReadOnlySpan<char> text) { ... }

// 2. Use Span<T> for synchronous manipulation
Span<byte> buffer = stackalloc byte[256];
buffer.Fill(0);

// 3. Use Memory<T> for async/storable scenarios
public async Task ProcessAsync(Memory<byte> memory) { ... }

// 4. Do not return spans over local or stackalloc memory
// BAD: Span<int> GetSpan() { Span<int> s = stackalloc int[10]; return s; }

// 5. Use AsSpan() and Slice() for allocation-free slicing
ReadOnlySpan<char> slice = text.AsSpan(start, length);

// 6. Use ArrayPool<T> for large or repeated buffers
var buffer = ArrayPool<byte>.Shared.Rent(1024);
try { ... }
finally { ArrayPool<byte>.Shared.Return(buffer); }

// 7. Use stackalloc only for small buffers
Span<byte> temp = stackalloc byte[64];   // ok
Span<byte> huge = stackalloc byte[1_000_000];   // stack overflow risk
```

**Component breakdown**

- **`ReadOnlySpan<T>` for read-only** — Prevents mutation; clearer API.
- **`Span<T>` for synchronous** — Stack-only; fastest.
- **`Memory<T>` for async** — Storable; async-friendly.
- **No escaping spans** — Lifetime rules.
- **`AsSpan`/`Slice`** — Allocation-free.
- **Pooling** — `ArrayPool<T>` for large buffers.
- **Small `stackalloc`** — Avoid stack overflow.

#### Rules and Constraints

- **Do** use `ReadOnlySpan<T>` for read-only APIs.
- **Do** use `Memory<T>` for async/storable scenarios.
- **Do** use `AsSpan()` and `Slice()` for allocation-free slicing.
- **Do** use `ArrayPool<T>` for large or repeated buffers.
- **Do not** return spans over local or `stackalloc` memory.
- **Do not** use `stackalloc` for large buffers.
- **Do not** use spans across `await`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Span API

```csharp
using System;
using System.Buffers;

public static class Parser
{
    public static int CountDigits(ReadOnlySpan<char> text)
    {
        int count = 0;
        foreach (char c in text)
            if (char.IsDigit(c)) count++;
        return count;
    }
}

class Program
{
    static void Main()
    {
        ReadOnlySpan<char> text = "Order 12345".AsSpan();
        Console.WriteLine(Parser.CountDigits(text));   // 5
    }
}
```

**Expected Output**
```
5
```

**Why this output occurs**
`ReadOnlySpan<char>` provides a zero-copy view of the string; the method counts digits without allocating.

---

### Real-World Cases

- **Parsers** — `ReadOnlySpan<char>` for tokenizing.
- **Networking** — `Span<byte>` / `Memory<byte>` for buffers.
- **Serialization** — `Span<T>` for zero-copy encoding.

---

### References

- Memory<T> and Span<T> usage guidelines — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines
- Span<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.span-1
- ReadOnlySpan<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.readonlyspan-1
- Memory<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.memory-1
- ReadOnlyMemory<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.readonlymemory-1
- stackalloc expression — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/stackalloc

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `Span<T>` / `ReadOnlySpan<T>` | Introduced in **.NET Core 2.1 / C# 7.2**; stable. |
| `Memory<T>` / `ReadOnlyMemory<T>` | Introduced in **.NET Core 2.1**; stable. |
| `stackalloc` in safe code | Introduced in **C# 7.2**; requires `Span<T>`. |
| `scoped` modifier | Introduced in **C# 11**; refines lifetime rules. |
| `ref` fields in `ref struct` | Introduced in **C# 11**; allows spans in `ref struct` fields. |
| `Span<T>` across `await` | **Compile-time error** (CS4007); use `Memory<T>`. |
| `Span<T>` in class fields | **Not allowed**; use `Memory<T>`. |
| `Span<T>` boxing | **Not allowed**. |

---

## References (Consolidated)

- Memory<T> and Span<T> usage guidelines — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines
- Span<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.span-1
- ReadOnlySpan<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.readonlyspan-1
- Memory<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.memory-1
- ReadOnlyMemory<T> Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.readonlymemory-1
- stackalloc expression — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/stackalloc

---