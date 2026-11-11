# C# Reflection — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**Reflection** is a .NET runtime feature that allows code to **inspect and manipulate assemblies, types, members, and metadata at runtime**. It enables dynamic discovery of types, invocation of methods, access to properties and fields, and creation of objects—without knowing those types at compile time. Reflection is the foundation of many .NET frameworks (dependency injection, serialization, ORMs, testing frameworks, and more).

**Technical Definition**
Reflection is the process by which a program can inspect its own metadata and use it to interact with types, members, and assemblies. It is exposed via the `System.Reflection` namespace, with the core type `System.Type` representing type metadata. Using `Type`, you can obtain `MethodInfo`, `PropertyInfo`, `FieldInfo`, `ConstructorInfo`, and other member descriptors, and invoke them dynamically. Reflection supports **late binding**—resolving types and members at runtime—and can be used to create instances (`Activator.CreateInstance`), invoke methods (`MethodInfo.Invoke`), read/write properties/fields, inspect attributes, and load assemblies. Reflection performance is significantly slower than direct calls (due to metadata lookups, boxing, and dynamic dispatch), but modern .NET has optimizations (caching, compiled expressions, `Expression<T>`, `MethodInfo.CreateDelegate`, and `System.Reflection.Metadata` for metadata-only reading). **`System.Reflection.Metadata`** provides a low-level, allocation-efficient API for reading metadata without loading assemblies into the runtime, useful for analyzers and source generators.

**Beginner-Friendly Explanation**
Reflection is **"code that looks at other code."** Normally, your program knows the types it uses at compile time. Reflection lets your program ask at runtime: "What types exist in this assembly? What methods does this class have? What properties does it expose?" You can then create objects, call methods, and read/write properties dynamically—even if you didn't know the type when you wrote the code. Frameworks use reflection heavily: dependency injection finds constructors to instantiate services, serializers inspect properties to convert objects to JSON, and test frameworks find methods marked `[Test]`. Reflection is powerful but slower than direct calls, so best practices focus on caching and avoiding reflection in hot paths.

---

### Key Characteristics

- **Runtime inspection** — Examine assemblies, types, members, attributes, and metadata at runtime.
- **Late binding** — Resolve and use types and members dynamically.
- **Dynamic invocation** — Call methods (`MethodInfo.Invoke`) and constructors (`Activator.CreateInstance`).
- **Member access** — Read/write properties and fields; inspect events and parameters.
- **Attribute inspection** — Read custom attributes applied to types and members.
- **Performance overhead** — Reflection is significantly slower than direct calls; cache results.
- **`System.Reflection.Metadata`** — Low-level, allocation-free metadata reading (for analyzers/generators).
- **AOT limitations** — Reflection is restricted in Native AOT and trimming scenarios.

---

### Prerequisites

- Basic understanding of **assemblies**, **types**, and **metadata**.
- Familiarity with **generics** and **attributes**.
- Knowledge of `System.Type` and `typeof`.
- Awareness of **late binding** and `Activator`.
- Understanding of **performance** (caching, avoiding hot-path reflection).

---

### Related Programming Areas

- **Dependency injection** — Uses reflection to instantiate services.
- **Serialization** — Inspects properties/fields to convert objects.
- **ORM frameworks** — Map classes to database tables.
- **Test frameworks** — Discover and invoke test methods.
- **Source generators** — Use `System.Reflection.Metadata` for compile-time analysis.
- **AOT compatibility** — Trimming restricts reflection.

---

### Core Concepts / Features

1. Reflection (Overarching Concept)
2. `Type`
3. `Assembly`
4. `MethodInfo`
5. `PropertyInfo`
6. `FieldInfo`
7. Reflection Performance
8. Reflection Best Practices
9. `System.Reflection.Metadata`

---

## 1. Reflection (Overarching Concept)

### Definitions

**Core Definition**
Reflection is the runtime ability to inspect and manipulate assemblies, types, and members.

**Technical Definition**
Reflection provides objects (`Type`, `Assembly`, `MethodInfo`, etc.) that describe assemblies, modules, and types. It allows you to dynamically create instances of types, bind types to existing objects, get the type of an existing object, and invoke methods or access fields and properties. Reflection is the foundation of late binding in .NET.

