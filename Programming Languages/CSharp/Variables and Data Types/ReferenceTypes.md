# C# Reference Types — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A reference type in C# is a type whose variable stores a **reference** (a memory address) to its data rather than the data itself. The actual object resides on the managed heap, and multiple variables can refer to the same object. Operations performed through one variable can therefore affect the object observed through another variable referencing the same instance.

**Technical Definition**
The C# type system divides all types into two main categories: **reference types** and **value types**. Variables of value types directly contain their data, whereas variables of reference types store references to their data, the latter being known as **objects**. With reference types, it is possible for two variables to reference the same object, and thus possible for operations on one variable to affect the object referenced by the other variable. A reference type is a class type, an interface type, an array type, a delegate type, or the `dynamic` type. The special value `null` is compatible with all reference types and indicates the absence of an instance.

**Beginner-Friendly Explanation**
Think of a reference-type variable as a **business card** rather than a sticky note. The business card does not contain the company’s data; it contains the **address** where the data lives. If you photocopy the business card and give it to a friend, both cards point to the same company. If your friend visits the company and changes something, you will see that change too, because you are both looking at the same place. Reference types are ideal for large, complex objects that should be shared rather than copied.

---

### Key Characteristics

- **Reference semantics** — Variables hold references, not the data itself; two variables can point to the same object.
- **Heap allocation** — Reference-type instances are allocated on the managed heap and reclaimed by the garbage collector.
- **`null` compatibility** — A reference-type variable can hold `null`, meaning it points to no object.
- **Unified type system** — Every type in C# directly or indirectly derives from `System.Object`; reference types are treated as objects simply by viewing their values as type `object`.
- **Five declaration keywords** — `class`, `interface`, `delegate`, `record`, and `dynamic` are used to declare reference types.
- **Built-in reference types** — `object`, `string`, and arrays are built-in reference types with special compiler support.

---

### Prerequisites

- Basic understanding of C# program structure (`class`, `Main`, namespaces).
- Familiarity with the distinction between value types and reference types.
- Awareness of the stack and heap as memory regions.
- Knowledge of `System.Object` as the root of the type hierarchy.
- Basic understanding of garbage collection in .NET.

---

### Related Programming Areas

- **Memory management** — Heap allocation, garbage collection, and object lifetime are central to reference types.
- **Object-oriented programming** — Classes, interfaces, inheritance, and polymorphism are built on reference types.
- **Functional programming** — Delegates, records, and immutability patterns rely on reference-type design.
- **Interoperability** — `dynamic` is heavily used in COM interop and dynamic-language scenarios.
- **API design** — Choosing between `class` and `struct` is a fundamental design decision, and reference-type best practices guide public API design.

---

### Core Concepts / Features

1. Reference types (overarching concept)
2. Heap allocation
3. `class`
4. `interface`
5. `delegate`
6. `record`
7. `object`
8. `string`
9. `array`
10. `dynamic`
11. Reference equality
12. Object equality
13. `ReferenceEquals()`
14. `Equals()`
15. `==` operator
16. Reference type best practices

Each concept below follows the same uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Reference Types (Overarching Concept)

### Definitions

**Core Definition**
A reference type is a type whose instances are accessed through references, with the actual data stored as an object on the managed heap.

**Technical Definition**
Reference types include class types, interface types, array types, delegate types, and the `dynamic` type. A reference-type value is a reference to an instance of the type, the latter known as an object. Reference types support inheritance (class types), and all reference types derive directly or indirectly from `System.Object`. Unlike value types, reference-type variables can be `null`.

**Beginner-Friendly Explanation**
A reference type is like a **remote control** for a TV. The remote (variable) does not contain the TV (data); it sends signals to the TV, which lives somewhere else (the heap). You can have multiple remotes controlling the same TV. If one remote changes the channel, the other remote sees the new channel too.

---

### Purposes

- To share data efficiently across multiple parts of a program without copying.
- To model complex objects with identity and behaviour.
- To support inheritance and polymorphism.
- To represent large data structures (objects, arrays) that would be expensive to copy.
- To enable `null` as a representation of “no object.”

---

### Syntax Rules and Structure

#### General Syntax

```
// Class type
class   ClassName   { ... }

// Interface type
interface   InterfaceName   { ... }

// Delegate type
delegate   returnType   DelegateName(parameters);

// Record type
record   RecordName(parameters);

// Array type
type[]   arrayName   =   new   type[length];

// Dynamic type
dynamic   identifier   =   expression;
```

**Component breakdown**

- `class` — Declares a reference type that supports inheritance.
- `interface` — Declares a contract that classes, records, and structs can implement.
- `delegate` — Declares a type that represents references to methods with a specific signature.
- `record` — Declares a reference type with built-in value-based equality.
- `type[]` — Declares an array reference type.
- `dynamic` — Declares a variable whose operations are bound at runtime.

#### Syntax Rules

- Reference-type variables can be `null`.
- Assignment copies the reference, not the object.
- Two variables can reference the same object.
- Reference types derive from `System.Object` (directly or indirectly).
- Reference types are allocated on the managed heap.

#### Constraints and Limitations

- Reference-type variables require a reference (typically 4 or 8 bytes on 32-bit or 64-bit systems).
- Heap allocation and garbage collection introduce overhead compared to stack allocation.
- Dereferencing a `null` reference causes `NullReferenceException`.
- Reference types cannot be used as `const` values (except `string`).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Shared Reference Semantics

```csharp
using System;

class Person
{
    public string Name;
}

class Program
{
    static void Main()
    {
        Person p1 = new Person { Name = "Alice" };
        Person p2 = p1;                    // copy the reference

        p2.Name = "Bob";

        Console.WriteLine(p1.Name);        // Bob (same object)
        Console.WriteLine(p2.Name);        // Bob
    }
}
```

**Expected Output**

```
Bob
Bob
```

**Why this output occurs**

- `p1` and `p2` are reference-type variables pointing to the **same** `Person` object.
- Changing `p2.Name` changes the shared object, so `p1.Name` also reflects the change.

---

#### Example 2: `null` Reference

```csharp
using System;

class Program
{
    static void Main()
    {
        Person p = null;

        Console.WriteLine(p == null);       // True
        // Console.WriteLine(p.Name);       // NullReferenceException
    }
}

class Person { public string Name; }
```

**Expected Output**

```
True
```

**Why this output occurs**

- A reference-type variable can hold `null`, meaning it refers to no object.
- Accessing a member on a `null` reference throws `NullReferenceException`.

---

### Real-World Cases

- **Domain models** — `Customer`, `Order`, `Product` classes represent entities with identity and behaviour.
- **Service layers** — Interfaces define contracts, and classes implement them.
- **Event systems** — Delegates encapsulate event handlers.
- **Data transfer objects** — Records provide concise, value-equal data containers.

---

### References

- Reference types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types
- Types — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types
- Built-in reference types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types

---

## 2. Heap Allocation

### Definitions

**Core Definition**
Heap allocation is the process of reserving memory for an object on the managed heap, a region of memory managed by the garbage collector.

**Technical Definition**
Reference-type instances (objects) are allocated on the managed heap. The heap is a large region of memory where objects are created using the `new` operator. Unlike the stack, the heap is not organized in a strict LIFO order; objects can be allocated and reclaimed in a non-deterministic order by the garbage collector. Memory is not returned to the heap when a method finishes; it is reclaimed only when the garbage collector determines that the object is no longer reachable.

**Beginner-Friendly Explanation**
The heap is like a **large warehouse** where objects are stored. When you create an object with `new`, the runtime finds space in the warehouse and gives you a claim ticket (the reference). The warehouse is cleaned up periodically by the garbage collector, which removes objects that nobody is using anymore.

---

### Purposes

- To provide flexible, long-lived storage for objects whose lifetime is not tied to a single method call.
- To enable sharing of objects across methods and classes.
- To support dynamic memory allocation for variable-sized data (arrays, collections).
- To decouple object lifetime from the call stack.
- To allow objects to be referenced from multiple locations.

