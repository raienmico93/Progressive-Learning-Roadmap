# C# Dynamic Programming — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Dynamic programming in C# refers to the use of the `dynamic` keyword and the **Dynamic Language Runtime (DLR)** to **defer type checking and member resolution from compile time to runtime**. A variable declared as `dynamic` bypasses static type checking, allowing any operation to be attempted syntactically; the operation is resolved and executed at runtime through a runtime binder. This enables C# to interoperate with dynamic languages, COM APIs, and scenarios where the shape of data is not known until execution.

**Technical Definition**
The `dynamic` type is a **static type** (it appears in signatures, fields, and generics), but expressions of type `dynamic` **bypass static type checking**. Instead, the compiler packages information about the intended operation into a **call site** (`CallSite<T>`), and the DLR resolves the operation at runtime using a **runtime binder** (`Microsoft.CSharp.RuntimeBinder`). The DLR is a runtime environment layered on top of the CLR that provides **expression trees**, **dynamic dispatch**, and **call-site caching** for dynamic languages and for C#’s `dynamic` feature. The DLR enables interoperability between C# and dynamic languages (IronPython, IronRuby, JavaScript), COM automation, and dynamic data shapes (`ExpandoObject`, `DynamicObject`).

**Beginner-Friendly Explanation**
Normally, C# checks everything at compile time: you cannot call a method that does not exist, and you cannot assign a `string` to an `int`. The `dynamic` keyword turns off those checks for a specific variable. The compiler says “I will not verify this; the runtime will figure it out.” At runtime, the DLR looks at the actual object, finds the right method or property, and executes it. If the operation is invalid, you get a `RuntimeBinderException` instead of a compiler error. This is powerful for **talking to Python, JavaScript, COM, or JSON data with unknown shape**, but it comes at the cost of **type safety, performance, and tooling support**. Use it as a **boundary tool**, not as a replacement for static types.

---

### Key Characteristics

- **Runtime binding** — Operations on `dynamic` are resolved at runtime, not compile time.
- **Static type, dynamic behaviour** — `dynamic` is a static type that behaves like `object` but with runtime dispatch.
- **DLR-powered** — The Dynamic Language Runtime provides call-site caching, expression trees, and dynamic dispatch.
- **Interop-focused** — Designed for COM, dynamic languages, and dynamic data shapes.
- **No IntelliSense** — The IDE cannot provide member completion for `dynamic`.
- **Performance cost** — Significantly slower than static calls; boxes value types; incurs DLR overhead.
- **Runtime failures** — Invalid operations throw `RuntimeBinderException` at runtime.
- **`ExpandoObject` and `DynamicObject`** — Two built-in ways to create dynamic objects with custom member resolution.

---

### Prerequisites

- Solid understanding of C# static typing (`object`, generics, interfaces).
- Familiarity with **reflection** (the DLR is a higher-level alternative).
- Basic knowledge of **COM interop** or dynamic languages (helpful).
- Understanding of **expression trees** (the DLR uses them internally).
- Awareness of **exception handling** (`RuntimeBinderException`).

---

### Related Programming Areas

- **Interoperability** — COM, IronPython, IronRuby, JavaScript engines.
- **Reflection** — The DLR is a faster, cached alternative to raw reflection.
- **Dynamic data** — JSON, XML, and NoSQL documents with variable shapes.
- **Scripting** — Embedding dynamic languages in .NET applications.
- **API design** — `ExpandoObject` and `DynamicObject` for flexible property bags.

---

### Core Concepts / Features

1. `dynamic`
2. Dynamic Binding
3. Dynamic Language Runtime (DLR)
4. `ExpandoObject`
5. `DynamicObject`
6. Dynamic Performance
7. Dynamic Best Practices

---

## 1. `dynamic`

### Definitions

**Core Definition**
`dynamic` is a static type that bypasses compile-time type checking; operations on a `dynamic` expression are bound at runtime.

**Technical Definition**
The `dynamic` type is a static type, but an object of type `dynamic` bypasses static type checking. It is similar to `object` but with runtime binding: the compiler packages operation information into a call site, and the DLR resolves the operation at runtime. You cannot use `new dynamic()`, and you cannot derive from `dynamic`; `dynamic` represents runtime-binding behaviour, not a specific .NET type alias. The result of most dynamic operations is itself `dynamic`; exceptions include conversions and constructor calls with dynamic arguments.