**Beginner-Friendly Explanation**
Reflection is **code that examines code at runtime**. It lets you discover types and members dynamically.

---

### Purposes

- To **inspect metadata** at runtime.
- To **create objects** dynamically (`Activator.CreateInstance`).
- To **invoke methods** and **access members** dynamically.
- To **read attributes** and configuration.
- To enable **frameworks** (DI, serialization, ORMs, testing).

---

### Syntax Rules and Structure (Reflection Flow)

```
Type → GetMethods/GetProperties/GetFields → invoke/read/write → use results
```

**Component breakdown**

- **`Type`** — Entry point for type metadata.
- **`GetMethods()`/`GetProperties()`/`GetFields()`** — Member discovery.
- **`Invoke()`** — Dynamic invocation.
- **`SetValue()`/`GetValue()`** — Property/field access.

#### Syntax Rules

- Reflection APIs are in `System.Reflection`.
- `typeof(T)` or `obj.GetType()` returns a `Type`.
- `Assembly.GetExecutingAssembly()` returns the current assembly.

#### Constraints and Limitations

- Slower than direct calls.
- Limited in Native AOT / trimming scenarios.
- Security-sensitive code should validate inputs.

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic Reflection

```csharp
using System;
using System.Reflection;

public class Person
{
    public string Name { get; set; } = "Alice";
    public void Greet() => Console.WriteLine("Hello");
}

class Program
{
    static void Main()
    {
        Type type = typeof(Person);
        Console.WriteLine($"Type: {type.Name}");

        foreach (var prop in type.GetProperties())
            Console.WriteLine($"Property: {prop.Name}");

        foreach (var method in type.GetMethods(BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly))
            Console.WriteLine($"Method: {method.Name}");
    }
}
```

**Expected Output**
```
Type: Person
Property: Name
Method: get_Name
Method: set_Name
Method: Greet
```

**Why this output occurs**
Reflection exposes the metadata of `Person`, including properties and methods.

---

### Real-World Cases

- **DI containers** — Instantiate services via constructors found by reflection.
- **JSON serializers** — Inspect properties to serialize/deserialize.
- **Test runners** — Find methods with `[Test]` and invoke them.

---

### References

