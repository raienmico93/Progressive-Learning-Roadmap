# C# Unsafe Code — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**Unsafe code** in C# is code that is **not verifiably safe** by the .NET runtime. It enables direct memory access through **pointers**, **stack allocation**, **function pointers**, and **unchecked bounds operations**, bypassing the safety guarantees of managed code. Unsafe code is written inside an `unsafe` context and compiled only when the `AllowUnsafeBlocks` compiler option is enabled .

**Technical Definition**
C# defines two models for unsafe code. In the **original model** (C# 1.0, still the default), an `unsafe` context is required to declare a pointer type, take an address with `&`, dereference a pointer, convert a `stackalloc` expression to a pointer, or apply `sizeof` to an arbitrary type . In the **updated memory safety model** (preview in C# 15 / .NET 11), the existence of a pointer is allowed in safe code; only operations that access unmanaged memory (dereference, member access, element access) require an `unsafe` context, and the `unsafe` modifier becomes a contract that propagates the obligation to audit safety to callers . Unsafe code introduces **security and stability risks** such as buffer overruns, access violations, and memory corruption . `stackalloc` assigned to `Span<T>` or `ReadOnlySpan<T>` is allowed in safe code; assigning it to a pointer requires `unsafe` .

**Beginner-Friendly Explanation**
Unsafe code is **“I know what I’m doing with memory.”** Normally, C# protects you: arrays are bounds-checked, references are tracked by the GC, and pointers do not exist. Unsafe code lets you use **raw pointers** (like C/C++), **pin** managed objects so the GC will not move them, and **allocate memory on the stack**. This is necessary for **calling native libraries that require pointers**, and sometimes for **squeezing out performance** by avoiding bounds checks. But it is called “unsafe” for a reason: a mistake can crash the process or corrupt data, and the runtime will not catch it for you.

---

### Key Characteristics

- **Requires `AllowUnsafeBlocks`** — Unsafe code does not compile without the `/unsafe` compiler option or `<AllowUnsafeBlocks>true</AllowUnsafeBlocks>` in the project file .
- **Pointer types** — `T*` for unmanaged types (`int*`, `byte*`, `void*`).
- **`fixed` statement** — Pins a movable managed variable (array, string, object field) so its address remains stable during pointer operations .
- **`stackalloc`** — Stack allocation; safe when assigned to `Span<T>`, unsafe when assigned to a pointer .
- **`sizeof`** — Safe for built-in unmanaged types; unsafe for arbitrary types and managed types .
- **Function pointers** — `delegate*<T, TResult>` for calling native or unmanaged function pointers.
- **Not verifiable** — The JIT cannot prove memory safety; errors are not caught at compile time or by the runtime.

---

### Prerequisites

- Solid understanding of **value types vs. reference types** and the **managed heap**.
- Familiarity with **garbage collection** and **object movement** (why pinning is needed).
- Knowledge of **unmanaged types** (types that contain no references).
- Awareness of **native interop** (P/Invoke, `DllImport`, `LibraryImport`).
- Basic understanding of **memory layout** and **stack vs. heap**.

---

### Related Programming Areas

- **Interoperability** — Calling native functions that require pointers.
- **Performance engineering** — Avoiding bounds checks in hot paths (with caution).
- **Span and Memory** — `Span<T>` provides safe, allocation-free memory access; unsafe is rarely needed for spans.
- **Function pointers** — Calling unmanaged callbacks.
- **Memory management** — `stackalloc`, `NativeMemory.Alloc`, `fixed`.

---

### Core Concepts / Features

1. Unsafe Code (Overarching Concept)
2. `unsafe` Keyword
3. Pointer Types
4. `fixed` Statement
5. `stackalloc`
6. `sizeof` Operator
7. Function Pointers
8. Unsafe Code Security
9. Unsafe Code Best Practices

---

## 1. Unsafe Code (Overarching Concept)

### Definitions

**Core Definition**
Unsafe code is code whose memory safety cannot be verified by .NET tools, typically because it uses pointers or raw memory.

**Technical Definition**
The C# language supports an `unsafe` context in which you can write unverifiable code: code can use pointers, allocate and free blocks of memory, and call methods by using function pointers . Unsafe code is not necessarily dangerous, but its safety cannot be verified by the runtime; it introduces security and stability risks .

**Beginner-Friendly Explanation**
Unsafe code is **code that steps outside the safety net**. It can be useful, but you are responsible for correctness.

---

### Purposes

