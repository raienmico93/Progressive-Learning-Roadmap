# C# JIT Compilation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**Just-In-Time (JIT) compilation** is the process by which the .NET runtime compiles **Intermediate Language (IL)** into **native machine code at runtime**, on demand, as methods are first called. This is the default compilation model for .NET applications. The JIT compiler (RyuJIT in modern .NET) balances startup performance, code quality, and platform portability by deferring native code generation until it is actually needed, and by optimizing based on the actual runtime environment and usage patterns.

**Technical Definition**
When a .NET assembly is executed, the CLR loads the IL and metadata. On first invocation of a method, the JIT compiler translates the method's IL into native code, which is then cached for subsequent calls. Modern .NET uses **tiered compilation** (introduced in .NET Core 2.1/3.0) to balance startup speed and steady-state throughput: methods initially compile with **Tier 0** (quick, unoptimized) and are later **recompiled with Tier 1** (fully optimized) when they are deemed "hot" by call counting. **Dynamic PGO** (.NET 7+) further improves Tier 1 by collecting profile data from Tier 0 execution. Alternatives to JIT include **ReadyToRun (R2R)** (AOT-compiled IL embedded in the assembly for faster startup) and **Native AOT** (.NET 7+) (fully native compilation with no JIT at runtime). The JIT is architecture-aware (x64, ARM64) and performs a wide range of optimizations: inlining, devirtualization, loop unrolling, bounds-check elimination, constant folding, and more.

**Beginner-Friendly Explanation**
JIT compilation is **"compile as you go."** When you build a C# program, the compiler produces **IL** (a platform-neutral intermediate language). When you run the program, the JIT compiler translates each method's IL into native machine code **the first time that method is called**. It then caches the native code, so subsequent calls are fast. This is why .NET apps can run on Windows, Linux, and macOS from the same assembly—the JIT tailors the native code to the actual CPU and OS. Modern .NET adds **tiered compilation**: methods are first compiled quickly (Tier 0) so startup is fast, then recompiled with full optimizations (Tier 1) if they turn out to be hot. **ReadyToRun** precompiles IL to native code at build time for faster startup, and **Native AOT** compiles the entire app to native code ahead of time, eliminating the JIT entirely.

---

### Key Characteristics

- **Runtime compilation** — IL is compiled to native code at runtime, on first call.
- **Method-level granularity** — Each method is compiled independently and cached.
- **Platform-adaptive** — The JIT targets the actual CPU and OS at runtime.
- **Tiered compilation** — Tier 0 (quick) then Tier 1 (optimized) for hot methods.
- **Dynamic PGO** — Profile-guided optimization based on real execution data (.NET 7+).
- **Rich optimizations** — Inlining, devirtualization, bounds-check elimination, constant folding, loop optimizations.
- **Startup cost** — JIT compilation adds startup latency (mitigated by R2R/Native AOT).
- **ReadyToRun (R2R)** — AOT-compiled IL for faster startup; JIT can still re-optimize hot methods.
- **Native AOT** — Fully native compilation; no JIT at runtime; requires trim-compatible code.

---

### Prerequisites

- Understanding of the **CLR** and the managed execution process.
- Familiarity with **IL** (Intermediate Language) and **assemblies**.
- Awareness of **assemblies**, **metadata**, and **method invocation**.
- Basic understanding of **performance profiling** (helpful).
- Knowledge of **GC** and **memory management** (helpful).

---

### Related Programming Areas

- **CLR architecture** — JIT is a core CLR component.
- **Performance engineering** — JIT optimizations affect throughput and latency.
- **Deployment** — ReadyToRun and Native AOT affect startup and size.
- **AOT compatibility** — Trimming, reflection restrictions, source generators.
- **Diagnostics** — JIT events, `DOTNET_JitDisasm`, and profiling tools.

---

### Core Concepts / Features

1. JIT Compilation (Overarching Concept)
2. Tiered Compilation
3. ReadyToRun (R2R)
4. AOT Compilation
5. Native AOT
6. JIT Optimization
7. JIT Best Practices

---

