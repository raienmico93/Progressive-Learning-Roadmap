# C# Classes and Objects — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **class** is a reference type that serves as a blueprint for creating objects. It encapsulates data (fields, properties) and behaviour (methods, events, constructors) into a single logical unit. An **object** (also called an **instance**) is a concrete entity created from a class blueprint, occupying memory on the managed heap and holding its own copy of the class's instance data. C# is a fully object-oriented language: every piece of data and every executed instruction is ultimately associated with a class or an object.

**Technical Definition**
In C#, a class is declared with the `class` keyword followed by a unique identifier. A class defines a type of object but is not itself an object; an object is a concrete entity based on a class, sometimes referred to as an instance of the class. When an object is created with the `new` operator, enough memory is allocated on the managed heap for that specific object, and the variable holds only a reference to that location. The memory used by an object is reclaimed by the CLR's automatic memory management, known as **garbage collection**. A class's **members** include all members declared in the class, along with all members (except constructors and finalizers) declared in all classes in its inheritance hierarchy. Private members in base classes are inherited but are not accessible from derived classes. The kinds of members a class can contain include **fields** (variables declared at class scope), **constants** (fields whose value is set at compile time), **properties** (methods accessed as if they were fields), **methods** (actions the class can perform), **events** (notifications), **operators** (overloaded operators), **indexers**, **constructors** (called when the object is first created), **finalizers** (called by the runtime when the object is about to be removed), and **nested types**.

**Beginner-Friendly Explanation**
A class is a **cookie cutter**, and objects are the **cookies**. The class defines the shape—what data each cookie holds (fields and properties) and what each cookie can do (methods). Every cookie made from the same cutter has the same shape but can have different icing, sprinkles, and decorations (its own data). You create cookies with the `new` operator. When you are done with a cookie, the garbage collector eventually cleans it up so you do not have to worry about throwing it away yourself. Classes are the fundamental building blocks of C# programs: they organise data and behaviour together, protect data through encapsulation, and enable inheritance and polymorphism.

---

### Key Characteristics

- **Reference type** — Class variables hold references to objects on the managed heap, not the object data itself.
- **Blueprint vs. instance** — The class is the type; the object is a concrete instance of that type.
- **Encapsulation** — Fields and properties hold data; methods define behaviour; access modifiers control visibility.
- **Constructor-driven initialisation** — Constructors run when objects are created; `this` chains to sibling constructors, `base` chains to base-class constructors.
- **Finalizer-driven cleanup** — Finalizers (historically called destructors) run before garbage collection, but only for unmanaged resources.
- **Garbage-collected lifetime** — The CLR reclaims object memory automatically; `StackOverflowException` is not the concern here—rather, finalizers are non-deterministic.
- **Three equality protocols** — `Equals`, `GetHashCode`, and `ToString` are the three core `System.Object` methods that every class inherits and can override.
- **Explicit object creation** — The `new` operator allocates and initialises objects; object initialisers provide concise member assignment.

---

### Prerequisites

- Basic understanding of C# syntax, types, and variables.
- Familiarity with methods, parameters, and return values.
- Awareness of the `System` namespace and the `Console` class.
- Knowledge of access modifiers (`public`, `private`, `protected`, `internal`).
- Understanding of value types vs. reference types.

---

### Related Programming Areas

- **Object-oriented programming** — Encapsulation, inheritance, and polymorphism are built on classes.
- **Memory management** — Heap allocation and garbage collection define object lifetimes.
- **Equality and hashing** — `Equals` and `GetHashCode` are essential for collections and value semantics.
- **API design** — Classes are the primary unit of abstraction in .NET libraries.
- **Serialisation** — Objects are the primary data shape for JSON, XML, and binary serialisation.

---

### Core Concepts / Features

1. Classes
2. Objects and Instances
3. Class Declaration
4. Class Members (Fields, Properties, Methods, Constructors, Destructors)
5. `this` Keyword
6. `base` Keyword
7. Object Creation (`new` Keyword)
8. Object Initialization
9. Object Lifecycle and Garbage Collection
10. Object Equality (`Equals`, `GetHashCode`, `ToString`)
11. Object Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Classes

### Definitions

**Core Definition**
A class is a reference type that acts as a blueprint for creating objects, encapsulating data and behaviour.

**Technical Definition**
A class is declared with the `class` keyword followed by a unique identifier. It defines a type of object, but it is not an object itself. A class can contain fields, properties, methods, events, operators, indexers, constructors, finalizers, and nested types. Classes are reference types: at run time, when you declare a variable of a reference type, the variable contains the value `null` until you explicitly create an instance of the class by using the `new` operator. Classes support inheritance, but only single inheritance from a base class; however, a class can implement multiple interfaces.

**Beginner-Friendly Explanation**
A class is a **recipe** or a **blueprint**. It describes what data an object will hold and what it can do, but it is not the object itself. You use the class to create as many objects as you need.

---

### Purposes

- To define a reusable template for creating objects with shared structure and behaviour.
- To encapsulate related data and operations into a cohesive unit.
- To enable inheritance and polymorphism through class hierarchies.
- To serve as the primary unit of abstraction in object-oriented design.
- To support access control and data hiding through encapsulation.

---

### Syntax Rules and Structure

```
[access modifier] class ClassName [: baseClass] [, interface1, interface2, ...]
{
    // Fields, properties, methods, events, constructors, etc.
}
```

**Component breakdown**

- `access modifier` — Optional; `public`, `internal`, etc.
- `class` — The keyword introducing a reference type.
- `ClassName` — A valid C# identifier.
- `: baseClass` — Optional base class.
- `: interface1, ...` — Optional interface implementations.

#### Syntax Rules

- The class name must be a valid C# identifier.
- A class can inherit from at most one base class.
- A class can implement multiple interfaces.
- Classes are reference types allocated on the managed heap.

