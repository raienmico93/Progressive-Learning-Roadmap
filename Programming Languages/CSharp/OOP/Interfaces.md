# C# Interfaces — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An interface in C# is a **contract** that defines a set of related functionalities (methods, properties, events, and indexers) that any implementing class, record, or struct must support. It specifies **what** a type must do, but not **how** it must do it. Interfaces are the primary tool for achieving **multiple inheritance of contracts** in C#, enabling unrelated types to be treated polymorphically through a shared capability, and for decoupling callers from concrete implementations.

**Technical Definition**
An interface defines a contract. A class or struct that implements an interface shall adhere to its contract. An interface may inherit from multiple base interfaces, and a class or struct may implement multiple interfaces. An interface declaration introduces a new type using the `interface` keyword, followed by an identifier, an optional type parameter list, an optional base interface list, and a body containing member declarations. Historically, interface members were implicitly `public` and `abstract`, with no implementations. Since C# 8, interfaces may provide **default implementations** for some or all of their members, and since C# 11, interfaces may declare **`static abstract`** and **`static virtual`** members, enabling generic programming over operators and other static members. Interface members without a body remain abstract and must be implemented by implementing types. Explicit interface implementation allows a class to provide separate implementations for members with the same signature from different interfaces.

**Beginner-Friendly Explanation**
An interface is a **job description**. It lists the tasks (methods and properties) that a role must be able to perform, but it does not say how to perform them. Any class that “applies for the job” must implement every task listed. Different classes can implement the same interface in completely different ways—a `Dog` and a `Robot` can both implement `IMakeSound`, but the dog barks and the robot beeps. Interfaces let you write code that works with **anything that can do the job**, without caring whether it is a dog, a robot, or something else entirely. C# does not allow a class to inherit from multiple classes, but it **does** allow a class to implement multiple interfaces—so interfaces are how C# achieves the flexibility of multiple inheritance without its complications.

---

### Key Characteristics