## 1. JIT Compilation (Overarching Concept)

### Definitions

**Core Definition**
JIT compilation is the runtime translation of IL to native machine code, performed on demand as methods are called.

**Technical Definition**
When a method is first invoked, the CLR's JIT compiler (RyuJIT) translates its IL into native code and caches it. Subsequent calls use the cached native code. The JIT is architecture-aware and performs optimizations. JIT compilation is the default execution model for .NET; alternatives include ReadyToRun and Native AOT.

**Beginner-Friendly Explanation**
JIT compilation is **"compile as you go"**—IL is compiled to native code the first time a method is called.

---

### Purposes

- To enable **platform portability** (same IL runs on any OS/CPU).
- To optimize for the **actual runtime environment**.
- To balance **startup performance** and **code quality**.
- To support **dynamic scenarios** (reflection, `Expression.Compile`).

---

### Syntax Rules and Structure (Execution Pipeline)

```
C# source → IL + metadata → CLR loads assembly → JIT compiles method on first call → native code cached
```

**Component breakdown**

- **IL** — Intermediate Language.
- **JIT** — RyuJIT compiler.
- **Native code** — Machine code for the actual CPU.
- **Cache** — Compiled methods are cached.

#### Syntax Rules

- Methods are JIT-compiled on first call.
- Native code is cached for subsequent calls.
- The JIT targets the actual CPU (x64, ARM64).

#### Constraints and Limitations

- JIT adds startup latency.
- JIT cannot optimize across methods that haven't been compiled yet (except via inlining).
- Native AOT eliminates JIT but restricts dynamic features.

---

### Annotated Code Examples and Expected Outputs

#### Example: Observing JIT Compilation

```csharp
using System;
using System.Diagnostics;

class Program
{
    static void Main()
    {
        var sw = Stopwatch.StartNew();
        long result = Compute(1000);   // first call: JIT compilation
        sw.Stop();
        Console.WriteLine($"First call: {sw.ElapsedMilliseconds} ms, result={result}");

        sw.Restart();
        result = Compute(1000);        // cached native code
        sw.Stop();
        Console.WriteLine($"Second call: {sw.ElapsedMilliseconds} ms");
    }

    static long Compute(int n)
    {
        long sum = 0;
        for (int i = 0; i < n; i++) sum += i;
        return sum;
    }
}
```

**Expected Output (approximate)**
```
First call: 2 ms, result=499500
Second call: 0 ms
```

**Why this output occurs**
The first call triggers JIT compilation; the second uses cached native code.

---

### Real-World Cases

- **Any .NET application** — Uses JIT by default.
- **Dynamic scenarios** — `Expression.Compile` relies on JIT.
- **Cross-platform apps** — Same IL, JIT adapts to the platform.

---

### References

- Managed execution process — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/managed-execution-process
- JIT compilation — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/runtime-config/compilation

---

## 2. Tiered Compilation

### Definitions

**Core Definition**
Tiered compilation is a .NET feature that **compiles methods quickly (Tier 0) on first call and recompiles them with full optimizations (Tier 1) once they become hot**.

**Technical Definition**
Introduced in .NET Core 2.1 (default in .NET Core 3.0), tiered compilation balances startup speed and steady-state throughput. Methods are initially compiled with **Tier 0** (quick, minimal optimizations) so startup is fast. The JIT tracks method call counts and, when a method is deemed "hot" (by default, after 30 calls with call counting, or 20 with OSR), recompiles it with **Tier 1** (fully optimized). **Dynamic PGO** (.NET 7+) collects profile data during Tier 0 execution and uses it to optimize Tier 1. **On-Stack Replacement (OSR)** allows long-running Tier 0 methods to be replaced with Tier 1 code without waiting for the method to return.

**Beginner-Friendly Explanation**
Tiered compilation is **"quick first, optimize later."** Methods start fast (Tier 0); if they are called often, the JIT recompiles them with full optimizations (Tier 1).

---

### Purposes

- To **reduce startup time** by compiling quickly first.
- To **improve steady-state performance** by optimizing hot methods.
- To use **real execution data** (Dynamic PGO) for better optimizations.
- To replace long-running methods with optimized code (OSR).