#### Constraints and Limitations

- Cannot inherit from a sealed class.
- Cannot be instantiated if declared `abstract`.
- Nested classes have access to the enclosing type's private members.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: A Simple Class

```csharp
using System;

public class Greeter
{
    public string Name { get; set; }

    public void Greet()
    {
        Console.WriteLine($"Hello, {Name}!");
    }
}

class Program
{
    static void Main()
    {
        var g = new Greeter { Name = "Alice" };
        g.Greet();   // Hello, Alice!
    }
}
```

**Expected Output**

```
Hello, Alice!
```

**Why this output occurs**

- `Greeter` is a class (blueprint); `g` is an object (instance) created with `new`. `Greet` uses the instance's `Name` property.

---

#### Example 2: Class with Inheritance

```csharp
using System;

public class Animal
{
    public string Name { get; set; }
    public virtual void Speak() => Console.WriteLine("...");
}

public class Dog : Animal
{
    public override void Speak() => Console.WriteLine($"{Name} says woof!");
}

class Program
{
    static void Main()
    {
        Animal a = new Dog { Name = "Rex" };
        a.Speak();   // Rex says woof!
    }
}
```

**Expected Output**

```
Rex says woof!
```

**Why this output occurs**

- `Dog` inherits from `Animal` and overrides `Speak`. Polymorphism dispatches to `Dog.Speak`.

---

### Real-World Cases

- **Domain models** — `Customer`, `Order`, `Product` classes represent business entities.
- **UI frameworks** — `Form`, `Control`, `Button` classes form hierarchies.
- **Service layers** — Classes implementing service interfaces encapsulate business logic.

---

### References