**Beginner-Friendly Explanation**
`dynamic` is **“trust me, compiler—figure it out at runtime.”** You can write `d.Foo()` without the compiler knowing whether `Foo` exists. At runtime, the DLR checks and throws if it does not.

---

### Purposes

- To **interoperate** with dynamic languages, COM, and dynamic data sources.
- To **defer member resolution** when the type is not known at compile time.
- To **simplify code** that would otherwise require extensive reflection.
- To **bridge** statically typed and dynamically typed worlds.

---

### Syntax Rules and Structure

```csharp
dynamic variable = expression;
```

**Component breakdown**

- `dynamic` — The contextual keyword.
- `variable` — The name.
- `expression` — Any expression; its operations are bound at runtime.

#### Syntax Rules

- `dynamic` can be used for locals, fields, properties, parameters, return types, and generic type arguments.
- Operations on `dynamic` are resolved at runtime by the DLR.
- The compiler packages operation metadata into call sites.
- You cannot use `new dynamic()` or derive from `dynamic`.

#### Constraints and Limitations

- No compile-time type checking for dynamic operations.
- No IntelliSense for dynamic members.
- Runtime errors (`RuntimeBinderException`) replace compiler errors.
- Performance overhead due to DLR binding and caching.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic `dynamic` Usage

```csharp
using System;

class Program
{
    static void Main()
    {
        dynamic d = "Hello";
        Console.WriteLine(d.Length);   // 5 (runtime resolution)

        d = 42;
        Console.WriteLine(d + 1);      // 43 (runtime resolution)
    }
}
```

**Expected Output**
```
5
43
```

**Why this output occurs**
The DLR resolves `Length` on the string and `+` on the integer at runtime, using the actual runtime type of `d`.

---

#### Example 2: Runtime Failure

```csharp
using System;

class Program
{
    static void Main()
    {
        dynamic d = 42;
        try
        {
            Console.WriteLine(d.Length);   // RuntimeBinderException
        }
        catch (Microsoft.CSharp.RuntimeBinder.RuntimeBinderException ex)
        {
            Console.WriteLine($"Binding failed: {ex.Message}");
        }
    }
}
```

**Expected Output**
```
Binding failed: 'int' does not contain a definition for 'Length'
```

**Why this output occurs**
`int` has no `Length` member; the DLR throws `RuntimeBinderException` at runtime.

---

### Real-World Cases

- **COM automation** — Office interop without verbose reflection.
- **Dynamic languages** — Calling IronPython/IronRuby from C#.
- **JSON/NoSQL data** — Accessing document properties by name.
- **Rapid prototyping** — Quick scripts where types are fluid.

---

### References