- To **call native functions** that require pointers.
- To **improve performance** by avoiding array-bounds checks in proven hot paths .
- To **interact with unmanaged memory** and hardware.
- To **implement low-level algorithms** (parsing, compression, cryptography).

---

### Syntax Rules and Structure

```
unsafe
{
    // pointer operations
}
```

**Component breakdown**

- `unsafe` — Establishes the unsafe context.
- `AllowUnsafeBlocks` — Required to compile.

#### Syntax Rules

- The `unsafe` modifier can be applied to types, members, blocks, and statements.
- Pointer types, address-of, dereference, and pointer `stackalloc` require `unsafe` in the original model.
- `stackalloc` assigned to `Span<T>` is safe.

#### Constraints and Limitations

- Unverifiable code can cause access violations and memory corruption.
- Requires explicit compiler opt-in.

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic Unsafe Block

```csharp
using System;

class Program
{
    static unsafe void Main()
    {
        int value = 42;
        int* p = &value;
        Console.WriteLine(*p);   // 42
    }
}
```

**Expected Output**
```
42
```

**Why this output occurs**
`&value` takes the address of the local; `*p` dereferences it to read `42`.

---

### Real-World Cases

- **P/Invoke** — Passing pointers to native functions.
- **Hardware access** — Memory-mapped I/O.
- **High-performance buffers** — Avoiding bounds checks.

---

### References

- Unsafe code, pointers to data, and function pointers — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code
- AllowUnsafeBlocks — C# compiler options — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/language#allowunsafeblocks

---

## 2. `unsafe` Keyword

### Definitions

**Core Definition**
The `unsafe` keyword establishes an **unsafe context** in which pointer operations and unverifiable constructs are permitted.

**Technical Definition**
The `unsafe` modifier can be applied to a class, struct, method, property, constructor, or block. It establishes the context for pointer declaration, address-of, dereference, pointer `stackalloc`, and `sizeof` on arbitrary types. In the original model, the modifier places no obligation on callers; in the updated model (preview), it propagates the safety obligation to callers .

**Beginner-Friendly Explanation**
`unsafe` is the **“enter raw memory mode”** marker.

---

### Purposes

- To **enable pointer syntax**.
- To **scope unsafe operations** to a minimal region.
- To **document** that the code requires manual memory safety audit.

---

### Syntax Rules and Structure

```csharp
unsafe
{
    // pointer operations
}

// Or as a modifier:
unsafe void Method() { ... }
unsafe class C { ... }
```

**Component breakdown**

- `unsafe` — Modifier or block keyword.
- Block scope — Limits the unsafe region.

#### Syntax Rules

- Applies to types, members, and blocks.
- Required for pointer declaration, address-of, dereference, and pointer `stackalloc` in the original model.
- In the updated model, `unsafe` on a member propagates the audit obligation to callers.

#### Constraints and Limitations