- Classes — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes
- Object-Oriented Programming (C#) — Microsoft Docs - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/object-oriented-programming
- class keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/class

---

## 2. Objects and Instances

### Definitions

**Core Definition**
An object (or instance) is a concrete entity created from a class, occupying memory and holding its own copy of the class's instance data.

**Technical Definition**
An object is a concrete entity based on a class, and is sometimes referred to as an instance of a class. Objects are created using the `new` operator followed by the class name. When an instance of a class is created, a reference to the object is passed back to the programmer. This reference refers to the new object but does not contain the object data itself. Multiple object variables can refer to the same object; assigning one object variable to another copies the reference, not the data.

**Beginner-Friendly Explanation**
An object is a **cookie made from the cookie cutter**. Each cookie is a separate physical thing with its own icing and sprinkles. You can have many cookies from the same cutter, and they are all independent objects.

---

### Purposes

- To create concrete, usable entities from class blueprints.
- To hold and manage instance-specific state.
- To invoke instance methods and access instance properties.
- To model real-world entities with identity and behaviour.
- To enable sharing of objects through references.

---

### Syntax Rules and Structure

```
ClassName variableName = new ClassName();
ClassName variableName = new ClassName { Property = value, ... };
```

**Component breakdown**

- `ClassName` — The class (type) from which to create the object.
- `variableName` — The reference variable.
- `new ClassName()` — Allocates and initialises the object.

#### Syntax Rules

- Objects are reference types; the variable holds a reference.
- Multiple variables can reference the same object.
- `null` means the variable refers to no object.
- Objects are garbage-collected when no references remain.

#### Constraints and Limitations

- Accessing a member on a `null` reference throws `NullReferenceException`.
- Object creation requires a constructor (explicit or implicit).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Creating and Sharing Objects

```csharp
using System;

public class Person
{
    public string Name { get; set; }
}

class Program
{
    static void Main()
    {
        var p1 = new Person { Name = "Alice" };
        var p2 = p1;                 // copy the reference

        p2.Name = "Bob";
        Console.WriteLine(p1.Name);   // Bob (same object)
    }
}
```

**Expected Output**

```
Bob
```

**Why this output occurs**

- `p1` and `p2` refer to the same object; modifying through `p2` changes the shared object.

---

#### Example 2: Null Reference

```csharp
using System;

public class Person { public string Name { get; set; } }

class Program
{
    static void Main()
    {
        Person p = null;
        Console.WriteLine(p == null);   // True
        // Console.WriteLine(p.Name);   // NullReferenceException
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `p` holds no reference; checking for `null` is safe, but dereferencing causes an exception.

---

### Real-World Cases

- **User objects** — Representing a logged-in user.
- **Data transfer objects (DTOs)** — Carrying data between layers.
- **Entities** — Persistent objects with identity.

---

### References

- Classes — C# reference (Objects) - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes
- Object-Oriented Programming (C#) — Microsoft Docs - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/object-oriented-programming

---

## 3. Class Declaration

### Definitions

**Core Definition**
Class declaration is the syntax that introduces a new class type, specifying its name, accessibility, base class, interfaces, and members.

**Technical Definition**
Classes are declared by using the `class` keyword followed by a unique identifier. An optional access modifier precedes the `class` keyword. The name of the class follows the `class` keyword and must be a valid C# identifier name. The remainder of the definition is the class body, where the behaviour and data are defined. A class can optionally inherit from a base class and implement interfaces.

**Beginner-Friendly Explanation**
Declaring a class is **writing the blueprint**. You give the class a name, optionally say what it inherits from, and then fill in its members.

---

### Purposes

- To introduce a new reference type into the program.
- To specify the class's accessibility and inheritance.
- To define the class's members and behaviour.
- To organise code into logical units.

---

### Syntax Rules and Structure

```
[access modifier] class ClassName [: baseClass] [, interfaces]
{
    // class body
}
```

**Component breakdown**

- `access modifier` — Optional; `public`, `internal`, etc.
- `class` — The keyword.
- `ClassName` — Valid identifier.
- `: baseClass` — Optional single base class.
- `: interfaces` — Optional interface list.
- `{ ... }` — The class body.

#### Syntax Rules

- The class name must be a valid identifier.
- Only one base class is allowed.
- Multiple interfaces are allowed.
- The class body can be empty.

#### Constraints and Limitations

- Cannot inherit from a sealed class.
- Cannot use `char` or other invalid identifiers.

---

### Annotated Code Examples and Expected Outputs

#### Example: Declaring a Class with Inheritance and Interfaces

```csharp
using System;

public interface IIdentifiable
{
    int Id { get; }
}

public abstract class Entity : IIdentifiable
{
    public int Id { get; protected set; }
}

public class Customer : Entity
{
    public string Name { get; set; }
}

class Program
{
    static void Main()
    {
        var c = new Customer { Id = 1, Name = "Acme" };
        Console.WriteLine($"{c.Id}: {c.Name}");   // 1: Acme
    }
}
```

**Expected Output**

```
1: Acme
```

**Why this output occurs**

- `Customer` inherits from `Entity` and implements `IIdentifiable` indirectly.

---

### Real-World Cases

- **Library design** — Declaring public classes with appropriate inheritance.
- **Domain modelling** — Base classes and derived classes.

---

### References

- Classes — C# reference (Declaring classes) - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes
- class keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/class

---

## 4. Class Members

### Definitions

**Core Definition**
Class members are the fields, properties, methods, constructors, finalizers, events, operators, indexers, and nested types that make up a class's data and behaviour.

**Technical Definition**
A class's members include all the members declared in the class, along with all members (except constructors and finalizers) declared in all classes in its inheritance hierarchy. Private members in base classes are inherited but are not accessible from derived classes. The main kinds of members are: **fields** (variables declared at class scope), **constants** (compile-time fields), **properties** (methods accessed as fields), **methods** (actions the class can perform), **events** (notifications), **operators** (overloaded operators), **indexers**, **constructors**, **finalizers**, and **nested types**.

**Beginner-Friendly Explanation**
Members are the **parts** of a class. Fields and properties are the data; methods are the actions; constructors build the object; finalizers clean it up.

---

### Purposes

- To store data (fields, properties).
- To define behaviour (methods).
- To initialise objects (constructors).
- To clean up resources (finalizers).
- To provide notifications (events).
- To support indexing and operator overloading.

---

### Syntax Rules and Structure (Member Kinds)

| Member | Description |
|---|---|
| **Fields** | Variables declared at class scope |
| **Constants** | Compile-time constant fields |
| **Properties** | Methods accessed as fields, providing encapsulation |
| **Methods** | Actions the class can perform |
| **Events** | Notifications using delegates |
| **Operators** | Overloaded operators |
| **Indexers** | Array-like indexing |
| **Constructors** | Called when the object is created |
| **Finalizers** | Called before garbage collection |
| **Nested types** | Types declared within another type |

**Component breakdown**

- **Fields** — Raw data storage.
- **Properties** — Controlled data access.
- **Methods** — Behaviour.
- **Constructors** — Initialisation.
- **Finalizers** — Cleanup.

#### Syntax Rules

- Members have accessibility modifiers.
- Static members belong to the type; instance members belong to objects.
- Members can be inherited (except constructors and finalizers).

#### Constraints and Limitations

- Private base members are inherited but inaccessible.
- Constructors and finalizers are not inherited.

---

### Annotated Code Examples and Expected Outputs

#### Example: A Class with Multiple Member Kinds

```csharp
using System;

public class BankAccount
{
    private decimal _balance;                  // field

    public const decimal MinimumBalance = 0;   // constant

    public string Owner { get; }               // property

    public BankAccount(string owner, decimal initial)  // constructor
    {
        Owner = owner;
        _balance = initial;
    }

    public void Deposit(decimal amount)        // method
    {
        _balance += amount;
    }

    public decimal Balance => _balance;        // read-only property

    ~BankAccount()                             // finalizer
    {
        Console.WriteLine($"Finalizing account for {Owner}");
    }
}

class Program
{
    static void Main()
    {
        var acc = new BankAccount("Alice", 100m);
        acc.Deposit(50m);
        Console.WriteLine(acc.Balance);   // 150
    }
}
```

**Expected Output**

```
150
```

**Why this output occurs**

- The class demonstrates fields, constants, properties, constructors, methods, and a finalizer (finalizer output is non-deterministic).

---

### Real-World Cases

- **All class design** — Members define the class's API and internal state.

---

### References

- Members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/members

---

## 5. Fields

### Definitions

**Core Definition**
A field is a variable declared directly in a class or struct, representing data that each object (or the type itself, if static) contains.

**Technical Definition**
Fields are variables declared at class scope. A field may be a built-in numeric type or an instance of another class. For example, a calendar class may have a field that contains the current date. Fields can be instance fields (one per object) or static fields (one per type). Fields are typically private or protected and accessed through properties or methods.

**Beginner-Friendly Explanation**
A field is a **data slot** inside an object. Each object has its own copy of each instance field.

---

### Purposes

- To store an object's data directly.
- To hold state that properties expose or methods manipulate.
- To provide backing storage for properties.
- To define type-level (static) state.

---

### Syntax Rules and Structure

```
[access modifier] [modifiers] type fieldName [= initializer];
```

**Component breakdown**

- `access modifier` — Usually `private` or `protected`.
- `modifiers` — `static`, `readonly`, `const`, etc.
- `type` — The field's type.
- `fieldName` — The field's identifier.

#### Syntax Rules

- Fields are initialised to default values if no initializer is supplied.
- `readonly` fields can only be assigned in the declaration or constructor.
- `const` fields are compile-time constants.

#### Constraints and Limitations

- Public fields are discouraged; use properties for encapsulation.
- Fields cannot be `abstract` or `virtual`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Private Field with Property

```csharp
using System;

public class Temperature
{
    private double _celsius;

    public double Celsius
    {
        get => _celsius;
        set => _celsius = value;
    }
}

class Program
{
    static void Main()
    {
        var t = new Temperature { Celsius = 25 };
        Console.WriteLine(t.Celsius);   // 25
    }
}
```

**Expected Output**

```
25
```

**Why this output occurs**

- `_celsius` is the backing field; `Celsius` is the property that controls access.

---

### Real-World Cases

- **Backing fields for properties** — `_name`, `_balance`.
- **Constants** — `const double Pi = 3.14159`.

---

### References

- Fields — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/fields

---

## 6. Properties

### Definitions

**Core Definition**
A property is a member that provides a flexible mechanism to read, write, or compute the value of a private field, accessed as if it were a public field.

**Technical Definition**
Properties are methods on a class that are accessed as if they were fields on that class. A property can provide protection for a class field to keep it from being changed without the knowledge of the object. Properties have `get` and `set` accessors. Auto-implemented properties (`public int Value { get; set; }`) create a hidden backing field automatically.

**Beginner-Friendly Explanation**
A property is a **smart field**. It looks like a field from the outside but can run code when you read or write it.

---

### Purposes

- To encapsulate data access with validation or computation.
- To provide read-only, write-only, or read-write access.
- To compute values on demand.
- To support data binding and serialisation.

---

### Syntax Rules and Structure

```
[access modifier] type PropertyName { get; set; }
[access modifier] type PropertyName => expression;
```

**Component breakdown**

- `access modifier` — Usually `public`.
- `type` — The property's type.
- `PropertyName` — The property's identifier.
- `{ get; set; }` — Accessors.
- `=> expression` — Expression-bodied property.

#### Syntax Rules

- Properties can have `get` only, `set` only, or both.
- Expression-bodied properties are read-only.
- Auto-implemented properties have compiler-generated backing fields.

#### Constraints and Limitations

- `set` accessor cannot be omitted for write access.
- Expression-bodied properties cannot have setters.

---

### Annotated Code Examples and Expected Outputs

#### Example: Auto-Implemented and Expression-Bodied Properties

```csharp
using System;

public class Circle
{
    public double Radius { get; set; }          // auto-implemented
    public double Area => Math.PI * Radius * Radius; // expression-bodied
}

class Program
{
    static void Main()
    {
        var c = new Circle { Radius = 2 };
        Console.WriteLine(c.Area.ToString("F2"));   // 12.57
    }
}
```

**Expected Output**

```
12.57
```

**Why this output occurs**

- `Radius` is an auto-property; `Area` computes from `Radius` on each access.

---

### Real-World Cases

- **DTOs** — Simple data-carrying properties.
- **Computed properties** — `FullName => $"{First} {Last}"`.

---

### References

- Properties — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties

---

## 7. Methods

### Definitions

**Core Definition**
A method is a named block of executable code that defines an action a class can perform.

**Technical Definition**
Methods define the actions that a class can perform. Methods can take parameters that provide input data, and can return output data through parameters. Methods can also return a value directly, without using a parameter. Methods can be instance methods (operating on the object) or static methods (operating on the type).

**Beginner-Friendly Explanation**
Methods are the **verbs** of a class—what the object can do.

---

### Purposes

- To define behaviour.
- To manipulate object state.
- To provide services to other objects.
- To implement algorithms.

---

### Syntax Rules and Structure

```
[access modifier] [modifiers] returnType MethodName(parameters)
{
    // body
}
```

**Component breakdown**

- `access modifier` — `public`, `private`, etc.
- `modifiers` — `static`, `virtual`, `abstract`, etc.
- `returnType` — The result type or `void`.
- `MethodName` — The method's identifier.
- `parameters` — Input parameters.

#### Syntax Rules

- Non-void methods must return a value on all paths.
- Methods can be overloaded by signature.
- Methods can be overridden if `virtual` or `abstract`.

#### Constraints and Limitations

- Cannot override non-virtual methods.
- Abstract methods have no body.

---

### Annotated Code Examples and Expected Outputs

#### Example: Instance Method

```csharp
using System;

public class Counter
{
    private int _count;
    public void Increment() => _count++;
    public int Count => _count;
}

class Program
{
    static void Main()
    {
        var c = new Counter();
        c.Increment();
        Console.WriteLine(c.Count);   // 1
    }
}
```

**Expected Output**

```
1
```

**Why this output occurs**

- `Increment` modifies the object's state; `Count` exposes it.

---

### Real-World Cases

- **Domain operations** — `Order.CalculateTotal()`.
- **Service methods** — `Repository.Save(entity)`.

---

### References

- Methods — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/methods

---

## 8. Constructors

### Definitions

**Core Definition**
A constructor is a special method that runs when an object is created, used to initialise the object's state.

**Technical Definition**
You declare an instance constructor to specify the code that is executed when you create a new instance of a type with the `new` expression. Constructors have the same name as the class and no return type. A class can have multiple constructors (overloaded). If a class has no explicit constructors, C# provides a parameterless constructor that initialises fields to their default values. Constructors can chain to other constructors in the same class using `this(...)`, or to a base-class constructor using `base(...)`.

**Beginner-Friendly Explanation**
A constructor is the **setup routine** that runs when you create an object. It puts the object into a valid initial state.

---

### Purposes

- To initialise object state.
- To enforce invariants at creation.
- To provide multiple ways to create an object (overloaded constructors).
- To chain initialisation logic and avoid duplication.
- To call base-class initialisation.

---

### Syntax Rules and Structure

```
[access modifier] ClassName(parameters) [: this(...) | : base(...)]
{
    // initialisation
}
```

**Component breakdown**

- `access modifier` — Usually `public`.
- `ClassName` — Must match the class name.
- `parameters` — Input values.
- `: this(...)` — Chain to another constructor in the same class.
- `: base(...)` — Chain to a base-class constructor.

#### Syntax Rules

- Constructors have no return type.
- Constructors can be overloaded.
- `this(...)` and `base(...)` must be the first part of the constructor.
- If no constructor is declared, a parameterless constructor is provided.

#### Constraints and Limitations

- Cannot be `virtual`, `abstract`, or `override`.
- Cannot be called explicitly except via `new` or constructor chaining.
- If any constructor is declared, the implicit parameterless constructor is not provided.

---

### Annotated Code Examples and Expected Outputs

#### Example: Overloaded Constructors with `this` Chaining

```csharp
using System;

public class Coords
{
    public int X { get; set; }
    public int Y { get; set; }

    public Coords() : this(0, 0) { }
    public Coords(int x, int y) => (X, Y) = (x, y);
}

class Program
{
    static void Main()
    {
        var p1 = new Coords();
        Console.WriteLine($"{p1.X},{p1.Y}");   // 0,0
        var p2 = new Coords(5, 3);
        Console.WriteLine($"{p2.X},{p2.Y}");   // 5,3
    }
}
```

**Expected Output**

```
0,0
5,3
```

**Why this output occurs**

- The parameterless constructor chains to the two-parameter constructor with `this(0, 0)`.

---

### Real-World Cases

- **Entity creation** — Constructors that validate and assign.
- **Dependency injection** — Constructors that accept services.

---

### References

- Instance constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors
- Constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors

---

## 9. Destructors (Finalizers)

### Definitions

**Core Definition**
A finalizer (historically called a destructor) is a special method that runs before an object is garbage-collected, used to clean up unmanaged resources.

**Technical Definition**
Finalizers (historically referred to as destructors) are used to perform any necessary final clean-up when a class instance is being collected by the garbage collector. Finalizers cannot be defined in structs, only in classes. A class can have only one finalizer, which cannot be inherited or overloaded. Finalizers cannot be called explicitly; they are invoked automatically by the GC. A finalizer does not take modifiers or parameters. The `~ClassName()` syntax is used.

**Beginner-Friendly Explanation**
A finalizer is the **cleanup routine** that runs when the garbage collector is about to reclaim the object. It is rarely needed because the GC handles managed memory automatically.

---

### Purposes

- To release unmanaged resources (file handles, native memory).
- To provide a safety net when `Dispose` is not called.
- To implement the dispose pattern.

---

### Syntax Rules and Structure

```
~ClassName()
{
    // cleanup
}
```

**Component breakdown**

- `~` — The finalizer marker.
- `ClassName` — The class name.
- `()` — No parameters.

#### Syntax Rules

- No access modifiers, parameters, or return type.
- Cannot be called explicitly.
- Cannot be inherited or overloaded.
- Only classes can have finalizers.

#### Constraints and Limitations

- Non-deterministic; timing is controlled by the GC.
- Unnecessary finalizers degrade performance.
- Prefer `SafeHandle` or `IDisposable` over finalizers.

---

### Annotated Code Examples and Expected Outputs

#### Example: Finalizer

```csharp
using System;

public class ResourceHolder
{
    ~ResourceHolder()
    {
        Console.WriteLine("Finalizer running");
    }
}

class Program
{
    static void Main()
    {
        var r = new ResourceHolder();
    }
}
```

**Expected Output (eventually)**

```
Finalizer running
```

**Why this output occurs**

- The finalizer runs when the GC determines the object is unreachable; timing is non-deterministic.

---

### Real-World Cases

- **SafeHandle** — The preferred wrapper for unmanaged handles.
- **Legacy interop** — Wrapping native resources.

---

### References

- Finalizers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers
- Destructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/destructors

---

## 10. `this` Keyword

### Definitions

**Core Definition**
The `this` keyword refers to the current instance of the class. It is also used to chain constructors in the same class.

**Technical Definition**
The `this` keyword refers to the current instance of the class and is also used as a modifier of the first parameter of an extension method. In a constructor, `this(...)` can call another constructor in the same class, enabling constructor chaining. `this` is useful for disambiguating between instance members and parameters, and for passing the current object to other methods.

**Beginner-Friendly Explanation**
`this` means **“me”**—the current object. It is how an object refers to itself.

---

### Purposes

- To refer to the current instance's members.
- To disambiguate between fields and parameters with the same name.
- To chain constructors in the same class.
- To pass the current object as an argument.

---

### Syntax Rules and Structure

```
this.memberName
this(...)   // constructor chaining
```

**Component breakdown**

- `this.memberName` — Accesses the current instance's member.
- `this(...)` — Calls another constructor in the same class.

#### Syntax Rules

- `this` is available in instance methods, constructors, and property accessors.
- `this(...)` must be the first part of a constructor.
- `this` cannot be used in static members.

#### Constraints and Limitations

- Cannot be used in static contexts.
- Constructor chaining with `this(...)` must eventually call a constructor that does not chain to `this(...)`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Disambiguating and Chaining

```csharp
using System;

public class Person
{
    private string _name;

    public Person(string name) => this._name = name;
    public Person() : this("Unknown") { }

    public void Print() => Console.WriteLine(this._name);
}

class Program
{
    static void Main()
    {
        var p1 = new Person("Alice");
        p1.Print();   // Alice
        var p2 = new Person();
        p2.Print();   // Unknown
    }
}
```

**Expected Output**

```
Alice
Unknown
```

**Why this output occurs**

- `this._name` disambiguates the field from the parameter; `this("Unknown")` chains the parameterless constructor.

---

### Real-World Cases

- **Fluent APIs** — Returning `this` for chaining.
- **Constructor overloading** — Delegating to a primary constructor.

---

### References

- this keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/this
- Instance constructors — C# Programming Guide (this chaining) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors

---

## 11. `base` Keyword

### Definitions

**Core Definition**
The `base` keyword accesses members of the base class from within a derived class. It is also used to call a base-class constructor.

**Technical Definition**
The `base` keyword is used to access members of the base class from within a derived class. Use it if you want to call a method on the base class that has been overridden by another method, or specify which base-class constructor to call when creating instances of the derived class. `base` can be used in instance methods, constructors, and instance property accessors.

**Beginner-Friendly Explanation**
`base` means **“my parent”**. It lets a derived class call the base class's version of a method or constructor.

---

### Purposes

- To call an overridden base method.
- To call a base-class constructor.
- To access base-class members hidden by derived members.
- To extend base behaviour rather than replace it.

---

### Syntax Rules and Structure

```
base.memberName
base(...)   // base constructor call
```

**Component breakdown**

- `base.memberName` — Accesses a base-class member.
- `base(...)` — Calls a base-class constructor.

#### Syntax Rules

- `base` is available in instance contexts of a derived class.
- `base(...)` must be the first part of a constructor.
- The base method must be accessible (not private).

#### Constraints and Limitations

- Cannot be used in static members.
- Cannot access private base members.

---

### Annotated Code Examples and Expected Outputs

#### Example: Calling Base Method and Constructor

```csharp
using System;

public class Shape
{
    protected double X, Y;

    public Shape(double x, double y)
    {
        X = x;
        Y = y;
    }

    public virtual double Area() => 0;
}

public class Circle : Shape
{
    public Circle(double radius) : base(radius, 0) { }
    public override double Area() => Math.PI * X * X;
}

public class Cylinder : Circle
{
    public Cylinder(double radius, double height) : base(radius)
    {
        Y = height;
    }

    public override double Area() => (2 * base.Area()) + (2 * Math.PI * X * Y);
}

class Program
{
    static void Main()
    {
        var c = new Cylinder(2.5, 3.0);
        Console.WriteLine(c.Area().ToString("F2"));   // 86.39
    }
}
```

**Expected Output**

```
86.39
```

**Why this output occurs**

- `Cylinder` calls `base(radius)` to initialise the circle, and `base.Area()` to include the circular ends.

---

### Real-World Cases

- **Template method pattern** — Derived classes call `base` to extend behaviour.
- **Constructor initialisation** — Passing parameters to base constructors.

---

### References

- base keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/base

---

## 12. Object Creation (`new` Keyword)

### Definitions

**Core Definition**
The `new` operator creates a new instance of a class, allocating memory and invoking a constructor.

**Technical Definition**
Objects can be created by using the `new` keyword followed by the name of the class: `Customer object1 = new Customer();`. When an instance of a class is created, a reference to the object is passed back to the programmer. The `new` operator allocates memory on the managed heap, runs the constructor, and returns a reference. Object creation expressions can omit the constructor argument list when an object initializer is provided.

**Beginner-Friendly Explanation**
`new` is the **factory** that builds an object from a class blueprint.

---

### Purposes

- To allocate memory for a new object.
- To invoke the constructor.
- To obtain a reference to the new object.
- To combine creation with initialisation via object initializers.

---

### Syntax Rules and Structure

```
new ClassName()
new ClassName { Property = value }
new ClassName(args) { Property = value }
```

**Component breakdown**

- `new` — The operator.
- `ClassName` — The type to instantiate.
- `()` — Constructor arguments (optional with initializer).
- `{ ... }` — Object initializer (optional).

#### Syntax Rules

- The class must have an accessible constructor.
- `new` returns a reference to the object.
- Object initializers can be combined with constructor calls.

#### Constraints and Limitations

- Cannot instantiate abstract classes or interfaces directly.
- Cannot instantiate static classes.

---

### Annotated Code Examples and Expected Outputs

#### Example: Creating with `new`

```csharp
using System;

public class Person
{
    public string Name { get; set; }
}

class Program
{
    static void Main()
    {
        var p = new Person();
        p.Name = "Alice";
        Console.WriteLine(p.Name);   // Alice
    }
}
```

**Expected Output**

```
Alice
```

**Why this output occurs**

- `new Person()` allocates and constructs the object; `p` holds the reference.

---

### Real-World Cases

- **All object creation** — Every object is created with `new`.
- **Factory methods** — Encapsulating `new` behind a method.

---

### References

- new operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/new-operator
- Object creation expressions — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#128163-object-creation-expressions

---

## 13. Object Initialization

### Definitions

**Core Definition**
Object initialization is the process of assigning values to an object's fields and properties, either through constructor parameters or object initializer syntax.

**Technical Definition**
Object initializers let you assign values to any accessible fields or properties of an object when you create it, without requiring an explicit constructor call for each value. The syntax is `new ClassName { Property = value, ... }`. Object initializers can be combined with constructor arguments: `new ClassName(args) { Property = value }`. They are especially useful for immutable or read-only properties with `init` accessors.

**Beginner-Friendly Explanation**
Object initialization is **filling in the object's details** when you create it. You can use a constructor, an object initializer, or both.

---

### Purposes

- To set initial values for fields and properties.
- To improve readability of object creation.
- To initialise immutable properties with `init` accessors.
- To avoid temporary variables.

---

### Syntax Rules and Structure

```
new ClassName { Property1 = value1, Property2 = value2 }
new ClassName(args) { Property1 = value1 }
```

**Component breakdown**

- `{ ... }` — Initializer list.
- `Property = value` — Member assignment.

#### Syntax Rules

- Properties/fields must be accessible and settable (or `init`-only).
- The constructor argument list can be omitted if an initializer is provided.
- Initializers execute after the constructor body.

#### Constraints and Limitations

- Cannot set read-only fields (except via constructor).
- Cannot use initializers with abstract types.

---

### Annotated Code Examples and Expected Outputs

#### Example: Object Initializer

```csharp
using System;

public class Point
{
    public int X { get; init; }
    public int Y { get; init; }
}

class Program
{
    static void Main()
    {
        var p = new Point { X = 1, Y = 2 };
        Console.WriteLine($"{p.X}, {p.Y}");   // 1, 2
    }
}
```

**Expected Output**

```
1, 2
```

**Why this output occurs**

- The object initializer sets `X` and `Y` on the newly created `Point`.

---

### Real-World Cases

- **Configuration objects** — Setting options at creation.
- **DTOs** — Initialising data properties.
- **Records** — Using `with` expressions for non-destructive mutation.

---

### References

- Object and collection initializers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers

---

## 14. Object Lifecycle and Garbage Collection

### Definitions

**Core Definition**
The object lifecycle spans creation (allocation and construction), use, and reclamation (garbage collection and finalization). The CLR manages memory automatically.

**Technical Definition**
The memory-management lifecycle of an object is as follows: when the object is created, memory is allocated for it, the constructor is run, and the object is considered live. If the object or any of its instance fields cannot be accessed by any possible continuation of execution (other than finalizers), the object is considered no longer in use and becomes eligible for finalization. Once eligible, the finalizer (if any) runs at some unspecified later time. After the finalizer runs, if the object is still inaccessible, it becomes eligible for collection, and the GC eventually releases the memory. The GC uses generations (0, 1, 2) to optimise collection.

**Beginner-Friendly Explanation**
Objects are born (created), live (used), and eventually die (garbage-collected). The garbage collector handles death automatically—you do not free memory yourself.

---

### Purposes

- To understand when memory is allocated and reclaimed.
- To manage unmanaged resources correctly.
- To avoid memory leaks.
- To use `IDisposable` and `SafeHandle` appropriately.

---

### Syntax Rules and Structure (Lifecycle Stages)

| Stage | Description |
|---|---|
| **Created** | Memory allocated, constructor runs |
| **Live** | Object is reachable and usable |
| **Eligible for finalization** | Object no longer accessible |
| **Finalized** | Finalizer runs (if present) |
| **Collected** | GC releases memory |

**Component breakdown**

- **Constructor** — Initialises the object.
- **Finalizer** — Cleans up unmanaged resources.
- **GC** — Reclaims memory.

#### Syntax Rules

- The GC decides when to collect.
- Finalizers run non-deterministically.
- `GC.Collect()` can force collection but is discouraged.

#### Constraints and Limitations

- Cannot control GC timing.
- Finalizers delay collection.
- `IDisposable` provides deterministic cleanup.

---

### Annotated Code Examples and Expected Outputs

#### Example: Lifecycle with Finalizer

```csharp
using System;

public class Lifecycle
{
    public Lifecycle() => Console.WriteLine("Created");
    ~Lifecycle() => Console.WriteLine("Finalized");
}

class Program
{
    static void Main()
    {
        var obj = new Lifecycle();
        Console.WriteLine("Using");
        obj = null;
        GC.Collect();
        GC.WaitForPendingFinalizers();
        Console.WriteLine("Done");
    }
}
```

**Expected Output (approximate)**

```
Created
Using
Finalized
Done
```

**Why this output occurs**

- `GC.Collect()` forces collection; the finalizer runs before `Done`.

---

### Real-World Cases

- **Resource management** — `IDisposable` and `using` statements.
- **Caching** — Understanding when objects can be collected.

---

### References

- Automatic memory management — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/basic-concepts#79-automatic-memory-management
- Garbage collection fundamentals — .NET - https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals

---

## 15. Object Equality (`Equals`, `GetHashCode`, `ToString`)

### Definitions

**Core Definition**
`Equals`, `GetHashCode`, and `ToString` are three virtual methods inherited from `System.Object` that define an object's equality semantics, hash code, and string representation.

**Technical Definition**
The `Object` class is the ultimate base class of all .NET classes. Derived classes can override: `Equals` (supports comparisons between objects), `GetHashCode` (generates a number corresponding to the object's value for hash tables), and `ToString` (manufactures a human-readable text string describing the instance). By default, `Equals` tests reference equality. When overriding `Equals`, you should also override `GetHashCode` to maintain consistency. `ToString` returns the fully qualified type name by default.

**Beginner-Friendly Explanation**
`Equals` asks **“are these two objects the same?”** `GetHashCode` gives the object a **numeric fingerprint** for hash tables. `ToString` gives a **readable description** of the object.

---

### Purposes

- To define value equality for custom types.
- To support hash-based collections (`Dictionary`, `HashSet`).
- To provide meaningful string representations.
- To enable comparisons in tests and debugging.

---

### Syntax Rules and Structure

```
public override bool Equals(object? obj) { ... }
public override int GetHashCode() { ... }
public override string ToString() { ... }
```

**Component breakdown**

- `Equals` — Compares with another object.
- `GetHashCode` — Returns an `int` hash.
- `ToString` — Returns a `string`.

#### Syntax Rules

- Override `Equals` and `GetHashCode` together.
- If `x.Equals(y)` is true, `x.GetHashCode()` must equal `y.GetHashCode()`.
- `ToString` should be concise and informative.

#### Constraints and Limitations

- Default `Equals` is reference equality.
- Inconsistent `Equals`/`GetHashCode` breaks hash tables.
- `ToString` is not for serialisation.

---

### Annotated Code Examples and Expected Outputs

#### Example: Overriding `Equals`, `GetHashCode`, `ToString`

```csharp
using System;

public class Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y) => (X, Y) = (x, y);

    public override bool Equals(object? obj) =>
        obj is Point p && X == p.X && Y == p.Y;

    public override int GetHashCode() => HashCode.Combine(X, Y);

    public override string ToString() => $"({X}, {Y})";
}