---

### Syntax Rules and Structure (Tiered Flow)

```
Method call → Tier 0 (quick compile) → call counting → Tier 1 (optimized compile) → cached
```

**Component breakdown**

- **Tier 0** — Quick compilation with minimal optimizations.
- **Tier 1** — Fully optimized compilation.
- **Dynamic PGO** — Profile-guided optimization.
- **OSR** — On-Stack Replacement for long-running methods.

#### Syntax Rules

- Tiered compilation is enabled by default.
- Configure via `TieredCompilation`, `TieredPGO`, `TC_QuickJit`, etc.
- Disable with `<TieredCompilation>false</TieredCompilation>` if needed.

#### Constraints and Limitations

- Tier 0 code is less optimized (slower) than Tier 1.
- Tier 1 compilation adds some overhead later.
- Disabling tiered compilation may hurt startup.

---

### Annotated Code Examples and Expected Outputs

#### Example: Configuring Tiered Compilation

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <TieredCompilation>true</TieredCompilation>
    <TieredPGO>true</TieredPGO>
  </PropertyGroup>
</Project>
```

**Expected Output (conceptual)**
```
// Faster startup (Tier 0), better steady-state throughput (Tier 1 + Dynamic PGO).
```

**Why this configuration matters**
Tiered compilation and Dynamic PGO balance startup and throughput.

---

### Real-World Cases

- **ASP.NET Core** — Enabled by default; important for both startup and throughput.
- **Desktop apps** — Faster startup.
- **Long-running services** — Optimized hot methods.

---

### References

- Tiered compilation — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/runtime-config/compilation
- Dynamic PGO — .NET 7 — https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-7

---

## 3. ReadyToRun (R2R)

### Definitions

**Core Definition**
ReadyToRun (R2R) is an **ahead-of-time (AOT) compilation format** that embeds precompiled native code in an assembly to improve startup performance, while still allowing the JIT to re-optimize hot methods.

**Technical Definition**
ReadyToRun (R2R) is a form of AOT compilation where IL is precompiled to native code at build time and stored in the assembly. At runtime, the precompiled code is used directly, avoiding JIT cost for those methods. The JIT can still recompile hot methods (via tiered compilation) for better optimization. R2R reduces startup time and is compatible with most .NET features (unlike Native AOT). It is enabled with `<PublishReadyToRun>true</PublishReadyToRun>`.

**Beginner-Friendly Explanation**
R2R is **"precompiled IL"**—native code is embedded in the assembly, reducing JIT cost at startup.

---

### Purposes

- To **reduce startup time**.
- To **preserve JIT re-optimization** for hot methods.
- To improve performance without AOT compatibility restrictions.

---

### Syntax Rules and Structure

```xml
<PropertyGroup>
  <PublishReadyToRun>true</PublishReadyToRun>
</PropertyGroup>
```

**Component breakdown**

- `PublishReadyToRun` — Enables R2R compilation at publish time.
- Native code embedded in the assembly.

#### Syntax Rules

- R2R is enabled at publish time.
- The JIT can still recompile hot methods.
- R2R assemblies are larger (native code included).

#### Constraints and Limitations

- Increases assembly size.
- Not as fast as Native AOT for startup.
- Some platform-specific code may still require JIT.

---

### Annotated Code Examples and Expected Outputs

#### Example: R2R Configuration

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <PublishReadyToRun>true</PublishReadyToRun>
  </PropertyGroup>
</Project>
```

**Expected Output (conceptual)**
```
// Faster startup; hot methods still re-optimized by JIT.
```

**Why this configuration matters**
R2R embeds precompiled native code, reducing JIT cost at startup.

---

### Real-World Cases

- **Desktop apps** — Faster startup.
- **ASP.NET Core** — Reduced cold-start latency.
- **Microservices** — Faster container startup.

---

### References

- ReadyToRun compilation — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/deploying/ready-to-run

---

## 4. AOT Compilation

### Definitions