- Requires `AllowUnsafeBlocks`.
- The updated model is preview-only (C# 15 / .NET 11).

---

### Annotated Code Examples and Expected Outputs

#### Example: Unsafe Method Modifier

```csharp
using System;

class Program
{
    static unsafe int Read(int* p) => *p;

    static unsafe void Main()
    {
        int x = 10;
        Console.WriteLine(Read(&x));   // 10
    }
}
```

**Expected Output**
```
10
```

**Why this output occurs**
The `unsafe` modifier on `Read` and `Main` allows pointer parameters and address-of.

---

### Real-World Cases

- **Interop methods** — Unsafe methods with pointer parameters.
- **Hot paths** — Minimally scoped unsafe blocks.

---

### References

- unsafe keyword — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/unsafe

---

## 3. Pointer Types

### Definitions

**Core Definition**
A pointer type (`T*`) holds the **memory address** of a variable of an unmanaged type `T`.

**Technical Definition**
Pointers can point to unmanaged types (numeric types, `bool`, `char`, enums, and structs composed entirely of unmanaged types). They support:
- **Address-of** (`&`),
- **Dereference** (`*`),
- **Member access** (`->`),
- **Element access** (`p[i]`),
- **Pointer arithmetic** (`p + n`, `p - n`, `p1 - p2`),
- **Comparison** (`==`, `!=`, `<`, `>`, etc.).

Pointers cannot point to managed types (classes, strings, arrays of references).

**Beginner-Friendly Explanation**
A pointer is a **raw memory address**. `int* p` says “p holds the address of an int.”

---

### Purposes

- To **access unmanaged memory** directly.
- To **call native functions** that require pointers.
- To **implement low-level data structures** (linked lists, buffers).
- To **manipulate memory** without bounds checks.

---

### Syntax Rules and Structure

```csharp
int* p;            // pointer to int
byte* b;           // pointer to byte
void* v;           // pointer to unknown type
int** pp;          // pointer to pointer
```

**Component breakdown**

- `T*` — Pointer type.
- `&x` — Address-of.
- `*p` — Dereference.
- `p->Member` — Member access.
- `p[i]` — Element access.
- `p + n` — Pointer arithmetic.

#### Syntax Rules

- Only unmanaged types can be pointed to.
- Pointer arithmetic operates in units of the pointed-to type.
- `void*` supports address-of and comparison but not dereference.

#### Constraints and Limitations

- Cannot point to managed types (classes, strings, arrays of references).
- Cannot use `is` or `as` with pointers.
- Cannot create pointer instances with `new`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Pointer Arithmetic

```csharp
using System;

class Program
{
    static unsafe void Main()
    {
        int[] numbers = { 10, 20, 30, 40 };
        fixed (int* p = numbers)
        {
            Console.WriteLine(*(p + 1));   // 20
            Console.WriteLine(p[3]);       // 40
        }
    }
}
```

**Expected Output**
```
20
40
```

**Why this output occurs**
`p + 1` advances by one `int`; `p[3]` accesses the fourth element.

---

### Real-World Cases

- **Native buffers** — `byte*` for raw data.
- **Interop** — Passing pointers to native code.
- **Low-level algorithms** — Pointer-based data structures.

---

### References

- Pointer types — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code#pointer-types
- Pointer related operators — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/pointer-related-operators

---

## 4. `fixed` Statement

### Definitions

**Core Definition**
The `fixed` statement **pins a movable managed variable** (array element, string, or object field) so the garbage collector does not relocate it, and declares a pointer to it for the duration of the statement.

**Technical Definition**
The `fixed` statement prevents the GC from relocating a movable variable and declares a pointer to that variable. The address of a fixed variable does not change during the execution of the statement, and the declared pointer is readonly and can be used only inside the corresponding `fixed` statement . You can initialize the pointer from an array (address of first element), the address of a variable (`&numbers[0]`), or an instance implementing `GetPinnableReference` (e.g., `Span<T>`) .

**Beginner-Friendly Explanation**
`fixed` is **“freeze this object in place so I can safely use its address.”**

---

### Purposes

- To **pin managed memory** for pointer operations.
- To **prevent GC movement** during native calls.
- To **safely obtain pointers to array elements and fields**.

---

### Syntax Rules and Structure

```csharp
fixed (int* p = array)
{
    // p is valid here
}
fixed (int* p = &array[0])
{
    // p points to first element
}
fixed (int* p = &obj.Field)
{
    // p points to field
}
```

**Component breakdown**

- `fixed (T* p = expr)` — Pins `expr` and declares pointer `p`.
- Scope — The pointer is valid only inside the block.

#### Syntax Rules

- The pointer is readonly and cannot be modified.
- The pinned object cannot be moved by the GC during the block.
- Can pin arrays, strings, object fields, and `GetPinnableReference` types.
- In the updated memory safety model (preview), `fixed` is allowed outside `unsafe`, but pointer indirection still requires `unsafe` .

#### Constraints and Limitations

- Pinning can cause heap fragmentation if held too long.
- Cannot pin a local variable directly (use `&` on a local in unsafe code).

---

### Annotated Code Examples and Expected Outputs

#### Example: Pinning an Array

```csharp
using System;

class Program
{
    static unsafe void Main()
    {
        byte[] bytes = { 1, 2, 3 };
        fixed (byte* p = bytes)
        {
            Console.WriteLine($"First: {*p}");   // First: 1
        }
    }
}
```

**Expected Output**
```
First: 1
```

**Why this output occurs**
`fixed` pins the array and gives a pointer to its first element; `*p` dereferences it.

---

### Real-World Cases

- **Native calls** — Passing array pointers to P/Invoke.
- **String manipulation** — Pinning strings for native encoding.
- **Span interoperability** — Pinning for pointer-based APIs.

---

### References

- fixed statement — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/fixed

---

## 5. `stackalloc` (Unsafe)

### Definitions

**Core Definition**
`stackalloc` allocates memory on the **stack**. In an unsafe context, it can produce a **pointer** (`T*`); in safe code, it can produce a `Span<T>` or `ReadOnlySpan<T>` .

**Technical Definition**
A `stackalloc` expression allocates `n` elements of an unmanaged type on the stack. When the result is assigned directly to a pointer-typed local variable (or `var` with pointer inference), the result is a `T*` and the expression must appear in unsafe code . Stack-allocated memory is automatically discarded when the function returns; there is no way to explicitly free it . Initializers are not permitted in `catch` or `finally` blocks .

**Beginner-Friendly Explanation**
`stackalloc` is **“allocate on the stack”**—fast, automatic, and no GC. In unsafe code it gives you a pointer; in safe code it gives you a `Span<T>`.

---

### Purposes

- To **avoid heap allocation** for small buffers.
- To **reduce GC pressure** in hot paths.
- To **obtain a pointer** for native APIs.

---

### Syntax Rules and Structure

```csharp
unsafe
{
    int* p = stackalloc int[10];   // pointer (unsafe)
}

Span<int> span = stackalloc int[10];   // safe (Span)
```

**Component breakdown**

- `stackalloc T[n]` — Allocates `n` elements on the stack.
- Pointer form — Requires `unsafe`.
- `Span<T>` form — Allowed in safe code.

#### Syntax Rules

- Pointer form requires `unsafe`.
- `Span<T>` form is safe.
- No explicit free; memory is freed when the method returns.
- Initializers allowed except in `catch`/`finally`.

#### Constraints and Limitations

- Stack space is limited (~1 MB per thread).
- Large allocations cause stack overflow.
- Uninitialized memory may contain stale data.

---

### Annotated Code Examples and Expected Outputs

#### Example: Unsafe `stackalloc` Pointer

```csharp
using System;

class Program
{
    static unsafe void Main()
    {
        int* values = stackalloc int[3];
        values[0] = 10;
        values[1] = 20;
        values[2] = 30;

        Console.WriteLine(values[1]);   // 20
    }
}
```

**Expected Output**
```
20
```

**Why this output occurs**
`stackalloc` allocates three ints on the stack; the pointer provides access.

---

### Real-World Cases

- **Temporary buffers** — Parsing, hashing, formatting.
- **Cryptography** — Temporary key material (clear after use).
- **Native interop** — Stack buffers for P/Invoke.

---

### References

- stackalloc expression — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/stackalloc
- stackalloc — C# language specification — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions

---

## 6. `sizeof` Operator

### Definitions

**Core Definition**
`sizeof(T)` returns the **number of bytes** occupied by a variable of type `T`.

**Technical Definition**
In **safe code**, the argument to `sizeof` must be the name of a built-in unmanaged type whose size is not platform-dependent, or an enumeration type . In **unsafe code**, `sizeof` can be applied to **any non-void type**, including pointer types and managed types; for managed types, it returns the size of a **reference**, not the object size . In the updated memory safety preview (C# 15), `sizeof` on unmanaged types is allowed outside `unsafe` .

**Beginner-Friendly Explanation**
`sizeof(int)` tells you **how many bytes an int takes**. In safe code it works for built-in types; in unsafe code it works for any type.

---

### Purposes

- To **determine memory requirements**.
- To **size buffers** for native interop.
- To **calculate offsets** in memory layouts.

---

### Syntax Rules and Structure

```csharp
int size = sizeof(int);          // safe
unsafe { int s = sizeof(MyStruct); }   // unsafe (arbitrary type)
```

**Component breakdown**

- `sizeof(T)` — Returns `int` byte count.
- Safe — Built-in unmanaged types.
- Unsafe — Any non-void type.

#### Syntax Rules

- Safe: built-in unmanaged types and enums only.
- Unsafe: any non-void type.
- Returns reference size for managed types.

#### Constraints and Limitations

- `sizeof` on managed types returns reference size, not object size.
- `sizeof` on pointers returns pointer size (4 or 8 bytes).

---

### Annotated Code Examples and Expected Outputs

#### Example: `sizeof` in Safe and Unsafe Code

```csharp
using System;

struct Point { public int X, Y; }

class Program
{
    static unsafe void Main()
    {
        Console.WriteLine(sizeof(int));     // 4
        Console.WriteLine(sizeof(Point));    // 8
    }
}
```

**Expected Output**
```
4
8
```

**Why this output occurs**
`int` is 4 bytes; `Point` contains two ints (8 bytes).

---

### Real-World Cases

- **Interop** — Sizing native structs.
- **Serialization** — Computing buffer sizes.
- **Memory layout** — Offset calculations.

---

### References

- sizeof operator — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/sizeof

---

## 7. Function Pointers

### Definitions

**Core Definition**
A function pointer (`delegate*<T, TResult>`) holds the **address of a function** and can be invoked directly, without a delegate object.

**Technical Definition**
Function pointers are declared with `delegate*` syntax and can point to static methods, `[UnmanagedCallersOnly]` methods, or native functions. They avoid delegate allocation and are useful for high-performance callbacks and native interop. They require an unsafe context.

**Beginner-Friendly Explanation**
A function pointer is a **raw pointer to a function**—no delegate wrapper, no allocation.

---

### Purposes

- To **call native callbacks** efficiently.
- To **avoid delegate allocations** in hot paths.
- To **implement low-level dispatch tables**.

---

### Syntax Rules and Structure

```csharp
unsafe
{
    delegate*<int, int> f = &Square;
    int result = f(5);
}
```

**Component breakdown**

- `delegate*<T, TResult>` — Function pointer type.
- `&Method` — Address of a method.

#### Syntax Rules

- Requires `unsafe`.
- Can point to static methods and `[UnmanagedCallersOnly]` methods.
- Invocation uses `f(args)`.

#### Constraints and Limitations

- Cannot point to instance methods (unless using `delegate* unmanaged` with explicit `this`).
- Native function pointers require `[UnmanagedCallersOnly]` or `DllImport`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Function Pointer to Static Method

```csharp
using System;

class Program
{
    static int Square(int x) => x * x;

    static unsafe void Main()
    {
        delegate*<int, int> f = &Square;
        Console.WriteLine(f(5));   // 25
    }
}
```

**Expected Output**
```
25
```

**Why this output occurs**
The function pointer references `Square` and invokes it directly.

---

### Real-World Cases

- **Native callbacks** — Passing function pointers to C libraries.
- **Virtual dispatch tables** — High-performance dispatch.
- **Interop** — Avoiding delegate marshaling.

---

### References

- Function pointers — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code#function-pointers
- UnmanagedCallersOnlyAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.unmanagedcallersonlyattribute

---

## 8. Unsafe Code Security

### Definitions

**Core Definition**
Unsafe code security concerns the **risks and mitigations** associated with bypassing the runtime’s memory safety guarantees.

**Technical Definition**
Unsafe code can cause **buffer overruns**, **access violations**, **memory corruption**, **data leaks**, and **undefined behaviour** when managed references are involved. The .NET documentation explicitly warns that unsafe code introduces security and stability risks . Best practices include **bounds validation**, **pinning discipline**, **avoiding untracked managed pointers**, and **not invoking non-public APIs via unsafe code** .

**Beginner-Friendly Explanation**
Unsafe code is **“you can shoot yourself in the foot.”** The runtime will not catch out-of-bounds access or use-after-free.

---

### Purposes

- To **understand the risks** of unsafe code.
- To **mitigate vulnerabilities** (bounds checks, pinning).
- To **audit unsafe code** for security.

---

### Security Risks

| Risk | Cause | Mitigation |
|---|---|---|
| Buffer overrun | Unchecked pointer arithmetic | Manual bounds checks |
| Access violation | Dereferencing invalid pointer | Validate pointers |
| Memory corruption | Writing outside buffer | Size tracking |
| Data leak | Uninitialized memory exposure | Clear buffers |
| GC movement | Pointer to unpinned object | Use `fixed` |

**Component breakdown**

- **Buffer overrun** — `p[i]` without bounds check.
- **Access violation** — Invalid pointer dereference.
- **GC movement** — Pointer to movable object.

#### Syntax Rules

- Always validate indices and sizes.
- Use `fixed` for managed memory.
- Clear sensitive buffers with `Span.Clear` or `memset`.

#### Constraints and Limitations

- The runtime cannot verify unsafe code.
- Security audits should focus on unsafe blocks.

---

### Annotated Code Examples and Expected Outputs

#### Example: Bounds Check Before Pointer Access

```csharp
using System;

class Program
{
    static unsafe int SafeRead(int* p, int length, int index)
    {
        if (index < 0 || index >= length)
            throw new ArgumentOutOfRangeException(nameof(index));
        return p[index];
    }

    static unsafe void Main()
    {
        int[] data = { 10, 20, 30 };
        fixed (int* p = data)
            Console.WriteLine(SafeRead(p, data.Length, 1));   // 20
    }
}
```

**Expected Output**
```
20
```

**Why this output occurs**
The bounds check prevents out-of-range access, mitigating the primary risk of unsafe pointer arithmetic.

---

### Real-World Cases

- **Native libraries** — Auditing pointer usage.
- **Security-sensitive code** — Validating all unsafe operations.

---

### References

- Unsafe code best practices — .NET — https://learn.microsoft.com/en-us/dotnet/standard/unsafe-code/best-practices
- Unsafe code, pointers to data, and function pointers — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code

---

## 9. Unsafe Code Best Practices

### General Guidelines

```
// 1. Minimize unsafe scope
unsafe { /* only the necessary operations */ }

// 2. Validate bounds manually
if (index < 0 || index >= length) throw new ArgumentOutOfRangeException();

// 3. Use fixed for managed memory
fixed (byte* p = array) { ... }

// 4. Prefer Span<T> over pointers for safe memory access
Span<byte> buffer = stackalloc byte[256];

// 5. Never expose pointers outside fixed scope
// 6. Do not access locals across threads (CS9123)
// 7. Do not invoke non-public APIs via unsafe code
// 8. Use Debug.Assert to document invariants
// 9. Measure before removing bounds checks (JIT may already optimize)
```

**Component breakdown**

- **Minimal scope** — Reduces risk surface.
- **Bounds validation** — Prevents overruns.
- **Pinning discipline** — Use `fixed`.
- **`Span<T>` preference** — Safe alternative.
- **No pointer escape** — Keep pointers inside `fixed`.
- **Thread safety** — Avoid cross-thread locals.
- **No private API access** — Stability and security.

#### Rules and Constraints

- **Do** keep unsafe blocks as small as possible.
- **Do** validate all pointer accesses.
- **Do** prefer `Span<T>` for safe memory access.
- **Do** use `Debug.Assert` to document invariants.
- **Do not** expose pointers outside `fixed`.
- **Do not** access locals across threads.
- **Do not** invoke non-public members via unsafe code .
- **Do not** remove bounds checks without measuring; the JIT is continually improving at eliminating them .

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Scoped Unsafe Code

```csharp
using System;

class Program
{
    static unsafe int Sum(byte* data, int length)
    {
        if (length < 0) throw new ArgumentOutOfRangeException(nameof(length));
        int sum = 0;
        for (int i = 0; i < length; i++)
            sum += data[i];
        return sum;
    }

    static unsafe void Main()
    {
        byte[] bytes = { 1, 2, 3, 4 };
        fixed (byte* p = bytes)
            Console.WriteLine(Sum(p, bytes.Length));   // 10
    }
}
```

**Expected Output**
```
10
```

**Why this output occurs**
The unsafe block is minimal, bounds are validated, and the pointer does not escape the `fixed` scope.

---

### Real-World Cases

- **Interop libraries** — Minimal unsafe blocks with validation.
- **High-performance code** — Measure before removing bounds checks.

---

### References

- Unsafe code best practices — .NET — https://learn.microsoft.com/en-us/dotnet/standard/unsafe-code/best-practices
- Unsafe code, pointers to data, and function pointers — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code
- Resolve errors related to unsafe code — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/unsafe-code-errors

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `unsafe` context | Original model (C# 1.0) is default; updated memory safety model is **preview in C# 15 / .NET 11** . |
| `fixed` outside `unsafe` | **Preview** in updated model; pointer indirection still requires `unsafe` . |
| `sizeof` outside `unsafe` | **Preview** for unmanaged types in updated model . |
| `stackalloc` pointer | **Requires `unsafe`** in original model; `Span<T>` form is safe . |
| `AllowUnsafeBlocks` | **Required** to compile unsafe code . |
| Function pointers | **Requires `unsafe`**; native targets need `[UnmanagedCallersOnly]`. |
| Cross-thread locals | **Undefined behaviour**; avoid . |
| Unchecked bounds | **Security risk**; the JIT may already eliminate checks . |

---

## References (Consolidated)

- Unsafe code, pointers to data, and function pointers — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code
- Unsafe code best practices — .NET — https://learn.microsoft.com/en-us/dotnet/standard/unsafe-code/best-practices
- fixed statement — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/fixed
- stackalloc expression — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/stackalloc
- sizeof operator — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/sizeof
- Pointer types — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code#pointer-types
- Pointer related operators — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/pointer-related-operators
- Function pointers — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code#function-pointers
- AllowUnsafeBlocks — C# compiler options — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/language#allowunsafeblocks
- unsafe keyword — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/unsafe
- UnmanagedCallersOnlyAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.unmanagedcallersonlyattribute
- Resolve errors related to unsafe code — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/unsafe-code-errors
- stackalloc — C# language specification — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions

---