class Program
{
    static void Main()
    {
        var p1 = new Point(1, 2);
        var p2 = new Point(1, 2);
        Console.WriteLine(p1.Equals(p2));   // True
        Console.WriteLine(p1);              // (1, 2)
    }
}
```

**Expected Output**

```
True
(1, 2)
```

**Why this output occurs**

- `Equals` compares `X` and `Y`; `GetHashCode` combines them; `ToString` formats the point.

---

### Real-World Cases

- **Value objects** — `Money`, `Address`, `Point`.
- **Collections** — Dictionary keys and HashSet elements.

---

### References

- System.Object class — .NET - https://learn.microsoft.com/en-us/dotnet/api/system.object
- Guidelines for Implementing Equals and the Equality Operator (==) — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-2.0/ms182243(v=vs.80)
- Object.Equals Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.equals
- Object.GetHashCode Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.gethashcode
- Object.ToString Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.tostring

---

## 16. Object Best Practices

### Definitions

**Core Definition**
Object best practices are the design guidelines for creating well-encapsulated, correctly-equality-managed, and maintainable classes and objects.

**Technical Definition**
Best practices for classes and objects derive from Microsoft's C# coding conventions, the .NET Framework Design Guidelines, and object-oriented design principles. Key recommendations include: **encapsulate data with properties** rather than public fields; **override `Equals` and `GetHashCode` together** for value-like types; **prefer `SafeHandle` and `IDisposable` over finalizers**; **use constructor chaining (`this`, `base`) to avoid duplication**; **prefer object initializers for readability**; **keep classes focused (single responsibility)**; **prefer composition over inheritance where appropriate**; and **make classes immutable where practical**.

**Beginner-Friendly Explanation**
Best practices are the habits that keep classes clean: **hide your data**, **define equality properly**, **do not rely on finalizers**, and **keep classes small and focused**.

---

### Purposes

- To improve encapsulation and maintainability.
- To ensure correct equality behaviour in collections.
- To avoid resource leaks.
- To make classes easier to test and reason about.
- To align with .NET conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Encapsulate data with properties
public string Name { get; private set; }

// Override Equals and GetHashCode together
public override bool Equals(object? obj) => ...;
public override int GetHashCode() => ...;

// Prefer SafeHandle/IDisposable over finalizers
// Use constructor chaining
public Person() : this("Unknown") { }

// Use object initializers
var p = new Person { Name = "Alice" };

// Keep classes focused (SRP)
// Prefer composition over inheritance where appropriate
```