---

### Syntax Rules and Structure

Heap allocation occurs implicitly when you create an instance of a reference type:

```
// Object creation expression triggers heap allocation
ClassName   obj   =   new   ClassName()   ;

// Array creation expression
type[]   arr   =   new   type[length]   ;

// Implicit allocation in collection initializers
var   list   =   new   List<int>   {   1, 2, 3   };
```

**Component breakdown**

- `new ClassName()` — Allocates memory for a new object on the heap and returns a reference.
- `new type[length]` — Allocates memory for an array on the heap.
- Collection initializers — Implicitly allocate the collection object.

#### Syntax Rules

- Reference-type objects are always allocated on the managed heap.
- The `new` operator returns a reference to the newly allocated object.
- The garbage collector reclaims memory for objects that are no longer reachable.
- Boxed value types are also allocated on the heap.

#### Constraints and Limitations

- Heap allocation is slower than stack allocation.
- Garbage collection introduces pauses and CPU overhead.
- Heap fragmentation can occur over time.
- Excessive allocation can lead to memory pressure and frequent garbage collection.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Heap Allocation of an Object

```csharp
using System;

class Data
{
    public int Value;
}

class Program
{
    static Data CreateData()
    {
        Data d = new Data { Value = 42 };   // allocated on the heap
        return d;                            // reference is returned
    }   // stack frame is popped, but the Data object remains on the heap

    static void Main()
    {
        Data result = CreateData();
        Console.WriteLine(result.Value);     // 42
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- `new Data` allocates the object on the heap.
- The reference is returned and remains valid after `CreateData` returns.
- The garbage collector will reclaim the object later when it is no longer reachable.

---

#### Example 2: Array Allocation on the Heap

```csharp
using System;

class Program
{
    static void Main()
    {
        int[] numbers = new int[3] { 10, 20, 30 };
        Console.WriteLine(numbers[1]);   // 20
    }
}
```

**Expected Output**

```
20
```

**Why this output occurs**

- The `int[]` array is a reference type; the array object is allocated on the heap.
- The elements are stored within the heap-allocated array.

---

### Real-World Cases

- **Caching** — Storing computed results in heap objects for reuse.
- **Collections** — `List<T>`, `Dictionary<TKey,TValue>`, and other collections live on the heap.
- **Long-lived services** — Singleton services and repositories are heap-allocated.
- **Data structures** — Trees, graphs, and linked lists use heap-allocated nodes.

---

### References

- Value and Reference Types — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2008/4d43ts61(v=vs.90)
- Memory allocation — .NET docs - https://raw.githubusercontent.com/dotnet/docs/main/docs/csharp/language-reference/language-specification/types
- Advanced Memory Management — Developer Knowledge Base - https://mintlify.wiki

---

## 3. `class`

### Definitions

**Core Definition**
A `class` is a reference type that serves as a blueprint for creating objects, encapsulating data (fields) and behaviour (methods).

**Technical Definition**
A class type defines a data structure that contains data members (constants and fields), function members (methods, properties, events, indexers, operators, instance constructors, finalizers, and static constructors), and nested types. Class types support inheritance, a mechanism whereby derived classes can extend and specialize base classes. Instances of class types are created using object creation expressions.

**Beginner-Friendly Explanation**
A class is a **cookie cutter**, and objects are the **cookies**. The class defines the shape (fields, properties, methods), and each object created from the class is a separate cookie with its own data. Classes are reference types, so variables hold references to the cookies, not the cookies themselves.

---

### Purposes

- To define custom types that encapsulate data and behaviour.
- To support inheritance and polymorphism through class hierarchies.
- To model real-world entities with identity and mutable state.
- To provide a unit of abstraction and encapsulation.
- To enable code reuse through inheritance and composition.

---

### Syntax Rules and Structure

```
[accessModifiers]   [modifiers]   class   ClassName   [: baseClass, interfaces]
{
    // fields, properties, methods, constructors, events, indexers, nested types
}
```

**Component breakdown**

- `class` — Keyword introducing a reference type.
- `ClassName` — The type name (PascalCase by convention).
- `: baseClass` — Optional base class (only one base class).
- `: interfaces` — Optional interface implementations (multiple allowed).
- Body — Contains members: fields, properties, methods, constructors, events, indexers, nested types.

#### Syntax Rules

- Classes are reference types.
- A class can inherit from one base class and implement multiple interfaces.
- Classes can be `abstract`, `sealed`, `static`, `partial`, or `record`.
- Instance constructors initialize new objects.
- Classes support `this` and `base` keywords.

#### Constraints and Limitations

- Only single inheritance for classes (one base class).
- `static` classes cannot be instantiated or inherited from.
- `sealed` classes cannot be inherited from.
- Classes are heap-allocated; excessive allocation can cause GC pressure.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Class with Properties and Methods

```csharp
using System;

public class BankAccount
{
    public string Owner { get; }
    public decimal Balance { get; private set; }

    public BankAccount(string owner, decimal initialBalance)
    {
        Owner = owner;
        Balance = initialBalance;
    }

    public void Deposit(decimal amount)
    {
        Balance += amount;
    }

    public override string ToString() => $"{Owner}: {Balance:C}";
}

class Program
{
    static void Main()
    {
        var account = new BankAccount("Alice", 100m);
        account.Deposit(50m);
        Console.WriteLine(account);   // Alice: $150.00
    }
}
```

**Expected Output**

```
Alice: $150.00
```

**Why this output occurs**

- `BankAccount` is a class (reference type); `account` holds a reference to the object.
- `Deposit` mutates the object’s state.
- `ToString` is overridden to provide a readable representation.

---

#### Example 2: Inheritance and Polymorphism

```csharp
using System;

public class Shape
{
    public virtual double Area() => 0;
}

public class Circle : Shape
{
    public double Radius { get; }
    public Circle(double radius) => Radius = radius;
    public override double Area() => Math.PI * Radius * Radius;
}

class Program
{
    static void Main()
    {
        Shape s = new Circle(2);
        Console.WriteLine($"{s.Area():F2}");   // 12.57
    }
}
```

**Expected Output**

```
12.57
```

**Why this output occurs**

- `Circle` inherits from `Shape` and overrides `Area`.
- The variable `s` is of type `Shape` but refers to a `Circle` object.
- Polymorphism dispatches the call to `Circle.Area` at runtime.

---

### Real-World Cases

- **Domain modelling** — `Customer`, `Order`, `Invoice` classes represent business entities.
- **UI frameworks** — `Form`, `Control`, `Button` classes form hierarchies.
- **Service implementations** — Classes implementing service interfaces.
- **Data access** — Repository classes encapsulating database operations.

---

### References

- Classes — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- Classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes
- class keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/class

---

## 4. `interface`

### Definitions

**Core Definition**
An `interface` is a reference type that defines a contract—a set of members (methods, properties, events, indexers)—that implementing classes, records, or structs must provide.

**Technical Definition**
An interface defines a contract. Any class, record, or struct that implements that contract must provide an implementation of the members defined in the interface. Interfaces can declare static abstract or static virtual members, and can provide default implementations for members. An interface is a reference type, but it is not a type in the same sense as a class; it is a contract for behaviour. Interfaces support multiple inheritance of contracts (a class can implement multiple interfaces).

**Beginner-Friendly Explanation**
An interface is like a **job description**. It lists the tasks (methods and properties) that a role (class) must be able to perform, but it does not say how to perform them. Any class that “applies for the job” must implement all the listed tasks. Different classes can implement the same interface in different ways.

---

### Purposes

- To define a contract that multiple unrelated classes can implement.
- To enable polymorphism without requiring a common base class.
- To support dependency injection and testability through abstractions.
- To allow default implementations of some members (C# 8+).
- To enable multiple “inheritance” of contracts.

---

### Syntax Rules and Structure

```
[accessModifiers]   interface   InterfaceName   [: baseInterfaces]
{
    // method signatures, property signatures, event declarations, indexer signatures
    // optional default implementations (C# 8+)
}
```

**Component breakdown**

- `interface` — Keyword introducing a contract.
- `InterfaceName` — The interface name (conventionally starts with `I`).
- `: baseInterfaces` — Optional inheritance from other interfaces.
- Body — Contains member signatures; implementations are provided by implementing types.

#### Syntax Rules

- Interfaces are reference types.
- Interface members are implicitly `public` (before C# 8); C# 8+ allows access modifiers and default implementations.
- A class can implement multiple interfaces.
- Interfaces can inherit from other interfaces.
- Implementing types must provide implementations for all non-default members.

#### Constraints and Limitations

- Interfaces cannot contain instance fields (though static fields are allowed in C# 8+).
- Interfaces cannot be instantiated directly.
- Default interface members require C# 8.0+ and .NET Core 3.0+.
- `ref struct` types must explicitly declare default interface members if they implement an interface with such members.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Interface Contract and Implementation

```csharp
using System;

