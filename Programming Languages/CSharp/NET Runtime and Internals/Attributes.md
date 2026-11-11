# C# Attributes — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An **attribute** in C# is a declarative piece of **metadata** that you attach to a program element—a type, method, property, parameter, assembly, and so on—to convey additional information to the compiler, the runtime, or external tools. Attributes are the primary extension mechanism for annotating code without changing its behaviour or signature.

**Technical Definition**
Attributes are classes derived from `System.Attribute` that are applied to declarations using the bracketed syntax `[AttributeName(...)]`. The attribute information is emitted into the assembly's **metadata** and can be retrieved at runtime via **reflection** (`MemberInfo.GetCustomAttributes`, `Attribute.GetCustomAttribute`) or read at compile time by the compiler, analyzers, and source generators. Attributes can take positional constructor arguments and named property/field arguments. Their behaviour is controlled by the **`[AttributeUsage]`** meta-attribute, which specifies valid targets (`AttributeTargets`), whether multiple applications are allowed (`AllowMultiple`), and whether the attribute is inherited by derived types (`Inherited`). C# also supports **pseudo-attributes** such as `[Obsolete]`, `[Conditional]`, and `[CallerMemberName]` that are recognized specially by the compiler. The C# specification notes that while `[Obsolete]`, `[Conditional]`, `[CallerMemberName]`, and a few others have compiler-special semantics, they are still ordinary attributes at the metadata level.

**Beginner-Friendly Explanation**
An attribute is a **sticky note you attach to code**. The note itself does not change what the code does—it is **information for someone else** to read. That "someone else" might be the compiler (which can warn you, strip out calls, or fill in default argument values), the runtime (which can use the info for reflection), a debugger (which can display a custom string), or your own code (which can inspect the attribute via reflection). Examples: `[Obsolete("Use NewMethod instead")]` tells the compiler to warn callers; `[DebuggerDisplay("{Name} ({Age})")]` tells the debugger how to display a type; `[CallerMemberName]` tells the compiler to insert the caller's name automatically. Attributes make code **self-describing** and enable frameworks (serialization, dependency injection, testing) to discover behaviour declaratively.

---

### Key Characteristics

- **Declarative metadata** — Attributes annotate code; they do not execute on their own.
- **Derived from `System.Attribute`** — Every attribute is a class inheriting (directly or indirectly) from `Attribute`.
- **Bracket syntax** — `[Name]` or `[Name(args)]`, optionally with `Attribute` suffix omitted.
- **Positional + named arguments** — Constructor args are positional; properties/fields can be set by name.
- **`AttributeUsage` governs placement** — `AttributeTargets`, `AllowMultiple`, and `Inherited` determine valid usage.
- **Reflection-readable** — Retrieved at runtime via `GetCustomAttributes`/`GetCustomAttribute`.
- **Compiler-special pseudo-attributes** — `[Obsolete]`, `[Conditional]`, `[CallerMemberName]`, `[CallerFilePath]`, `[CallerLineNumber]` have built-in compiler behaviour.
- **Nullable flow attributes** — `[NotNull]`, `[MaybeNull]`, and friends refine null-state analysis for the compiler.
- **Baked into metadata** — Attributes are stored in the assembly and visible to tools and other assemblies.

---

### Prerequisites

- Basic understanding of C# **classes**, **constructors**, **properties**, and **namespaces**.
- Familiarity with **reflection** (`Type`, `MemberInfo`, `GetCustomAttributes`).
- Awareness of the **compiler pipeline** (attributes are emitted as metadata).
- Knowledge of **nullable reference types** (for `[NotNull]`/`[MaybeNull]`).
- Basic understanding of **preprocessor symbols** (`#define`, `DEBUG`) for `[Conditional]`.

---

### Related Programming Areas

- **Reflection** — The primary way attributes are consumed at runtime.
- **Serialization** — Frameworks like `System.Text.Json` use attributes (`[JsonPropertyName]`) for mapping.
- **Testing** — `[Test]`, `[Fact]`, `[Theory]` drive test discovery.
- **ASP.NET Core** — `[HttpGet]`, `[Authorize]`, `[Route]` route and secure requests.
- **Analyzers and source generators** — Read attributes to generate code or issue diagnostics.
- **Debugging** — `[DebuggerDisplay]`, `[DebuggerStepThrough]` customize debugger behaviour.

---

### Core Concepts / Features

1. Attributes (Overarching Concept)
2. Attribute Declaration
3. Attribute Usage (`[AttributeUsage]`)
4. Custom Attributes
5. Attribute Reflection
6. Built-in Attributes: `[Obsolete]`, `[Serializable]`, `[Conditional]`, `[DebuggerDisplay]`, `[CallerMemberName]`, `[CallerFilePath]`, `[CallerLineNumber]`, `[NotNull]`, `[MaybeNull]`
7. Attribute Best Practices