- Reflection (C#) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/reflection
- System.Reflection — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection

---

## 2. `Type`

### Definitions

**Core Definition**
`Type` is the reflection class that represents **type metadata**—the runtime description of a class, struct, interface, enum, delegate, or array.

**Technical Definition**
`System.Type` is the entry point for reflection. It exposes members: `Name`, `FullName`, `Namespace`, `BaseType`, `IsClass`, `IsValueType`, `IsInterface`, `GetMethods()`, `GetProperties()`, `GetFields()`, `GetConstructors()`, `GetAttributes()`, `GetGenericArguments()`, and more. Obtain a `Type` via `typeof(T)`, `obj.GetType()`, or `Type.GetType(string)`. It supports `BindingFlags` to control visibility and static/instance member selection.

**Beginner-Friendly Explanation**
`Type` is the **"description of a type"**. Use it to discover methods, properties, and other members.

---

### Purposes

- To **inspect type metadata**.
- To **discover members** (methods, properties, fields, constructors).
- To **create instances** (`Activator.CreateInstance`).
- To **inspect attributes** and generic arguments.

---

### Syntax Rules and Structure

```csharp
Type t1 = typeof(Person);
Type t2 = obj.GetType();
Type? t3 = Type.GetType("Namespace.Person, AssemblyName");
```

**Component breakdown**

- `typeof(T)` — Compile-time type.
- `obj.GetType()` — Runtime type.
- `Type.GetType(name)` — Resolve by name (late binding).

#### Syntax Rules

- `Type` is abstract; instances are runtime-specific (`RuntimeType`).
- `BindingFlags` control visibility (public/non-public, static/instance).
- `GetMethods()` without flags returns public instance/static methods.

#### Constraints and Limitations

- `Type.GetType(string)` requires the assembly-qualified name for non-corlib types.
- Some members are inherited; use `DeclaredOnly` to limit.

---

### Annotated Code Examples and Expected Outputs

#### Example: Discovering Members

```csharp
using System;
using System.Reflection;

public class Person
{
    public string Name { get; set; } = "Alice";
    private int _age = 30;
    public void Greet() => Console.WriteLine("Hello");
}

class Program
{
    static void Main()
    {
        Type t = typeof(Person);
        Console.WriteLine($"Base: {t.BaseType?.Name}");   // Object

        foreach (var m in t.GetMethods(BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly))
            Console.WriteLine($"Method: {m.Name}");
    }
}
```

**Expected Output**
```
Base: Object
Method: get_Name
Method: set_Name
Method: Greet
```

**Why this output occurs**
`GetMethods` with `DeclaredOnly` returns only methods declared on `Person`.

---

### Real-World Cases

- **Serialization** — Discover properties to serialize.
- **DI** — Find constructors to instantiate.
- **Testing** — Find test methods.

---

### References

- Type Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.type

---

## 3. `Assembly`

### Definitions

**Core Definition**
`Assembly` is the reflection class representing a **loaded assembly**—the unit of deployment containing types and resources.

**Technical Definition**
`System.Reflection.Assembly` represents an assembly. It exposes `GetName()`, `Location`, `GetTypes()`, `GetExportedTypes()`, `GetType(string)`, `GetManifestResourceStream()`, `GetCustomAttributes()`, and loading APIs (`Load`, `LoadFrom`, `LoadFile`). It is the entry point for discovering types within an assembly.

**Beginner-Friendly Explanation**
`Assembly` represents a **loaded `.dll`/`.exe`**. Use it to discover types.

---

### Purposes

- To **load assemblies** at runtime.
- To **discover types** in an assembly.
- To **read assembly metadata** (name, version, location).
- To **read embedded resources**.

---

### Syntax Rules and Structure

```csharp
Assembly asm = Assembly.GetExecutingAssembly();
Type[] types = asm.GetTypes();
Type? t = asm.GetType("Namespace.Person");
```

**Component breakdown**

- `GetExecutingAssembly()` — Current assembly.
- `GetTypes()` — All types (may throw `ReflectionTypeLoadException`).
- `GetType(name)` — Specific type.

#### Syntax Rules

- `GetTypes()` returns all types, including non-public.
- `GetExportedTypes()` returns only public types.
- `GetType(name)` returns `null` if not found.

#### Constraints and Limitations

- `GetTypes()` can throw `ReflectionTypeLoadException` if some types fail to load.
- Loading assemblies dynamically requires care (security, versioning).

---

### Annotated Code Examples and Expected Outputs

#### Example: Enumerating Types

```csharp
using System;
using System.Reflection;

class Program
{
    static void Main()
    {
        Assembly asm = Assembly.GetExecutingAssembly();
        Console.WriteLine($"Assembly: {asm.GetName().Name}");

        foreach (var t in asm.GetExportedTypes())
            Console.WriteLine(t.FullName);
    }
}
```

**Expected Output (example)**
```
Assembly: MyApp
Program
```

**Why this output occurs**
`GetExportedTypes` returns public types in the executing assembly.

---

### Real-World Cases

- **Plugins** — Load assemblies and discover plugin types.
- **Serialization** — Discover types for registration.
- **Diagnostics** — Inspect assembly metadata.

---

### References

- Assembly Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.assembly

---

## 4. `MethodInfo`

### Definitions

**Core Definition**
`MethodInfo` is the reflection class representing a **method** (or constructor) of a type, enabling inspection and dynamic invocation.

**Technical Definition**
`System.Reflection.MethodInfo` derives from `MethodBase` and exposes `Name`, `ReturnType`, `GetParameters()`, `GetCustomAttributes()`, `IsStatic`, `IsPublic`, `Invoke(object? obj, object?[]? parameters)`, and `CreateDelegate()`. It is obtained via `Type.GetMethod(name)` or `Type.GetMethods()`. `Invoke` performs dynamic invocation; `CreateDelegate` creates a strongly typed delegate for faster repeated calls.

**Beginner-Friendly Explanation**
`MethodInfo` describes a **method**—its name, parameters, return type—and lets you call it dynamically.

---

### Purposes

- To **inspect method metadata**.
- To **invoke methods dynamically**.
- To **create delegates** for repeated fast calls.
- To **read method attributes**.

---

### Syntax Rules and Structure

```csharp
MethodInfo? method = type.GetMethod("Greet");
object? result = method?.Invoke(instance, null);
Delegate del = method!.CreateDelegate(typeof(Action));
```

**Component breakdown**

- `GetMethod(name)` — Find method.
- `Invoke(obj, args)` — Call method.
- `CreateDelegate()` — Strongly typed delegate.

#### Syntax Rules

- `Invoke` requires a matching instance (or `null` for static methods) and argument array.
- `CreateDelegate` is faster for repeated calls.
- Exceptions from `Invoke` are wrapped in `TargetInvocationException`.

#### Constraints and Limitations

- `Invoke` is slower than direct calls.
- Parameter binding is fragile (type mismatches throw).

---

### Annotated Code Examples and Expected Outputs

#### Example: Invoking a Method

```csharp
using System;
using System.Reflection;

public class Greeter
{
    public string Greet(string name) => $"Hello, {name}";
}

class Program
{
    static void Main()
    {
        var greeter = new Greeter();
        MethodInfo? method = typeof(Greeter).GetMethod("Greet");
        object? result = method?.Invoke(greeter, new object[] { "Alice" });
        Console.WriteLine(result);   // Hello, Alice
    }
}
```

**Expected Output**
```
Hello, Alice
```

**Why this output occurs**
`Invoke` dynamically calls `Greet` on the `greeter` instance with the argument `"Alice"`.

---

### Real-World Cases

- **Serialization** — Invoke getters/setters.
- **Testing** — Invoke test methods.
- **Dynamic proxies** — Intercept method calls.

---

### References

- MethodInfo Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.methodinfo

---

## 5. `PropertyInfo`

### Definitions

**Core Definition**
`PropertyInfo` is the reflection class representing a **property** of a type, enabling inspection and dynamic read/write.

**Technical Definition**
`System.Reflection.PropertyInfo` exposes `Name`, `PropertyType`, `CanRead`, `CanWrite`, `GetValue(object? obj)`, `SetValue(object? obj, object? value)`, `GetGetMethod()`, `GetSetMethod()`, and `GetIndexParameters()`. It is obtained via `Type.GetProperty(name)` or `Type.GetProperties()`.

**Beginner-Friendly Explanation**
`PropertyInfo` describes a **property** and lets you get/set its value dynamically.

---

### Purposes

- To **inspect property metadata**.
- To **dynamically read/write property values**.
- To **discover property attributes**.
- To **support serialization and data binding**.

---

### Syntax Rules and Structure

```csharp
PropertyInfo? prop = type.GetProperty("Name");
object? value = prop?.GetValue(instance);
prop?.SetValue(instance, "Bob");
```

**Component breakdown**

- `GetProperty(name)` — Find property.
- `GetValue(obj)` — Read.
- `SetValue(obj, value)` — Write.

#### Syntax Rules

- `GetValue`/`SetValue` require a matching instance (or `null` for static).
- Indexers require index arguments.

#### Constraints and Limitations

- Slower than direct property access.
- Read-only/write-only properties throw on unsupported operations.

---

### Annotated Code Examples and Expected Outputs

#### Example: Reading and Writing Properties

```csharp
using System;
using System.Reflection;

public class Person
{
    public string Name { get; set; } = "Alice";
}

class Program
{
    static void Main()
    {
        var p = new Person();
        PropertyInfo? prop = typeof(Person).GetProperty("Name");
        Console.WriteLine(prop?.GetValue(p));   // Alice
        prop?.SetValue(p, "Bob");
        Console.WriteLine(p.Name);               // Bob
    }
}
```

**Expected Output**
```
Alice
Bob
```

**Why this output occurs**
`GetValue` reads the property; `SetValue` writes it.

---

### Real-World Cases

- **Serialization** — Read/write properties for JSON/XML.
- **Data binding** — Bind UI to properties dynamically.
- **ORM** — Map properties to columns.

---

### References

- PropertyInfo Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.propertyinfo

---

## 6. `FieldInfo`

### Definitions

**Core Definition**
`FieldInfo` is the reflection class representing a **field** (public, private, static, or instance) of a type, enabling inspection and dynamic read/write.

**Technical Definition**
`System.Reflection.FieldInfo` exposes `Name`, `FieldType`, `IsStatic`, `IsPublic`, `IsInitOnly`, `GetValue(object? obj)`, and `SetValue(object? obj, object? value)`. It is obtained via `Type.GetField(name, BindingFlags)` or `Type.GetFields()`.

**Beginner-Friendly Explanation**
`FieldInfo` describes a **field** and lets you get/set its value dynamically.

---

### Purposes

- To **inspect field metadata**.
- To **dynamically read/write field values**.
- To **access private fields** (in reflection scenarios).

---

### Syntax Rules and Structure

```csharp
FieldInfo? field = type.GetField("_age", BindingFlags.NonPublic | BindingFlags.Instance);
object? value = field?.GetValue(instance);
field?.SetValue(instance, 42);
```

**Component breakdown**

- `GetField(name, flags)` — Find field.
- `GetValue(obj)` — Read.
- `SetValue(obj, value)` — Write.

#### Syntax Rules

- Private fields require `BindingFlags.NonPublic`.
- `readonly` fields can be set via reflection (in some cases).
- Static fields use `null` instance.

#### Constraints and Limitations

- Accessing private fields is fragile and discouraged.
- Trimming may remove metadata for private fields.

---

### Annotated Code Examples and Expected Outputs

#### Example: Accessing a Private Field

```csharp
using System;
using System.Reflection;

public class Counter
{
    private int _count = 0;
    public int Count => _count;
}

class Program
{
    static void Main()
    {
        var c = new Counter();
        FieldInfo? field = typeof(Counter).GetField("_count", BindingFlags.NonPublic | BindingFlags.Instance);
        Console.WriteLine(field?.GetValue(c));   // 0
        field?.SetValue(c, 42);
        Console.WriteLine(c.Count);              // 42
    }
}
```

**Expected Output**
```
0
42
```

**Why this output occurs**
Reflection can access private fields with `BindingFlags.NonPublic`.

---

### Real-World Cases

- **Serialization** — Access private fields.
- **Testing** — Set up private state.
- **Diagnostics** — Inspect object state.

---

### References

- FieldInfo Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.fieldinfo

---

## 7. Reflection Performance

### Definitions

**Core Definition**
Reflection performance is the **cost of inspecting and invoking members dynamically**, which is significantly higher than direct calls.

**Technical Definition**
Reflection is slower than direct calls because it involves metadata lookups, dynamic dispatch, boxing, parameter array allocation, and security checks. The cost sources include: `Type.GetMethod()` lookups, `Invoke` parameter binding, `TargetInvocationException` wrapping, and value-type boxing. Optimizations include: **caching `MethodInfo`/`PropertyInfo`**, **using `MethodInfo.CreateDelegate`** for strongly typed delegates, **compiling expressions** with `Expression<T>`, and **avoiding reflection in hot paths**. In .NET 7+, reflection has been optimized (faster `Invoke`), but it is still much slower than direct calls.

**Beginner-Friendly Explanation**
Reflection is **slow**—cache `MethodInfo` and use `CreateDelegate` to speed it up.

---

### Purposes

- To understand the cost of reflection.
- To optimize reflection-heavy code.
- To choose alternatives (delegates, expressions, source generators).

---

### Syntax Rules and Structure (Optimization Techniques)

| Technique | Benefit |
|---|---|
| Cache `MethodInfo` | Avoid repeated lookups |
| `CreateDelegate` | Faster repeated invocation |
| `Expression<T>` | Compile dynamic calls to delegates |
| Source generators | Avoid runtime reflection |

**Component breakdown**

- **Caching** — Store `MethodInfo`/`PropertyInfo`.
- **Delegates** — `CreateDelegate` for repeated calls.
- **Expressions** — Compile to delegates.
- **Source generators** — Compile-time code generation.

#### Syntax Rules

- Always cache reflection lookups.
- Use `CreateDelegate` when invoking the same method repeatedly.
- Prefer source generators for serialization and DI in AOT.

#### Constraints and Limitations

- Caching adds complexity.
- `CreateDelegate` requires a compatible delegate type.

---

### Annotated Code Examples and Expected Outputs

#### Example: Caching and `CreateDelegate`

```csharp
using System;
using System.Reflection;

public class Greeter
{
    public string Greet(string name) => $"Hello, {name}";
}

class Program
{
    static void Main()
    {
        var greeter = new Greeter();
        MethodInfo method = typeof(Greeter).GetMethod("Greet")!;
        var del = (Func<string, string>)method.CreateDelegate(typeof(Func<string, string>), greeter);
        Console.WriteLine(del("Alice"));   // Hello, Alice
    }
}
```

**Expected Output**
```
Hello, Alice
```

**Why this output occurs**
`CreateDelegate` creates a strongly typed delegate bound to `greeter`; invocation is much faster than `Invoke`.

---

### Real-World Cases

- **Hot paths** — Cache `MethodInfo` and use delegates.
- **Serialization** — Use source generators to avoid reflection.
- **DI containers** — Compile expressions for instantiation.

---

### References

- Reflection performance — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/reflection

---

## 8. Reflection Best Practices

### Definitions

**Core Definition**
Reflection best practices are the guidelines for using reflection safely, efficiently, and maintainably.

**Technical Definition**
Best practices include: **cache reflection results** (`Type`, `MethodInfo`, `PropertyInfo`), **use `CreateDelegate` or compiled expressions** for repeated invocations, **avoid reflection in hot paths**, **prefer source generators** for serialization and DI (especially in AOT), **validate inputs** (type names, member names) to avoid exceptions, **use `BindingFlags` correctly**, **avoid accessing private members** unless necessary, **handle `TargetInvocationException`**, and **be AOT-aware** (trimming removes unreferenced metadata).

**Beginner-Friendly Explanation**
The best practices are: **cache results**, **use delegates**, **avoid hot-path reflection**, and **prefer source generators for AOT**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Cache reflection results
private static readonly MethodInfo _greet = typeof(Greeter).GetMethod("Greet")!;

// 2. Use CreateDelegate for repeated invocations
var del = (Func<string, string>)_greet.CreateDelegate(typeof(Func<string, string>), instance);

// 3. Avoid reflection in hot paths
// Use direct calls or precompiled delegates.

// 4. Prefer source generators for AOT
// e.g., System.Text.Json source generation, LoggerMessage, GeneratedRegex.

// 5. Validate inputs
if (type.GetMethod(name) is not MethodInfo m) throw new InvalidOperationException();

// 6. Handle TargetInvocationException
try { method.Invoke(instance, args); }
catch (TargetInvocationException ex) { /* handle ex.InnerException */ }
```

**Component breakdown**

- **Caching** — Store metadata.
- **Delegates** — Faster invocation.
- **Source generators** — AOT-friendly.
- **Validation** — Avoid exceptions.
- **Exception handling** — Unwrap `TargetInvocationException`.

#### Rules and Constraints

- **Do** cache reflection results.
- **Do** use `CreateDelegate`/expressions for repeated calls.
- **Do** prefer source generators for AOT.
- **Do** validate inputs.
- **Do not** use reflection in hot paths.
- **Do not** access private members unnecessarily.
- **Do not** ignore `TargetInvocationException`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Cached Reflection with Error Handling

```csharp
using System;
using System.Reflection;

public class Greeter
{
    public string Greet(string name) => $"Hello, {name}";
}

public static class GreeterInvoker
{
    private static readonly MethodInfo _method = typeof(Greeter).GetMethod("Greet")!;

    public static string Invoke(Greeter g, string name)
    {
        try { return (string)_method.Invoke(g, new object[] { name })!; }
        catch (TargetInvocationException ex) { throw ex.InnerException!; }
    }
}

class Program
{
    static void Main()
    {
        Console.WriteLine(GreeterInvoker.Invoke(new Greeter(), "Alice"));   // Hello, Alice
    }
}
```

**Expected Output**
```
Hello, Alice
```

**Why this output occurs**
`MethodInfo` is cached; `TargetInvocationException` is unwrapped for cleaner error handling.

---

### Real-World Cases

- **DI containers** — Cache constructors; use compiled expressions.
- **Serialization** — Source generators for AOT.
- **Testing** — Cache `MethodInfo` for test methods.

---

### References

- Reflection (C#) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/reflection
- Reflection performance — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/reflection

---

## 9. `System.Reflection.Metadata`

### Definitions

**Core Definition**
`System.Reflection.Metadata` is a **low-level, allocation-efficient API** for reading ECMA-335 metadata without loading assemblies into the runtime.

**Technical Definition**
`System.Reflection.Metadata` (introduced in .NET Core 1.0) provides `MetadataReader` and related types that read metadata from a `PEReader`. It is designed for **tools, analyzers, and source generators** that need to inspect assemblies without loading them (which would require the runtime and may execute code). It is **allocation-efficient** and **AOT-friendly**, making it the preferred API for compile-time analysis.

**Beginner-Friendly Explanation**
`System.Reflection.Metadata` is **reflection without loading the assembly**—fast, allocation-efficient, and AOT-friendly.

---

### Purposes

- To **inspect metadata without loading assemblies**.
- To support **analyzers and source generators**.
- To provide **allocation-efficient** metadata reading.
- To be **AOT-friendly** (no runtime dependency).

---

### Syntax Rules and Structure

```csharp
using System.Reflection.Metadata;
using System.Reflection.PortableExecutable;

using var stream = File.OpenRead("MyAssembly.dll");
using var peReader = new PEReader(stream);
MetadataReader reader = peReader.GetMetadataReader();

foreach (var handle in reader.TypeDefinitions)
{
    TypeDefinition type = reader.GetTypeDefinition(handle);
    string name = reader.GetString(type.Name);
    Console.WriteLine(name);
}
```

**Component breakdown**

- `PEReader` — Reads the PE file.
- `MetadataReader` — Reads metadata.
- `TypeDefinition` — Type metadata.

#### Syntax Rules

- Use `MetadataReader` for metadata-only reading.
- Combine with `PEReader` for file-based reading.
- Supports types, methods, fields, attributes, and references.

#### Constraints and Limitations

- Low-level API; more complex than `System.Reflection`.
- Does not execute code.
- Requires careful handle management.

---

### Annotated Code Examples and Expected Outputs

#### Example: Reading Type Names from an Assembly

```csharp
using System;
using System.IO;
using System.Reflection.Metadata;
using System.Reflection.PortableExecutable;

class Program
{
    static void Main()
    {
        using var stream = File.OpenRead(typeof(Program).Assembly.Location);
        using var peReader = new PEReader(stream);
        MetadataReader reader = peReader.GetMetadataReader();

        foreach (var handle in reader.TypeDefinitions)
        {
            var type = reader.GetTypeDefinition(handle);
            string name = reader.GetString(type.Name);
            Console.WriteLine(name);
        }
    }
}
```

**Expected Output (example)**
```
<Module>
Program
```

**Why this output occurs**
`MetadataReader` enumerates type definitions without loading the assembly.

---

### Real-World Cases

- **Source generators** — Analyze assemblies at compile time.
- **Analyzers** — Inspect metadata without loading.
- **AOT-friendly tooling** — Avoid runtime reflection.

---

### References

- System.Reflection.Metadata — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.metadata
- MetadataReader — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.metadata.metadatareader

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `System.Reflection` | Core since .NET 1.0; stable. |
| `MethodInfo.Invoke` | Slower than `CreateDelegate`; cache and use delegates. |
| `CreateDelegate` | Available since .NET 2.0; recommended for repeated calls. |
| `Expression<T>` compilation | Available since .NET 3.5; compile dynamic calls. |
| `System.Reflection.Metadata` | Introduced in **.NET Core 1.0**; stable. |
| Reflection and trimming | **Trimming removes unreferenced metadata**; use `DynamicDependency` or source generators. |
| Reflection in Native AOT | **Restricted**; use source generators. |
| `Activator.CreateInstance` | **Slow**; cache and use delegates/expressions. |

---

## References (Consolidated)

- Reflection (C#) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/reflection
- System.Reflection — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection
- Type Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.type
- Assembly Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.assembly
- MethodInfo Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.methodinfo
- PropertyInfo Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.propertyinfo
- FieldInfo Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.fieldinfo
- Reflection performance — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/reflection
- System.Reflection.Metadata — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.metadata
- MetadataReader — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.reflection.metadata.metadatareader

---