public interface ILogger
{
    void Log(string message);
}

public class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine($"[LOG] {message}");
}

class Program
{
    static void Main()
    {
        ILogger logger = new ConsoleLogger();
        logger.Log("Application started");
    }
}
```

**Expected Output**

```
[LOG] Application started
```

**Why this output occurs**

- `ILogger` defines the contract `Log`.
- `ConsoleLogger` implements the contract.
- The variable `logger` is of type `ILogger` and calls the implementation through the interface.

---

#### Example 2: Multiple Interface Implementation

```csharp
using System;

public interface IReadable { string Read(); }
public interface IWritable { void Write(string data); }

public class Document : IReadable, IWritable
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

- `Document` implements both `IReadable` and `IWritable`.
- It provides implementations for both contracts.

---

### Real-World Cases

- **Dependency injection** — `IService`, `IRepository`, `ILogger` abstractions.
- **Plugin architectures** — `IPlugin` contracts for extensibility.
- **Testing** — Mocking interfaces with frameworks like Moq.
- **Fluent APIs** — Interfaces for builders and configuration.

---

### References

- interface keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface
- Interfaces — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces
- Interfaces — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces

---

## 5. `delegate`

### Definitions

**Core Definition**
A `delegate` is a reference type that represents a reference to a method with a specific parameter list and return type.

**Technical Definition**
A delegate is a type that represents references to methods with a particular parameter list and return type. When you instantiate a delegate, you can associate its instance with any method with a compatible signature and return type. Each delegate type is a reference type derived from `System.Delegate`. Delegates are used to pass methods as arguments to other methods, define callback methods, and implement event handling.

**Beginner-Friendly Explanation**
A delegate is like a **phone number for a method**. You can store the number in a variable, pass it to someone else, and call it later. The delegate knows which method to call because it holds a reference to it.

---

### Purposes

- To pass methods as arguments to other methods.
- To define callback mechanisms.
- To implement event handling (events are based on delegates).
- To support functional programming patterns (e.g., LINQ operators).
- To enable loose coupling between components.

---

### Syntax Rules and Structure

```
// Delegate declaration
[accessModifiers]   delegate   returnType   DelegateName(parameters);

// Delegate instantiation
DelegateName   d   =   new   DelegateName(MethodName);

// Method group conversion (shorthand)
DelegateName   d2   =   MethodName;

// Lambda expression
DelegateName   d3   =   (x)   =>   x   *   2;
```

**Component breakdown**

- `delegate` — Keyword introducing a delegate type.
- `returnType` — The return type of methods the delegate can reference.
- `DelegateName` — The delegate type name.
- `parameters` — The parameter list.
- `new DelegateName(MethodName)` — Creates a delegate instance pointing to a method.
- Method group conversion — Shorthand for creating a delegate from a method.
- Lambda expression — Inline anonymous method.

#### Syntax Rules