---

## 1. Attributes (Overarching Concept)

### Definitions

**Core Definition**
An attribute is a declarative annotation applied to a program element to associate metadata with it.

**Technical Definition**
An attribute is a class deriving from `System.Attribute`. It is applied using `[AttributeName]` syntax before a declaration. The attribute's constructor arguments and named property values are stored in the assembly metadata and can be read at runtime via reflection. The compiler treats certain attributes specially (e.g., `[Obsolete]`, `[Conditional]`), but most attributes are pure metadata consumed by frameworks and tools.

**Beginner-Friendly Explanation**
An attribute is a **sticky note of information** attached to code so that tools, the compiler, or the runtime can read it later.

---

### Purposes

- To **annotate code** with declarative metadata.
- To give the **compiler hints** (warnings, conditional calls, caller info).
- To enable **frameworks** (serialization, DI, testing, routing) to discover behaviour.
- To customize **debugger** and **tooling** behaviour.
- To make code **self-documenting** and machine-readable.

---

### Syntax Rules and Structure

```
[AttributeName]
[AttributeName(argument1, argument2)]
[AttributeName(NamedProperty = value)]
[AttributeName(argument, NamedProperty = value)]
```

**Component breakdown**

- `[` and `]` — Attribute brackets.
- `AttributeName` — The attribute class name; the `Attribute` suffix may be omitted.
- `(args)` — Positional constructor arguments.
- `NamedProperty = value` — Named property/field arguments.

#### Syntax Rules

- The `Attribute` suffix is optional: `[Obsolete]` ≡ `[ObsoleteAttribute]`.
- Multiple attributes can be applied in one bracket: `[A, B]`.
- Multiple bracket groups are equivalent: `[A][B]`.
- An attribute without parentheses uses the parameterless constructor.
- Named arguments must refer to public fields or properties of the attribute class.

#### Constraints and Limitations

- Attribute arguments must be **compile-time constants** (literals, `typeof`, `nameof`, enum values, arrays of these).
- Not every attribute is valid on every target (governed by `AttributeUsage`).
- Attributes are **metadata only**; they do not execute code unless a consumer reads them.

---

### Annotated Code Examples and Expected Outputs

#### Example: Multiple Attributes on a Method

```csharp
using System;
using System.Diagnostics;

class Program
{
    [Obsolete("Use NewMethod instead.")]
    static void OldMethod() => Console.WriteLine("Old");

    [Conditional("DEBUG")]
    static void TraceOnly() => Console.WriteLine("Debug trace");

    static void Main()
    {
        OldMethod();      // compiler warning CS0618
        TraceOnly();      // removed in Release builds
    }
}
```

**Expected Output (Debug build)**
```
warning CS0618: 'OldMethod()' is obsolete: 'Use NewMethod instead.'
Old
Debug trace
```

**Expected Output (Release build)**
```
warning CS0618: 'OldMethod()' is obsolete: 'Use NewMethod instead.'
Old
```

**Why this output occurs**
`[Obsolete]` produces a compiler warning; `[Conditional("DEBUG")]` causes the compiler to omit the call in non-`DEBUG` builds.

---

### Real-World Cases

- **Web APIs** — `[HttpGet]`, `[Route]` routing metadata.
- **Serialization** — `[JsonPropertyName]`, `[XmlElement]`.
- **Testing** — `[Fact]`, `[Test]` discovery.
- **Debugging** — `[DebuggerDisplay]` custom display.

---

### References

- Attributes (C#) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/
- Attribute Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.attribute

---

## 2. Attribute Declaration

### Definitions

**Core Definition**
Attribute declaration is the process of defining a new attribute class by deriving from `System.Attribute` and optionally applying `[AttributeUsage]`.

**Technical Definition**
A custom attribute is a class that inherits from `System.Attribute` (directly or indirectly). It can declare constructors (which become positional arguments) and public properties/fields (which become named arguments). By convention, the class name ends with `Attribute`, though the suffix is optional at the use site. The `[AttributeUsage]` attribute controls where the attribute can be applied, whether it can be repeated, and whether it is inherited.

**Beginner-Friendly Explanation**
Declaring an attribute means **creating a new sticky-note type**—a class deriving from `Attribute` with constructor parameters and properties.

---

### Purposes

- To define a **reusable annotation type**.
- To supply **constructor arguments** and **named properties**.
- To control **usage targets** with `[AttributeUsage]`.
- To encode domain-specific metadata for tools or runtime consumers.

---

### Syntax Rules and Structure

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = true, Inherited = false)]
public sealed class MyAttribute : Attribute
{
    public string Name { get; }
    public int Priority { get; set; }