**Core Definition**
Ahead-of-Time (AOT) compilation is the process of **compiling IL to native code before execution**, rather than at runtime.

**Technical Definition**
AOT compilation produces native code at build time, eliminating or reducing JIT compilation at runtime. .NET supports several AOT approaches: **ReadyToRun (R2R)** (partial AOT with JIT fallback), **Native AOT** (fully native, no JIT), and **Mono AOT** (used in Xamarin/MAUI, WebAssembly). AOT improves startup time, reduces memory usage, and enables scenarios where JIT is unavailable (iOS, WebAssembly, some embedded systems). AOT requires code to be **trim-compatible** and may restrict dynamic features (reflection, `Reflection.Emit`).

**Beginner-Friendly Explanation**
AOT is **"compile ahead of time"**—native code is produced at build time, not at runtime.

---

### Purposes

- To **eliminate JIT startup cost**.
- To **reduce memory usage** (no JIT).
- To enable **platforms without JIT** (iOS, WebAssembly).
- To support **faster container startup**.

---

### Syntax Rules and Structure

AOT is configured via project properties:

```xml
<PublishAot>true</PublishAot>                 <!-- Native AOT -->
<PublishReadyToRun>true</PublishReadyToRun>   <!-- R2R -->
```

**Component breakdown**

- `PublishAot` — Enables Native AOT.
- `PublishReadyToRun` — Enables R2R.
- Trimming — Required for Native AOT.

#### Syntax Rules

- Native AOT requires trimming-compatible code.
- Reflection and `Reflection.Emit` are restricted.
- Some libraries may not be AOT-compatible.

#### Constraints and Limitations

- Increased build time.
- Restrictions on dynamic code.
- Larger binary size (Native AOT).

---

### Annotated Code Examples and Expected Outputs

#### Example: Native AOT Configuration

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <PublishAot>true</PublishAot>
  </PropertyGroup>
</Project>
```

**Expected Output (conceptual)**
```
// Fully native executable; no JIT at runtime.
```

**Why this configuration matters**
Native AOT produces a self-contained native executable with fast startup and low memory.

---

### Real-World Cases

- **CLI tools** — Fast startup and small memory.
- **Serverless** — Reduced cold-start latency.
- **Mobile apps** — AOT on iOS (no JIT).

---

### References

- Native AOT deployment — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/
- AOT compilation — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/deploying/

---

## 5. Native AOT

### Definitions

**Core Definition**
Native AOT is a .NET deployment model that **compiles the entire application to native code ahead of time**, producing a self-contained native executable with **no JIT at runtime**.

**Technical Definition**
Native AOT (introduced in .NET 7) compiles IL to native code at build time, links the runtime libraries, and produces a native executable. At runtime, there is **no JIT**; all code is native. Native AOT enables fast startup, low memory, and small binaries (when trimmed), but requires **trim-compatible** code: reflection on trimmed types, `Reflection.Emit`, and some dynamic features are restricted. Source generators (e.g., `System.Text.Json`, `LibraryImport`) are commonly used to make libraries AOT-compatible.

**Beginner-Friendly Explanation**
Native AOT is **"compile everything to native code ahead of time"**—no JIT at runtime.

---

### Purposes

- To **eliminate JIT startup cost**.
- To **reduce memory usage**.
- To produce **self-contained native executables**.
- To support **platforms without JIT** (iOS, WebAssembly).

---

### Syntax Rules and Structure

```xml
<PropertyGroup>
  <PublishAot>true</PublishAot>
</PropertyGroup>
```

**Component breakdown**

- `PublishAot` — Enables Native AOT.
- Trimming is enabled automatically.
- The output is a native executable.

#### Syntax Rules

- Requires trim-compatible code.
- Reflection on trimmed members is restricted.
- `Reflection.Emit` is not supported.

#### Constraints and Limitations

- No dynamic code loading.
- Some libraries are not AOT-compatible.
- Binary size can be larger than JIT apps (unless trimmed aggressively).

---

### Annotated Code Examples and Expected Outputs

#### Example: AOT-Compatible P/Invoke with `LibraryImport`

```csharp
using System.Runtime.InteropServices;