- Delegates are reference types derived from `System.Delegate`.
- Delegates can be combined using `+` and `-` (multicast delegates).
- Delegates can reference static or instance methods.
- Delegates are immutable; combining creates a new delegate.
- Covariance and contravariance are supported for delegate types (C# 2.0+).

#### Constraints and Limitations

- Delegates can reference methods with compatible signatures only.
- Multicast delegates return only the last method’s return value.
- Delegates hold strong references to their target objects, which can affect garbage collection.
- `Delegate.Combine` and `Delegate.Remove` are used internally for multicast operations.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Delegate Usage

```csharp
using System;

public delegate int MathOperation(int a, int b);

class Program
{
    static int Add(int x, int y) => x + y;
    static int Multiply(int x, int y) => x * y;

    static void Main()
    {
        MathOperation op = Add;
        Console.WriteLine(op(3, 4));    // 7

        op = Multiply;
        Console.WriteLine(op(3, 4));    // 12
    }
}
```

**Expected Output**

```
7
12
```

**Why this output occurs**

- The delegate variable `op` can reference different methods with the same signature.
- Calling `op` invokes the currently referenced method.

---

#### Example 2: Multicast Delegate

```csharp
using System;

public delegate void Notifier(string message);

class Program
{
    static void Email(string m) => Console.WriteLine($"Email: {m}");
    static void SMS(string m) => Console.WriteLine($"SMS: {m}");

    static void Main()
    {
        Notifier n = Email;
        n += SMS;
        n("Hello");
    }
}
```

**Expected Output**

```
Email: Hello
SMS: Hello
```

**Why this output occurs**

- The `+=` operator combines delegates into a multicast delegate.
- Invoking the multicast delegate calls all referenced methods in order.

---

### Real-World Cases

- **Event handling** — `EventHandler`, `Action`, `Func` delegates.
- **LINQ** — `Func<T, bool>` for `Where`, `Func<T, TResult>` for `Select`.
- **Async programming** — `Task` continuations use delegates.
- **Strategy pattern** — Injecting algorithms as delegates.

---

### References

- Delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/
- delegate keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- Delegates — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/delegates

---

## 6. `record`

### Definitions

**Core Definition**
A `record` is a reference type (or value type, when declared as `record struct`) that provides built-in functionality for encapsulating data, including value-based equality.

**Technical Definition**
The `record` modifier provides built-in functionality for encapsulating data. The `record class` and `record` syntax define reference types; the `record struct` syntax defines a value type. A record is a class that provides value semantics: two record instances are equal if their types match and all their field and property values are equal. Records support `with` expressions for non-destructive mutation, positional syntax for concise declarations, and built-in `ToString`, `Equals`, and `GetHashCode` implementations.

**Beginner-Friendly Explanation**
A record is a special kind of class designed for holding data. Unlike a regular class, two records with the same data are considered equal. You can also create a modified copy of a record using the `with` expression without changing the original. Think of it as a **fill-in-the-blank form** where two forms with identical answers are considered the same.

---

### Purposes

- To model data-centric types with concise syntax.
- To provide value-based equality for reference types.
- To enable non-destructive mutation via `with` expressions.
- To reduce boilerplate for data-carrying classes.
- To support pattern matching with positional deconstruction.

---

### Syntax Rules and Structure

```
// Positional record (reference type)
public   record   Person(string FirstName, string LastName);

// Record with additional members
public   record   Person(string FirstName, string LastName)
{
    public string FullName => $"{FirstName} {LastName}";
}

// Record struct (value type)
public   record   struct   Point(int X, int Y);
```

**Component breakdown**

- `record` — Keyword introducing a data-centric type.
- `(parameters)` — Positional parameters that generate public init-only properties.
- `record struct` — Declares a value-type record.
- Additional members — Properties, methods, etc., can be added.

#### Syntax Rules

- `record` or `record class` declares a reference type.
- `record struct` declares a value type.
- Records provide compiler-generated `Equals`, `GetHashCode`, and `ToString`.
- `with` expressions create modified copies.
- Positional parameters generate public `init`-only properties.
- Records support deconstruction into positional parameters.

#### Constraints and Limitations

- Records are immutable by default (properties are `init`-only), but mutable properties can be added.
- `with` expressions perform a shallow copy.
- Records cannot be `static`.
- `record struct` cannot use `with` expressions in the same way as reference-type records.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Value-Based Equality

```csharp
using System;

public record Person(string FirstName, string LastName);

class Program
{
    static void Main()
    {
        var p1 = new Person("Alice", "Smith");
        var p2 = new Person("Alice", "Smith");

        Console.WriteLine(p1 == p2);             // True
        Console.WriteLine(p1.Equals(p2));         // True
        Console.WriteLine(ReferenceEquals(p1, p2)); // False
    }
}
```

**Expected Output**

```
True
True
False
```

**Why this output occurs**

- Records use value-based equality: `p1` and `p2` have the same data, so they are equal.
- `ReferenceEquals` returns `false` because they are different objects on the heap.

---

#### Example 2: `with` Expression

```csharp
using System;

public record Person(string FirstName, string LastName);

class Program
{
    static void Main()
    {
        var p1 = new Person("Alice", "Smith");
        var p2 = p1 with { LastName = "Jones" };

        Console.WriteLine(p1);   // Person { FirstName = Alice, LastName = Smith }
        Console.WriteLine(p2);   // Person { FirstName = Alice, LastName = Jones }
    }
}
```

**Expected Output**

```
Person { FirstName = Alice, LastName = Smith }
Person { FirstName = Alice, LastName = Jones }
```

**Why this output occurs**

- The `with` expression creates a new record with the specified property changed.
- The original record `p1` is unchanged.

---

### Real-World Cases

- **DTOs and view models** — Concise data containers with value equality.
- **Domain events** — Records representing events with immutable data.
- **Configuration** — Immutable settings records.
- **Functional programming** — Records as immutable data carriers.

---

### References

- Records — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record
- Records — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/records
- record keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record

---

## 7. `object`

### Definitions

**Core Definition**
`object` is the ultimate base class of all types in C#; every type, directly or indirectly, derives from `System.Object`.

**Technical Definition**
The `object` type is an alias for `System.Object` in .NET. In the unified type system of C#, all types, predefined and user-defined, reference types and value types, inherit directly or indirectly from `System.Object`. You can assign values of any type (except `ref struct`) to variables of type `object`. When you convert a value type to `object`, the value is boxed; when you convert an `object` back to a value type, it is unboxed.

**Beginner-Friendly Explanation**
`object` is the **universal container**. Anything can be put into an `object` variable—numbers, strings, custom classes, arrays. It is the root of the entire type hierarchy. However, putting a value type into an `object` requires boxing, which has performance implications.

---

### Purposes

- To provide a common base type for all types.
- To enable polymorphic containers (e.g., `ArrayList` before generics).
- To accept parameters of any type in APIs.
- To support reflection and runtime type inspection.
- To define core methods (`ToString`, `Equals`, `GetHashCode`) inherited by all types.

---

### Syntax Rules and Structure

```
object   identifier   [ = value ]   ;
```

**Component breakdown**

- `object` — Alias for `System.Object`.
- `identifier` — The variable name.
- `= value` — Any value; value types are boxed, reference types are assigned directly.

#### Syntax Rules

- `object` is a reference type.
- All types derive from `object`.
- Value types are boxed when assigned to `object`.
- `object` variables can hold `null`.
- Core methods: `ToString()`, `Equals(object)`, `GetHashCode()`, `GetType()`.

#### Constraints and Limitations

- Boxing/unboxing overhead for value types.
- Loss of type safety when using `object` for arbitrary data.
- `ref struct` types cannot be assigned to `object`.
- `object` is not the same as `dynamic`; operations on `object` are bound at compile time.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `object` as Universal Container

```csharp
using System;

class Program
{
    static void Main()
    {
        object o1 = 42;              // boxing
        object o2 = "hello";         // reference type, no boxing
        object o3 = new int[] { 1, 2, 3 }; // array is a reference type

        Console.WriteLine(o1);       // 42
        Console.WriteLine(o2);       // hello
        Console.WriteLine(o3.GetType()); // System.Int32[]
    }
}
```

**Expected Output**

```
42
hello
System.Int32[]
```

**Why this output occurs**

- `o1` boxes the integer `42`.
- `o2` holds a string reference (strings are reference types).
- `o3` holds an array reference.
- `GetType()` reveals the runtime type.

---

#### Example 2: Polymorphism with `object`

```csharp
using System;

class Program
{
    static void Print(object value)
    {
        Console.WriteLine(value.ToString());
    }

    static void Main()
    {
        Print(42);           // 42
        Print("hello");      // hello
        Print(3.14);         // 3.14
    }
}
```

**Expected Output**

```
42
hello
3.14
```

**Why this output occurs**

- `Print` accepts any `object`; value types are boxed.
- `ToString()` is called polymorphically on the boxed values.

---

### Real-World Cases

- **Logging frameworks** — Accepting `object` parameters for arbitrary data.
- **Reflection** — `object` is used to represent values retrieved via reflection.
- **Non-generic collections** — `ArrayList`, `Hashtable` store `object`.
- **Event arguments** — `EventArgs` derives from `object`.

---

### References

- object — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- System.Object — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object
- Types — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types

---

## 8. `string`

### Definitions

**Core Definition**
`string` is a reference type that represents an immutable sequence of Unicode characters.

**Technical Definition**
The `string` type represents a sequence of zero or more Unicode characters. In .NET, `string` is an alias for `System.String`. Although `string` is a reference type, the equality operators `==` and `!=` are defined to compare the values of string objects, not references. Strings are **immutable**: you cannot change the contents of a string object after you create it. Operations that appear to modify a string (e.g., `+=`, `Replace`) actually create new string objects.

**Beginner-Friendly Explanation**
A string is a **read-only sequence of characters**. You can create strings and read their contents, but you cannot change them in place. When you “modify” a string, C# creates a brand-new string and leaves the old one unchanged. Strings look like value types in many ways (equality compares content), but they are actually reference types.

---

### Purposes

- To represent text data.
- To provide intuitive value-based equality for textual comparison.
- To support Unicode and culture-aware operations.
- To enable string manipulation through a rich API (`Substring`, `Replace`, `Split`, etc.).
- To serve as the primary text type in .NET.

---

### Syntax Rules and Structure

```
string   identifier   [ = "literal" ]   ;
```

**Component breakdown**

- `string` — Alias for `System.String`.
- `identifier` — The variable name.
- `= "literal"` — String literal (regular, verbatim, raw, or interpolated).

#### Syntax Rules

- Strings are immutable reference types.
- `==` and `!=` compare string **values**, not references.
- String literals are interned (pooled) by the runtime.
- `+` concatenates strings, creating new objects.
- Interpolated strings (`$"..."`) provide formatting.
- Verbatim strings (`@"..."`) ignore escape sequences.
- Raw string literals (`"""..."""`) are available in C# 11+.

#### Constraints and Limitations

- Immutability means frequent concatenation can create many temporary strings; use `StringBuilder` for heavy manipulation.
- String comparison can be culture-sensitive or ordinal.
- `string` is a reference type but behaves like a value type for equality.
- `null` strings must be handled carefully to avoid `NullReferenceException`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Value Equality vs. Reference Equality

```csharp
using System;

class Program
{
    static void Main()
    {
        string a = "hello";
        string b = "h";
        b += "ello";                     // creates a new string

        Console.WriteLine(a == b);                // True (value equality)
        Console.WriteLine(ReferenceEquals(a, b)); // False (different objects)
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- `a` and `b` have the same content, so `==` returns `True` (value-based string equality).
- `ReferenceEquals` returns `False` because they are different string objects.

---

#### Example 2: Immutability and `StringBuilder`

```csharp
using System;
using System.Text;

class Program
{
    static void Main()
    {
        string s = "Hello";
        s += " World";                   // creates a new string
        Console.WriteLine(s);            // Hello World

        var sb = new StringBuilder();
        sb.Append("Hello");
        sb.Append(" World");
        Console.WriteLine(sb.ToString()); // Hello World
    }
}
```

**Expected Output**

```
Hello World
Hello World
```

**Why this output occurs**

- `s += " World"` creates a new string object; the original `"Hello"` is unchanged.
- `StringBuilder` mutates its internal buffer, avoiding repeated allocations.

---

### Real-World Cases

- **Text processing** — Parsing, formatting, and searching text.
- **User input** — Reading and validating strings.
- **Serialization** — JSON, XML, and CSV data as strings.
- **Localization** — Culture-specific string formatting.

---

### References

- string — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- Strings in C# — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/
- System.String — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string

---

## 9. `array`

### Definitions

**Core Definition**
An `array` is a reference type that represents a fixed-size, ordered collection of elements of the same type.

**Technical Definition**
An array is a reference type, so the array can be a nullable reference type. Array types are reference types derived from the abstract base type `System.Array`. Arrays are allocated on the managed heap. The elements of an array can be value types or reference types. Arrays support single-dimensional, multidimensional, and jagged forms. Array elements are variables in their own right and can be passed as `ref` or `out` parameters.

**Beginner-Friendly Explanation**
An array is a **row of numbered lockers**. All lockers are the same size (same type), and you access each locker by its number (index). The array itself is a reference; when you pass an array to a method, the method gets a reference to the same row of lockers, not a copy of all the lockers.

---

### Purposes

- To store multiple values of the same type in a single variable.
- To provide efficient indexed access to elements.
- To represent fixed-size collections with contiguous storage.
- To serve as the foundation for many collection types.
- To enable passing groups of data to methods efficiently.

---

### Syntax Rules and Structure

```
// Single-dimensional array
type[]   arrayName   =   new   type[length]   ;
type[]   arrayName   =   {   element1, element2, ...   }   ;

// Multidimensional array
type[,]   arrayName   =   new   type[dim1, dim2]   ;

// Jagged array
type[][]   arrayName   =   new   type[outerLength][]   ;
```

**Component breakdown**

- `type[]` — Array type with element type `type`.
- `new type[length]` — Creates an array of the specified length.
- `{ ... }` — Array initializer.
- `type[,]` — Multidimensional array.
- `type[][]` — Jagged array (array of arrays).

#### Syntax Rules

- Arrays are reference types derived from `System.Array`.
- Arrays are zero-indexed.
- Array length is fixed at creation; it cannot be resized.
- Arrays are allocated on the managed heap.
- Array elements are automatically initialized (default values for value types, `null` for reference types).
- Arrays implement `IList<T>`, `ICollection<T>`, `IEnumerable<T>`.

#### Constraints and Limitations

- Fixed size; no dynamic resizing (use `List<T>` for dynamic collections).
- Bounds checking on every access (safe but slightly slower).
- Jagged arrays are arrays of arrays; each inner array must be allocated separately.
- Multidimensional arrays can be slower than jagged arrays for some operations.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Single-Dimensional Array

```csharp
using System;

class Program
{
    static void Main()
    {
        int[] numbers = { 10, 20, 30, 40, 50 };
        Console.WriteLine(numbers.Length);   // 5
        Console.WriteLine(numbers[2]);        // 30

        numbers[2] = 35;
        Console.WriteLine(numbers[2]);        // 35
    }
}
```

**Expected Output**

```
5
30
35
```

**Why this output occurs**

- The array is a reference type allocated on the heap.
- `Length` returns the number of elements.
- Indexing accesses or modifies elements.

---

#### Example 2: Jagged Array

```csharp
using System;

class Program
{
    static void Main()
    {
        int[][] matrix = new int[3][];
        matrix[0] = new int[] { 1, 2 };
        matrix[1] = new int[] { 3, 4, 5 };
        matrix[2] = new int[] { 6 };

        foreach (var row in matrix)
        {
            Console.WriteLine(string.Join(", ", row));
        }
    }
}
```

**Expected Output**

```
1, 2
3, 4, 5
6
```

**Why this output occurs**

- `matrix` is an array of arrays; each element is a reference to an inner array.
- Each inner array can have a different length.

---

### Real-World Cases

- **Image processing** — Pixel data stored in 2D arrays.
- **Game boards** — Grids stored as multidimensional arrays.
- **Buffers** — Byte arrays for I/O operations.
- **Lookup tables** — Precomputed arrays for fast access.

---

### References

- Arrays — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/arrays
- Array types — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/arrays
- System.Array — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.array

---

## 10. `dynamic`

### Definitions

**Core Definition**
`dynamic` is a reference type that enables operations to bypass compile-time type checking and be resolved at runtime.

**Technical Definition**
The `dynamic` type is similar to `object`, but operations on `dynamic` expressions are bound at **runtime**, not at compile time. `dynamic` is a static type in the C# type system, but an object of type `dynamic` bypasses static type checking. You cannot use `new dynamic()` and cannot derive a type from `dynamic`. Operations are bound by the Dynamic Language Runtime (DLR).

**Beginner-Friendly Explanation**
`dynamic` means “I will not tell the compiler what type this is; figure it out while the program is running.” It is flexible but shifts type errors from compile time to runtime. Think of it as a **wildcard** that lets you call any method or property, trusting that it will exist at runtime.

---

### Purposes

- To interoperate with dynamic languages and COM APIs.
- To simplify reflection-heavy code.
- To defer method resolution and member access until runtime.
- To handle scenarios where the type is not known at compile time.
- To reduce boilerplate in metaprogramming and scripting scenarios.

---

### Syntax Rules and Structure

```
dynamic   identifier   =   expression   ;
```

**Component breakdown**

- `dynamic` — Contextual keyword for runtime binding.
- `identifier` — The variable name.
- `expression` — Any expression; the result is bound at runtime.

#### Syntax Rules

- `dynamic` can be used as a local variable type, field type, parameter type, return type, and generic type argument.
- Operations are resolved at runtime by the DLR.
- `dynamic` is a static type, but it bypasses static checking for operations.
- You cannot use `new dynamic()`.
- You cannot derive a type from `dynamic`.

#### Constraints and Limitations

- Runtime binding can fail with `RuntimeBinderException`.
- Performance overhead due to runtime binding.
- Loss of compile-time safety and IntelliSense support.
- Limited use in some contexts (e.g., `const` declarations).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Dynamic Method Invocation

```csharp
using System;

class Program
{
    static void Main()
    {
        dynamic d = "Hello";
        Console.WriteLine(d.Length);          // 5 (runtime binding)

        d = 42;
        Console.WriteLine(d + 1);             // 43 (runtime binding)

        d = new int[] { 1, 2, 3 };
        Console.WriteLine(d[1]);              // 2 (runtime binding)
    }
}
```

**Expected Output**

```
5
43
2
```

**Why this output occurs**

- The runtime binder resolves `Length` on the string, `+` on the integer, and indexing on the array.
- The same `dynamic` variable holds different types at different times.

---

#### Example 2: Dynamic vs. Static Typing

```csharp
using System;

class Program
{
    static void Main()
    {
        object o = "Hello";
        // Console.WriteLine(o.Length);       // Compile error

        dynamic d = "Hello";
        Console.WriteLine(d.Length);          // OK: runtime resolution

        d = 10;
        try
        {
            Console.WriteLine(d.Length);      // RuntimeBinderException
        }
        catch (Microsoft.CSharp.RuntimeBinder.RuntimeBinderException)
        {
            Console.WriteLine("Binding failed");
        }
    }
}
```

**Expected Output**

```
5
Binding failed
```

**Why this output occurs**

- `object` requires a cast to access `Length`.
- `dynamic` resolves `Length` at runtime; when `d` holds an `int`, the member does not exist and binding fails.

---

### Real-World Cases

- **COM interop** — Simplifying Office automation and legacy COM APIs.
- **Dynamic languages** — Interoperating with IronPython, IronRuby, and JavaScript.
- **Reflection-heavy code** — Reducing boilerplate when invoking members by name.
- **Rapid prototyping** — When the exact type is intentionally fluid.

---

### References

- dynamic type — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/built-in-types
- Using type dynamic — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/using-type-dynamic
- dynamic keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types

---

## 11. Reference Equality

### Definitions

**Core Definition**
Reference equality (also called identity) means that two object references point to the **same underlying object** in memory.

**Technical Definition**
Reference equality means that two object references refer to the same underlying object. This can occur through simple assignment. Use the `ReferenceEquals` method to determine whether two references refer to the same object. The concept of reference equality applies only to reference types; value types cannot have reference equality because assignment copies the value. If you use `ReferenceEquals` to compare two value types, the result will always be `false`, even if their values are identical, because each variable is boxed into a separate object instance.

**Beginner-Friendly Explanation**
Reference equality asks: “Are these two business cards pointing to the **same** company?” It does not care whether the companies look similar; it only cares whether they are literally the same company.

---

### Purposes

- To determine whether two variables refer to the same object instance.
- To implement identity-based caching and object pooling.
- To distinguish between shared and independent objects.
- To avoid infinite recursion in object graphs (e.g., `ReferenceEquals(this, other)`).
- To support correct implementations of `Equals` and `==`.

---

### Syntax Rules and Structure

```
bool   result   =   object.ReferenceEquals(obj1, obj2)   ;
```

**Component breakdown**

- `object.ReferenceEquals` — Static method inherited by all types.
- `obj1`, `obj2` — The references to compare.
- Returns `true` if both refer to the same object or both are `null`.

#### Syntax Rules

- `ReferenceEquals` is a static method on `System.Object`.
- It returns `true` if both references are `null`.
- It returns `false` if one is `null` and the other is not.
- For value types, `ReferenceEquals` always returns `false` (each is boxed separately).

#### Constraints and Limitations

- Cannot be used to compare value types meaningfully (always `false` after boxing).
- Does not consider object content; two distinct objects with identical data are not reference-equal.
- Overriding `==` does not affect `ReferenceEquals`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Reference Equality After Assignment

```csharp
using System;

class Test
{
    public int Num { get; set; }
}

class Program
{
    static void Main()
    {
        Test a = new Test { Num = 1 };
        Test b = new Test { Num = 1 };

        Console.WriteLine(ReferenceEquals(a, b));  // False

        b = a;                                      // both refer to the same object
        Console.WriteLine(ReferenceEquals(a, b));  // True
    }
}
```

**Expected Output**

```
False
True
```

**Why this output occurs**

- Initially, `a` and `b` refer to different objects with the same data → `False`.
- After `b = a`, both refer to the same object → `True`.

---

#### Example 2: Reference Equality with Value Types

```csharp
using System;

class Program
{
    static void Main()
    {
        int x = 42;
        int y = 42;

        Console.WriteLine(ReferenceEquals(x, y));   // False (each boxed separately)
    }
}
```

**Expected Output**

```
False
```

**Why this output occurs**

- Each value type is boxed into a separate object when passed to `ReferenceEquals`.
- The two boxed objects are distinct, so reference equality is `false`.

---

### Real-World Cases

- **Object pooling** — Checking whether a pooled object is the same instance.
- **Caching** — Verifying cache hits by identity.
- **Equality implementations** — Short-circuiting `Equals` when references are identical.
- **Graph algorithms** — Avoiding revisiting the same node instance.

---

### References

- Equality comparisons — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons
- Object.ReferenceEquals — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.referenceequals
- How to test for reference equality (Identity) — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-test-for-reference-equality-identity

---

## 12. Object Equality

### Definitions

**Core Definition**
Object equality (also called value equality or equivalence) means that two objects contain the **same values**, even if they are different instances.

**Technical Definition**
Value equality means that two objects contain the same value or values. For primitive value types, tests for value equality are straightforward using `==`. For reference types, testing for value equality is more complex because it requires understanding how the type defines equality. By default, `Object.Equals(Object)` tests for reference equality for reference types. Types can override `Equals` to provide value-based equality. Records provide value-based equality automatically.

**Beginner-Friendly Explanation**
Object equality asks: “Do these two business cards point to companies that **do the same thing**?” It does not care whether they are the same company, only whether their data is equivalent.

---

### Purposes

- To compare objects based on their content rather than identity.
- To implement correct behaviour in collections (`Dictionary`, `HashSet`).
- To support value-based semantics for records and custom types.
- To enable meaningful comparisons in business logic.
- To satisfy the expectations of `==` for types that represent values.

---

### Syntax Rules and Structure

```
// Using Equals method
bool   result   =   obj1.Equals(obj2)   ;

// Using == operator (if overloaded)
bool   result2   =   obj1   ==   obj2   ;

// Using IEquatable<T>
public   bool   Equals(T   other)   { ... }
```

**Component breakdown**

- `Equals` — Virtual method on `System.Object`; can be overridden.
- `==` — Operator that can be overloaded.
- `IEquatable<T>` — Interface for strongly typed equality.

#### Syntax Rules

- The default `Object.Equals` for reference types uses reference equality.
- Types can override `Equals` to provide value equality.
- When overriding `Equals`, you should also override `GetHashCode`.
- `IEquatable<T>` provides a strongly typed `Equals(T)` method.
- Records generate value-based `Equals` and `GetHashCode`.

#### Constraints and Limitations

- Overriding `Equals` without `GetHashCode` can break hash-based collections.
- `==` and `Equals` may behave differently unless both are consistently implemented.
- Floating-point equality can be tricky due to precision issues.
- Value equality for mutable objects can be problematic if the object is used as a dictionary key.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Default Reference Equality vs. Overridden Value Equality

```csharp
using System;

class Point
{
    public int X, Y;
    public Point(int x, int y) { X = x; Y = y; }

    public override bool Equals(object obj) =>
        obj is Point p && X == p.X && Y == p.Y;

    public override int GetHashCode() => HashCode.Combine(X, Y);
}

class Program
{
    static void Main()
    {
        var p1 = new Point(1, 2);
        var p2 = new Point(1, 2);

        Console.WriteLine(p1.Equals(p2));          // True
        Console.WriteLine(ReferenceEquals(p1, p2)); // False
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- `Equals` is overridden to compare `X` and `Y` values.
- `ReferenceEquals` still checks identity and returns `False`.

---

#### Example 2: `==` Operator Overloading

```csharp
using System;

class Point
{
    public int X, Y;
    public Point(int x, int y) { X = x; Y = y; }

    public static bool operator ==(Point a, Point b) =>
        a is null ? b is null : a.Equals(b);

    public static bool operator !=(Point a, Point b) => !(a == b);

    public override bool Equals(object obj) =>
        obj is Point p && X == p.X && Y == p.Y;

    public override int GetHashCode() => HashCode.Combine(X, Y);
}

class Program
{
    static void Main()
    {
        var p1 = new Point(1, 2);
        var p2 = new Point(1, 2);

        Console.WriteLine(p1 == p2);    // True
        Console.WriteLine(p1.Equals(p2)); // True
    }
}
```

**Expected Output**

```
True
True
```

**Why this output occurs**

- The `==` operator is overloaded to use value equality.
- Both `==` and `Equals` return `True` for equivalent points.

---

### Real-World Cases

- **Value objects** — `Money`, `Address`, `DateRange` should use value equality.
- **Records** — Automatically provide value equality.
- **Collections** — `Dictionary` and `HashSet` rely on `Equals` and `GetHashCode`.
- **Testing** — Asserting that two objects have the same data.

---

### References

- Equality comparisons — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons
- Object.Equals — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.equals
- IEquatable<T> — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.iequatable-1

---

## 13. `ReferenceEquals()`

### Definitions

**Core Definition**
`ReferenceEquals()` is a static method on `System.Object` that determines whether two references point to the same object instance.

**Technical Definition**
`Object.ReferenceEquals(object objA, object objB)` returns `true` if `objA` and `objB` refer to the same object or are both `null`; otherwise, it returns `false`. It cannot be overridden and always performs reference identity comparison. For value types, it returns `false` because each argument is boxed separately.

**Beginner-Friendly Explanation**
`ReferenceEquals` is a direct question: “Are these two business cards pointing to the **exact same** company?” It does not look at the company’s data; it only checks identity.

---

### Purposes

- To test for reference identity without being affected by `Equals` or `==` overrides.
- To implement correct equality logic (short-circuit when references match).
- To detect shared object instances.
- To compare objects in pooling and caching scenarios.
- To avoid infinite recursion in `Equals` implementations.

---

### Syntax Rules and Structure

```
bool   result   =   object.ReferenceEquals(obj1, obj2)   ;
```

**Component breakdown**

- `object.ReferenceEquals` — Static method.
- `obj1`, `obj2` — References to compare.
- Returns `true` if both are the same object or both are `null`.

#### Syntax Rules

- Cannot be overridden.
- Returns `true` for two `null` references.
- Returns `false` if one is `null` and the other is not.
- For value types, always returns `false` (after boxing).

#### Constraints and Limitations

- Not meaningful for value types.
- Does not consider object content.
- Cannot be used to compare `null` against a value type without boxing.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `ReferenceEquals` with `null`

```csharp
using System;

class Program
{
    static void Main()
    {
        object a = null;
        object b = null;

        Console.WriteLine(ReferenceEquals(a, b));   // True
        Console.WriteLine(ReferenceEquals(a, new object())); // False
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- Two `null` references are considered reference-equal.
- A `null` reference and a non-null reference are not.

---

#### Example 2: `ReferenceEquals` in `Equals` Implementation

```csharp
using System;

class Person
{
    public string Name;

    public override bool Equals(object obj)
    {
        if (ReferenceEquals(this, obj)) return true;
        if (obj is null) return false;
        if (obj is not Person other) return false;
        return Name == other.Name;
    }

    public override int GetHashCode() => Name?.GetHashCode() ?? 0;
}
```

**Expected Output (conceptual)**

```
// Equals returns true for the same instance without further comparison
```

**Why this output occurs**

- `ReferenceEquals(this, obj)` short-circuits when comparing an object to itself.
- This is a standard pattern for efficient and correct `Equals` implementations.

---

### Real-World Cases

- **Equality implementations** — Short-circuiting when references are identical.
- **Object pooling** — Checking whether a returned object is the pooled instance.
- **Caching** — Verifying cache hits by identity.
- **Graph traversal** — Avoiding revisiting the same node instance.

---

### References

- Object.ReferenceEquals — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.referenceequals
- Equality comparisons — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons

---

## 14. `Equals()`

### Definitions

**Core Definition**
`Equals()` is a virtual method on `System.Object` used to determine whether two objects are equal.

**Technical Definition**
`Object.Equals(Object)` is a virtual method that determines whether the current instance is equal to another object. For reference types, the default implementation tests for reference equality, and a call to `Equals(Object)` is equivalent to a call to `ReferenceEquals`. Types can override `Equals` to provide value-based equality. The `IEquatable<T>` interface provides a strongly typed `Equals(T)` method. When overriding `Equals`, you should also override `GetHashCode`.

**Beginner-Friendly Explanation**
`Equals` is the general question: “Are these two things equal?” The default answer for classes is “only if they are the same object.” But you can teach your class to answer “yes” when the data matches, even if they are different objects.

---

### Purposes

- To provide a customizable equality comparison.
- To support value equality for custom types.
- To enable correct behaviour in hash-based collections.
- To satisfy the expectations of collection APIs (`List.Contains`, `Dictionary` lookups).
- To provide a consistent equality contract across types.

---

### Syntax Rules and Structure

```
// Object.Equals virtual method
public   virtual   bool   Equals(object obj)   ;

// IEquatable<T> strongly typed method
public   bool   Equals(T   other)   ;
```

**Component breakdown**

- `Equals(object)` — Virtual method on `System.Object`.
- `Equals(T)` — Strongly typed method from `IEquatable<T>`.
- Override — Provides custom equality logic.

#### Syntax Rules

- The default `Equals(object)` for reference types uses reference equality.
- Overriding `Equals` should also override `GetHashCode`.
- `Equals` should be reflexive, symmetric, transitive, and consistent.
- `Equals(null)` should return `false` (unless overridden otherwise).
- `Equals` should not throw exceptions.

#### Constraints and Limitations

- Overriding `Equals` without `GetHashCode` breaks hash-based collections.
- Mutable objects used as keys can cause issues if mutated after insertion.
- Floating-point `NaN` is not equal to itself under `Equals` (following IEEE 754).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Overriding `Equals` for Value Equality

```csharp
using System;

class Product
{
    public string Name;
    public decimal Price;

    public override bool Equals(object obj) =>
        obj is Product p && Name == p.Name && Price == p.Price;

    public override int GetHashCode() => HashCode.Combine(Name, Price);
}

class Program
{
    static void Main()
    {
        var a = new Product { Name = "Widget", Price = 9.99m };
        var b = new Product { Name = "Widget", Price = 9.99m };

        Console.WriteLine(a.Equals(b));          // True
        Console.WriteLine(a.Equals(null));        // False
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- `Equals` compares `Name` and `Price`.
- `Equals(null)` returns `false`.

---

#### Example 2: `IEquatable<T>` Implementation

```csharp
using System;

class Product : IEquatable<Product>
{
    public string Name;
    public decimal Price;

    public bool Equals(Product other) =>
        other is not null && Name == other.Name && Price == other.Price;

    public override bool Equals(object obj) => Equals(obj as Product);

    public override int GetHashCode() => HashCode.Combine(Name, Price);
}
```

**Expected Output (conceptual)**

```
// Strongly typed Equals avoids boxing and type checking overhead
```

**Why this output occurs**

- `IEquatable<Product>.Equals` provides a strongly typed comparison.
- The `object` override delegates to the strongly typed version.

---

### Real-World Cases

- **Domain models** — Value-based equality for entities with natural keys.
- **Collections** — `Dictionary`, `HashSet`, `List.Contains` rely on `Equals`.
- **Testing** — Asserting object equality in unit tests.
- **Caching** — Comparing cached values.

---

### References

- Object.Equals — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.equals
- IEquatable<T> — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.iequatable-1
- Equality comparisons — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons

---

## 15. `==` Operator

### Definitions

**Core Definition**
The `==` operator compares two operands for equality; its behaviour depends on the operand types and whether the operator is overloaded.

**Technical Definition**
For reference types, the default `==` operator compares references (identity), unless the operator is overloaded. For `string`, the `==` and `!=` operators are defined to compare the **values** of string objects, not references. For value types, `==` compares values. Types can overload `==` to provide custom equality semantics. When overloading `==`, you should also overload `!=` and override `Equals` and `GetHashCode`.

**Beginner-Friendly Explanation**
`==` is the familiar equality symbol, but its meaning depends on what you are comparing. For numbers, it compares values. For classes, the default is “same object.” For strings, it compares text content. You can customize `==` for your own types.

---

### Purposes

- To provide intuitive equality syntax for types.
- To support value-based comparison for strings and custom types.
- To enable concise conditional expressions.
- To allow types to define their own equality semantics.
- To align with user expectations for equality.

---

### Syntax Rules and Structure

```
bool   result   =   left   ==   right   ;
```

**Component breakdown**

- `left`, `right` — Operands of compatible types.
- `==` — Equality operator.
- Returns `true` if the operands are equal according to the type’s equality rules.

#### Syntax Rules

- For reference types without an overload, `==` compares references.
- For `string`, `==` compares content (value equality).
- For value types, `==` compares values.
- `==` can be overloaded for custom types.
- When overloading `==`, also overload `!=`.
- When overloading `==`, override `Equals` and `GetHashCode` for consistency.

#### Constraints and Limitations

- Overloading `==` without overriding `Equals` can cause inconsistent behaviour.
- Reference equality for classes can surprise developers expecting value equality.
- `==` for `double` and `float` follows IEEE 754 (e.g., `NaN != NaN`).
- `==` on `object` references uses reference equality unless overloaded.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `==` with Strings vs. Objects

```csharp
using System;

class Program
{
    static void Main()
    {
        string s1 = "hello";
        string s2 = "hello";

        Console.WriteLine(s1 == s2);              // True (value equality)

        object o1 = s1;
        object o2 = s2;
        Console.WriteLine(o1 == o2);              // True (same reference due to interning)
    }
}
```

**Expected Output**

```
True
True
```

**Why this output occurs**

- `==` on `string` compares content.
- `==` on `object` compares references; here the references are the same due to string interning.

---

#### Example 2: Overloading `==`

```csharp
using System;

public struct Point
{
    public int X, Y;

    public static bool operator ==(Point a, Point b) => a.X == b.X && a.Y == b.Y;
    public static bool operator !=(Point a, Point b) => !(a == b);

    public override bool Equals(object obj) =>
        obj is Point p && this == p;

    public override int GetHashCode() => HashCode.Combine(X, Y);
}

class Program
{
    static void Main()
    {
        var p1 = new Point { X = 1, Y = 2 };
        var p2 = new Point { X = 1, Y = 2 };

        Console.WriteLine(p1 == p2);    // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- The `==` operator is overloaded to compare `X` and `Y`.
- `!=` is also overloaded for consistency.

---

### Real-World Cases

- **String comparison** — Intuitive text equality.
- **Value objects** — Custom `==` for `Money`, `Point`, etc.
- **Null checks** — `obj == null` is a common idiom (reference comparison).
- **LINQ queries** — Equality comparisons in `Where` clauses.

---

### References

- Equality operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/equality-operators
- Equality comparisons — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons
- String equality — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/equality-operators#string-equality

---

## 16. Reference Type Best Practices

### Definitions

**Core Definition**
Reference type best practices are the guidelines for choosing, designing, and using reference types to maximize correctness, performance, and maintainability.

**Technical Definition**
Best practices for reference types derive from the .NET Framework Design Guidelines and the C# language specification. They cover when to use `class` versus `struct`, how to design immutable reference types, how to implement equality correctly, and how to avoid common pitfalls such as null-reference exceptions and unnecessary allocations.

**Beginner-Friendly Explanation**
Best practices are the habits that keep reference-type code safe and efficient: use classes for complex objects, prefer immutability where possible, implement equality consistently, and avoid unnecessary heap allocations.

---

### Purposes

- To choose the right type (class vs. struct) for the job.
- To design reference types that are safe, predictable, and efficient.
- To implement equality consistently.
- To avoid common pitfalls such as null-reference exceptions.
- To reduce heap allocations and GC pressure when appropriate.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer classes for complex objects with identity
public class Customer
{
    public int Id { get; }
    public string Name { get; }

    public Customer(int id, string name) => (Id, Name) = (id, name);
}

// Use records for data-centric types
public record CustomerDto(int Id, string Name);

// Implement equality consistently
// Override Equals and GetHashCode when providing value equality
// Overload == and != for value-based equality

// Use interfaces for contracts
public interface IRepository<T> { T GetById(int id); }

// Avoid null-reference exceptions
// Use null-conditional operators, nullable reference types
```

**Component breakdown**

- **Classes for identity** — Use classes when objects have identity and mutable state.
- **Records for data** — Use records for immutable data with value equality.
- **Consistent equality** — Override `Equals`, `GetHashCode`, and optionally `==`/`!=`.
- **Interfaces for abstraction** — Depend on interfaces, not concrete classes.
- **Null safety** — Use nullable reference types and null checks.

#### Rules and Constraints

- **Consider `class` over `struct`** if the type is large, has reference-like behaviour, or requires inheritance.
- **Prefer `readonly` and immutability** where possible.
- **Implement `IEquatable<T>`** for value-equality types to avoid boxing.
- **Override `GetHashCode` whenever you override `Equals`**.
- **Use `ReferenceEquals`** to short-circuit equality checks.
- **Avoid public fields**; use properties.
- **Use `sealed`** on classes not designed for inheritance.
- **Avoid excessive heap allocations** in hot paths.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Good Class Design

```csharp
using System;

public sealed class Temperature
{
    public double Celsius { get; }

    public Temperature(double celsius)
    {
        if (celsius < -273.15)
            throw new ArgumentOutOfRangeException(nameof(celsius));
        Celsius = celsius;
    }

    public double Fahrenheit => Celsius * 9 / 5 + 32;

    public override bool Equals(object obj) =>
        obj is Temperature t && Celsius == t.Celsius;

    public override int GetHashCode() => Celsius.GetHashCode();
}
```

**Expected Output (conceptual)**

```
// Immutable, validated, value-equal, sealed class
```

**Why this is good practice**

- The class is `sealed`, preventing unintended inheritance.
- The property is read-only, making the object immutable.
- Constructor validation ensures a valid state.
- `Equals` and `GetHashCode` provide value equality.

---

#### Example 2: Avoiding Null-Reference Exceptions

```csharp
using System;

class Program
{
    static void Main()
    {
        string? name = null;
        Console.WriteLine(name?.Length ?? 0);   // 0

        var person = GetPerson();
        Console.WriteLine(person?.Name ?? "Unknown");
    }

    static Person? GetPerson() => null;
}

class Person { public string Name { get; set; } }
```

**Expected Output**

```
0
Unknown
```

**Why this is good practice**

- Null-conditional operator `?.` safely accesses members.
- Null-coalescing operator `??` provides fallbacks.
- Nullable reference types (`string?`, `Person?`) express nullability intent.

---

### Real-World Cases

- **API design** — Public APIs should prefer abstract classes and interfaces for extensibility.
- **Domain modelling** — Classes for entities, records for value objects.
- **Performance-sensitive code** — Avoiding heap allocations with `struct` where appropriate.
- **Testing** — Using interfaces for mocking and dependency injection.

---

### References

- Choosing Between Classes and Structures - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct
- Framework Design Guidelines — Equality - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/equality
- C# Coding Conventions - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- Write safe and efficient C# code - https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/performance/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `dynamic` | Not deprecated, but runtime binding introduces performance overhead and runtime errors; use only when necessary. |
| Default interface members | Introduced in **C# 8.0**; version-specific. |
| `record` / `record struct` | Introduced in **C# 9.0** (record) and **C# 10.0** (record struct); version-specific. |
| Raw string literals | Introduced in **C# 11**; version-specific. |
| Nullable reference types | Introduced in **C# 8.0**; version-specific. |
| Heap allocation | **Implementation-dependent**; the specification guarantees reference semantics, not physical storage details. |
| String interning | Implementation detail; do not rely on it for correctness. |
| `ArrayList` / non-generic collections | **Legacy**; cause boxing of value types. Prefer generic collections. |
| Overloading `==` without `Equals` | **Discouraged**; can lead to inconsistent equality behaviour. |
| Mutable objects as dictionary keys | **Risky**; mutation after insertion can corrupt hash tables. |

---

## References (Consolidated)

- Reference types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types
- Types — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types
- Built-in reference types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- Value and Reference Types — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2008/4d43ts61(v=vs.90)
- Classes — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- Classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes
- class keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/class
- interface keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface
- Interfaces — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/interfaces
- Interfaces — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces
- Delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/
- delegate keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- Delegates — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/delegates
- Records — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record
- Records — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/records
- object — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- System.Object — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object
- string — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- Strings in C# — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/
- System.String — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string
- Arrays — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/arrays
- Array types — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/arrays
- System.Array — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.array
- dynamic type — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/built-in-types
- Using type dynamic — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/using-type-dynamic
- Equality comparisons — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons
- Object.ReferenceEquals — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.referenceequals
- Object.Equals — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.equals
- IEquatable<T> — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.iequatable-1
- Equality operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/equality-operators
- How to test for reference equality (Identity) — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-test-for-reference-equality-identity
- Choosing Between Classes and Structures - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct
- Framework Design Guidelines — Equality - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/equality
- C# Coding Conventions - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- Write safe and efficient C# code - https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/performance/
- Advanced Memory Management — Developer Knowledge Base - https://mintlify.wiki

---