    public MyAttribute(string name) => Name = name;
}
```

**Component breakdown**

- `[AttributeUsage(...)]` — Meta-attribute (optional).
- `: Attribute` — Required base class.
- Constructor — Defines positional arguments.
- Public properties/fields — Define named arguments.

#### Syntax Rules

- The class must derive from `Attribute`.
- Constructors define positional arguments.
- Public fields/properties define named arguments.
- Convention: class name ends with `Attribute`.
- `sealed` is recommended for attribute classes.

#### Constraints and Limitations

- Attribute classes cannot be generic.
- Attribute classes cannot be static.
- Constructor arguments must be compile-time constants.

---

### Annotated Code Examples and Expected Outputs

#### Example: Declaring a Custom Attribute

```csharp
using System;

[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = true)]
public sealed class AuthorAttribute : Attribute
{
    public string Name { get; }
    public int Year { get; set; }

    public AuthorAttribute(string name) => Name = name;
}

[Author("Alice", Year = 2024)]
[Author("Bob")]
public class Document
{
    [Author("Carol", Year = 2023)]
    public void Save() { }
}

class Program
{
    static void Main() => Console.WriteLine("Compiled successfully");
}
```

**Expected Output**
```
Compiled successfully
```

**Why this output occurs**
The custom attribute is declared and applied to a class and a method; compilation succeeds because the targets match `AttributeUsage`.

---

### Real-World Cases

- **Domain metadata** — `[Author]`, `[ReviewedBy]`, `[RequiresPermission]`.
- **Framework integration** — `[ModuleInitializer]`, `[SkipLocalsInit]`.
- **Tooling** — Custom analyzers reading attributes.

---

### References

- Writing custom attributes — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/attribute-tutorial
- AttributeUsageAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.attributeusageattribute

---

## 3. Attribute Usage (`[AttributeUsage]`)

### Definitions

**Core Definition**
`[AttributeUsage]` is a meta-attribute that controls **where** a custom attribute can be applied, whether it can be applied **multiple times**, and whether it is **inherited** by derived types.

**Technical Definition**
`System.AttributeUsageAttribute` has three properties: `ValidOn` (a bitwise `AttributeTargets` enum: `Class`, `Struct`, `Method`, `Property`, `Field`, `Parameter`, `Assembly`, `Module`, `Delegate`, `Enum`, `Event`, `Interface`, `GenericParameter`, `ReturnValue`, `Constructor`, `All`), `AllowMultiple` (whether the attribute may be applied more than once to the same target), and `Inherited` (whether the attribute is inherited by derived types/overridden members). If `[AttributeUsage]` is omitted, the default is `AttributeTargets.All`, `AllowMultiple = false`, `Inherited = true`.

**Beginner-Friendly Explanation**
`[AttributeUsage]` is **the rules for your sticky note**: where it can go, whether you can use it more than once, and whether children inherit it.

---

### Purposes

- To restrict **where** the attribute can be applied.
- To control **multiple applications** on the same target.
- To control **inheritance** of the attribute.
- To catch misuse at **compile time**.

---

### Syntax Rules and Structure

```csharp
[AttributeUsage(
    AttributeTargets.Method | AttributeTargets.Property,
    AllowMultiple = false,
    Inherited = true)]
public sealed class MyAttribute : Attribute { }
```

**Component breakdown**

- `AttributeTargets` — Bit flags for valid targets.
- `AllowMultiple` — `true`/`false` (default `false`).
- `Inherited` — `true`/`false` (default `true`).

#### Syntax Rules

- `ValidOn` accepts a bitwise combination of `AttributeTargets`.
- `AllowMultiple = true` requires a consumer that handles multiple instances.
- `Inherited = false` prevents propagation to derived types.

#### Constraints and Limitations

- Misapplied attributes produce compiler error **CS0592**.
- Multiple applications without `AllowMultiple = true` produce **CS0579**.

---

### Annotated Code Examples and Expected Outputs

#### Example: Restricting and Repeating

```csharp
using System;

[AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
public sealed class TestAttribute : Attribute
{
    public string Name { get; }
    public TestAttribute(string name) => Name = name;
}

public class MyTests
{
    [Test("Add")]
    [Test("Subtract")]
    public void Maths() { }
}

class Program
{
    static void Main() => Console.WriteLine("OK");
}
```

**Expected Output**
```
OK
```

**Why this output occurs**
`AllowMultiple = true` permits multiple `[Test]` annotations; restricting to `Method` prevents application to a class.

---

### Real-World Cases

- **Test frameworks** — Multiple `[Test]` or `[InlineData]`.
- **Serialization** — `[JsonProperty]` typically allows only one instance.
- **Assembly metadata** — `[assembly: AssemblyVersion]`.

---

### References

- AttributeUsageAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.attributeusageattribute
- AttributeTargets — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.attributetargets

---

## 4. Custom Attributes

### Definitions

**Core Definition**
A custom attribute is a user-defined attribute class derived from `Attribute` that provides application-specific metadata.

**Technical Definition**
Custom attributes are classes inheriting from `Attribute`, typically `sealed`, with a name ending in `Attribute`. They can accept constructor arguments (positional) and public properties/fields (named). Consumers read them via reflection (`GetCustomAttribute<T>()`, `GetCustomAttributes(typeof(T), inherit)`). Custom attributes are pervasive in frameworks (DI, serialization, validation, routing).

**Beginner-Friendly Explanation**
A custom attribute is **your own sticky note type**—you design the class, the constructor, and the properties; frameworks or your own code read it later.

---

### Purposes

- To encode **domain-specific metadata**.
- To enable **declarative configuration** for frameworks.
- To drive **code generation** or **analysis** (via source generators/analyzers).
- To make intent **explicit and discoverable**.

---

### Syntax Rules and Structure

```csharp
[AttributeUsage(AttributeTargets.Class)]
public sealed class EntityAttribute : Attribute
{
    public string TableName { get; }
    public EntityAttribute(string tableName) => TableName = tableName;
}

[Entity("customers")]
public class Customer { }
```

**Component breakdown**

- Attribute class — derives from `Attribute`.
- Application — `[Entity("customers")]`.
- Consumer — reflection-based lookup.

#### Syntax Rules

- Constructor args are positional; property initializers are named.
- The `Attribute` suffix may be omitted at the use site.
- Attributes may be combined: `[Entity("customers"), Serializable]`.

#### Constraints and Limitations

- Arguments must be compile-time constants.
- Attributes on generic parameters are restricted to `GenericParameter`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Custom Attribute and Reflection Consumer

```csharp
using System;
using System.Reflection;

[AttributeUsage(AttributeTargets.Class)]
public sealed class EntityAttribute : Attribute
{
    public string TableName { get; }
    public EntityAttribute(string tableName) => TableName = tableName;
}

[Entity("customers")]
public class Customer { }

class Program
{
    static void Main()
    {
        Type t = typeof(Customer);
        var attr = t.GetCustomAttribute<EntityAttribute>();
        Console.WriteLine(attr?.TableName);   // customers
    }
}
```

**Expected Output**
```
customers
```

**Why this output occurs**
`GetCustomAttribute<EntityAttribute>()` retrieves the applied attribute and its constructor argument.

---

### Real-World Cases

- **ORMs** — `[Table]`, `[Column]` mapping.
- **Validation** — `[Required]`, `[Range]`.
- **DI** — `[Inject]`, `[Service]` markers.

---

### References

- Writing custom attributes — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/attribute-tutorial

---

## 5. Attribute Reflection

### Definitions

**Core Definition**
Attribute reflection is the process of **reading attributes at runtime** from assemblies, types, and members using reflection APIs.

**Technical Definition**
Attribute reflection uses `MemberInfo.GetCustomAttributes()` (returns `object[]` or `IEnumerable<Attribute>`), `MemberInfo.GetCustomAttribute<T>()` (returns `T?` or `Attribute?`), and `Attribute.IsDefined(member, typeof(T))` (returns `bool`). `inherit: true` follows the inheritance chain. `Attribute.GetCustomAttribute(Assembly, Type)` reads assembly-level attributes. Attributes are instantiated lazily on first reflection call and cached by the runtime.

**Beginner-Friendly Explanation**
Attribute reflection is **asking an object or type “what sticky notes do you have?”** at runtime.

---

### Purposes

- To **discover** custom attributes at runtime.
- To **configure frameworks** declaratively.
- To **validate** or **transform** based on metadata.
- To **generate** runtime behaviour from declarative annotations.

---

### Syntax Rules and Structure

```csharp
T? attr = member.GetCustomAttribute<T>(inherit: true);
bool defined = member.IsDefined(typeof(T), inherit: false);
IEnumerable<T> all = member.GetCustomAttributes<T>(inherit: true);
```

**Component breakdown**

- `GetCustomAttribute<T>()` — Single attribute or `null`.
- `GetCustomAttributes<T>()` — All attributes of type `T`.
- `IsDefined(...)` — Boolean check.

#### Syntax Rules

- `inherit: true` traverses base types (if `Inherited = true` on the attribute).
- `GetCustomAttribute` returns `null` if absent (or `Attribute?` in older APIs).
- Assembly attributes use `Assembly.GetCustomAttribute<T>()`.

#### Constraints and Limitations

- Reflection is slow; cache results.
- Attributes are instantiated on demand; constructors should be side-effect-free.

---

### Annotated Code Examples and Expected Outputs

#### Example: Enumerating and Filtering Attributes

```csharp
using System;
using System.Linq;
using System.Reflection;

[AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
public sealed class TagAttribute : Attribute
{
    public string Value { get; }
    public TagAttribute(string value) => Value = value;
}

public class Service
{
    [Tag("api"), Tag("v1")]
    public void Get() { }
}

class Program
{
    static void Main()
    {
        MethodInfo m = typeof(Service).GetMethod("Get")!;
        var tags = m.GetCustomAttributes<TagAttribute>().Select(a => a.Value);
        Console.WriteLine(string.Join(", ", tags));   // api, v1
    }
}
```

**Expected Output**
```
api, v1
```

**Why this output occurs**
`GetCustomAttributes<TagAttribute>()` returns both applied instances; LINQ projects their values.

---

### Real-World Cases

- **Routing** — ASP.NET Core reads `[HttpGet]` at request time.
- **Validation** — Frameworks apply `[Required]` rules.
- **Serialization** — Mapping properties to JSON names.

---

### References

- Accessing attributes by using reflection (C#) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/accessing-attributes-by-using-reflection
- MemberInfo.GetCustomAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.memberinfo.getcustomattribute

---

## 6. Built-in Attributes

### 6.1 `[Obsolete]`

**Definitions**

- **Core:** Marks a program element as obsolete, producing a compiler warning or error.
- **Technical:** `System.ObsoleteAttribute` has a `Message` and an `IsError` property. When `IsError = true`, usage is a compile error; otherwise it is a warning.
- **Beginner:** Tells callers “don’t use this anymore.”

**Purposes**

- To warn callers about deprecated APIs.
- To fail the build on forbidden usage (`IsError = true`).
- To guide migration.

**Syntax Rules**

```csharp
[Obsolete]
[Obsolete("Use NewMethod instead.")]
[Obsolete("Use NewMethod instead.", error: true)]
```

**Constraints:** Warnings are suppressible (`#pragma warning disable CS0618`); errors are not suppressible without removing the call.

**Annotated Example**

```csharp
using System;

class Program
{
    [Obsolete("Use NewMethod instead.", false)]
    static void OldMethod() => Console.WriteLine("Old");

    [Obsolete("This API is removed.", true)]
    static void RemovedMethod() => Console.WriteLine("Removed");

    static void Main()
    {
        OldMethod();   // warning CS0618
        // RemovedMethod(); // error CS0619
    }
}
```

**Expected Output**
```
warning CS0618: 'OldMethod()' is obsolete: 'Use NewMethod instead.'
Old
```

**Real-World Cases:** Deprecating APIs across major versions; guiding users to replacements.

**References:** ObsoleteAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.obsoleteattribute

---

### 6.2 `[Serializable]`

**Definitions**

- **Core:** Marks a type as serializable by binary formatters.
- **Technical:** `System.SerializableAttribute` indicates the type can be serialized by `BinaryFormatter` (legacy), `DataContractSerializer`, and similar. In .NET 8+, `[Serializable]` remains metadata, but **binary serialization (`BinaryFormatter`) is obsolete and unsafe** (`SYSLIB0011`, `SYSLIB0050`).
- **Beginner:** Tells legacy serializers “this type can be serialized.”

**Purposes**

- To indicate binary-serializable types (legacy).
- To participate in serialization frameworks that check the attribute.

**Syntax Rules**

```csharp
[Serializable]
public class MyData { }
```

**Constraints:** `BinaryFormatter` is unsafe and should be avoided; prefer `System.Text.Json` or `DataContractSerializer`.

**Annotated Example**

```csharp
using System;
using System.Text.Json;

[Serializable]
public class Person { public string Name { get; set; } = "Alice"; }

class Program
{
    static void Main()
    {
        string json = JsonSerializer.Serialize(new Person());
        Console.WriteLine(json);   // {"Name":"Alice"}
    }
}
```

**Expected Output**
```
{"Name":"Alice"}
```

**Real-World Cases:** Legacy .NET Framework code; modern code should use JSON/XML serialization.

**References:** SerializableAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.serializableattribute

---

### 6.3 `[Conditional]`

**Definitions**

- **Core:** `[Conditional("SYMBOL")]` on a method means calls are included **only if** `SYMBOL` is defined at the call site.
- **Technical:** `System.Diagnostics.ConditionalAttribute` accepts one or more preprocessor symbols. When the symbol is not defined, calls to the method are **removed by the compiler**. It applies to `void` methods returning `void` (and attribute classes), so the compiler can safely drop calls.
- **Beginner:** “Only include this call if a symbol is defined.”

**Purposes**

- To include diagnostic or tracing code only in certain builds.
- To avoid `#if` clutter around calls.
- To conditionally add attributes to types.

**Syntax Rules**

```csharp
[Conditional("DEBUG")]
void Trace() { }

[Conditional("DEBUG"), Conditional("TRACE")]
void TraceBoth() { }
```

**Constraints:** Only methods returning `void`; conditional attributes must be applied before use; the symbol is checked **at the call site**.

**Annotated Example**

```csharp
using System;
using System.Diagnostics;

class Program
{
    [Conditional("DEBUG")]
    static void DebugLog(string message) => Console.WriteLine($"DEBUG: {message}");

    static void Main()
    {
        DebugLog("Started");
        Console.WriteLine("Done");
    }
}
```

**Expected Output (Debug build)**
```
DEBUG: Started
Done
```

**Expected Output (Release build)**
```
Done
```

**Why this output occurs:** In Release builds, `DEBUG` is undefined at the call site; the compiler omits `DebugLog("Started")`.

**Real-World Cases:** `Debug.Assert`, `Debug.WriteLine`, and conditional tracing.

**References:** ConditionalAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.conditionalattribute

---

### 6.4 `[DebuggerDisplay]`

**Definitions**

- **Core:** `[DebuggerDisplay("...")]` customizes how a type is displayed in the debugger.
- **Technical:** `System.Diagnostics.DebuggerDisplayAttribute` takes a format string with `{expression}` fields (e.g., `{Name}`) and prefixes `nq` (no quotes) or `raw` (raw expression). Properties used must be readable by the debugger (fields, properties).
- **Beginner:** “Show me this string in the debugger instead of the type name.”

**Purposes**

- To display meaningful summaries in debugger tooltips.
- To avoid expanding the whole object.
- To highlight key properties.

**Syntax Rules**

```csharp
[DebuggerDisplay("{Name} ({Age})")]
public class Person { public string Name { get; set; } = "Alice"; public int Age { get; set; } = 30; }
```

**Constraints:** Expressions are evaluated by the debugger; side effects or unsupported members may show errors.

**Annotated Example**

```csharp
using System;
using System.Diagnostics;

[DebuggerDisplay("Person: {Name}, Age {Age}")]
public class Person
{
    public string Name { get; set; } = "Alice";
    public int Age { get; set; } = 30;
}

class Program
{
    static void Main()
    {
        var p = new Person();
        Console.WriteLine(p.Name);   // Alice
    }
}
```

**Expected Output (debugger tooltip)**
```
Person: Alice, Age 30
```

**Real-World Cases:** Domain objects, DTOs, entity display in debugger.

**References:** DebuggerDisplayAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.debuggerdisplayattribute

---

### 6.5 `[CallerMemberName]`

**Definitions**

- **Core:** A parameter attribute that causes the compiler to insert the **caller’s member name** as the argument.
- **Technical:** `System.Runtime.CompilerServices.CallerMemberNameAttribute` is applied to an optional parameter; the compiler inserts the name of the calling method, property, or event. Requires C# 5+.
- **Beginner:** “Tell me the name of whoever called me, automatically.”

**Purposes**

- To implement `INotifyPropertyChanged` without magic strings.
- To add automatic context to logging.
- To avoid hard-coded caller names.

**Syntax Rules**

```csharp
void Log(string message, [CallerMemberName] string? member = null) { }
```

**Constraints:** Only works for optional parameters; only valid on methods (and their overloads); the argument is supplied by the compiler, not the caller.

**Annotated Example**

```csharp
using System;
using System.Runtime.CompilerServices;

class Program
{
    static void Main() => Foo();

    static void Foo([CallerMemberName] string caller = "")
    {
        Console.WriteLine($"Called from: {caller}");   // Called from: Main
    }
}
```

**Expected Output**
```
Called from: Main
```

**Real-World Cases:** `INotifyPropertyChanged`, logging helpers, audit trails.

**References:** CallerMemberNameAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.callermembernameattribute

---

### 6.6 `[CallerFilePath]`

**Definitions**

- **Core:** A parameter attribute that causes the compiler to insert the **source file path** of the caller.
- **Technical:** `System.Runtime.CompilerServices.CallerFilePathAttribute`; the compiler inserts the full path of the source file containing the call.
- **Beginner:** “Tell me which source file called me.”

**Purposes**

- To log the source file for diagnostics.
- To identify caller location without hard-coding.
- To support build/trace tooling.

**Syntax Rules**

```csharp
void Log(string msg, [CallerFilePath] string path = "") { }
```

**Constraints:** Path is compile-time, usually full; different environments may produce different paths; affects reproducibility (use `DeterministicSourcePaths`).

**Annotated Example**

```csharp
using System;
using System.Runtime.CompilerServices;

class Program
{
    static void Main() => Log("Test");

    static void Log(string msg, [CallerFilePath] string path = "")
    {
        Console.WriteLine($"File: {path}");
    }
}
```

**Expected Output (example)**
```
File: C:\src\Program.cs
```

**Real-World Cases:** Logging with file context; assertion messages.

**References:** CallerFilePathAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.callerfilepathattribute

---

### 6.7 `[CallerLineNumber]`

**Definitions**

- **Core:** A parameter attribute that causes the compiler to insert the **line number** of the call.
- **Technical:** `System.Runtime.CompilerServices.CallerLineNumberAttribute`; the compiler inserts the line number as an `int`.
- **Beginner:** “Tell me the line number where I was called.”

**Purposes**

- To log precise source locations.
- To improve diagnostics and assertions.
- To avoid manual line tracking.

**Syntax Rules**

```csharp
void Log(string msg, [CallerLineNumber] int line = 0) { }
```

**Constraints:** Line numbers shift with edits; not stable across versions.

**Annotated Example**

```csharp
using System;
using System.Runtime.CompilerServices;

class Program
{
    static void Main() => Log("Test");

    static void Log(string msg, [CallerLineNumber] int line = 0)
    {
        Console.WriteLine($"Line: {line}");
    }
}
```

**Expected Output**
```
Line: 7
```

**Real-World Cases:** Assertion messages, trace logs, diagnostics.

**References:** CallerLineNumberAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.callerlinenumberattribute

---

### 6.8 `[NotNull]`

**Definitions**

- **Core:** A nullable analysis attribute that tells the compiler a value is **not null** when the annotated member returns (or the parameter is used).
- **Technical:** `System.Diagnostics.CodeAnalysis.NotNullAttribute` is applied to return values, `out`/`ref` parameters, fields, or properties; it refines the compiler’s **null-state analysis** without changing runtime behaviour. It is commonly used in post-condition methods (`[NotNull]` on `out` after a method validates it) and in fluent APIs.
- **Beginner:** “Trust me, compiler—this value is not null even though the signature doesn’t prove it.”

**Purposes**

- To refine nullability for the compiler.
- To reduce false warnings in APIs with post-conditions.
- To document non-null guarantees in signatures.

**Syntax Rules**

```csharp
[return: NotNull]
public string? GetName() => _name; // compiler trusts non-null

public void Ensure([NotNull] string? value) { ... } // post-condition
```

**Constraints:** Only affects compile-time analysis; runtime behaviour unchanged; useful only with nullable annotations enabled.

**Annotated Example**

```csharp
#nullable enable
using System;
using System.Diagnostics.CodeAnalysis;

class Program
{
    static void Ensure([NotNull] string? value)
    {
        if (value is null) throw new ArgumentNullException(nameof(value));
    }

    static void Main()
    {
        string? maybe = "hello";
        Ensure(maybe);
        Console.WriteLine(maybe.Length);   // no warning: 'maybe' is not null
    }
}
```

**Expected Output**
```
5
```

**Real-World Cases:** APIs with `TryGet` patterns; fluent builders; interop.

**References:** NotNullAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.codeanalysis.notnullattribute

---

### 6.9 `[MaybeNull]`

**Definitions**

- **Core:** A nullable analysis attribute that tells the compiler a value **might be null** even when the type suggests otherwise.
- **Technical:** `System.Diagnostics.CodeAnalysis.MaybeNullAttribute` refines null-state analysis on return values, `out`/`ref` parameters, fields, and properties. It is the counterpart to `[NotNull]`.
- **Beginner:** “Don’t trust the type—this could be null.”

**Purposes**

- To correct overly optimistic nullability in APIs.
- To model “default might be null” for generic types.
- To reduce or produce warnings accurately.

**Syntax Rules**

```csharp
[return: MaybeNull]
public T Find<T>(...) { ... }   // T may be null (default(T))
```

**Constraints:** Only affects compile-time analysis; must use with nullable context enabled.

**Annotated Example**

```csharp
#nullable enable
using System;
using System.Diagnostics.CodeAnalysis;

class Program
{
    [return: MaybeNull]
    static T FindOrDefault<T>(T[] items, int index) =>
        index >= 0 && index < items.Length ? items[index] : default;

    static void Main()
    {
        string[] names = { "Alice" };
        string name = FindOrDefault(names, 5)!;
        Console.WriteLine(name ?? "(none)");   // (none)
    }
}
```

**Expected Output**
```
(none)
```

**Real-World Cases:** Generic lookup APIs; `FirstOrDefault`-style patterns.

**References:** MaybeNullAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.codeanalysis.maybenullattribute

---

## 7. Attribute Best Practices

### Guidelines

#### General Guidelines

```
// 1. Name attributes with the Attribute suffix
public sealed class AuthorAttribute : Attribute { }

// 2. Always specify AttributeUsage
[AttributeUsage(AttributeTargets.Class, AllowMultiple = false, Inherited = false)]
public sealed class EntityAttribute : Attribute { }

// 3. Make attribute constructors side-effect-free
public EntityAttribute(string table) => TableName = table;

// 4. Cache reflection results
private static readonly EntityAttribute? _entity = typeof(Customer).GetCustomAttribute<EntityAttribute>();

// 5. Prefer source generators for hot paths / AOT
// [JsonSerializable] and friends

// 6. Use caller-info attributes instead of magic strings
void Log(string msg, [CallerMemberName] string? member = null) { }

// 7. Use nullable flow attributes to refine analysis
static void Ensure([NotNull] string? value) { ... }

// 8. Avoid [Serializable] with BinaryFormatter (obsolete)
```

**Component breakdown**

- **`Attribute` suffix** — Readability and convention.
- **Explicit `AttributeUsage`** — Prevents misuse.
- **Side-effect-free constructors** — Attributes may be instantiated by reflection.
- **Caching** — Reflection is costly.
- **Source generators** — AOT/trimming-friendly.
- **Caller-info** — Avoids magic strings.
- **Nullable flow attributes** — Clear null contracts.
- **Avoid legacy binary serialization** — `[Serializable]` is metadata only; `BinaryFormatter` is unsafe.

#### Rules and Constraints

- **Do** name custom attributes with the `Attribute` suffix.
- **Do** apply `[AttributeUsage]` explicitly.
- **Do** keep attribute constructors side-effect-free.
- **Do** cache reflection lookups.
- **Do** use caller-info attributes instead of string literals.
- **Do not** rely on attribute constructors for runtime work.
- **Do not** use `[Serializable]` with `BinaryFormatter`.
- **Do not** apply attributes where they are not valid.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Attribute

```csharp
using System;
using System.Reflection;

[AttributeUsage(AttributeTargets.Class, AllowMultiple = false, Inherited = false)]
public sealed class EntityAttribute : Attribute
{
    public string TableName { get; }
    public EntityAttribute(string tableName) => TableName = tableName;
}

[Entity("customers")]
public class Customer { }

class Program
{
    private static readonly EntityAttribute? _entity =
        typeof(Customer).GetCustomAttribute<EntityAttribute>();

    static void Main() => Console.WriteLine(_entity?.TableName);   // customers
}
```

**Expected Output**
```
customers
```

**Why this output occurs**
The custom attribute follows the `Attribute` suffix, `[AttributeUsage]` restricts it to classes, and reflection is cached in a static field.

---

### Real-World Cases

- **API design** — `[Obsolete]` and `[EditorBrowsable]` guide consumers.
- **Framework metadata** — `[JsonPropertyName]`, `[Table]`, `[Route]`.
- **Diagnostics** — Caller-info attributes improve logs.
- **Nullable analysis** — Flow attributes refine warnings.

---

### References

- Attributes (C#) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/
- Writing custom attributes — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/attribute-tutorial
- Accessing attributes by using reflection — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/accessing-attributes-by-using-reflection
- AttributeUsageAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.attributeusageattribute
- Nullable reference types — attributes — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/attributes/nullable-analysis

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `[Obsolete]` | Stable; `IsError = true` makes usage a compile error. |
| `[Serializable]` | **Metadata only in .NET 8+**; `BinaryFormatter` is obsolete and unsafe (`SYSLIB0011`, `SYSLIB0050`). |
| `[Conditional]` | Stable; only valid on `void` methods and attribute classes. |
| `[DebuggerDisplay]` | Stable; expression syntax supports `nq` and `raw` prefixes. |
| `[CallerMemberName]` / `[CallerFilePath]` / `[CallerLineNumber]` | Introduced in **C# 5.0**; stable. |
| `[NotNull]` / `[MaybeNull]` | Introduced in **C# 8.0** nullable analysis; stable. |
| `[NotNullWhen]`, `[MaybeNullWhen]`, `[DoesNotReturn]`, `[DoesNotReturnIf]`, `[MemberNotNull]` | Additional nullable flow attributes in `System.Diagnostics.CodeAnalysis`. |
| Reflection on attributes | **Trim/AOT-sensitive**; use `DynamicDependency` or source generators. |
| `[Serializable]` and trimming | Not a substitute for modern serialization attributes. |

---

## References (Consolidated)

- Attributes (C#) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/
- Writing custom attributes — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/attribute-tutorial
- Accessing attributes by using reflection — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/accessing-attributes-by-using-reflection
- Attribute Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.attribute
- AttributeUsageAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.attributeusageattribute
- AttributeTargets — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.attributetargets
- ObsoleteAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.obsoleteattribute
- SerializableAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.serializableattribute
- ConditionalAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.conditionalattribute
- DebuggerDisplayAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.debuggerdisplayattribute
- CallerMemberNameAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.callermembernameattribute
- CallerFilePathAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.callerfilepathattribute
- CallerLineNumberAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.callerlinenumberattribute
- NotNullAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.codeanalysis.notnullattribute
- MaybeNullAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.codeanalysis.maybenullattribute
- Nullable analysis attributes — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/attributes/nullable-analysis

---