public static partial class NativeMethods
{
    [LibraryImport("native", EntryPoint = "MyNativeFunction")]
    public static partial int MyNativeFunction(int x);
}
```

**Expected Output (conceptual)**
```
// Source-generated P/Invoke; AOT-compatible.
```

**Why this is important**
`LibraryImport` generates marshaling code at compile time, making it AOT-friendly (unlike `DllImport`).

---

### Real-World Cases

- **AWS Lambda** — Native AOT for fast cold starts.
- **CLI tools** — Self-contained native binaries.
- **iOS apps** — Native AOT (no JIT allowed).

---

### References

- Native AOT deployment — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/
- AOT compatibility — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/#aot-compatibility

---

## 6. JIT Optimization

### Definitions

**Core Definition**
JIT optimization is the set of **code transformations** the JIT performs to improve native code performance, including inlining, devirtualization, bounds-check elimination, and more.

**Technical Definition**
RyuJIT performs a wide range of optimizations: **inlining** (small methods), **devirtualization** (calling non-virtual methods directly when the type is known), **bounds-check elimination** (removing redundant array bounds checks), **constant folding**, **loop unrolling**, **strength reduction**, **copy propagation**, **dead code elimination**, **escape analysis** (stack allocation in .NET 9+), and **intrinsic recognition** (e.g., `Span<T>`, `Math` functions). Tiered compilation and Dynamic PGO further improve optimization quality over time.

**Beginner-Friendly Explanation**
JIT optimization is **the JIT's performance tricks**—inlining small methods, removing bounds checks, and devirtualizing calls.

---

### Purposes

- To **improve native code quality**.
- To **reduce overhead** (bounds checks, virtual calls).
- To **exploit runtime information** (Dynamic PGO).

---

### Syntax Rules and Structure (Optimization Examples)

| Optimization | Description |
|---|---|
| Inlining | Replaces calls with method body |
| Devirtualization | Direct call when type is known |
| Bounds-check elimination | Removes redundant checks |
| Escape analysis | Stack-allocates non-escaping objects (.NET 9+) |
| Intrinsics | Uses CPU instructions for `Span`, `Math` |

**Component breakdown**

- **Inlining** — Small methods are inlined.
- **Devirtualization** — Sealed types enable direct calls.
- **Escape analysis** — Objects that don't escape may be stack-allocated.

#### Syntax Rules

- Sealed types and `readonly struct` enable better optimization.
- `Span<T>` operations are intrinsic and vectorized.
- Tier 1 + Dynamic PGO produce the best code.

#### Constraints and Limitations

- Optimization quality varies by method size and complexity.
- Some optimizations (escape analysis) are new and limited.

---

### Annotated Code Examples and Expected Outputs

#### Example: Sealed Class Enables Devirtualization

```csharp
using System;

public class Base
{
    public virtual int Compute(int x) => x * 2;
}

public sealed class Derived : Base
{
    public override int Compute(int x) => x * 3;
}

class Program
{
    static void Main()
    {
        Base b = new Derived();
        Console.WriteLine(b.Compute(10));   // 30
    }
}
```

**Expected Output**
```
30
```

**Why this output occurs**
`Derived` is sealed, so the JIT can devirtualize `Compute` when the type is known.

---

### Real-World Cases

- **Hot loops** — Inlining and bounds-check elimination.
- **Sealed types** — Devirtualization.
- **`Span<T>`** — Intrinsic vectorized operations.

---

### References

- JIT optimization — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/runtime-config/compilation
- Performance improvements in .NET 9 — .NET Blog — https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-9/

---

## 7. JIT Best Practices

### Definitions

**Core Definition**
JIT best practices are the guidelines for writing code that the JIT can optimize effectively and for choosing the right compilation model (JIT, R2R, Native AOT) for the scenario.

**Technical Definition**
Best practices include: **keep methods small and simple** (easier inlining), **use sealed types and `readonly struct`** (devirtualization), **avoid unnecessary allocations** (reduce GC pressure), **use `Span<T>` and `Memory<T>`** (intrinsics), **enable tiered compilation and Dynamic PGO** (default), **use ReadyToRun for startup**, **use Native AOT for minimal startup and memory**, **avoid reflection-heavy code in AOT scenarios**, **use source generators** (e.g., `System.Text.Json`, `LibraryImport`), and **measure with BenchmarkDotNet** before micro-optimizing.

**Beginner-Friendly Explanation**
The best practices are: **keep methods small**, **use sealed types**, **use `Span<T>`**, **enable tiered compilation**, and **choose the right compilation model**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Keep methods small for inlining
public static int Add(int a, int b) => a + b;

// 2. Use sealed types for devirtualization
public sealed class Service { ... }

// 3. Use readonly struct for immutability and JIT friendliness
public readonly struct Point { public readonly int X, Y; }

// 4. Use Span<T> for allocation-free operations
ReadOnlySpan<char> slice = text.AsSpan(start, length);

// 5. Avoid unnecessary allocations
// Use ArrayPool<T>, stackalloc, and structs.
```

