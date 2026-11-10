# C# Static Members — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **static member** in C# is a member that belongs to the **type itself** rather than to any specific instance (object) of that type. Static members are shared across all instances of the type, exist for the entire lifetime of the application domain, and are accessed through the type name (e.g., `Math.PI`, `Console.WriteLine`) instead of through an object reference. The `static` modifier can be applied to classes, fields, methods, properties, operators, events, and constructors, enabling functionality that is independent of instance state.

**Technical Definition**
When a field, method, property, event, operator, or constructor declaration includes a `static` modifier, it declares a static member. In addition, a constant or type declaration implicitly declares a static member. Static members have the following characteristics: a static field in a non-generic class identifies exactly one storage location, regardless of how many instances are created; each distinct closed constructed generic type has its own set of static fields; a static function member does not operate on a specific instance, and it is a compile-time error to refer to `this` in such a member; static members are referenced through the type name (`E.M` where `E` denotes a type), and it is a compile-time error to reference a static member through an instance. A **static class** is a class that is sealed, abstract, and contains only static members (plus inherited `System.Object` members and possibly a private constructor). A **static constructor** is called automatically at most once, before the first instance is created or any static member is referenced, and is used to initialise static data.

**Beginner-Friendly Explanation**
Static members are things that belong to the **class itself**, not to individual objects. Imagine a `Math` class: you do not create a `Math` object to calculate a square root—you just call `Math.Sqrt(16)` directly on the class. That is because `Sqrt` is a static method. A static field is like a **whiteboard in a classroom** shared by all students (instances); if one student writes on it, everyone sees the change. An instance field, by contrast, is like each student's **personal notebook**. A static class is a **utility drawer** that cannot be turned into an object—it just holds useful static tools. A static constructor is the **one-time setup routine** that runs the very first time the class is touched, no matter whether you create an object or just access a static member.

---

### Key Characteristics

- **Type-level membership** — Static members belong to the type, not to instances; they are shared across all objects of that type.
- **Single storage location** — A static field in a non-generic class has exactly one storage location, no matter how many instances exist.
- **Accessed via type name** — Static members are referenced through the type (`TypeName.Member`), not through an object reference; using an instance to access a static member is a compile-time error.
- **No `this` reference** — Static function members cannot use the `this` keyword because they do not operate on a specific instance.
- **Static classes are sealed and abstract** — A static class cannot be instantiated, cannot be used as a base class, and contains only static members.
- **One-time static constructor** — A static constructor runs at most once, automatically, before the first instance creation or static member access.
- **Implicit static members** — Constants and nested type declarations are implicitly static members.
- **Per-closed-generic-type statics** — Each closed constructed generic type has its own copy of static fields.

---

### Prerequisites

- Basic understanding of C# classes, objects, and instance members.
- Familiarity with fields, methods, properties, and constructors.
- Knowledge of access modifiers (`public`, `private`, `internal`, `protected`).
- Awareness of the difference between value types and reference types.
- Basic understanding of type initialisation and the CLR execution model.

---

### Related Programming Areas

- **Utility and helper classes** — Static classes are the standard home for stateless helper methods (e.g., `Math`, `Console`, `File`).
- **Extension methods** — Extension methods must be declared in static classes and are themselves static methods.
- **Global state and configuration** — Static fields are often used for application-wide settings, though this should be done cautiously.
- **Singleton pattern** — Static fields and static constructors are commonly used to implement singletons.
- **Performance** — Static methods can be slightly faster to invoke because they do not require an instance reference.
- **Thread safety** — Static fields are shared mutable state and require synchronisation in multithreaded scenarios.

---

### Core Concepts / Features

1. Static Classes
2. Static Methods
3. Static Fields
4. Static Properties
5. Static Constructors
6. Static Members Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Static Classes

### Definitions

**Core Definition**
A static class is a class that cannot be instantiated and is designed to serve as a container for static members only. It is declared with the `static` modifier and is implicitly `sealed` and `abstract`.

**Technical Definition**
A static class is a class that contains only static members (besides the instance members inherited from `System.Object` and possibly a private constructor). In C# 2.0 and later, when a class is declared `static`, it is **sealed** (cannot be inherited from) and **abstract** (cannot be instantiated), and no instance members can be declared or overridden. A static class cannot be used as a base class, cannot inherit from any class other than `object`, and cannot contain instance constructors (though it can contain a static constructor). Static classes are a compromise between pure object-oriented design and simplicity, commonly used for utility functions, constants, extension methods, and operations that do not require object state.