**Component breakdown**

- **Properties over fields** — Encapsulation.
- **Equals + GetHashCode** — Correct hashing.
- **SafeHandle/IDisposable** — Deterministic cleanup.
- **Constructor chaining** — DRY initialisation.
- **Object initializers** — Readable creation.

#### Rules and Constraints

- **Do** use properties for data access.
- **Do** override `Equals` and `GetHashCode` together.
- **Do** implement `IDisposable` for unmanaged resources.
- **Do** use constructor chaining.
- **Do not** expose public fields.
- **Do not** write unnecessary finalizers.
- **Do not** make classes do too much.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Class

```csharp
using System;

public sealed class Temperature : IEquatable<Temperature>
{
    public double Celsius { get; }

    public Temperature(double celsius) => Celsius = celsius;

    public bool Equals(Temperature? other) =>
        other is not null && Celsius == other.Celsius;

    public override bool Equals(object? obj) => Equals(obj as Temperature);

    public override int GetHashCode() => Celsius.GetHashCode();

    public override string ToString() => $"{Celsius}°C";
}

class Program
{
    static void Main()
    {
        var t1 = new Temperature(25);
        var t2 = new Temperature(25);
        Console.WriteLine(t1.Equals(t2));   // True
        Console.WriteLine(t1);              // 25°C
    }
}
```