- Using type dynamic — C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interop/using-type-dynamic
- Built-in types — C# reference (dynamic) — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/built-in-types
- dynamic (C# Reference) — dotnet/docs (GitHub) — https://github.com/dotnet/docs/blob/main/docs/csharp/language-reference/keywords/dynamic.md

---

## 2. Dynamic Binding

### Definitions

**Core Definition**
Dynamic binding is the runtime resolution of operations on `dynamic` expressions, performed by the DLR’s runtime binder.

**Technical Definition**
When an operation involves a `dynamic` expression, the C# compiler emits a **call site** containing the operation’s metadata (member name, argument types, context). At runtime, the DLR’s `RuntimeBinder` resolves the operation using the actual runtime types of the operands. Overload resolution, member lookup, and conversions are performed at runtime. Call-site caching stores the resolution result so repeated calls with the same types are fast.

**Beginner-Friendly Explanation**
Dynamic binding is **“the runtime decides which method to call.”** The compiler leaves a note (call site), and the DLR uses it to find the right member when the code runs.

---

### Purposes

- To resolve members **based on actual runtime types**.
- To support **overload resolution at runtime**.
- To enable **late binding** without manual reflection.
- To cache binding results for performance.

---

### Syntax Rules and Structure (Binding Flow)

```
dynamic expression → compiler emits CallSite<T> → DLR RuntimeBinder resolves → result cached → operation executed
```

**Component breakdown**

- **`CallSite<T>`** — Compiled call site with caching.
- **`RuntimeBinder`** — Resolves operations at runtime (`Microsoft.CSharp.RuntimeBinder`).
- **Cache** — Stores resolutions by runtime type signature.

#### Syntax Rules

- Binding is triggered when any operand is `dynamic`.
- Overload resolution uses **runtime types**.
- The result of most dynamic operations is `dynamic`.
- Conversions to a static type are resolved at runtime.

#### Constraints and Limitations

- Binding failures throw `RuntimeBinderException`.
- Caching is per-call-site; many distinct types reduce cache efficiency.

---

### Annotated Code Examples and Expected Outputs

#### Example: Runtime Overload Resolution

```csharp
using System;

class Program
{
    static void M(int x) => Console.WriteLine("int");
    static void M(string x) => Console.WriteLine("string");

    static void Main()
    {
        dynamic d = "hello";
        M(d);   // string (runtime overload resolution)
    }
}
```

**Expected Output**
```
string
```

**Why this output occurs**
The DLR inspects the runtime type of `d` (`string`) and selects the `M(string)` overload at runtime.

---

### Real-World Cases

- **COM interop** — Runtime dispatch to COM members.
- **Dynamic proxies** — Runtime interception of calls.
- **Dynamic data binding** — Selecting members by runtime shape.

---

### References

- Microsoft.CSharp.RuntimeBinder Namespace — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.csharp.runtimebinder
- System.Dynamic Namespace — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic

---

## 3. Dynamic Language Runtime (DLR)

### Definitions

**Core Definition**
The DLR is a runtime environment that adds **dynamic language services** to the CLR, including expression trees, dynamic dispatch, and call-site caching.

**Technical Definition**
The DLR is a runtime environment that adds a set of services for dynamic languages to the CLR. Its primary components are:
- **Expression trees** — Extended LINQ expression trees representing language semantics (control flow, assignment, etc.).
- **Dynamic dispatch** — Allows different languages to communicate and resolve operations dynamically.
- **Call-site caching** — Caches binding results for performance.
- **Dynamic objects** — `IDynamicMetaObjectProvider` implementations like `ExpandoObject` and `DynamicObject` participate in DLR binding.

**Beginner-Friendly Explanation**
The DLR is **the engine behind `dynamic`**. It provides the runtime infrastructure that makes runtime binding, dynamic objects, and dynamic-language interop possible.

---

### Purposes

- To **support dynamic languages** on .NET.
- To **add dynamic features** to C#.
- To **enable interop** between static and dynamic languages.
- To **provide call-site caching** for performance.

---

### Syntax Rules and Structure (DLR Architecture)

| Component | Role |
|---|---|
| Expression Trees | Represent language semantics |
| Dynamic Dispatch | Resolve operations at runtime |
| Call-Site Caching | Cache binding results |
| `IDynamicMetaObjectProvider` | Dynamic object participation |

**Component breakdown**

- **Expression trees** — DLR extends LINQ expression trees for language modelling.
- **Dynamic dispatch** — Resolves operations based on runtime types.
- **Caching** — Improves repeated-call performance.
- **Dynamic objects** — `ExpandoObject`, `DynamicObject` implement `IDynamicMetaObjectProvider`.

#### Syntax Rules

- The DLR is part of .NET (available in `System.Dynamic` and `Microsoft.CSharp.RuntimeBinder`).
- Dynamic objects implement `IDynamicMetaObjectProvider` to participate in DLR binding.
- Call sites are compiler-generated for `dynamic` expressions.

#### Constraints and Limitations

- DLR binding is slower than static calls.
- Dynamic objects must override DLR methods (`TryGetMember`, etc.) to customize behaviour.

---

### Annotated Code Examples and Expected Outputs

#### Example: DLR Dynamic Object Participation

```csharp
using System;
using System.Dynamic;

class Program
{
    static void Main()
    {
        dynamic expando = new ExpandoObject();
        expando.Name = "Alice";   // dynamic member addition
        Console.WriteLine(expando.Name);   // Alice
    }
}
```

**Expected Output**
```
Alice
```

**Why this output occurs**
`ExpandoObject` implements `IDynamicMetaObjectProvider`; the DLR resolves `Name` dynamically.

---

### Real-World Cases

- **IronPython/IronRuby** — Hosting dynamic languages.
- **Dynamic libraries** — Building DLR-based DSLs.
- **COM interop** — Dynamic dispatch to COM objects.

---

### References

- Dynamic Language Runtime Overview — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/docs/main/docs/framework/reflection-and-codedom/dynamic-language-runtime-overview.md
- System.Dynamic Namespace — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic

---

## 4. `ExpandoObject`

### Definitions

**Core Definition**
`ExpandoObject` is a sealed class representing a **dynamic property bag** whose members can be added, removed, and accessed at runtime.

**Technical Definition**
`System.Dynamic.ExpandoObject` implements `IDynamicMetaObjectProvider`, `IDictionary<string, object>`, and `INotifyPropertyChanged`. It allows adding, deleting, and getting/setting members at runtime using natural syntax (`obj.Member = value`) rather than dictionary syntax. It is **thread-safe for member access? Not guaranteed**; it is a simple, pre-chosen dynamic dispatch behaviour.

**Beginner-Friendly Explanation**
`ExpandoObject` is a **dynamic bag** where you can add properties on the fly: `obj.Name = "Alice"; obj.Age = 30;`.

---

### Purposes

- To create **flexible property bags** at runtime.
- To **serialize/deserialize** dynamic shapes.
- To **interop** with dynamic languages.
- To support **data binding** (`INotifyPropertyChanged`).

---

### Syntax Rules and Structure

```csharp
dynamic expando = new ExpandoObject();
expando.Name = "Alice";
expando.Age = 30;
Console.WriteLine(expando.Name);
```

**Component breakdown**

- `ExpandoObject` — The dynamic bag type.
- Member assignment — Creates dynamic members.
- Member access — Reads dynamic members.

#### Syntax Rules

- Members are added on first assignment.
- Members can be removed via `IDictionary.Remove`.
- Implements `IDictionary<string, object>` for enumeration.
- Implements `INotifyPropertyChanged`.

#### Constraints and Limitations

- Members are `object`; no compile-time type safety.
- No IntelliSense for members.
- Not designed for inheritance (sealed).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Adding and Reading Members

```csharp
using System;
using System.Dynamic;

class Program
{
    static void Main()
    {
        dynamic person = new ExpandoObject();
        person.Name = "Alice";
        person.Age = 30;

        Console.WriteLine($"{person.Name} is {person.Age} years old.");
        // Alice is 30 years old.
    }
}
```

**Expected Output**
```
Alice is 30 years old.
```

**Why this output occurs**
`ExpandoObject` dynamically stores `Name` and `Age`; the DLR resolves access at runtime.

---

#### Example 2: Enumerating Members as a Dictionary

```csharp
using System;
using System.Dynamic;

class Program
{
    static void Main()
    {
        dynamic expando = new ExpandoObject();
        expando.A = 1;
        expando.B = 2;

        var dict = (IDictionary<string, object>)expando;
        foreach (var kvp in dict)
            Console.WriteLine($"{kvp.Key}: {kvp.Value}");
    }
}
```

**Expected Output**
```
A: 1
B: 2
```

**Why this output occurs**
`ExpandoObject` implements `IDictionary<string, object>`, enabling enumeration.

---

### Real-World Cases

- **JSON deserialization** — Representing dynamic JSON shapes.
- **Data binding** — `INotifyPropertyChanged` support.
- **Dynamic configuration** — Runtime property bags.

---

### References

- ExpandoObject Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic.expandoobject

---

## 5. `DynamicObject`

### Definitions

**Core Definition**
`DynamicObject` is an abstract base class that allows **full control over dynamic dispatch** by overriding `TryGetMember`, `TrySetMember`, `TryInvokeMember`, and other DLR methods.

**Technical Definition**
`System.Dynamic.DynamicObject` is an abstract class implementing `IDynamicMetaObjectProvider`. Derived classes override `TryGetMember`, `TrySetMember`, `TryInvokeMember`, `TryGetIndex`, `TrySetIndex`, and related methods to define custom runtime behaviour. To enable dynamic behaviour on instances of a derived class, the reference must be of type `dynamic`. `DynamicObject` is the advanced option for custom dispatch; `ExpandoObject` is a pre-built property bag.

**Beginner-Friendly Explanation**
`DynamicObject` is **a base class for building your own dynamic object**. You decide what happens when someone reads a property, calls a method, or uses an indexer.

---

### Purposes

- To create **custom dynamic objects** with bespoke dispatch logic.
- To **intercept** member access and method calls.
- To build **dynamic proxies** and **DSLs**.
- To **wrap** dictionaries or external APIs with property syntax.

---

### Syntax Rules and Structure

```csharp
public class MyDynamic : DynamicObject
{
    public override bool TryGetMember(GetMemberBinder binder, out object? result)
    {
        // custom logic
        result = null;
        return true;
    }

    public override bool TrySetMember(SetMemberBinder binder, object? value)
    {
        // custom logic
        return true;
    }

    public override bool TryInvokeMember(InvokeMemberBinder binder, object?[]? args, out object? result)
    {
        // custom logic
        result = null;
        return true;
    }
}
```

**Component breakdown**

- `DynamicObject` — Base class.
- `TryGetMember` — Override for property reads.
- `TrySetMember` — Override for property writes.
- `TryInvokeMember` — Override for method calls.

#### Syntax Rules

- Derive from `DynamicObject`.
- Override the required `Try*` methods.
- Use `dynamic` references to enable dispatch.
- Return `true` if handled; `false` falls back to default behaviour.

#### Constraints and Limitations

- Cannot instantiate `DynamicObject` directly.
- Must use `dynamic` references for dispatch.
- `TryCreateInstance` is not usable from C#.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Custom Dynamic Dictionary

```csharp
using System;
using System.Collections.Generic;
using System.Dynamic;

public class DynamicDictionary : DynamicObject
{
    private readonly Dictionary<string, object?> _dict = new();

    public override bool TryGetMember(GetMemberBinder binder, out object? result) =>
        _dict.TryGetValue(binder.Name, out result);

    public override bool TrySetMember(SetMemberBinder binder, object? value)
    {
        _dict[binder.Name] = value;
        return true;
    }

    public int Count => _dict.Count;
}

class Program
{
    static void Main()
    {
        dynamic d = new DynamicDictionary();
        d.Name = "Alice";
        d.Age = 30;

        Console.WriteLine(d.Name);   // Alice
        Console.WriteLine(d.Count);  // 2
    }
}
```

**Expected Output**
```
Alice
2
```

**Why this output occurs**
`TryGetMember` and `TrySetMember` route property access to the internal dictionary.

---

#### Example 2: Custom Method Invocation

```csharp
using System;
using System.Dynamic;

public class MethodDynamic : DynamicObject
{
    public override bool TryInvokeMember(InvokeMemberBinder binder, object?[]? args, out object? result)
    {
        Console.WriteLine($"Called {binder.Name} with {args?.Length ?? 0} args");
        result = null;
        return true;
    }
}

class Program
{
    static void Main()
    {
        dynamic d = new MethodDynamic();
        d.DoSomething(1, 2, 3);   // Called DoSomething with 3 args
    }
}
```

**Expected Output**
```
Called DoSomething with 3 args
```

**Why this output occurs**
`TryInvokeMember` intercepts the method call and logs the invocation.

---

### Real-World Cases

- **Dynamic proxies** — Intercepting calls for logging, caching, or authorization.
- **DSLs** — Domain-specific languages with custom syntax.
- **API wrappers** — Exposing dictionary-like APIs with property syntax.

---

### References

- DynamicObject Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic.dynamicobject
- DynamicObject.TryGetMember — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic.dynamicobject.trygetmember
- Walkthrough: Creating and Using Dynamic Objects (C#) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interop/walkthrough-creating-and-using-dynamic-objects

---

## 6. Dynamic Performance

### Definitions

**Core Definition**
Dynamic performance refers to the **runtime overhead** of dynamic binding compared to statically typed operations.

**Technical Definition**
Dynamic operations are significantly slower than static calls because they involve:
- **Call-site setup** — Compiler-generated `CallSite<T>` construction.
- **Runtime binding** — DLR resolution of members and overloads.
- **Caching** — Cached resolutions speed up repeated calls, but the first call is costly.
- **Boxing** — Value types are boxed when used dynamically.
- **No JIT inlining** — Dynamic calls cannot be inlined.

Benchmarks indicate dynamic calls can be **~20× slower than static C#** but **~12–30× faster than raw reflection** due to call-site caching. The DLR caches binding results per call site, so repeated calls with the same runtime types are much faster than the first call.

**Beginner-Friendly Explanation**
Dynamic is **fast for dynamic, but slow for static**. It is faster than reflection because it caches results, but much slower than direct calls because the runtime must still do work.

---

### Purposes

- To understand the cost of `dynamic`.
- To choose between `dynamic`, reflection, and static types.
- To optimize dynamic-heavy code with caching and delegates.

---

### Syntax Rules and Structure (Performance Comparison)

| Approach | Relative Speed | Notes |
|---|---|---|
| Static C# | Baseline (fastest) | Compile-time binding |
| `dynamic` (cached) | ~20× slower | DLR call-site caching |
| Reflection | ~12–30× slower than dynamic | No caching by default |
| Compiled expressions | Near-static | `Expression<T>.Compile()` |

**Component breakdown**

- **Static** — Direct IL call.
- **Dynamic** — DLR call site + cache.
- **Reflection** — Metadata lookup each time.
- **Compiled expressions** — JIT-compiled delegate.

#### Syntax Rules

- Cache `dynamic` call sites (automatic per site).
- Avoid `dynamic` in hot loops.
- Prefer static types at performance-critical boundaries.
- Use `Expression<T>.Compile()` for repeated dynamic invocation.

#### Constraints and Limitations

- `dynamic` boxes value types.
- First-call overhead is high; caching helps repeated calls.
- Cannot be inlined by the JIT.

---

### Annotated Code Examples and Expected Outputs

#### Example: Dynamic vs. Static Performance

```csharp
using System;
using System.Diagnostics;

class Program
{
    static void Main()
    {
        const int N = 1_000_000;

        // Static
        var sw = Stopwatch.StartNew();
        int sumStatic = 0;
        for (int i = 0; i < N; i++) sumStatic += i;
        sw.Stop();
        Console.WriteLine($"Static: {sw.ElapsedMilliseconds} ms");

        // Dynamic
        dynamic sumDynamic = 0;
        sw.Restart();
        for (int i = 0; i < N; i++) sumDynamic += i;
        sw.Stop();
        Console.WriteLine($"Dynamic: {sw.ElapsedMilliseconds} ms");
    }
}
```

**Expected Output (approximate)**
```
Static: 1 ms
Dynamic: 150 ms
```

**Why this output occurs**
Dynamic addition is resolved through DLR call sites on every iteration, with caching but still significant overhead.

---

### Real-World Cases

- **Interop boundaries** — Acceptable cost at the boundary, not in hot loops.
- **Rapid prototyping** — Trade performance for flexibility.
- **Plugin systems** — Dynamic dispatch where types are unknown.

---

### References

- Using type dynamic — C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interop/using-type-dynamic
- Dynamic Language Runtime Overview — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/docs/main/docs/framework/reflection-and-codedom/dynamic-language-runtime-overview.md

---

## 7. Dynamic Best Practices

### Guidelines

#### General Guidelines

```csharp
// 1. Use dynamic at boundaries, not in core logic
public void Process(dynamic payload) { ... }   // boundary

// 2. Prefer static types for hot paths
// dynamic in loops is expensive

// 3. Use ExpandoObject for property bags
dynamic bag = new ExpandoObject();
bag.Name = "Alice";

// 4. Use DynamicObject for custom dispatch
public class MyDynamic : DynamicObject { ... }

// 5. Validate dynamic inputs early
if (payload is null) throw new ArgumentNullException(nameof(payload));

// 6. Catch RuntimeBinderException at boundaries
try { dynamicResult = d.Foo(); }
catch (RuntimeBinderException ex) { /* fallback */ }

// 7. Prefer source generators or reflection alternatives for AOT
```

**Component breakdown**

- **Boundary use** — Dynamic for interop, static for core.
- **Hot path avoidance** — No dynamic in loops.
- **`ExpandoObject`** — Simple property bags.
- **`DynamicObject`** — Custom dispatch.
- **Validation** — Guard against null and invalid shapes.
- **Exception handling** — Catch binding failures.
- **AOT awareness** — Prefer source generators.

#### Rules and Constraints

- **Do** use `dynamic` at interoperability boundaries.
- **Do** use `ExpandoObject` for dynamic property bags.
- **Do** use `DynamicObject` for custom dispatch.
- **Do** validate dynamic inputs.
- **Do not** use `dynamic` in hot paths.
- **Do not** rely on `dynamic` for core application logic.
- **Do not** assume compile-time safety for dynamic operations.

---

### Annotated Code Examples and Expected Outputs

#### Example: Boundary-Oriented Dynamic Usage

```csharp
using System;
using System.Dynamic;

public static class PayloadProcessor
{
    public static void Process(dynamic payload)
    {
        if (payload is null) throw new ArgumentNullException(nameof(payload));

        try
        {
            string name = payload.Name ?? "Unknown";
            Console.WriteLine($"Processing {name}");
        }
        catch (Microsoft.CSharp.RuntimeBinder.RuntimeBinderException ex)
        {
            Console.WriteLine($"Invalid payload: {ex.Message}");
        }
    }
}

class Program
{
    static void Main()
    {
        dynamic p = new ExpandoObject();
        p.Name = "Alice";
        PayloadProcessor.Process(p);   // Processing Alice
    }
}
```

**Expected Output**
```
Processing Alice
```

**Why this output occurs**
`dynamic` is used at the boundary, with null validation and `RuntimeBinderException` handling; the core logic remains statically typed where possible.

---

### Real-World Cases

- **COM interop** — `dynamic` for Office automation.
- **JSON APIs** — `ExpandoObject` for unknown shapes.
- **Plugin systems** — `DynamicObject` for custom proxies.
- **Dynamic languages** — IronPython/IronRuby integration.

---

### References

- Using type dynamic — C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interop/using-type-dynamic
- Dynamic Language Runtime Overview — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/docs/main/docs/framework/reflection-and-codedom/dynamic-language-runtime-overview.md
- ExpandoObject Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic.expandoobject
- DynamicObject Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic.dynamicobject
- Microsoft.CSharp.RuntimeBinder Namespace — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.csharp.runtimebinder
- System.Dynamic Namespace — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `dynamic` | Introduced in **C# 4.0 / .NET Framework 4.0**; stable. |
| DLR | Part of .NET since **.NET Framework 4.0**; open source. |
| `ExpandoObject` | Stable; sealed; implements `IDynamicMetaObjectProvider`. |
| `DynamicObject` | Stable; abstract base class. |
| `RuntimeBinderException` | Thrown on binding failure; catch at boundaries. |
| `dynamic` in Native AOT | **Restricted**; dynamic binding may not be fully supported. |
| `dynamic` performance | **Implementation-dependent**; cached call sites help repeated calls. |
| `dynamic` and trimming | **May break**; dynamic member names are not statically visible. |
| `dynamic` with `async` | Limited; cannot use `dynamic` in some async contexts. |

---

## References (Consolidated)

- Using type dynamic — C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interop/using-type-dynamic
- Built-in types — C# reference (dynamic) — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/built-in-types
- Dynamic Language Runtime Overview — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/docs/main/docs/framework/reflection-and-codedom/dynamic-language-runtime-overview.md
- ExpandoObject Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic.expandoobject
- DynamicObject Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic.dynamicobject
- DynamicObject.TryGetMember — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic.dynamicobject.trygetmember
- DynamicObject.TryInvokeMember — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic.dynamicobject.tryinvokemember
- Microsoft.CSharp.RuntimeBinder Namespace — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.csharp.runtimebinder
- System.Dynamic Namespace — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dynamic
- Walkthrough: Creating and Using Dynamic Objects (C#) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interop/walkthrough-creating-and-using-dynamic-objects

---