**Beginner-Friendly Explanation**
A static class is a **sealed toolbox**. You cannot create objects from it, and you cannot inherit from it. It simply holds useful static methods and fields that you call directly through the class name.

---

### Purposes

- To group related utility functions that do not require object state.
- To serve as a container for extension methods.
- To hold application-wide constants and configuration values.
- To provide factory methods that create and return objects.
- To prevent instantiation of a class that is only meant to hold static members.

---

### Syntax Rules and Structure

```
[access modifier] static class ClassName
{
    // static members only
}
```

**Component breakdown**

- `static` — The modifier that declares the class as static.
- `ClassName` — The class identifier (PascalCase by convention).
- `{ ... }` — The class body containing only static members.

#### Syntax Rules

- A static class cannot be instantiated with `new`.
- A static class cannot be used as a base class.
- A static class cannot inherit from any class other than `object`.
- A static class can only contain static members (methods, fields, properties, events).
- A static class cannot contain instance constructors, but can contain a static constructor.
- All members of a static class must be static.

#### Constraints and Limitations

- Cannot be used with `new`.
- Cannot be inherited from.
- Cannot implement interfaces (though static abstract interface members in C# 11+ provide related capabilities).
- Cannot contain instance members.
- Cannot be used as a type argument for generic constraints that require instantiation.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Static Utility Class

```csharp
using System;

public static class StringHelper
{
    public static bool IsNullOrEmpty(string text)
        => string.IsNullOrEmpty(text);

    public static string Reverse(string text)
    {
        if (text == null) return null;
        char[] chars = text.ToCharArray();
        Array.Reverse(chars);
        return new string(chars);
    }

    public static int CountWords(string text)
    {
        if (string.IsNullOrWhiteSpace(text)) return 0;
        return text.Split(new[] { ' ', '\t', '\n' },
            StringSplitOptions.RemoveEmptyEntries).Length;
    }
}

class Program
{
    static void Main()
    {
        string sample = "Hello, world!";
        Console.WriteLine(StringHelper.IsNullOrEmpty(sample)); // False
        Console.WriteLine(StringHelper.Reverse(sample));       // !dlrow ,olleH
        Console.WriteLine(StringHelper.CountWords(sample));    // 2
    }
}
```

**Expected Output**

```
False
!dlrow ,olleH
2
```

**Why this output occurs**

- `StringHelper` is a static class; its methods are called directly through the class name without creating an instance.

---

#### Example 2: Static Class with a Static Constructor

```csharp
using System;

public static class AppConfig
{
    public static readonly string AppName;
    public static readonly int MaxRetries;

    static AppConfig()
    {
        AppName = "MyApplication";
        MaxRetries = 3;
        Console.WriteLine("Static constructor ran");
    }
}

class Program
{
    static void Main()
    {
        Console.WriteLine(AppConfig.AppName);   // Static constructor ran
                                                 // MyApplication
        Console.WriteLine(AppConfig.MaxRetries); // 3
    }
}
```

**Expected Output**

```
Static constructor ran
MyApplication
3
```

**Why this output occurs**

- The static constructor runs once before the first access to `AppName`; subsequent accesses do not re-run it.

---

### Real-World Cases

- **`System.Math`** — Static class with mathematical constants and functions.
- **`System.Console`** — Static class for console input/output.
- **`System.IO.File`** — Static class providing shortcuts to file operations.
- **`System.Environment`** — Static class exposing environment information.

---

### References

- Static Class Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/static-class
- Static classes and static class members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members
- static modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/static

---

## 2. Static Methods

### Definitions

**Core Definition**
A static method is a method that belongs to the type itself and does not operate on a specific instance. It is declared with the `static` modifier and is invoked through the type name.

**Technical Definition**
A method declared with the `static` modifier is a static method. A static method does not operate on a specific instance and can only directly access static members of its containing type. It is a compile-time error to refer to `this` in a static method. Static methods are invoked using the type name (`TypeName.MethodName(...)`), not through an object reference. A static method can be overloaded, but it cannot be marked `virtual`, `abstract`, or `override` (though static abstract members are allowed in interfaces since C# 11).

**Beginner-Friendly Explanation**
A static method is a **utility function attached to a class**. You do not need an object to call it—just the class name. `Math.Sqrt(16)` is a static method: you do not create a `Math` object to compute a square root.

---

### Purposes

- To provide utility functions that do not depend on object state.
- To implement factory methods that create and return objects.
- To group related operations under a type name.
- To avoid unnecessary object allocation.
- To serve as the implementation mechanism for extension methods (which are static methods).

---

### Syntax Rules and Structure

```
[access modifier] static returnType MethodName(parameters)
{
    // body
}
```

**Component breakdown**

- `static` — The modifier.
- `returnType` — The method's return type, or `void`.
- `MethodName` — The method identifier (PascalCase by convention).
- `parameters` — The parameter list.

#### Syntax Rules

- Static methods cannot access instance members directly (no `this`).
- They can access other static members of the same type.
- They are invoked via the type name, not an instance.
- They can be overloaded.
- They cannot be `virtual`, `abstract`, or `override` (except static abstract interface members in C# 11+).

#### Constraints and Limitations

- Cannot use `this`.
- Cannot directly access instance fields or methods.
- Cannot be overridden.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Static Method

```csharp
using System;

public class MathHelper
{
    public static int Square(int x) => x * x;
}

class Program
{
    static void Main()
    {
        Console.WriteLine(MathHelper.Square(5));   // 25
    }
}
```

**Expected Output**

```
25
```

**Why this output occurs**

- `Square` is static and is invoked through the type name `MathHelper`.

---

#### Example 2: Static Method in a Non-Static Class

```csharp
using System;

public class Calculator
{
    public static double Add(double a, double b) => a + b;
    public static double Subtract(double a, double b) => a - b;
}

class Program
{
    static void Main()
    {
        Console.WriteLine(Calculator.Add(3, 4));       // 7
        Console.WriteLine(Calculator.Subtract(10, 3)); // 7
    }
}
```

**Expected Output**

```
7
7
```

**Why this output occurs**

- The static methods are called directly on the `Calculator` class without creating an instance.

---

### Real-World Cases

- **`Math.Abs`, `Math.Max`** — Static utility methods.
- **`DateTime.Parse`, `Guid.NewGuid`** — Static factory methods.
- **`string.Concat`, `string.Join`** — Static string helpers.
- **`File.ReadAllText`** — Static file I/O convenience method.

---

### References

- static modifier — C# reference (methods) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/static
- Static classes and static class members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members
- Classes — C# language specification (static members) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes

---

## 3. Static Fields

### Definitions

**Core Definition**
A static field is a variable declared with the `static` modifier that belongs to the type itself. There is exactly one copy of a static field per type (or per closed generic type), shared by all instances.

**Technical Definition**
A field declared with the `static` modifier defines a static field. In a non-generic class, a static field identifies exactly one storage location, regardless of how many instances of the class are created. Each distinct closed constructed generic type has its own set of static fields. Static fields are initialised to their default values (`0`, `false`, `null`) and are then subject to static field initialisers and the static constructor. Static fields marked with `[ThreadStatic]` have one copy per thread.

**Beginner-Friendly Explanation**
A static field is a **shared variable** that belongs to the class, not to any object. Every object sees the same value. If you change it through one object (or through the class name), every other object sees the change.

---

### Purposes

- To store data shared across all instances of a type.
- To implement counters and caches at the type level.
- To hold application-wide configuration or constants.
- To provide a single storage location for type-wide state.
- To support the singleton pattern and similar shared-state designs.

---

### Syntax Rules and Structure

```
[access modifier] static type FieldName [= initializer];
```

**Component breakdown**

- `static` — The modifier.
- `type` — The field's type.
- `FieldName` — The field identifier (camelCase for private, PascalCase for public).
- `= initializer` — Optional initial value.

#### Syntax Rules

- There is one copy of a static field per type (or per closed generic type).
- Static fields are initialised to default values before static field initialisers run.
- Static field initialisers run in textual order before the static constructor (if any).
- Access is through the type name (`TypeName.FieldName`).

#### Constraints and Limitations

- Shared mutable static state requires synchronisation in multithreaded code.
- Static fields can create hidden dependencies and make testing harder.
- `[ThreadStatic]` fields have per-thread copies.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Shared Counter

```csharp
using System;

public class Employee
{
    public static int EmployeeCount;   // static field
    public string Name;

    public Employee(string name)
    {
        Name = name;
        EmployeeCount++;
    }
}

class Program
{
    static void Main()
    {
        var e1 = new Employee("Alice");
        var e2 = new Employee("Bob");
        Console.WriteLine(Employee.EmployeeCount);   // 2
    }
}
```

**Expected Output**

```
2
```

**Why this output occurs**

- `EmployeeCount` is static; each constructor increments the same shared storage location.

---

#### Example 2: Static Field in a Generic Type

```csharp
using System;

public class Counter<T>
{
    public static int Count;
}

class Program
{
    static void Main()
    {
        Counter<int>.Count = 5;
        Counter<string>.Count = 10;

        Console.WriteLine(Counter<int>.Count);    // 5
        Console.WriteLine(Counter<string>.Count); // 10
    }
}
```

**Expected Output**

```
5
10
```

**Why this output occurs**

- Each closed generic type (`Counter<int>` and `Counter<string>`) has its own copy of the static field.

---

### Real-World Cases

- **Instance counters** — `public static int InstanceCount;`
- **Caches** — `private static Dictionary<string, object> _cache;`
- **Configuration** — `public static readonly string AppName;`
- **Thread-local storage** — `[ThreadStatic] private static int _threadId;`

---

### References

- static modifier — C# reference (fields) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/static
- Classes — C# language specification (static fields) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- ThreadStaticAttribute — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.threadstaticattribute

---

## 4. Static Properties

### Definitions

**Core Definition**
A static property is a property declared with the `static` modifier. It belongs to the type itself and is accessed through the type name, not through an instance.

**Technical Definition**
When a property declaration includes the `static` modifier, the property is a static property. A static property is not associated with a specific instance, and its accessors (`get` and `set`) cannot use `this`. Static properties can be read-only, read-write, or init-only, and they can be auto-implemented or manually implemented. They are invoked through the type name (`TypeName.PropertyName`).

**Beginner-Friendly Explanation**
A static property is a **property on the class itself**. `Math.PI` is a static property: you access it through the `Math` type, not through an object.

---

### Purposes

- To expose type-level state with controlled access.
- To provide singleton-like values (`Instance` property).
- To share configuration or constants across all instances.
- To implement utility properties that do not depend on object state.
- To provide lazy-initialised static values.

---

### Syntax Rules and Structure

```
[access modifier] static type PropertyName { get; set; }
[access modifier] static type PropertyName => expression;
```

**Component breakdown**

- `static` — The modifier.
- `type` — The property's type.
- `PropertyName` — The property identifier.
- Accessors — `get`, `set`, or `init`.

#### Syntax Rules

- Static properties cannot use `this`.
- They are accessed via the type name.
- They can be auto-implemented or manually implemented.
- They can have different access levels for `get` and `set`.

#### Constraints and Limitations

- Shared mutable static properties require synchronisation in concurrent scenarios.
- Cannot be `virtual`, `abstract`, or `override` in the traditional sense.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Static Auto-Property

```csharp
using System;

public class Configuration
{
    public static string AppName { get; set; } = "MyApp";
}

class Program
{
    static void Main()
    {
        Console.WriteLine(Configuration.AppName);   // MyApp
        Configuration.AppName = "NewApp";
        Console.WriteLine(Configuration.AppName);   // NewApp
    }
}
```

**Expected Output**

```
MyApp
NewApp
```

**Why this output occurs**

- `AppName` is a static auto-property; it is accessed through the type name and can be read and written.

---

#### Example 2: Static Computed Property

```csharp
using System;

public class Circle
{
    public static double Pi => Math.PI;
    public static double UnitCircleArea => Pi;
}

class Program
{
    static void Main()
    {
        Console.WriteLine(Circle.UnitCircleArea.ToString("F5"));   // 3.14159
    }
}
```

**Expected Output**

```
3.14159
```

**Why this output occurs**

- `UnitCircleArea` is a static read-only computed property that returns the area of a unit circle.

---

### Real-World Cases

- **Constants** — `Math.PI`, `string.Empty`.
- **Singleton instances** — `public static Singleton Instance { get; }`.
- **Environment information** — `Environment.NewLine`, `Environment.MachineName`.

---

### References

- static modifier — C# reference (properties) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/static
- Classes — C# language specification (static properties) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- Static classes and static class members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members

---

## 5. Static Constructors

### Definitions

**Core Definition**
A static constructor is a special constructor that runs **automatically at most once**, before the first instance is created or any static member is referenced, and is used to initialise static data or perform a one-time action.

**Technical Definition**
A static constructor is used to initialise any static data, or to perform a particular action that needs to be performed only once. It is called automatically before the first instance is created or any static members are referenced, and it is called at most once. A static constructor does not take access modifiers or parameters, a class or struct can have only one static constructor, and it cannot be inherited or overloaded. It cannot be called directly and is invoked automatically by the CLR. The order of static constructor execution is not specified, but all static constructors in a hierarchy run before any instances are created. A notable exception: if a static field initializer creates an instance of the type, the instance constructor runs before the static constructor.

**Beginner-Friendly Explanation**
A static constructor is the **one-time setup routine** for a class. It runs the very first time the class is used—whether you create an object or just touch a static member. It runs once and never again.

---

### Purposes

- To initialise static fields with non-trivial logic.
- To read configuration or environment data once.
- To enforce invariants on static state.
- To perform one-time setup for the type.
- To implement the singleton pattern.

---

### Syntax Rules and Structure

```
static ClassName()
{
    // static initialisation
}
```

**Component breakdown**

- `static` — The modifier.
- `ClassName` — The type name (must match the enclosing type).
- `()` — No parameters.
- Body — Initialisation statements.

#### Syntax Rules

- No access modifiers, no parameters.
- Cannot be called explicitly.
- Runs at most once per application domain.
- Runs before the first instance creation or static member access.
- Cannot be inherited or overloaded.

#### Constraints and Limitations

- If a static constructor throws an exception, the type becomes unusable for the lifetime of the application domain (the CLR wraps the exception in `TypeInitializationException`).
- The exact timing of execution is implementation-dependent in the absence of explicit triggers, but it is guaranteed to run before first use.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Static Constructor Initialising a Field

```csharp
using System;

public class SimpleClass
{
    static readonly long baseline;

    static SimpleClass()
    {
        baseline = DateTime.Now.Ticks;
        Console.WriteLine("Static constructor ran");
    }

    public static long Baseline => baseline;
}

class Program
{
    static void Main()
    {
        Console.WriteLine(SimpleClass.Baseline);
        // Output: Static constructor ran
        //         <ticks value>
        Console.WriteLine(SimpleClass.Baseline); // <ticks value> (no rerun)
    }
}
```

**Expected Output (approximate)**

```
Static constructor ran
638123456789012345
638123456789012345
```

**Why this output occurs**

- The static constructor runs once, before the first access to `Baseline`; subsequent accesses reuse the initialised value.

---

#### Example 2: Static Constructor and Static Field Initialisers

```csharp
using System;

class Test
{
    static void Main()
    {
        Console.WriteLine($"{B.Y} {A.X}");
    }

    public static int F(string s)
    {
        Console.WriteLine(s);
        return 1;
    }
}

class A
{
    static A() { }
    public static int X = Test.F("Init A");
}

class B
{
    static B() { }
    public static int Y = Test.F("Init B");
}
```

**Expected Output**

```
Init B
Init A
1 1
```

**Why this output occurs**

- The presence of static constructors makes the initialisation order deterministic: `B`'s static constructor (and thus its field initialisers) runs before `A`'s, because `B.Y` is referenced first.

---

### Real-World Cases

- **Configuration loading** — Reading environment variables once.
- **Singleton pattern** — `private static Singleton instance = new Singleton();` with a static constructor for additional setup.
- **Precomputed lookup tables** — Building static arrays or dictionaries at first use.

---

### References

- Static Constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-constructors
- Classes — C# language specification (static field initialisation) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- TypeInitializationException — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.typeinitializationexception

---

## 6. Static Members Best Practices

### Definitions

**Core Definition**
Static member best practices are the design guidelines that ensure static members are used **sparingly, safely, and maintainably**, avoiding hidden dependencies, global mutable state, and testing difficulties.

**Technical Definition**
Best practices for static members derive from the Framework Design Guidelines and community guidance. Key recommendations include: **use static classes sparingly**, only as supporting classes for the object-oriented core of the framework; **do not treat static classes as a miscellaneous bucket**; **avoid static state** because it is shared across the application and can create hidden dependencies; **keep static methods pure** where possible; **name static classes appropriately** and group related functionality; **declare static classes as sealed, abstract, and add a private instance constructor if the language does not have built-in support for static classes**; and **avoid global mutable state** because it complicates testing and concurrent programming.

**Beginner-Friendly Explanation**
Best practices for static members are simple: **use them when you have a stateless utility**, **keep them free of global mutable state**, and **do not turn a static class into a dumping ground for unrelated helpers**. Static members are convenient but can make code harder to test and reason about if overused.

---

### Purposes

- To prevent overuse of static members and global state.
- To keep static classes focused and cohesive.
- To avoid hidden dependencies and testing difficulties.
- To ensure thread safety for shared state.
- To align with .NET design conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Use static classes sparingly for utility functions
public static class MathHelper
{
    public static int Square(int x) => x * x;
}

// Avoid static mutable state
// Prefer stateless static methods
// Do not treat static classes as miscellaneous buckets
// Name static classes appropriately (e.g., StringHelper, MathUtilities)
// Group related functionality
```

**Component breakdown**

- **Sparingly** — Static classes should be supporting classes, not the core of the design.
- **No miscellaneous bucket** — Static classes should have a clear, focused purpose.
- **Avoid state** — Static mutable state creates hidden dependencies and testing challenges.
- **Pure methods** — Prefer static methods that depend only on their parameters.
- **Appropriate naming** — Use names that reflect the purpose (e.g., `StringHelper`).

#### Rules and Constraints

- **Do** use static classes sparingly.
- **Do** keep static methods pure when possible.
- **Do** avoid static mutable state.
- **Do** group related functionality in static classes.
- **Do not** treat static classes as a miscellaneous bucket.
- **Do not** declare instance members in static classes.
- **Do not** rely on global state for application logic.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Static Class

```csharp
using System;

public static class MathHelper
{
    public static int Square(int x) => x * x;
    public static int Cube(int x) => x * x * x;
    public static bool IsEven(int x) => x % 2 == 0;
}

class Program
{
    static void Main()
    {
        Console.WriteLine(MathHelper.Square(5));   // 25
        Console.WriteLine(MathHelper.Cube(3));     // 27
        Console.WriteLine(MathHelper.IsEven(4));   // True
    }
}
```

**Expected Output**

```
25
27
True
```

**Why this output occurs**

- `MathHelper` is a focused static class with pure, stateless methods that depend only on their parameters.

---

### Real-World Cases

- **`System.Math`** — A well-designed static class with pure mathematical functions.
- **`System.Console`** — Static class for console I/O (has some state but is carefully designed).
- **Extension method containers** — Focused static classes for specific types (e.g., `Enumerable`).

---

### References

- Static Class Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/static-class
- Static classes and static class members — C# Programming Guide (best practices) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members
- C# static classes best practices — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/2-net-object-oriented-programming/12-net-static-classes.mdx

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Static classes | Core since C# 2.0; stable. |
| Static constructors | Core since C# 1.0; stable. |
| Static abstract interface members | Introduced in **C# 11**; enables generic math. |
| `[ThreadStatic]` fields | Available since .NET 1.0; per-thread copies. |
| Static local functions | Introduced in **C# 8.0**; cannot capture enclosing state. |
| Static lambdas / anonymous methods | Introduced in **C# 9.0**; cannot capture enclosing state. |
| Static field initialisation order | **Implementation-dependent** unless a static constructor is present. |
| Static constructor exceptions | **Uncatchable** for the type; wrapped in `TypeInitializationException`. |
| Global mutable static state | **Discouraged**; complicates testing and concurrency. |

---

## References (Consolidated)

- static modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/static
- Static classes and static class members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members
- Static Class Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/static-class
- Static Constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-constructors
- Classes — C# language specification (static members, static field initialisation) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- ThreadStaticAttribute — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.threadstaticattribute
- TypeInitializationException — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.typeinitializationexception
- C# static classes best practices — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/2-net-object-oriented-programming/12-net-static-classes.mdx

---