**Expected Output**

```
True
25°C
```

**Why this output occurs**

- The class is immutable, implements `IEquatable<T>`, overrides `Equals`/`GetHashCode` consistently, and provides a clear `ToString`.

---

### Real-World Cases

- **API design** — Public classes follow encapsulation and equality guidelines.
- **Domain modelling** — Value objects implement value equality.
- **Resource wrappers** — `SafeHandle` for unmanaged resources.

---

### References

- C# Coding Conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- Framework Design Guidelines — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/
- Guidelines for Implementing Equals and the Equality Operator (==) — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-2.0/ms182243(v=vs.80)
- Finalizers — C# Programming Guide (SafeHandle recommendation) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Finalizers (destructors) | Non-deterministic; prefer `IDisposable`/`SafeHandle`. |
| `this()` constructor chaining | Available since C# 1.0; stable. |
| `base()` constructor chaining | Available since C# 1.0; stable. |
| Object initializers | Introduced in C# 3.0. |
| Primary constructors (C# 12) | Version-specific; parameters in scope throughout the type. |
| `init` accessors (C# 9) | Version-specific; enables immutable object initializers. |
| Public fields | Discouraged; use properties. |
| Overriding `Equals` without `GetHashCode` | Breaks hash tables; violates guidelines. |
| `GC.Collect()` | Discouraged except in specific testing scenarios. |

---

## References (Consolidated)

- Classes — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes
- Object-Oriented Programming (C#) — Microsoft Docs - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/object-oriented-programming
- Members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/members
- Instance constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors
- Finalizers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers
- Object and collection initializers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers
- System.Object class — .NET - https://learn.microsoft.com/en-us/dotnet/api/system.object
- Automatic memory management — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/basic-concepts#79-automatic-memory-management
- Guidelines for Implementing Equals and the Equality Operator (==) — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-2.0/ms182243(v=vs.80)
- C# Coding Conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- Framework Design Guidelines — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/
- this keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/this
- base keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/base
- new operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/new-operator
- class keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/class

---