- **Contract-based** — An interface defines a contract that implementing types must fulfill.
- **Multiple implementation** — A class or struct can implement multiple interfaces, though it can inherit from only one class.
- **Interface inheritance** — Interfaces can inherit from one or more base interfaces, composing contracts.
- **Implicitly public abstract members** — Members without implementations are `public` and `abstract` by default.
- **Default interface methods (C# 8)** — Interfaces can provide default implementations, allowing non-breaking evolution of published interfaces.
- **Static abstract/virtual members (C# 11)** — Interfaces can declare static members for generic math and operator constraints.
- **Explicit implementation** — A class can implement an interface member explicitly, making it callable only through the interface.
- **No instance fields** — Interfaces cannot contain instance fields (though static fields are allowed in C# 8+).
- **Value type support** — Structs and records can implement interfaces, enabling polymorphism across value and reference types.

---

### Prerequisites

- Understanding of C# classes, structs, methods, and properties.
- Familiarity with access modifiers (`public`, `private`, `protected`, `internal`).
- Knowledge of inheritance and polymorphism.
- Awareness of the difference between abstract classes and interfaces.
- Basic understanding of generics and type constraints (for static abstract members).

---

### Related Programming Areas

- **Polymorphism** — Interfaces enable contract-based polymorphism without a shared base class.
- **Dependency injection** — Interfaces are the primary abstraction for DI containers.
- **Testability** — Mocking frameworks rely on interfaces to create test doubles.
- **Design patterns** — Strategy, Adapter, Repository, and Plugin patterns use interfaces extensively.
- **Generic math** — Static abstract interface members enable operator-constrained generic algorithms.

---

### Core Concepts / Features

1. Interfaces (Overarching Concept)
2. `interface` Keyword
3. Interface Declaration
4. Interface Methods
5. Default Interface Methods (C# 8)
6. Static Interface Methods (C# 11)
7. Interface Inheritance
8. Multiple Interfaces
9. Explicit Interface Implementation
10. Interface Best Practices
11. Interface vs. Abstract Class

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Interfaces (Overarching Concept)

### Definitions

**Core Definition**
An interface is a reference type that defines a contract—a set of members that implementing types must provide—without specifying implementation details (unless default implementations are supplied).

**Technical Definition**
An interface defines a contract. A class or struct that implements an interface shall adhere to its contract. An interface may inherit from multiple base interfaces, and a class or struct may implement multiple interfaces. Interfaces may contain methods, properties, events, and indexers. The interface itself may provide an implementation for some or all of the function members that it declares; members without implementations are abstract and must be supplied by implementing types. Interfaces are reference types, but they can be implemented by both reference and value types.

**Beginner-Friendly Explanation**
An interface is a **contract**—a promise that a type will provide certain capabilities. It says “anything that implements me can do these things,” without saying how.

---

### Purposes

- To define a contract that multiple unrelated types can implement.
- To enable polymorphism without requiring a shared base class.
- To decouple callers from concrete implementations.
- To support multiple “inheritance” of contracts (since classes support only single inheritance).
- To provide a common API for both reference types and value types.

---

### Syntax Rules and Structure

```
[access modifier] interface IInterfaceName
{
    // member declarations
}
```

**Component breakdown**

- `access modifier` — Optional; `public` or `internal` for top-level interfaces (default `internal`).
- `interface` — The keyword.
- `IInterfaceName` — The interface name (conventionally starts with `I`).
- `{ ... }` — The member declarations.

#### Syntax Rules

- Interfaces can be top-level (in a namespace) or nested inside a type.
- Interface members without bodies are implicitly `public` and `abstract`.
- Interfaces can inherit from multiple base interfaces.
- A class or struct can implement multiple interfaces.

#### Constraints and Limitations

- Interfaces cannot contain instance fields.
- Interfaces cannot be instantiated directly.
- A class can inherit from only one base class (but multiple interfaces).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Interface and Implementation

```csharp
using System;

public interface IShape
{
    double Area();
}

public class Circle : IShape
{
    public double Radius { get; set; }
    public double Area() => Math.PI * Radius * Radius;
}

public class Square : IShape
{
    public double Side { get; set; }
    public double Area() => Side * Side;
}

class Program
{
    static void Main()
    {
        IShape[] shapes = { new Circle { Radius = 2 }, new Square { Side = 3 } };

        foreach (IShape s in shapes)
            Console.WriteLine(s.Area().ToString("F2"));
    }
}
```

**Expected Output**

```
12.57
9.00
```

**Why this output occurs**

- Each class implements `IShape` with its own `Area`; the interface reference dispatches polymorphically.

---

### Real-World Cases

- **`IDisposable`** — Defines a `Dispose` contract for resource cleanup.
- **`IEnumerable<T>`** — Defines iteration over a sequence.
- **`IComparable<T>`** — Defines ordering for sorting.

---

### References

- Interfaces — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces
- Interfaces — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces
- interface keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface

---

## 2. `interface` Keyword

### Definitions

**Core Definition**
The `interface` keyword is the syntactic marker that introduces a new interface type declaration in C#.

**Technical Definition**
The `interface` keyword is used in an `interface_declaration` to declare a new interface type. It follows optional attributes and modifiers, an optional `partial` modifier, and precedes the interface identifier, an optional type parameter list, an optional base interface list, and the interface body. The keyword distinguishes interface declarations from class and struct declarations.

**Beginner-Friendly Explanation**
The `interface` keyword is simply **how you start writing an interface**. You write `interface IMyContract { ... }`.

---

### Purposes

- To introduce a new interface type.
- To distinguish interfaces from classes and structs.
- To enable the compiler to treat the declaration as a contract.

---

### Syntax Rules and Structure

```
[attributes] [modifiers] [partial] interface IName [: baseInterfaces]
{
    // members
}
```

**Component breakdown**

- `interface` — The keyword.
- `IName` — The interface identifier.
- `: baseInterfaces` — Optional base interfaces.

#### Syntax Rules

- The keyword must be followed by a valid identifier.
- Interface names conventionally start with `I`.
- The interface body is enclosed in braces.

#### Constraints and Limitations

- The keyword cannot be used to declare classes or structs.

---

### Annotated Code Examples and Expected Outputs

#### Example: Using the `interface` Keyword

```csharp
using System;

public interface ILogger
{
    void Log(string message);
}

class Program
{
    static void Main() { }
}
```

**Expected Output**

```
(no output; compiles successfully)
```

**Why this output occurs**

- The `interface` keyword declares `ILogger` as an interface type.

---

### Real-World Cases

- **Every interface declaration** — The keyword is mandatory.

---

### References

- interface keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface
- Interfaces — C# language specification (declarations) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces

---

## 3. Interface Declaration

### Definitions

**Core Definition**
Interface declaration is the full syntax that introduces an interface, specifying its name, accessibility, base interfaces, and members.

**Technical Definition**
An `interface_declaration` consists of optional attributes, optional `interface_modifier`s (such as `public`, `internal`, `new`, `unsafe`), an optional `partial` modifier, the `interface` keyword, an identifier, an optional variant type parameter list, an optional base interface specification, optional type parameter constraint clauses, and an interface body. A top-level interface can be `public` or `internal` (default `internal`); nested interfaces can use any access modifier.

**Beginner-Friendly Explanation**
Interface declaration is **writing the whole contract**: the name, who can see it, what it inherits from, and what it requires.

---

### Purposes

- To specify the interface’s name and accessibility.
- To declare base interfaces.
- To list the required members.
- To support generic, partial, and nested interfaces.

---

### Syntax Rules and Structure

```
[attributes] [access modifier] [partial] interface IName<T> : IBase
{
    // member signatures
}
```

**Component breakdown**

- `access modifier` — `public` or `internal` for top-level; any for nested.
- `partial` — Allows splitting the interface across files.
- `IName<T>` — Generic interfaces.
- `: IBase` — Base interfaces.

#### Syntax Rules

- Top-level interfaces default to `internal`.
- Members without bodies are implicitly `public` and `abstract`.
- Interfaces can be generic, partial, and nested.

#### Constraints and Limitations

- Cannot declare instance fields.
- Cannot contain implementations unless using default interface methods (C# 8+).

---

### Annotated Code Examples and Expected Outputs

#### Example: Generic Interface Declaration

```csharp
using System;

public interface IRepository<T>
{
    T GetById(int id);
    void Save(T entity);
}

public class InMemoryRepository<T> : IRepository<T>
{
    private readonly System.Collections.Generic.Dictionary<int, T> _store = new();
    public T GetById(int id) => _store[id];
    public void Save(T entity) => _store[entity!.GetHashCode()] = entity;
}

class Program
{
    static void Main() { }
}
```

**Expected Output**

```
(no output; compiles successfully)
```

**Why this output occurs**

- `IRepository<T>` is a generic interface; `InMemoryRepository<T>` implements it.

---

### Real-World Cases

- **Repository contracts** — `IRepository<T>`.
- **Service abstractions** — `ILogger`, `IConfiguration`.

---

### References

- Interfaces — C# language specification (declarations) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces
- interface keyword — C# reference (access modifiers) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface

---

## 4. Interface Methods

### Definitions

**Core Definition**
Interface methods are member declarations within an interface that specify a method signature (name, parameters, return type) without providing an implementation (unless a default implementation is supplied).

**Technical Definition**
An interface declaration can contain method signatures. Methods without bodies are abstract and implicitly `public`; implementing types must provide an implementation. Since C# 8, interface methods can include a body, which serves as a default implementation. Since C# 11, interfaces can also declare `static abstract` and `static virtual` methods. Interface methods cannot be instance fields; they represent behavior that implementing types must support.

**Beginner-Friendly Explanation**
Interface methods are the **tasks listed in the job description**. They say “you must be able to do this,” without saying how.

---

### Purposes

- To specify required behavior.
- To define a contract for implementing types.
- To enable polymorphic dispatch through the interface.
- To support default implementations for backward-compatible evolution.

---

### Syntax Rules and Structure

```
// Abstract method (no body)
ReturnType MethodName(parameters);

// Default method (C# 8+)
ReturnType MethodName(parameters) { ... }

// Static abstract/virtual (C# 11+)
static abstract ReturnType MethodName(parameters);
static virtual ReturnType MethodName(parameters) { ... }
```

**Component breakdown**

- `ReturnType` — The method’s return type.
- `MethodName` — The method name.
- `parameters` — The parameter list.
- Body — Optional default implementation.

#### Syntax Rules

- Abstract interface methods have no body (`;`).
- Default methods have a body.
- Static abstract members require implementing types to provide the member.
- Interface methods are implicitly `public` unless they have a default implementation, in which case they are `private` by default but can have any access modifier.

#### Constraints and Limitations

- Cannot contain instance fields.
- Cannot use `sealed`, `virtual`, or `override` on abstract interface members (static virtual is a special case).

---

### Annotated Code Examples and Expected Outputs

#### Example: Abstract and Default Interface Methods

```csharp
using System;

public interface IGreeter
{
    void Greet(string name);                          // abstract
    void Farewell(string name) => Console.WriteLine($"Goodbye, {name}!"); // default
}

public class EnglishGreeter : IGreeter
{
    public void Greet(string name) => Console.WriteLine($"Hello, {name}!");
}

class Program
{
    static void Main()
    {
        IGreeter g = new EnglishGreeter();
        g.Greet("Alice");     // Hello, Alice!
        g.Farewell("Alice");  // Goodbye, Alice!
    }
}
```

**Expected Output**

```
Hello, Alice!
Goodbye, Alice!
```

**Why this output occurs**

- `Greet` is abstract and implemented by `EnglishGreeter`; `Farewell` uses the default interface implementation.

---

### Real-World Cases

- **`IDisposable.Dispose`** — Abstract method contract.
- **`IEnumerable<T>.GetEnumerator`** — Abstract method contract.
- **Default logging methods** — Adding `LogWarning` to `ILogger` with a default implementation.

---

### References

- Interface members — Interfaces (C# language specification) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces
- Default interface methods — C# 8.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#default-interface-members

---

## 5. Default Interface Methods (C# 8)

### Definitions

**Core Definition**
Default interface methods (C# 8.0+) allow an interface to provide a **default implementation** for a method or other member, so implementing types do not have to provide their own implementation unless they want to override the default behavior.

**Technical Definition**
Beginning with C# 8.0 on .NET Core 3.0, you can define an implementation when you declare a member of an interface. This feature allows an interface to provide a default implementation for classes and structs that do not provide an overriding implementation. Default interface methods are **virtual by default**, so method dispatch always chooses the implementation in the class if one exists. This enables **non-breaking evolution** of published interfaces: new members can be added with default implementations without breaking existing implementers. If you add default interface members, any `ref struct` that implements the interface must explicitly declare that member.

**Beginner-Friendly Explanation**
Default interface methods let an interface say **“here is a default way to do this, but you can override it if you want.”** This means you can add new methods to an interface without breaking all the classes that already implement it.

---

### Purposes

- To evolve published interfaces without breaking existing implementers.
- To provide shared default behavior in an interface.
- To enable mixin-like functionality.
- To reduce boilerplate in implementing types.
- To support backward-compatible API evolution.

---

### Syntax Rules and Structure

```
public interface IExample
{
    ReturnType Method() { /* default implementation */ }
}
```

**Component breakdown**

- `ReturnType Method()` — The member signature.
- `{ ... }` — The default implementation body.

#### Syntax Rules

- Default interface methods can have bodies.
- They are virtual by default; implementing classes can override them.
- Access modifiers are allowed (default is `private` for members with bodies).
- `ref struct` implementers must explicitly declare default interface members.

#### Constraints and Limitations

- Requires C# 8.0 / .NET Core 3.0 or later.
- Default implementations are not inherited by implementing classes in the same way as base class methods.
- Calling a default method requires a reference typed as the interface, not the concrete class.

---

### Annotated Code Examples and Expected Outputs

#### Example: Default Interface Method

```csharp
using System;

public interface ILogger
{
    void Log(string message);
    void LogWarning(string message) => Log($"[WARN] {message}");
}

public class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine(message);
}

class Program
{
    static void Main()
    {
        ILogger logger = new ConsoleLogger();
        logger.LogWarning("Low disk space");   // [WARN] Low disk space
    }
}
```

**Expected Output**

```
[WARN] Low disk space
```

**Why this output occurs**

- `LogWarning` has a default implementation that calls the abstract `Log` method.

---

#### Example: Overriding a Default Method

```csharp
using System;

public interface IGreeter
{
    string Greet(string name) => $"Hello, {name}";
}

public class SpanishGreeter : IGreeter
{
    public string Greet(string name) => $"Hola, {name}";
}

class Program
{
    static void Main()
    {
        IGreeter g = new SpanishGreeter();
        Console.WriteLine(g.Greet("Ana"));   // Hola, Ana
    }
}
```

**Expected Output**

```
Hola, Ana
```

**Why this output occurs**

- `SpanishGreeter` overrides the default `Greet` implementation.

---

### Real-World Cases

- **Logging abstractions** — Adding severity-specific methods with defaults.
- **Plugin interfaces** — Adding optional capabilities with default behavior.
- **API versioning** — Evolving interfaces without breaking consumers.

---

### References

- Create mixin types using default interface methods — C# - https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interface-implementation/mixins-with-default-interface-methods
- Default interface methods — C# 8.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#default-interface-members
- Interfaces — C# language specification (default implementations) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces

---

## 6. Static Interface Methods (C# 11)

### Definitions

**Core Definition**
Static interface methods (C# 11+) allow an interface to declare `static abstract` or `static virtual` members, enabling generic algorithms to rely on static methods, operators, and other static members of the implementing types.

**Technical Definition**
C# 11 and .NET 7 include **static virtual members** in interfaces. This feature enables you to define interfaces that include overloaded operators or other static members. Once you’ve defined interfaces with static members, you can use those interfaces as constraints to create generic types that use operators or other static methods. An interface can declare `static abstract` and `static virtual` members for all member types except fields. `static abstract` members require implementing types to provide the member; `static virtual` members provide a default implementation that implementing types can override. A common use case is generic math, where a generic method can be written against `INumber<TSelf>` and use operators like `+` and `/`.

**Beginner-Friendly Explanation**
Static interface methods let you write **generic code that works with any type that supports certain operators**, like addition or comparison. You can say “give me any type that can be added to itself” and write a generic algorithm once.

---

### Purposes

- To enable generic math and operator-constrained algorithms.
- To define contracts for static members.
- To support generic algorithms that rely on operators.
- To provide default static implementations that implementers can override.

---

### Syntax Rules and Structure

```
public interface IExample
{
    static abstract ReturnType Method();
    static virtual ReturnType Method() { ... }
}
```

**Component breakdown**

- `static abstract` — Implementing types must provide the member.
- `static virtual` — Provides a default; implementing types can override.
- `ReturnType Method()` — The static method signature.

#### Syntax Rules

- Available in C# 11 / .NET 7+.
- `static abstract` members must be implemented by all implementing types.
- `static virtual` members provide a default implementation.
- Cannot be fields.

#### Constraints and Limitations

- Requires C# 11 or later.
- Implementing types must provide `static abstract` members.
- Generic constraints are required to use static interface members in generic algorithms.

---

### Annotated Code Examples and Expected Outputs

#### Example: Generic Math with `INumber<T>`

```csharp
using System;
using System.Numerics;

public static class MathHelper
{
    public static T MidPoint<T>(T left, T right) where T : INumber<T>
        => (left + right) / T.CreateChecked(2);
}

class Program
{
    static void Main()
    {
        Console.WriteLine(MathHelper.MidPoint(3, 7));       // 5
        Console.WriteLine(MathHelper.MidPoint(2.5, 3.5));   // 3
    }
}
```

**Expected Output**

```
5
3
```

**Why this output occurs**

- `INumber<T>` declares static abstract members including `+`, `/`, and `CreateChecked`.
- The generic method works for any type implementing `INumber<T>`.

---

### Real-World Cases

- **Generic math** — `INumber<T>`, `IAdditionOperators<T, T, T>`.
- **Generic algorithms** — Sorting, averaging, and mathematical computations.
- **Operator contracts** — Interfaces that require specific operators.

---

### References

- Explore static virtual members in C# interfaces — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/static-virtual-interface-members
- Static abstract interface members — C# 11 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-11#static-abstract-members-in-interfaces
- Interfaces — C# language specification (static members) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces

---

## 7. Interface Inheritance

### Definitions

**Core Definition**
Interface inheritance allows an interface to inherit members from one or more base interfaces, composing a larger contract from smaller ones.

**Technical Definition**
An interface may inherit from multiple base interfaces. The derived interface inherits the members from its base interfaces, and any type that implements the derived interface must implement all members inherited from the base interfaces as well as those declared directly in the derived interface. For multiple-inheritance interfaces, ambiguities can occur when two or more unrelated base interfaces declare members with the same name or signature; explicit interface implementation can resolve such ambiguities.

**Beginner-Friendly Explanation**
Interface inheritance lets you **build bigger contracts from smaller ones**. `IAdvancedLogger` can inherit from `ILogger` and add more requirements.

---

### Purposes

- To compose contracts from smaller, focused interfaces.
- To extend an existing contract with additional members.
- To support interface-based versioning (derived interfaces for new capabilities).
- To enable multiple inheritance of contracts.

---

### Syntax Rules and Structure

```
public interface IDerived : IBase1, IBase2
{
    // additional members
}
```

**Component breakdown**

- `IDerived` — The derived interface.
- `: IBase1, IBase2` — Base interfaces (comma-separated).

#### Syntax Rules

- An interface can inherit from multiple base interfaces.
- Implementing types must implement all members from the entire inheritance chain.
- Ambiguities from multiple base interfaces can be resolved with explicit implementation.

#### Constraints and Limitations

- Cannot inherit from classes.
- Ambiguous members may require explicit implementation.

---

### Annotated Code Examples and Expected Outputs

#### Example: Interface Inheritance

```csharp
using System;

public interface IReadable { string Read(); }
public interface IWritable { void Write(string data); }

public interface IReadWrite : IReadable, IWritable { }

public class Document : IReadWrite
{
    private string _content = "";
    public string Read() => _content;
    public void Write(string data) => _content = data;
}

class Program
{
    static void Main()
    {
        var doc = new Document();
        doc.Write("Hello");
        Console.WriteLine(doc.Read());   // Hello
    }
}
```

**Expected Output**

```
Hello
```

**Why this output occurs**

- `IReadWrite` inherits `Read` and `Write`; `Document` implements both.

---

### Real-World Cases

- **`ICollection<T>`** inherits from `IEnumerable<T>`.
- **`IList<T>`** inherits from `ICollection<T>` and `IEnumerable<T>`.
- **Composite contracts** — Combining multiple small interfaces into a domain-specific contract.

---

### References

- Interfaces — C# language specification (inheritance) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces
- Interfaces — C# Programming Guide (inheritance) - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces

---

## 8. Multiple Interfaces

### Definitions

**Core Definition**
Multiple interface implementation allows a class or struct to implement more than one interface, enabling it to provide multiple capabilities or participate in multiple contracts simultaneously.

**Technical Definition**
A class or struct can implement multiple interfaces, but a class can only inherit from a single class. To implement multiple interfaces, each interface member must be implemented with the same name and signature as specified in the interface. Combining multiple interfaces in a single class involves implementing all required methods and properties from each interface. When two interfaces contain a member with the same signature, a single implicit implementation can satisfy both, or explicit implementation can provide separate implementations. This capability is important because C# does not support multiple inheritance of classes; multiple interfaces achieve a similar effect.

**Beginner-Friendly Explanation**
Multiple interfaces let a class **wear several hats**. A `Smartphone` can be a `ICamera`, a `IPhone`, and an `IMusicPlayer` all at once.

---

### Purposes

- To provide multiple capabilities in a single type.
- To achieve the effect of multiple inheritance without class inheritance complications.
- To support diverse client expectations through different interfaces.
- To enable flexible composition of contracts.

---

### Syntax Rules and Structure

```
public class MyClass : IInterface1, IInterface2, IInterface3
{
    // implement all members
}
```

**Component breakdown**

- `MyClass` — The implementing class.
- `: IInterface1, IInterface2, ...` — The interface list.

#### Syntax Rules

- A class can implement any number of interfaces.
- All interface members must be implemented.
- A base class can be combined with interfaces (`class MyClass : BaseClass, IInterface`).
- When signatures conflict, use explicit implementation.

#### Constraints and Limitations

- Cannot implement two interfaces with conflicting member signatures implicitly; explicit implementation is required.
- All members must be implemented.

---

### Annotated Code Examples and Expected Outputs

#### Example: Multiple Interfaces

```csharp
using System;

public interface IDrive { void Drive(); }
public interface IFly { void Fly(); }

public class SuperCar : IDrive, IFly
{
    public void Drive() => Console.WriteLine("Driving on the road");
    public void Fly() => Console.WriteLine("Flying in the sky");
}

class Program
{
    static void Main()
    {
        var car = new SuperCar();
        car.Drive();   // Driving on the road
        car.Fly();     // Flying in the sky
    }
}
```

**Expected Output**

```
Driving on the road
Flying in the sky
```

**Why this output occurs**

- `SuperCar` implements both interfaces and provides both behaviors.

---

#### Example: Combining Interfaces with Inheritance

```csharp
using System;

public interface IControl { void Play(); void Pause(); }
public interface ISkipControl : IControl { void SkipToNext(); }
public interface IVolumeControl : IControl { void AdjustVolume(int level); }

public class MusicPlayer : ISkipControl, IVolumeControl
{
    public void Play() => Console.WriteLine("Playing");
    public void Pause() => Console.WriteLine("Paused");
    public void SkipToNext() => Console.WriteLine("Skipping");
    public void AdjustVolume(int level) => Console.WriteLine($"Volume: {level}");
}

class Program
{
    static void Main()
    {
        var player = new MusicPlayer();
        player.Play();            // Playing
        player.SkipToNext();      // Skipping
        player.AdjustVolume(8);   // Volume: 8
    }
}
```

**Expected Output**

```
Playing
Skipping
Volume: 8
```

**Why this output occurs**

- `MusicPlayer` implements `ISkipControl` and `IVolumeControl`, which both inherit `IControl`.

---

### Real-World Cases

- **`List<T>`** implements `IList<T>`, `ICollection<T>`, `IEnumerable<T>`, and more.
- **`Stream`** implements `IDisposable`, `IAsyncDisposable`.
- **Domain models** — Implementing `IValidatable`, `IAuditable`, and `IEntity`.

---

### References

- Use and combine multiple interfaces — Microsoft Learn training - https://learn.microsoft.com/en-us/training/modules/design-resilient-code-interfaces/4-use-combine-multiple-interfaces
- Interfaces — C# language specification (multiple interfaces) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces

---

## 9. Explicit Interface Implementation

### Definitions

**Core Definition**
Explicit interface implementation is a technique where a class member is implemented with the interface name as a prefix, making it callable **only through the interface**, not through the class instance.

**Technical Definition**
When a class implements two interfaces that contain a member with the same signature, implementing that member on the class causes both interfaces to use that member as their implementation. To call a different implementation depending on which interface is in use, you can implement an interface member explicitly. An explicit interface implementation is a class member that is only called through the specified interface; it is named by prefixing the member with the interface name and a period. Explicit implementations do not have access modifiers and are not accessible as members of the implementing type. Explicit implementation is also used to resolve cases where two interfaces declare different kinds of members with the same name (e.g., a property and a method).

**Beginner-Friendly Explanation**
Explicit implementation is **“only when you ask through this specific interface.”** The method is hidden from the class itself but available when the object is used as the interface.

---

### Purposes

- To resolve conflicts when multiple interfaces declare members with the same signature.
- To hide interface-specific implementations from the class’s public API.
- To provide different behavior for the same member name across interfaces.
- To implement interfaces whose members would otherwise conflict with class members.

---

### Syntax Rules and Structure

```
ReturnType IInterfaceName.MemberName(parameters)
{
    // implementation
}
```

**Component breakdown**

- `IInterfaceName.` — The interface prefix.
- `MemberName` — The member being implemented.
- No access modifier — Explicit implementations are private to the interface.

#### Syntax Rules

- The interface name and member name are separated by a period.
- No access modifier is allowed.
- The member is only accessible through the interface.
- Can be used for methods, properties, events, and indexers.

#### Constraints and Limitations

- Cannot be called directly on the class instance.
- Cannot be overridden in derived classes (unless the derived class re-implements the interface).
- No access modifier.

---

### Annotated Code Examples and Expected Outputs

#### Example: Resolving Conflicting Signatures

```csharp
using System;

public interface IControl { void Paint(); }
public interface ISurface { void Paint(); }

public class SampleClass : IControl, ISurface
{
    void IControl.Paint() => Console.WriteLine("IControl.Paint");
    void ISurface.Paint() => Console.WriteLine("ISurface.Paint");
}

class Program
{
    static void Main()
    {
        var sample = new SampleClass();
        IControl control = sample;
        ISurface surface = sample;

        control.Paint();   // IControl.Paint
        surface.Paint();   // ISurface.Paint
        // sample.Paint(); // Compile error: not accessible directly
    }
}
```

**Expected Output**

```
IControl.Paint
ISurface.Paint
```

**Why this output occurs**

- Each interface has its own explicit implementation; the class itself does not expose a public `Paint` method.

---

#### Example: Resolving Property/Method Conflict

```csharp
using System;

interface ILeft { int P { get; } }
interface IRight { int P(); }

class Middle : ILeft, IRight
{
    public int P() => 0;          // implicit for IRight
    int ILeft.P => 0;              // explicit for ILeft
}

class Program
{
    static void Main()
    {
        var m = new Middle();
        ILeft left = m;
        IRight right = m;

        Console.WriteLine(left.P);    // 0
        Console.WriteLine(right.P()); // 0
    }
}
```

**Expected Output**

```
0
0
```

**Why this output occurs**

- The property and method with the same name are resolved using explicit implementation.

---

### Real-World Cases

- **`IEnumerable<T>` and `IEnumerable`** — `GetEnumerator` is implemented explicitly in many collection types.
- **Custom collections** — Implementing multiple enumeration interfaces.
- **Framework interfaces** — Resolving `Dispose` conflicts.

---

### References

- Explicit Interface Implementation — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/explicit-interface-implementation
- Interfaces — C# language specification (explicit implementation) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces

---

## 10. Interface Best Practices

### Definitions

**Core Definition**
Interface best practices are the design guidelines that ensure interfaces are **focused, stable, and maintainable**, and that they are used appropriately instead of alternative mechanisms.

**Technical Definition**
Best practices for interfaces derive from the Framework Design Guidelines and the .NET design conventions. Key recommendations include: **define an interface if you need a common API supported by a set of types that includes value types**; **consider an interface if you need to support functionality on types that already inherit from another type**; **avoid marker interfaces (interfaces with no members)**; **do not add members to an interface that has previously shipped**; **use the `I` prefix for interface names**; **keep interfaces small and focused (Interface Segregation Principle)**; **provide at least one implementation and one consuming API for each interface**; and **prefer explicit implementation to resolve conflicts or hide implementation details**.

**Beginner-Friendly Explanation**
Best practices keep interfaces clean and safe: name them with `I`, keep them small, do not change published interfaces, and provide implementations so you know the design works.

---

### Purposes

- To ensure interfaces are stable and evolvable.
- To keep contracts focused and understandable.
- To avoid breaking changes in published APIs.
- To improve discoverability and consistency.
- To align with .NET conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Use the "I" prefix
public interface IRepository { }

// Keep interfaces small and focused
public interface IReader { string Read(); }
public interface IWriter { void Write(string data); }

// Avoid marker interfaces
// Use attributes instead for markers

// Do not add members to shipped interfaces
// Create a new derived interface instead

// Provide an implementation and a consumer
public class FileRepository : IRepository { ... }
public class Service { public Service(IRepository repo) { ... } }
```

**Component breakdown**

- **`I` prefix** — Distinguishes interfaces from classes.
- **Small, focused interfaces** — Follow the Interface Segregation Principle.
- **No marker interfaces** — Use attributes for marking.
- **Stable published interfaces** — Add new interfaces instead of modifying existing ones.
- **Validation** — Provide implementations and consumers to prove the design.

#### Rules and Constraints

- **Do** name interfaces with the `I` prefix.
- **Do** define interfaces when value types or unrelated types need a common API.
- **Do** provide at least one implementation and one consuming API.
- **Do not** use marker interfaces; use attributes.
- **Do not** add members to published interfaces.
- **Do not** put fields in interfaces.

---

### Annotated Code Examples and Expected Outputs

#### Example: Focused Interfaces

```csharp
using System;

public interface IReader { string Read(); }
public interface IWriter { void Write(string data); }

public class FileHandler : IReader, IWriter
{
    public string Read() => "data";
    public void Write(string data) { /* write */ }
}

class Program
{
    static void Main()
    {
        var handler = new FileHandler();
        Console.WriteLine(handler.Read());   // data
    }
}
```

**Expected Output**

```
data
```

**Why this output occurs**

- `FileHandler` implements two focused interfaces instead of one fat interface.

---

### Real-World Cases

- **.NET BCL** — `IDisposable`, `IComparable<T>`, `IEnumerable<T>`.
- **Application architecture** — `IRepository`, `ILogger`, `IUnitOfWork`.

---

### References

- Interface Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/interface
- C# Interface Best Practices — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/5-c-sharp-interfac-sharpes/11-c-sharp-interface-best-practices.mdx
- Interfaces — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces

---

## 11. Interface vs. Abstract Class

### Definitions

**Core Definition**
Interfaces and abstract classes are both abstraction mechanisms, but they differ in **contract purity**, **inheritance semantics**, **implementation sharing**, **versioning flexibility**, and **value-type support**. Choosing between them is a fundamental design decision.

**Technical Definition**
An **abstract class** can provide both abstract members (no implementation) and concrete members (shared implementation), can declare fields and constructors, and supports single inheritance. An **interface** defines a pure contract (historically) with no implementation, supports multiple inheritance of contracts, cannot contain instance fields, and (since C# 8) can provide default implementations. The Framework Design Guidelines recommend **favoring abstract classes in most cases** because they version better—you can add a new virtual method to an abstract class without breaking derived classes, whereas adding a member to a shipped interface is a breaking change (unless a default implementation is provided). However, interfaces are **required** when you need a common API supported by value types (structs cannot inherit from classes) or when a type must implement functionality while already inheriting from another class.

**Beginner-Friendly Explanation**
An **abstract class** is a **partial blueprint**—it can have some completed parts and some parts you must fill in. An **interface** is a **pure contract**—it only says what must be done, not how. Abstract classes are better when you want to share code; interfaces are better when you need maximum flexibility or need to work with structs.

---

### Purposes

- To choose the right abstraction for the scenario.
- To balance code sharing against flexibility.
- To account for versioning and evolution.
- To support value types and multiple contracts.

---

### Syntax Rules and Structure (Comparison)

| Aspect | Abstract Class | Interface |
|---|---|---|
| **Implementation sharing** | Yes (concrete members) | Default methods (C# 8+), but historically no |
| **Inheritance** | Single class inheritance | Multiple interface implementation |
| **Fields** | Can have instance fields | Cannot have instance fields |
| **Constructors** | Can have constructors | Cannot have constructors |
| **Value types** | Structs cannot inherit | Structs can implement |
| **Versioning** | Easier (add virtual members) | Harder (breaking change unless default methods) |
| **Access modifiers** | Full range | Limited (public for abstract members; any for default members) |

**Component breakdown**

- **Shared code** — Abstract classes.
- **Multiple contracts** — Interfaces.
- **Value types** — Interfaces only.
- **Versioning** — Abstract classes generally easier.

#### Syntax Rules

- Use abstract classes when you control the hierarchy and need shared implementation.
- Use interfaces for decoupled contracts, multiple inheritance, and value types.
- Both can be combined: an abstract class can implement interfaces.

#### Constraints and Limitations

- Cannot inherit from multiple abstract classes.
- Interfaces cannot contain instance state.

---

### Annotated Code Examples and Expected Outputs

#### Example: Abstract Class for Shared Code

```csharp
using System;

public abstract class Shape
{
    public abstract double Area();
    public virtual string Describe() => $"Area = {Area():F2}";
}

public class Circle : Shape
{
    public double Radius { get; set; }
    public override double Area() => Math.PI * Radius * Radius;
}

class Program
{
    static void Main()
    {
        Shape s = new Circle { Radius = 2 };
        Console.WriteLine(s.Describe());   // Area = 12.57
    }
}
```

**Expected Output**

```
Area = 12.57
```

**Why this output occurs**

- `Shape` provides shared `Describe` behavior; `Circle` implements `Area`.

---

#### Example: Interface for Multiple Contracts and Structs

```csharp
using System;

public interface IIdentifiable { int Id { get; } }

public struct Point : IIdentifiable
{
    public int Id { get; set; }
    public int X { get; set; }
    public int Y { get; set; }
}

class Program
{
    static void Main()
    {
        IIdentifiable p = new Point { Id = 1, X = 2, Y = 3 };
        Console.WriteLine(p.Id);   // 1
    }
}
```

**Expected Output**

```
1
```

**Why this output occurs**

- `Point` (a struct) implements `IIdentifiable`; structs cannot inherit from abstract classes.

---

### Real-World Cases

- **Framework design** — Abstract classes for base frameworks; interfaces for cross-cutting contracts.
- **Application architecture** — Interfaces for repositories and services; abstract classes for shared domain logic.
- **Value types** — Only interfaces can provide polymorphism for structs.

---

### References

- Abstract Class or Interface I — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/peaceofmind/abstract-class-or-interface-i
- Abstract class vs Interface in our days — Microsoft Q&A - https://learn.microsoft.com/en-us/answers/questions/1062583/abstract-class-vs-interface-in-our-days
- Interface Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/interface
- Abstract class vs. interface — Framework Design Guidelines (GitHub discussion) - https://github.com/dotnet/runtime/discussions/102616

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Default interface methods | Introduced in **C# 8.0**; version-specific. |
| Static abstract/virtual interface members | Introduced in **C# 11**; version-specific. |
| Marker interfaces (no members) | **Discouraged**; use attributes instead. |
| Adding members to shipped interfaces | **Breaking change** unless a default implementation is provided. |
| Interfaces with instance fields | **Not allowed**; use properties instead. |
| Explicit interface implementation | Available since C# 1.0; stable. |
| `ref struct` implementing interfaces with default members | **Must explicitly declare** default members. |

---

## References (Consolidated)

- Interfaces — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces
- Interfaces — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces
- interface keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface
- Default interface methods — C# 8.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#default-interface-members
- Create mixin types using default interface methods — C# - https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interface-implementation/mixins-with-default-interface-methods
- Explore static virtual members in C# interfaces — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/static-virtual-interface-members
- Static abstract interface members — C# 11 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-11#static-abstract-members-in-interfaces
- Explicit Interface Implementation — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/explicit-interface-implementation
- Use and combine multiple interfaces — Microsoft Learn training - https://learn.microsoft.com/en-us/training/modules/design-resilient-code-interfaces/4-use-combine-multiple-interfaces
- Interface Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/interface
- Abstract Class or Interface I — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/peaceofmind/abstract-class-or-interface-i
- Abstract class vs Interface in our days — Microsoft Q&A - https://learn.microsoft.com/en-us/answers/questions/1062583/abstract-class-vs-interface-in-our-days
- Abstract class vs. interface — Framework Design Guidelines (GitHub discussion) - https://github.com/dotnet/runtime/discussions/102616
- C# Interface Best Practices — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/5-c-sharp-interfac-sharpes/11-c-sharp-interface-best-practices.mdx