```xml
<!-- 6. Enable tiered compilation and Dynamic PGO (default) -->
<TieredCompilation>true</TieredCompilation>
<TieredPGO>true</TieredPGO>

<!-- 7. Use ReadyToRun for startup -->
<PublishReadyToRun>true</PublishReadyToRun>

<!-- 8. Use Native AOT for minimal startup -->
<PublishAot>true</PublishAot>
```

**Component breakdown**

- **Small methods** — Inlinable.
- **Sealed types** — Devirtualizable.
- **`Span<T>`** — Intrinsic.
- **Tiered compilation** — Default.
- **R2R / Native AOT** — Startup optimization.

#### Rules and Constraints

- **Do** keep methods small and simple.
- **Do** use sealed types and `readonly struct`.
- **Do** use `Span<T>` / `Memory<T>` for buffers.
- **Do** enable tiered compilation and Dynamic PGO.
- **Do not** rely on dynamic code in Native AOT.
- **Do not** micro-optimize without measuring.

---

### Annotated Code Examples and Expected Outputs

#### Example: JIT-Friendly Code

```csharp
using System;

public sealed class Calculator
{
    public int Add(int a, int b) => a + b;   // small, inlinable
}

class Program
{
    static void Main()
    {
        var calc = new Calculator();
        Console.WriteLine(calc.Add(3, 4));   // 7
    }
}
```

**Expected Output**
```
7
```

**Why this output occurs**
The sealed class and small method enable inlining and devirtualization.

---

### Real-World Cases

- **High-performance libraries** — Sealed types, spans, intrinsics.
- **Startup-sensitive apps** — R2R, Native AOT.
- **Cloud/Serverless** — Native AOT for cold starts.

---

### References

- JIT compilation — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/runtime-config/compilation
- ReadyToRun compilation — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/deploying/ready-to-run
- Native AOT deployment — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/
- Performance improvements in .NET 9 — .NET Blog — https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-9/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| JIT (RyuJIT) | Default in CoreCLR; stable. |
| Tiered compilation | **.NET Core 3.0+**; default. |
| Dynamic PGO | **.NET 7+**; default with tiered compilation. |
| On-Stack Replacement (OSR) | **.NET Core 3.0+**; allows Tier 1 replacement. |
| ReadyToRun (R2R) | **.NET Core 3.0+**; stable. |
| Native AOT | **.NET 7+**; stable. |
| `Reflection.Emit` | **Not supported** in Native AOT. |
| `DllImport` | **Legacy**; prefer source-generated `LibraryImport`. |
| Escape analysis (stack allocation) | **.NET 9+**; limited. |

---

## References (Consolidated)

- Managed execution process — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/managed-execution-process
- JIT compilation — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/runtime-config/compilation
- Tiered compilation — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/runtime-config/compilation
- ReadyToRun compilation — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/deploying/ready-to-run
- Native AOT deployment — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/
- AOT compatibility — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/#aot-compatibility
- Performance improvements in .NET 9 — .NET Blog — https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-9/

---